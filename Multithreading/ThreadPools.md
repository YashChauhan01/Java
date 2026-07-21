# Java Thread Pool: Architecture, Configuration & Sizing Strategy

## Executive Summary
This lesson covers Java's **ThreadPoolExecutor** — what a thread pool is, why it exists, and exactly how it decides whether to use an idle thread, queue a task, spin up a new thread, or reject the task outright. It closes with a real interview question — *"why did you choose this core pool size?"* — and walks through a practical, memory-aware formula for sizing a pool correctly.

## Core Concepts

### Thread Pool
**The Layman's Definition:** A **thread pool** is a pre-built collection of reusable worker threads that sit ready to execute tasks, instead of your code creating a brand-new thread for every single task.

**How it Works / The Logic:**
- A fixed (or bounded) set of threads is created upfront.
- When a task arrives, an idle thread is assigned to it.
- Once a thread finishes its task, it doesn't die — it returns to the pool and waits for the next task.
- If no thread is free when a task arrives, the task waits in a **queue** until a thread frees up.

**Example:** Threads T1 and T2 are created once. Task A is assigned to T1, Task B to T2. When T1 finishes Task A, it goes back into the pool. A new Task C then reuses T1 — no new thread was created for Task C.

---

### Why Use a Thread Pool (Advantages)

**The Layman's Definition:** Thread pools exist to avoid the cost and chaos of creating a fresh thread for every task.

**How it Works / The Logic:**
1. **Saves thread-creation time** — creating a thread involves allocating memory (stack space, program counter, etc.), which takes time. Reusing threads skips this repeated cost.
2. **Removes lifecycle management overhead** — a thread naturally moves through states (new → runnable → running → waiting → terminated). Managing this manually is complex; the executor framework abstracts it away.
3. **Improves performance by limiting context switching** — if you spawn hundreds of threads on a CPU with only a few cores, the CPU spends most of its time swapping threads in and out ("**context switching**") rather than doing real work. A thread pool caps the number of live threads, keeping context switching low and CPU utilization high.

**Example:** A server with 2 CPU cores receiving 100 tasks. Without a pool, 100 threads are created and the CPU constantly switches between them, wasting cycles. With a pool of, say, 4–8 threads, tasks queue up and get processed efficiently without excessive switching.

---

### The Executor Framework Hierarchy

**The Layman's Definition:** Java provides a built-in framework (in `java.util.concurrent`) of interfaces and classes to manage thread pools so you don't build one from scratch.

**How it Works / The Logic:**
- **Executor** (interface) — the top-level interface with just one method: `execute()`.
- **ExecutorService** (interface, extends `Executor`) — adds richer lifecycle controls like `shutdown()`, `isTerminated()`, etc.
- **ThreadPoolExecutor** (class, implements `ExecutorService`) — the customizable, concrete thread pool implementation (main focus of this lesson).
- **ForkJoinPool** (class, implements `ExecutorService`) — a specialized pool for divide-and-conquer parallel tasks.
- **ScheduledExecutorService** (interface, extends `ExecutorService`) — adds the ability to run tasks on a schedule.

| Interface/Class | Role |
|---|---|
| `Executor` | Bare-bones: only `execute()` |
| `ExecutorService` | Adds shutdown/termination controls |
| `ThreadPoolExecutor` | Concrete, configurable thread pool |
| `ForkJoinPool` | Parallel, recursive task splitting |
| `ScheduledExecutorService` | Time-based/scheduled execution |

---

### ThreadPoolExecutor Constructor Parameters

**The Layman's Definition:** `ThreadPoolExecutor` is configured through a constructor with several parameters that together define how many threads exist, how tasks queue up, and what happens under overload.

**How it Works / The Logic (parameter by parameter):**

- **corePoolSize** — the minimum number of threads created immediately and kept alive in the pool, even while idle.
- **allowCoreThreadTimeOut** (boolean, default `false`) — if `true`, even core threads can be terminated after sitting idle for `keepAliveTime`. If `false`, core threads live forever regardless of `keepAliveTime`.
- **keepAliveTime** — how long an idle thread (beyond `corePoolSize`, or all threads if `allowCoreThreadTimeOut` is `true`) is kept alive before being terminated.
- **TimeUnit** — the unit (seconds, minutes, etc.) applied to `keepAliveTime`.
- **maximumPoolSize** — the absolute ceiling on the number of threads the pool is allowed to create.
- **workQueue** (a `BlockingQueue`) — holds tasks waiting for a free thread.
  - **Bounded queue**: fixed capacity (e.g., `ArrayBlockingQueue`) — generally preferred, since it gives you control over how much work can pile up.
  - **Unbounded queue**: no capacity limit (e.g., `LinkedBlockingQueue`) — generally avoided, since it can grow without limit and hide overload problems.
