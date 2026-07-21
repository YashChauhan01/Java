# Java Executors Utility Class & Fork/Join Pool: Prebuilt Thread Pools and Work Stealing

## Executive Summary
This lesson introduces the **Executors** utility class, which provides ready-made factory methods for common thread pool configurations, sparing you from manually customizing a `ThreadPoolExecutor` every time. It then dives deep into the **Fork/Join Pool** and its **work-stealing** mechanism — a sophisticated pattern for splitting large tasks into subtasks and dynamically balancing them across threads for maximum parallelism.

## Core Concepts

### Executors Utility Class
**The Layman's Definition:** **Executors** is a helper class in `java.util.concurrent` offering static factory methods that instantly create pre-configured thread pools for common scenarios, instead of you manually specifying every `ThreadPoolExecutor` parameter (core size, max size, queue, etc.).

**How it Works / The Logic:** It exposes several factory methods, each tuned for a different use case. You call them directly on the class (e.g., `Executors.newFixedThreadPool(5)`), and they return a ready-to-use executor.

**Example:** Instead of writing a full `new ThreadPoolExecutor(...)` constructor with six parameters, you simply call `Executors.newFixedThreadPool(5)` to get an equivalent pool with sensible defaults.

---

### Fixed Thread Pool Executor
**The Layman's Definition:** A thread pool with a **fixed, unchanging number of threads** — the minimum and maximum pool size are identical.

**How it Works / The Logic:**
- Created via `Executors.newFixedThreadPool(n)`.
- `corePoolSize` = `maximumPoolSize` = `n` (whatever you pass in).
- Uses an **unbounded queue** (backed by a linked list) — no fixed capacity limit.
- Threads stay alive even when idle (they are never terminated due to inactivity).

**When to use:** When you know the exact number of concurrent async tasks you need and don't want more.

**Disadvantage:** Not suited for heavy/unpredictable workloads — since the pool size is capped and fixed, it leads to limited concurrency during traffic spikes.

**Example:**
```java
ExecutorService executor = Executors.newFixedThreadPool(5);
executor.submit(() -> System.out.println("Task running"));
```

---

### Cached Thread Pool Executor
**The Layman's Definition:** A thread pool that **starts empty and dynamically creates new threads on demand**, ideal for bursts of short-lived work.

**How it Works / The Logic:**
- Created via `Executors.newCachedThreadPool()`.
- Starts with **0 threads**; minimum pool size = 0.
- Maximum pool size = `Integer.MAX_VALUE` (essentially unlimited, bounded only by system memory).
- **Queue size = 0** — tasks are never queued; a new thread is created immediately if none is free.
- Idle threads are terminated after **60 seconds** of inactivity.

**When to use:** Best for handling bursts of short-lived tasks.

**Disadvantage:** If many long-running tasks are submitted rapidly, the pool can spawn excessive threads, causing high memory usage — not suitable for long-lived tasks.

**Example:**
```java
ExecutorService executor = Executors.newCachedThreadPool();
executor.submit(() -> System.out.println("Quick task"));
```

---

### Single Thread Executor
**The Layman's Definition:** A thread pool with **exactly one worker thread**, guaranteeing tasks run one at a time, in order.

**How it Works / The Logic:**
- Created via `Executors.newSingleThreadExecutor()`.
- Minimum and maximum pool size = 1.
- Uses an unbounded queue for waiting tasks.
- The single thread stays alive even when idle.

**When to use:** When tasks must be processed strictly sequentially.

**Disadvantage:** Zero concurrency — only one task executes at any given moment.

**Example:**
```java
ExecutorService executor = Executors.newSingleThreadExecutor();
executor.submit(() -> System.out.println("Processed in order"));
```

---

### Comparison of Executors Factory Methods

| Executor Type | Core / Max Threads | Queue | Idle Behavior | Best For | Weakness |
|---|---|---|---|---|---|
| **Fixed Thread Pool** | Equal, fixed (n / n) | Unbounded | Threads stay alive | Known, stable concurrency needs | Poor under heavy/unpredictable load |
| **Cached Thread Pool** | 0 / `Integer.MAX_VALUE` | None (size 0) | Threads die after 60s idle | Bursts of short-lived tasks | Risk of excessive threads/memory with long tasks |
| **Single Thread Executor** | 1 / 1 | Unbounded | Thread stays alive | Strict sequential processing | No concurrency at all |

