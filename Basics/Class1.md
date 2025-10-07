# 📝 Types of Java Classes (Part 1)

## Table of Contents

1. [Introduction](#introduction)
2. [Concrete Class](#1-concrete-class-)
   - [Key Characteristics](#key-characteristics)
   - [Example](#example)
3. [Abstract Class](#2-abstract-class-)
   - [Purpose and Features](#purpose-and-features)
   - [Abstract vs Concrete Methods](#abstract-vs-concrete-methods)
   - [Example](#example-1)
   - [When to Use Abstract Classes](#when-to-use-abstract-classes)
4. [Superclass and Subclass (Inheritance)](#3-superclass-and-subclass-inheritance-)
   - [Basic Concepts](#basic-concepts)
   - [The Object Class: The Grandparent of All Classes](#the-object-class-the-grandparent-of-all-classes)
   - [Common Object Class Methods](#common-object-class-methods)
5. [Nested Class](#4-nested-class-)
   - [Why Use Nested Classes?](#why-use-nested-classes)
   - [Static Nested Class](#a-static-nested-class)
   - [Non-Static Nested Class (Inner Class)](#b-non-static-nested-class-inner-class)
     - [Member Inner Class](#1-member-inner-class)
     - [Local Inner Class](#2-local-inner-class)
     - [Anonymous Inner Class](#3-anonymous-inner-class)
   - [Nested Classes Comparison Table](#nested-classes-comparison-table)
6. [Best Practices](#best-practices)
7. [Common Interview Questions](#common-interview-questions)

---

## Introduction

Java provides several types of classes, each serving different purposes in object-oriented programming. Understanding these class types is fundamental to writing clean, maintainable, and efficient Java code. This guide covers the essential class types from basic concrete classes to complex nested structures.

**What you'll learn:**
- How to create and use different class types
- When to choose one class type over another
- The relationship between classes through inheritance
- Advanced concepts like nested and anonymous classes

---

## 1. Concrete Class 🧱

A **Concrete Class** is the most common type of class you'll write. It's a complete class with no missing implementation details—everything is fully defined and ready to use.

### Key Characteristics

**Can be Instantiated:**
- It can be used to create an instance (an object) directly using the `new` keyword.
- Example: `Person p = new Person();`

**Complete Implementation:**
- All of its methods must have a full implementation (a method body).
- No abstract or incomplete methods are allowed.

**Inheritance Flexibility:**
- It can extend other classes (concrete or abstract).
- It can implement one or more interfaces.
- It can be extended by other classes (unless declared `final`).

**Access Modifiers:**
- A top-level concrete class can only be declared as `public` or left with the default (package-private) modifier.
- Cannot be `private` or `protected` at the top level.

### Example

```java
// A simple concrete class
public class Person {
    private String name;
    private int age;
    
    // Constructor
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    // Concrete method with full implementation
    public void introduce() {
        System.out.println("Hi, I'm " + name + " and I'm " + age + " years old.");
    }
    
    // Getters and setters
    public String getName() {
        return name;
    }
}

// Another concrete class implementing an interface
public class Rectangle implements Shape {
    private double width;
    private double height;
    
    @Override
    public double calculateArea() {
        return width * height;
    }
}

// Usage
Person person = new Person("John", 25);
person.introduce(); // Output: Hi, I'm John and I'm 25 years old.

Rectangle rect = new Rectangle();
double area = rect.calculateArea();
```

**Key Point:** If you can create an object of a class using `new ClassName()`, it's a concrete class.

---

## 2. Abstract Class 🎨

An **Abstract Class** is like a template or a partially completed blueprint. It's designed to be extended by other classes and serves as a foundation for creating related classes.

### Purpose and Features

**Achieving Abstraction:**
- It's a primary way to achieve **abstraction** in Java.
- Abstraction means hiding complex implementation details and showing only essential features to the user.
- Focuses on "what" an object does rather than "how" it does it.

**Declaration:**
- Must be declared with the `abstract` keyword.
```java
public abstract class Car {
    // class body
}
```

**Cannot be Instantiated:**
- You **cannot** create an object of an abstract class using `new`.
- Example: `Car car = new Car();` // ❌ Compilation error!
- You must create an instance of a concrete subclass instead.

**Can Have Constructors:**
- Abstract classes can have constructors.
- These constructors are called when a concrete subclass is instantiated.

### Abstract vs Concrete Methods

An abstract class can contain a mix of:

**Abstract Methods:**
- Methods with no implementation (no body), just a signature.
- Declared with the `abstract` keyword.
- End with a semicolon instead of method body.
```java
public abstract void pressBrake();
```

**Concrete Methods:**
- Regular methods with full implementation.
- Provide common functionality to all subclasses.
```java
public int getNumberOfWheels() {
    return 4;
}
```

**Rule for Subclasses:**
Any concrete class that extends an abstract class **must** provide an implementation for all inherited abstract methods. If it doesn't, it must also be declared abstract.

### Example

```java
// Abstract class
public abstract class Car {
    private String brand;
    
    // Constructor
    public Car(String brand) {
        this.brand = brand;
    }
    
    // Abstract method (no implementation)
    public abstract void pressBrake();
    
    // Abstract method
    public abstract void accelerate();
    
    // Concrete method (with implementation)
    public int getNumberOfWheels() {
        return 4;
    }
    
    // Concrete method
    public String getBrand() {
        return brand;
    }
}

// Concrete subclass - must implement all abstract methods
public class Audi extends Car {
    
    public Audi() {
        super("Audi");
    }
    
    @Override
    public void pressBrake() {
        System.out.println("Audi brake system activated - smooth and precise.");
    }
    
    @Override
    public void accelerate() {
        System.out.println("Audi accelerating with turbo boost!");
    }
}

// Another concrete subclass
public class Tesla extends Car {
    
    public Tesla() {
        super("Tesla");
    }
    
    @Override
    public void pressBrake() {
        System.out.println("Tesla regenerative braking engaged.");
    }
    
    @Override
    public void accelerate() {
        System.out.println("Tesla instant electric acceleration!");
    }
}

// Usage
Car audi = new Audi();
audi.pressBrake(); // Output: Audi brake system activated - smooth and precise.
System.out.println(audi.getNumberOfWheels()); // Output: 4

Car tesla = new Tesla();
tesla.accelerate(); // Output: Tesla instant electric acceleration!
```

### When to Use Abstract Classes

Use an abstract class when:
- You want to share code among several closely related classes
- You expect classes that extend your abstract class to have many common methods or fields
- You want to declare non-static or non-final fields (instance variables)
- You need to provide a common base with some default behavior

---

## 3. Superclass and Subclass (Inheritance) 👪

This concept describes the parent-child relationship between classes through inheritance, one of the fundamental principles of object-oriented programming.

### Basic Concepts

**Superclass (Parent Class):**
- The class that is being extended.
- Provides common attributes and methods to its subclasses.
- Also called base class or parent class.

**Subclass (Child Class):**
- The class that extends the parent using the `extends` keyword.
- Inherits methods and variables from the superclass.
- Can add its own unique methods and variables.
- Can override inherited methods to provide specific behavior.
- Also called derived class or child class.

```java
// Superclass
public class Animal {
    protected String name;
    
    public void eat() {
        System.out.println(name + " is eating.");
    }
}

// Subclass
public class Dog extends Animal {
    public void bark() {
        System.out.println(name + " is barking!");
    }
}

// Usage
Dog dog = new Dog();
dog.name = "Buddy";
dog.eat();  // Inherited from Animal
dog.bark(); // Defined in Dog
```

### The Object Class: The Grandparent of All Classes

**⭐ This is a very common interview question!**

**Universal Parent:**
- In Java, if a class does not explicitly extend another class, it **implicitly extends the `Object` class**.
- This makes `Object` the ultimate superclass of every class in Java.
- Even if you write `class Person {}`, Java treats it as `class Person extends Object {}`.

**Universal Reference:**
- Because it's the parent of all classes, you can use a reference of type `Object` to hold an instance of *any* class.
- You can then use the `obj.getClass()` method to find out the object's actual class type at runtime.

```java
// Both Person and Audi implicitly extend Object
Object obj1 = new Person("Alice", 30);
Object obj2 = new Audi();

// We can find out their true type at runtime
System.out.println(obj1.getClass()); // Output: class Person
System.out.println(obj2.getClass()); // Output: class Audi

// We can also check instance types
if (obj1 instanceof Person) {
    System.out.println("obj1 is a Person");
}
```

### Common Object Class Methods

Every class in Java inherits these methods from `Object`:

| Method | Purpose |
|--------|---------|
| `toString()` | Returns a string representation of the object |
| `equals(Object obj)` | Compares this object with another for equality |
| `hashCode()` | Returns a hash code value for the object |
| `getClass()` | Returns the runtime class of the object |
| `clone()` | Creates and returns a copy of the object |
| `wait()` | Causes the current thread to wait |
| `notify()` | Wakes up a single thread waiting on this object |
| `notifyAll()` | Wakes up all threads waiting on this object |
| `finalize()` | Called by garbage collector before object destruction (deprecated) |

**Example of overriding Object methods:**

```java
public class Person {
    private String name;
    private int age;
    
    // Override toString() for better string representation
    @Override
    public String toString() {
        return "Person{name='" + name + "', age=" + age + "}";
    }
    
    // Override equals() for proper equality comparison
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        Person person = (Person) obj;
        return age == person.age && name.equals(person.name);
    }
}
```

---

## 4. Nested Class 🪆

A **Nested Class** is simply a class that is defined inside another class. This is useful for grouping classes that are only used in one place, which increases encapsulation and makes code more organized.

### Why Use Nested Classes?

**Logical Grouping:**
- If a class is only useful to one other class, it makes sense to keep them together.

**Increased Encapsulation:**
- Nested classes can access private members of the outer class.
- Can be hidden from other classes by making them private.

**More Readable and Maintainable Code:**
- Keeps related classes close together.
- Reduces clutter in the package.

There are two main categories: **Static Nested Classes** and **Non-Static Nested Classes (Inner Classes)**.

### A. Static Nested Class

A **Static Nested Class** is a static member of its outer class, behaving much like a static method or variable.

**Key Characteristics:**

**Declaration:**
```java
public class OuterClass {
    static class StaticNestedClass {
        // class body
    }
}
```

**Access to Outer Class Members:**
- Can **only access static members** (variables and methods) of the outer class.
- Cannot access instance variables or instance methods of the outer class.
- This is because static nested classes don't have a reference to an instance of the outer class.

**Instantiation:**
- You can create an object of it **without** needing an object of the outer class.
```java
OuterClass.StaticNestedClass nestedObj = new OuterClass.StaticNestedClass();
```

**Access Modifiers:**
- Unlike top-level classes, nested classes **can be declared as `private` or `protected`**, in addition to `public` and default (package-private).

**Example:**

```java
public class OuterClass {
    private static String staticOuterField = "Static Outer Field";
    private String instanceOuterField = "Instance Outer Field";
    
    // Static nested class
    static class StaticNestedClass {
        public void display() {
            // Can access static members of outer class
            System.out.println("Accessing: " + staticOuterField);
            
            // Cannot access instance members
            // System.out.println(instanceOuterField); // ❌ Compilation error!
        }
    }
}

// Usage - No outer class instance needed
OuterClass.StaticNestedClass nested = new OuterClass.StaticNestedClass();
nested.display(); // Output: Accessing: Static Outer Field
```

### B. Non-Static Nested Class (Inner Class)

An **Inner Class** is a non-static nested class. It has a strong association with an instance of the outer class and can access all members of the outer class.

**Key Characteristics:**

**Access to Outer Class Members:**
- Can access **all members** of the outer class, including `private` instance variables and methods.
- Has an implicit reference to an instance of the outer class.

**Instantiation:**
- You **must** have an instance of the outer class to create an instance of an inner class.
```java
OuterClass outerObj = new OuterClass();
OuterClass.InnerClass innerObj = outerObj.new InnerClass(); // Note the syntax
```

There are three main types of inner classes:

#### 1. Member Inner Class

This is the standard inner class defined at the same level as instance variables inside the outer class.

**Example:**

```java
public class OuterClass {
    private String outerField = "Outer field";
    private static String staticOuterField = "Static outer field";
    
    // Member inner class
    class InnerClass {
        private String innerField = "Inner field";
        
        public void display() {
            // Can access all outer class members
            System.out.println("Outer field: " + outerField);
            System.out.println("Static outer field: " + staticOuterField);
            System.out.println("Inner field: " + innerField);
        }
    }
    
    public void createInner() {
        InnerClass inner = new InnerClass();
        inner.display();
    }
}

// Usage
OuterClass outer = new OuterClass();
OuterClass.InnerClass inner = outer.new InnerClass();
inner.display();
```

**Key Points:**
- Can access all members (static and non-static) of the outer class
- Cannot define static members (except final static constants)
- Can be declared with any access modifier: `public`, `private`, `protected`, or default

#### 2. Local Inner Class

A class defined inside a specific block, like a **method**, constructor, or an `if` statement.

**Key Characteristics:**

**Scope:**
- Its scope is restricted to the block where it is defined.
- Cannot be accessed outside that block.

**Instantiation:**
- Can only be instantiated within the block where it's defined.

**Access Modifiers:**
- Cannot be declared `public`, `private`, `protected`, or `static`.
- Can be declared `abstract` or `final`.

**Access to Variables:**
- Can access all members of the outer class.
- Can only access local variables of the enclosing block if they are `final` or effectively final.

**Example:**

```java
public class OuterClass {
    private String outerField = "Outer field";
    
    public void someMethod() {
        final String localVar = "Local variable";
        
        // Local inner class defined inside a method
        class LocalInnerClass {
            public void display() {
                System.out.println("Outer field: " + outerField);
                System.out.println("Local variable: " + localVar);
            }
        }
        
        // Instantiate and use the local inner class
        LocalInnerClass local = new LocalInnerClass();
        local.display();
        
    } // LocalInnerClass is not accessible outside this method
}

// Usage
OuterClass outer = new OuterClass();
outer.someMethod();
// Output:
// Outer field: Outer field
// Local variable: Local variable
```

#### 3. Anonymous Inner Class

An inner class **without a name**. It's the most concise form of inner class.

**Purpose:**
- Used to create a one-time-use class.
- Typically for quickly overriding methods of a class or implementing an interface.
- Great for event handlers, callbacks, and short implementations.

**How it Works:**
- The compiler automatically creates a class file for it behind the scenes.
- File naming: `OuterClass$1.class`, `OuterClass$2.class`, etc.

**Syntax:**
```java
// Extending a class
SuperClass obj = new SuperClass() {
    // Override methods here
};

// Implementing an interface
InterfaceName obj = new InterfaceName() {
    // Implement methods here
};
```

**Example from the video:**

Instead of creating a whole new `Audi.java` file to extend the `Car` class, you can do it on the fly:

```java
// 'Car' is an abstract class with an abstract 'pressBrake()' method
public abstract class Car {
    public abstract void pressBrake();
    public abstract void accelerate();
}

// Creating an anonymous inner class
public class TestCar {
    public static void main(String[] args) {
        // Anonymous inner class extending Car
        Car audiCarObj = new Car() {
            @Override
            public void pressBrake() {
                System.out.println("Audi brake system activated.");
            }
            
            @Override
            public void accelerate() {
                System.out.println("Audi accelerating smoothly.");
            }
        }; // Semicolon is required here!
        
        audiCarObj.pressBrake(); // Output: Audi brake system activated.
        audiCarObj.accelerate(); // Output: Audi accelerating smoothly.
        
        // Another anonymous inner class with different implementation
        Car teslaCarObj = new Car() {
            @Override
            public void pressBrake() {
                System.out.println("Tesla regenerative braking.");
            }
            
            @Override
            public void accelerate() {
                System.out.println("Tesla instant acceleration!");
            }
        };
        
        teslaCarObj.accelerate(); // Output: Tesla instant acceleration!
    }
}
```

**Anonymous Inner Class with Interface:**

```java
// Interface
interface Greeting {
    void greet(String name);
}

// Using anonymous inner class to implement interface
public class Test {
    public static void main(String[] args) {
        // Anonymous inner class implementing Greeting interface
        Greeting greeting = new Greeting() {
            @Override
            public void greet(String name) {
                System.out.println("Hello, " + name + "!");
            }
        };
        
        greeting.greet("Alice"); // Output: Hello, Alice!
    }
}
```

**Modern Alternative - Lambda Expressions:**

For interfaces with a single abstract method (functional interfaces), you can use lambda expressions (Java 8+):

```java
// Using lambda instead of anonymous inner class
Greeting greeting = (name) -> System.out.println("Hello, " + name + "!");
greeting.greet("Bob"); // Output: Hello, Bob!
```

### Nested Classes Comparison Table

| Feature | Static Nested Class | Member Inner Class | Local Inner Class | Anonymous Inner Class |
|---------|-------------------|-------------------|-------------------|---------------------|
| **Location** | Inside outer class | Inside outer class | Inside method/block | Inline at usage point |
| **Access to outer instance members** | No | Yes | Yes | Yes |
| **Access to outer static members** | Yes | Yes | Yes | Yes |
| **Requires outer instance to create** | No | Yes | Yes | Yes |
| **Can be declared `private`/`protected`** | Yes | Yes | No | No |
| **Can have static members** | Yes | Only `static final` | Only `static final` | Only `static final` |
| **Has a name** | Yes | Yes | Yes | No |
| **Can be reused** | Yes | Yes | Limited to method | No, one-time use |

---

## Best Practices

1. **Use Concrete Classes for standard objects:**
   - When you have a complete, ready-to-use class with no abstraction needed.
   - Most of your application classes will be concrete classes.

2. **Use Abstract Classes when:**
   - You want to provide a common base with some default behavior.
   - You need to share code among several related classes.
   - You want to declare non-static or non-final fields.

3. **Prefer Interfaces over Abstract Classes when:**
   - You only need to define a contract (method signatures).
   - You want to support multiple inheritance.
   - You're designing for unrelated classes to implement.

4. **Use Static Nested Classes when:**
   - The nested class doesn't need access to the outer class's instance members.
   - You want to improve encapsulation and packaging.

5. **Use Inner Classes when:**
   - The class logically belongs to the outer class.
   - You need access to outer class's private members.

6. **Use Anonymous Inner Classes for:**
   - One-time implementations.
   - Short, simple implementations.
   - Event handlers and callbacks.
   - **Note:** In modern Java (8+), prefer lambda expressions for functional interfaces.

7. **Keep the Object class in mind:**
   - Always override `toString()` for meaningful string representation.
   - Override `equals()` and `hashCode()` together when defining equality.
   - Remember that all your classes implicitly extend `Object`.

---

## Common Interview Questions

**Q1: What is the difference between an abstract class and an interface?**

**Answer:** 
- Abstract classes can have both abstract and concrete methods; interfaces (before Java 8) could only have abstract methods.
- A class can extend only one abstract class but implement multiple interfaces.
- Abstract classes can have constructors; interfaces cannot.
- Abstract classes can have instance variables; interfaces can only have constants (`public static final`).

**Q2: Why can't we instantiate an abstract class?**

**Answer:** 
Abstract classes are incomplete—they may have abstract methods without implementations. Creating an instance wouldn't make sense because you couldn't call those methods. The purpose is to provide a template for subclasses to complete.

**Q3: What is the root class of all Java classes?**

**Answer:** 
The `Object` class is the root class of all classes in Java. Every class implicitly extends `Object` if it doesn't explicitly extend another class.

**Q4: What is the difference between static nested class and inner class?**

**Answer:**
- Static nested classes don't have access to instance members of the outer class; inner classes do.
- Static nested classes can be instantiated without an outer class instance; inner classes require one.
- Static nested classes can have static members; inner classes cannot (except `static final` constants).

**Q5: When would you use an anonymous inner class?**

**Answer:**
- When you need a one-time implementation of a class or interface.
- For event handlers and callbacks.
- When the implementation is short and simple.
- In modern Java, lambdas are often preferred for functional interfaces.

**Q6: Can an abstract class be declared final?**

**Answer:**
No! This would be contradictory. `final` classes cannot be extended, but abstract classes are meant to be extended to provide implementations for abstract methods.

**Q7: Can we have a constructor in an abstract class?**

**Answer:**
Yes! Abstract classes can have constructors. They're called when a concrete subclass is instantiated, allowing the abstract class to initialize its own fields.