- **ThreadFactory** — lets you customize how each thread is created (custom name, priority, daemon flag). If omitted, a default factory is used.
- **RejectedExecutionHandler** — defines what happens to a task that cannot be accepted anywhere (pool full, queue full, max threads reached).

| Queue Type | Capacity | Typical Use |
|---|---|---|
| Bounded (e.g. `ArrayBlockingQueue`) | Fixed | Preferred — controlled backpressure |
| Unbounded (e.g. `LinkedBlockingQueue`) | Unlimited | Generally avoided — risk of unbounded growth |

---

### Task Execution Flow (The Decision Sequence)

**The Layman's Definition:** When a new task arrives, the executor follows a strict, ordered decision process before ever rejecting work.

**How it Works / The Logic (in order):**
1. **Is a core thread free?** If yes → assign the task to it.
2. **Is the queue not full?** If yes → put the task in the queue (wait for a thread).
3. **Can a new thread be created (below maximumPoolSize)?** If yes → create a new thread and assign the task.
4. **None of the above possible** → the task is **rejected**, handled by the `RejectedExecutionHandler`.

**Example:** `corePoolSize = 3`, `maximumPoolSize = 5`, queue size = 5.
- Tasks 1–3 → assigned directly to the 3 core threads.
- Tasks 4–8 → placed in the queue (now full).
- Task 9 → no free thread, queue full → a 4th thread is created.
- Task 10 → a 5th thread is created (hits `maximumPoolSize`).
- Task 11 → no thread free, queue full, max threads reached → **rejected**.
- When a busy thread finishes, it immediately pulls the next task from the queue.

**Why queue before creating new threads?** Even though creating a new thread is technically allowed (below max), the design deliberately favors the queue first. This is because `corePoolSize` is meant to represent the *average sufficient* number of threads for typical load. Creating extra threads for every burst means those threads, once done, sit idle in the pool going forward — wasting resources on the long-term average case just to handle a short-term spike.

---

### Rejection Policies

**The Layman's Definition:** When a task truly cannot be accepted (pool and queue are both maxed out), the `RejectedExecutionHandler` decides what to do with it.

**How it Works / The Logic — four built-in strategies:**

| Policy | Behavior |
|---|---|
| **AbortPolicy** | Throws a `RejectedExecutionException` |
| **DiscardPolicy** | Silently drops the task — no exception, no notice |
| **CallerRunsPolicy** | Executes the rejected task on the thread that submitted it (e.g., the main thread) |
| **DiscardOldestPolicy** | Removes the oldest task currently in the queue to make room for the new one |

You can also implement a **custom handler** by implementing the `RejectedExecutionHandler` interface — commonly used to log the rejection for debugging.

**Example (custom handler, from the transcript):**
```java
class CustomHandler implements RejectedExecutionHandler {
    @Override
    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
        System.out.println("Task rejected: " + r.toString());
    }
}
```

---

### ThreadPoolExecutor Lifecycle

**The Layman's Definition:** A thread pool itself moves through defined states, similar to how an individual thread does.

**How it Works / The Logic:**
- **Running** — default state; accepts and processes new tasks normally.
- **Shutdown** (triggered by `shutdown()`) — stops accepting new tasks, but lets already-submitted/running tasks finish.
- **Stop** (triggered by `shutdownNow()`) — stops accepting new tasks **and** forcefully interrupts currently running tasks.
- **Terminated** — final state once all threads are eliminated; checked via `isTerminated()`.

| Method | Accepts New Tasks? | Finishes In-Flight Tasks? |
|---|---|---|
| `shutdown()` | No | Yes — lets them complete |
| `shutdownNow()` | No | No — forcefully stops them |

---

### Worked Code Example

**The Layman's Definition:** Putting it all together — a pool with `corePoolSize=2`, `maximumPoolSize=4`, and a queue of size 2, fed a custom thread factory and a custom rejection handler.

**How it Works / The Logic:**
```java
class MyCustomThreadFactory implements ThreadFactory {
    @Override
    public Thread newThread(Runnable r) {
        Thread thread = new Thread(r);
        thread.setPriority(Thread.NORM_PRIORITY);
        thread.setDaemon(false);
        thread.setName("custom-thread");
        return thread;
    }
}

class CustomHandler implements RejectedExecutionHandler {
    @Override
    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
        System.out.println("Task rejected: " + r.toString());
    }
}

ThreadPoolExecutor executor = new ThreadPoolExecutor(
    2,                              // corePoolSize
    4,                              // maximumPoolSize
    10, TimeUnit.MINUTES,           // keepAliveTime, unit
    new ArrayBlockingQueue<>(2),    // bounded work queue
    new MyCustomThreadFactory(),    // custom thread factory
    new CustomHandler()             // custom rejection handler
);

for (int i = 1; i <= 7; i++) {
    int taskId = i;
    executor.submit(() -> {
        System.out.println("Task " + taskId + " processed by " + Thread.currentThread().getName());
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    });
}
executor.shutdown();
```

