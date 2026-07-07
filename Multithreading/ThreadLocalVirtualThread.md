# ThreadLocal & Virtual Threads vs Platform Threads
## Java Multithreading

---

## 1. ThreadLocal

### What is ThreadLocal?
- Every thread in Java has its own **ThreadLocal variable** storage
- `ThreadLocal<T>` is a generic class — can hold String, Integer, or any object
- Each thread gets its **own independent copy** of the variable
- Only **one `ThreadLocal` object** needed — each thread automatically uses its own slot

```
┌─────────────────────────────────────────────────────────┐
│                   ThreadLocal Concept                    │
│                                                          │
│  ThreadLocal<String> tl = new ThreadLocal<>();           │
│                                                          │
│  Main Thread      Thread-1         Thread-2             │
│  ┌──────────┐    ┌──────────┐     ┌──────────┐          │
│  │ tl =     │    │ tl =     │     │ tl =     │          │
│  │ "main"   │    │ "pool-1" │     │ "pool-2" │          │
│  └──────────┘    └──────────┘     └──────────┘          │
│                                                          │
│  Same object, but INDEPENDENT values per thread!        │
└─────────────────────────────────────────────────────────┘
```

---

### How ThreadLocal Works Internally

When you call `threadLocal.set(value)`:

```
Step 1: Get the CURRENT running thread
        → Thread.currentThread()

Step 2: Fetch that thread's internal ThreadLocalMap

Step 3: Store the value in that thread's map
        → No need to specify which thread explicitly!
```

When you call `threadLocal.get()`:
```
Step 1: Get the CURRENT running thread
Step 2: Fetch that thread's ThreadLocalMap
Step 3: Return the value stored for this thread
```

> **Key Insight**: You never specify which thread — Java automatically uses the currently executing thread.

---

### Basic Example

```java
ThreadLocal<String> threadLocal = new ThreadLocal<>();

// Main thread sets its own value
threadLocal.set(Thread.currentThread().getName());
// Main thread's ThreadLocal = "main"

// New thread sets its own value
Thread t1 = new Thread(() -> {
    threadLocal.set(Thread.currentThread().getName());
    // Thread-1's ThreadLocal = "Thread-1" (separate copy!)

    String val = threadLocal.get(); // returns "Thread-1"
    System.out.println(val);
});
t1.start();

// Main thread gets its own value (unaffected by t1)
String mainVal = threadLocal.get(); // returns "main"
System.out.println(mainVal);
```

**Output:**
```
Thread-1
main
```

---

### ⚠️ Critical: ThreadLocal with Thread Pools

**The Problem**: Thread pools **reuse** threads across multiple tasks.

```
Thread Pool (5 threads): [T1] [T2] [T3] [T4] [T5]

Task 1 → picked by T1 → threadLocal.set("task1-data")
Task 1 completes → T1 goes back to pool
                              ↓
Task 6 → picked by T1 (reused!) → threadLocal.get()
                                   returns "task1-data" ← STALE DATA!
```

**Real Output Without Cleanup:**
```
// Submitted 15 tasks after setting ThreadLocal for first task
// Expected: all 15 should have null
// Actual: 3 tasks returned stale value because same threads were reused!
```

**The Fix: Always call `remove()` when done**

```java
ExecutorService pool = Executors.newFixedThreadPool(5);

pool.submit(() -> {
    try {
        threadLocal.set(Thread.currentThread().getName());
        // ... do your task work ...
        System.out.println(threadLocal.get());
    } finally {
        threadLocal.remove(); // ← ALWAYS clean up!
    }
});
```

**Output after fix:**
```
// All 15 subsequent tasks → null (clean state guaranteed)
```

---

### ThreadLocal Summary

| Feature | Detail |
|---------|--------|
| Scope | Per-thread isolation |
| Type | Generic `ThreadLocal<T>` |
| Set | `threadLocal.set(value)` |
| Get | `threadLocal.get()` |
| Clean | `threadLocal.remove()` |
| Auto-target | Always targets current running thread |
| Risk | Stale data if thread is reused without cleanup |

---

## 2. Virtual Threads vs Platform (Normal) Threads

### Platform Thread (Normal Thread — what we used till now)

**Definition**: A platform thread is a JVM **wrapper around an OS thread**.

```
Java Code           JVM              OS
─────────           ───              ──
new Thread()   →   JVM wrapper  →   OS Thread (1:1 mapping)
t.start()      →   System call  →   OS creates native thread
```

**Key Facts:**
- Every platform thread = 1 OS thread (1:1 mapping)
- Platform threads are **managed by JVM**
- JVM is just a **mediator** between you and OS threads
- Creating a thread = making an expensive **system call** to OS

---

### Disadvantages of Platform Threads

#### Disadvantage 1: Slow & Expensive Thread Creation

```
t1.start()
    ↓
JVM calls OS (system call) → "Hey OS, create a native thread"
    ↓
OS creates thread (takes 2-3ms per thread)
    ↓
Expensive! That's why we use ThreadPoolExecutor
(pre-create threads to avoid repeated creation)
```

#### Disadvantage 2: Wasted OS Thread During I/O Waits

```
Thread-1 (attached to OS-Thread-1)
    │
    ├── executes some logic
    │
    ├── makes DB call (waiting 4 seconds...)
    │       ↓
    │   OS-Thread-1 is BLOCKED and IDLE for 4 seconds
    │   Cannot serve any other work!
    │       ↓
    └── DB response received → continues
```

