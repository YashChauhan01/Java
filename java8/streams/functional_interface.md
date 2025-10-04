# ☕ Java Functional Interfaces & Lambda Expressions ☕

> A comprehensive guide to understanding the core concepts of functional programming in Java 8, making your code more concise, readable, and powerful.

---

## 📚 Table of Contents

1. [What are Functional Interfaces?](#-what-are-functional-interfaces)
2. [Lambda Expressions](#-lambda-expressions)
3. [Common Built-in Functional Interfaces](#-common-built-in-functional-interfaces)
4. [Method References](#-method-references-the-ultimate-shortcut)
5. [Practical Examples](#-putting-it-all-together-practical-examples)
6. [Java Streams API Guide](#-java-streams-api-complete-guide-)
7. [Real-World Applications](#-real-world-applications)
8. [Cheat Sheet](#-cheat-sheet)

---

## 🤔 What are Functional Interfaces?

A **Functional Interface** is simply an interface that contains **exactly one abstract method**. They are the foundation for lambda expressions and a key part of Java's move towards functional programming.

### Key Points

- **Annotation**: You can use the `@FunctionalInterface` annotation to mark an interface. This is optional but highly recommended, as it allows the compiler to verify that the interface has only one abstract method.
- **Purpose**: They act as a target type for lambda expressions and method references.

### Traditional Implementation (Before Java 8)

Before lambdas, you would use an anonymous inner class to implement a functional interface.

**1. Defining the Interface:**

```java
@FunctionalInterface
interface Calculator {
    int operate(int a, int b);
}
```

**2. Implementing with an Anonymous Class:**

```java
public class Main {
    public static void main(String[] args) {
        // Traditional anonymous inner class implementation
        Calculator addition = new Calculator() {
            @Override
            public int operate(int a, int b) {
                return a + b;
            }
        };
        
        System.out.println("Result: " + addition.operate(5, 3)); // Output: Result: 8
    }
}
```

---

## ⚡ Lambda Expressions

A **Lambda Expression** is a short, anonymous function that can be used to provide an implementation for a functional interface. It's a way to write code that is more concise and expressive.

> **Analogy**: Think of it as a shortcut for writing a method. You don't need a name, return type, or public/private modifiers.

### Syntax Breakdown

```
(parameters) -> { body }
```

- **`(parameters)`**: A list of parameters for the method
    - You can omit the data types (e.g., `(a, b)` instead of `(int a, int b)`)
    - If there's only one parameter, you can omit the parentheses (e.g., `n` instead of `(n)`)

- **`->`**: The "arrow token" that separates the parameters from the body

- **`{ body }`**: The code that implements the abstract method
    - If the body is a single expression, you can omit the curly braces `{}` and the `return` keyword

### Modern Implementation (With Lambda Expressions)

```java
public class Main {
    public static void main(String[] args) {
        // Lambda expression implementation (concise and clean!)
        Calculator addition = (a, b) -> a + b;
        Calculator subtraction = (a, b) -> a - b;
        Calculator multiplication = (a, b) -> a * b;
        Calculator division = (a, b) -> a / b;
        
        System.out.println("Addition: " + addition.operate(5, 3));       // Output: 8
        System.out.println("Subtraction: " + subtraction.operate(5, 3)); // Output: 2
        System.out.println("Multiplication: " + multiplication.operate(5, 3)); // Output: 15
    }
}
```

**Notice**: 5 lines of code for the anonymous class were reduced to a single, readable line!

### Lambda Expression Variations

```java
// No parameters
Runnable task = () -> System.out.println("Hello World");

// Single parameter (parentheses optional)
Consumer<String> printer = message -> System.out.println(message);

// Multiple parameters
Comparator<String> comparator = (s1, s2) -> s1.compareTo(s2);

// Multiple statements in body
Calculator complexOperation = (a, b) -> {
    int result = a + b;
    System.out.println("Calculating: " + a + " + " + b);
    return result;
};
```

---

## 🛠️ Common Built-in Functional Interfaces

Java provides a set of pre-defined functional interfaces in the `java.util.function` package so you don't always have to create your own.

| Interface | Abstract Method | Purpose | Use Case |
|-----------|----------------|---------|----------|
| `Predicate<T>` | `boolean test(T t)` | Takes one argument and returns a boolean | Filtering data (e.g., `stream.filter(...)`) |
| `Function<T, R>` | `R apply(T t)` | Takes one argument (T) and returns a result (R) | Transforming/mapping data (e.g., `stream.map(...)`) |
| `Consumer<T>` | `void accept(T t)` | Takes one argument and returns nothing (void) | Performing an action on an element (e.g., `stream.forEach(...)`) |
| `Supplier<T>` | `T get()` | Takes no arguments but returns a value | Generating or supplying values |
| `BinaryOperator<T>` | `T apply(T t1, T t2)` | Takes two arguments of the same type and returns a result of that same type | Aggregating or reducing elements (e.g., `stream.reduce(...)`) |
| `UnaryOperator<T>` | `T apply(T t)` | Takes one argument and returns result of same type | Single operand operations |
| `BiFunction<T,U,R>` | `R apply(T t, U u)` | Takes two arguments and returns a result | Two-argument transformations |

### Examples of Built-in Functional Interfaces

```java
import java.util.function.*;

public class FunctionalInterfaceExamples {
    public static void main(String[] args) {
        
        // Predicate - tests a condition
        Predicate<String> isLong = s -> s.length() > 5;
        System.out.println("Is 'Hello' long? " + isLong.test("Hello")); // false
        
        // Function - transforms input to output
        Function<String, Integer> stringLength = s -> s.length();
        System.out.println("Length: " + stringLength.apply("Java")); // 4
        
        // Consumer - consumes without returning
        Consumer<String> printer = s -> System.out.println("Value: " + s);
        printer.accept("Hello World");
        
        // Supplier - provides values
        Supplier<Double> randomSupplier = () -> Math.random();
        System.out.println("Random: " + randomSupplier.get());
        
        // BinaryOperator - combines two values
        BinaryOperator<Integer> adder = (a, b) -> a + b;
        System.out.println("Sum: " + adder.apply(5, 3)); // 8
    }
}
```

---

## ✨ Method References: The Ultimate Shortcut

A **Method Reference** is a compact, easy-to-read shorthand for a lambda expression that only calls an existing method.

| Type | Syntax | Lambda Equivalent | Example |
|------|--------|-------------------|---------|
| Static Method | `ClassName::methodName` | `(args) -> ClassName.methodName(args)` | `stringList.stream().map(Integer::parseInt)` |
| Instance Method (of a specific object) | `object::methodName` | `(args) -> object.methodName(args)` | `System.out::println` |
| Instance Method (of an arbitrary object) | `ClassName::methodName` | `(obj, args) -> obj.methodName(args)` | `stringList.stream().map(String::toLowerCase)` |
| Constructor | `ClassName::new` | `(args) -> new ClassName(args)` | `names.stream().map(StringBuilder::new)` |

### Method Reference Examples

```java
import java.util.*;
import java.util.function.*;

public class MethodReferenceExamples {
    public static void main(String[] args) {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
        
        // Static method reference
        names.forEach(System.out::println);
        
        // Instance method reference of arbitrary object
        List<String> upperCaseNames = names.stream()
            .map(String::toUpperCase)
            .toList();
            
        // Constructor reference
        Supplier<List<String>> listSupplier = ArrayList::new;
        List<String> newList = listSupplier.get();
        
        // Instance method reference of specific object
        String prefix = "Hello ";
        Function<String, String> greeter = prefix::concat;
        System.out.println(greeter.apply("World")); // Hello World
    }
}
```

---

## 🚀 Putting It All Together: Practical Examples

### Example 1: Name Processing

Let's use what we've learned to process a list of names. We'll filter for names longer than 4 characters, convert them to uppercase, and then print them.

```java
import java.util.Arrays;
import java.util.List;
import java.util.function.Predicate;
import java.util.function.Function;
import java.util.stream.Collectors;

public class FunctionalExample {
    public static void main(String[] args) {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "Dave", "Eve");
        
        // 1. Define Predicate with a lambda to filter names
        Predicate<String> isLongerThanFour = name -> name.length() > 4;
        
        // 2. Define Function with a lambda to transform to uppercase
        Function<String, String> toUpperCase = name -> name.toUpperCase();
        
        // 3. Process the list using a stream
        List<String> processedNames = names.stream()
            .filter(isLongerThanFour)       // Use the Predicate
            .map(toUpperCase)               // Use the Function
            .collect(Collectors.toList());  // Terminal operation
        
        // 4. Print the results using a Consumer (via a method reference)
        processedNames.forEach(System.out::println);
    }
}
```

**Output:**
```
ALICE
CHARLIE
```

### Example 2: Employee Management System

```java
import java.util.*;
import java.util.function.*;
import java.util.stream.*;

class Employee {
    private String name;
    private String department;
    private double salary;
    private int age;
    
    public Employee(String name, String department, double salary, int age) {
        this.name = name;
        this.department = department;
        this.salary = salary;
        this.age = age;
    }
    
    // Getters
    public String getName() { return name; }
    public String getDepartment() { return department; }
    public double getSalary() { return salary; }
    public int getAge() { return age; }
    
    @Override
    public String toString() {
        return String.format("%s (%s) - $%.2f - %d years", 
            name, department, salary, age);
    }
}

public class EmployeeManagement {
    public static void main(String[] args) {
        List<Employee> employees = Arrays.asList(
            new Employee("John", "IT", 50000, 25),
            new Employee("Alice", "HR", 45000, 30),
            new Employee("Bob", "IT", 60000, 35),
            new Employee("Anna", "Finance", 55000, 28),
            new Employee("Alex", "IT", 52000, 32)
        );
        
        // Predicate: Employees in IT department
        Predicate<Employee> isIT = e -> "IT".equals(e.getDepartment());
        
        // Function: Extract employee name
        Function<Employee, String> getName = Employee::getName;
        
        // Consumer: Print employee details
        Consumer<Employee> printEmployee = System.out::println;
        
        // Supplier: Create default employee
        Supplier<Employee> defaultEmployee = () -> new Employee("Unknown", "General", 40000, 25);
        
        System.out.println("=== IT Employees ===");
        employees.stream()
            .filter(isIT)
            .forEach(printEmployee);
            
        System.out.println("\n=== Employee Names ===");
        List<String> names = employees.stream()
            .map(getName)
            .collect(Collectors.toList());
        names.forEach(System.out::println);
            
        System.out.println("\n=== High Salary Employees ===");
        employees.stream()
            .filter(e -> e.getSalary() > 50000)
            .forEach(printEmployee);
    }
}
```

---

## 📊 Java Streams API Complete Guide 🚀

### 🔄 Stream Pipeline

**Pipeline Structure:**
```
Source → Intermediate → Intermediate → Terminal
         Operations     Operations    Operation
   ↓          ↓             ↓           ↓
[Data] → [filter/map] → [sorted/etc] → [Result]
```

**Example:**
```java
List<String> result = list.stream()          // Source
    .filter(s -> s.length() > 3)            // Intermediate
    .map(String::toUpperCase)               // Intermediate  
    .sorted()                               // Intermediate
    .collect(Collectors.toList());          // Terminal
```

### 🏗️ Creating Streams

```java
// From Collections
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream = list.stream();

// From Arrays
String[] array = {"a", "b", "c"};
Stream<String> stream = Arrays.stream(array);

// Using Stream.of()
Stream<String> stream = Stream.of("a", "b", "c");

// Primitive Streams
IntStream.range(1, 5)        // 1, 2, 3, 4
IntStream.rangeClosed(1, 5)  // 1, 2, 3, 4, 5
```

### ⚡ Intermediate Operations

#### 1. filter() 🔍

```java
List<String> names = Arrays.asList("John", "Alice", "Bob", "Anna");
List<String> result = names.stream()
    .filter(name -> name.startsWith("A"))
    .collect(Collectors.toList());
// Result: ["Alice", "Anna"]
```

#### 2. map() 🗺️

```java
List<String> names = Arrays.asList("John", "Alice", "Bob");
List<String> upperCase = names.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
// Result: ["JOHN", "ALICE", "BOB"]
```

#### 3. flatMap() 📦

```java
List<List<String>> listOfLists = Arrays.asList(
    Arrays.asList("a", "b"),
    Arrays.asList("c", "d")
);
List<String> flatList = listOfLists.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());
// Result: ["a", "b", "c", "d"]
```

#### 4. distinct() 🚫

```java
List<Integer> numbers = Arrays.asList(1, 2, 2, 3, 3, 3, 4);
List<Integer> unique = numbers.stream()
    .distinct()
    .collect(Collectors.toList());
// Result: [1, 2, 3, 4]
```

#### 5. sorted() 📊

```java
List<String> names = Arrays.asList("John", "Alice", "Bob");
List<String> sorted = names.stream()
    .sorted()
    .collect(Collectors.toList());
// Result: ["Alice", "Bob", "John"]
```

### 🎯 Terminal Operations

#### 1. forEach() 🔄

```java
List<String> names = Arrays.asList("John", "Alice", "Bob");
names.stream().forEach(System.out::println);
```

#### 2. collect() 📥

```java
List<String> names = Arrays.asList("John", "Alice", "Bob");
List<String> list = names.stream().collect(Collectors.toList());
Set<String> set = names.stream().collect(Collectors.toSet());
```

#### 3. reduce() ➕

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
Optional<Integer> sum = numbers.stream()
    .reduce((a, b) -> a + b);
// Result: Optional[15]
```

#### 4. Matching Operations ✅❌

```java
List<String> names = Arrays.asList("John", "Alice", "Bob", "Anna");

boolean anyMatch = names.stream()
    .anyMatch(name -> name.startsWith("A")); // true

boolean allMatch = names.stream()
    .allMatch(name -> name.startsWith("A")); // false

boolean noneMatch = names.stream()
    .noneMatch(name -> name.startsWith("Z")); // true
```

### 📊 Collectors

#### 1. Joining 🔗

```java
List<String> names = Arrays.asList("John", "Alice", "Bob");
String joined = names.stream()
    .collect(Collectors.joining(", "));
// Result: "John, Alice, Bob"
```

#### 2. Grouping 📂

```java
List<String> names = Arrays.asList("John", "Alice", "Bob", "Anna", "Alex");
Map<Character, List<String>> grouped = names.stream()
    .collect(Collectors.groupingBy(name -> name.charAt(0)));
// Result: {'J': ["John"], 'A': ["Alice", "Anna", "Alex"], 'B': ["Bob"]}
```

#### 3. Partitioning ✂️

```java
List<String> names = Arrays.asList("John", "Alice", "Bob", "Anna");
Map<Boolean, List<String>> partitioned = names.stream()
    .collect(Collectors.partitioningBy(name -> name.startsWith("A")));
// Result: {false: ["John", "Bob"], true: ["Alice", "Anna"]}
```

#### 4. Summarizing 📈

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
IntSummaryStatistics stats = numbers.stream()
    .collect(Collectors.summarizingInt(Integer::intValue));
// stats: {count=5, sum=15, min=1, max=5, average=3.0}
```

---

## 🎯 Real-World Applications

### E-commerce Analytics Example

```java
import java.util.*;
import java.util.stream.*;
import java.time.*;

class Order {
    private String customerId;
    private List<OrderItem> items;
    private LocalDate orderDate;
    private double totalAmount;
    
    // Constructors, getters
    public String getCustomerId() { return customerId; }
    public double getTotalAmount() { return totalAmount; }
    public LocalDate getOrderDate() { return orderDate; }
}

public class EcommerceAnalytics {
    public static void main(String[] args) {
        List<Order> orders = // ... get orders
        
        // Total sales by month
        Map<YearMonth, Double> salesByMonth = orders.stream()
            .collect(Collectors.groupingBy(
                order -> YearMonth.from(order.getOrderDate()),
                Collectors.summingDouble(Order::getTotalAmount)
            ));
        
        // Top 5 customers
        List<String> topCustomers = orders.stream()
            .collect(Collectors.groupingBy(
                Order::getCustomerId,
                Collectors.summingDouble(Order::getTotalAmount)
            ))
            .entrySet().stream()
            .sorted(Map.Entry.<String, Double>comparingByValue().reversed())
            .limit(5)
            .map(Map.Entry::getKey)
            .collect(Collectors.toList());
    }
}
```

### File Processing Pipeline

```java
import java.io.IOException;
import java.nio.file.*;
import java.util.*;
import java.util.stream.*;

public class FileProcessor {
    public static void main(String[] args) {
        try {
            // Process log files and analyze errors
            Map<String, Long> errorAnalysis = Files.lines(Paths.get("application.log"))
                .filter(line -> line.contains("ERROR"))
                .map(line -> extractErrorType(line))
                .filter(Optional::isPresent)
                .map(Optional::get)
                .collect(Collectors.groupingBy(
                    errorType -> errorType,
                    Collectors.counting()
                ));
            
            // Print error statistics
            errorAnalysis.entrySet().stream()
                .sorted(Map.Entry.<String, Long>comparingByValue().reversed())
                .forEach(entry -> 
                    System.out.println(entry.getKey() + ": " + entry.getValue()));
                    
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    private static Optional<String> extractErrorType(String logLine) {
        // Implementation to extract error type from log line
        if (logLine.contains("NullPointer")) return Optional.of("NullPointerException");
        if (logLine.contains("Timeout")) return Optional.of("TimeoutException");
        return Optional.empty();
    }
}
```

---

## 📝 Cheat Sheet

### Functional Interfaces Quick Reference

| Interface | Method | Use Case | Example |
|-----------|--------|----------|---------|
| `Predicate<T>` | `test(T t)` | Filtering | `.filter(s -> s.isEmpty())` |
| `Function<T,R>` | `apply(T t)` | Transformation | `.map(String::length)` |
| `Consumer<T>` | `accept(T t)` | Side effects | `.forEach(System.out::println)` |
| `Supplier<T>` | `get()` | Value generation | `() -> new ArrayList<>()` |
| `BinaryOperator<T>` | `apply(T t1, T t2)` | Reduction | `.reduce(0, Integer::sum)` |

### Stream Operations Reference

| Operation | Type | Description |
|-----------|------|-------------|
| `filter()` | Intermediate | Filters elements based on predicate |
| `map()` | Intermediate | Transforms each element |
| `flatMap()` | Intermediate | Flattens nested structures |
| `distinct()` | Intermediate | Removes duplicates |
| `sorted()` | Intermediate | Sorts elements |
| `forEach()` | Terminal | Iterates over each element |
| `collect()` | Terminal | Collects to collection |
| `reduce()` | Terminal | Reduces to single value |
| `count()` | Terminal | Counts elements |

### Method Reference Types

| Type | Syntax | Example |
|------|--------|---------|
| Static Method | `Class::staticMethod` | `Integer::parseInt` |
| Instance Method | `object::instanceMethod` | `System.out::println` |
| Arbitrary Object | `Class::instanceMethod` | `String::toUpperCase` |
| Constructor | `Class::new` | `ArrayList::new` |

---

## Best Practices 🚀

1. ✅ Use method references over lambdas for better readability
2. ✅ Avoid side effects in stream operations
3. ✅ Use primitive streams (`IntStream`, `LongStream`) for better performance
4. ✅ Prefer `collect()` over complex `reduce()` operations
5. ✅ Use parallel streams only for CPU-intensive operations on large datasets

---

## 🎉 Conclusion

Java 8's functional programming features have revolutionized how we write Java code:

- ✅ **Concise**: Less boilerplate code with lambdas
- ✅ **Readable**: Method references make code self-documenting
- ✅ **Composable**: Stream operations can be chained together
- ✅ **Parallelizable**: Easy to leverage multi-core processors
- ✅ **Maintainable**: Functional style reduces bugs and side effects

### Next Steps

- Practice with different data structures and algorithms
- Explore advanced collectors and custom reductions
- Learn about reactive streams and Java 9+ enhancements
- Experiment with parallel streams on large datasets

---

<div align="center">

## 🚀 Happy Functional Programming!

**"Write better Java code with functional elegance"** ✨

⭐ Star this guide if you found it helpful! ⭐

</div>