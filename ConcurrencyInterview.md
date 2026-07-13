# UBS Concurrency & Multithreading Interview — 22 Complete Answers

**Prep target: July 17 technical interview**
Same delivery format as the OOP set — talk-track length answers, banking-anchored, with code where you'll likely be asked to whiteboard it.

---

## Category 1: Thread Lifecycle & Core Synchronization

### 1. Process vs Thread — OS memory allocation and context-switching overhead
A **Process** is an independent execution unit with its **own isolated memory space** (its own heap, code, data segments) — the OS allocates it a completely separate address space. A **Thread** is a lightweight unit of execution *within* a process; all threads in a process **share the same heap and code segment**, but each thread gets its own **stack**, program counter, and register set.
- **Context-switching**: switching between processes is expensive — the OS must swap the entire memory address space (flush/reload the MMU's page tables, CPU cache invalidation). Switching between threads of the *same* process is much cheaper since the address space stays the same; only the stack pointer, registers, and program counter change.
- This is exactly why a multi-threaded trading engine (many threads, one process, shared heap) is far more efficient than spawning a new process per request.

### 2. Java Thread Lifecycle states
- **`NEW`**: thread object created (`new Thread(...)`) but `start()` not yet called.
- **`RUNNABLE`**: after `start()` — the thread is either actually running on a CPU core or ready and waiting for the OS scheduler to give it CPU time (Java doesn't distinguish these two as separate states).
- **`BLOCKED`**: the thread is waiting to acquire an **intrinsic lock** (entering a `synchronized` block/method) that another thread currently holds.
- **`WAITING`**: the thread is waiting **indefinitely** for another thread's action — e.g., called `Object.wait()` with no timeout, `Thread.join()`, or `LockSupport.park()`.
- **`TIMED_WAITING`**: same as `WAITING` but bounded by a timeout — e.g., `Thread.sleep(1000)`, `wait(timeout)`, `join(timeout)`.
- **`TERMINATED`**: the thread has completed execution (normally or via an uncaught exception) and cannot be restarted.

### 3. `thread.start()` vs `thread.run()`
`start()` asks the **JVM/OS to allocate a new call stack and actually spawn a new thread of execution**, which then invokes `run()` on that new thread. Calling `run()` directly is just a **normal method call** on the current thread — no new thread is created, so the code executes sequentially on whatever thread called it, with zero concurrency. This is a classic trap: `myThread.run()` compiles fine and "works" in the sense that the logic executes, but it silently defeats the entire purpose of using threads — everything runs on the caller's thread, one line at a time.

### 4. Intrinsic lock (monitor lock) and `synchronized`
Every Java object has an associated **intrinsic lock** (also called a monitor), even though most objects never use it. When a thread enters a `synchronized(obj) { ... }` block, it must first **acquire `obj`'s monitor lock** — if another thread already holds it, the requesting thread blocks until the lock is released. Only one thread can hold a given object's monitor at a time, which enforces **mutual exclusion**: two threads can never execute `synchronized` blocks guarded by the same lock object concurrently. The lock is automatically released when the thread exits the block — normally or via an exception — so there's no risk of a permanently held lock due to a forgotten "unlock" call (unlike manual locks).

### 5. Instance `synchronized` vs static `synchronized` — can both run simultaneously?
- A `synchronized` **instance method** locks on `this` — the specific object instance.
- A `synchronized` **static method** locks on the **Class object** (`ClassName.class`) — shared across *all* instances of that class.
These are **two different locks**. So yes — **a thread executing a static synchronized method and a thread executing an instance synchronized method on the same object can run simultaneously**, because they're not competing for the same monitor. This is a common interview trap: people assume "synchronized" always means mutual exclusion across the board, but it's always scoped to a specific lock object.

### 6. Why `wait()`/`notify()`/`notifyAll()` must be called inside a `synchronized` block
These methods operate on an object's **monitor** — to call them, the calling thread must already **own that object's intrinsic lock**, because `wait()` needs to atomically release the lock and suspend the thread (avoiding a race where another thread could sneak in a change between a "check" and going to sleep), and `notify()`/`notifyAll()` need to safely wake a waiting thread without corrupting the lock's internal waiting-thread queue. If called outside a `synchronized` block on that object, the JVM throws `IllegalMonitorStateException` at runtime.

### 7. `Thread.sleep()` vs `Object.wait()` — lock release and CPU usage
- **`Thread.sleep(ms)`**: pauses the *current thread* for a fixed duration **without releasing any locks it holds**. The thread still occupies its monitor locks the entire time — dangerous if called inside a `synchronized` block, since it blocks other threads needlessly.
- **`Object.wait()`**: must be called on an object whose monitor the thread holds, and it **releases that lock** while waiting, allowing other threads to acquire it and make progress (and eventually call `notify()`/`notifyAll()` to wake the waiting thread). `wait()` doesn't consume CPU while parked — the thread is genuinely suspended, not spinning.
Rule of thumb: `sleep()` is for "pause execution for X time, I don't care what else happens"; `wait()` is for "release the lock and pause until someone signals a state change."

### 8. Producer-Consumer problem — clean `wait()`/`notifyAll()` solution
Producers add items to a shared bounded buffer; consumers remove them. The challenge: producers must block when the buffer is full, consumers must block when it's empty, and neither should corrupt shared state or starve.

```java
class BoundedBuffer<T> {
    private final Queue<T> queue = new LinkedList<>();
    private final int capacity;

    BoundedBuffer(int capacity) { this.capacity = capacity; }

    public synchronized void produce(T item) throws InterruptedException {
        while (queue.size() == capacity) {   // 'while', not 'if' — guards against spurious wakeups
            wait();
        }
        queue.add(item);
        notifyAll();                          // wake any waiting consumers (and producers)
    }

    public synchronized T consume() throws InterruptedException {
        while (queue.isEmpty()) {
            wait();
        }
        T item = queue.poll();
        notifyAll();
        return item;
    }
}
```
Two details interviewers probe:
1. **Always use `while`, never `if`**, to re-check the condition after waking — spurious wakeups are allowed by the JVM spec, and multiple waiting threads racing to consume the same signal can otherwise cause corruption.
2. **Use `notifyAll()` over `notify()`** in general-purpose solutions — `notify()` wakes an arbitrary single thread, which if it happens to be the wrong "type" (e.g., wakes another producer instead of a consumer when the buffer just got space) can lead to missed signals and starvation. `notifyAll()` avoids that at the cost of some extra wakeup/re-check overhead.

---

## Category 2: Memory Consistency, `volatile`, & Atomic Classes

### 9. Java Memory Model — CPU caches, registers, main memory, and stale reads
Modern CPUs have multiple cores, each with its **own L1/L2 cache** and registers, sitting in front of shared **main memory (RAM)**. For performance, a thread running on one core may read/write a variable's value **in its local CPU cache** rather than going all the way to main memory on every access — and the compiler/CPU may also **reorder instructions** for optimization as long as single-threaded correctness is preserved. This means **Thread A's write to a shared variable may sit in Core 1's cache and not yet be flushed to main memory (or visible to Core 2's cache)** — so Thread B, reading the "same" variable, can see a **stale value** indefinitely, or in the worst case, never see the update at all. The JMM defines the rules (happens-before relationships) for when writes by one thread are *guaranteed* to become visible to another thread — `synchronized`, `volatile`, and JUC utilities all establish these guarantees explicitly; without them, visibility is not guaranteed.

### 10. What problem does `volatile` solve? Visibility and reordering
`volatile` guarantees two things for a variable:
1. **Visibility**: every write to a `volatile` variable is flushed immediately to main memory, and every read fetches directly from main memory (not a stale cached copy) — so a write by one thread is always visible to the next read by any other thread.
2. **Ordering (happens-before)**: the JMM establishes that all writes that happened **before** a `volatile` write (in program order, on the writing thread) become visible to any thread that subsequently **reads** that same `volatile` variable — and the compiler/CPU cannot reorder instructions across that `volatile` read/write boundary. This is exactly why `volatile` is required on the `instance` field in double-checked-locking Singleton (Q42 from your OOP set) — without it, another thread could observe a half-constructed object due to reordering.

### 11. Why `volatile` is insufficient for compound operations like `count++`
`volatile` only guarantees **visibility**, not **atomicity**. `count++` is actually three separate operations: **read** `count`, **increment** the value, **write** it back. Even with `volatile`, two threads can both read the same value (say, 5) before either writes back, both compute 6, and both write 6 — one increment is silently lost, even though every individual read/write was perfectly visible. **Atomicity** means an operation completes as a single, indivisible unit with no possibility of another thread observing or interleaving with an intermediate state; **visibility** just means once a value *is* written, others can see it correctly. `volatile` solves the second problem, not the first — for compound operations you need `synchronized`, a `Lock`, or an atomic class like `AtomicInteger`.

### 12. Compare-And-Swap (CAS) — how `AtomicInteger`/`AtomicLong` achieve thread safety without blocking locks
CAS is a hardware-level atomic instruction (supported directly by the CPU) taking three operands: a memory location, an **expected** value, and a **new** value. It atomically checks: "is the value currently at this location equal to what I expect? If yes, swap it to the new value. If no, do nothing." — and this entire check-and-swap happens as one indivisible hardware operation, with no lock needed. `AtomicInteger.incrementAndGet()` internally loops: read the current value, compute `current + 1`, attempt a CAS — if another thread modified the value in between (CAS fails because the "expected" no longer matches), it **retries** the read-compute-CAS cycle until it succeeds. This is called a **lock-free / optimistic** approach: instead of blocking other threads out with a lock, threads just retry on conflict — which tends to be significantly faster than locking under low-to-moderate contention, since there's no thread suspension/context-switch overhead.

### 13. The ABA problem, and how `AtomicStampedReference` resolves it
In CAS, a thread checks "is the value still A?" before swapping — but if another thread changed the value from **A → B → back to A** in between, the CAS check sees "A" and proceeds, wrongly assuming nothing changed, even though the value went through an intermediate state that could have invalidated the operation (e.g., in a lock-free stack, the underlying nodes could have been popped and different nodes re-pushed, so structurally things did change even though the reference value matches). This is the **ABA problem**. `AtomicStampedReference<T>` fixes it by pairing the reference with an integer **stamp/version number** that's incremented on every update — CAS now checks *both* the reference **and** the stamp, so even if the value cycles back to A, the stamp will have moved from, say, 1 → 3, and the CAS will correctly fail because the expected stamp (1) no longer matches.

### 14. `ThreadLocal` — what it is, and its use in Spring Boot for transaction/session context
`ThreadLocal<T>` gives **each thread its own independent, isolated copy** of a variable — reads/writes by one thread never affect the copy seen by another thread, even though they're all referencing the "same" `ThreadLocal` object. Internally, each `Thread` object holds its own small map (`ThreadLocalMap`) keyed by the `ThreadLocal` instance. Spring Boot uses this heavily for **per-request context** in web applications: e.g., `TransactionSynchronizationManager` uses `ThreadLocal` to bind the current DB `Connection`/transaction to the thread handling that specific HTTP request, so nested service calls within the same request can access "the current transaction" without passing it explicitly through every method signature — while a concurrent request on a different thread gets a completely separate transaction context. Same pattern is used for `SecurityContextHolder` to carry the logged-in user's identity per-request.

### 15. `ThreadLocal` memory leaks in a Thread Pool, and why you must call `remove()`
Thread pools **reuse** threads across many tasks/requests rather than creating a new thread each time. If a task sets a `ThreadLocal` value (e.g., current user ID) but never clears it, the value **stays attached to that pooled thread's `ThreadLocalMap` even after the task finishes** — because the thread itself doesn't die, it just goes back into the pool. The next, unrelated task that happens to run on that same pooled thread can then **accidentally see stale data left over from a previous, different request** (a serious bug and potential security/data-leak issue — e.g., seeing another user's session context). Over time, if many distinct `ThreadLocal` values accumulate and are never cleaned up, this also causes a genuine **memory leak**, since the pooled threads live for the lifetime of the application and their `ThreadLocalMap` entries are never garbage collected. The fix: always call **`threadLocal.remove()`** in a `finally` block after the task completes, explicitly clearing the value before the thread returns to the pool.

---

## Category 3: The `java.util.concurrent` (JUC) Locks & Utilities

### 16. `ReentrantLock` vs `synchronized` — why choose it in a high-performance financial engine
`ReentrantLock` (from `java.util.concurrent.locks`) offers capabilities `synchronized` simply doesn't have:
- **`tryLock()`**: attempt to acquire the lock without blocking indefinitely — optionally with a timeout — letting a thread back off and do something else (e.g., retry, log, fail fast) instead of waiting forever, which matters a lot for latency-sensitive trading/settlement paths.
- **Interruptibility (`lockInterruptibly()`)**: a thread blocked waiting for the lock can be **interrupted** and respond (e.g., abort on shutdown), whereas a thread blocked on `synchronized` cannot be interrupted out of that wait.
- **Fairness**: `new ReentrantLock(true)` enforces a **FIFO ordering** for waiting threads acquiring the lock, preventing thread starvation under high contention — `synchronized` gives no such fairness guarantee, and a thread could theoretically be starved indefinitely.
- **Explicit `lock()`/`unlock()`** (must be paired in a `try/finally`) also allows more flexible lock-acquisition patterns (e.g., acquiring one lock, doing work, releasing, acquiring a different lock) that `synchronized`'s block-scoped nature doesn't support cleanly.
The trade-off: `ReentrantLock` requires disciplined manual `unlock()` in a `finally` block — forgetting it causes a permanent deadlock, unlike `synchronized`'s automatic release.

### 17. `ReadWriteLock` — optimizing for read-heavy workloads (100:1 reads to writes)
`ReadWriteLock` splits locking into two separate locks: a **read lock**, which can be held by **multiple threads simultaneously** as long as no thread holds the write lock, and a **write lock**, which is fully exclusive (blocks all readers and other writers). In a system where reads vastly outnumber writes (e.g., reading account balances/reference data far more often than updating them), a plain `synchronized`/`ReentrantLock` would force *every* reader to wait for *every other* reader — even though concurrent reads are perfectly safe since nobody's mutating anything. `ReadWriteLock` (via `ReentrantReadWriteLock`) allows all those reads to proceed **in parallel**, only serializing access when an actual write occurs — dramatically improving throughput for read-dominated access patterns, since contention is now only between reads-vs-writes and writes-vs-writes, not reads-vs-reads.

### 18. What is deadlock, and the four Coffman conditions
Deadlock is a state where two or more threads are each waiting on a resource held by another thread in the group, and none can proceed — a permanent standstill. It requires **all four** Coffman conditions simultaneously:
1. **Mutual Exclusion**: at least one resource is held in a non-shareable mode (only one thread can hold it at a time).
2. **Hold and Wait**: a thread holds at least one resource while simultaneously waiting to acquire additional resources held by others.
3. **No Preemption**: a resource can only be released voluntarily by the thread holding it — it can't be forcibly taken away.
4. **Circular Wait**: a closed chain of threads exists, where each thread waits for a resource held by the next thread in the chain (Thread A waits on B, B waits on C, ..., which waits on A).
Breaking **any one** of these four conditions prevents deadlock — in practice, breaking **circular wait** (via consistent lock ordering) is the most common and practical fix, since the other three are often intrinsic to how locks work.

### 19. Detecting deadlock in a running JVM, and prevention strategies for concurrent account transfers
**Detection**: use `jstack <pid>` (or `jcmd <pid> Thread.print`) to take a thread dump — the JVM explicitly detects cycles in lock-ownership and prints a section like `"Found one Java-level deadlock"` naming the exact threads and locks involved. `ThreadMXBean.findDeadlockedThreads()` can do this programmatically at runtime for monitoring/alerting. VisualVM and JConsole also visualize this graphically.
**Prevention strategies**, most relevant to concurrent account transfers (ties directly to Q45 from the OOP set):
- **Lock ordering**: always acquire multiple locks in a single, globally consistent order (e.g., by account ID) regardless of transfer direction — this breaks the circular-wait condition entirely.
- **Timed locks (`tryLock(timeout)`)**: instead of blocking forever, a thread gives up after a timeout, backs off, and retries — avoiding a permanent deadlock even if ordering discipline is imperfect somewhere in the codebase.
- **Avoiding nested locks** where possible, or minimizing the scope/duration a lock is held, to reduce the window in which a deadlock can form.

### 20. `CountDownLatch` — ensuring microservices are initialized before opening an API endpoint
`CountDownLatch` is a one-time synchronization gate initialized with a fixed count; threads call `await()` to block until the count reaches zero, and other threads call `countDown()` to decrement it. It **cannot be reset** once it hits zero.

```java
CountDownLatch initLatch = new CountDownLatch(3); // e.g., DB pool, Kafka consumer, cache warm-up

// each initialization task, possibly on its own thread:
initializeDatabasePool();
initLatch.countDown();

initializeKafkaConsumer();
initLatch.countDown();

warmUpCache();
initLatch.countDown();

// the thread that opens the API endpoint:
initLatch.await();          // blocks until all 3 have completed
server.start();             // only now, expose the endpoint
```
This guarantees the service never accepts traffic until every dependency it needs has finished starting up — avoiding a class of "started but not actually ready" production bugs (e.g., accepting a request before the DB pool is initialized and throwing errors on the first few real requests).

### 21. `CountDownLatch` vs `CyclicBarrier` — which can be reused?
- **`CountDownLatch`**: a **one-shot** gate. Once the count reaches zero, it's done — cannot be reset or reused. Threads calling `countDown()` don't block; only threads calling `await()` block.
- **`CyclicBarrier`**: makes a fixed group of threads all **wait for each other** to reach a common point before any of them proceed — and unlike the latch, **it automatically resets** once the barrier is tripped, so it can be reused for another "round" (hence "cyclic"). It also supports an optional `Runnable` action that runs once, automatically, when the last thread arrives at the barrier.
**`CyclicBarrier` is the one that can be reset and reused across multiple thread batches** — e.g., running the same group of worker threads through repeated phases of a batch job, synchronizing them at the end of each phase before starting the next.

### 22. `Semaphore` — limiting concurrent DB connections / rate-limiting a third-party API
A `Semaphore` maintains a set number of **permits**; a thread calls `acquire()` to take a permit (blocking if none are available) and `release()` to return it. Unlike a lock (binary — one owner), a semaphore allows **up to N threads** to hold a permit concurrently.

```java
Semaphore dbConnectionLimiter = new Semaphore(10); // max 10 concurrent DB calls

public Result queryDatabase() throws InterruptedException {
    dbConnectionLimiter.acquire();
    try {
        return jdbcTemplate.query(...);
    } finally {
        dbConnectionLimiter.release();
    }
}
```
For a **connection pool**, this caps how many threads can simultaneously use a scarce resource (e.g., 10 physical DB connections), forcing the 11th concurrent caller to wait rather than overwhelming the database. For **rate-limiting a third-party API call** (e.g., a payment gateway with a "max 5 concurrent requests" SLA), the same pattern throttles outbound concurrency to stay within the provider's limits — often combined with a scheduled task that periodically replenishes permits for time-window-based (rather than purely concurrency-based) rate limiting.

---

## Quick priority pass for the 17th
Given UBS's emphasis on high-throughput, thread-safe financial systems: **Category 2 (Q9–15: JMM, `volatile`, CAS, `ThreadLocal`)** and **Category 3 (Q16–22: locks, deadlock, JUC utilities)** are where banking interviewers dig deepest — they map directly onto real production concerns (stale reads, race conditions on balances, connection-pool exhaustion). Make sure you can whiteboard **Q8 (Producer-Consumer)**, **Q18–19 (deadlock + prevention)**, and **Q22 (Semaphore for connection pooling)** cold — these are the concurrency equivalents of your OOP set's Singleton/BankAccount questions and get asked just as often.
