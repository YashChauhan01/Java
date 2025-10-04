# 🏛️ Java OOPS Fundamentals: Comprehensive Notes

> A complete and easy-to-understand guide on Object-Oriented Programming (OOPS) fundamentals in Java, covering everything from basic concepts to the four main pillars.

---

## 📚 Table of Contents

1. [OOPS Overview](#-oops-overview)
2. [Objects and Classes](#-objects-and-classes)
3. [The Four Pillars of OOPS](#-the-four-pillars-of-oops)
   - [Data Abstraction](#1-data-abstraction)
   - [Encapsulation](#2-encapsulation)
   - [Inheritance](#3-inheritance)
   - [Polymorphism](#4-polymorphism)
4. [Object Relationships](#-object-relationships)

---

## 🧐 OOPS Overview

**OOPS** stands for **Object-Oriented Programming**. The core idea is to structure a program around "objects," which can be thought of as real-world entities like a car, a bike, or a person.

### Procedural vs. Object-Oriented Programming

| Feature | Procedural Programming | Object-Oriented Programming (OOPS) |
|---------|------------------------|-------------------------------------|
| **Focus** | Divides the program into functions. | Divides the program into objects. |
| **Data Security** | Less secure; data moves freely between functions. | More secure; data is hidden and controlled. |
| **Core Concept** | Importance is given to functions. | Importance is given to data. |
| **Inheritance** | Not possible. | Possible. |
| **Code Reusability** | Not easily achieved. | Easily achieved through inheritance. |
| **Examples** | C, Pascal, FORTRAN | Java, Python, C++ |

---

## 🧱 Objects and Classes

The two foundational concepts of OOPS are **objects** and **classes**.

### Objects

An object is a real-world entity that has two main characteristics:

1. **Properties (or State):** These are the attributes or data that describe the object.
2. **Behavior (or Function):** These are the actions that the object can perform.

**Example: A `Dog` Object**

- **Properties:** `age`, `breed`, `color`
- **Behaviors:** `bark()`, `sleep()`, `eat()`

### Classes

A **class** is a **template** or **blueprint** from which objects are created. It defines the properties and behaviors that all objects of that type will have.

- From a single class, you can create multiple objects.
- The `class` keyword is used to create a class in Java.

### Example: Creating a `Student` Class and Objects

```java
// Demo of Class and Object creation

// 1. Create the class (the blueprint)
class Student {
    // Properties (data variables)
    int age;
    String address;

    // Behaviors (data methods)
    public void updateAddress(String newAddress) {
        this.address = newAddress;
    }

    public int getAge() {
        return this.age;
    }
}

// 2. Create objects (instances) from the class
Student engStudent = new Student(); // Engineering student object
Student mbaStudent = new Student(); // MBA student object

// Each object has its own separate properties
engStudent.age = 23;
mbaStudent.age = 25;
```

---

## 🏛️ The Four Pillars of OOPS

OOPS is built on four main principles, often called the "four pillars."

### 1. Data Abstraction

**Abstraction** means hiding the complex internal implementation details and showing only the essential functionality to the user.

#### Real-world Example: A Car's Brake Pedal

When you press the brake pedal, the car slows down. You only need to know *what* it does (slows the car), not *how* it does it (the internal mechanics of the braking system). That complexity is abstracted away.

#### How it's achieved in Java

Through `interfaces` and `abstract classes`.

#### Advantages

- **Simplicity:** Makes the system easier to use.
- **Security & Confidentiality:** Hides sensitive implementation details.

### Code Example: Abstraction

```java
// Abstraction through an interface

// 1. The interface defines WHAT the car can do, not HOW.
interface Car {
    void applyBrake();
    void pressAccelerator();
}

// 2. The class provides the hidden implementation.
class CarImplementation implements Car {
    @Override
    public void applyBrake() {
        // Step 1: Reduce fuel flow
        // Step 2: Engage brake pads
        // ... (complex internal logic)
        System.out.println("Car is slowing down.");
    }

    @Override
    public void pressAccelerator() {
        // ... (complex internal logic)
    }
}

// The client/user only interacts with the simple interface.
Car myCar = new CarImplementation();
myCar.applyBrake(); // The user doesn't need to know the internal steps.
```

---

### 2. Encapsulation

**Encapsulation** is the practice of bundling the data (variables) and the code that works on that data (methods) together into a single unit (a class). This is also known as **data hiding**.

#### How it's achieved

1. Declare the variables of a class as `private`.
2. Provide public `getter` and `setter` methods to view and modify the variable values.

#### Advantages

- **Control:** The class has full control over its data. You can add validation logic inside the setters.
- **Loosely Coupled Code:** Reduces dependencies between different parts of your code.
- **Security:** Protects the data from accidental or unauthorized modification.

### Code Example: Encapsulation

```java
// Encapsulation example

public class Dog {
    // 1. Data member is private
    private String color;

    // 2. Public getter to view the data
    public String getColor() {
        return color;
    }

    // 3. Public setter to modify the data
    public void setColor(String color) {
        // Here you could add validation if needed
        this.color = color;
    }
}

// Main method in another class
public static void main(String[] args) {
    Dog myDog = new Dog();
    myDog.setColor("Black"); // Set the color via the public method
    
    // You cannot do this: myDog.color = "White"; // This would cause a compilation error
    
    System.out.println(myDog.getColor()); // Access the color via the public method
}
```

---

### 3. Inheritance

**Inheritance** is a mechanism where a new class (child class) inherits properties and behaviors from an existing class (parent class).

- It promotes **code reusability**.
- It's achieved using the `extends` keyword (for classes) or `implements` keyword (for interfaces).

### Types of Inheritance

- **Single Inheritance:** A class inherits from only one parent class. (`Class B extends Class A`)
- **Multilevel Inheritance:** A class inherits from another class, which in turn inherits from another class. (`Class C extends B`, `Class B extends A`)
- **Hierarchical Inheritance:** Multiple classes inherit from a single parent class. (`Class B extends A`, `Class C extends A`)
- **Multiple Inheritance (The Diamond Problem):** A class inheriting from two or more parent classes. **Java does not support multiple inheritance with classes** to avoid the "diamond problem" (ambiguity if two parent classes have a method with the same name). However, it can be achieved using interfaces.

### Code Example: Single Inheritance

```java
// Parent class
class Vehicle {
    boolean hasEngine = true;

    public boolean getEngineStatus() {
        return hasEngine;
    }
}

// Child class inherits from Vehicle
class Car extends Vehicle {
    String carType = "Sedan";

    public String getCarType() {
        return carType;
    }
}

// Main method
public static void main(String[] args) {
    Car myCar = new Car();
    System.out.println(myCar.getEngineStatus()); // Accessing method from the parent (Vehicle) class
    System.out.println(myCar.getCarType());      // Accessing its own method
}
```

**Output:**
```
true
Sedan
```

---

### 4. Polymorphism

**Polymorphism** means "many forms." It's the ability of a method to behave differently in different situations.

There are two types of polymorphism in Java:

#### 1. Compile-Time Polymorphism (Static or Method Overloading)

This occurs when there are multiple methods in the **same class** with the **same name** but different **parameters** (either different types or a different number of parameters). The correct method to call is decided at compile time.

### Code Example: Method Overloading

```java
class Sum {
    // Method 1: Takes two integers
    public int doSum(int a, int b) {
        return a + b;
    }

    // Method 2: Same name, but takes two strings
    public String doSum(String a, String b) {
        return a + b;
    }

    // Method 3: Same name, but takes three integers
    public int doSum(int a, int b, int c) {
        return a + b + c;
    }
}

// In the main method:
Sum calculator = new Sum();
calculator.doSum(5, 2);       // Calls Method 1 → Returns 7
calculator.doSum("A", "B");   // Calls Method 2 → Returns "AB"
calculator.doSum(3, 4, 2);    // Calls Method 3 → Returns 9
```

#### 2. Run-Time Polymorphism (Dynamic or Method Overriding)

This occurs when a child class provides a specific implementation for a method that is already defined in its parent class. The method to be executed is determined at runtime based on the object's type.

### Code Example: Method Overriding

```java
// Parent class
class Animal {
    public void getSound() {
        System.out.println("Animal makes a sound");
    }
}

// Child class
class Cat extends Animal {
    // Overriding the parent's method
    @Override
    public void getSound() {
        System.out.println("Meow");
    }
}

// Main method
Animal myCat = new Cat(); // Parent reference, but a Cat object
myCat.getSound();         // At runtime, Java knows it's a Cat object and calls the Cat's method.
                          // Output: Meow
```

**Output:**
```
Meow
```

---

## 🤝 Object Relationships

There are two primary ways objects can be related to each other:

### 1. Is-a Relationship (Inheritance)

- This is achieved through inheritance.
- It forms a parent-child relationship.
- **Example:** A `Dog` **is an** `Animal`. A `Car` **is a** `Vehicle`.

### 2. Has-a Relationship

This occurs when an object of one class is used as a member (a variable) inside another class.

- **Example:** A `School` **has** `Students`. A `Bike` **has an** `Engine`.

#### Types of Has-a Relationships

**Aggregation (Weak Relationship):**
- Both objects can exist independently.
- If one is destroyed, the other can still exist.
- **Example:** A school and its students.

```java
class Student {
    String name;
    int rollNo;
}

class School {
    String schoolName;
    List<Student> students; // School HAS students
}
// If the school is closed, students still exist
```

**Composition (Strong Relationship):**
- One object is a part of the other and cannot exist without it.
- If the owner object is destroyed, the contained object is also destroyed.
- **Example:** A house and its rooms.

```java
class Room {
    String roomType;
}

class House {
    String address;
    List<Room> rooms; // House HAS rooms
}
// If the house is demolished, the rooms no longer exist
```

---

## 📊 Quick Reference Summary

### The Four Pillars

| Pillar | Definition | Key Benefit | How Achieved |
|--------|------------|-------------|--------------|
| **Abstraction** | Hiding implementation details | Simplicity & Security | Interfaces, Abstract classes |
| **Encapsulation** | Bundling data and methods | Data Protection & Control | Private variables + Public getters/setters |
| **Inheritance** | Deriving new classes from existing ones | Code Reusability | `extends` keyword |
| **Polymorphism** | One interface, multiple implementations | Flexibility | Method Overloading & Overriding |

### Polymorphism Types

| Type | When Decided | Also Known As | Example |
|------|--------------|---------------|---------|
| **Compile-Time** | At compile time | Static / Method Overloading | Multiple methods with same name, different parameters |
| **Run-Time** | At runtime | Dynamic / Method Overriding | Child class overrides parent method |

### Object Relationships

| Relationship | Type | Example | Independence |
|--------------|------|---------|--------------|
| **Is-a** | Inheritance | Dog is an Animal | Child depends on parent |
| **Has-a (Aggregation)** | Weak | School has Students | Both can exist independently |
| **Has-a (Composition)** | Strong | House has Rooms | Contained object depends on owner |

---

## 🎯 Key Takeaways

1. **Objects** are instances of **Classes** (blueprints)
2. **Abstraction** hides complexity and shows only what's necessary
3. **Encapsulation** protects data through access modifiers
4. **Inheritance** enables code reuse and establishes hierarchies
5. **Polymorphism** allows flexible, adaptable code
6. Java uses **Is-a** (inheritance) and **Has-a** (composition/aggregation) relationships

---

## 💡 Best Practices

- Use abstraction to simplify complex systems
- Always encapsulate your data (use private variables with getters/setters)
- Favor composition over inheritance when possible
- Use polymorphism to write flexible, maintainable code
- Follow the principle: "Code to an interface, not an implementation"

---

<div align="center">

## 🚀 Master Java OOPS!

**"Understanding OOPS is fundamental to becoming a proficient Java developer"**

⭐ Keep Learning, Keep Growing! ⭐

</div>