---

### Fork/Join Pool
**The Layman's Definition:** A specialized thread pool designed to break a **large task into smaller subtasks** ("fork"), run them in parallel across multiple threads, and then combine ("join") their results into a final answer — maximizing parallelism for divide-and-conquer problems.

**How it Works / The Logic:**
- **Fork**: Splits a task into smaller subtasks that can be further subdivided recursively.
- **Join**: Waits for each subtask to finish, then combines the results into the final output.
- Purpose: Instead of one thread doing all the work of one big task, multiple threads can each work on a piece — bringing true parallelism to a single logical task.

**Example (Synthesized Example — restaurant analogy):** Imagine one chef preparing an entire 100-dish banquet alone versus splitting the menu among 10 chefs, each cooking 10 dishes in parallel, then combining all dishes onto the banquet table at the end. The splitting is "fork," the combining/waiting is "join."

---

### Work-Stealing Pool Executor
**The Layman's Definition:** **Work stealing** is the load-balancing strategy that a Fork/Join Pool uses so that idle threads can "steal" pending subtasks from busy threads' queues, instead of sitting idle while other threads are overloaded.

**How it Works / The Logic:**
- Created via `Executors.newWorkStealingPool()`, which internally builds a **ForkJoinPool**.
- The pool maintains **two types of queues**:
  1. **Submission Queue** — a single shared queue where new incoming tasks wait if no thread is immediately free.
  2. **Work-Stealing Queue (a Deque)** — each individual thread has its *own* private deque for subtasks it creates via `fork()`.
- **Priority order when a thread becomes free:**
  1. Check its **own work-stealing queue** first — pick up any pending subtask there.
  2. If empty, check the shared **submission queue** for any new incoming task.
  3. If that's also empty, **steal** a subtask from the back of another *busy* thread's work-stealing queue.
- **Why a Deque (double-ended queue)?** The owning thread consumes its own subtasks from the **front**; any other thread stealing work takes from the **back** — this minimizes contention between the owner and thieves.
- **Sizing:** If you don't specify a thread count, it defaults to `Runtime.getRuntime().availableProcessors()` (one thread per CPU core). You can also pass an explicit number, which fixes both min and max pool size to that value.

**Example (traced flow from the transcript):**
1. Task 1 (simple, non-divisible) arrives → assigned to Thread 1.
2. Task 2 (a divisible recursive task) arrives → assigned to Thread 2.
3. Task 3 arrives while both threads are busy → placed in the **submission queue**.
4. Thread 2's task splits into Subtask A and Subtask B via `fork()`: Thread 2 keeps working on Subtask A; Subtask B goes into **Thread 2's own work-stealing queue**.
5. Thread 1 finishes Task 1 → checks its own (empty) work-stealing queue → checks submission queue → finds and picks up Task 3.
6. Thread 1 finishes Task 3 → checks its own work-stealing queue (empty) → checks submission queue (empty) → **steals** Subtask B from the back of Thread 2's work-stealing queue and starts working on it.

---

### Implementing Splittable Tasks: RecursiveTask vs. RecursiveAction
**The Layman's Definition:** To make a task "forkable" (splittable into subtasks) in a Fork/Join Pool, you must define it as a subclass of either **RecursiveTask** or **RecursiveAction**, depending on whether it needs to return a value.

**How it Works / The Logic:**

| Class | Returns a Value? | Use Case |
|---|---|---|
| **RecursiveTask\<T>** | Yes | When each subtask must compute and return a result |
| **RecursiveAction** | No (`void`) | When each subtask performs work but returns nothing |