**Traced behavior with 7 tasks** (core=2, max=4, queue=2):
- Tasks 1–2 → assigned to the 2 core threads.
- Tasks 3–4 → fill the queue.
- Task 5 → no thread free, queue full → new thread #3 created.
- Task 6 → no thread free, queue full → new thread #4 created (hits max).
- Task 7 → no thread free, queue full, max reached → **rejected**.
- As threads free up, they pull Tasks 3 and 4 from the queue.

---

### Interview Question: How Do You Choose `corePoolSize` and `maximumPoolSize`?

**The Layman's Definition:** There's no single "correct" number — pool sizing is an engineering estimate based on hardware limits, memory limits, and the nature of the work being done.

**How it Works / The Logic — key factors to weigh:**
1. **CPU core count** — too many threads on too few cores just causes excessive context switching.
2. **JVM memory** — each thread consumes memory (stack, program counter, etc.); the JVM's total memory budget caps how many threads can realistically exist.
3. **Task nature: CPU-intensive vs. I/O-intensive:**
   - **CPU-intensive** tasks (heavy computation) → fewer threads, ideally close to the number of CPU cores, since more threads than cores just adds switching overhead.
   - **I/O-intensive** tasks (DB calls, network calls) → more threads can help, since threads are frequently idle waiting on I/O, and the CPU can work on other threads during that wait.
4. **Concurrency requirements** — how much simultaneous load is expected.
5. **Memory required per request** — how much heap a single task consumes.
6. **Throughput** — how fast the system needs to process requests overall.

| Task Type | Threads vs. CPU Cores | Reasoning |
|---|---|---|
| CPU-intensive | Threads ≈ number of cores | More threads than cores just adds context-switch overhead |
| I/O-intensive | Threads can exceed core count | Threads are frequently idle waiting on I/O, freeing the CPU for others |

**Step 1 — CPU-based formula:**

`Number of threads = CPU cores × (1 + (Wait Time / Processing Time))`

**Example:** 64 CPU cores, request waiting time = 50ms, processing time = 100ms (fairly CPU-intensive workload):
`64 × (1 + 50/100) ≈ 64` (approximately equal to core count for high-CPU-intensity work).

**Step 2 — JVM memory constraint:**
Given a 2GB JVM: Heap = 1000MB, code cache = 128MB, JVM overhead = 256MB → roughly 500MB left over. If each thread needs ~5MB (stack + registers, etc.), that leaves room for **~100 threads** based purely on memory for thread structures.

**Step 3 — Per-request heap memory constraint:**
If each request needs ~10MB of heap space to process (loaded data, DB results, etc.), and you want to use only ~60% of a 1000MB heap as a safety buffer (600MB), that supports:
`600MB ÷ 10MB per request ≈ 60 threads`

**Final estimate:** Combining the CPU formula (~64) and the memory constraint (~60), a safe range is roughly **60 (core) to 64–70 (maximum)**, refined further through **load testing** and monitoring/profiling tools in production.

## Key Takeaways & Quick Reference
- A **thread pool** reuses a fixed set of worker threads instead of creating a new thread per task, saving creation time and reducing context switching.
- The execution order is strict: **free thread → queue → new thread (up to max) → reject** — new threads are *not* created before the queue is full, to avoid idle threads sitting around after bursts subside.
- `corePoolSize` = minimum threads always kept alive; `maximumPoolSize` = hard ceiling on total threads.
- `allowCoreThreadTimeOut` must be `true` for `keepAliveTime` to have any effect on core threads.
- Prefer **bounded queues** (`ArrayBlockingQueue`) over unbounded ones for predictable backpressure.
- Rejection is handled via a `RejectedExecutionHandler` — built-ins are `AbortPolicy`, `DiscardPolicy`, `CallerRunsPolicy`, and `DiscardOldestPolicy`.
- `shutdown()` finishes in-flight tasks before terminating; `shutdownNow()` forcefully stops everything immediately.
- Correct pool sizing depends on **CPU cores, JVM memory, and task nature (CPU-bound vs I/O-bound)** — not an arbitrary number — and should be refined with real load testing.

## Glossary of Terms
- **Context Switching**: The CPU saving the state of one thread and loading another, which introduces overhead and idle CPU time.
- **Runnable**: A functional interface representing a task to be executed by a thread.
- **BlockingQueue**: A queue implementation that supports waiting for space to become available (used to hold tasks awaiting a thread).
- **Daemon Thread**: A background thread that doesn't prevent the JVM from exiting once all non-daemon threads finish.
- **Heap**: The JVM memory region used for storing objects created during program execution.
