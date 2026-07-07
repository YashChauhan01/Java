# Atomic Variables, Volatile & Concurrent Collections
## Java Multithreading

---

## 1. Ways to Achieve Concurrency

There are **two mechanisms** to achieve concurrency in Java:

```
┌─────────────────────────────────────────────────────┐
│              Concurrency Mechanisms                  │
├──────────────────────┬──────────────────────────────┤
│   Lock-Based         │   Lock-Free                  │
│                      │                              │
│ - synchronized       │ - Atomic Variables           │
│ - ReentrantLock      │   (AtomicInteger, etc.)      │
│ - StampedLock        │                              │
│ - ReadWriteLock      │                              │
│ - Semaphores         │                              │
│                      │                              │
│ More versatile       │ Faster but very specific     │
│ Works for complex    │ use case only                │
│ business logic       │                              │
└──────────────────────┴──────────────────────────────┘
```

> ⚠️ **Important**: Lock-free is NOT an alternative to lock-based. It works only for very specific use cases (read-modify-update). For complex business logic, always use lock-based.

---

## 2. Compare and Swap (CAS)

### What is CAS?
- A **low-level CPU operation** supported by all modern processors
- The technique behind lock-free concurrency
- **Atomic** — guaranteed to be a single unit even across multiple CPU cores
- Inspired **Optimistic Concurrency Control** at the DB level

### CAS Parameters
CAS accepts **three parameters**:

```
CAS(memory, expectedValue, newValue)
  │             │               │
  │             │               └── Value to write if match succeeds
  │             └────────────────── What we expect the memory to hold
  └──────────────────────────────── Memory location of variable
```

### CAS Operation Steps

```
Step 1: READ    → Load variable value from memory (M1)
Step 2: COMPARE → Is memory value == expectedValue?
Step 3: SWAP    → If YES → update memory with newValue
                  If NO  → retry (loop back to Step 1)
```

### CAS vs Optimistic Concurrency Control

| Feature | CAS | Optimistic Concurrency Control |
|--------|-----|-------------------------------|
| Level | CPU / Hardware | Database |
| Versioning | ABA problem (version needed) | Row version column |
| Speed | Extremely fast | Slower (DB round trip) |
| Use case | In-memory atomic ops | DB concurrent updates |

**Both work on the same principle**: Read → Compare → Update only if unchanged.

### CAS Example

```
Memory M1: x = 10

Thread 1 calls: CAS(M1, expected=10, new=12)

CAS Step 1: Read M1 → value = 10
CAS Step 2: Compare 10 == 10 ✅
CAS Step 3: Update M1 → x = 12  ✓ SUCCESS
```

---

## 3. ABA Problem

### What is it?
A subtle issue with CAS where a value changes from A → B → A, making CAS think nothing changed.

```
Initial: M1 = 10 (version 1)

Thread 2 changes: 10 → 12  (version 2)
Thread 3 changes: 12 → 10  (version 3)

Thread 1 runs CAS(M1, expected=10, new=13)
CAS sees: memory=10, expected=10 → MATCH ✅
But the "10" is NOT the same 10 thread 1 originally read!
```

### Solution: Add a Version/Timestamp

```
Read:  (value=10, version=1)
After changes: (value=10, version=3)

CAS(M1, expected=(10, version=1), new=(13, version=4))
Compare: version=1 ≠ version=3 → FAIL ✅ Correctly rejected!
```

> Java's `AtomicStampedReference` solves the ABA problem using exactly this approach.

---

## 4. Why `counter++` is NOT Atomic

### The Problem

```java
class SharedResource {
    int counter = 0;

    void increment() {
        counter++;  // NOT atomic!
    }
}
```

`counter++` expands into **3 non-atomic steps**:

```
Step 1: LOAD    → read counter value from memory
Step 2: INCREMENT → add 1 to it
Step 3: ASSIGN  → write new value back to memory
```

### Race Condition Scenario

