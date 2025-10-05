# 📦 Java Variables in Depth

> A comprehensive guide to understanding variables in Java, covering definitions, declarations, naming conventions, primitive types, conversions, and variable kinds.

---

## 📑 Table of Contents

1. [What is a Variable?](#1-what-is-a-variable)
2. [Variable Naming Conventions](#2-variable-naming-conventions)
3. [Types of Variables (Primitive Types)](#3-types-of-variables-primitive-types)
   - [char (Character)](#1-char-character)
   - [byte](#2-byte)
   - [short](#3-short)
   - [int (Integer)](#4-int-integer)
   - [long](#5-long)
   - [float](#6-float)
   - [double](#7-double)
   - [boolean](#8-boolean)
4. [Types of Conversion](#4-types-of-conversion)
   - [Widening/Automatic Conversion](#1-wideningautomatic-conversion)
   - [Narrowing/Explicit Casting](#2-narrowingexplicit-casting-downcasting)
   - [Promotion during Expression](#3-promotion-during-expression)
5. [Kinds of Variables](#5-kinds-of-variables)
   - [Instance/Member Variables](#1-instancemember-variables)
   - [Static/Class Variables](#2-staticclass-variables)
   - [Local Variables](#3-local-variables)
   - [Method Parameters](#4-method-parameters)
   - [Constructor Parameters](#5-constructor-parameters)
6. [Quick Reference Tables](#quick-reference-tables)

---

## 1. What is a Variable?

[[00:30](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=30)]

A variable is a container that holds a value.

### Declaration Syntax

```java
Datatype VariableName = value;
```

**Example:**
```java
int var = 32;
```
[[01:08](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=68)]

### Key Characteristics

- **Static-Typed Language:** Java requires you to declare the data type of a variable before using it (e.g., `int var`). [[01:39](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=99)]

- **Strongly-Typed Language:** There are restrictions on the type of values a variable can hold, based on its declared data type. [[02:47](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=167)]

---

## 2. Variable Naming Conventions

[[03:03](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=183)]

### Rules and Best Practices

1. **Case-Sensitive:** Variable names are case-sensitive (e.g., `int A` and `int a` are different). [[04:10](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=250)]

2. **Legal Identifiers:** Variable names can be any legal identifier including Unicode letters and digits. [[04:28](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=268)]

3. **Starting Characters:** Variable names can start with:
   - A **dollar sign (`$`)**
   - An **underscore (`_`)**
   - A **letter**
   
   They **cannot** start with a digit. [[05:08](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=308)]
   
   ```java
   // Valid
   int $var;
   int _var;
   int var;
   
   // Invalid
   int 9var;  // Cannot start with a digit
   ```

4. **Reserved Keywords:** Variable names cannot be Java reserved keywords (e.g., `new`, `class`, `for`, `int`, `float`, etc.). [[05:50](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=350)]

5. **Single Word:** If a variable name is a single word, it should be in **all lowercase**. [[06:29](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=389)]
   ```java
   int jaipur;
   ```

6. **Multiple Words:** For multiple-word variable names, use **camelCase**. [[06:42](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=402)]
   ```java
   int jaipurCity;
   ```

7. **Constants:** For constant variable names, use **all uppercase letters**. [[07:09](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=429)]
   ```java
   static final int JAIPUR;
   ```

---

## 3. Types of Variables (Primitive Types)

[[08:22](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=502)]

Java has **8 primitive data types**:

### 1. `char` (Character)

[[09:00](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=540)]

- **Size:** 2 bytes (16 bits)
- **Representation:** Character representation of ASCII values
- **Range:** 0 to 65535 (from `\u0000` to `\uffff`) [[09:54](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=594)]

**Examples:**
```java
char var = 'a';     // prints 'a'
char var = 97;      // prints 'a' (97 is ASCII value for 'a')
```
[[10:08](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=608)]

### 2. `byte`

[[12:16](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=736)]

- **Size:** 1 byte (8 bits)
- **Representation:** Signed 2's complement
- **Range:** -128 to 127 [[12:40](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=760)]
- **Default Value:** 0

### 3. `short`

[[20:30](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=1230)]

- **Size:** 2 bytes (16 bits)
- **Representation:** Signed 2's complement
- **Range:** -32768 to 32767 [[20:42](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=1242)]
- **Default Value:** 0

### 4. `int` (Integer)

[[20:49](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=1249)]

- **Size:** 4 bytes (32 bits)
- **Representation:** Signed 2's complement
- **Range:** -2³¹ to 2³¹ - 1 [[21:01](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=1261)]
- **Default Value:** 0

### 5. `long`

[[21:26](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=1286)]

- **Size:** 8 bytes (64 bits)
- **Representation:** Signed 2's complement
- **Range:** -2⁶³ to 2⁶³ - 1 [[21:35](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=1295)]
- **Default Value:** 0

**Example:**
```java
long var = 100L;  // The 'L' indicates a long literal
```
[[21:52](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=1312)]

### 6. `float`

[[22:28](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=1348)]

- **Size:** 32 bits
- **Representation:** IEEE 754 floating-point value

**Example:**
```java
float var = 63.20f;  // The 'f' indicates a float literal
```
[[23:09](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=1389)]

### 7. `double`

[[22:54](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=1374)]

- **Size:** 64 bits
- **Representation:** IEEE 754 floating-point value

**Example:**
```java
double var = 63.20d;  // The 'd' indicates a double literal
```
[[23:25](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=1405)]

### 8. `boolean`

[[26:14](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=1574)]

- **Size:** 1 bit
- **Values:** `true` or `false`
- **Default Value:** `false`

---

## 4. Types of Conversion

[[28:03](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=1683)]

### 1. Widening/Automatic Conversion

[[28:14](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=1694)]

Happens automatically when converting a smaller data type to a larger data type.

**Conversion Path:**
```
byte → short → int → long → float → double
```

**Example:**
```java
int var = 10;
long varLong = var;  // Automatic conversion from int to long
```
[[28:47](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=1727)]

### 2. Narrowing/Explicit Casting (Downcasting)

[[30:23](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=1823)]

Occurs when converting a larger data type to a smaller data type. This requires **explicit casting** and can lead to **data loss** or **overflow**.

**Example:**
```java
int integerVariable = 10;
byte byteVariable = (byte) integerVariable;  // Explicit casting required
```
[[30:44](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=1844)]

**⚠️ Caution:** If the larger value exceeds the range of the smaller data type, unexpected results will occur.

```java
int value = 128;
byte result = (byte) value;  // Results in -128 (overflow)
// byte max is 127, so 128 wraps around
```
[[31:30](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=1890)]

### 3. Promotion during Expression

[[33:04](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=1984)]

When an arithmetic operation involves multiple data types, smaller data types are automatically promoted to the largest data type in the expression to prevent data loss.

**Example:**
```java
byte a = 127;
byte b = 1;
int sum = a + b;  // a and b are promoted to int
// byte cannot hold 128, so promotion occurs
```
[[33:23](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=2003)]

---

## 5. Kinds of Variables

[[42:07](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=2527)]

### 1. Instance/Member Variables

[[40:37](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=2437)]

- Declared inside a class but outside any method, constructor, or block
- Each object created from the class gets its **own copy** of instance variables

**Example:**
```java
class Employee {
    int memberVariable = 10;  // Instance variable
}

// Each object has its own copy
Employee obj1 = new Employee();
Employee obj2 = new Employee();
// obj1.memberVariable and obj2.memberVariable are separate
```
[[40:41](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=2441)]

### 2. Static/Class Variables

[[43:16](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=2596)]

- Declared with the `static` keyword inside a class but outside any method
- Only **one copy** of the static variable exists per class, shared by all objects
- Accessed directly using the class name

**Example:**
```java
class Employee {
    static int staticVariable = 10;  // Static variable
}

// Access using class name, not object
Employee.staticVariable;
```
[[43:21](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=2601)]

### 3. Local Variables

[[39:34](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=2374)]

- Declared inside a method, constructor, or block
- Scope is limited to the block in which they are declared
- **No default value** is assigned; must be initialized before use

**Example:**
```java
public void dummyMethod() {
    byte localVariable = 100;  // Local variable
    // Only accessible within dummyMethod
}
```
[[42:15](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=2535)]

### 4. Method Parameters

[[39:04](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=2344)]

- Variables passed to a method as arguments
- Scope is limited to that method

**Example:**
```java
public int dummyMethod2(int a, int b) {
    return a + b;  // a and b are method parameters
}
```

### 5. Constructor Parameters

[[47:10](http://www.youtube.com/watch?v=Q6_jrvz-R7w&t=2830)]

- Variables passed to a constructor when an object is created
- Used to initialize instance variables

**Example:**
```java
class Employee {
    int id;
    
    Employee(int a) {  // a is a constructor parameter
        this.id = a;
    }
}
```

---

## 📊 Quick Reference Tables

### Primitive Data Types

| Type | Size | Range | Default Value | Literal Example |
|------|------|-------|---------------|-----------------|
| `byte` | 1 byte | -128 to 127 | 0 | `byte b = 100;` |
| `short` | 2 bytes | -32,768 to 32,767 | 0 | `short s = 1000;` |
| `int` | 4 bytes | -2³¹ to 2³¹-1 | 0 | `int i = 100000;` |
| `long` | 8 bytes | -2⁶³ to 2⁶³-1 | 0 | `long l = 100L;` |
| `float` | 4 bytes | IEEE 754 | 0.0 | `float f = 63.20f;` |
| `double` | 8 bytes | IEEE 754 | 0.0 | `double d = 63.20d;` |
| `char` | 2 bytes | 0 to 65,535 | '\u0000' | `char c = 'A';` |
| `boolean` | 1 bit | true/false | false | `boolean b = true;` |

### Variable Kinds Comparison

| Variable Kind | Declared In | Scope | Default Value | Shared? |
|---------------|-------------|-------|---------------|---------|
| **Instance** | Class (outside methods) | Object level | Yes | No (per object) |
| **Static** | Class with `static` keyword | Class level | Yes | Yes (one copy) |
| **Local** | Method/Block | Method/Block | No | No |
| **Method Parameter** | Method signature | Method | N/A | No |
| **Constructor Parameter** | Constructor signature | Constructor | N/A | No |

### Type Conversion Summary

| Conversion Type | Direction | Requires Casting | Data Loss Risk |
|----------------|-----------|------------------|----------------|
| **Widening** | Smaller → Larger | No (Automatic) | No |
| **Narrowing** | Larger → Smaller | Yes (Explicit) | Yes |
| **Promotion** | During operations | No (Automatic) | No |

### Naming Convention Quick Guide

| Type | Convention | Example |
|------|------------|---------|
| **Single word variable** | lowercase | `int jaipur;` |
| **Multi-word variable** | camelCase | `int jaipurCity;` |
| **Constant** | UPPERCASE | `static final int JAIPUR;` |
| **Valid start characters** | Letter, $, _ | `$var`, `_var`, `var` |
| **Invalid start** | Digit | `9var` ❌ |

---

*Source: [YouTube Video - Java Variables in Depth](https://www.youtube.com/watch?v=Q6_jrvz-R7w)*
