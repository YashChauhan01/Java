# Java Multithreading - Advanced Locks and Synchronization

## Table of Contents
1. [Introduction to Custom Locks](#introduction-to-custom-locks)
2. [ReentrantLock](#reentrantlock)
3. [Shared vs Exclusive Locks](#shared-vs-exclusive-locks)
4. [ReadWriteLock](#readwritelock)
5. [Optimistic vs Pessimistic Locking](#optimistic-vs-pessimistic-locking)
6. [StampedLock](#stampedlock)
7. [Semaphore](#semaphore)
8. [Condition Interface (Inter-thread Communication)](#condition-interface-inter-thread-communication)
9. [Complete Working Examples](#complete-working-examples)
10. [Summary and Best Practices](#summary-and-best-practices)

---

## Introduction to Custom Locks

### The Problem with synchronized Keyword

**Limitation of synchronized:**
- Works based on **monitor locks** which are **object-dependent**
- Only works when all threads use the **same object**
- Cannot control lock behavior across different objects

### Example: synchronized Limitation

```java
class SharedResource {
    public synchronized void producer() {
        System.out.println("Lock acquired by: " + Thread.currentThread().getName());
        try {
            Thread.sleep(4000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Lock released by: " + Thread.currentThread().getName());
    }
}

public class SynchronizedProblem {
    public static void main(String[] args) {
        // Two DIFFERENT objects
        SharedResource resource1 = new SharedResource();
        SharedResource resource2 = new SharedResource();
        
        // Thread-1 uses resource1
        Thread t1 = new Thread(() -> resource1.producer(), "Thread-0");
        
        // Thread-2 uses resource2 (different object)
        Thread t2 = new Thread(() -> resource2.producer(), "Thread-1");
        
        t1.start();
        t2.start();
    }
}
```

**Output:**
```
Lock acquired by: Thread-0
Lock acquired by: Thread-1   ← Both threads enter! (Different objects)
Lock released by: Thread-0
Lock released by: Thread-1
```

**Problem:** Both threads acquire locks because they're using different objects. Monitor lock is per-object.

### The Solution: Custom Locks

Java provides **four types of custom locks** that don't depend on objects:

1. **ReentrantLock** - Basic locking mechanism
2. **ReadWriteLock** - Separate read and write locks
3. **StampedLock** - Optimistic locking with read/write capabilities
4. **Semaphore** - Controls number of threads accessing resource

**Key Advantage:** These locks are **independent of objects** - based on lock instances.

---

## ReentrantLock

### What is ReentrantLock?

A `ReentrantLock` is a **lock object** that provides explicit locking mechanism, independent of object references.

**Package:** `java.util.concurrent.locks.ReentrantLock`

### Basic Usage

```java
import java.util.concurrent.locks.ReentrantLock;

class SharedResource {
    public void producer(ReentrantLock lock) {
        try {
            // Acquire lock
            lock.lock();
            
            System.out.println("Lock acquired by: " + 
                              Thread.currentThread().getName());
            
            // Critical section
            Thread.sleep(4000);
            
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            // ALWAYS release lock in finally block
            System.out.println("Lock released by: " + 
                              Thread.currentThread().getName());
            lock.unlock();
        }
    }
}
```

**Important:** Always use `try-finally` to ensure lock is released even if exception occurs.

### Complete Example: Different Objects, Same Lock

```java
import java.util.concurrent.locks.ReentrantLock;

class SharedResource {
    public void producer(ReentrantLock lock) {
        try {
            lock.lock();
            System.out.println("Lock acquired by: " + 
                              Thread.currentThread().getName());
            Thread.sleep(4000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println("Lock released by: " + 
                              Thread.currentThread().getName());
            lock.unlock();
        }
    }
}

public class ReentrantLockExample {
    public static void main(String[] args) {
        // Create ONE lock
        ReentrantLock lock = new ReentrantLock();
        
        // Create TWO different resource objects
        SharedResource resource1 = new SharedResource();
        SharedResource resource2 = new SharedResource();
        
        // Both threads use SAME lock (but different objects)
        Thread t1 = new Thread(() -> resource1.producer(lock), "Thread-0");
        Thread t2 = new Thread(() -> resource2.producer(lock), "Thread-1");
        
        t1.start();
        t2.start();
    }
}
```

**Output:**
```
Lock acquired by: Thread-0
[4 seconds pass]
Lock released by: Thread-0
Lock acquired by: Thread-1    ← Waits for Thread-0 to release
[4 seconds pass]
Lock released by: Thread-1
```

**Key Point:** Even though using different objects (resource1, resource2), both threads share the SAME lock, so only one can proceed at a time.

### Why "Reentrant"?

A thread can acquire the same lock multiple times (re-enter):

```java
ReentrantLock lock = new ReentrantLock();

lock.lock();  // Acquire lock (count = 1)
lock.lock();  // Acquire again (count = 2) - allowed!
lock.unlock(); // Release (count = 1)
lock.unlock(); // Release (count = 0) - fully released
```

### ReentrantLock Methods

| Method | Description |
|--------|-------------|
| `lock()` | Acquires the lock, waits if unavailable |
| `unlock()` | Releases the lock |
| `tryLock()` | Tries to acquire lock, returns immediately |
| `tryLock(time, unit)` | Tries to acquire lock within timeout |
| `isLocked()` | Checks if lock is held by any thread |
| `getHoldCount()` | Returns number of times current thread holds lock |

### Advanced Usage

```java
ReentrantLock lock = new ReentrantLock();

// Try to acquire lock without blocking
if (lock.tryLock()) {
    try {
        // Critical section
    } finally {
        lock.unlock();
    }
} else {
    System.out.println("Could not acquire lock");
}

// Try with timeout
try {
    if (lock.tryLock(2, TimeUnit.SECONDS)) {
        try {
            // Critical section
        } finally {
            lock.unlock();
        }
    }
} catch (InterruptedException e) {
    e.printStackTrace();
}
```

---

## Shared vs Exclusive Locks

### Understanding Lock Types

Before understanding ReadWriteLock, you must understand these concepts:

#### Shared Lock (Read Lock)
- **Multiple threads** can acquire this lock simultaneously
- **Read-only** access - cannot modify data
- Also called "S-lock"

#### Exclusive Lock (Write Lock)
- **Only ONE thread** can acquire this lock
- **Read and write** access - can modify data
- Also called "X-lock"

### Lock Compatibility Matrix

```
┌──────────────────┬─────────────┬────────────────┐
│                  │ Shared Lock │ Exclusive Lock │
├──────────────────┼─────────────┼────────────────┤
│ Shared Lock      │     ✓ YES   │     ✗ NO       │
│ Exclusive Lock   │     ✗ NO    │     ✗ NO       │
└──────────────────┴─────────────┴────────────────┘
```

### Visual Representation

```
Thread-1 has SHARED Lock (Reading)
    ↓
Thread-2 wants SHARED Lock → ✓ ALLOWED (can also read)
Thread-3 wants EXCLUSIVE Lock → ✗ BLOCKED (must wait)

─────────────────────────────────────────────────

Thread-1 has EXCLUSIVE Lock (Writing)
    ↓
Thread-2 wants SHARED Lock → ✗ BLOCKED (cannot read while writing)
Thread-3 wants EXCLUSIVE Lock → ✗ BLOCKED (cannot write while writing)
```

### Rules Summary

1. **Shared + Shared = ✓ Allowed**
   - Multiple threads can read simultaneously
   
2. **Shared + Exclusive = ✗ Not Allowed**
   - Cannot write while someone is reading
   
3. **Exclusive + Shared = ✗ Not Allowed**
   - Cannot read while someone is writing
   
4. **Exclusive + Exclusive = ✗ Not Allowed**
   - Only one writer at a time

---

## ReadWriteLock

### What is ReadWriteLock?

`ReadWriteLock` maintains a pair of locks:
- **Read Lock (Shared)**: For read-only operations
- **Write Lock (Exclusive)**: For write operations

**Package:** `java.util.concurrent.locks.ReadWriteLock`

### When to Use ReadWriteLock?

**Use when:**
- **Reads are frequent** (1000 reads)
- **Writes are rare** (10 writes)

**Benefit:** Multiple threads can read simultaneously, improving performance.

### Creating ReadWriteLock

```java
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

// ReadWriteLock is an interface
ReadWriteLock rwLock = new ReentrantReadWriteLock();
```

### Complete Example

```java
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

class SharedResource {
    private int data = 10;
    
    // Producer - READ operation
    public void producer(ReadWriteLock lock) {
        try {
            // Acquire READ lock (shared)
            lock.readLock().lock();
            
            System.out.println("Read lock acquired by: " + 
                              Thread.currentThread().getName());
            System.out.println("Reading data: " + data);
            
            Thread.sleep(8000);  // Simulate reading
            
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println("Read lock released by: " + 
                              Thread.currentThread().getName());
            lock.readLock().unlock();
        }
    }
    
    // Consumer - WRITE operation
    public void consumer(ReadWriteLock lock) {
        try {
            // Acquire WRITE lock (exclusive)
            lock.writeLock().lock();
            
            System.out.println("Write lock acquired by: " + 
                              Thread.currentThread().getName());
            data++;
            System.out.println("Updated data to: " + data);
            
            Thread.sleep(4000);  // Simulate writing
            
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println("Write lock released by: " + 
                              Thread.currentThread().getName());
            lock.writeLock().unlock();
        }
    }
}

public class ReadWriteLockExample {
    public static void main(String[] args) {
        ReadWriteLock rwLock = new ReentrantReadWriteLock();
        SharedResource resource = new SharedResource();
        
        // Thread-1: READ (acquires shared lock)
        Thread t1 = new Thread(() -> resource.producer(rwLock), "Reader-1");
        
        // Thread-2: READ (acquires shared lock - allowed!)
        Thread t2 = new Thread(() -> resource.producer(rwLock), "Reader-2");
        
        // Thread-3: WRITE (wants exclusive lock - must wait)
        Thread t3 = new Thread(() -> resource.consumer(rwLock), "Writer-1");
        
        t1.start();
        t2.start();
        t3.start();
    }
}
```

**Output:**
```
Read lock acquired by: Reader-1
Reading data: 10
Read lock acquired by: Reader-2  ← Both readers proceed simultaneously!
Reading data: 10
[8 seconds pass]
Read lock released by: Reader-1
Read lock released by: Reader-2
Write lock acquired by: Writer-1  ← Writer waits for all readers to finish
Updated data to: 11
[4 seconds pass]
Write lock released by: Writer-1
```

### Execution Timeline

```
Time 0s:  Reader-1 acquires READ lock
Time 0s:  Reader-2 acquires READ lock (allowed - multiple readers)
Time 0s:  Writer-1 tries to acquire WRITE lock → BLOCKED
          (must wait for all readers to release)

Time 8s:  Reader-1 releases READ lock
Time 8s:  Reader-2 releases READ lock
Time 8s:  Writer-1 acquires WRITE lock (no readers now)

Time 12s: Writer-1 releases WRITE lock
```

### Important Notes

**✓ DO:**
- Use read lock for read-only operations
- Use write lock for modifications
- Useful when reads >> writes

**✗ DON'T:**
- Don't modify data with read lock
- Don't mix synchronized with ReadWriteLock

---

## Optimistic vs Pessimistic Locking

### Pessimistic Locking

**Definition:** Assume conflicts WILL happen, so lock resources preemptively.

**Examples:** synchronized, ReentrantLock, ReadWriteLock

**Mechanism:**
1. Acquire lock BEFORE accessing resource
2. Perform operation
3. Release lock

```java
// Pessimistic approach
lock.lock();
try {
    // Critical section
} finally {
    lock.unlock();
}
```

### Optimistic Locking

**Definition:** Assume conflicts are RARE, so don't lock. Verify before updating.

**Mechanism:**
1. Read data WITHOUT lock
2. Remember version/state
3. Perform operation
4. Before updating, check if version changed
5. If changed → Rollback and retry
6. If unchanged → Update

### Database Example of Optimistic Locking

```
Database Table: Users
┌────┬──────┬─────────┬─────────┐
│ ID │ Name │  Type   │ Version │
├────┼──────┼─────────┼─────────┤
│123 │ Ram  │ Student │    1    │  ← Initial state
│456 │ Shyam│ Student │    1    │
└────┴──────┴─────────┴─────────┘
```

#### Scenario: Two Threads Update Same Row

```
Thread-1 and Thread-2 both want to update row ID=123

Time T1: Both threads READ
─────────────────────────────────────────
Thread-1 reads: ID=123, Type=Student, Version=1
Thread-2 reads: ID=123, Type=Student, Version=1

Time T2: Both perform operations locally
─────────────────────────────────────────
Thread-1: Changes Type to "Teacher"
Thread-2: Changes Type to "ExStudent"

Time T3: Thread-2 updates FIRST
─────────────────────────────────────────
UPDATE Users 
SET Type = 'ExStudent', Version = Version + 1
WHERE ID = 123 AND Version = 1;

✓ Success! Row updated:
┌────┬──────┬────────────┬─────────┐
│123 │ Ram  │ ExStudent  │    2    │  ← Version incremented!
└────┴──────┴────────────┴─────────┘

Time T4: Thread-1 tries to update
─────────────────────────────────────────
UPDATE Users 
SET Type = 'Teacher', Version = Version + 1
WHERE ID = 123 AND Version = 1;  ← Looking for Version=1

✗ FAIL! Version is now 2, not 1
Thread-1 must:
1. Rollback changes
2. Re-read row (now Version=2, Type=ExStudent)
3. Re-apply logic
4. Try update again with Version=2
```

### Visual Flow

```
┌─────────────────────────────────────────────────────┐
│         PESSIMISTIC LOCKING                         │
├─────────────────────────────────────────────────────┤
│  Thread-1                  Thread-2                 │
│     │                         │                     │
│     ├─→ LOCK                  ├─→ LOCK              │
│     │   (acquired)            │   (BLOCKED)         │
│     ├─→ Read                  │   (waiting...)      │
│     ├─→ Modify                │   (waiting...)      │
│     ├─→ Write                 │   (waiting...)      │
│     ├─→ UNLOCK                │                     │
│     │                         ├─→ (now acquires)    │
│     │                         ├─→ Read              │
│     │                         ├─→ Modify            │
│     │                         ├─→ Write             │
│     │                         ├─→ UNLOCK            │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│         OPTIMISTIC LOCKING                          │
├─────────────────────────────────────────────────────┤
│  Thread-1                  Thread-2                 │
│     │                         │                     │
│     ├─→ Read (v=1)            ├─→ Read (v=1)        │
│     │   (no lock!)            │   (no lock!)        │
│     ├─→ Modify locally        ├─→ Modify locally    │
│     │                         │                     │
│     │                         ├─→ Update (v=1→2) ✓  │
│     │                         │                     │
│     ├─→ Update (v=1→2) ✗      │                     │
│     │   (version mismatch!)   │                     │
│     ├─→ Rollback              │                     │
│     ├─→ Re-read (v=2)         │                     │
│     ├─→ Re-modify             │                     │
│     ├─→ Update (v=2→3) ✓      │                     │
└─────────────────────────────────────────────────────┘
```

### Comparison

| Feature | Pessimistic | Optimistic |
|---------|-------------|------------|
| **Locking** | Yes, before access | No locking |
| **Conflicts** | Prevented | Detected and resolved |
| **Performance** | Lower (due to locking) | Higher (no locks) |
| **Best for** | High contention | Low contention |
| **Retry** | Not needed | May need retry |

---

## StampedLock

### What is StampedLock?

`StampedLock` provides:
1. **ReadWriteLock functionality** (like ReadWriteLock)
2. **Optimistic read** capability (unique feature)

**Package:** `java.util.concurrent.locks.StampedLock`

**Introduced:** Java 8

### Key Concept: Stamp

A `StampedLock` returns a **stamp** (long value) when acquiring locks:
- Stamp represents the **state/version** of the lock
- Used to validate if lock state changed
- Essential for optimistic locking

### Three Locking Modes

1. **Writing (Exclusive Lock)**
   ```java
   long stamp = lock.writeLock();
   // Critical section
   lock.unlockWrite(stamp);
   ```

2. **Reading (Shared Lock)**
   ```java
   long stamp = lock.readLock();
   // Read operation
   lock.unlockRead(stamp);
   ```

3. **Optimistic Reading (No Lock)**
   ```java
   long stamp = lock.tryOptimisticRead();
   // Read data
   if (!lock.validate(stamp)) {
       // Someone wrote, retry
   }
   ```

### Example 1: ReadWrite Functionality

```java
import java.util.concurrent.locks.StampedLock;

class SharedResource {
    private StampedLock lock = new StampedLock();
    private int data = 10;
    
    // READ with StampedLock
    public void read() {
        long stamp = lock.readLock();  // Returns stamp
        try {
            System.out.println("Read lock acquired by: " + 
                              Thread.currentThread().getName());
            System.out.println("Data: " + data);
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println("Read lock released by: " + 
                              Thread.currentThread().getName());
            lock.unlockRead(stamp);  // Must pass stamp!
        }
    }
    
    // WRITE with StampedLock
    public void write() {
        long stamp = lock.writeLock();  // Returns stamp
        try {
            System.out.println("Write lock acquired by: " + 
                              Thread.currentThread().getName());
            data++;
            System.out.println("Updated data to: " + data);
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println("Write lock released by: " + 
                              Thread.currentThread().getName());
            lock.unlockWrite(stamp);  // Must pass stamp!
        }
    }
}
```

**Why pass stamp during unlock?**
- Stamp tracks lock state
- Validates you're unlocking the correct acquisition
- Important for optimistic reads

### Example 2: Optimistic Read

```java
import java.util.concurrent.locks.StampedLock;

class SharedResource {
    private StampedLock lock = new StampedLock();
    private int data = 10;
    
    // OPTIMISTIC READ
    public void optimisticRead() {
        // NO LOCK acquired! Just get current stamp
        long stamp = lock.tryOptimisticRead();
        
        System.out.println("Optimistic read started by: " + 
                          Thread.currentThread().getName());
        
        // Read data (no lock held)
        int localData = data;
        
        try {
            // Simulate some work
            System.out.println("Performing operation on: " + localData);
            Thread.sleep(6000);
            
            // Try to update
            localData = 11;
            
            // VALIDATE: Did anyone write while we were working?
            if (lock.validate(stamp)) {
                // No writes happened, safe to update
                data = localData;
                System.out.println("✓ Optimistic read SUCCESS - Updated to: " + data);
            } else {
                // Someone wrote! Must rollback
                System.out.println("✗ Optimistic read FAILED - Rolling back");
                // In real scenario: re-read and retry
            }
            
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    
    // WRITE (will invalidate optimistic reads)
    public void write() {
        long stamp = lock.writeLock();
        try {
            System.out.println("Write lock acquired by: " + 
                              Thread.currentThread().getName());
            data = 20;
            System.out.println("Data written: " + data);
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println("Write lock released");
            lock.unlockWrite(stamp);
            // Stamp changed! Optimistic reads will fail validation
        }
    }
}

public class StampedLockOptimistic {
    public static void main(String[] args) {
        SharedResource resource = new SharedResource();
        
        // Thread-1: Optimistic read
        Thread t1 = new Thread(() -> resource.optimisticRead(), "Reader");
        
        // Thread-2: Write (will invalidate Thread-1's optimistic read)
        Thread t2 = new Thread(() -> {
            try {
                Thread.sleep(2000);  // Start after optimistic read
                resource.write();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "Writer");
        
        t1.start();
        t2.start();
    }
}
```

**Output Scenario 1 (Write happens during optimistic read):**
```
Optimistic read started by: Reader
Performing operation on: 10
Write lock acquired by: Writer
Data written: 20
Write lock released
✗ Optimistic read FAILED - Rolling back
```

**Output Scenario 2 (No write during optimistic read):**
```
Optimistic read started by: Reader
Performing operation on: 10
✓ Optimistic read SUCCESS - Updated to: 11
```

### How Optimistic Locking Works in StampedLock

```
Step 1: tryOptimisticRead()
┌────────────────────────────┐
│ Stamp = 100                │  ← Current stamp value
│ No lock acquired           │
└────────────────────────────┘

Step 2: Read data and perform work
┌────────────────────────────┐
│ Local copy = 10            │
│ Perform calculations       │
│ New value = 11             │
└────────────────────────────┘

Step 3: Another thread writes
┌────────────────────────────┐
│ writeLock() called         │
│ Stamp changes: 100 → 101   │  ← Stamp incremented!
│ unlockWrite() called       │
└────────────────────────────┘

Step 4: validate(stamp)
┌────────────────────────────┐
│ Current stamp = 101        │
│ My stamp = 100             │
│ Mismatch! Return false     │  ← Validation fails
└────────────────────────────┘

Step 5: Handle validation failure
┌────────────────────────────┐
│ Rollback changes           │
│ Re-read data               │
│ Retry operation            │
└────────────────────────────┘
```

### When to Use StampedLock

**Use for:**
- Very high read throughput needed
- Reads are much more frequent than writes
- Can handle retry logic for failed optimistic reads

**Don't use for:**
- Simple scenarios (use synchronized or ReentrantLock)
- Equal read/write operations
- When retry logic is complex

---

## Semaphore

### What is Semaphore?

A `Semaphore` controls access to a resource by **limiting the number of threads** that can access it simultaneously.

**Package:** `java.util.concurrent.Semaphore`

**Key Feature:** Allows **N threads** to acquire lock at the same time (not just 1)

### Creating Semaphore

```java
import java.util.concurrent.Semaphore;

// Allow 2 threads to access simultaneously
Semaphore semaphore = new Semaphore(2);  // permits = 2
```

### How It Works

```
Semaphore with 2 permits:

Initial state: [Permit-1: Available] [Permit-2: Available]

Thread-1 arrives → acquire() → [Permit-1: Thread-1] [Permit-2: Available]
Thread-2 arrives → acquire() → [Permit-1: Thread-1] [Permit-2: Thread-2]
Thread-3 arrives → acquire() → BLOCKED (no permits available)
Thread-4 arrives → acquire() → BLOCKED (no permits available)

Thread-1 finishes → release() → [Permit-1: Available] [Permit-2: Thread-2]
Thread-3 acquires → [Permit-1: Thread-3] [Permit-2: Thread-2]
```

### Complete Example

```java
import java.util.concurrent.Semaphore;

class SharedResource {
    private Semaphore semaphore;
    
    public SharedResource(int permits) {
        this.semaphore = new Semaphore(permits);
    }
    
    public void accessResource() {
        try {
            // Try to acquire permit
            System.out.println(Thread.currentThread().getName() + 
                              " - Waiting for permit...");
            semaphore.acquire();
            
            // Got permit!
            System.out.println(Thread.currentThread().getName() + 
                              " - ✓ Permit acquired!");
            
            // Use resource
            Thread.sleep(4000);
            
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            // Release permit
            System.out.println(Thread.currentThread().getName() + 
                              " - Permit released");
            semaphore.release();
        }
    }
}

public class SemaphoreExample {
    public static void main(String[] args) {
        // Allow 2 threads simultaneously
        SharedResource resource = new SharedResource(2);
        
        // Create 4 threads
        Thread t1 = new Thread(() -> resource.accessResource(), "Thread-1");
        Thread t2 = new Thread(() -> resource.accessResource(), "Thread-2");
        Thread t3 = new Thread(() -> resource.accessResource(), "Thread-3");
        Thread t4 = new Thread(() -> resource.accessResource(), "Thread-4");
        
        t1.start();
        t2.start();
        t3.start();
        t4.start();
    }
}
```

**Output:**
```
Thread-1 - Waiting for permit...
Thread-1 - ✓ Permit acquired!
Thread-2 - Waiting for permit...
Thread-2 - ✓ Permit acquired!
Thread-3 - Waiting for permit...      ← Blocked (no permits)
Thread-4 - Waiting for permit...      ← Blocked (no permits)
[4 seconds pass]
Thread-1 - Permit released
Thread-3 - ✓ Permit acquired!         ← Gets Thread-1's permit
[4 seconds pass]
Thread-2 - Permit released
Thread-4 - ✓ Permit acquired!         ← Gets Thread-2's permit
[4 seconds pass]
Thread-3 - Permit released
Thread-4 - Permit released
```

### Timeline Visualization

```
Time 0s:   Thread-1 acquires (permits: 1/2 used)
Time 0s:   Thread-2 acquires (permits: 2/2 used)
Time 0s:   Thread-3 WAITS (no permits available)
Time 0s:   Thread-4 WAITS (no permits available)

Time 4s:   Thread-1 releases (permits: 1/2 used)
Time 4s:   Thread-3 acquires (permits: 2/2 used)

Time 8s:   Thread-2 releases (permits: 1/2 used)
Time 8s:   Thread-4 acquires (permits: 2/2 used)

Time 12s:  Thread-3 releases (permits: 1/2 used)
Time 12s:  Thread-4 releases (permits: 0/2 used)
```

### Real-World Use Cases

#### Use Case 2: Printer Queue

```java
class PrinterManager {
    private Semaphore printers = new Semaphore(2);  // 2 printers
    
    public void print(String document) {
        try {
            System.out.println(Thread.currentThread().getName() + 
                              " - Waiting for printer...");
            printers.acquire();
            
            System.out.println(Thread.currentThread().getName() + 
                              " - Printing: " + document);
            Thread.sleep(3000);  // Printing takes 3 seconds
            
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println(Thread.currentThread().getName() + 
                              " - Print complete");
            printers.release();
        }
    }
}

// Usage: 5 users, but only 2 printers available
```

#### Use Case 3: Rate Limiting

```java
class APIRateLimiter {
    private Semaphore rateLimiter = new Semaphore(10);  // 10 requests/sec
    
    public void makeAPICall() {
        try {
            rateLimiter.acquire();
            // Make API call
            callExternalAPI();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            // Release after 1 second
            new Timer().schedule(new TimerTask() {
                public void run() {
                    rateLimiter.release();
                }
            }, 1000);
        }
    }
}
```

### Semaphore Methods

| Method | Description |
|--------|-------------|
| `acquire()` | Acquires a permit, blocks if unavailable |
| `acquire(n)` | Acquires n permits |
| `release()` | Releases a permit |
| `release(n)` | Releases n permits |
| `tryAcquire()` | Tries to acquire, returns immediately |
| `tryAcquire(timeout, unit)` | Tries with timeout |
| `availablePermits()` | Returns available permits |

### Example with tryAcquire()

```java
Semaphore semaphore = new Semaphore(2);

if (semaphore.tryAcquire()) {
    try {
        // Got permit
        processTask();
    } finally {
        semaphore.release();
    }
} else {
    System.out.println("No permit available, skipping task");
}

// With timeout
if (semaphore.tryAcquire(2, TimeUnit.SECONDS)) {
    try {
        // Got permit within 2 seconds
    } finally {
        semaphore.release();
    }
}
```

---

## Condition Interface (Inter-thread Communication)

### The Problem: wait/notify Don't Work with Custom Locks

**With synchronized:**
```java
synchronized(obj) {
    obj.wait();      // Works fine
    obj.notify();    // Works fine
}
```

**With custom locks:**
```java
lock.lock();
try {
    obj.wait();      // ERROR! IllegalMonitorStateException
    obj.notify();    // ERROR! IllegalMonitorStateException
} finally {
    lock.unlock();
}
```

**Why?** `wait()` and `notify()` only work with **monitor locks** (synchronized), not custom locks.

### The Solution: Condition Interface

The `Condition` interface provides:
- `await()` → equivalent to `wait()`
- `signal()` → equivalent to `notify()`
- `signalAll()` → equivalent to `notifyAll()`

**Package:** `java.util.concurrent.locks.Condition`

### Creating Condition

```java
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.Condition;

ReentrantLock lock = new ReentrantLock();
Condition condition = lock.newCondition();  // Create condition from lock
```

**Important:** Condition is bound to a specific lock!

### Comparison Table

| synchronized | Custom Locks (Condition) |
|--------------|--------------------------|
| `wait()` | `condition.await()` |
| `notify()` | `condition.signal()` |
| `notifyAll()` | `condition.signalAll()` |

### Complete Producer-Consumer Example

```java
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.Condition;

class SharedResource {
    private ReentrantLock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();
    private boolean isAvailable = false;
    
    // PRODUCER
    public void produce() {
        lock.lock();  // Acquire lock
        try {
            // Wait while item is already available
            while (isAvailable) {
                System.out.println(Thread.currentThread().getName() + 
                                  " - Item already available, producer waiting...");
                condition.await();  // Release lock and wait
            }
            
            // Produce item
            isAvailable = true;
            System.out.println(Thread.currentThread().getName() + 
                              " - ✓ Item produced");
            
            // Notify consumers
            condition.signalAll();  // Wake up waiting consumers
            
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();  // Always release lock
        }
    }
    
    // CONSUMER
    public void consume() {
        lock.lock();  // Acquire lock
        try {
            // Wait while no item available
            while (!isAvailable) {
                System.out.println(Thread.currentThread().getName() + 
                                  " - No item available, consumer waiting...");
                condition.await();  // Release lock and wait
            }
            
            // Consume item
            isAvailable = false;
            System.out.println(Thread.currentThread().getName() + 
                              " - ✓ Item consumed");
            
            // Notify producers
            condition.signalAll();  // Wake up waiting producers
            
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();  // Always release lock
        }
    }
}

public class ConditionExample {
    public static void main(String[] args) {
        SharedResource resource = new SharedResource();
        
        // Producer thread
        Thread producer = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                resource.produce();
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "Producer");
        
        // Consumer thread
        Thread consumer = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                resource.consume();
                try {
                    Thread.sleep(1500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "Consumer");
        
        consumer.start();  // Start consumer first
        producer.start();
    }
}
```

**Output:**
```
Consumer - No item available, consumer waiting...
Producer - ✓ Item produced
Consumer - ✓ Item consumed
Producer - ✓ Item produced
Consumer - ✓ Item consumed
Producer - ✓ Item produced
Consumer - ✓ Item consumed
Producer - ✓ Item produced
Consumer - ✓ Item consumed
Producer - ✓ Item produced
Consumer - ✓ Item consumed
```

### Execution Flow

```
Time 0s:  Consumer starts
          → lock.lock() (acquires lock)
          → isAvailable = false
          → condition.await() (releases lock, waits)

Time 100ms: Producer starts
           → lock.lock() (acquires lock - consumer released it)
           → isAvailable = false (can produce)
           → isAvailable = true
           → condition.signalAll() (wakes consumer)
           → lock.unlock()

Time 101ms: Consumer wakes up
           → Re-acquires lock
           → isAvailable = true (can consume)
           → isAvailable = false
           → condition.signalAll() (wakes producer if waiting)
           → lock.unlock()
```

### Multiple Conditions

You can create multiple conditions from one lock:

```java
ReentrantLock lock = new ReentrantLock();
Condition notFull = lock.newCondition();   // For producers
Condition notEmpty = lock.newCondition();  // For consumers

// Producer waits on notFull
public void produce() {
    lock.lock();
    try {
        while (isFull()) {
            notFull.await();  // Wait for space
        }
        addItem();
        notEmpty.signal();  // Signal consumers
    } finally {
        lock.unlock();
    }
}

// Consumer waits on notEmpty
public void consume() {
    lock.lock();
    try {
        while (isEmpty()) {
            notEmpty.await();  // Wait for items
        }
        removeItem();
        notFull.signal();  // Signal producers
    } finally {
        lock.unlock();
    }
}
```

### Important Rules

1. **Must hold lock before calling await/signal**
   ```java
   condition.await();  // ERROR! Must acquire lock first
   
   lock.lock();
   condition.await();  // Correct
   lock.unlock();
   ```

2. **Always use while loop with await**
   ```java
   // BAD
   if (condition) {
       condition.await();
   }
   
   // GOOD
   while (condition) {
       condition.await();  // Re-checks after waking
   }
   ```

3. **Use try-finally for lock**
   ```java
   lock.lock();
   try {
       // Critical section
   } finally {
       lock.unlock();  // Always releases
   }
   ```

---

## Complete Working Examples

### Example 1: Bank Account with ReentrantLock

```java
import java.util.concurrent.locks.ReentrantLock;

class BankAccount {
    private int balance = 1000;
    private ReentrantLock lock = new ReentrantLock();
    
    public void withdraw(String customer, int amount) {
        lock.lock();
        try {
            System.out.println(customer + " - Requesting withdrawal: $" + amount);
            
            if (balance >= amount) {
                System.out.println(customer + " - Processing withdrawal...");
                Thread.sleep(1000);
                balance -= amount;
                System.out.println(customer + " - ✓ Withdrawn $" + amount + 
                                  " | Balance: $" + balance);
            } else {
                System.out.println(customer + " - ✗ Insufficient funds");
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
    
    public void deposit(String customer, int amount) {
        lock.lock();
        try {
            System.out.println(customer + " - Depositing: $" + amount);
            Thread.sleep(500);
            balance += amount;
            System.out.println(customer + " - ✓ Deposited $" + amount + 
                              " | Balance: $" + balance);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}

public class BankExample {
    public static void main(String[] args) {
        BankAccount account = new BankAccount();
        
        Thread t1 = new Thread(() -> account.withdraw("Alice", 500), "Thread-1");
        Thread t2 = new Thread(() -> account.withdraw("Bob", 400), "Thread-2");
        Thread t3 = new Thread(() -> account.deposit("Charlie", 200), "Thread-3");
        
        t1.start();
        t2.start();
        t3.start();
    }
}
```

### Example 2: Cache with ReadWriteLock

```java
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

class Cache {
    private Map<String, String> data = new HashMap<>();
    private ReadWriteLock rwLock = new ReentrantReadWriteLock();
    
    // READ operation (shared lock)
    public String get(String key) {
        rwLock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + 
                              " - Reading: " + key);
            Thread.sleep(1000);
            String value = data.get(key);
            System.out.println(Thread.currentThread().getName() + 
                              " - Read value: " + value);
            return value;
        } catch (InterruptedException e) {
            e.printStackTrace();
            return null;
        } finally {
            rwLock.readLock().unlock();
        }
    }
    
    // WRITE operation (exclusive lock)
    public void put(String key, String value) {
        rwLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + 
                              " - Writing: " + key + " = " + value);
            Thread.sleep(2000);
            data.put(key, value);
            System.out.println(Thread.currentThread().getName() + 
                              " - Write complete");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            rwLock.writeLock().unlock();
        }
    }
}

public class CacheExample {
    public static void main(String[] args) throws InterruptedException {
        Cache cache = new Cache();
        
        // Initial write
        Thread writer = new Thread(() -> cache.put("user", "John"), "Writer");
        writer.start();
        writer.join();  // Wait for write to complete
        
        // Multiple readers (can read simultaneously)
        Thread r1 = new Thread(() -> cache.get("user"), "Reader-1");
        Thread r2 = new Thread(() -> cache.get("user"), "Reader-2");
        Thread r3 = new Thread(() -> cache.get("user"), "Reader-3");
        
        r1.start();
        r2.start();
        r3.start();
    }
}
```

**Output:**
```
Writer - Writing: user = John
Writer - Write complete
Reader-1 - Reading: user
Reader-2 - Reading: user      ← Multiple readers simultaneously!
Reader-3 - Reading: user
Reader-1 - Read value: John
Reader-2 - Read value: John
Reader-3 - Read value: John
```

### Example 3: Resource Pool with Semaphore

```java
import java.util.concurrent.Semaphore;

class ResourcePool {
    private Semaphore semaphore;
    private String[] resources;
    private boolean[] available;
    
    public ResourcePool(int poolSize) {
        this.semaphore = new Semaphore(poolSize);
        this.resources = new String[poolSize];
        this.available = new boolean[poolSize];
        
        for (int i = 0; i < poolSize; i++) {
            resources[i] = "Resource-" + (i + 1);
            available[i] = true;
        }
    }
    
    public String acquire() throws InterruptedException {
        semaphore.acquire();
        return getResource();
    }
    
    private synchronized String getResource() {
        for (int i = 0; i < available.length; i++) {
            if (available[i]) {
                available[i] = false;
                System.out.println(Thread.currentThread().getName() + 
                                  " acquired " + resources[i]);
                return resources[i];
            }
        }
        return null;
    }
    
    public void release(String resource) {
        if (releaseResource(resource)) {
            semaphore.release();
        }
    }
    
    private synchronized boolean releaseResource(String resource) {
        for (int i = 0; i < resources.length; i++) {
            if (resources[i].equals(resource)) {
                available[i] = true;
                System.out.println(Thread.currentThread().getName() + 
                                  " released " + resources[i]);
                return true;
            }
        }
        return false;
    }
}

public class ResourcePoolExample {
    public static void main(String[] args) {
        ResourcePool pool = new ResourcePool(3);  // 3 resources
        
        // 5 workers competing for 3 resources
        for (int i = 1; i <= 5; i++) {
            new Thread(() -> {
                try {
                    String resource = pool.acquire();
                    Thread.sleep(2000);  // Use resource
                    pool.release(resource);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }, "Worker-" + i).start();
        }
    }
}
```

**Output:**
```
Worker-1 acquired Resource-1
Worker-2 acquired Resource-2
Worker-3 acquired Resource-3
Worker-4 - Waiting...
Worker-5 - Waiting...
[2 seconds later]
Worker-1 released Resource-1
Worker-4 acquired Resource-1
Worker-2 released Resource-2
Worker-5 acquired Resource-2
...
```

### Example 4: Bounded Buffer with Condition

```java
import java.util.LinkedList;
import java.util.Queue;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

class BoundedBuffer<T> {
    private Queue<T> queue = new LinkedList<>();
    private int capacity;
    private ReentrantLock lock = new ReentrantLock();
    private Condition notFull = lock.newCondition();
    private Condition notEmpty = lock.newCondition();
    
    public BoundedBuffer(int capacity) {
        this.capacity = capacity;
    }
    
    public void put(T item) throws InterruptedException {
        lock.lock();
        try {
            while (queue.size() == capacity) {
                System.out.println(Thread.currentThread().getName() + 
                                  " - Buffer full, waiting...");
                notFull.await();
            }
            
            queue.add(item);
            System.out.println(Thread.currentThread().getName() + 
                              " - Added: " + item + 
                              " | Size: " + queue.size());
            notEmpty.signal();
            
        } finally {
            lock.unlock();
        }
    }
    
    public T take() throws InterruptedException {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                System.out.println(Thread.currentThread().getName() + 
                                  " - Buffer empty, waiting...");
                notEmpty.await();
            }
            
            T item = queue.poll();
            System.out.println(Thread.currentThread().getName() + 
                              " - Removed: " + item + 
                              " | Size: " + queue.size());
            notFull.signal();
            return item;
            
        } finally {
            lock.unlock();
        }
    }
}

public class BoundedBufferExample {
    public static void main(String[] args) {
        BoundedBuffer<Integer> buffer = new BoundedBuffer<>(3);
        
        // Producer
        Thread producer = new Thread(() -> {
            try {
                for (int i = 1; i <= 10; i++) {
                    buffer.put(i);
                    Thread.sleep(500);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "Producer");
        
        // Consumer
        Thread consumer = new Thread(() -> {
            try {
                for (int i = 1; i <= 10; i++) {
                    buffer.take();
                    Thread.sleep(1000);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "Consumer");
        
        producer.start();
        consumer.start();
    }
}
```

---

## Summary and Best Practices

### Lock Types Quick Reference

| Lock Type | Use Case | Concurrency | Complexity |
|-----------|----------|-------------|------------|
| **synchronized** | Simple scenarios, object-based locking | Low | Low |
| **ReentrantLock** | Explicit control, different objects | Medium | Medium |
| **ReadWriteLock** | High reads, low writes | High | Medium |
| **StampedLock** | Very high reads, optimistic scenarios | Very High | High |
| **Semaphore** | Limited resources, multiple permits | Controlled | Medium |

### Decision Tree: Which Lock to Use?

```
┌─────────────────────────────────────────────────┐
│ Need simple locking on same object?             │
│ YES → Use synchronized                          │
└─────────────────────────────────────────────────┘
                    ↓ NO
┌─────────────────────────────────────────────────┐
│ Need locking across different objects?          │
│ YES → Use ReentrantLock                         │
└─────────────────────────────────────────────────┘
                    ↓ NO
┌─────────────────────────────────────────────────┐
│ Reads >> Writes? (90% reads, 10% writes)       │
│ YES → Use ReadWriteLock                         │
└─────────────────────────────────────────────────┘
                    ↓ NO
┌─────────────────────────────────────────────────┐
│ Need optimistic locking? (very rare conflicts)  │
│ YES → Use StampedLock (optimistic read)        │
└─────────────────────────────────────────────────┘
                    ↓ NO
┌─────────────────────────────────────────────────┐
│ Need to limit concurrent access? (N threads)    │
│ YES → Use Semaphore                             │
└─────────────────────────────────────────────────┘
```

### Best Practices

#### ✓ DO's

1. **Always use try-finally with locks**
   ```java
   lock.lock();
   try {
       // Critical section
   } finally {
       lock.unlock();  // Always executed
   }
   ```

2. **Use appropriate lock for scenario**
   ```java
   // High reads, low writes
   ReadWriteLock rwLock = new ReentrantReadWriteLock();
   
   // Limited resources
   Semaphore semaphore = new Semaphore(5);
   ```

3. **Use Condition instead of wait/notify with custom locks**
   ```java
   ReentrantLock lock = new ReentrantLock();
   Condition condition = lock.newCondition();
   
   condition.await();  // Not wait()
   condition.signal(); // Not notify()
   ```

4. **Use while loop with await/wait**
   ```java
   while (condition) {
       condition.await();
   }
   ```

5. **Consider fairness for locks**
   ```java
   ReentrantLock lock = new ReentrantLock(true);  // Fair lock
   // Threads acquire lock in order they requested
   ```

#### ✗ DON'Ts

1. **Don't forget to unlock**
   ```java
   lock.lock();
   // ... code ...
   lock.unlock();  // If exception, never executes!
   
   // Use try-finally instead
   ```

2. **Don't mix synchronized with custom locks**
   ```java
   // BAD
   synchronized(obj) {
       condition.await();  // ERROR!
   }
   ```

3. **Don't hold locks longer than necessary**
   ```java
   // BAD
   lock.lock();
   performDatabaseOperation();  // Long operation
   performNetworkCall();        // Long operation
   lock.unlock();
   
   // GOOD
   prepareData();
   lock.lock();
   updateSharedData();  // Only critical section
   lock.unlock();
   sendToNetwork();
   ```

4. **Don't acquire multiple locks without caution**
   ```java
   // Can cause DEADLOCK
   lock1.lock();
   lock2.lock();  // If another thread locks in opposite order
   ```

### Performance Comparison

```
Scenario: 1000 threads, 90% reads, 10% writes

synchronized:        ████████████████████ (100% baseline)
ReentrantLock:       ████████████████████ (similar)
ReadWriteLock:       ████████ (60% time - faster!)
StampedLock:         ████ (40% time - fastest!)
```

### Common Interview Questions

**Q1: What's the difference between synchronized and ReentrantLock?**
- synchronized: Object-based, implicit lock/unlock, simpler
- ReentrantLock: Lock-based, explicit lock/unlock, more features (tryLock, fairness)

**Q2: When to use ReadWriteLock?**
- When reads are significantly more frequent than writes (e.g., cache)

**Q3: What is optimistic locking?**
- No actual lock acquired, validates before updating using version/stamp

**Q4: Difference between Semaphore(1) and ReentrantLock?**
- Similar behavior, but Semaphore allows different thread to release
- ReentrantLock requires same thread to lock/unlock

**Q5: Can we use wait/notify with ReentrantLock?**
- No! Use Condition interface (await/signal) instead

**Q6: What is the stamp in StampedLock?**
- A long value representing lock state/version, used for validation in optimistic reads

**Q7: Can Semaphore have 0 permits?**
- Yes! All threads will block until permits are released

### Real-World Applications

1. **Database Connection Pools**
   - Use Semaphore to limit concurrent connections

2. **Cache Systems**
   - Use ReadWriteLock for high-read scenarios

3. **Rate Limiting APIs**
   - Use Semaphore to limit requests per second

4. **Optimistic Concurrency in Databases**
   - Use version numbers (like StampedLock's optimistic read)

5. **Resource Management**
   - Use Semaphore for printer queues, thread pools

---

## Quick Reference Card

### Lock Acquisition

```java
// ReentrantLock
lock.lock();
lock.tryLock();
lock.tryLock(time, unit);
lock.unlock();

// ReadWriteLock
rwLock.readLock().lock();
rwLock.readLock().unlock();
rwLock.writeLock().lock();
rwLock.writeLock().unlock();

// StampedLock
long stamp = sLock.readLock();
sLock.unlockRead(stamp);
long stamp = sLock.writeLock();
sLock.unlockWrite(stamp);
long stamp = sLock.tryOptimisticRead();
boolean valid = sLock.validate(stamp);

// Semaphore
semaphore.acquire();
semaphore.acquire(n);
semaphore.release();
semaphore.release(n);
semaphore.tryAcquire();

// Condition
condition.await();
condition.signal();
condition.signalAll();
```

### Common Patterns

```java
// Pattern 1: Basic lock
lock.lock();
try {
    // Critical section
} finally {
    lock.unlock();
}

// Pattern 2: Try lock
if (lock.tryLock()) {
    try {
        // Critical section
    } finally {
        lock.unlock();
    }
}

// Pattern 3: Read-write
rwLock.readLock().lock();
try {
    // Read operation
} finally {
    rwLock.readLock().unlock();
}

// Pattern 4: Producer-consumer
lock.lock();
try {
    while (condition) {
        condition.await();
    }
    // Work
    condition.signalAll();
} finally {
    lock.unlock();
}
```

---

## Conclusion

You now understand:
- ✓ Limitations of synchronized and need for custom locks
- ✓ ReentrantLock for explicit locking control
- ✓ Shared vs Exclusive locks concept
- ✓ ReadWriteLock for high-read scenarios
- ✓ Optimistic vs Pessimistic locking strategies
- ✓ StampedLock for optimistic reads
- ✓ Semaphore for limiting concurrent access
- ✓ Condition interface for inter-thread communication

**Practice Tasks:**
1. Implement a thread-safe cache using ReadWriteLock
2. Create a connection pool using Semaphore
3. Build a producer-consumer with Condition
4. Experiment with StampedLock's optimistic read

**Remember:**
- Choose the right lock for your scenario
- Always use try-finally
- Prefer simple solutions (synchronized) unless you need advanced features
- Test thoroughly for race conditions and deadlocks

---

*Master these concepts and you'll handle any concurrent programming challenge!* 🔒1: Connection Pool

```java
// Database has 5 connections
Semaphore connectionPool = new Semaphore(5);

// Maximum 5 threads can get connections simultaneously
connectionPool.acquire();  // Get connection
try {
    // Use database connection
    executeQuery();
} finally {
    connectionPool.release();  // Return connection
}
```

#### Use Case