```
counter = 0

Thread 1: LOAD  → reads 0
Thread 2: LOAD  → reads 0  (simultaneously!)
Thread 1: ADD 1 → gets 1
Thread 2: ADD 1 → gets 1
Thread 1: ASSIGN → writes 1
Thread 2: ASSIGN → writes 1  ← OVERWRITES Thread 1!

Expected: 2   Actual: 1  ← DATA LOSS!
```

### Real Output Example
```java
// Two threads each calling increment() 200 times
// Expected: 400
// Actual:   371  ← race condition loss
```

---

## 5. Solutions for Thread-Safe Counter

### Solution 1: Lock-Based (synchronized)

```java
class SharedResource {
    int counter = 0;

    synchronized void increment() {
        counter++;  // Only one thread enters at a time
    }
}
```

✅ Correct, ❌ Slower (thread waiting/blocking)

---

### Solution 2: Lock-Free (AtomicInteger) ← Preferred for simple counters

```java
import java.util.concurrent.atomic.AtomicInteger;

class SharedResource {
    AtomicInteger counter = new AtomicInteger(0);

    void increment() {
        counter.incrementAndGet();  // Internally uses CAS
    }

    int get() {
        return counter.get();
    }
}
```

✅ Correct, ✅ Faster (no blocking)

---

## 6. How AtomicInteger Works Internally

```java
// Simplified view of incrementAndGet() internally:
public final int incrementAndGet() {
    int expected;
    do {
        expected = this.value;           // Step 1: Read from memory
    } while (!CAS(memory, expected, expected + 1));  // Step 2 & 3: Compare & Swap
    return expected + 1;
}
```

### Two Threads Scenario with AtomicInteger

```
Initial: counter = 0

Thread 1 & Thread 2 both call incrementAndGet()

Thread 1 reads expected = 0
Thread 2 reads expected = 0

CAS is atomic → only ONE thread enters at a time:

Thread 1: CAS(memory=0, expected=0, new=1)
  → Read memory: 0
  → Compare: 0 == 0 ✅
  → Update: memory = 1  ✓ SUCCESS

Thread 2: CAS(memory=1, expected=0, new=1)
  → Read memory: 1
  → Compare: 1 == 0 ❌
  → RETRY

Thread 2 retry: reads expected = 1
  → CAS(memory=1, expected=1, new=2)
  → Compare: 1 == 1 ✅
  → Update: memory = 2  ✓ SUCCESS

Final: counter = 2  ← CORRECT!
```

---

## 7. Atomic Classes in Java

| Class | Use Case |
|-------|---------|
| `AtomicInteger` | Thread-safe integer operations |
| `AtomicLong` | Thread-safe long operations |
| `AtomicBoolean` | Thread-safe boolean flag |
| `AtomicReference<T>` | Thread-safe reference to any object |
| `AtomicStampedReference<T>` | Solves ABA problem with version stamp |

### Common Methods

```java
AtomicInteger ai = new AtomicInteger(0);

ai.get();                    // Read current value
ai.set(5);                   // Set value
ai.incrementAndGet();        // ++counter (returns new value)
ai.getAndIncrement();        // counter++ (returns old value)
ai.addAndGet(5);             // Add specific value, return new
ai.compareAndSet(0, 10);     // CAS: if value==0, set to 10
```

> 🎯 **When to use Atomic**: ONLY for **read-modify-update** simple operations. For complex logic, use locks.

---

## 8. Volatile

### The CPU Cache Problem

```
CPU Core 1              CPU Core 2
┌──────────┐            ┌──────────┐
│ Thread 1 │            │ Thread 2 │
│          │            │          │
│ L1 Cache │            │ L1 Cache │
│  x = 11  │            │  x = 10  │ ← Stale value!
└────┬─────┘            └────┬─────┘
     │                       │
     └──────────┬────────────┘
                │
         ┌──────┴──────┐
         │    Memory   │
         │   x = 10    │ ← Not yet synced!
         └─────────────┘
```

**Problem**: Thread 1 updates `x = 11` in its L1 cache but hasn't synced to main memory yet. Thread 2 reads stale `x = 10`.