```
Platform Thread Model (1:1):

OS-Thread-1  ──── Platform-Thread-1 (WAITING for DB)
OS-Thread-2  ──── Platform-Thread-2 (WAITING for API)
OS-Thread-3  ──── Platform-Thread-3 (running)

OS-Thread-1 and OS-Thread-2 are completely WASTED
during their wait periods!
```

---

### Virtual Threads (JDK 19+)

**Definition**: Virtual threads are **JVM-managed objects** that are NOT directly tied to OS threads.

```
Virtual Thread Model (M:N mapping):

OS-Thread-1  ─┐
               ├── Many virtual threads share few OS threads
OS-Thread-2  ─┘

Virtual threads: [VT1] [VT2] [VT3] ... [VT10000]
                  (can create thousands!)
```

---

### How Virtual Threads Work

**Normal execution:**
```
VT1 wants to run → attached to OS-Thread-1 → executes
VT2 wants to run → attached to OS-Thread-2 → executes
```

**When a Virtual Thread waits (I/O, DB call, etc.):**
```
VT1 starts DB call (enters waiting state)
    ↓
JVM DETACHES VT1 from OS-Thread-1
    ↓
OS-Thread-1 is now FREE
    ↓
JVM ATTACHES VT3 (ready to run) to OS-Thread-1
    ↓
When VT1's DB call completes → reattached to any free OS thread
```

```
Virtual Thread Model During I/O:

                    VT1 (waiting for DB)
                   /
OS-Thread-1 ──── VT3 (running — was waiting)
                   \
                    VT5 (next in queue)

OS-Thread-1 never sits idle!
```

---

### Platform Thread vs Virtual Thread — Full Comparison

| Feature | Platform Thread | Virtual Thread |
|---------|----------------|----------------|
| Mapping to OS Thread | 1:1 (always attached) | M:N (dynamic attachment) |
| Managed by | JVM (wrapper around OS thread) | JVM (pure JVM object) |
| Creation cost | Expensive (system call) | Cheap (just a JVM object) |
| Max practical count | ~Thousands | Millions |
| I/O blocking | OS thread blocked too | OS thread freed, reused |
| Use case | CPU-bound tasks | High-throughput I/O-bound tasks |
| Goal | - | Higher **throughput** (not latency) |
| Available from | Always | JDK 19+ |
| Backward compatible | - | ✅ Yes |

> 🎯 **Key Goal of Virtual Threads**: **Higher Throughput**
> - Old: handle 100 requests/sec
> - New: handle 1000 requests/sec
> - Throughput = how many tasks completed per second

---

### Creating Virtual Threads

**Method 1: Using Thread class**
```java
// Create and start a virtual thread directly
Thread vt = Thread.ofVirtual().start(() -> {
    System.out.println("Running in virtual thread: "
        + Thread.currentThread());
});
```

**Method 2: Using ExecutorService (Recommended)**
```java
// Create executor that uses virtual threads
ExecutorService executor =
    Executors.newVirtualThreadPerTaskExecutor();

// Submit tasks — each runs in its own virtual thread
executor.submit(() -> {
    // DB call, API call, any I/O work
    System.out.println("Task running in: "
        + Thread.currentThread());
});

executor.shutdown();
```

---

### When to Use What?

```
┌─────────────────────────────────────────────────────┐
│                  Decision Guide                      │
│                                                      │
│  Is your task CPU-intensive?                         │
│  (heavy computation, no I/O waits)                   │
│            │                                         │
│           YES → Use Platform Thread / ThreadPool     │
│            │                                         │
│           NO → Is it I/O-bound?                     │
│               (DB calls, API calls, file reads)      │
│                         │                            │
│                        YES → Use Virtual Thread ✅   │
└─────────────────────────────────────────────────────┘
```

---

### Why Virtual Threads Don't Improve Latency

> Virtual threads improve **throughput**, not **latency**.

```
Latency  = time taken for ONE request to complete
           (Virtual threads don't make your DB call faster)

Throughput = how many requests served per second
           (More OS threads free → more tasks can run → higher throughput)
```

---

### Visual Summary

```
PLATFORM THREAD MODEL
──────────────────────
[OS-T1]──[PT1 waiting for DB]   ← OS thread WASTED
[OS-T2]──[PT2 running]
[OS-T3]──[PT3 waiting for API]  ← OS thread WASTED
Result: Only 1 of 3 OS threads doing real work

VIRTUAL THREAD MODEL
─────────────────────
[OS-T1]──[VT2 running]         ← VT1 detached (waiting)
[OS-T2]──[VT4 running]         ← VT3 detached (waiting)
VT1 & VT3 waiting, but OS threads free to serve VT2 & VT4!
Result: All OS threads doing real work → higher throughput!
```

---

## Quick Interview Answers

| Question | Answer |
|----------|--------|
| What is ThreadLocal? | Per-thread isolated variable storage, one object serves all threads |
| When to call `remove()`? | Always, when thread is reused (thread pool scenarios) |
| What is platform thread? | JVM wrapper around OS thread, 1:1 mapping |
| What is virtual thread? | JVM-managed object, M:N mapping with OS threads, JDK 19+ |
| Why virtual threads? | Higher throughput — OS threads not wasted during I/O waits |
| Virtual thread improves latency? | ❌ No — only throughput |
| Can we have millions of virtual threads? | ✅ Yes — they're just JVM objects |
| Are virtual threads backward compatible? | ✅ Yes — same API, different management |
