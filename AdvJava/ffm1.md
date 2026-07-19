# The Foreign Function & Memory API: Java's New Foundation for Native Interoperability

## Executive Summary

This talk covers Java's **Foreign Function & Memory API (FFM)**, finalized in Java 20 and refined for Java 22, which replaces older, clunkier mechanisms for calling native (C/C++) code and managing off-heap memory. It matters because modern Java applications increasingly need to interface with high-performance native libraries (for machine learning, GPU computation, etc.), and the old approach — **JNI** combined with the **direct buffer API** — is inflexible, unsafe, and hard to maintain. FFM introduces deterministic memory management, a Java-first programming model, and tooling to auto-generate native bindings.

## Core Concepts

### The Problem with JNI (Java Native Interface)

* **The Layman's Definition:** **JNI** is Java's traditional mechanism for calling native code, but it forces developers to declare "native methods" (like abstract methods with bodies defined in C/C++) and shifts most application logic into native code to minimize costly back-and-forth transitions.
* **How it Works / The Logic:** JNI is a **native-first programming model** — it's built to give native code access to Java functionality, not the reverse. Calling into it requires: writing native method declarations in Java, compiling with a special flag to generate C header files, implementing the function in C, compiling that with a C/C++ compiler, and producing a "shim" DLL. This shim is a *third* native library sitting between the application and the actual target library — extra complexity and deployment overhead. There's also no idiomatic way to pass data; developers often resort to passing raw memory addresses ("longs as pointers") stored in Java objects.
* **Example:** The speaker describes needing two native libraries just to call one target library — the desired library itself, plus the generated JNI shim DLL — which he calls "a little bit suboptimal."

### The Problem with Direct ByteBuffers

* **The Layman's Definition:** Before FFM, **direct buffers** (from the `ByteBuffer` API, introduced in Java 1.4) were the main way to work with off-heap memory, but they have serious limitations for modern use cases.
* **How it Works / The Logic:** Direct buffers have no deterministic way to be freed — off-heap memory tied to a buffer is only released once the garbage collector determines the buffer object is unreachable, introducing unpredictable latency. They also use `int` offsets, capping addressable memory at roughly 2GB, which is increasingly restrictive with persistent memory hardware. Addressing options are also limited: either a **relative addressing scheme** (mutating an internal index, which hurts JIT optimization) or fully explicit offsets scattered throughout code (making it brittle).
* **Example:** The speaker notes that using low-latency garbage collectors like **ZGC**, direct buffers can take significantly longer to be collected, since these GCs avoid frequently recomputing the full reachability graph — leaving large chunks of off-heap memory lingering unnecessarily.

| Old Approach | Limitation |
|---|---|
| JNI | Native-first model; requires shim DLLs; no idiomatic data-passing |
| Direct ByteBuffer | Non-deterministic deallocation (GC-dependent); ~2GB addressing limit; limited addressing schemes |

### Memory Segments

* **The Layman's Definition:** A **memory segment** is FFM's core abstraction representing a contiguous region of memory, replacing the role that `ByteBuffer` played for off-heap data.
* **How it Works / The Logic:** There are two kinds — **heap segments** (backed by on-heap memory) and **native segments** (backed by off-heap memory). Every segment has a defined size (accessing out of bounds throws an error), a lifetime (accessing a freed segment throws an exception), and optionally **confinement** to the thread that created it. Segments are allocated and populated by specifying byte offsets, similar to how `ByteBuffer` works.
* **Example:** To model a `Point` struct with `x` and `y` double fields (8 bytes each), the speaker allocates a 16-byte segment and writes a double value at offset 0 for `x` and offset 8 for `y`.

### Arenas and Deterministic Memory Management

