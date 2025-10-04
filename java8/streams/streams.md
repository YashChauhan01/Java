## 🚀 Java 8 Streams Notes 🚀

> A comprehensive overview of Java 8 Streams, covering their core concepts, operations, and benefits. Perfect for understanding how to leverage functional programming for efficient data processing!

---

### 🤔 What is a Stream?

* **Pipeline Analogy:** Think of a Stream as a **pipeline** through which your collection elements (data) pass.
* **Operations:** While elements pass through the pipeline, you can perform various operations like sorting, filtering, and mapping.
* **Bulk Processing:** Extremely useful for dealing with **bulk processing**, especially with the capability for **parallel processing**.

### 🧩 The Stream Pipeline: Three Key Steps

The processing of data using Streams typically involves three distinct steps:

1.  **🌀 Step 1: Create Stream (Source)**
    * Streams are **created from a data source**, such as a `Collection` (e.g., `List`, `Set`), or an `Array`.
    * This is the initial step where your data becomes a `Stream`.

2.  **⚙️ Step 2: Intermediate Operations**
    * These operations **transform the stream** into another stream.
    * Examples: `filter()`, `sorted()`, `map()`, `distinct()`, etc.
    * **Lazy in Nature:** Intermediate operations are **not executed immediately**. They are only executed when a **terminal operation** is invoked. This means they build a pipeline without actual processing until needed.

3.  **🎯 Step 3: Terminal Operations**
    * These operations **trigger the processing** of the entire stream pipeline.
    * Examples: `collect()`, `reduce()`, `count()`, `forEach()`, `toArray()`, `min()`, `max()`, `anyMatch()`, `allMatch()`, `noneMatch()`, `findFirst()`, `findAny()`.
    * **Final Output:** After a terminal operation is used, the stream is considered **consumed/closed** and cannot be used again.

---

### 📝 Different Ways to Create a Stream

1.  **From a `Collection`:**
    ```java
    List<Integer> salaryList = Arrays.asList(3000, 4100, 9000, 1000, 3500);
    Stream<Integer> streamFromIntegerList = salaryList.stream();
    ```

2.  **From an `Array`:**
    ```java
    Integer[] salaryArray = {3000, 4100, 9000, 1000, 3500};
    Stream<Integer> streamFromIntegerArray = Arrays.stream(salaryArray);
    ```

3.  **From a `Static Method` (e.g., `Stream.of()`):**
    ```java
    Stream<Integer> streamFromStaticMethod = Stream.of(1000, 3500, 4000, 9000);
    ```

4.  **From `Stream.Builder`:**
    ```java
    Stream.Builder<Integer> streamBuilder = Stream.builder();
    streamBuilder.add(1000).add(9000).add(3500);
    Stream<Integer> streamFromStreamBuilder = streamBuilder.build();
    ```

5.  **From `Stream.iterate()`:** (For generating infinite sequences)
    ```java
    // Creates an infinite sequential ordered stream
    Stream<Integer> streamFromIterate = Stream.iterate(1000, n -> n + 5000).limit(5);
    // Output (after collecting): 1000, 6000, 11000, 16000, 21000
    ```

---

### 🛠️ Intermediate Operations

| No. | Intermediate Operation | Description |
| :-- | :--------------------- | :---------- |
| 1.  | `filter(Predicate<T> predicate)` | **Filters** elements based on a `Predicate`. |
| 2.  | `map(Function<T, R> mapper)` | **Transforms** each element to a new value. |
| 3.  | `flatMap(Function<T, Stream<R>> mapper)` | **Flattens** a stream of streams into a single stream. |
| 4.  | `distinct()` | **Removes duplicate** elements. |
| 5.  | `sorted()` / `sorted(Comparator<T> comparator)` | **Sorts** elements in natural or custom order. |
| 6.  | `peek(Consumer<T> action)` | **Inspects** elements for debugging without modifying them. |
| 7.  | `limit(long maxSize)` | **Truncates** the stream to a maximum size. |
| 8.  | `skip(long n)` | **Skips** the first `n` elements. |
| 9.  | `mapToInt()` / `mapToLong()` / `mapToDouble()` | Works with **primitive** `int`, `long`, and `double` types. |

---

### 🏁 Terminal Operations

| No. | Terminal Operation | Description |
| :-- | :----------------- | :---------- |
| 1.  | `forEach(Consumer<T> action)` | Performs an action on **each element**. Does not return a value. |
| 2.  | `toArray()` | **Collects** elements into an `Array`. |
| 3.  | `reduce(BinaryOperator<T> accumulator)` | **Reduces** elements to a single value (e.g., sum, min, max). |
| 4.  | `collect(Collector<T, A, R> collector)` | **Collects** elements into a `List`, `Set`, `Map`, etc. |
| 5.  | `min(Comparator<T> comparator)` / `max(...)` | Finds the **minimum** or **maximum** element. |
| 6.  | `count()` | Returns the **count** of elements. |
| 7.  | `anyMatch(Predicate<T> predicate)` | Checks if **any** element matches a condition. |
| 8.  | `allMatch(Predicate<T> predicate)` | Checks if **all** elements match a condition. |
| 9.  | `noneMatch(Predicate<T> predicate)` | Checks if **no** elements match a condition. |
| 10. | `findFirst()` | Finds the **first** element. |
| 11. | `findAny()` | Finds **any** element (useful in parallel streams). |

---

### ⚡ Parallel Stream

* Helps perform operations **concurrently** using multi-core CPUs for better performance on large datasets.
* Use `collection.parallelStream()` instead of `collection.stream()`.
* Internally uses the **Fork-Join Pool** framework to split the task into smaller sub-tasks, process them in parallel, and then join the results.

---

### ⚠️ Important: Stream is Consumed Once!

* A single stream can only have **one terminal operation**.
* Once a terminal operation is used, the stream is **closed/consumed** and cannot be used again.
* Attempting to reuse a stream will result in an `IllegalStateException`.
