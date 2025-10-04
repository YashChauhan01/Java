# 🚀 Java 8 Streams: Visually Enhanced Notes 🚀

> A comprehensive overview of Java 8 Streams, covering their core concepts, operations, and benefits. Perfect for understanding how to leverage functional programming for efficient data processing!

---

## 📚 Table of Contents

1. [What is a Stream?](#-what-is-a-stream)
2. [The Stream Pipeline: Three Key Steps](#-the-stream-pipeline-three-key-steps)
3. [Traditional vs. Stream API Example](#-example-traditional-vs-stream-api-for-filtering)
4. [Different Ways to Create a Stream](#-different-ways-to-create-a-stream)
5. [Intermediate Operations](#-different-intermediate-operations)
6. [Terminal Operations](#-different-terminal-operations)
7. [Why Intermediate Operations are "Lazy"](#-why-intermediate-operations-are-lazy)
8. [Sequence of Stream Operations](#-sequence-of-stream-operations)
9. [Parallel Streams](#-parallel-stream)
10. [Important: Stream is Consumed Once!](#-important-stream-is-consumed-once)

---

## 🤔 What is a Stream?

**Pipeline Analogy**: Think of a Stream as a **pipeline** through which your collection elements (data) pass.

### Key Concepts

- **Operations**: While elements pass through the pipeline, you can perform various operations like sorting, filtering, and mapping.
- **Bulk Processing**: Extremely useful for dealing with bulk processing, especially with the capability for parallel processing.

---

## 🧩 The Stream Pipeline: Three Key Steps

The processing of data using Streams typically involves three distinct steps:

### 🌀 Step 1: Create Stream (Source)

- Streams are created from a data source, such as a **Collection** (e.g., List, Set), or an **Array**.
- This is the initial step where your data becomes a Stream.

### ⚙️ Step 2: Intermediate Operations

- These operations **transform the stream** into another stream.
- **Examples**: `filter()`, `sorted()`, `map()`, `distinct()`, etc.
- **Lazy in Nature**: Intermediate operations are not executed immediately. They are only executed when a terminal operation is invoked. This means they build a pipeline without actual processing until needed.

### 🎯 Step 3: Terminal Operations

- These operations **trigger the processing** of the entire stream pipeline.
- **Examples**: `collect()`, `reduce()`, `count()`, `forEach()`, `toArray()`, `min()`, `max()`, `anyMatch()`, `allMatch()`, `noneMatch()`, `findFirst()`, `findAny()`.
- **Final Output**: After a terminal operation is used, the stream is considered consumed/closed and cannot be used again.

---

## 💡 Example: Traditional vs. Stream API for Filtering

### Traditional Approach

```java
public class StreamExample {
    public static void main(String args[]) {
        List<Integer> salaryList = new ArrayList<>();
        salaryList.add(3000);
        salaryList.add(4100);
        salaryList.add(9000);
        salaryList.add(1000);
        salaryList.add(3500);

        int count = 0;
        for (Integer sal : salaryList) {
            if (sal > 3000) {
                count++;
            }
        }
        System.out.println("Total Employee with salary > 3000: " + count);
    }
}
// Output: Total Employee with salary > 3000: 3
```

### Using Stream API ✨

```java
public class StreamExample {
    public static void main(String args[]) {
        List<Integer> salaryList = new ArrayList<>();
        salaryList.add(3000);
        salaryList.add(4100);
        salaryList.add(9000);
        salaryList.add(1000);
        salaryList.add(3500);

        long output = salaryList.stream()                // Step 1: Create Stream
                .filter(sal -> sal > 3000)   // Step 2: Intermediate Operation (Lazy)
                .count();                    // Step 3: Terminal Operation (Triggers execution)

        System.out.println("Total Employee with salary > 3000: " + output);
    }
}
// Output: Total Employee with salary > 3000: 3
```

**Key Advantage**: More concise, readable, and functional approach!

---

## 📝 Different Ways to Create a Stream

### 1. From a Collection

```java
List<Integer> salaryList = Arrays.asList(3000, 4100, 9000, 1000, 3500);
Stream<Integer> streamFromIntegerList = salaryList.stream();
```

### 2. From an Array

```java
Integer[] salaryArray = {3000, 4100, 9000, 1000, 3500};
Stream<Integer> streamFromIntegerArray = Arrays.stream(salaryArray);
```

### 3. From a Static Method (`Stream.of()`)

```java
Stream<Integer> streamFromStaticMethod = Stream.of(1000, 3500, 4000, 9000);
```

### 4. From Stream.Builder

```java
Stream.Builder<Integer> streamBuilder = Stream.builder();
streamBuilder.add(1000).add(9000).add(3500);
Stream<Integer> streamFromStreamBuilder = streamBuilder.build();
```

### 5. From Stream.iterate() (For generating infinite sequences)

```java
// Creates an infinite sequential ordered stream
Stream<Integer> streamFromIterate = Stream.iterate(1000, n -> n + 5000).limit(5);
// Output (after collecting): 1000, 6000, 11000, 16000, 21000
```

---

## 🛠️ Different Intermediate Operations

Intermediate operations are chained together to perform complex processing before a terminal operation produces the result.

| No. | Intermediate Operation | Description | Example |
|-----|------------------------|-------------|---------|
| 1 | `filter(Predicate<T> predicate)` | Filters elements based on a given Predicate (a boolean-returning function). | `Stream<String> nameStream = Stream.of("HELLO", "EVERYBODY", "HOW", "ARE", "YOU", "DOING");`<br>`Stream<String> filteredNameStream = nameStream.filter(name -> name.length() <= 3);`<br>`List<String> filteredNames = filteredNameStream.collect(Collectors.toList());`<br>**OUTPUT**: `[HOW, ARE, YOU]` |
| 2 | `map(Function<T, R> mapper)` | Transforms each element in the stream to a new type or value using a Function. | `Stream<String> nameStream = Stream.of("HELLO", "EVERYBODY", "HOW", "ARE", "YOU", "DOING");`<br>`Stream<String> transformedNames = nameStream.map(name -> name.toLowerCase());`<br>**OUTPUT**: `[hello, everybody, how, are, you, doing]` |
| 3 | `flatMap(Function<T, Stream<R>> mapper)` | Flattens a stream of streams into a single stream. Used to iterate over each element of a complex collection and flatten it. | `List<List<String>> sentenceList = Arrays.asList(`<br>&nbsp;&nbsp;`Arrays.asList("I", "LOVE", "JAVA"),`<br>&nbsp;&nbsp;`Arrays.asList("CONCEPTS", "ARE", "CLEAR"),`<br>&nbsp;&nbsp;`Arrays.asList("ITS", "VERY", "EASY")`<br>`);`<br>`Stream<String> wordsStream = sentenceList.stream()`<br>&nbsp;&nbsp;`.flatMap(sentence -> sentence.stream());`<br>`Stream<String> lowerCaseWords = wordsStream.map(String::toLowerCase);`<br>**OUTPUT**: `[i, love, java, concepts, are, clear, its, very, easy]` |
| 4 | `distinct()` | Removes duplicate elements from the stream. | `Integer[] arr = {1, 5, 2, 7, 4, 4, 2, 0, 9};`<br>`Stream<Integer> arrStream = Arrays.stream(arr).distinct();`<br>**OUTPUT**: `[1, 5, 2, 7, 4, 0, 9]` |
| 5 | `sorted()` / `sorted(Comparator<T> comparator)` | Sorts the elements in natural order or using a custom Comparator. | `Integer[] arr = {1, 5, 2, 7, 4, 4, 2, 0, 9};`<br>`Stream<Integer> arrStream = Arrays.stream(arr).sorted();`<br>**OUTPUT (natural order)**: `[0, 1, 2, 2, 4, 4, 5, 7, 9]`<br><br>Custom sort (descending):<br>`Stream<Integer> arrStreamDesc = Arrays.stream(arr).sorted((val1, val2) -> val2 - val1);`<br>**OUTPUT (descending)**: `[9, 7, 5, 4, 4, 2, 2, 1, 0]` |
| 6 | `peek(Consumer<T> action)` | Helps to see the intermediate result of the stream processing. It's a non-interfering operation for debugging. | `List<Integer> numbers = Arrays.asList(2, 1, 4, 7, 10);`<br>`Stream<Integer> numbersStream = numbers.stream()`<br>&nbsp;&nbsp;`.filter(val -> val > 2)`<br>&nbsp;&nbsp;`.peek(val -> System.out.println("after filter: " + val));`<br>**(Output will print values > 2 as they pass through filter)** |
| 7 | `limit(long maxSize)` | Truncates the stream to contain no more than maxSize elements. | `List<Integer> numbers = Arrays.asList(2, 1, 3, 4, 6);`<br>`Stream<Integer> truncatedStream = numbers.stream().limit(3);`<br>**OUTPUT**: `[2, 1, 3]` |
| 8 | `skip(long n)` | Skips the first n elements of the stream. | `List<Integer> numbers = Arrays.asList(2, 1, 3, 4, 6);`<br>`Stream<Integer> skippedStream = numbers.stream().skip(3);`<br>**OUTPUT**: `[4, 6]` |
| 9 | `mapToInt()` / `mapToLong()` / `mapToDouble()` | Helps to work with primitive int, long, and double data types respectively. | `List<String> stringNumbers = Arrays.asList("2", "1", "4", "7");`<br>`IntStream intStream = stringNumbers.stream().mapToInt(Integer::parseInt);`<br>To convert IntStream back to Array:<br>`int[] numbersArray = intStream.toArray();`<br>**OUTPUT for numbersArray**: `{2, 1, 4, 7}` |

---

## 🏁 Different Terminal Operations

Terminal operations produce a final result and mark the end of the stream pipeline.

| No. | Terminal Operation | Description | Example |
|-----|-------------------|-------------|---------|
| 1 | `forEach(Consumer<T> action)` | Performs an action on each element of the stream. **DOES NOT return any value**. | `numbers.stream().filter(val -> val > 3).forEach(System.out::println);`<br>**OUTPUT**: `4, 7, 10` |
| 2 | `toArray()` | Collects the elements of the stream into an Array. | `Integer[] filteredNumberArr = numbers.stream()`<br>&nbsp;&nbsp;`.filter(val -> val > 3)`<br>&nbsp;&nbsp;`.toArray(Integer[]::new);`<br>**OUTPUT**: `{4, 7, 10}` |
| 3 | `reduce(BinaryOperator<T> accumulator)` | Does reduction on the elements of the stream. Performs an associative aggregation function. | `Optional<Integer> reducedValue = numbers.stream()`<br>&nbsp;&nbsp;`.reduce((val1, val2) -> val1 + val2);`<br>`System.out.println(reducedValue.get());`<br>**OUTPUT**: `24` (2+1+4+7+10) |
| 4 | `collect(Collector<T, A, R> collector)` | Can be used to collect the elements of the stream into a List, Set, Map, etc. | `List<Integer> filteredNumbers = numbers.stream()`<br>&nbsp;&nbsp;`.filter(val -> val > 3)`<br>&nbsp;&nbsp;`.collect(Collectors.toList());`<br>**OUTPUT**: `[4, 7, 10]` |
| 5 | `min(Comparator<T> comparator)` / `max(Comparator<T> comparator)` | Finds the minimum or maximum element from the stream based on the Comparator provided. | `Optional<Integer> minValue = numbers.stream()`<br>&nbsp;&nbsp;`.filter(val -> val > 3)`<br>&nbsp;&nbsp;`.min((val1, val2) -> val1 - val2);`<br>`System.out.println(minValue.get());`<br>**OUTPUT**: `4`<br><br>`Optional<Integer> maxValue = numbers.stream()`<br>&nbsp;&nbsp;`.filter(val -> val > 3)`<br>&nbsp;&nbsp;`.max((val1, val2) -> val1 - val2);`<br>`System.out.println(maxValue.get());`<br>**OUTPUT**: `10` |
| 6 | `count()` | Returns the count of elements present in the stream. | `long noOfValuesPresent = numbers.stream()`<br>&nbsp;&nbsp;`.filter(val -> val > 3)`<br>&nbsp;&nbsp;`.count();`<br>`System.out.println(noOfValuesPresent);`<br>**OUTPUT**: `3` |
| 7 | `anyMatch(Predicate<T> predicate)` | Checks if any value in the stream matches the given Predicate and returns a boolean. | `boolean hasValueGreaterThanThree = numbers.stream()`<br>&nbsp;&nbsp;`.anyMatch(val -> val > 3);`<br>`System.out.println(hasValueGreaterThanThree);`<br>**OUTPUT**: `true` |
| 8 | `allMatch(Predicate<T> predicate)` | Checks if all values in the stream match the given Predicate and returns a boolean. | `boolean allValuesGreaterThanZero = numbers.stream()`<br>&nbsp;&nbsp;`.allMatch(val -> val > 0);`<br>`System.out.println(allValuesGreaterThanZero);`<br>**OUTPUT**: `true` (assuming no zeros) |
| 9 | `noneMatch(Predicate<T> predicate)` | Checks if no value in the stream matches the given Predicate and returns a boolean. | `boolean noNegativeValues = numbers.stream()`<br>&nbsp;&nbsp;`.noneMatch(val -> val < 0);`<br>`System.out.println(noNegativeValues);`<br>**OUTPUT**: `true` (assuming no negative numbers) |
| 10 | `findFirst()` | Finds the first element of the stream (if present, wrapped in an Optional). | `Optional<Integer> firstValue = numbers.stream()`<br>&nbsp;&nbsp;`.filter(val -> val > 3)`<br>&nbsp;&nbsp;`.findFirst();`<br>`System.out.println(firstValue.get());`<br>**OUTPUT**: `4` |
| 11 | `findAny()` | Finds any random element of the stream (if present, wrapped in an Optional). Useful for parallel streams where order doesn't matter. | `Optional<Integer> anyValue = numbers.stream()`<br>&nbsp;&nbsp;`.filter(val -> val > 3)`<br>&nbsp;&nbsp;`.findAny();`<br>`System.out.println(anyValue.get());`<br>**OUTPUT**: could be `4`, `7`, or `10` depending on execution |

---

## 🧐 Why Intermediate Operations are "Lazy"

Intermediate operations do not execute until a terminal operation is invoked. They build a chain of operations.

**Example**: If you have a `filter()` and then a `peek()` operation, `peek()` (which prints) won't show any output until a terminal operation like `count()` is called later in the pipeline.

### Code Example

```java
public class StreamExample {
    public static void main(String args[]) {
        List<Integer> numbers = Arrays.asList(2, 1, 4, 7, 10);

        // Example 1: No Terminal Operation - Nothing is printed
        Stream<Integer> numbersStream = numbers.stream()
                .filter(val -> val > 3)
                .peek(val -> System.out.println(val)); // This will NOT print
        // Output: Nothing would be printed in the Output

        // Example 2: With Terminal Operation - Values are printed
        long count = numbers.stream()
                .filter(val -> val > 3)
                .peek(val -> System.out.println(val)) // Now this will print
                .count(); // Terminal operation
        // Output: 4, 7, 10 (as they pass through the filter and peek)
    }
}
```

---

## ➡️ Sequence of Stream Operations

Elements generally process **sequentially** through the pipeline. This means an element goes through all intermediate operations before the next element starts.

### Code Example

```java
public class StreamExample {
    public static void main(String args[]) {
        List<Integer> numbers = Arrays.asList(2, 1, 4, 7, 10);
        List<Integer> filteredNumberStream = numbers.stream()
                .filter(val -> val > 3) // Filter values > 3
                .peek(val -> System.out.println("after filter: " + val)) // Print after filter
                .map(val -> val * -1) // Negate the value
                .peek(val -> System.out.println("after negating: " + val)) // Print after negating
                .sorted() // Sort the remaining elements
                .peek(val -> System.out.println("after sorted: " + val)) // Print after sorting
                .collect(Collectors.toList()); // Collect into a list

        System.out.println(filteredNumberStream);
    }
}
```

### Expected Output vs. Actual Output

| Expected Output (Sequential Thinking) | Actual Output (Stream Execution) |
|---------------------------------------|----------------------------------|
| after filter: 4 | after filter: 4 |
| after filter: 7 | after negating: -4 |
| after filter: 10 | after filter: 7 |
| after negating: -4 | after negating: -7 |
| after negating: -7 | after filter: 10 |
| after negating: -10 | after negating: -10 |
| after sorted: -10 | after sorted: -10 |
| after sorted: -7 | after sorted: -7 |
| after sorted: -4 | after sorted: -4 |
| `[-10, -7, -4]` | `[-10, -7, -4]` |

### Key Insight

Each element (e.g., `4`) passes through `filter`, then `peek`, then `map`, then `peek`, and so on. It doesn't mean all filtering finishes first, then all mapping, etc.

**Exception**: Operations like `sorted()` require all elements to be present in the stream before they can execute.

---

## ⚡ Parallel Stream

Helps to perform operation on stream **concurrently**, taking advantage of multi-core CPUs.

- `parallelStream()` method is used instead of `stream()` method for Collection (or `Stream.parallel()` for Stream).

### How it Works Internally

1. **Task Splitting**: Uses a "spliterator" function to split the data into multiple chunks.
2. **Task Submission & Parallel Processing**: Uses the Fork-Join Pool technique.

### Example: Sequential vs. Parallel Processing Time

```java
public class StreamExample {
    public static void main(String args[]) {
        List<Integer> numbers = Arrays.asList(11, 22, 33, 44, 55, 66, 77, 88, 99, 110);

        // Sequential processing
        long sequentialProcessingStartTime = System.currentTimeMillis();
        numbers.stream()
                .map(val -> val * val)
                .forEach(val -> System.out.println(val)); // Consumes the stream
        System.out.println("Sequential Processing Time Taken: " +
                (System.currentTimeMillis() - sequentialProcessingStartTime) + " millisecond");

        // Parallel processing
        long parallelProcessingStartTime = System.currentTimeMillis();
        numbers.parallelStream() // Only change is .parallelStream()
                .map(val -> val * val)
                .forEach(val -> System.out.println(val)); // Consumes the stream
        System.out.println("Parallel Processing Time Taken: " +
                (System.currentTimeMillis() - parallelProcessingStartTime) + " millisecond");
    }
}
```

**Output:**
```
121
484
1089
1936
3025
4356
5929
7744
9801
12100
Sequential Processing Time Taken: 64 millisecond

121
484
1089
1936
3025
4356
5929
7744
9801
12100
Parallel Processing Time Taken: 5 millisecond
```

**Observation**: Parallel processing is significantly faster for larger datasets due to concurrent execution.

### Fork-Join Pool Technique (Conceptual)

1. A large **Task** is recursively **forked** (divided) into smaller sub-tasks.
2. These sub-tasks are processed independently and concurrently.
3. The results from the sub-tasks are then **joined** together to produce the final output.

---

## ⚠️ Important: Stream is Consumed Once!

A single stream can only have **one terminal operation**.

- Once a terminal operation is used on a stream, it is considered **closed/consumed** and cannot be used again for another terminal operation.
- If you try to use a consumed stream, you will get an `IllegalStateException`.

### Code Example

```java
public class StreamExample {
    public static void main(String args[]) {
        List<Integer> numbers = Arrays.asList(2, 1, 4, 7, 10);
        Stream<Integer> filteredNumbersStream = numbers.stream()
                .filter(val -> val > 3);

        // First terminal operation (consumes the stream)
        filteredNumbersStream.forEach(val -> System.out.println(val));
        // Output: 4, 7, 10

        // Trying to use the closed stream again (will throw exception)
        filteredNumbersStream.collect(Collectors.toList());
        // Output: java.lang.IllegalStateException: stream has already been operated upon or closed
    }
}
```

---

## 🎯 Quick Reference Summary

### Stream Pipeline Structure

```
Source (Collection/Array) 
    → stream() or parallelStream()
    → Intermediate Operations (filter, map, sorted, etc.) [Lazy]
    → Terminal Operation (collect, forEach, count, etc.) [Triggers execution]
    → Result
```

### Key Points to Remember

✅ **Streams are lazy** - Intermediate operations don't execute until a terminal operation is called

✅ **Streams are consumed once** - After a terminal operation, the stream cannot be reused

✅ **Sequential processing** - Elements pass through the pipeline one at a time (except for operations like `sorted()`)

✅ **Parallel streams** - Use `parallelStream()` for concurrent processing on multi-core systems

✅ **Functional approach** - Streams promote cleaner, more readable code compared to traditional loops

---

## 🎉 Conclusion

Java 8 Streams provide a powerful, functional approach to data processing:

- **Declarative**: Focus on *what* to do, not *how* to do it
- **Readable**: Clean, expressive code that's easy to understand
- **Efficient**: Built-in support for parallel processing
- **Composable**: Chain multiple operations together seamlessly

### Best Practices

1. Use streams for bulk data operations
2. Prefer method references over lambda expressions when possible
3. Use parallel streams only for large datasets and CPU-intensive operations
4. Avoid side effects in stream operations
5. Remember that streams are consumed after terminal operations

---

<div align="center">

## 🚀 Master Java 8 Streams!

**"Write cleaner, more efficient Java code with the power of streams"** ✨

⭐ Happy Streaming! ⭐

</div>