### What `volatile` Does

```java
volatile int x = 10;
```

- **Reads** always go directly to **main memory** (bypass L1/L2 cache)
- **Writes** go directly to **main memory** immediately

```
With volatile:

Thread 1 writes x = 11 → directly to Memory
Thread 2 reads x       → directly from Memory → gets 11 ✅
```

### volatile vs atomic — KEY DIFFERENCE

| Feature | `volatile` | `AtomicInteger` |
|---------|-----------|-----------------|
| Visibility | ✅ Guarantees latest value visible | ✅ Also guarantees visibility |
| Thread Safety | ❌ NOT thread-safe | ✅ Thread-safe |
| Atomicity | ❌ No | ✅ Yes (via CAS) |
| Use case | Simple flag/status variable | Counter, accumulator |
| Example | `volatile boolean running = true` | `AtomicInteger counter` |

```java
// volatile is safe for THIS (single read/write, no modify)
volatile boolean isRunning = true;
// Thread 1: isRunning = false;  → safe
// Thread 2: while(isRunning){}  → safe

// volatile is NOT safe for THIS (read-modify-write)
volatile int counter = 0;
counter++;  // Still 3 non-atomic steps! Still unsafe!
```

> 💡 **Rule**: `volatile` = visibility guarantee only. `AtomicInteger` = visibility + atomicity.

---

## 9. Concurrent Collections

### Why Needed?
Standard Java collections (`ArrayList`, `HashMap`, etc.) are **not thread-safe**.

### Concurrent Alternatives

| Standard Collection | Thread-Safe Version | Internal Mechanism |
|--------------------|--------------------|--------------------|
| `PriorityQueue` | `PriorityBlockingQueue` | `ReentrantLock` |
| `LinkedList` | `ConcurrentLinkedDeque` | CAS (Lock-Free) |
| `HashMap` | `ConcurrentHashMap` | Segmented locks + CAS |
| `ArrayList` | `CopyOnWriteArrayList` | Copy-on-write |
| `HashSet` | `CopyOnWriteArraySet` | Copy-on-write |

### Internal Mechanism Examples

```java
// PriorityBlockingQueue → uses ReentrantLock (lock-based)
PriorityBlockingQueue<Integer> pbq = new PriorityBlockingQueue<>();
pbq.add(5);  // internally acquires ReentrantLock

// ConcurrentLinkedDeque → uses CAS (lock-free)
ConcurrentLinkedDeque<Integer> cld = new ConcurrentLinkedDeque<>();
cld.add(5);  // internally uses compareAndSwap
```

---

## 10. Complete Mental Model

```
                    CONCURRENCY
                         │
          ┌──────────────┴──────────────┐
          │                             │
     Lock-Based                    Lock-Free
          │                             │
   synchronized                    AtomicInteger
   ReentrantLock                   AtomicLong
   Semaphore                       AtomicBoolean
   ReadWriteLock                   AtomicReference
          │                             │
   Complex logic               Simple read-modify-update
   Any use case                Only specific use case
          │                             │
          └──────────────┬──────────────┘
                         │
                    volatile
                    (Not for concurrency)
                    (Only for visibility)
                    (Direct memory read/write)
```

---

## 11. Quick Interview Summary

| Question | Answer |
|----------|--------|
| Ways to achieve concurrency? | Lock-based & Lock-free |
| What is CAS? | CPU-level atomic operation: Read → Compare → Swap |
| Is `counter++` atomic? | ❌ No — it's 3 steps: load, increment, assign |
| What is ABA problem? | Value changes A→B→A, CAS wrongly thinks unchanged |
| Fix for ABA? | Add version/timestamp (`AtomicStampedReference`) |
| `volatile` vs `atomic`? | volatile = visibility only; atomic = visibility + thread safety |
| When to use AtomicInteger? | Only for read-modify-update patterns |
| What does volatile guarantee? | All threads read/write directly from main memory |
| Is volatile thread-safe? | ❌ No — it only ensures latest value visibility |
