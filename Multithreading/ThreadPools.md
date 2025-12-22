# Java Atomic Variables, CAS & Volatile - Complete Notes

## Table of Contents
1. [Concurrency Mechanisms](#concurrency-mechanisms)
2. [Compare and Swap (CAS)](#compare-and-swap-cas)
3. [Atomic Variables](#atomic-variables)
4. [Volatile Keyword](#volatile-keyword)
5. [Atomic vs Volatile](#atomic-vs-volatile)
6. [Concurrent Collections](#concurrent-collections)

---

## Concurrency Mechanisms

### Two Ways to Achieve Concurrency

```
┌─────────────────────────────────────┐
│   Concurrency Mechanisms            │
├─────────────────────────────────────┤
│  1. Lock-Based Mechanism            │
│     - synchronized                  │
│     - ReentrantLock                 │
│     - StampedLock                   │
│     - ReadWriteLock                 │
│     - Semaphores                    │
│                                     │
│  2. Lock-Free Mechanism ⚡          │
│     - CAS (Compare and Swap)        │
│     - Atomic Variables              │
│     - AtomicInteger                 │
│     - AtomicBoolean                 │
│     - AtomicLong                    │
│     - AtomicReference               │
└─────────────────────────────────────┘
```

### Comparison

| Feature | Lock-Based | Lock-Free |
|---------|-----------|-----------|
| **Speed** | Slower | ⚡ Faster |
| **Complexity** | High | Low |
| **Use Cases** | Complex logic, critical sections | Simple read-modify-update |
| **Overhead** | Thread blocking, context switching | Minimal |
| **Scalability** | Limited | Better |

**Important**: Lock-free is **NOT** an alternative to lock-based, but a **complement** for specific use cases.

---

## Compare and Swap (CAS)

### What is CAS?

**CAS** = **C**ompare **A**nd **S**wap

- **Low-level CPU operation**
- **Atomic** (guaranteed by hardware)
- Supported by all modern CPUs
- Foundation of lock-free programming

### CAS Parameters

```java
CAS(memory, expectedValue, newValue)
```

1. **Memory**: Location of variable
2. **Expected Value**: What we expect to be there
3. **New Value**: What we want to set

---

## CAS Algorithm

### Three Steps

```
Step 1: READ
   Read current value from memory

Step 2: COMPARE
   Compare memory value with expected value
   
Step 3: SWAP (if match)
   If they match → Update memory with new value
   If they don't match → Fail (retry)
```

### Visual Flow

```
Memory: x = 10
           ↓
CAS(M1, expected=10, new=12)
           ↓
   ┌───────────────────┐
   │  Step 1: READ     │
   │  Read M1 → 10     │
   └───────┬───────────┘
           ↓
   ┌───────────────────┐
   │  Step 2: COMPARE  │
   │  10 == 10? ✅     │
   └───────┬───────────┘
           ↓
   ┌───────────────────┐
   │  Step 3: SWAP     │
   │  M1 = 12          │
   └───────────────────┘
           ↓
     Memory: x = 12
```

---

## CAS vs Optimistic Locking

### Similarities

Both use the same principle:
1. Read current value
2. Compare with expected
3. Update if match
4. Retry if mismatch

### Optimistic Locking (Database)

```sql
-- Read with version
SELECT * FROM students WHERE id = 123;
-- Row: id=123, name="Raj", version=1

-- Update with version check
UPDATE students 
SET name = "Raj K", version = version + 1
WHERE id = 123 AND version = 1;

-- If version changed → Update fails → Retry
```

### CAS (CPU Level)

```java
AtomicInteger counter = new AtomicInteger(10);

// CAS operation
counter.compareAndSet(10, 12);
// If counter == 10 → set to 12
// If counter != 10 → fail, retry
```

### Key Differences

| Aspect | Optimistic Locking | CAS |
|--------|-------------------|-----|
| **Level** | Application/DB | CPU hardware |
| **Speed** | Slower (DB query) | ⚡ Faster (CPU instruction) |
| **Atomicity** | DB transaction | CPU guarantee |
| **Use** | Database operations | In-memory variables |

---

## ABA Problem

### What is ABA Problem?

Value changes from A → B → A, but CAS only sees A.

### Example

```
Initial: Memory = 10 (version 1)

Thread 1 reads: expected = 10

Meanwhile:
  Memory: 10 → 12 (version 2)
  Memory: 12 → 10 (version 3)

Thread 1 CAS: CAS(memory, 10, 13)
  Compare: 10 == 10 ✅ (but it's different 10!)
  Swap: Memory = 13 ❌ (Should have failed!)
```

### Solution: Add Version

```
Memory: value=10, version=1

Thread 1 reads: expected=(10, v1)

Meanwhile:
  Memory: (10, v1) → (12, v2)
  Memory: (12, v2) → (10, v3)

Thread 1 CAS: CAS(memory, (10, v1), (13, v2))
  Compare: (10, v3) != (10, v1) ❌
  Fail: Versions don't match ✅
```

---

## Atomic Variables

### What is Atomic?

**Atomic** = Single, indivisible operation (all or nothing)

### Why Needed?

#### Problem: Non-Atomic Operation

```java
class Counter {
    private int count = 0;
    
    public void increment() {
        count++;  // ❌ NOT ATOMIC!
    }
}
```

**Why not atomic?** `count++` is actually **3 operations**:

```
Step 1: READ    → temp = count
Step 2: MODIFY  → temp = temp + 1
Step 3: WRITE   → count = temp
```

#### Race Condition Example

```
Initial: count = 0

Thread 1          Thread 2
--------          --------
Read: 0           Read: 0
Add: 0+1=1        Add: 0+1=1
Write: 1          Write: 1

Result: count = 1 ❌ (Should be 2!)
```

---

## Atomic Classes

### Available Atomic Classes

```java
// Numeric
AtomicInteger
AtomicLong
AtomicBoolean

// Reference
AtomicReference<T>

// Arrays
AtomicIntegerArray
AtomicLongArray
AtomicReferenceArray<T>

// Field Updaters
AtomicIntegerFieldUpdater
AtomicLongFieldUpdater
AtomicReferenceFieldUpdater
```

---

## AtomicInteger Example

### Problem: Non-Thread-Safe Counter

```java
class SharedResource {
    private int counter = 0;
    
    public void increment() {
        counter++;  // Race condition!
    }
    
    public int get() {
        return counter;
    }
}

// Single thread
SharedResource resource = new SharedResource();
for (int i = 0; i < 400; i++) {
    resource.increment();
}
System.out.println(resource.get());  // ✅ 400

// Two threads
Thread t1 = new Thread(() -> {
    for (int i = 0; i < 200; i++) {
        resource.increment();
    }
});

Thread t2 = new Thread(() -> {
    for (int i = 0; i < 200; i++) {
        resource.increment();
    }
});

t1.start();
t2.start();
t1.join();
t2.join();

System.out.println(resource.get());  // ❌ 371 (Should be 400!)
```

---

### Solution 1: Synchronized (Lock-Based)

```java
class SharedResource {
    private int counter = 0;
    
    public synchronized void increment() {
        counter++;  // ✅ Thread-safe
    }
    
    public int get() {
        return counter;
    }
}

// Output: ✅ 400
```

**Pros**: Simple, thread-safe  
**Cons**: Slower (blocking, context switching)

---

### Solution 2: AtomicInteger (Lock-Free)

```java
class SharedResource {
    private AtomicInteger counter = new AtomicInteger(0);
    
    public void increment() {
        counter.incrementAndGet();  // ✅ Thread-safe + Fast
    }
    
    public int get() {
        return counter.get();
    }
}

// Output: ✅ 400
```

**Pros**: ⚡ Faster, no blocking  
**Cons**: Limited to simple operations

---

## How AtomicInteger Works Internally

### Internal Structure

```java
public class AtomicInteger {
    private volatile int value;  // ⚠️ Marked as volatile
    
    public AtomicInteger(int initialValue) {
        value = initialValue;
    }
}
```

### incrementAndGet() Implementation

```java
public final int incrementAndGet() {
    int expected;
    int newValue;
    
    do {
        expected = get();           // Step 1: Read current value
        newValue = expected + 1;    // Step 2: Calculate new value
    } while (!compareAndSet(expected, newValue));  // Step 3: CAS
    
    return newValue;
}

// Native method (C/C++)
public final native boolean compareAndSet(int expected, int newValue);
```

---

## CAS Retry Loop

### Visual Flow

```
Memory: counter = 0

Thread 1                    Thread 2
--------                    --------
Read: 0                     Read: 0
CAS(0→1)                    CAS(0→1)
  ↓                           ↓
  CAS atomic!               CAS fails!
  ✅ Success                ❌ Retry
  Memory: 1                   ↓
                            Read: 1
                            CAS(1→2)
                            ✅ Success
                            Memory: 2
```

### Step-by-Step Execution

```
Initial: memory = 0

Thread 1                    Thread 2
--------                    --------
expected = 0                expected = 0
newValue = 1                newValue = 1

CAS(mem, 0, 1):             CAS(mem, 0, 1):
  Read: 0                     (waiting - atomic!)
  Compare: 0 == 0 ✅
  Swap: mem = 1
  Return: true
                            Read: 1
                            Compare: 1 != 0 ❌
                            Return: false
                            
                            (Retry loop)
                            expected = 1
                            newValue = 2
                            
                            CAS(mem, 1, 2):
                              Read: 1
                              Compare: 1 == 1 ✅
                              Swap: mem = 2
                              Return: true
```

---

## AtomicInteger Methods

```java
AtomicInteger counter = new AtomicInteger(0);

// Get current value
int value = counter.get();  // 0

// Set new value
counter.set(10);

// Get and increment
int old = counter.getAndIncrement();  // Returns 10, becomes 11

// Increment and get
int current = counter.incrementAndGet();  // Returns 11

// Get and decrement
old = counter.getAndDecrement();  // Returns 11, becomes 10

// Decrement and get
current = counter.decrementAndGet();  // Returns 9

// Add and get
current = counter.addAndGet(5);  // Returns 14

// Get and add
old = counter.getAndAdd(3);  // Returns 14, becomes 17

// Compare and set
boolean success = counter.compareAndSet(17, 20);  // ✅ true

// Get and set
old = counter.getAndSet(100);  // Returns 20, becomes 100
```

---

## Volatile Keyword

### What is Volatile?

**Volatile** ensures:
1. Reads always from **main memory** (not cache)
2. Writes always to **main memory** (not cache)
3. Changes are **visible** to all threads

**NOT thread-safe** for compound operations!

---

## CPU Cache Architecture

```
┌─────────────────────────────────────────┐
│             Main Memory (RAM)            │
│            x = 10                        │
└─────────────────┬───────────────────────┘
                  │
        ┌─────────┴─────────┐
        │                   │
┌───────▼─────┐     ┌───────▼─────┐
│  L2 Cache   │     │  L2 Cache   │
│             │     │             │
└──────┬──────┘     └──────┬──────┘
       │                   │
┌──────▼──────┐     ┌──────▼──────┐
│  L1 Cache   │     │  L1 Cache   │
│  x = 11     │     │             │
└──────┬──────┘     └──────┬──────┘
       │                   │
┌──────▼──────┐     ┌──────▼──────┐
│   Core 1    │     │   Core 2    │
│  Thread 1   │     │  Thread 2   │
└─────────────┘     └─────────────┘
```

---

## Volatile Problem & Solution

### Problem: Stale Data

```java
class Example {
    private int x = 10;  // ❌ Not volatile
    
    // Thread 1 (Core 1)
    public void write() {
        x = 11;  // Writes to L1 cache only!
    }
    
    // Thread 2 (Core 2)
    public int read() {
        return x;  // Reads from memory → 10 (stale!)
    }
}
```

**Flow**:
```
Thread 1:
  x = 11 → L1 Cache (Core 1)
  
Thread 2:
  Read x
    Check L1 Cache (Core 2) → Not found
    Check L2 Cache → Not found
    Check Main Memory → 10 ❌ (Stale!)
```

---

### Solution: Volatile

```java
class Example {
    private volatile int x = 10;  // ✅ Volatile
    
    // Thread 1
    public void write() {
        x = 11;  // Writes directly to main memory
    }
    
    // Thread 2
    public int read() {
        return x;  // Reads directly from main memory → 11 ✅
    }
}
```

**Flow**:
```
Thread 1:
  x = 11 → Main Memory (bypasses cache)
  
Thread 2:
  Read x → Main Memory → 11 ✅
```

---

## Volatile Limitations

### ❌ NOT Thread-Safe for Compound Operations

```java
class Counter {
    private volatile int count = 0;
    
    public void increment() {
        count++;  // ❌ Still NOT thread-safe!
    }
}
```

**Why?** `count++` is still 3 operations:
1. Read from memory
2. Increment
3. Write to memory

**Race condition still possible!**

```
Thread 1          Thread 2
--------          --------
Read: 0           Read: 0
Add: 1            Add: 1
Write: 1          Write: 1

Result: 1 ❌ (Should be 2!)
```

---

## Atomic vs Volatile

### Complete Comparison

| Feature | Atomic | Volatile |
|---------|--------|----------|
| **Thread-Safe** | ✅ Yes | ❌ No |
| **Visibility** | ✅ Yes (via volatile internally) | ✅ Yes |
| **Operations** | Compound (read-modify-write) | Simple read/write only |
| **Use Case** | Counters, flags with operations | Simple flags, status |
| **Performance** | Fast (CAS, lock-free) | Faster (no CAS overhead) |
| **Mechanism** | CAS (Compare and Swap) | Memory barriers |
| **Atomicity** | ✅ Guaranteed | ❌ Not for compounds |

---

## When to Use What?

### Use Atomic When:
✅ Need thread-safe **read-modify-write** operations  
✅ Simple use cases (counters, flags)  
✅ Want **lock-free** performance  
✅ Example: `counter++`, `flag = !flag`

```java
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet();  // Thread-safe
```

### Use Volatile When:
✅ Need **visibility** only (not atomicity)  
✅ Simple **read/write** operations  
✅ One thread writes, others read  
✅ Example: status flags, stop signals

```java
private volatile boolean running = true;

// Thread 1
public void stop() {
    running = false;  // Visible to Thread 2
}

// Thread 2
public void run() {
    while (running) {  // Always sees latest value
        // work
    }
}
```

### Use Synchronized When:
✅ Complex operations with **multiple steps**  
✅ Need to protect **critical sections**  
✅ Multiple variables need **atomic updates**

```java
private int balance = 1000;

public synchronized void withdraw(int amount) {
    if (balance >= amount) {  // Complex logic
        balance -= amount;
    }
}
```

---

## Concurrent Collections

### Thread-Safe Collection Alternatives

| Non-Thread-Safe | Thread-Safe Alternative | Mechanism |
|-----------------|------------------------|-----------|
| `ArrayList` | `CopyOnWriteArrayList` | Lock on write |
| `LinkedList` | `ConcurrentLinkedQueue` | CAS (lock-free) |
| `HashMap` | `ConcurrentHashMap` | Segmented locks |
| `HashSet` | `ConcurrentHashMap.newKeySet()` | CAS |
| `PriorityQueue` | `PriorityBlockingQueue` | ReentrantLock |
| `ArrayDeque` | `ConcurrentLinkedDeque` | CAS |

---

## Concurrent Collection Mechanisms

### ConcurrentLinkedQueue (Lock-Free)

```java
public class ConcurrentLinkedQueue<E> {
    public boolean offer(E e) {
        // Uses CAS operation
        for (;;) {
            Node<E> t = tail;
            Node<E> s = t.next;
            
            if (t == tail) {
                if (s == null) {
                    if (casNext(t, s, newNode(e))) {  // CAS!
                        casTail(t, newNode(e));
                        return true;
                    }
                }
            }
        }
    }
}
```

**Mechanism**: CAS (lock-free, fast)

---

### PriorityBlockingQueue (Lock-Based)

```java
public class PriorityBlockingQueue<E> {
    private final ReentrantLock lock;
    
    public boolean offer(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();  // Lock!
        try {
            // Insert logic
            return true;
        } finally {
            lock.unlock();
        }
    }
}
```

**Mechanism**: ReentrantLock (lock-based)

---

## Practical Examples

### Example 1: Counter (Atomic)

```java
import java.util.concurrent.atomic.AtomicInteger;

class ThreadSafeCounter {
    private AtomicInteger count = new AtomicInteger(0);
    
    public void increment() {
        count.incrementAndGet();
    }
    
    public int get() {
        return count.get();
    }
}

// Usage
ThreadSafeCounter counter = new ThreadSafeCounter();

Thread t1 = new Thread(() -> {
    for (int i = 0; i < 1000; i++) {
        counter.increment();
    }
});

Thread t2 = new Thread(() -> {
    for (int i = 0; i < 1000; i++) {
        counter.increment();
    }
});

t1.start();
t2.start();
t1.join();
t2.join();

System.out.println(counter.get());  // ✅ 2000
```

---

### Example 2: Stop Flag (Volatile)

```java
class Worker implements Runnable {
    private volatile boolean running = true;
    
    public void run() {
        while (running) {
            // Do work
            System.out.println("Working...");
        }
        System.out.println("Stopped!");
    }
    
    public void stop() {
        running = false;  // Visible to run()
    }
}

// Usage
Worker worker = new Worker();
Thread t = new Thread(worker);
t.start();

Thread.sleep(1000);
worker.stop();  // Thread sees change immediately
```

---

### Example 3: Complex Operation (Synchronized)

```java
class BankAccount {
    private int balance = 1000;
    
    public synchronized boolean transfer(BankAccount target, int amount) {
        if (this.balance >= amount) {
            this.balance -= amount;
            target.deposit(amount);
            return true;
        }
        return false;
    }
    
    public synchronized void deposit(int amount) {
        this.balance += amount;
    }
    
    public synchronized int getBalance() {
        return balance;
    }
}
```

---

## Key Takeaways

### CAS (Compare and Swap)
✅ CPU-level atomic operation  
✅ Read → Compare → Swap (if match)  
✅ Foundation of lock-free programming  
✅ Retry loop on failure  

### Atomic Variables
✅ Thread-safe without locks  
✅ Uses CAS internally  
✅ Fast for simple operations  
✅ Limited to read-modify-write patterns  

### Volatile
✅ Ensures visibility across threads  
✅ Reads/writes go to main memory  
✅ **NOT** thread-safe for compounds  
✅ Use for simple flags only  

### Atomic vs Volatile
✅ **Atomic** = Thread-safe + Visibility  
✅ **Volatile** = Visibility only  
✅ Use Atomic for operations, Volatile for flags  

### When to Use
✅ **Lock-Free** (Atomic): Simple, high-performance  
✅ **Lock-Based** (Synchronized): Complex, critical sections  
✅ Choose based on use case complexity  

---

*Happy Coding! 🚀*
