# Complete Java Multithreading Guide - Part 2

## Table of Contents
1. [Introduction to Thread Creation](#introduction-to-thread-creation)
2. [Method 1: Creating Threads Using Runnable Interface](#method-1-creating-threads-using-runnable-interface)
3. [Method 2: Creating Threads by Extending Thread Class](#method-2-creating-threads-by-extending-thread-class)
4. [Deep Dive: Thread Lifecycle](#deep-dive-thread-lifecycle)
5. [Understanding Monitor Locks](#understanding-monitor-locks)
6. [Thread Synchronization in Detail](#thread-synchronization-in-detail)
7. [Inter-thread Communication](#inter-thread-communication)
8. [Complete Working Examples](#complete-working-examples)
9. [Practice Assignment](#practice-assignment)
10. [Common Pitfalls and Best Practices](#common-pitfalls-and-best-practices)

---

## Introduction to Thread Creation

### Why Two Different Ways to Create Threads?

Java provides **two ways** to create threads, and understanding WHY is crucial:

**The Java Inheritance Problem:**
```java
// In Java, you can only extend ONE class
class MyClass extends ParentClass {
    // Cannot extend Thread here because already extending ParentClass
}

// But you CAN implement multiple interfaces
class MyClass extends ParentClass implements Interface1, Interface2, Runnable {
    // This is allowed!
}
```

**Key Insight:** Java gives you both options so that:
1. If your class already extends another class → Use **Runnable interface**
2. If your class doesn't need to extend anything → You can use either method (but Runnable is still preferred)

### The Two Methods Are:
1. **Implementing Runnable Interface** (Recommended - Used in Production)
2. **Extending Thread Class** (Less flexible but good to know)

---

## Method 1: Creating Threads Using Runnable Interface

### Understanding the Architecture

Let's understand what exists in Java BEFORE we create anything:

```
┌─────────────────────────────┐
│  Runnable (Interface)       │  ← This is a Functional Interface
│  - run() method (abstract)  │  ← Has only ONE abstract method
└─────────────────────────────┘
              ↑
              │ implements
              │
┌─────────────────────────────┐
│  Thread Class               │  ← This actually creates/manages threads
│  - implements Runnable      │  ← Thread itself implements Runnable!
│  - run() method             │  ← Provides implementation of run()
│  - start() method           │  ← Starts the thread
│  - sleep() method           │  ← Makes thread sleep
│  - Many other methods...    │
└─────────────────────────────┘
```

**Important Understanding:**
- `Runnable` is NOT a thread - it's just an interface
- `Thread` is the actual class that creates and manages threads
- `Thread` class itself implements `Runnable` interface

### Step-by-Step Implementation

#### Step 1: Create a Class that Implements Runnable

```java
// This is YOUR custom class
class MultithreadingLearning implements Runnable {
    
    // You MUST implement the run() method (it's abstract in Runnable)
    @Override
    public void run() {
        // Whatever code you write here will be executed by the thread
        System.out.println("Code executed by thread: " + 
                          Thread.currentThread().getName());
    }
}
```

**What you've created:**
- A class that implements Runnable
- This is NOT a thread yet - it's just a normal class
- It has a `run()` method that defines WHAT the thread should do

#### Step 2: Create an Instance of Your Class

```java
// Create object of your Runnable class
MultithreadingLearning runnableObject = new MultithreadingLearning();
```

**What this is:**
- An object that implements Runnable
- Still NOT a thread - just an object with a run() method

#### Step 3: Pass Runnable Object to Thread Constructor

```java
// NOW we create the actual thread
Thread thread = new Thread(runnableObject);
```

**What happened here:**
- We created a REAL thread object
- We passed our Runnable object to the Thread constructor
- The thread now knows: "When I start, I should execute the run() method of this runnableObject"

#### Step 4: Start the Thread

```java
// Start the thread
thread.start();
```

**What happens when you call start():**
1. Thread moves from NEW state to RUNNABLE state
2. JVM's thread scheduler gives it CPU time
3. The `start()` method internally calls the `run()` method
4. Your code in `run()` gets executed

### Complete Example

```java
public class ThreadCreationExample {
    public static void main(String[] args) {
        System.out.println("Main thread: " + Thread.currentThread().getName());
        
        // Step 1 & 2: Create Runnable object
        MultithreadingLearning runnable = new MultithreadingLearning();
        
        // Step 3: Create Thread with Runnable
        Thread thread = new Thread(runnable);
        
        // Step 4: Start the thread
        thread.start();
        
        System.out.println("Main method finished");
    }
}

class MultithreadingLearning implements Runnable {
    @Override
    public void run() {
        System.out.println("New thread: " + Thread.currentThread().getName());
    }
}
```

**Output:**
```
Main thread: main
Main method finished
New thread: Thread-0
```

### The Magic: How start() Calls Your run() Method

Let's understand the internal mechanism:

```java
// This is inside the Thread class (simplified)
public class Thread implements Runnable {
    private Runnable target;  // Stores the Runnable object you passed
    
    // Constructor
    public Thread(Runnable target) {
        this.target = target;  // Saves your Runnable object
    }
    
    // When you call start(), it eventually calls run()
    @Override
    public void run() {
        if (target != null) {
            target.run();  // Calls YOUR run() method!
        }
        // If target is null, does nothing
    }
}
```

**Step-by-step execution:**

1. You create: `Thread thread = new Thread(runnableObject);`
   - Thread stores: `target = runnableObject`

2. You call: `thread.start();`
   - Internally calls: `thread.run()`

3. Inside `thread.run()`:
   - Checks: `if (target != null)` → TRUE
   - Calls: `target.run()` → YOUR run() method executes!

### Using Lambda Expression (Modern Java)

Since Runnable is a functional interface (only one abstract method), you can use lambda:

```java
public static void main(String[] args) {
    // Old way - creating class
    Thread thread1 = new Thread(new MultithreadingLearning());
    
    // Modern way - using lambda
    Thread thread2 = new Thread(() -> {
        System.out.println("Lambda thread: " + Thread.currentThread().getName());
    });
    
    thread1.start();
    thread2.start();
}
```

**Both do the exact same thing!**

### Why This Method is Preferred in Industry

```java
// Your class can do MANY things
class DatabaseHandler extends BaseHandler implements Runnable, Serializable {
    // Extends a parent class
    // Implements Runnable (for threading capability)
    // Implements Serializable (for saving objects)
    
    private Database db;
    
    // Regular methods
    public void connectToDatabase() { }
    public void saveData() { }
    
    // Thread method
    @Override
    public void run() {
        // Threaded task
    }
}
```

**Benefits:**
1. Your class can extend another class (DatabaseHandler extends BaseHandler)
2. Your class can implement multiple interfaces
3. Your class can have its own business logic
4. Yet it can STILL be executed by a thread!

---

## Method 2: Creating Threads by Extending Thread Class

### Understanding the Architecture

```
┌─────────────────────────────┐
│  Thread Class               │
│  - implements Runnable      │
│  - run() method             │
│  - start() method           │
└─────────────────────────────┘
              ↑
              │ extends
              │
┌─────────────────────────────┐
│  MultithreadingLearning     │  ← YOUR class (child of Thread)
│  - run() method (override)  │  ← You override the run() method
└─────────────────────────────┘
```

**Key Point:** Your class IS a Thread (it inherits all Thread capabilities)

### Step-by-Step Implementation

#### Step 1: Create a Class that Extends Thread

```java
class MultithreadingLearning extends Thread {
    
    // Override the run() method to define what thread should do
    @Override
    public void run() {
        System.out.println("Thread running: " + Thread.currentThread().getName());
    }
}
```

**What you've created:**
- A class that IS a Thread (it's a child of Thread class)
- It has all Thread capabilities (start, sleep, interrupt, etc.)

#### Step 2: Create Instance and Start

```java
public static void main(String[] args) {
    // Create object of your Thread class
    MultithreadingLearning thread = new MultithreadingLearning();
    
    // Start the thread directly (no need to create Thread object)
    thread.start();
}
```

**Why no need to create Thread object?**
- Your `MultithreadingLearning` class IS already a Thread!
- It inherited everything from Thread class

### Complete Example

```java
public class ThreadExtensionExample {
    public static void main(String[] args) {
        System.out.println("Main thread: " + Thread.currentThread().getName());
        
        // Create your custom thread object
        MultithreadingLearning myThread = new MultithreadingLearning();
        
        // Start it
        myThread.start();
        
        System.out.println("Main method finished");
    }
}

class MultithreadingLearning extends Thread {
    @Override
    public void run() {
        System.out.println("Custom thread: " + Thread.currentThread().getName());
    }
}
```

**Output:**
```
Main thread: main
Main method finished
Custom thread: Thread-0
```

### How start() Calls Your run() Method Here

```java
// When you call myThread.start():
// 1. start() is inherited from Thread class
// 2. start() internally calls run()
// 3. Which run() is called? YOUR overridden run() method!
// 4. Java polymorphism: child's method overrides parent's method
```

**Method Resolution:**
```
myThread.start() called
    ↓
Thread's start() method executes
    ↓
start() calls run()
    ↓
Which run()? The overridden one in MultithreadingLearning!
    ↓
Your custom code executes
```

### Why Override run()?

Let's see what happens if you DON'T override:

```java
// Thread class's run() method (simplified)
public void run() {
    if (target != null) {
        target.run();
    }
    // Otherwise, does NOTHING!
}
```

**If you don't override run():**
- Thread starts
- Calls parent's run() method
- target is null (you didn't pass any Runnable)
- Does nothing and terminates!

### Why This Method is Less Preferred

```java
// Problem: Can only extend ONE class
class DatabaseHandler extends Thread {
    // Cannot extend BaseHandler now!
    // Already extending Thread
}

// Better approach:
class DatabaseHandler extends BaseHandler implements Runnable {
    // Can extend BaseHandler
    // Can implement Runnable
    // Much more flexible!
}
```

---

## Deep Dive: Thread Lifecycle

### Visual Representation of Thread States

```
                    ┌──────────┐
                    │   NEW    │  ← Thread created but not started
                    └──────────┘
                         │
                         │ start()
                         ↓
    ┌──────────────────────────────────────┐
    │         RUNNABLE STATE               │
    │  ┌────────────┐      ┌────────────┐ │
    │  │  Runnable  │ ←──→ │  Running   │ │
    │  │ (waiting   │      │ (executing │ │
    │  │  for CPU)  │      │  on CPU)   │ │
    │  └────────────┘      └────────────┘ │
    └──────────────────────────────────────┘
           ↑                      ↓
           │                      │ run() completes
           │                      ↓
           │                 ┌──────────┐
           │                 │TERMINATED│
           │                 └──────────┘
           │
           │ IO complete / Lock acquired / notify() called / sleep time over
           │
    ┌──────────────────────────────────────┐
    │    BLOCKED / WAITING / TIMED_WAITING │
    │                                      │
    │  - BLOCKED: Waiting for lock        │
    │  - WAITING: wait() called           │
    │  - TIMED_WAITING: sleep() called    │
    └──────────────────────────────────────┘
```

### State 1: NEW

**When:** Thread object is created but `start()` not yet called

```java
Thread thread = new Thread(() -> System.out.println("Hello"));
// Thread is in NEW state here
// It's just an object in memory
// No execution has started
```

**Characteristics:**
- Thread object exists in memory
- No system resources allocated yet
- Not scheduled by thread scheduler

### State 2: RUNNABLE

**When:** After calling `start()` method

```java
thread.start();  // Thread moves to RUNNABLE state
```

**Two Sub-States (Important!):**

#### a) Ready (Runnable)
- Thread is ready to run
- Waiting in the queue for CPU time
- Thread scheduler decides when to give CPU

#### b) Running
- Thread got CPU time
- Currently executing instructions
- Can switch back to Ready due to context switching

**Context Switching Example:**
```
Time 0ms:  Thread1 (Running)  Thread2 (Ready)    Thread3 (Ready)
Time 5ms:  Thread1 (Ready)    Thread2 (Running)  Thread3 (Ready)
Time 10ms: Thread1 (Ready)    Thread2 (Ready)    Thread3 (Running)
```

**Important Note:**
- "Running" is NOT an official Java state
- Java considers both as "RUNNABLE"
- They continuously switch based on CPU scheduling

### State 3: BLOCKED

**When:** Thread is waiting to acquire a monitor lock

```java
class SharedResource {
    public synchronized void method1() {
        // Thread1 is here (has the lock)
        Thread.sleep(5000);
    }
    
    public synchronized void method2() {
        // Thread2 wants to enter but BLOCKED
        // Waiting for Thread1 to release lock
    }
}
```

**Other Blocking Scenarios:**
```java
// I/O Operations
FileInputStream fis = new FileInputStream("file.txt");
int data = fis.read();  // Thread BLOCKED until data is read

// Database Operations
ResultSet rs = statement.executeQuery("SELECT * FROM users");
// Thread BLOCKED until query completes
```

**Key Characteristic:**
- **Releases ALL monitor locks** when blocked
- Automatically returns to RUNNABLE when:
  - Lock is acquired
  - I/O operation completes

### State 4: WAITING

**When:** Explicitly calling `wait()` method

```java
synchronized(object) {
    object.wait();  // Thread goes to WAITING state
    // Will stay here FOREVER until notify() is called
}
```

**How to Return to RUNNABLE:**
```java
// In another thread
synchronized(object) {
    object.notify();      // Wakes up one waiting thread
    // OR
    object.notifyAll();   // Wakes up all waiting threads
}
```

**Key Characteristics:**
- **Releases ALL monitor locks**
- Waits indefinitely (no automatic wake up)
- Must be called from synchronized context

### State 5: TIMED_WAITING

**When:** Thread waits for specific time period

```java
// Method 1: sleep()
Thread.sleep(5000);  // Sleep for 5 seconds, then automatically wake up

// Method 2: wait() with timeout
synchronized(object) {
    object.wait(3000);  // Wait max 3 seconds
}

// Method 3: join() with timeout
thread.join(2000);  // Wait for thread to finish (max 2 seconds)
```

**Key Characteristics:**
- **Does NOT release monitor locks** (for sleep)
- Automatically returns to RUNNABLE after time expires
- No need for notify()

**Important Difference:**
```java
// WAITING - needs notify()
synchronized(obj) {
    obj.wait();  // Releases lock, needs notify()
}

// TIMED_WAITING - automatic wake up
Thread.sleep(1000);  // Does NOT release lock, auto wake up
```

### State 6: TERMINATED

**When:** Thread completes execution

```java
Thread thread = new Thread(() -> {
    System.out.println("Task completed");
});  // After this executes, thread is TERMINATED
```

**Characteristics:**
- Thread has finished execution
- Cannot be restarted
- Calling `start()` again throws `IllegalThreadStateException`

```java
thread.start();  // OK - moves to RUNNABLE
// ... thread completes ...
thread.start();  // ERROR! IllegalThreadStateException
```

### Complete Lifecycle Example

```java
public class ThreadLifecycleDemo {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            System.out.println("1. Thread is RUNNING");
            
            try {
                // TIMED_WAITING
                System.out.println("2. Going to TIMED_WAITING");
                Thread.sleep(2000);
                System.out.println("3. Back to RUNNABLE after sleep");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            
            System.out.println("4. Thread finishing - moving to TERMINATED");
        });
        
        System.out.println("Thread state: " + thread.getState());  // NEW
        
        thread.start();
        System.out.println("Thread state: " + thread.getState());  // RUNNABLE
        
        Thread.sleep(1000);
        System.out.println("Thread state: " + thread.getState());  // TIMED_WAITING
        
        Thread.sleep(2000);
        System.out.println("Thread state: " + thread.getState());  // TERMINATED
    }
}
```

### Monitor Lock Release Summary

| State | Releases Monitor Lock? |
|-------|----------------------|
| NEW | N/A (no lock acquired) |
| RUNNABLE | No (keeps lock if acquired) |
| BLOCKED | **YES** (releases all locks) |
| WAITING | **YES** (releases all locks) |
| TIMED_WAITING | **NO** (keeps lock - for sleep) |
| TERMINATED | N/A (thread finished) |

---

## Understanding Monitor Locks

### What is a Monitor Lock?

A monitor lock (also called intrinsic lock or mutex) is a mechanism that ensures **only ONE thread** can execute a synchronized block of code at a time for a **specific object**.

### The Fundamental Concept

```
Every Object in Java has ONE monitor lock

Object 1 → Has Monitor Lock 1
Object 2 → Has Monitor Lock 2
Object 3 → Has Monitor Lock 3
```

**Key Rule:** A thread must acquire the monitor lock before entering synchronized code.

### How Monitor Locks Work

#### Scenario 1: Same Object, Multiple Threads

```java
class Counter {
    private int count = 0;
    
    public synchronized void increment() {
        count++;
    }
}

// Main method
Counter counter = new Counter();  // ONE object

Thread t1 = new Thread(() -> counter.increment());  // Uses same object
Thread t2 = new Thread(() -> counter.increment());  // Uses same object
Thread t3 = new Thread(() -> counter.increment());  // Uses same object

t1.start();
t2.start();
t3.start();
```

**What Happens:**
```
1. Thread-1 reaches increment() first
   → Acquires monitor lock on 'counter' object
   → Enters method
   → Executes count++
   
2. Thread-2 reaches increment()
   → Tries to acquire lock on 'counter' object
   → Lock already held by Thread-1
   → Thread-2 goes to BLOCKED state
   → Waits...
   
3. Thread-3 reaches increment()
   → Tries to acquire lock on 'counter' object
   → Lock already held by Thread-1
   → Thread-3 goes to BLOCKED state
   → Waits...
   
4. Thread-1 finishes increment()
   → Releases monitor lock
   → Thread-2 or Thread-3 acquires lock (scheduler decides)
   → That thread enters method
   → Other thread continues waiting
```

#### Scenario 2: Different Objects, Multiple Threads

```java
Counter counter1 = new Counter();  // Object 1
Counter counter2 = new Counter();  // Object 2

Thread t1 = new Thread(() -> counter1.increment());  // Uses object 1
Thread t2 = new Thread(() -> counter2.increment());  // Uses object 2

t1.start();
t2.start();
```

**What Happens:**
```
1. Thread-1 reaches increment() on counter1
   → Acquires monitor lock on counter1 object
   → Enters method
   
2. Thread-2 reaches increment() on counter2
   → Acquires monitor lock on counter2 object (different object!)
   → Enters method immediately (no waiting!)
   
Both threads execute simultaneously because:
- counter1 has its own lock
- counter2 has its own lock
- They are independent!
```

### Synchronized Method vs Synchronized Block

#### Method-Level Synchronization

```java
public synchronized void method() {
    // Entire method is synchronized
    // Lock is on 'this' object
}

// Equivalent to:
public void method() {
    synchronized(this) {
        // Entire method code here
    }
}
```

#### Block-Level Synchronization

```java
public void method() {
    // Code here can run without lock
    System.out.println("Before synchronized");
    
    synchronized(this) {
        // Only this block requires lock
        System.out.println("Inside synchronized");
    }
    
    // Code here can run without lock
    System.out.println("After synchronized");
}
```

**Advantage of Block-Level:**
- Only critical section needs lock
- Better performance
- Other threads can execute non-critical parts

### Complete Monitor Lock Example

```java
class MonitorLockExample {
    
    // Method 1: Synchronized method
    public synchronized void task1() {
        System.out.println(Thread.currentThread().getName() + 
                          " - Inside Task 1");
        try {
            Thread.sleep(10000);  // Hold lock for 10 seconds
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + 
                          " - Task 1 Completed");
    }
    
    // Method 2: Synchronized block
    public void task2() {
        System.out.println(Thread.currentThread().getName() + 
                          " - Task 2: Before synchronized");
        
        synchronized(this) {
            System.out.println(Thread.currentThread().getName() + 
                              " - Task 2: Inside synchronized");
        }
        
        System.out.println(Thread.currentThread().getName() + 
                          " - Task 2: After synchronized");
    }
    
    // Method 3: No synchronization
    public void task3() {
        System.out.println(Thread.currentThread().getName() + 
                          " - Task 3: No synchronization");
    }
}

public class MonitorLockDemo {
    public static void main(String[] args) {
        MonitorLockExample obj = new MonitorLockExample();
        
        // All threads work on SAME object
        Thread t1 = new Thread(() -> obj.task1(), "Thread-1");
        Thread t2 = new Thread(() -> obj.task2(), "Thread-2");
        Thread t3 = new Thread(() -> obj.task3(), "Thread-3");
        
        t1.start();  // Starts first
        t2.start();  // Starts immediately after
        t3.start();  // Starts immediately after
    }
}
```

**Expected Output:**
```
Thread-1 - Inside Task 1
Thread-2 - Task 2: Before synchronized
Thread-3 - Task 3: No synchronization
[10 seconds pass...]
Thread-1 - Task 1 Completed
Thread-2 - Task 2: Inside synchronized
Thread-2 - Task 2: After synchronized
```

**Step-by-Step Execution:**

```
Time 0ms:
- Thread-1 starts, calls task1()
- Acquires monitor lock on 'obj'
- Prints "Inside Task 1"
- Starts sleeping (holds lock for 10 seconds)

Time 5ms:
- Thread-2 starts, calls task2()
- Prints "Before synchronized" (no lock needed yet)
- Reaches synchronized(this) block
- Tries to acquire lock on 'obj'
- Lock is held by Thread-1
- Thread-2 BLOCKS (waits)

Time 10ms:
- Thread-3 starts, calls task3()
- No synchronization needed
- Prints "No synchronization"
- Completes immediately

Time 10000ms (10 seconds later):
- Thread-1 wakes up from sleep
- Completes task1()
- Releases monitor lock on 'obj'
- Thread-2 acquires lock
- Prints "Inside synchronized"
- Releases lock
- Prints "After synchronized"
```

### Advanced Monitor Lock Concepts

#### Static Synchronized Methods

```java
class MyClass {
    public static synchronized void staticMethod() {
        // Lock is on MyClass.class object
        // Not on instance
    }
}
```

**Key Difference:**
- Regular synchronized: Lock on instance (`this`)
- Static synchronized: Lock on class object (`MyClass.class`)

```java
MyClass obj1 = new MyClass();
MyClass obj2 = new MyClass();

Thread t1 = new Thread(() -> MyClass.staticMethod());
Thread t2 = new Thread(() -> MyClass.staticMethod());

// Both threads compete for SAME lock (class lock)
// Even though no object is involved!
```

#### Synchronized on Different Objects

```java
class MonitorExample {
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();
    
    public void method1() {
        synchronized(lock1) {
            // Protected by lock1
        }
    }
    
    public void method2() {
        synchronized(lock2) {
            // Protected by lock2 (different lock!)
        }
    }
}

// Two threads can execute method1 and method2 simultaneously
// Because they use different locks!
```

### Monitor Lock Best Practices

1. **Lock on appropriate object**
   ```java
   // Bad - locks entire object
   public synchronized void updateName() {
       name = newName;
   }
   
   // Good - locks only when needed
   private final Object nameLock = new Object();
   public void updateName() {
       synchronized(nameLock) {
           name = newName;
       }
   }
   ```

2. **Minimize synchronized scope**
   ```java
   // Bad - entire method locked
   public synchronized void process() {
       readFromFile();      // IO operation
       processData();       // CPU operation
       writeToDatabase();   // IO operation
   }
   
   // Good - only critical section locked
   public void process() {
       readFromFile();
       synchronized(this) {
           processData();   // Only this needs lock
       }
       writeToDatabase();
   }
   ```

3. **Avoid nested locks (can cause deadlock)**
   ```java
   // Dangerous!
   synchronized(obj1) {
       synchronized(obj2) {
           // If another thread locks obj2 then obj1
           // DEADLOCK!
       }
   }
   ```

---

## Thread Synchronization in Detail

### Why Synchronization is Needed

Consider this scenario without synchronization:

```java
class BankAccount {
    private int balance = 1000;
    
    // NO synchronization
    public void withdraw(int amount) {
        if (balance >= amount) {
            // Context switch can happen here!
            System.out.println(Thread.currentThread().getName() + 
                              " is withdrawing " + amount);
            balance = balance - amount;
            System.out.println("New balance: " + balance);
        }
    }
}

// Two threads trying to withdraw
Thread t1 = new Thread(() -> account.withdraw(600), "Person-1");
Thread t2 = new Thread(() -> account.withdraw(600), "Person-2");
```

**What Can Go Wrong:**
```
Initial balance: 1000

Thread-1 checks: balance (1000) >= 600? YES
Thread-2 checks: balance (1000) >= 600? YES  [context switch before Thread-1 subtracts]

Thread-1 withdraws: balance = 1000 - 600 = 400
Thread-2 withdraws: balance = 400 - 600 = -200  [PROBLEM!]

Final balance: -200 (should have been 400)
```

This is called a **race condition**.

### The synchronized Keyword

#### Syntax 1: Method-Level

```java
public synchronized void method() {
    // Thread-safe code
}
```

**What it does:**
- Acquires lock on `this` object
- Only one thread can execute this method at a time (for same object)
- Releases lock when method completes

#### Syntax 2: Block-Level

```java
public void method() {
    synchronized(lockObject) {
        // Thread-safe code
    }
}
```

**What it does:**
- Acquires lock on `lockObject`
- Only one thread can execute this block at a time (for same lockObject)
- Releases lock when block completes

### Fixed Bank Account Example

```java
class BankAccount {
    private int balance = 1000;
    
    // WITH synchronization
    public synchronized void withdraw(int amount) {
        if (balance >= amount) {
            System.out.println(Thread.currentThread().getName() + 
                              " is withdrawing " + amount);
            balance = balance - amount;
            System.out.println(Thread.currentThread().getName() + 
                              " - New balance: " + balance);
        } else {
            System.out.println(Thread.currentThread().getName() + 
                              " - Insufficient balance");
        }
    }
}

public class SynchronizationDemo {
    public static void main(String[] args) {
        BankAccount account = new BankAccount();
        
        Thread t1 = new Thread(() -> account.withdraw(600), "Person-1");
        Thread t2 = new Thread(() -> account.withdraw(600), "Person-2");
        
        t1.start();
        t2.start();
    }
}
```

**Output:**
```
Person-1 is withdrawing 600
Person-1 - New balance: 400
Person-2 - Insufficient balance
```

**What Happened:**
```
1. Person-1 acquires lock
2. Person-1 checks balance: 1000 >= 600? YES
3. Person-1 withdraws 600
4. Person-1 releases lock (balance now 400)
5. Person-2 acquires lock
6. Person-2 checks balance: 400 >= 600? NO
7. Person-2 cannot withdraw
8. Person-2 releases lock
```

### Choosing Between Method and Block Synchronization

#### Use Method-Level When:
```java
public synchronized void criticalMethod() {
    // Entire method needs protection
    updateSharedData();
    validateData();
    saveData();
}
```

#### Use Block-Level When:
```java
public void optimizedMethod() {
    // Non-critical code (no lock needed)
    String data = prepareData();
    
    // Only critical section needs lock
    synchronized(this) {
        sharedList.add(data);
    }
    
    // Non-critical code (no lock needed)
    logOperation();
}
```

**Performance Impact:**
```java
// Slow - holds lock during I/O
public synchronized void slowMethod() {
    readFromFile();        // 100ms with lock
    updateSharedData();    // 1ms with lock
    writeToDatabase();     // 200ms with lock
}
// Total lock time: 301ms

// Fast - locks only critical section
public void fastMethod() {
    readFromFile();        // 100ms without lock
    synchronized(this) {
        updateSharedData(); // 1ms with lock
    }
    writeToDatabase();     // 200ms without lock
}
// Total lock time: 1ms
```

---

## Inter-thread Communication

### The Problem: Coordination Between Threads

Consider two threads:
- **Producer**: Creates data
- **Consumer**: Processes data

**Challenge:** Consumer should wait if no data available, Producer should wait if buffer is full.

### The wait() and notify() Methods

#### wait() Method

```java
synchronized(object) {
    object.wait();  // Current thread releases lock and waits
}
```

**What happens:**
1. Current thread releases ALL monitor locks
2. Thread goes to WAITING state
3. Thread stays there until `notify()` or `notifyAll()` is called
4. When notified, thread re-acquires lock and continues

**Important Rules:**
- MUST be called from synchronized context
- Throws `InterruptedException`
- Releases monitor locks (unlike sleep)

#### notify() Method

```java
synchronized(object) {
    object.notify();  // Wakes up ONE waiting thread
}
```

**What happens:**
1. Wakes up one thread that's waiting on this object
2. If multiple threads are waiting, which one wakes up is arbitrary
3. Woken thread moves to RUNNABLE state
4. Woken thread must re-acquire lock before continuing

#### notifyAll() Method

```java
synchronized(object) {
    object.notifyAll();  // Wakes up ALL waiting threads
}
```

**What happens:**
1. Wakes up ALL threads waiting on this object
2. All woken threads compete to acquire lock
3. Only one gets the lock, others wait again

### Why Use while Loop Instead of if?

```java
// WRONG - Using if
public synchronized void consume() {
    if (!itemAvailable) {
        wait();  // What if spurious wakeup?
    }
    // Might proceed even if condition false!
    consumeItem();
}

// CORRECT - Using while
public synchronized void consume() {
    while (!itemAvailable) {
        wait();  // Re-checks after waking up
    }
    // Guaranteed condition is true
    consumeItem();
}
```

**Spurious Wakeup:** A thread can wake up from `wait()` without anyone calling `notify()` (due to system-level events). Using `while` ensures the condition is re-checked.

### Complete Producer-Consumer Example

```java
// Shared resource between producer and consumer
class SharedResource {
    private boolean itemAvailable = false;
    
    // Producer calls this
    public synchronized void addItem() {
        System.out.println(Thread.currentThread().getName() + 
                          " - Adding item");
        itemAvailable = true;
        
        // Notify waiting consumers
        notifyAll();
        
        System.out.println(Thread.currentThread().getName() + 
                          " - Item added, notified consumers");
    }
    
    // Consumer calls this
    public synchronized void consumeItem() {
        System.out.println(Thread.currentThread().getName() + 
                          " - Trying to consume");
        
        // Wait while item not available
        while (!itemAvailable) {
            try {
                System.out.println(Thread.currentThread().getName() + 
                                  " - No item available, waiting...");
                wait();  // Releases lock and waits
                
                System.out.println(Thread.currentThread().getName() + 
                                  " - Woke up, re-checking condition");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        
        // Item is available, consume it
        System.out.println(Thread.currentThread().getName() + 
                          " - Consuming item");
        itemAvailable = false;
    }
}

public class ProducerConsumerExample {
    public static void main(String[] args) {
        SharedResource resource = new SharedResource();
        
        // Producer thread (waits 3 seconds before producing)
        Thread producer = new Thread(() -> {
            try {
                Thread.sleep(3000);  // Delay to ensure consumer waits
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            resource.addItem();
        }, "Producer");
        
        // Consumer thread (tries to consume immediately)
        Thread consumer = new Thread(() -> {
            resource.consumeItem();
        }, "Consumer");
        
        consumer.start();  // Start consumer first
        producer.start();  // Start producer after
    }
}
```

**Expected Output:**
```
Consumer - Trying to consume
Consumer - No item available, waiting...
[3 seconds pass]
Producer - Adding item
Producer - Item added, notified consumers
Consumer - Woke up, re-checking condition
Consumer - Consuming item
```

**Detailed Execution Flow:**

```
Time 0ms:
├─ Consumer thread starts
├─ Calls consumeItem()
├─ Acquires lock on 'resource'
├─ Checks: itemAvailable? NO
├─ Prints "No item available, waiting..."
├─ Calls wait()
│  ├─ Releases lock on 'resource'
│  └─ Goes to WAITING state
└─ [Consumer is waiting]

Time 100ms:
├─ Producer thread starts
├─ Sleeps for 3000ms (TIMED_WAITING)
└─ [Producer is sleeping]

Time 3100ms:
├─ Producer wakes up from sleep
├─ Calls addItem()
├─ Acquires lock on 'resource' (available because Consumer released it)
├─ Prints "Adding item"
├─ Sets itemAvailable = true
├─ Calls notifyAll()
│  └─ Wakes up Consumer (moves to RUNNABLE)
├─ Prints "Item added, notified consumers"
├─ Exits addItem()
└─ Releases lock on 'resource'

Time 3101ms:
├─ Consumer (now RUNNABLE) tries to re-acquire lock
├─ Acquires lock on 'resource'
├─ Wakes up from wait()
├─ Prints "Woke up, re-checking condition"
├─ Checks: itemAvailable? YES (while loop exits)
├─ Prints "Consuming item"
├─ Sets itemAvailable = false
├─ Exits consumeItem()
└─ Releases lock on 'resource'
```

### wait() vs sleep() - Critical Differences

| Feature | wait() | sleep() |
|---------|--------|---------|
| **Class** | Object class | Thread class |
| **Synchronized Required** | YES (must be in synchronized block) | NO |
| **Releases Lock** | YES (releases all monitor locks) | NO (keeps locks) |
| **Wakeup** | notify() or notifyAll() needed | Automatic after time |
| **Usage** | Inter-thread communication | Pause execution |

```java
// wait() example
synchronized(obj) {
    obj.wait();  // Releases lock on obj
}

// sleep() example
Thread.sleep(1000);  // Does NOT release any locks
```

### Real-World Example: Print Server

```java
class PrintServer {
    private boolean printerAvailable = true;
    
    public synchronized void print(String document) {
        // Wait while printer is busy
        while (!printerAvailable) {
            try {
                System.out.println(Thread.currentThread().getName() + 
                                  " - Printer busy, waiting...");
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        
        // Printer available, use it
        printerAvailable = false;
        System.out.println(Thread.currentThread().getName() + 
                          " - Printing: " + document);
        
        // Simulate printing time
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
        // Printing done
        System.out.println(Thread.currentThread().getName() + 
                          " - Printing complete");
        printerAvailable = true;
        
        // Notify waiting threads
        notifyAll();
    }
}

public class PrintServerDemo {
    public static void main(String[] args) {
        PrintServer server = new PrintServer();
        
        Thread user1 = new Thread(() -> server.print("Document1"), "User-1");
        Thread user2 = new Thread(() -> server.print("Document2"), "User-2");
        Thread user3 = new Thread(() -> server.print("Document3"), "User-3");
        
        user1.start();
        user2.start();
        user3.start();
    }
}
```

**Output:**
```
User-1 - Printing: Document1
User-2 - Printer busy, waiting...
User-3 - Printer busy, waiting...
[2 seconds]
User-1 - Printing complete
User-2 - Printing: Document2
[2 seconds]
User-2 - Printing complete
User-3 - Printing: Document3
[2 seconds]
User-3 - Printing complete
```

---

## Complete Working Examples

### Example 1: Thread Creation Comparison

```java
public class ThreadCreationComparison {
    
    public static void main(String[] args) {
        System.out.println("=== Method 1: Runnable Interface ===");
        
        // Creating thread using Runnable
        Runnable task1 = new MyRunnableTask();
        Thread thread1 = new Thread(task1);
        thread1.setName("Runnable-Thread");
        thread1.start();
        
        // Creating thread using lambda (also Runnable)
        Thread thread2 = new Thread(() -> {
            System.out.println("Lambda thread: " + 
                              Thread.currentThread().getName());
        }, "Lambda-Thread");
        thread2.start();
        
        System.out.println("\n=== Method 2: Extending Thread ===");
        
        // Creating thread by extending Thread
        Thread thread3 = new MyThreadTask();
        thread3.setName("Extended-Thread");
        thread3.start();
        
        System.out.println("\nMain thread: " + 
                          Thread.currentThread().getName());
    }
}

class MyRunnableTask implements Runnable {
    @Override
    public void run() {
        System.out.println("Runnable task executed by: " + 
                          Thread.currentThread().getName());
    }
}

class MyThreadTask extends Thread {
    @Override
    public void run() {
        System.out.println("Thread task executed by: " + 
                          Thread.currentThread().getName());
    }
}
```

### Example 2: Thread Lifecycle Demonstration

```java
public class ThreadLifecycleDemo {
    
    public static void main(String[] args) throws InterruptedException {
        
        Thread thread = new Thread(new LifecycleTask(), "Demo-Thread");
        
        // NEW state
        System.out.println("State after creation: " + thread.getState());
        
        // Start thread - moves to RUNNABLE
        thread.start();
        System.out.println("State after start(): " + thread.getState());
        
        // Give thread time to execute
        Thread.sleep(500);
        System.out.println("State while sleeping: " + thread.getState());
        
        // Wait for thread to complete
        thread.join();
        System.out.println("State after completion: " + thread.getState());
    }
}

class LifecycleTask implements Runnable {
    @Override
    public void run() {
        System.out.println("Thread started execution");
        
        try {
            // TIMED_WAITING state
            System.out.println("Going to sleep for 2 seconds");
            Thread.sleep(2000);
            System.out.println("Woke up from sleep");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
        System.out.println("Thread finishing execution");
    }
}
```

**Output:**
```
State after creation: NEW
State after start(): RUNNABLE
Thread started execution
Going to sleep for 2 seconds
State while sleeping: TIMED_WAITING
Woke up from sleep
Thread finishing execution
State after completion: TERMINATED
```

### Example 3: Synchronized Counter

```java
public class SynchronizedCounterDemo {
    
    public static void main(String[] args) throws InterruptedException {
        Counter counter = new Counter();
        
        // Create 5 threads, each incrementing 1000 times
        Thread[] threads = new Thread[5];
        for (int i = 0; i < 5; i++) {
            threads[i] = new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    counter.increment();
                }
            }, "Thread-" + i);
            threads[i].start();
        }
        
        // Wait for all threads to complete
        for (Thread thread : threads) {
            thread.join();
        }
        
        System.out.println("Final count: " + counter.getCount());
        System.out.println("Expected: 5000");
    }
}

class Counter {
    private int count = 0;
    
    // Synchronized method ensures thread safety
    public synchronized void increment() {
        count++;
    }
    
    public int getCount() {
        return count;
    }
}
```

### Example 4: Producer-Consumer with Queue

```java
import java.util.LinkedList;
import java.util.Queue;

public class ProducerConsumerQueue {
    
    public static void main(String[] args) {
        SharedQueue queue = new SharedQueue(5);  // Max capacity 5
        
        // Producer thread
        Thread producer = new Thread(() -> {
            for (int i = 1; i <= 10; i++) {
                try {
                    queue.produce(i);
                    Thread.sleep(500);  // Produce every 500ms
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "Producer");
        
        // Consumer thread
        Thread consumer = new Thread(() -> {
            for (int i = 1; i <= 10; i++) {
                try {
                    queue.consume();
                    Thread.sleep(1000);  // Consume every 1000ms
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "Consumer");
        
        producer.start();
        consumer.start();
    }
}

class SharedQueue {
    private Queue<Integer> queue = new LinkedList<>();
    private int capacity;
    
    public SharedQueue(int capacity) {
        this.capacity = capacity;
    }
    
    public synchronized void produce(int item) throws InterruptedException {
        // Wait while queue is full
        while (queue.size() == capacity) {
            System.out.println(Thread.currentThread().getName() + 
                              " - Queue full, waiting...");
            wait();
        }
        
        // Add item to queue
        queue.add(item);
        System.out.println(Thread.currentThread().getName() + 
                          " - Produced: " + item + 
                          " (Queue size: " + queue.size() + ")");
        
        // Notify consumers
        notifyAll();
    }
    
    public synchronized void consume() throws InterruptedException {
        // Wait while queue is empty
        while (queue.isEmpty()) {
            System.out.println(Thread.currentThread().getName() + 
                              " - Queue empty, waiting...");
            wait();
        }
        
        // Remove item from queue
        int item = queue.poll();
        System.out.println(Thread.currentThread().getName() + 
                          " - Consumed: " + item + 
                          " (Queue size: " + queue.size() + ")");
        
        // Notify producers
        notifyAll();
    }
}
```

---

## Practice Assignment

### Producer-Consumer Problem

**Problem Description:**

Implement a Producer-Consumer problem with the following specifications:

1. **Shared Buffer**: A fixed-size queue (capacity = 10)
2. **Producer Thread**: 
   - Generates integers (1, 2, 3, ...)
   - Adds them to the queue
   - Waits if queue is full
3. **Consumer Thread**:
   - Removes integers from the queue
   - Processes them (print)
   - Waits if queue is empty

**Requirements:**

```java
// Your implementation should handle:
1. Thread safety (use synchronized)
2. Proper waiting (use wait() when buffer full/empty)
3. Proper notification (use notifyAll() after produce/consume)
4. Use while loop (not if) with wait()
5. Handle InterruptedException
```

**Starter Template:**

```java
import java.util.LinkedList;
import java.util.Queue;

public class ProducerConsumerAssignment {
    
    public static void main(String[] args) {
        // TODO: Create SharedBuffer with capacity 10
        // TODO: Create Producer thread that produces 20 items
        // TODO: Create Consumer thread that consumes 20 items
        // TODO: Start both threads
    }
}

class SharedBuffer {
    private Queue<Integer> buffer;
    private int capacity;
    
    public SharedBuffer(int capacity) {
        this.buffer = new LinkedList<>();
        this.capacity = capacity;
    }
    
    public synchronized void produce(int item) throws InterruptedException {
        // TODO: Implement producer logic
        // 1. Wait while buffer is full
        // 2. Add item to buffer
        // 3. Print production message
        // 4. Notify waiting consumers
    }
    
    public synchronized int consume() throws InterruptedException {
        // TODO: Implement consumer logic
        // 1. Wait while buffer is empty
        // 2. Remove item from buffer
        // 3. Print consumption message
        // 4. Notify waiting producers
        // 5. Return consumed item
        return 0;  // Replace with actual item
    }
}
```

**Expected Output Pattern:**
```
Producer - Produced: 1 (Buffer size: 1)
Producer - Produced: 2 (Buffer size: 2)
Consumer - Consumed: 1 (Buffer size: 1)
Producer - Produced: 3 (Buffer size: 2)
Consumer - Consumed: 2 (Buffer size: 1)
...
Producer - Buffer full, waiting...
Consumer - Consumed: 10 (Buffer size: 9)
Producer - Produced: 11 (Buffer size: 10)
...
```

**Bonus Challenges:**

1. Add multiple producers and consumers
2. Add production/consumption delays using `Thread.sleep()`
3. Track total items produced and consumed
4. Implement graceful shutdown mechanism

**Test Cases to Verify:**

1. Producer waits when buffer is full
2. Consumer waits when buffer is empty
3. No race conditions (count should be accurate)
4. No deadlocks (program should complete)

---

## Common Pitfalls and Best Practices

### Pitfall 1: Calling run() Instead of start()

```java
// WRONG - Directly calling run()
Thread thread = new Thread(() -> System.out.println("Hello"));
thread.run();  // Executes in SAME thread (main), not new thread!

// CORRECT - Calling start()
Thread thread = new Thread(() -> System.out.println("Hello"));
thread.start();  // Creates NEW thread and executes
```

**Why it's wrong:**
- `run()` is just a normal method call
- Executes in the current thread (main)
- No new thread is created
- No parallelism

### Pitfall 2: Using if Instead of while with wait()

```java
// WRONG - Using if
public synchronized void consume() {
    if (queue.isEmpty()) {
        wait();  // Spurious wakeup might skip check
    }
    processItem(queue.poll());
}

// CORRECT - Using while
public synchronized void consume() {
    while (queue.isEmpty()) {
        wait();  // Re-checks condition after wakeup
    }
    processItem(queue.poll());
}
```

### Pitfall 3: Not Handling InterruptedException

```java
// WRONG - Swallowing exception
try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    // Empty catch block!
}

// CORRECT - Proper handling
try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    // Restore interrupt status
    Thread.currentThread().interrupt();
    // Or handle appropriately
    System.err.println("Thread interrupted: " + e.getMessage());
}
```

### Pitfall 4: Calling wait() Outside Synchronized Block

```java
// WRONG - Not synchronized
public void consume() {
    wait();  // IllegalMonitorStateException!
}

// CORRECT - Inside synchronized
public synchronized void consume() {
    wait();  // Works fine
}

// Also CORRECT - Synchronized block
public void consume() {
    synchronized(this) {
        wait();  // Works fine
    }
}
```

### Pitfall 5: Starting Same Thread Twice

```java
// WRONG
Thread thread = new Thread(() -> System.out.println("Hello"));
thread.start();
thread.start();  // IllegalThreadStateException!

// CORRECT - Create new thread
Thread thread1 = new Thread(() -> System.out.println("Hello"));
Thread thread2 = new Thread(() -> System.out.println("World"));
thread1.start();
thread2.start();
```

### Best Practice 1: Use Meaningful Thread Names

```java
// Poor naming
Thread t1 = new Thread(task);
t1.start();

// Better naming
Thread userRegistrationThread = new Thread(task, "User-Registration-Thread");
userRegistrationThread.start();

// Even better - helps in debugging
Thread thread = new Thread(task);
thread.setName("PaymentProcessor-" + transactionId);
thread.start();
```

### Best Practice 2: Prefer Runnable Over Thread Extension

```java
// Less flexible
class MyTask extends Thread {
    public void run() {
        // Task logic
    }
}

// More flexible
class MyTask implements Runnable {
    public void run() {
        // Task logic
    }
}

// Best - Can extend other classes too
class MyTask extends BaseTask implements Runnable, Serializable {
    public void run() {
        // Task logic
    }
}
```

### Best Practice 3: Minimize Synchronized Scope

```java
// Poor - Entire method locked
public synchronized void process() {
    String data = readFromFile();     // 100ms
    processData(data);                 // 1ms
    writeToDatabase(data);             // 200ms
}
// Lock held for 301ms!

// Better - Only critical section locked
public void process() {
    String data = readFromFile();      // 100ms (no lock)
    
    synchronized(this) {
        processData(data);              // 1ms (locked)
    }
    
    writeToDatabase(data);              // 200ms (no lock)
}
// Lock held for 1ms!
```

### Best Practice 4: Use try-finally for Resource Cleanup

```java
// Ensures cleanup even if exception occurs
Lock lock = new ReentrantLock();
lock.lock();
try {
    // Critical section
} finally {
    lock.unlock();  // Always executed
}
```

### Best Practice 5: Document Thread Safety

```java
/**
 * Thread-safe counter implementation.
 * All methods are synchronized to ensure thread safety.
 */
public class Counter {
    private int count = 0;
    
    /**
     * Increments counter by 1.
     * Thread-safe method.
     */
    public synchronized void increment() {
        count++;
    }
}
```

---

## Quick Reference Tables

### Thread States Summary

| State | Description | Entry Point | Exit Point |
|-------|-------------|-------------|------------|
| NEW | Thread created, not started | `new Thread()` | `start()` |
| RUNNABLE | Ready to run or running | `start()` | `run()` completes |
| BLOCKED | Waiting for monitor lock | Lock unavailable | Lock acquired |
| WAITING | Waiting indefinitely | `wait()` | `notify()`/`notifyAll()` |
| TIMED_WAITING | Waiting for specific time | `sleep()`/`wait(timeout)` | Time expires |
| TERMINATED | Execution completed | `run()` completes | Cannot restart |

### Thread Methods Comparison

| Method | Class | Releases Lock? | Needs Synchronized? | Wakeup Method |
|--------|-------|----------------|---------------------|---------------|
| `sleep()` | Thread | No | No | Automatic (time) |
| `wait()` | Object | Yes | Yes | `notify()`/`notifyAll()` |
| `notify()` | Object | N/A | Yes | N/A |
| `notifyAll()` | Object | N/A | Yes | N/A |
| `join()` | Thread | No | No | Thread completes |
| `yield()` | Thread | No | No | Thread scheduler |

### Common Exceptions

| Exception | Cause | Solution |
|-----------|-------|----------|
| `IllegalThreadStateException` | Calling `start()` twice | Create new thread |
| `IllegalMonitorStateException` | `wait()`/`notify()` without lock | Use synchronized block |
| `InterruptedException` | Thread interrupted while waiting | Handle or propagate |

---

## Deprecated Methods and Alternatives

### stop() Method (Deprecated)

```java
// DEPRECATED - Don't use
thread.stop();  // Unsafe!

// ALTERNATIVE - Use flag
class SafeTask implements Runnable {
    private volatile boolean running = true;
    
    public void run() {
        while (running) {
            // Do work
        }
    }
    
    public void stopTask() {
        running = false;
    }
}
```

### suspend() and resume() Methods (Deprecated)

```java
// DEPRECATED - Don't use
thread.suspend();  // Unsafe!
thread.resume();   // Unsafe!

// ALTERNATIVE - Use wait() and notify()
public synchronized void pauseTask() {
    while (paused) {
        wait();
    }
}

public synchronized void resumeTask() {
    paused = false;
    notifyAll();
}
```

---

## Summary

### Key Takeaways

1. **Thread Creation**
   - Runnable interface (preferred - more flexible)
   - Extending Thread class (less flexible)
   - Lambda expressions for concise code

2. **Thread Lifecycle**
   - NEW → RUNNABLE → TERMINATED (main flow)
   - BLOCKED, WAITING, TIMED_WAITING (intermediate states)
   - Understanding state transitions is crucial

3. **Monitor Locks**
   - Every object has one monitor lock
   - Only one thread can hold lock at a time
   - Critical for thread synchronization

4. **Synchronization**
   - Prevents race conditions
   - Can be method-level or block-level
   - Always minimize synchronized scope

5. **Inter-thread Communication**
   - `wait()` - releases lock, waits for notification
   - `notify()`/`notifyAll()` - wakes up waiting threads
   - Always use `while` loop with `wait()`

6. **Best Practices**
   - Prefer Runnable over Thread extension
   - Use meaningful thread names
   - Handle exceptions properly
   - Document thread safety
   - Minimize synchronized scope

### Next Steps

1. Complete the Producer-Consumer assignment
2. Practice with multiple producers and consumers
3. Study advanced topics: ThreadPools, ExecutorService, Concurrent Collections
4. Learn about java.util.concurrent package
5. Understand Lock interface and its implementations

---

## Additional Resources

### Useful Thread Methods

```java
// Get current thread
Thread currentThread = Thread.currentThread();

// Get thread name
String name = thread.getName();

// Set thread name
thread.setName("MyThread");

// Check if thread is alive
boolean alive = thread.isAlive();

// Wait for thread to die
thread.join();

// Get thread priority
int priority = thread.getPriority();

// Set thread priority (1-10, default 5)
thread.setPriority(Thread.MAX_PRIORITY);

// Check if thread is interrupted
boolean interrupted = thread.isInterrupted();

// Interrupt thread
thread.interrupt();
```

### Debugging Tips

```java
// Print thread information
System.out.println("Thread: " + Thread.currentThread().getName());
System.out.println("State: " + thread.getState());
System.out.println("Priority: " + thread.getPriority());
System.out.println("Is Alive: " + thread.isAlive());
System.out.println("Is Daemon: " + thread.isDaemon());

// Stack trace for debugging
Thread.dumpStack();

// Get all stack traces
Map<Thread, StackTraceElement[]> traces = Thread.getAllStackTraces();
```

---

*Remember: Multithreading is powerful but complex. Practice with simple examples before moving to complex scenarios. Always test your code thoroughly for race conditions and deadlocks!*