* **The Layman's Definition:** An **Arena** is an abstraction that governs the *lifetime* of one or more memory segments — all segments allocated within the same arena share that lifetime, and can be explicitly and deterministically freed rather than waiting on the garbage collector.
* **How it Works / The Logic:** This is a **lifetime-centric approach**: first decide the intended lifetime of the memory, create an arena embodying that lifetime, then allocate within it. There are several arena types: the **global arena** (memory lives forever, never collected), the **automatic arena** (garbage-collected, similar to old `ByteBuffer` behavior), and the **confined** and **shared arenas**, which implement `AutoCloseable` — calling `close()` on them deterministically frees all memory allocated within, with strong safety guarantees (a freed segment can never be accessed again). For shared arenas, the JVM team used **safepointing mechanisms** (rather than costly locks) to guarantee a segment can't be closed while another thread is accessing it.
* **Example:** Using a `try`-with-resources block, the speaker creates a confined arena, allocates and populates a `Point` struct inside it, and when the block closes, all associated memory is immediately released — no garbage collector involvement required.

| Arena Type | Deallocation Behavior |
|---|---|
| Global Arena | Never collected; lives forever |
| Automatic Arena | Garbage-collected (similar to old ByteBuffer) |
| Confined Arena | Explicitly closed via `try`-with-resources; single-thread access |
| Shared Arena | Explicitly closed; safely accessible across multiple threads |

FFM's design goal is described as balancing two extremes:

| Extreme | Tradeoff |
|---|---|
| C-style manual memory management (`malloc`/`free`) | Very flexible, but unsafe (use-after-free, leaks) |
| Rust's strict ownership model | Very safe, but restrictive (e.g., linked lists become difficult) |

### Memory Layouts and VarHandles

* **The Layman's Definition:** A **memory layout** is a Java object that describes the structure of a native struct (its fields, sizes, and offsets), so developers no longer need to manually track raw byte offsets.
* **How it Works / The Logic:** Instead of hardcoding "offset 0" and "offset 8" for a struct's fields, a developer defines the layout as a Java object representing the C struct definition. From that layout, a **VarHandle** can be derived for each field, automatically encoding the offset computation. The layout object also directly informs the allocation call how much memory to reserve.
* **Example:** For the `Point2D` struct, the speaker derives one VarHandle for accessing field `x` and another for field `y`, eliminating manual offset math (`offset 8` for `y`) from the code entirely.

### The Native Linker and Calling C Functions

* **The Layman's Definition:** The **Native Linker** is the object that understands a platform's **calling convention** (the platform-specific rules for how function arguments are passed in registers or on the stack) and uses it to let Java code call native functions directly, or expose Java code as a callable native function pointer.
* **How it Works / The Logic:**
  1. Use layouts to describe a C function's signature (a **function descriptor**: one layout for the return type, one for each argument).
  2. Obtain the function's memory address via a **symbol lookup**.
  3. Call `downcallHandle` on the Native Linker, passing the address and function descriptor, to receive a **MethodHandle** representing the native function.
  4. Invoke the MethodHandle directly from Java, passing memory segments (e.g., a populated struct) as arguments.
  5. The Linker inspects the described signature and generates the exact machine instructions needed for that specific platform's calling convention — this can differ dramatically even on the same architecture.
* **Example:** The speaker shows a `distance` function taking a `Point` struct and returning a `double`. On **Linux** (System V calling convention), the point's fields are small enough to be passed directly in floating-point registers. On **Windows** (even on the same x64 architecture), the calling convention requires the struct to be spilled onto the stack, with a pointer to it stored in the `RCX` register — completely different assembly instructions for the same logical call, which the Native Linker handles automatically based on the described signature.

| Platform | Calling Convention for Point Struct |
|---|---|
| Linux (x64) | Fields passed directly in floating-point registers |
| Windows (x64) | Struct spilled to stack; pointer passed via RCX register |

### Restricted Methods and Safety