- Both require implementing a `compute()` method, where you:
  1. Check a **terminating condition** — if the task is small enough, compute it directly without further splitting.
  2. Otherwise, split the task into two (or more) smaller subtasks.
  3. Call `fork()` on subtasks to schedule them for parallel execution (one subtask continues on the current thread; the other is placed into that thread's work-stealing queue for potential stealing).
  4. Call `join()` to wait for each subtask's result and combine them into the final output.

**Example (summing numbers 1 to 100 via divide-and-conquer):**
```java
class ComputeSumTask extends RecursiveTask<Integer> {
    private int start, end;

    ComputeSumTask(int start, int end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected Integer compute() {
        // Terminating condition: small enough to compute directly
        if (end - start <= 4) {
            int sum = 0;
            for (int i = start; i <= end; i++) sum += i;
            return sum;
        }

        // Split into two subtasks (like binary search)
        int mid = (start + end) / 2;
        ComputeSumTask leftTask = new ComputeSumTask(start, mid);
        ComputeSumTask rightTask = new ComputeSumTask(mid + 1, end);

        leftTask.fork();              // schedule left subtask for parallel execution
        int rightResult = rightTask.compute(); // continue right subtask on current thread
        int leftResult = leftTask.join();      // wait for left subtask's result

        return leftResult + rightResult; // combine results
    }
}

// Usage:
ForkJoinPool pool = ForkJoinPool.commonPool();
ComputeSumTask task = new ComputeSumTask(1, 100);
int totalSum = pool.invoke(task);
```

**Two ways to submit work to a Fork/Join Pool:**
1. `Executors.newWorkStealingPool()` — internally builds a `ForkJoinPool` for you.
2. `ForkJoinPool.commonPool()` — directly access/create a `ForkJoinPool` and call `submit()`.

**Where tasks land:**

| Action | Destination |
|---|---|
| `submit()` a new task | Submission Queue (if no thread is free) |
| `fork()` a subtask | The forking thread's own Work-Stealing Queue |
| Idle thread looking for work | 1️⃣ Own work-stealing queue → 2️⃣ Submission queue → 3️⃣ Steal from another thread's work-stealing queue (from the back) |

## Key Takeaways & Quick Reference
- The **Executors** class provides quick factory methods (`newFixedThreadPool`, `newCachedThreadPool`, `newSingleThreadExecutor`, `newWorkStealingPool`) as shortcuts over manually configuring `ThreadPoolExecutor`.
- **Fixed thread pool**: equal min/max size, unbounded queue, threads never die — good for known, stable workloads.
- **Cached thread pool**: starts at 0 threads, can grow to `Integer.MAX_VALUE`, no queue, threads die after 60s idle — good for short-lived task bursts only.
- **Single thread executor**: exactly one thread — guarantees strict sequential task execution, zero concurrency.
- **Fork/Join Pool** splits a large task into subtasks (`fork()`) and recombines their results (`join()`) to parallelize a single logical task across multiple threads.
- **Work stealing** lets an idle thread pull unfinished subtasks from the back of a busy thread's private queue, maximizing CPU utilization.
- Each thread in a work-stealing pool has its own **work-stealing queue (Deque)**; there's also one shared **submission queue** for freshly submitted tasks.
- To make a task splittable, extend **RecursiveTask** (if it returns a value) or **RecursiveAction** (if it doesn't), and implement `compute()` with a terminating condition, `fork()` calls, and `join()` calls.
- Priority order for an idle thread seeking work: **own work-stealing queue → submission queue → steal from another thread's queue**.

## Glossary of Terms
- **Fork**: The act of splitting a task into smaller subtasks that can run in parallel.
- **Join**: Waiting for a forked subtask to complete and retrieving its result.
- **Deque (Double-Ended Queue)**: A queue that allows insertion/removal from both ends; used for work-stealing queues so the owner and thieves access opposite ends.
- **Submission Queue**: The shared queue in a Fork/Join Pool where newly submitted (not yet forked) tasks wait if no thread is free.
- **RecursiveTask**: An abstract class for defining a Fork/Join subtask that returns a computed value.
- **RecursiveAction**: An abstract class for defining a Fork/Join subtask that performs work but returns no value.
- **Available Processors**: `Runtime.getRuntime().availableProcessors()` — the number of CPU cores visible to the JVM, often used as a default thread count.
