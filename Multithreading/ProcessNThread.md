# Java Multithreading - Complete Notes (Part 1)

## Table of Contents
1. [Process vs Thread](#process-vs-thread)
2. [What is a Process?](#what-is-a-process)
3. [What is a Thread?](#what-is-a-thread)
4. [JVM Memory Structure](#jvm-memory-structure)
5. [Memory Segments Deep Dive](#memory-segments-deep-dive)
6. [Complete Execution Flow](#complete-execution-flow)
7. [Context Switching](#context-switching)
8. [Multithreading Benefits & Challenges](#multithreading-benefits--challenges)
9. [Multitasking vs Multithreading](#multitasking-vs-multithreading)

---

## Process vs Thread

### High-Level Overview

```
Process
  ├── Thread 1
  ├── Thread 2
  └── Thread 3
```

### Quick Comparison

| Aspect | Process | Thread |
|--------|---------|--------|
| **Definition** | Instance of executing program | Smallest unit of execution |
| **Resources** | Own memory space | Shares process memory |
| **Independence** | Independent | Dependent on process |
| **Communication** | Expensive (IPC) | Cheap (shared memory) |
| **Creation** | Heavy | Lightweight |

---

## What is a Process?

### Definition

**Process**: An instance of a program that is being executed.

### How Process is Created

```java
// Step 1: Write code
// File: Test.java
public class Test {
    public static void main(String[] args) {
        System.out.println("Hello World");
    }
}

// Step 2: Compile
// Command: javac Test.java
// Output: Test.class (bytecode)

// Step 3: Execute
// Command: java Test
// ✅ JVM creates a NEW PROCESS
```

### Process Creation Flow

```
Write Code (Test.java)
        ↓
Compile (javac Test.java)
        ↓
Bytecode (Test.class)
        ↓
Execute (java Test)
        ↓
JVM creates PROCESS
        ↓
Allocates Resources (Memory, Threads, etc.)
        ↓
Program Executes
```

---

## Process Characteristics

### 1. Own Resources

Each process has **independent resources**:

```
Process 1                    Process 2
├── Heap Memory (separate)   ├── Heap Memory (separate)
├── Stack                    ├── Stack
├── Code Segment             ├── Code Segment
├── Data Segment             ├── Data Segment
└── Threads                  └── Threads
```

**Key Point**: Processes **DO NOT share** memory with each other!

### 2. JVM Instance Per Process

```
Process 1 → JVM Instance 1 → Own Heap (e.g., 2GB)
Process 2 → JVM Instance 2 → Own Heap (e.g., 1GB)
Process 3 → JVM Instance 3 → Own Heap (e.g., 3GB)
```

### 3. Configuring Heap Size

```bash
# Syntax
java -Xms<initial> -Xmx<maximum> ClassName

# Example
java -Xms256m -Xmx2g Test
```

| Flag | Meaning | Example |
|------|---------|---------|
| `-Xms` | Initial heap size | `-Xms256m` (256 MB) |
| `-Xmx` | Maximum heap size | `-Xmx2g` (2 GB) |

**Note**: Even if JVM has 10GB total heap, each process gets only allocated heap!

---

## What is a Thread?

### Definition

**Thread**: The smallest sequence of instructions that can be executed by CPU independently.

Also called: **Lightweight Process**

### Thread Creation

**When process starts**:
- Automatically creates **1 thread** → **Main Thread**
- From main thread, you can create more threads

### Visual Example

```java
public class MultiThreadingLearning {
    public static void main(String[] args) {
        // This prints: main
        System.out.println(Thread.currentThread().getName());
    }
}
```

**Output**: `main`

**Why?** First thread created is always the **main thread**.

---

## Thread Characteristics

### 1. Multiple Threads in One Process

```
Process
  ├── Main Thread ──→ Executes main()
  ├── Thread 1 ──────→ Custom task
  └── Thread 2 ──────→ Custom task
```

### 2. Share Process Resources

```
Process Memory (Shared by all threads)
├── Heap ✅ (Shared)
├── Code Segment ✅ (Shared)
├── Data Segment ✅ (Shared)
│
Per-Thread Memory (NOT Shared)
├── Stack (Each thread has own)
├── Register (Each thread has own)
└── Program Counter (Each thread has own)
```

---

## JVM Memory Structure

### Complete Memory Layout

```
JVM Instance
├── Heap (Shared)
├── Code Segment (Shared)
├── Data Segment (Shared)
│
Per Thread:
├── Stack (Thread-local)
├── Register (Thread-local)
└── Program Counter (Thread-local)
```

### Visual Representation

```
┌─────────────────────────────────────┐
│         JVM Instance                │
│                                     │
│  ┌──────────────────────────────┐  │
│  │    Heap Memory (Shared)      │  │
│  └──────────────────────────────┘  │
│                                     │
│  ┌──────────────────────────────┐  │
│  │  Code Segment (Shared)       │  │
│  │  (Machine code)              │  │
│  └──────────────────────────────┘  │
│                                     │
│  ┌──────────────────────────────┐  │
│  │  Data Segment (Shared)       │  │
│  │  (Global/Static variables)   │  │
│  └──────────────────────────────┘  │
│                                     │
│  Thread 1        Thread 2           │
│  ┌──────┐        ┌──────┐          │
│  │Stack │        │Stack │          │
│  │Reg   │        │Reg   │          │
│  │PC    │        │PC    │          │
│  └──────┘        └──────┘          │
└─────────────────────────────────────┘
```

---

## Memory Segments Deep Dive

### 1. Code Segment

**Stores**: Compiled machine code (bytecode → machine code)

**Properties**:
- ✅ **Shared** by all threads
- ✅ **Read-only** (cannot be modified)
- Contains CPU-executable instructions

**Flow**:
```
Bytecode (Test.class)
        ↓
JVM Interpreter/JIT Compiler
        ↓
Machine Code
        ↓
Stored in Code Segment
```

---

### 2. Data Segment

**Stores**: Global and static variables

**Properties**:
- ✅ **Shared** by all threads
- ✅ **Read & Write** (threads can modify)
- ⚠️ **Needs synchronization** (race conditions possible)

**Example**:
```java
public class Example {
    static int counter = 0;  // Stored in Data Segment
    
    public static void main(String[] args) {
        counter++;  // All threads can access/modify
    }
}
```

---

### 3. Heap

**Stores**: Objects created with `new` keyword

**Properties**:
- ✅ **Shared** by all threads within same process
- ❌ **NOT shared** across processes
- ✅ **Read & Write** (threads can modify)
- ⚠️ **Needs synchronization**

**Example**:
```java
public class Example {
    public static void main(String[] args) {
        String name = new String("Java");  // Allocated in Heap
        List<Integer> list = new ArrayList<>();  // Allocated in Heap
    }
}
```

---

### 4. Stack (Thread-Local)

**Stores**: Method calls, local variables

**Properties**:
- ✅ **Each thread has own stack**
- ❌ **NOT shared** between threads
- Follows LIFO (Last In First Out)

**Example**:
```java
public void method1() {
    int x = 10;  // Stored in Thread's Stack
    method2(x);
}

public void method2(int y) {
    int z = y + 5;  // Stored in Thread's Stack
}
```

**Stack Frame**:
```
Thread 1 Stack       Thread 2 Stack
┌──────────────┐    ┌──────────────┐
│ method2()    │    │ method3()    │
│ z = 15       │    │ a = 20       │
├──────────────┤    ├──────────────┤
│ method1()    │    │ method1()    │
│ x = 10       │    │ x = 30       │
└──────────────┘    └──────────────┘
```

---

### 5. Register (Thread-Local)

**Stores**: Intermediate values during bytecode → machine code conversion

**Properties**:
- ✅ **Each thread has own register**
- ❌ **NOT shared**
- Used by JIT compiler for optimization
- Critical for **context switching**

**Purpose**:
- Store intermediate computation results
- Optimize instruction execution
- Save thread state during context switch

---

### 6. Program Counter (PC) (Thread-Local)

**Stores**: Address of current instruction being executed

**Properties**:
- ✅ **Each thread has own PC**
- ❌ **NOT shared**
- Points to instruction in Code Segment

**Example**:
```
Code Segment:
[0x100] instruction1
[0x104] instruction2
[0x108] instruction3

Thread 1 PC: 0x100  (executing instruction1)
Thread 2 PC: 0x108  (executing instruction3)
```

---

## Complete Execution Flow

### Step-by-Step Process

#### Step 1: Write Code
```java
// Main.java
public class Main {
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            System.out.println("Thread 1");
        });
        
        Thread t2 = new Thread(() -> {
            System.out.println("Thread 2");
        });
        
        t1.start();
        t2.start();
    }
}
```

#### Step 2: Compile
```bash
javac Main.java
# Output: Main.class (bytecode)
```

#### Step 3: Execute
```bash
java Main
```

#### Step 4: Process Creation
```
JVM creates Process
        ↓
Allocates JVM Instance
        ↓
JVM Instance has:
  - Heap (1GB)
  - Code Segment
  - Data Segment
```

#### Step 5: Bytecode → Machine Code
```
JVM Interpreter/JIT Compiler
        ↓
Converts bytecode → machine code
        ↓
Stores in Code Segment
```

#### Step 6: Thread Creation
```
JVM creates 3 threads:
  - Main Thread
  - Thread 1 (t1)
  - Thread 2 (t2)

Each thread gets:
  - Own Stack
  - Own Register
  - Own Program Counter
```

#### Step 7: Program Counter Assignment
```
Main Thread PC → Points to main() instructions
Thread 1 PC    → Points to t1 lambda instructions
Thread 2 PC    → Points to t2 lambda instructions
```

#### Step 8: CPU Execution
```
OS Scheduler assigns threads to CPU
        ↓
CPU executes instructions
        ↓
Uses Register for intermediate results
        ↓
Program Counter increments after each instruction
```

---

## Context Switching

### What is Context Switching?

**Definition**: Saving state of one thread and loading state of another thread.

**Why needed?** When there are more threads than CPU cores.

### Scenario: 1 CPU, 3 Threads

```
Timeline:
[0-1s]  CPU executes Thread 1 (50% complete)
        ↓
[1s]    Context Switch!
        - Save Thread 1 state to Register
        - Load Thread 2 state from Register
        ↓
[1-2s]  CPU executes Thread 2 (70% complete)
        ↓
[2s]    Context Switch!
        - Save Thread 2 state to Register
        - Load Thread 3 state from Register
        ↓
[2-3s]  CPU executes Thread 3 (30% complete)
        ↓
[3s]    Context Switch!
        - Save Thread 3 state to Register
        - Load Thread 1 state from Register
        ↓
[3-4s]  CPU resumes Thread 1 from 50%
```

### Context Switch Process

```
Thread 1 executing
        ↓
Time slice expires (e.g., 1 second)
        ↓
OS saves Thread 1 state:
  - Register values
  - Program Counter value
  - Stack pointer
        ↓
OS loads Thread 2 state:
  - Restore Register values
  - Restore Program Counter
  - Restore Stack pointer
        ↓
Thread 2 starts executing
```

### Why Register is Important

**Register saves**:
- Intermediate computation results
- CPU state
- Allows thread to resume from exact point

**Without Register**: Thread would have to restart from beginning!

---

## Parallelism vs Concurrency

### 1 CPU Core (Concurrency)

```
Thread 1: ████░░░░████░░░░████
Thread 2: ░░░░████░░░░████░░░░
Thread 3: ░░░░░░░░░░░░░░░░░░░░████

Time →
```

**Appears** parallel, but uses context switching.

### 4 CPU Cores (True Parallelism)

```
CPU 1: Thread 1 ████████████████
CPU 2: Thread 2 ████████████████
CPU 3: Thread 3 ████████████████
CPU 4: Idle     ░░░░░░░░░░░░░░░░

Time →
```

**Actually** runs in parallel!

---

## Multithreading Benefits & Challenges

### Benefits ✅

#### 1. Improved Performance
```java
// Without multithreading
processTask1();  // 2 seconds
processTask2();  // 2 seconds
// Total: 4 seconds

// With multithreading (2 CPU cores)
Thread t1 = new Thread(() -> processTask1());
Thread t2 = new Thread(() -> processTask2());
t1.start();
t2.start();
// Total: 2 seconds (parallel execution)
```

#### 2. Better Responsiveness
```java
// UI remains responsive while background task runs
new Thread(() -> {
    downloadLargeFile();  // Long-running task
}).start();

// UI can still respond to user clicks
```

#### 3. Resource Sharing
- Threads share heap, code, data segments
- Less memory overhead than multiple processes
- Efficient use of system resources

---

### Challenges ⚠️

#### 1. Concurrency Issues

**Race Condition**:
```java
class Counter {
    private int count = 0;
    
    public void increment() {
        count++;  // Not atomic! Race condition!
    }
}

// Thread 1 and Thread 2 both call increment()
// Expected: count = 2
// Actual: count = 1 (race condition)
```

**Deadlock**:
```java
// Thread 1 holds Lock A, waits for Lock B
// Thread 2 holds Lock B, waits for Lock A
// Both stuck forever! 💀
```

#### 2. Synchronization Overhead
```java
public synchronized void increment() {
    count++;  // Safe but slower (locking overhead)
}
```

#### 3. Complex Debugging
- Non-deterministic behavior
- Hard to reproduce bugs
- Difficult to write unit tests

---

## Multitasking vs Multithreading

### Visual Comparison

```
Multitasking (Multiple Processes):
┌─────────────┐    ┌─────────────┐
│  Process 1  │    │  Process 2  │
│  ┌────────┐ │    │  ┌────────┐ │
│  │ Heap   │ │    │  │ Heap   │ │
│  │ Code   │ │    │  │ Code   │ │
│  │ Data   │ │    │  │ Data   │ │
│  └────────┘ │    │  └────────┘ │
└─────────────┘    └─────────────┘
     ❌ NO SHARING


Multithreading (Multiple Threads):
┌───────────────────────────┐
│       Process             │
│  ┌──────────────────────┐ │
│  │  Heap (Shared)       │ │
│  │  Code (Shared)       │ │
│  │  Data (Shared)       │ │
│  └──────────────────────┘ │
│                           │
│  Thread 1    Thread 2     │
│  ┌──────┐   ┌──────┐     │
│  │Stack │   │Stack │     │
│  └──────┘   └──────┘     │
└───────────────────────────┘
     ✅ SHARE RESOURCES
```

### Comparison Table

| Feature | Multitasking | Multithreading |
|---------|-------------|----------------|
| **Unit** | Process | Thread |
| **Memory** | Separate | Shared |
| **Communication** | IPC (expensive) | Shared memory (cheap) |
| **Creation** | Heavy | Lightweight |
| **Context Switch** | Expensive | Cheaper |
| **Independence** | Fully independent | Dependent on process |
| **Example** | Chrome + Word + Spotify | Multiple tabs in Chrome |

---

## Key Takeaways

### Process
✅ Instance of executing program  
✅ Has own memory space (heap, code, data)  
✅ Each process has own JVM instance  
✅ Processes don't share resources  
✅ Heavy to create  

### Thread
✅ Lightweight unit of execution  
✅ Shares process memory (heap, code, data)  
✅ Has own stack, register, program counter  
✅ Multiple threads per process  
✅ Cheap to create  

### Memory Segments
✅ **Shared**: Heap, Code Segment, Data Segment  
✅ **Thread-local**: Stack, Register, Program Counter  
✅ Synchronization needed for shared memory  

### Context Switching
✅ Saves thread state in register  
✅ Loads another thread state  
✅ Allows concurrent execution on limited CPUs  
✅ Has performance overhead  

### Parallelism
✅ **Concurrency**: Appears parallel (context switching)  
✅ **True Parallelism**: Multiple CPU cores  

---

## Interview Questions

### Q1: Difference between Process and Thread?
**Answer**: 
- **Process**: Instance of executing program with own memory space
- **Thread**: Lightweight execution unit sharing process memory
- Processes are independent; threads share resources
- Process creation is heavy; thread creation is lightweight

### Q2: What happens when you run `java Main`?
**Answer**:
1. JVM creates a new process
2. Allocates JVM instance with heap, code, data segments
3. JIT/Interpreter converts bytecode to machine code
4. Creates main thread automatically
5. Main thread starts executing from main() method

### Q3: Why do threads share heap but not stack?
**Answer**:
- **Heap**: Objects need to be shared across threads (communication)
- **Stack**: Method calls and local variables are thread-specific
- Shared heap enables inter-thread communication
- Separate stacks ensure thread independence

### Q4: What is context switching?
**Answer**: Process of saving current thread's state (register, PC) and loading another thread's state. Required when threads > CPU cores. Has performance overhead.

### Q5: Multitasking vs Multithreading?
**Answer**:
- **Multitasking**: Multiple processes, no shared memory
- **Multithreading**: Multiple threads in one process, shared memory
- Multitasking is heavyweight; Multithreading is lightweight

---

*Happy Coding! 🚀*