* **The Layman's Definition:** Because calling into native code is inherently unsafe (wrong function signatures, use-after-free, incorrect pointer sizes), FFM marks certain operations as **restricted methods** requiring explicit developer opt-in.
* **How it Works / The Logic:** Restricted methods (e.g., creating a downcall MethodHandle) currently only trigger a warning, but Java plans to eventually turn this into a hard error requiring the `--enable-native-access` command-line flag, which grants specific modules (or the classpath's unnamed module) permission to use restricted functionality. This is part of a broader push toward **integrity by default** — ensuring native code cannot violate Java invariants like mutating `final` fields.
* **Example:** (Synthesized Example) A team shipping a Java application that calls into a native compression library would need to explicitly pass `--enable-native-access=<module-name>` at startup once the warning becomes an enforced error, signaling intentional acceptance of the associated risk.

### The `jextract` Tool

* **The Layman's Definition:** **`jextract`** is a tool that automatically generates the Java bindings (layouts, VarHandles, MethodHandles, function descriptors) needed to call a native library, instead of requiring developers to hand-write all of that setup code.
* **How it Works / The Logic:** Point `jextract` at a library's header file (e.g., a standard C library header), and it generates static Java declarations that wrap the underlying MethodHandles in clean, easy-to-call static methods — including support for function-pointer arguments (like comparator callbacks) via generated factory methods that convert Java lambdas into native function pointers.
* **Example:** The speaker demonstrates generating bindings for C's `qsort` function (which takes a comparator function pointer). With `jextract`, calling `qsort` from Java requires only creating the function pointer from a lambda and calling the generated static wrapper — compared to the JNI equivalent, which requires native method declarations, a generated header file, and substantial C implementation code. The FFM-based approach is measured at roughly **2–3x faster** than a heavily optimized JNI implementation, particularly for **upcalls** (native code calling back into Java), where JNI historically left significant performance on the table.

## Key Takeaways & Quick Reference

* FFM (**Foreign Function & Memory API**) was finalized in Java 20 and refined in Java 22, replacing JNI and direct ByteBuffers for native interoperability and off-heap memory access.
* **Memory segments** replace ByteBuffers for modeling off-heap (and heap) memory, with defined size, lifetime, and optional thread confinement.
* **Arenas** provide deterministic, lifetime-centric memory management — confined and shared arenas can be explicitly closed via `try`-with-resources, unlike the GC-dependent cleanup of old direct buffers.
* **Memory layouts** and **VarHandles** let developers describe struct shapes and access fields without manual byte-offset math.
* The **Native Linker** reads platform-specific calling conventions and generates correct machine code to call native functions (downcalls) or expose Java functions to native code (upcalls) — no shim DLL required.
* **Restricted methods** currently warn (and will eventually require `--enable-native-access`) to flag inherently unsafe native operations.
* The **`jextract`** tool auto-generates Java bindings from native library headers, dramatically reducing boilerplate compared to JNI.
* FFM is roughly 2–3x faster than heavily optimized JNI, especially for upcalls from native code into Java.
* FFM is part of **Project Panama**, which also includes the Vector API (SIMD access) and Project Babylon (introspectable Java method bodies, usable for GPU kernel generation).

## Glossary of Terms

* **FFM (Foreign Function & Memory API)** — The finalized Java API (Java 20+) for calling native code and managing off-heap memory safely and efficiently.
* **JNI (Java Native Interface)** — Java's older, native-first mechanism for interoperating with native code via generated shim libraries.
* **Off-heap memory** — Memory allocated outside the Java heap, not managed by the garbage collector's normal object model.
* **Direct Buffer** — A `ByteBuffer` backed by off-heap memory, historically used for native interop before FFM.
* **Memory Segment** — FFM's abstraction for a contiguous region of memory (heap or native).
* **Arena** — An abstraction modeling the lifetime of one or more memory segments, supporting deterministic deallocation.
* **VarHandle** — A handle enabling typed, offset-aware access to a specific field within a memory segment.
* **MethodHandle** — A typed, directly invokable reference to a method (Java or native) used by FFM to represent callable functions.
* **Native Linker** — The FFM object that encodes platform-specific calling conventions to bridge Java and native function calls.
* **Downcall** — A call from Java code into native code.
* **Upcall** — A call from native code back into Java code.
* **Calling Convention** — Platform-specific rules governing how function arguments and return values are passed (registers vs. stack).
* **`jextract`** — A tool that generates Java FFM bindings automatically from native library header files.
* **Restricted Method** — An FFM method flagged as inherently unsafe, requiring explicit opt-in via a JVM flag.
* **Project Panama** — The broader OpenJDK project encompassing FFM, the Vector API, and Project Babylon.
* **Scalarization** — A planned JVM optimization (tied to Project Valhalla/value classes) to eliminate allocation overhead for memory segment objects.
