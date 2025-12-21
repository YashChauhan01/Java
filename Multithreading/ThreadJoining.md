# Java Multithreading Notes - Part 3

## Table of Contents
1. [Producer-Consumer Problem Solution](#producer-consumer-problem-solution)
2. [Why stop(), suspend(), resume() are Deprecated](#why-stop-suspend-resume-are-deprecated)
3. [Thread Joining (join() method)](#thread-joining-join-method)
4. [Thread Priority](#thread-priority)
5. [Daemon Threads](#daemon-threads)
6. [Complete Working Examples](#complete-working-examples)
7. [Summary and Best Practices](#summary-and-best-practices)

---

## Producer-Consumer Problem Solution

### Problem Statement

**Scenario:** Two threads (Producer and Consumer) share a common fixed-size buffer (Queue)

**Requirements:**
1. **Producer**: Generates data and adds it to the queue
2. **Consumer**: Consumes data from the queue
3. **Constraints:**
   - Producer must WAIT if queue is FULL
   - Consumer must WAIT if queue is EMPTY

### Complete Solution

```java
import java.util.LinkedList;
import java.util.Queue;

class SharedResource {
    private Queue<Integer> queue;
    private int bufferSize;
    
    public SharedResource(int bufferSize) {
        this.queue = new LinkedList<>();
        this.bufferSize = bufferSize;
    }
    
    // Producer calls this method
    public synchronized void produce(int item) throws InterruptedException {
        // Wait while buffer is full
        while (queue.size() == bufferSize) {
            System.out.println(Thread.currentThread().getName() + 
                              " - Buffer is FULL, waiting...");
            wait();  // Releases lock and waits
        }
        
        // Add item to queue
        queue.add(item);
        System.out.println(Thread.currentThread().getName() + 
                          " - Produced: " + item + 
                          " | Queue size: " + queue.size());
        
        // Notify waiting consumers
        notifyAll();
    }
    
    // Consumer calls this method
    public synchronized int consume() throws InterruptedException {
        // Wait while buffer is empty
        while (queue.isEmpty()) {
            System.out.println(Thread.currentThread().getName() + 
                              " - Buffer is EMPTY, waiting...");
            wait();  // Releases lock and waits
        }
        
        // Remove item from queue
        int item = queue.poll();
        System.out.println(Thread.currentThread().getName() + 
                          " - Consumed: " + item + 
                          " | Queue size: " + queue.size());
        
        // Notify waiting producers
        notifyAll();
        
        return item;
    }
}

public class ProducerConsumerSolution {
    public static void main(String[] args) {
        // Shared buffer with capacity 3
        SharedResource sharedBuffer = new SharedResource(3);
        
        // Producer thread - produces 6 items
        Thread producer = new Thread(() -> {
            try {
                for (int i = 1; i <= 6; i++) {
                    sharedBuffer.produce(i);
                    Thread.sleep(500);  // Simulate production time
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "Producer");
        
        // Consumer thread - consumes 6 items
        Thread consumer = new Thread(() -> {
            try {
                for (int i = 1; i <= 6; i++) {
                    sharedBuffer.consume();
                    Thread.sleep(1000);  // Simulate consumption time
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

### Expected Output

```
Producer - Produced: 1 | Queue size: 1
Producer - Produced: 2 | Queue size: 2
Consumer - Consumed: 1 | Queue size: 1
Producer - Produced: 3 | Queue size: 2
Producer - Produced: 4 | Queue size: 3
Producer - Buffer is FULL, waiting...
Consumer - Consumed: 2 | Queue size: 2
Producer - Produced: 5 | Queue size: 3
Consumer - Consumed: 3 | Queue size: 2
Producer - Produced: 6 | Queue size: 3
Consumer - Consumed: 4 | Queue size: 2
Consumer - Consumed: 5 | Queue size: 1
Consumer - Consumed: 6 | Queue size: 0
```

### How It Works - Step by Step

```
Initial State: Queue is empty, capacity = 3
┌─────┬─────┬─────┐
│     │     │     │  Empty Queue
└─────┴─────┴─────┘

Step 1: Producer produces item 1
┌─────┬─────┬─────┐
│  1  │     │     │  Queue size: 1
└─────┴─────┴─────┘

Step 2: Producer produces item 2
┌─────┬─────┬─────┐
│  1  │  2  │     │  Queue size: 2
└─────┴─────┴─────┘

Step 3: Consumer consumes item 1
┌─────┬─────┬─────┐
│  2  │     │     │  Queue size: 1
└─────┴─────┴─────┘

Step 4: Producer produces items 3, 4 (Queue FULL)
┌─────┬─────┬─────┐
│  2  │  3  │  4  │  Queue size: 3 (FULL!)
└─────┴─────┴─────┘

Step 5: Producer tries to produce item 5
→ Queue is FULL
→ Producer calls wait()
→ Producer releases lock
→ Producer goes to WAITING state

Step 6: Consumer consumes item 2
┌─────┬─────┬─────┐
│  3  │  4  │     │  Queue size: 2
└─────┴─────┴─────┘
→ Consumer calls notifyAll()
→ Producer wakes up
→ Producer produces item 5
```

### Key Points to Remember

1. **Use while loop with wait()** (not if loop)
   - Prevents spurious wakeup issues
   - Re-checks condition after waking up

2. **Use notifyAll() instead of notify()**
   - Wakes up all waiting threads
   - More reliable than notify() which wakes only one thread

3. **synchronized is mandatory**
   - Both produce() and consume() must be synchronized
   - Ensures only one thread modifies queue at a time

4. **wait() releases the lock**
   - Allows other thread to acquire lock
   - Essential for avoiding deadlock

---

## Why stop(), suspend(), resume() are Deprecated

### The Problem with These Methods

These methods are **DEPRECATED** because they are inherently **UNSAFE** and can cause:
- Deadlocks
- Data corruption
- Resource leaks

### 1. stop() Method - Why Deprecated?

**What stop() does:**
- Terminates the thread **ABRUPTLY**
- Thread goes to TERMINATED state immediately
- **NO lock release happens**
- **NO resource cleanup happens**

#### The Danger

```java
class Resource {
    public synchronized void criticalOperation() {
        // Step 1: Acquire lock
        System.out.println("Lock acquired");
        
        // Step 2: Do some work
        performDatabaseUpdate();
        
        // Step 3: If thread is stopped here...
        // Lock is NEVER released!
        
        // Step 4: This cleanup never happens
        cleanup();
    }
}

Thread t1 = new Thread(() -> resource.criticalOperation());
t1.start();
t1.stop();  // DANGEROUS! Lock not released!
```

#### Deadlock Scenario

```
Thread-1:
├─ Acquires lock on Resource-1
├─ Thread-1.stop() is called
├─ Thread-1 DIES immediately
└─ Lock on Resource-1 is NEVER released

Thread-2:
├─ Tries to acquire lock on Resource-1
├─ Waits forever...
└─ DEADLOCK! Program hangs
```

**Visual Representation:**

```
Time T1: Thread-1 acquires lock on Resource-1
┌──────────┐
│ Thread-1 │ ──[Lock]──> Resource-1
└──────────┘

Time T2: Thread-2 wants same resource
┌──────────┐
│ Thread-2 │ ──[Waiting]──> Resource-1 (locked by Thread-1)
└──────────┘

Time T3: Thread-1.stop() is called
┌──────────┐
│ Thread-1 │ ──[DEAD]──> Resource-1 (STILL LOCKED!)
└──────────┘

Time T4: Thread-2 still waiting
┌──────────┐
│ Thread-2 │ ──[Waiting Forever]──> DEADLOCK!
└──────────┘
```

### 2. suspend() Method - Why Deprecated?

**What suspend() does:**
- Puts thread on hold (like pause)
- Thread keeps ALL locks
- **NO lock release happens**

**Difference from wait():**
- `wait()` → Releases ALL monitor locks
- `suspend()` → Keeps ALL locks (DANGEROUS!)

#### The Danger

```java
Thread t1 = new Thread(() -> {
    synchronized(resource) {
        System.out.println("Lock acquired");
        // If suspended here, lock is NOT released!
    }
});

t1.start();
t1.suspend();  // Thread paused but still holds lock!

// Any other thread trying to access resource will be blocked forever
```

### 3. resume() Method - Why Deprecated?

**What resume() does:**
- Resumes a suspended thread
- Since `suspend()` is deprecated, `resume()` must also be deprecated
- They work as a pair (like wait/notify)

### Complete Example Demonstrating the Problem

```java
class SharedResource {
    public synchronized void produce() {
        try {
            System.out.println(Thread.currentThread().getName() + 
                              " - Lock acquired");
            
            // Simulate work that takes 8 seconds
            Thread.sleep(8000);
            
            System.out.println(Thread.currentThread().getName() + 
                              " - Lock released");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

public class DeprecatedMethodsDemo {
    public static void main(String[] args) throws InterruptedException {
        SharedResource resource = new SharedResource();
        
        System.out.println("Main thread started");
        
        // Thread-1: Will acquire lock and get suspended
        Thread thread1 = new Thread(() -> {
            System.out.println("Thread-1 calling produce()");
            resource.produce();
        }, "Thread-1");
        
        // Thread-2: Will wait for lock
        Thread thread2 = new Thread(() -> {
            Thread.sleep(1000);  // Sleep to ensure Thread-1 goes first
            System.out.println("Thread-2 calling produce()");
            resource.produce();  // Will wait for lock
        }, "Thread-2");
        
        thread1.start();
        thread2.start();
        
        // Wait 3 seconds, then suspend Thread-1
        Thread.sleep(3000);
        
        thread1.suspend();  // DANGEROUS!
        System.out.println("Thread-1 is SUSPENDED");
        
        System.out.println("Main thread finishing");
    }
}
```

**Output WITHOUT resume():**
```
Main thread started
Thread-1 calling produce()
Thread-1 - Lock acquired
Thread-2 calling produce()
Thread-1 is SUSPENDED
Main thread finishing
[Program hangs forever - Thread-2 waiting for lock]
```

**What Happened:**
```
1. Thread-1 starts, acquires lock
2. Thread-1 starts sleeping (holding lock)
3. Thread-2 starts, tries to acquire lock, BLOCKS
4. Main suspends Thread-1 (lock NOT released)
5. Main finishes
6. Thread-2 waits FOREVER (DEADLOCK)
```

**With resume():**
```java
// Add this after suspend
Thread.sleep(3000);
thread1.resume();  // Resume Thread-1
System.out.println("Thread-1 is RESUMED");
```

**Output WITH resume():**
```
Main thread started
Thread-1 calling produce()
Thread-1 - Lock acquired
Thread-2 calling produce()
Thread-1 is SUSPENDED
Main thread finishing
Thread-1 is RESUMED
Thread-1 - Lock released
Thread-2 - Lock acquired
Thread-2 - Lock released
```

### Safe Alternatives

#### Instead of stop() - Use Flag

```java
class SafeTask implements Runnable {
    private volatile boolean running = true;
    
    public void run() {
        while (running) {
            // Do work
            if (!running) {
                cleanup();  // Proper cleanup
                break;
            }
        }
    }
    
    public void stopTask() {
        running = false;  // Safe way to stop
    }
}
```

#### Instead of suspend()/resume() - Use wait()/notify()

```java
class SafeTask implements Runnable {
    private boolean paused = false;
    
    public void run() {
        synchronized(this) {
            while (true) {
                while (paused) {
                    wait();  // Releases lock!
                }
                // Do work
            }
        }
    }
    
    public synchronized void pause() {
        paused = true;
    }
    
    public synchronized void resume() {
        paused = false;
        notifyAll();  // Wake up thread
    }
}
```

---

## Thread Joining (join() method)

### What is join()?

The `join()` method makes the **current thread wait** for another thread to **finish** its execution.

**Syntax:**
```java
thread.join();  // Current thread waits indefinitely
thread.join(milliseconds);  // Current thread waits for specified time
```

### Why Use join()?

**Use Cases:**
1. When you need threads to complete in a specific order
2. When a thread depends on results from another thread
3. When you need to coordinate thread execution
4. When you want to ensure tasks complete before proceeding

### Basic Example

```java
public class JoinExample {
    public static void main(String[] args) throws InterruptedException {
        System.out.println("Main thread started");
        
        Thread thread1 = new Thread(() -> {
            try {
                System.out.println("Thread-1 started working");
                Thread.sleep(3000);  // Simulate 3 seconds of work
                System.out.println("Thread-1 finished working");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "Thread-1");
        
        thread1.start();
        
        System.out.println("Main thread finishing");
    }
}
```

**Output WITHOUT join():**
```
Main thread started
Main thread finishing
Thread-1 started working
[3 seconds later]
Thread-1 finished working
```

**Notice:** Main thread finishes BEFORE Thread-1 completes!

### Using join() to Wait

```java
public class JoinExample {
    public static void main(String[] args) throws InterruptedException {
        System.out.println("Main thread started");
        
        Thread thread1 = new Thread(() -> {
            try {
                System.out.println("Thread-1 started working");
                Thread.sleep(3000);  // Simulate 3 seconds of work
                System.out.println("Thread-1 finished working");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "Thread-1");
        
        thread1.start();
        
        System.out.println("Main thread waiting for Thread-1 to finish");
        thread1.join();  // Main waits here!
        
        System.out.println("Main thread finishing");
    }
}
```

**Output WITH join():**
```
Main thread started
Main thread waiting for Thread-1 to finish
Thread-1 started working
[3 seconds later]
Thread-1 finished working
Main thread finishing
```

**Notice:** Main thread waits for Thread-1 to complete!

### Visual Representation

**Without join():**
```
Main Thread:  [Start]──[Creates Thread-1]──[Finish] ✓
                         │
                         └──→ Thread-1: [Start]──[Work]──[Finish] ✓

Main finishes before Thread-1!
```

**With join():**
```
Main Thread:  [Start]──[Creates Thread-1]──[join()]──[Wait...]──[Finish] ✓
                         │                              ↑
                         └──→ Thread-1: [Start]──[Work]──[Finish]──┘

Main waits for Thread-1 to finish!
```

### Multiple Threads with join()

```java
public class MultipleJoinExample {
    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread(() -> {
            System.out.println("Thread-1 working");
            sleep(2000);
            System.out.println("Thread-1 done");
        }, "Thread-1");
        
        Thread thread2 = new Thread(() -> {
            System.out.println("Thread-2 working");
            sleep(3000);
            System.out.println("Thread-2 done");
        }, "Thread-2");
        
        Thread thread3 = new Thread(() -> {
            System.out.println("Thread-3 working");
            sleep(1000);
            System.out.println("Thread-3 done");
        }, "Thread-3");
        
        // Start all threads
        thread1.start();
        thread2.start();
        thread3.start();
        
        // Wait for all to complete
        thread1.join();
        thread2.join();
        thread3.join();
        
        System.out.println("All threads completed!");
    }
    
    private static void sleep(int ms) {
        try {
            Thread.sleep(ms);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

**Output:**
```
Thread-1 working
Thread-2 working
Thread-3 working
Thread-3 done       [after 1 second]
Thread-1 done       [after 2 seconds]
Thread-2 done       [after 3 seconds]
All threads completed!
```

### join() with Timeout

```java
thread.join(2000);  // Wait maximum 2 seconds

if (thread.isAlive()) {
    System.out.println("Thread still running after 2 seconds");
} else {
    System.out.println("Thread completed within 2 seconds");
}
```

### Real-World Example: Data Processing Pipeline

```java
public class DataProcessingPipeline {
    public static void main(String[] args) throws InterruptedException {
        // Step 1: Fetch data from database
        Thread fetchThread = new Thread(() -> {
            System.out.println("Fetching data from database...");
            sleep(2000);
            System.out.println("Data fetched successfully");
        }, "Fetch-Thread");
        
        // Step 2: Process data (depends on Step 1)
        Thread processThread = new Thread(() -> {
            System.out.println("Processing data...");
            sleep(3000);
            System.out.println("Data processed successfully");
        }, "Process-Thread");
        
        // Step 3: Save results (depends on Step 2)
        Thread saveThread = new Thread(() -> {
            System.out.println("Saving results...");
            sleep(1000);
            System.out.println("Results saved successfully");
        }, "Save-Thread");
        
        // Execute pipeline in order
        fetchThread.start();
        fetchThread.join();  // Wait for fetch to complete
        
        processThread.start();
        processThread.join();  // Wait for processing to complete
        
        saveThread.start();
        saveThread.join();  // Wait for save to complete
        
        System.out.println("Pipeline completed!");
    }
    
    private static void sleep(int ms) {
        try {
            Thread.sleep(ms);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

**Output:**
```
Fetching data from database...
[2 seconds]
Data fetched successfully
Processing data...
[3 seconds]
Data processed successfully
Saving results...
[1 second]
Results saved successfully
Pipeline completed!
```

---

## Thread Priority

### What is Thread Priority?

Thread priority is a **hint** to the thread scheduler about which thread should be given preference for execution.

**Priority Range:** 1 to 10
- **1** = Lowest priority (`Thread.MIN_PRIORITY`)
- **5** = Normal priority (`Thread.NORM_PRIORITY`) - Default
- **10** = Highest priority (`Thread.MAX_PRIORITY`)

### Setting Thread Priority

```java
Thread thread = new Thread(() -> {
    System.out.println("Working...");
});

// Set priority
thread.setPriority(Thread.MAX_PRIORITY);  // 10
thread.setPriority(Thread.MIN_PRIORITY);  // 1
thread.setPriority(Thread.NORM_PRIORITY); // 5
thread.setPriority(7);  // Custom priority

// Get priority
int priority = thread.getPriority();
System.out.println("Priority: " + priority);
```

### Important: Priority is NOT Guaranteed!

**CRITICAL CONCEPT:** Thread priority is just a **HINT** to the thread scheduler, **NOT a strict rule**.

```java
Thread t1 = new Thread(() -> System.out.println("T1"), "T1");
Thread t2 = new Thread(() -> System.out.println("T2"), "T2");
Thread t3 = new Thread(() -> System.out.println("T3"), "T3");
Thread t4 = new Thread(() -> System.out.println("T4"), "T4");

t1.setPriority(10);  // Highest
t2.setPriority(7);
t3.setPriority(3);
t4.setPriority(1);   // Lowest

t1.start();
t2.start();
t3.start();
t4.start();

// You might expect: T1 → T2 → T3 → T4
// But actual order is UNPREDICTABLE!
```

**Possible Outputs:**
```
Run 1: T1, T2, T3, T4  ✓ (Follows priority)
Run 2: T3, T1, T4, T2  ✗ (Ignores priority)
Run 3: T2, T4, T1, T3  ✗ (Random order)
Run 4: T1, T2, T3, T4  ✓ (Follows priority)
```

### Why Priority is Unreliable

1. **JVM doesn't guarantee priority order**
2. **Thread scheduler has its own logic**
3. **OS-dependent behavior**
4. **Other factors affect scheduling:**
   - CPU load
   - Number of cores
   - Other running processes

### Example Demonstrating Unreliability

```java
public class PriorityExample {
    public static void main(String[] args) {
        System.out.println("Main priority: " + 
                          Thread.currentThread().getPriority());
        
        Thread highPriority = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                System.out.println("HIGH Priority: " + i);
            }
        }, "High-Priority");
        
        Thread lowPriority = new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                System.out.println("LOW Priority: " + i);
            }
        }, "Low-Priority");
        
        highPriority.setPriority(Thread.MAX_PRIORITY);  // 10
        lowPriority.setPriority(Thread.MIN_PRIORITY);   // 1
        
        System.out.println("High thread priority: " + highPriority.getPriority());
        System.out.println("Low thread priority: " + lowPriority.getPriority());
        
        highPriority.start();
        lowPriority.start();
    }
}
```

**Output (Run 1):**
```
HIGH Priority: 0
HIGH Priority: 1
LOW Priority: 0
HIGH Priority: 2
LOW Priority: 1
...
```

**Output (Run 2):**
```
LOW Priority: 0
HIGH Priority: 0
LOW Priority: 1
LOW Priority: 2
HIGH Priority: 1
...
```

### Priority Inheritance

When a new thread is created, it **inherits** the priority of its parent thread.

```java
public class PriorityInheritance {
    public static void main(String[] args) {
        // Main thread has default priority 5
        System.out.println("Main priority: " + 
                          Thread.currentThread().getPriority());  // 5
        
        Thread child = new Thread(() -> {
            System.out.println("Child priority: " + 
                              Thread.currentThread().getPriority());  // 5
        });
        
        child.start();
        
        // Change main priority
        Thread.currentThread().setPriority(8);
        
        Thread child2 = new Thread(() -> {
            System.out.println("Child2 priority: " + 
                              Thread.currentThread().getPriority());  // 8
        });
        
        child2.start();
    }
}
```

### Best Practice

**❌ DON'T rely on thread priority in production code**

```java
// BAD - Don't do this
thread1.setPriority(10);  // Hoping it runs first
thread2.setPriority(1);   // Hoping it runs last
// Order is NOT guaranteed!
```

**✓ DO use proper synchronization mechanisms**

```java
// GOOD - Use join() for ordering
thread1.start();
thread1.join();  // Wait for thread1 to complete
thread2.start();  // Then start thread2
```

**✓ DO use wait/notify for coordination**

```java
// GOOD - Use wait/notify for coordination
synchronized(lock) {
    while (!ready) {
        lock.wait();
    }
    // Proceed when ready
}
```

---

## Daemon Threads

### What is a Daemon Thread?

**Daemon** = Background service thread that supports user threads

**Key Characteristic:** Daemon threads **automatically terminate** when all **user threads** finish execution.

### Types of Threads

1. **User Thread** (Default)
   - Normal threads
   - JVM waits for these to complete
   - Created by default

2. **Daemon Thread**
   - Background service threads
   - JVM doesn't wait for these
   - Dies when all user threads die

### Creating Daemon Threads

```java
Thread thread = new Thread(() -> {
    // Thread work
});

// Set as daemon BEFORE starting
thread.setDaemon(true);  // Make it daemon
thread.start();

// Check if daemon
boolean isDaemon = thread.isDaemon();
System.out.println("Is daemon: " + isDaemon);
```

**IMPORTANT:** Must call `setDaemon(true)` **BEFORE** calling `start()`

```java
thread.start();
thread.setDaemon(true);  // ERROR! IllegalThreadStateException
```

### User Thread vs Daemon Thread Behavior

#### Example 1: User Thread (Default)

```java
public class UserThreadExample {
    public static void main(String[] args) {
        System.out.println("Main thread started");
        
        Thread userThread = new Thread(() -> {
            try {
                System.out.println("User thread working...");
                Thread.sleep(5000);  // Sleep 5 seconds
                System.out.println("User thread completed");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        
        userThread.start();  // Default: User thread
        
        System.out.println("Main thread finished");
    }
}
```

**Output:**
```
Main thread started
Main thread finished
User thread working...
[5 seconds later]
User thread completed
[Program exits after user thread completes]
```

**Behavior:** JVM waits for user thread to complete even though main finished!

#### Example 2: Daemon Thread

```java
public class DaemonThreadExample {
    public static void main(String[] args) {
        System.out.println("Main thread started");
        
        Thread daemonThread = new Thread(() -> {
            try {
                System.out.println("Daemon thread working...");
                Thread.sleep(5000);  // Sleep 5 seconds
                System.out.println("Daemon thread completed");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        
        daemonThread.setDaemon(true);  // Make it daemon
        daemonThread.start();
        
        System.out.println("Main thread finished");
    }
}
```

**Output:**
```
Main thread started
Main thread finished
Daemon thread working...
[Program exits immediately - daemon thread killed!]
```

**Behavior:** JVM doesn't wait for daemon thread! Program exits when main (user thread) finishes.

### Complete Comparison Example

```java
public class DaemonVsUserThread {
    public static void main(String[] args) throws InterruptedException {
        System.out.println("Main started (User thread)");
        
        // User thread
        Thread userThread = new Thread(() -> {
            for (int i = 1; i <= 10; i++) {
                System.out.println("User thread: " + i);
                sleep(500);
            }
        }, "User-Thread");
        
        // Daemon thread
        Thread daemonThread = new Thread(() -> {
            for (int i = 1; i <= 10; i++) {
                System.out.println("Daemon thread: " + i);
                sleep(500);
            }
        }, "Daemon-Thread");
        
        daemonThread.setDaemon(true);  // Set as daemon
        
        userThread.start();
        daemonThread.start();
        
        // Main sleeps for 3 seconds then finishes
        Thread.sleep(3000);
        System.out.println("Main finished");
    }
    
    private static void sleep(int ms) {
        try {
            Thread.sleep(ms);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

**Output:**
```
Main started (User thread)
User thread: 1
Daemon thread: 1
User thread: 2
Daemon thread: 2
...
[After 3 seconds]
Main finished
User thread: 7
Daemon thread: 7
User thread: 8
Daemon thread: 8
...
User thread: 10  [Completes all 10]
[Daemon thread killed - might not complete all 10]
[Program exits when user thread completes]
```

### Real-World Use Cases for Daemon Threads

#### 1. Garbage Collection

```java
// JVM's garbage collector runs as daemon thread
// Cleans up unused objects while your program runs
// Automatically stops when program exits
```

#### 2. Auto-Save Feature

```java
public class AutoSaveExample {
    public static void main(String[] args) {
        // Auto-save daemon thread
        Thread autoSave = new Thread(() -> {
            while (true) {
                try {
                    Thread.sleep(30000);  // Every 30 seconds
                    System.out.println("Auto-saving document...");
                    saveDocument();
                } catch (InterruptedException e) {
                    break;
                }
            }
        }, "Auto-Save");
        
        autoSave.setDaemon(true);
        autoSave.start();
        
        // Main work (simulating user typing)
        System.out.println("User working on document...");
        sleep(120000);  // Work for 2 minutes
        
        System.out.println("User closed the application");
        // Program exits, auto-save daemon automatically stops
    }
    
    private static void saveDocument() {
        System.out.println("Document saved at: " + new Date());
    }
    
    private static void sleep(int ms) {
        try {
            Thread.sleep(ms);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

**Output:**
```
User working on document...
[After 30 seconds]
Auto-saving document...
Document saved at: Sun Dec 21 10:30:00 IST 2025
[After 30 seconds]
Auto-saving document...
Document saved at: Sun Dec 21 10:30:30 IST 2025
...
User closed the application
[Auto-save daemon automatically stops]
```

#### 3. Logging Service

```java
public class LoggingServiceExample {
    private static Queue<String> logQueue = new LinkedList<>();
    
    public static void main(String[] args) {
        // Logger daemon thread
        Thread logger = new Thread(() -> {
            while (true) {
                try {
                    Thread.sleep(1000);
                    if (!logQueue.isEmpty()) {
                        String log = logQueue.poll();
                        System.out.println("[LOG] " + log);
                        writeToFile(log);
                    }
                } catch (InterruptedException e) {
                    break;
                }
            }
        }, "Logger-Daemon");
        
        logger.setDaemon(true);
        logger.start();
        
        // Application work
        logQueue.add("Application started");
        performTask1();
        logQueue.add("Task 1 completed");
        performTask2();
        logQueue.add("Task 2 completed");
        
        System.out.println("Application finished");
        // Logger daemon stops automatically
    }
    
    private static void performTask1() {
        System.out.println("Performing task 1...");
    }
    
    private static void performTask2() {
        System.out.println("Performing task 2...");
    }
    
    private static void writeToFile(String log) {
        // Write to log file
    }
}
```

#### 4. Health Check Monitor

```java
public class HealthCheckMonitor {
    public static void main(String[] args) throws InterruptedException {
        // Health check daemon
        Thread healthCheck = new Thread(() -> {
            while (true) {
                try {
                    Thread.sleep(5000);  // Every 5 seconds
                    checkSystemHealth();
                } catch (InterruptedException e) {
                    break;
                }
            }
        }, "Health-Check");
        
        healthCheck.setDaemon(true);
        healthCheck.start();
        
        // Main application work
        System.out.println("Application running...");
        Thread.sleep(20000);  // Run for 20 seconds
        
        System.out.println("Application shutting down...");
        // Health check daemon stops automatically
    }
    
    private static void checkSystemHealth() {
        System.out.println("Health check: CPU=" + getCPUUsage() + 
                          "%, Memory=" + getMemoryUsage() + "%");
    }
    
    private static int getCPUUsage() {
        return (int)(Math.random() * 100);
    }
    
    private static int getMemoryUsage() {
        return (int)(Math.random() * 100);
    }
}
```

### Important Rules for Daemon Threads

1. **Set daemon BEFORE starting**
   ```java
   thread.setDaemon(true);  // Must be before start()
   thread.start();
   ```

2. **Daemon threads inherit daemon status**
   ```java
   Thread daemonParent = new Thread(() -> {
       Thread child = new Thread(() -> {
           // Child is automatically daemon too!
       });
       child.start();
   });
   daemonParent.setDaemon(true);
   daemonParent.start();
   ```

3. **JVM exits when only daemon threads remain**
   ```java
   // If only daemon threads are running
   // JVM will exit immediately
   ```

4. **Daemon threads can be killed mid-execution**
   ```java
   // Daemon thread might not complete its work
   // Don't use for critical operations!
   ```

### When to Use Daemon Threads

**✓ Good Use Cases:**
- Background monitoring
- Periodic cleanup
- Auto-save features
- Health checks
- Logging services
- Cache refresh

**✗ Bad Use Cases:**
- Database transactions (might not commit)
- File operations (might not complete)
- Critical business logic
- Resource cleanup (might not release resources)

### Visual Summary

```
┌─────────────────────────────────────────────────────┐
│              JVM Process Lifetime                    │
│                                                      │
│  User Threads:    [────────]                        │
│                      └─→ JVM waits for completion   │
│                                                      │
│  Daemon Threads:  [────X]                           │
│                      └─→ JVM kills when users done  │
│                                                      │
│  Result: JVM exits when all USER threads complete   │
└─────────────────────────────────────────────────────┘
```

---

## Complete Working Examples

### Example 1: Full Producer-Consumer Implementation

```java
import java.util.LinkedList;
import java.util.Queue;

class SharedBuffer {
    private Queue<Integer> buffer = new LinkedList<>();
    private int capacity;
    
    public SharedBuffer(int capacity) {
        this.capacity = capacity;
    }
    
    public synchronized void produce(int item) throws InterruptedException {
        while (buffer.size() == capacity) {
            System.out.println(Thread.currentThread().getName() + 
                              " - Buffer FULL, producer waiting...");
            wait();
        }
        
        buffer.add(item);
        System.out.println(Thread.currentThread().getName() + 
                          " - Produced: " + item + 
                          " | Size: " + buffer.size() + "/" + capacity);
        notifyAll();
    }
    
    public synchronized int consume() throws InterruptedException {
        while (buffer.isEmpty()) {
            System.out.println(Thread.currentThread().getName() + 
                              " - Buffer EMPTY, consumer waiting...");
            wait();
        }
        
        int item = buffer.poll();
        System.out.println(Thread.currentThread().getName() + 
                          " - Consumed: " + item + 
                          " | Size: " + buffer.size() + "/" + capacity);
        notifyAll();
        return item;
    }
}

public class CompleteProducerConsumer {
    public static void main(String[] args) {
        SharedBuffer buffer = new SharedBuffer(5);
        
        // Multiple producers
        Thread producer1 = new Thread(() -> {
            try {
                for (int i = 1; i <= 10; i++) {
                    buffer.produce(i);
                    Thread.sleep(300);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "Producer-1");
        
        Thread producer2 = new Thread(() -> {
            try {
                for (int i = 100; i <= 105; i++) {
                    buffer.produce(i);
                    Thread.sleep(500);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "Producer-2");
        
        // Multiple consumers
        Thread consumer1 = new Thread(() -> {
            try {
                for (int i = 1; i <= 8; i++) {
                    buffer.consume();
                    Thread.sleep(600);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "Consumer-1");
        
        Thread consumer2 = new Thread(() -> {
            try {
                for (int i = 1; i <= 7; i++) {
                    buffer.consume();
                    Thread.sleep(700);
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "Consumer-2");
        
        producer1.start();
        producer2.start();
        consumer1.start();
        consumer2.start();
    }
}
```

### Example 2: Thread Coordination with join()

```java
public class ThreadCoordinationExample {
    private static String[] data;
    
    public static void main(String[] args) throws InterruptedException {
        System.out.println("Starting data processing pipeline...\n");
        
        // Step 1: Fetch data
        Thread fetchThread = new Thread(() -> {
            System.out.println("Step 1: Fetching data from API...");
            sleep(2000);
            data = new String[]{"Record1", "Record2", "Record3"};
            System.out.println("Step 1: Data fetched successfully!");
            System.out.println();
        }, "Fetch-Thread");
        
        // Step 2: Transform data
        Thread transformThread = new Thread(() -> {
            System.out.println("Step 2: Transforming data...");
            sleep(1500);
            for (int i = 0; i < data.length; i++) {
                data[i] = data[i].toUpperCase();
            }
            System.out.println("Step 2: Data transformed successfully!");
            System.out.println();
        }, "Transform-Thread");
        
        // Step 3: Save data
        Thread saveThread = new Thread(() -> {
            System.out.println("Step 3: Saving data to database...");
            sleep(1000);
            for (String record : data) {
                System.out.println("Saved: " + record);
            }
            System.out.println("Step 3: Data saved successfully!");
            System.out.println();
        }, "Save-Thread");
        
        // Execute in sequence using join()
        long startTime = System.currentTimeMillis();
        
        fetchThread.start();
        fetchThread.join();  // Wait for fetch to complete
        
        transformThread.start();
        transformThread.join();  // Wait for transform to complete
        
        saveThread.start();
        saveThread.join();  // Wait for save to complete
        
        long endTime = System.currentTimeMillis();
        System.out.println("Pipeline completed in " + 
                          (endTime - startTime) + "ms");
    }
    
    private static void sleep(int ms) {
        try {
            Thread.sleep(ms);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

### Example 3: Daemon Thread for Background Service

```java
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class BackgroundServiceExample {
    private static volatile boolean applicationRunning = true;
    private static int taskCounter = 0;
    
    public static void main(String[] args) throws InterruptedException {
        System.out.println("Application started");
        
        // Daemon thread 1: System monitor
        Thread monitor = new Thread(() -> {
            DateTimeFormatter formatter = DateTimeFormatter.ofPattern("HH:mm:ss");
            while (true) {
                try {
                    Thread.sleep(5000);
                    System.out.println("[" + LocalDateTime.now().format(formatter) + 
                                      "] System Monitor: Tasks completed = " + taskCounter);
                } catch (InterruptedException e) {
                    break;
                }
            }
        }, "System-Monitor");
        monitor.setDaemon(true);
        monitor.start();
        
        // Daemon thread 2: Memory cleanup
        Thread cleanup = new Thread(() -> {
            while (true) {
                try {
                    Thread.sleep(10000);
                    System.out.println("[Memory Cleanup] Running garbage collection...");
                    System.gc();
                } catch (InterruptedException e) {
                    break;
                }
            }
        }, "Memory-Cleanup");
        cleanup.setDaemon(true);
        cleanup.start();
        
        // Main application work (User thread)
        for (int i = 1; i <= 5; i++) {
            System.out.println("\nExecuting Task " + i);
            Thread.sleep(3000);
            taskCounter++;
            System.out.println("Task " + i + " completed");
        }
        
        System.out.println("\nApplication shutting down...");
        applicationRunning = false;
        
        // Daemon threads will automatically stop when main ends
    }
}
```

**Output:**
```
Application started

Executing Task 1
[10:30:05] System Monitor: Tasks completed = 0
Task 1 completed

Executing Task 2
[10:30:10] System Monitor: Tasks completed = 1
[Memory Cleanup] Running garbage collection...
Task 2 completed

Executing Task 3
[10:30:15] System Monitor: Tasks completed = 2
Task 3 completed

Executing Task 4
[10:30:20] System Monitor: Tasks completed = 3
[Memory Cleanup] Running garbage collection...
Task 4 completed

Executing Task 5
[10:30:25] System Monitor: Tasks completed = 4
Task 5 completed

Application shutting down...
[Daemon threads automatically stop]
```

### Example 4: Real-World Banking System

```java
import java.util.concurrent.atomic.AtomicInteger;

class BankAccount {
    private int balance = 10000;
    private AtomicInteger transactionId = new AtomicInteger(1);
    
    public synchronized void withdraw(String customer, int amount) {
        int txnId = transactionId.getAndIncrement();
        System.out.println("[TXN-" + txnId + "] " + customer + 
                          " requesting withdrawal: $" + amount);
        
        while (balance < amount) {
            try {
                System.out.println("[TXN-" + txnId + "] " + customer + 
                                  " waiting... Insufficient balance");
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        
        balance -= amount;
        System.out.println("[TXN-" + txnId + "] " + customer + 
                          " withdrew $" + amount + 
                          " | Remaining balance: $" + balance);
        notifyAll();
    }
    
    public synchronized void deposit(String customer, int amount) {
        int txnId = transactionId.getAndIncrement();
        System.out.println("[TXN-" + txnId + "] " + customer + 
                          " depositing: $" + amount);
        
        balance += amount;
        System.out.println("[TXN-" + txnId + "] " + customer + 
                          " deposited $" + amount + 
                          " | New balance: $" + balance);
        notifyAll();
    }
    
    public synchronized int getBalance() {
        return balance;
    }
}

public class BankingSystemExample {
    public static void main(String[] args) throws InterruptedException {
        BankAccount account = new BankAccount();
        
        // Customer 1: Multiple withdrawals
        Thread customer1 = new Thread(() -> {
            try {
                account.withdraw("Alice", 3000);
                Thread.sleep(1000);
                account.withdraw("Alice", 4000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "Customer-1");
        
        // Customer 2: Withdrawal
        Thread customer2 = new Thread(() -> {
            try {
                Thread.sleep(500);
                account.withdraw("Bob", 5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "Customer-2");
        
        // Customer 3: Deposit
        Thread customer3 = new Thread(() -> {
            try {
                Thread.sleep(2000);
                account.deposit("Charlie", 8000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "Customer-3");
        
        customer1.start();
        customer2.start();
        customer3.start();
        
        // Wait for all transactions
        customer1.join();
        customer2.join();
        customer3.join();
        
        System.out.println("\nFinal balance: $" + account.getBalance());
    }
}
```

**Output:**
```
[TXN-1] Alice requesting withdrawal: $3000
[TXN-1] Alice withdrew $3000 | Remaining balance: $7000
[TXN-2] Bob requesting withdrawal: $5000
[TXN-2] Bob withdrew $5000 | Remaining balance: $2000
[TXN-3] Alice requesting withdrawal: $4000
[TXN-3] Alice waiting... Insufficient balance
[TXN-4] Charlie depositing: $8000
[TXN-4] Charlie deposited $8000 | New balance: $10000
[TXN-3] Alice withdrew $4000 | Remaining balance: $6000

Final balance: $6000
```

---

## Summary and Best Practices

### Key Concepts Summary

#### 1. Producer-Consumer Problem
- **Use synchronized methods** for thread-safe queue access
- **Use while loop** with wait() (not if loop)
- **Use notifyAll()** to wake all waiting threads
- **wait() releases locks**, allowing other threads to proceed

#### 2. Deprecated Methods
- **stop()**: Terminates thread abruptly, doesn't release locks → **Use flags instead**
- **suspend()**: Pauses thread but keeps locks → **Use wait()/notify() instead**
- **resume()**: Resumes suspended thread → **Use notify() instead**

#### 3. Thread Join
- **join()**: Makes current thread wait for another thread to finish
- **Use for coordination** between dependent threads
- **Can specify timeout**: `join(milliseconds)`

#### 4. Thread Priority
- **Priority is a HINT**, not a guarantee
- **Range: 1-10** (1=lowest, 10=highest, 5=default)
- **Don't rely on priority** in production code
- **Use proper synchronization** instead

#### 5. Daemon Threads
- **Background service threads** that support user threads
- **Automatically die** when all user threads finish
- **Set before starting**: `setDaemon(true)` before `start()`
- **Use for**: logging, monitoring, auto-save, cleanup

### Best Practices

#### ✓ DO's

1. **Always use while with wait()**
   ```java
   while (condition) {
       wait();
   }
   ```

2. **Use join() for thread coordination**
   ```java
   thread.start();
   thread.join();  // Wait for completion
   ```

3. **Set daemon before starting**
   ```java
   thread.setDaemon(true);
   thread.start();
   ```

4. **Use flags to stop threads safely**
   ```java
   private volatile boolean running = true;
   
   public void run() {
       while (running) {
           // Work
       }
   }
   
   public void stopThread() {
       running = false;
   }
   ```

5. **Handle InterruptedException properly**
   ```java
   try {
       Thread.sleep(1000);
   } catch (InterruptedException e) {
       Thread.currentThread().interrupt();
       // Handle appropriately
   }
   ```

#### ✗ DON'Ts

1. **Don't use stop(), suspend(), resume()**
   ```java
   thread.stop();     // NEVER!
   thread.suspend();  // NEVER!
   thread.resume();   // NEVER!
   ```

2. **Don't rely on thread priority**
   ```java
   thread.setPriority(10);  // Don't assume it runs first!
   ```

3. **Don't use daemon threads for critical operations**
   ```java
   // BAD - might not complete
   daemonThread.setDaemon(true);
   daemonThread.start();  // Saving critical data
   ```

4. **Don't call setDaemon() after start()**
   ```java
   thread.start();
   thread.setDaemon(true);  // IllegalThreadStateException!
   ```

5. **Don't use if with wait()**
   ```java
   if (condition) {
       wait();  // BAD - spurious wakeup issue
   }
   ```

### Quick Reference Table

| Method | Purpose | Releases Lock? | When to Use |
|--------|---------|----------------|-------------|
| `wait()` | Wait for notification | Yes | Inter-thread communication |
| `notify()` | Wake one waiting thread | N/A | Signal one thread |
| `notifyAll()` | Wake all waiting threads | N/A | Signal all threads |
| `join()` | Wait for thread to die | No | Thread coordination |
| `sleep()` | Pause execution | No | Delay execution |
| `setDaemon()` | Mark as daemon | N/A | Background services |
| `setPriority()` | Hint to scheduler | N/A | Rarely (not reliable) |

### Common Interview Questions

#### Q1: Why is stop() deprecated?
**A:** Because it terminates threads abruptly without releasing locks or cleaning up resources, potentially causing deadlocks and data corruption.

#### Q2: What's the difference between wait() and suspend()?
**A:** `wait()` releases all monitor locks and can be resumed with `notify()`. `suspend()` keeps all locks and must be resumed with `resume()`. `suspend()` can cause deadlocks.

#### Q3: What's the difference between wait() and sleep()?
**A:** 
- `wait()`: Releases locks, needs `notify()` to wake up, must be in synchronized block
- `sleep()`: Keeps locks, wakes up automatically, can be called anywhere

#### Q4: Can we restart a terminated thread?
**A:** No! Once a thread reaches TERMINATED state, it cannot be restarted. Calling `start()` again throws `IllegalThreadStateException`.

#### Q5: What happens if all threads are daemon threads?
**A:** JVM will exit immediately because there are no user threads to keep it running.

#### Q6: Is thread priority guaranteed?
**A:** No! Thread priority is just a hint to the scheduler. Actual execution order depends on OS and JVM implementation.

#### Q7: When should we use join()?
**A:** Use `join()` when:
- You need threads to complete in specific order
- A thread depends on results from another thread
- You want to wait for multiple threads to complete before proceeding

### Real-World Applications

1. **Web Servers**: Producer-Consumer for request handling
2. **Databases**: Thread coordination for transaction management
3. **GUI Applications**: Daemon threads for auto-save
4. **Monitoring Systems**: Daemon threads for health checks
5. **Data Processing**: join() for pipeline stages
6. **Game Development**: Daemon threads for background music

### Testing Your Knowledge

Try implementing these scenarios:

1. **Read-Write Lock**: Multiple readers, single writer
2. **Dining Philosophers**: Deadlock prevention
3. **Thread Pool**: Fixed number of worker threads
4. **Print Queue**: Multiple printers, multiple print jobs
5. **Download Manager**: Multiple parallel downloads with coordination

---

## Additional Resources

### Thread States Diagram

```
                    ┌──────────┐
                    │   NEW    │
                    └──────────┘
                         │ start()
                         ↓
    ┌────────────────────────────────────┐
    │         RUNNABLE                   │
    └────────────────────────────────────┘
       ↑         ↓                ↓
       │         │                │
       │  wait() │                │ run() completes
       │  join() │                │
       │         ↓                ↓
    ┌──────────┐              ┌──────────┐
    │ WAITING  │              │TERMINATED│
    │ BLOCKED  │              └──────────┘
    │TIMED_WAIT│
    └──────────┘
       ↑
       │ notify()
       │ timeout
       └───────┘
```

### Method Cheat Sheet

```java
// Thread creation
Thread t = new Thread(runnable);
t.start();

// Thread coordination
t.join();                    // Wait for thread to die
t.join(1000);               // Wait max 1 second

// Daemon threads
t.setDaemon(true);          // Must be before start()
boolean isDaemon = t.isDaemon();

// Priority (rarely used)
t.setPriority(Thread.MAX_PRIORITY);
int p = t.getPriority();

// Inter-thread communication (in synchronized block)
wait();                     // Release lock and wait
notify();                   // Wake one waiting thread
notifyAll();                // Wake all waiting threads

// Thread info
String name = t.getName();
t.setName("MyThread");
boolean alive = t.isAlive();
Thread.State state = t.getState();

// Sleep
Thread.sleep(1000);         // Sleep 1 second

// Interrupt
t.interrupt();
boolean interrupted = t.isInterrupted();
```

---

## Conclusion

You now have a complete understanding of:
- ✓ Producer-Consumer problem and its solution
- ✓ Why certain thread methods are deprecated and their alternatives
- ✓ Thread coordination using join()
- ✓ Thread priority and its limitations
- ✓ Daemon threads and their use cases

**Remember:**
- Use proper synchronization mechanisms (wait/notify)
- Don't use deprecated methods (stop, suspend, resume)
- Don't rely on thread priority
- Use daemon threads for background services only
- Always handle InterruptedException properly

**Practice:** Implement the Producer-Consumer problem with multiple producers and consumers to solidify your understanding!

---

*Happy Threading! 🧵*
