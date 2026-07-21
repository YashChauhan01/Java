# Java Future, Callable & CompletableFuture: Mastering Asynchronous Results

## Executive Summary
This lesson builds directly on `ThreadPoolExecutor` by answering a critical question: once you submit an async task, how does the caller find out its status or result? It covers **Future** and **Callable** as the foundational tools for tracking and retrieving async task outcomes, then introduces **CompletableFuture** — a Java 8 upgrade that adds powerful chaining capabilities for composing complex asynchronous workflows.

## Core Concepts

### Future
**The Layman's Definition:** A **Future** is an interface representing the eventual result of an asynchronous task — a placeholder you can check later to see if the task finished, get its output, or handle any error it threw.

**How it Works / The Logic:**
- When you call `executor.submit(task)`, it immediately returns a `Future` object — even though the task itself may still be running on another thread.
- The calling thread doesn't wait; it continues its own work while holding the `Future` reference for later use.
- Five key methods on `Future`:

| Method | Purpose |
|---|---|
| `cancel(boolean mayInterrupt)` | Attempts to stop the task; returns `false` if the task already completed (can't be cancelled) |
| `isCancelled()` | Returns `true`/`false` — was the task cancelled before it finished normally? |
| `isDone()` | Returns `true` if the task finished for *any* reason — normal completion, exception, or cancellation |
| `get()` | **Blocks** the calling thread indefinitely until the task completes, then returns the result |
| `get(timeout, unit)` | Blocks only up to the given timeout; throws `TimeoutException` if the task isn't done in time |

**Example:**
```java
ExecutorService executor = Executors.newFixedThreadPool(1);
Future<?> future = executor.submit(() -> {
    try {
        Thread.sleep(7000); // simulate a 7-second task
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
});

System.out.println(future.isDone()); // false — task still running

try {
    future.get(2, TimeUnit.SECONDS); // waits only 2 seconds
} catch (TimeoutException e) {
    System.out.println("Timeout exception happened");
}

future.get(); // blocks indefinitely until task completes
System.out.println(future.isDone());      // true
System.out.println(future.isCancelled()); // false — completed naturally
```

**Internal mechanics:** When you call `submit()`, the executor wraps your task inside a **FutureTask** (a class that implements `RunnableFuture`, which itself extends both `Runnable` and `Future`). This `FutureTask` bundles the runnable/callable logic together with its execution state, and the thread pool updates that state as the task progresses.

---

### Runnable vs. Callable (The Three Flavors of `submit()`)

**The Layman's Definition:** `ThreadPoolExecutor.submit()` can accept three different kinds of tasks, differing mainly in whether the task returns a value.

**How it Works / The Logic:**

| Flavor | Signature | Returns a Value? | `Future.get()` Result |
|---|---|---|---|
| `submit(Runnable)` | Plain runnable | No | Always `null` |
| `submit(Runnable, T result)` | Runnable + a shared result object | Indirectly (via shared object workaround) | Returns the same object you passed in, now mutated by the task |
| `submit(Callable<T>)` | Callable | Yes — natively | Returns the actual computed value |

**Example — `submit(Runnable)` (always null):**
```java
Future<?> f = executor.submit(() -> System.out.println("Do something"));
Object result = f.get(); // result is always null — Runnable has no return type
```

**Example — `submit(Runnable, T)` (shared object workaround):**
```java
class MyRunnable implements Runnable {
    private List<Integer> list;
    MyRunnable(List<Integer> list) { this.list = list; }
    @Override
    public void run() {
        list.add(300); // mutates the shared object
    }
}

List<Integer> output = new ArrayList<>();
Future<List<Integer>> future = executor.submit(new MyRunnable(output), output);
List<Integer> result = future.get(); // waits, then returns the mutated list
System.out.println(result.get(0)); // 300
```

**Example — `submit(Callable<T>)` (clean, native return value):**
```java
Future<List<Integer>> future = executor.submit(() -> {
    List<Integer> output = new ArrayList<>();
    output.add(300);
    return output; // Callable can return directly
});
List<Integer> result = future.get();
System.out.println(result.get(0)); // 300
```

**Key distinction:** `Runnable` and `Callable` both represent a unit of work to execute, but **Runnable has no return type**, while **Callable can return a value** — making `Callable` the cleaner choice whenever you need an actual result back.

---

### CompletableFuture
**The Layman's Definition:** **CompletableFuture** (introduced in Java 8) is an enhanced version of `Future` that supports **chaining** — letting you attach follow-up actions that automatically run once an async step completes, without manually blocking and re-submitting tasks.

**How it Works / The Logic:**
- `CompletableFuture` implements `Future`, so it inherits everything `Future` can do (`get()`, `isDone()`, etc.), plus chaining methods.
- If you don't supply your own executor, it defaults to the shared **ForkJoinPool** (dynamically sized based on available processors) — giving you no control over thread count.
- If you want control over pool size, pass your own `ThreadPoolExecutor` explicitly.

---

#### 1. `supplyAsync()`
**The Layman's Definition:** Kicks off the initial asynchronous computation — the entry point of a chain.

**How it Works / The Logic:** Takes a `Supplier<T>` (a no-argument function that returns a value) and optionally an `Executor`. Runs the supplier on a new thread and returns a `CompletableFuture<T>`.

**Example:**
```java
CompletableFuture<String> cf = CompletableFuture.supplyAsync(() -> {
    return "Task completed";
}, poolExecutor);

System.out.println(cf.get()); // blocks, then prints "Task completed"
```

---

#### 2. `thenApply()` vs `thenApplyAsync()`
**The Layman's Definition:** Both apply a transformation function to the result of the previous step and return a new `CompletableFuture`. The difference is *which thread* runs the transformation.

**How it Works / The Logic:**

| Method | Thread Used |
|---|---|
| `thenApply()` | **Synchronous** — reuses the same thread that completed the previous async step |
| `thenApplyAsync()` | **Asynchronous** — releases that thread and picks a new one (from ForkJoinPool or your executor) |

**Example:**
```java
CompletableFuture.supplyAsync(() -> {
    System.out.println("supplyAsync thread: " + Thread.currentThread().getName());
    return "Concept";
}, poolExecutor)
.thenApply(val -> {
    System.out.println("thenApply thread: " + Thread.currentThread().getName());
    return val + " and Coding";
});
// Output: same thread name for both lines (e.g., "pool-1-thread-1")
```
With `thenApplyAsync()` instead, the second `thenApply` line would print a **different** thread name (typically from the ForkJoinPool, unless a custom executor is passed).

---

#### 3. `thenCompose()` vs `thenComposeAsync()`
**The Layman's Definition:** Used to **chain dependent async operations in a guaranteed order** — when Task B must only start after Task A finishes, and both return `CompletableFuture`s.

**How it Works / The Logic:**
- Unlike `thenApply` (which transforms a plain value), `thenCompose` expects the mapping function to itself return another `CompletableFuture` — it "flattens" nested futures instead of wrapping them.
- Internally maintains an ordered stack of dependent actions, so even when using the async variant across multiple steps, execution order is guaranteed.

**Example:**
```java
CompletableFuture<String> result = CompletableFuture.supplyAsync(() -> "Hello", poolExecutor)
    .thenCompose(value -> CompletableFuture.supplyAsync(() -> value + " World", poolExecutor))
    .thenCompose(value -> CompletableFuture.supplyAsync(() -> value + " All", poolExecutor));

System.out.println(result.get()); // Always prints: "Hello World All" (never out of order)
```

---

#### 4. `thenAccept()` / `thenAcceptAsync()`
**The Layman's Definition:** A **terminal (end-of-chain) step** — it consumes the previous result but doesn't return anything.

**How it Works / The Logic:** Accepts an input parameter (via `Consumer<T>`) but its return type is `Void`, meaning nothing further can meaningfully be chained after it with `thenApply` (since there's no value to work on).

**Example:**
```java
CompletableFuture.supplyAsync(() -> "Final Value", poolExecutor)
    .thenAccept(val -> System.out.println("Printing the value: " + val));
// Prints: "Printing the value: Final Value" — returns nothing further
```

---

#### 5. `thenCombine()` / `thenCombineAsync()`
**The Layman's Definition:** Merges the results of **two independent** `CompletableFuture`s once both complete.

**How it Works / The Logic:** Takes a second `CompletableFuture` plus a `BiFunction` that receives both results as input and produces a combined output.

**Example:**
```java
CompletableFuture<Integer> task1 = CompletableFuture.supplyAsync(() -> 10);
CompletableFuture<String> task2 = CompletableFuture.supplyAsync(() -> "K");

CompletableFuture<String> combined = task1.thenCombine(task2, (t, u) -> t + "" + u);

System.out.println(combined.get()); // "10K"
```

---

### CompletableFuture Method Summary

| Method Pair | Purpose | Sync Variant Thread | Async Variant Thread |
|---|---|---|---|
| `supplyAsync()` | Start the initial async operation | N/A (always new) | Uses ForkJoinPool or provided executor |
| `thenApply()` / `thenApplyAsync()` | Transform the result into a new value | Same thread as previous step | New thread |
| `thenCompose()` / `thenComposeAsync()` | Chain dependent async ops, preserving order | Same thread | New thread (order still guaranteed) |
| `thenAccept()` / `thenAcceptAsync()` | Consume result, terminal step (no return) | Same thread | New thread |
| `thenCombine()` / `thenCombineAsync()` | Merge results of two independent futures | Same thread | New thread |

## Key Takeaways & Quick Reference
- `Future` lets a caller check status (`isDone`, `isCancelled`), retrieve results (`get`), or cancel (`cancel`) an async task submitted to a thread pool.
- `get()` **blocks indefinitely**; `get(timeout, unit)` blocks only up to a limit and throws `TimeoutException` if exceeded.
- Internally, `submit()` wraps your task into a `FutureTask` (implements `RunnableFuture`, a child of `Future`).
- `Runnable` has **no return value** (`Future.get()` always returns `null`); `Callable` **can return a value** natively — prefer `Callable` when you need output.
- `CompletableFuture` (Java 8+) extends `Future` and adds **chaining** via methods like `thenApply`, `thenCompose`, `thenAccept`, and `thenCombine`.
- The **sync vs. async** suffix (`thenApply` vs `thenApplyAsync`) determines whether the *same* thread or a *new* thread executes the next step.
- Without a custom executor, `CompletableFuture` defaults to the shared **ForkJoinPool**; pass your own `ThreadPoolExecutor` for full control.
- Use `thenCompose` (not `thenApply`) when chaining functions that themselves return a `CompletableFuture` — it flattens nested futures and preserves ordering.
- In real-world usage, `supplyAsync()` followed by a simple `get()` is by far the most common pattern; the advanced chaining methods (`thenCompose`, `thenCombine`) are used less often but remain important for interviews.

## Glossary of Terms
- **Async (Asynchronous) Task**: A unit of work executed independently of the calling thread's main flow, allowing the caller to continue without waiting.
- **Supplier**: A functional interface that takes no arguments and returns a value (`T get()`).
- **BiFunction**: A functional interface that takes two input arguments and produces one output.
- **ForkJoinPool**: Java's default shared thread pool used internally by `CompletableFuture` when no custom executor is specified; dynamically adjusts based on available processors.
- **Wildcard (`?`)**: A generic type placeholder meaning "unknown type," used when the exact type of a result isn't known or needs flexible bounds (e.g., `? super T`, `? extends CompletionStage`).`
