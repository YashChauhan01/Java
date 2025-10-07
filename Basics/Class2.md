# Java Generics - Complete Guide

## Table of Contents

1. [Introduction](#introduction)
2. [The Problem: Why Do We Need Generics?](#the-problem-why-do-we-need-generics)
   - [The Old Way Without Generics](#the-old-way-without-generics)
   - [Major Drawbacks](#major-drawbacks)
3. [The Solution: Generic Classes](#the-solution-generic-classes)
   - [Benefits of Generics](#benefits-of-generics)
   - [Basic Usage Example](#basic-usage-example)
4. [Advanced Concepts](#advanced-concepts)
   - [Generic Methods](#generic-methods)
   - [Multiple Type Parameters](#multiple-type-parameters)
   - [Bounded Generics](#bounded-generics)
   - [Wildcards for Flexibility](#wildcards-for-flexibility)
5. [Type Erasure: How It Works](#type-erasure-how-it-works)
6. [Best Practices](#best-practices)
7. [Common Pitfalls](#common-pitfalls)

---

## Introduction

Java Generics is a powerful feature introduced in Java 5 that enables types (classes and interfaces) to be parameters when defining classes, interfaces, and methods. Generics provide compile-time type safety and eliminate the need for explicit type casting, making code more robust and easier to maintain.

---

## The Problem: Why Do We Need Generics?

### The Old Way Without Generics

Before generics, to create a reusable class that could hold any type of object, you had to use the `Object` class. Let's look at the `Print` class example to see the problems this caused.

```java
// The OLD way, without generics
public class Print {
    Object value;

    public Object getPrintValue() {
        return value;
    }

    public void setPrintValue(Object value) {
        this.value = value;
    }
}
```

### Major Drawbacks

This approach has two major drawbacks:

1. **No Type Safety:** The `setPrintValue` method accepts any `Object`. This means you can accidentally put the wrong type of data into your class. You might expect an `Integer`, but someone could pass a `String` like "hello," and the compiler wouldn't stop them.

2. **Requires Manual Typecasting:** When you call `getPrintValue()`, it returns a generic `Object`. To use it as its original type, you must manually cast it. This is risky because if you cast to the wrong type, your program will crash with a `ClassCastException` at runtime.

```java
Print printObj1 = new Print();
printObj1.setPrintValue(1); // Storing an Integer

// We get an Object back, not an Integer
Object printValue = printObj1.getPrintValue(); 

// DANGEROUS: We have to manually cast it.
// This will throw an error if printValue was not an Integer.
if (((int)printValue) == 1) { 
    // do something
}
```

---

## The Solution: Generic Classes 🚀

**Generics** solve these problems by allowing us to create classes and methods that work with different types while providing **compile-time type safety**. We use a **type parameter** (a placeholder, commonly `T`) in angle brackets `<>` to make a class generic.

```java
// The NEW way, with generics
public class Print<T> {
    T value; // "T" is a placeholder for a future data type

    public T getPrintValue() {
        return value;
    }

    public void setPrintValue(T value) {
        this.value = value;
    }
}
```

### Benefits of Generics

Now, when we create an object, we specify the type we want to use. This brings two huge benefits:

1. **Compile-Time Safety:** The compiler enforces the type. If you declare a `Print<Integer>`, you can *only* pass it `Integer` values.

2. **No More Typecasting:** When you get the value back, it's already the correct type, so no casting is needed.

### Basic Usage Example

```java
// We declare that this Print object will only work with Integers
Print<Integer> printObj1 = new Print<Integer>();

printObj1.setPrintValue(1); // This is fine
// printObj1.setPrintValue("hello"); // COMPILE ERROR! Type safety in action.

// We get an Integer back directly, no casting needed!
Integer printValue = printObj1.getPrintValue();

if (printValue == 1) {
    // do something
}
```

**Important Note:** Generic types can only be **non-primitive** object types (e.g., `Integer`, `String`, `Double`), not primitive types (`int`, `double`).

---

## Advanced Concepts

### Generic Methods

You can make a single method generic without making the whole class generic. The type parameter `<T>` is declared before the method's return type.

```java
public <T> void setValue(T busObject) {
    // do something
}
```

**Example with return type:**

```java
public <T> T getFirstElement(List<T> list) {
    if (list.isEmpty()) {
        return null;
    }
    return list.get(0);
}
```

### Multiple Type Parameters

A class can have multiple generic types, like a key-value pair.

```java
public class Pair<K, V> {
    private K key;
    private V value;
    
    public void put(K key, V value) {
        this.key = key;
        this.value = value;
    }
    
    public K getKey() {
        return key;
    }
    
    public V getValue() {
        return value;
    }
}

// Usage
Pair<String, Integer> pairObj = new Pair<>();
pairObj.put("hello", 1243);
```

### Bounded Generics

Sometimes you don't want your generic type to be *any* object; you want to restrict it. This is done with **bounded generics**.

#### Upper Bound (`extends`)

This restricts the type to be a specific class or any of its subclasses (children). This is the most common type of bound.

- **Syntax:** `<T extends ClassName>`
- **Example:** `public class Print<T extends Number>` means `T` can be `Number`, `Integer`, `Double`, `Float`, etc., but not `String`.

```java
public class NumberPrinter<T extends Number> {
    private T value;
    
    public void printDouble() {
        // We can safely call Number methods
        System.out.println(value.doubleValue());
    }
}
```

#### Multi-Bound (`&`)

You can require a type to extend a class *and* implement one or more interfaces. The class must always be listed first.

- **Syntax:** `<T extends ParentClass & Interface1 & Interface2>`

```java
public class Processor<T extends Number & Comparable<T>> {
    public boolean isGreater(T first, T second) {
        return first.compareTo(second) > 0;
    }
}
```

### Wildcards for Flexibility

Wildcards are used to create more flexible methods that can accept a generic type of various kinds. They are especially useful with collections.

#### Upper Bounded Wildcard `<? extends Vehicle>`

- **Meaning:** "A `List` of `Vehicle`s or any class that is a child of `Vehicle` (e.g., `Bus`, `Car`)."
- **Use Case:** Ideal when you only need to **read** items from the list.

```java
public void printVehicles(List<? extends Vehicle> vehicles) {
    for (Vehicle v : vehicles) {
        System.out.println(v.getName());
    }
}
```

#### Lower Bounded Wildcard `<? super Vehicle>`

- **Meaning:** "A `List` of `Vehicle`s or any class that is a parent of `Vehicle` (e.g., `Object`)."
- **Use Case:** Ideal when you need to **add** `Vehicle` objects to the list.

```java
public void addVehicles(List<? super Vehicle> vehicles) {
    vehicles.add(new Car());
    vehicles.add(new Bus());
}
```

#### Unbounded Wildcard `<?>`

- **Meaning:** "A `List` of an unknown type."
- **Use Case:** Used when the type doesn't matter, and you will only be using methods available to all objects (like `size()` or `toString()`).

```java
public void printListSize(List<?> list) {
    System.out.println("Size: " + list.size());
}
```

---

## Type Erasure: How It Works

Generics are primarily a **compile-time feature**. The Java compiler uses generic type information to check your code for errors, but then it **erases** this information before creating the final bytecode.

- An unbounded generic class like `Print<T>` becomes a non-generic `Print` class where `T` is replaced by `Object`.
- A bounded generic class like `Print<T extends Number>` becomes a `Print` class where `T` is replaced by `Number`.

This process ensures that the generated code is compatible with older versions of the JVM that don't know about generics.

**Example:**

```java
// Before type erasure
public class Box<T> {
    private T value;
    public T getValue() { return value; }
}

// After type erasure (conceptually)
public class Box {
    private Object value;
    public Object getValue() { return value; }
}
```

---

## Best Practices

1. **Use descriptive type parameter names** for clarity:
   - `T` - Type
   - `E` - Element (used in collections)
   - `K` - Key
   - `V` - Value
   - `N` - Number

2. **Prefer generics over raw types** to maintain type safety.

3. **Use bounded wildcards** to increase API flexibility.

4. **Follow PECS principle** (Producer Extends, Consumer Super):
   - Use `<? extends T>` when you only get values out (producer)
   - Use `<? super T>` when you only put values in (consumer)

5. **Avoid generic array creation** as it can lead to runtime errors.

---

## Common Pitfalls

1. **Cannot instantiate generic types:** You cannot create instances like `new T()`.

2. **Cannot create arrays of parameterized types:** `new List<String>[10]` is not allowed.

3. **Static fields cannot use class type parameters:** Static members are shared across all instances.

4. **Cannot use primitives as type parameters:** Use wrapper classes instead (`Integer` not `int`).

5. **Type erasure limitations:** You cannot check instanceof with parameterized types at runtime.

---

**Remember:** Generics are your friend! They make your code safer, more readable, and easier to maintain. Always prefer using generics over raw types when working with collections and creating reusable components.
