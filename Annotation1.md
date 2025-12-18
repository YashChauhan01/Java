# Java Annotations - Complete Guide

## What are Annotations?

**Annotations** are a way to add **metadata** (data about data) to Java code.

### Key Characteristics

```
Annotations:
├─ Add metadata to Java code
├─ Usage is OPTIONAL (won't break code if not used)
├─ Can be accessed at runtime using Reflection
├─ Allow adding logic based on metadata
└─ Start with @ symbol
```

### Basic Syntax

```java
@AnnotationName
public class MyClass { }

@AnnotationName
public void myMethod() { }
```

---

## Why Use Annotations?

### Purpose

| Purpose | Description |
|---------|-------------|
| **Metadata** | Provide information about code to compiler/runtime |
| **Compile-time checks** | Compiler can validate code based on annotations |
| **Runtime processing** | Use reflection to read annotations and add logic |
| **Code generation** | Frameworks use annotations to generate code |
| **Documentation** | Self-documenting code |

---

## Prerequisite: Reflection

**IMPORTANT:** You must understand **Reflection** before learning annotations!

**Why?** Annotations are accessed using Reflection.

```java
// Annotations are metadata
// Reflection is used to read metadata
// Therefore: Reflection reads annotations
```

**Recommendation:** Watch/read the Reflection tutorial first.

---

## Where Can Annotations Be Applied?

Annotations can be used on:

```
✓ Classes
✓ Interfaces
✓ Methods
✓ Fields (variables)
✓ Parameters
✓ Constructors
✓ Local variables
✓ Packages
✓ Enums
```

**Key Point:** Annotations can be applied almost anywhere in Java code!

---

## Simple Example

### The @Override Annotation

```java
// Interface
interface Bird {
    void fly();
}

// Implementation
class Eagle implements Bird {
    
    @Override  // ← Annotation (OPTIONAL)
    public void fly() {
        System.out.println("Eagle flying");
    }
}
```

**What @Override does:**
1. Provides metadata to compiler
2. Compiler checks: "Is this method in parent/interface?"
3. If not found → Compilation error
4. Helps catch typos and mistakes

**Without @Override:**
```java
class Eagle implements Bird {
    // Missing @Override - still works but no compile-time check
    public void fly() {
        System.out.println("Eagle flying");
    }
}
```

---

## Types of Annotations

```
Annotations
│
├── 1. Predefined (Built-in Java)
│   │
│   ├── Used on Java Code
│   │   ├── @Deprecated
│   │   ├── @Override
│   │   ├── @SuppressWarnings
│   │   ├── @FunctionalInterface
│   │   └── @SafeVarargs
│   │
│   └── Meta-Annotations (Used on Annotations)
│       ├── @Retention
│       ├── @Target
│       ├── @Documented
│       └── @Inherited
│
└── 2. Custom (User-defined)
    └── @MyCustomAnnotation
```

---

## Part 1: Predefined Annotations (Used on Java Code)

### 1. @Deprecated

**Purpose:** Mark class/method/field as deprecated (outdated, not recommended)

**When to use:**
- Class/method is old and has a better alternative
- No further development/support
- Want to warn users to use new implementation

---

#### Example: Deprecated Method

```java
public class Mobile {
    
    @Deprecated  // Mark as deprecated
    public void oldFeature() {
        System.out.println("Old feature");
    }
    
    // New alternative
    public void newFeature() {
        System.out.println("New improved feature");
    }
}

// Usage
public class Main {
    public static void main(String[] args) {
        Mobile mobile = new Mobile();
        mobile.oldFeature();  // ⚠️ Compiler shows warning
        // Warning: 'oldFeature()' is deprecated
    }
}
```

**What happens:**
- Compiler shows **warning** (not error)
- Code still runs, but warns developer
- Indicates: "Use alternative instead"

---

#### Where @Deprecated Can Be Used

```
✓ Class
✓ Method
✓ Field
✓ Constructor
✓ Package
✓ Parameter
✓ Local variable
✓ Type (interface, enum)
```

---

### 2. @Override

**Purpose:** Indicate method is overriding parent class/interface method

**Benefits:**
1. Compile-time check: Method actually exists in parent
2. Catches typos in method name
3. Documents intent clearly

---

#### Example: @Override

```java
interface Bird {
    boolean fly();
}

class Eagle implements Bird {
    
    @Override
    public boolean fly() {  // ✓ Correct
        return true;
    }
}

class Sparrow implements Bird {
    
    @Override
    public boolean fly1() {  // ❌ Compilation Error!
        // Error: Method does not override method from its superclass
        // Typo: fly1 instead of fly
        return true;
    }
}
```

**Without @Override:**
```java
class Sparrow implements Bird {
    // No @Override - compiles but doesn't implement interface!
    public boolean fly1() {  // Creates NEW method, doesn't override
        return true;
    }
    // Missing implementation of fly() → Runtime error
}
```

---

#### Where @Override Can Be Used

```
✓ Methods only
```

---

### 3. @SuppressWarnings

**Purpose:** Tell compiler to ignore specific warnings

**Common Use Cases:**
- Ignore deprecation warnings
- Ignore unused variable warnings
- Ignore unchecked type warnings

---

#### Example 1: Suppress Deprecation Warning

```java
public class Mobile {
    @Deprecated
    public void oldMethod() {
        System.out.println("Deprecated method");
    }
}

public class Main {
    
    @SuppressWarnings("deprecation")  // Suppress warning
    public static void main(String[] args) {
        Mobile mobile = new Mobile();
        mobile.oldMethod();  // No warning shown!
    }
}
```

**Without @SuppressWarnings:**
```java
public static void main(String[] args) {
    Mobile mobile = new Mobile();
    mobile.oldMethod();  // ⚠️ Warning: Using deprecated method
}
```

---

#### Example 2: Suppress Multiple Warnings

```java
@SuppressWarnings({"deprecation", "unused"})
public class Main {
    
    public static void main(String[] args) {
        Mobile mobile = new Mobile();
        mobile.oldMethod();  // No deprecation warning
        
        int unusedVar = 10;  // No unused warning
    }
}
```

---

#### Example 3: Suppress ALL Warnings

```java
@SuppressWarnings("all")  // Suppress ALL warnings
public class Main {
    // No warnings shown for anything in this class
}
```

---

#### Common Warning Types

| Warning Type | Description |
|--------------|-------------|
| **"deprecation"** | Using deprecated elements |
| **"unused"** | Unused variables/methods |
| **"unchecked"** | Unchecked type casts |
| **"rawtypes"** | Using raw types (no generics) |
| **"all"** | All warnings |

---

#### Where @SuppressWarnings Can Be Used

```
✓ Class
✓ Method
✓ Field
✓ Parameter
✓ Constructor
✓ Local variable
```

**Scope:**
- Class level: Affects entire class
- Method level: Affects only that method

---

#### ⚠️ Warning About Warnings

**Be Careful:**
```java
// Bad practice
@SuppressWarnings("all")
public void riskyMethod() {
    int result = 5 / 0;  // ArithmeticException!
    // Warning suppressed, but exception still happens at runtime!
}
```

**Best Practice:**
- Only suppress warnings you understand
- Be specific: Use "deprecation" instead of "all"
- Warnings can prevent runtime errors!

---

### 4. @FunctionalInterface

**Purpose:** Mark interface as functional interface (must have exactly one abstract method)

**Reminder: What is Functional Interface?**
- Interface with **exactly ONE abstract method**
- Can have default and static methods
- Used with lambda expressions

---

#### Example: @FunctionalInterface

```java
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);  // Only ONE abstract method ✓
}

// Usage with lambda
Calculator add = (a, b) -> a + b;
System.out.println(add.calculate(5, 3));  // Output: 8
```

---

#### What Happens Without It?

```java
// Without annotation - still works but no compile-time check
interface Calculator {
    int calculate(int a, int b);
    // If someone adds another method, still compiles
}

// With annotation - compile-time error if rules broken
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
    int multiply(int a, int b);  // ❌ Compilation Error!
    // Error: Multiple non-overriding abstract methods found
}
```

---

#### Where @FunctionalInterface Can Be Used

```
✓ Interfaces only
(Actually: Type level - class, interface, enum)
```

---

### 5. @SafeVarargs

**Purpose:** Suppress heap pollution warnings for methods with varargs

**Prerequisites to understand:**
1. What are varargs?
2. What is heap pollution?
3. How varargs can cause heap pollution?

---

#### Understanding Varargs (Variable Arguments)

**What are Varargs?**

```java
// Without varargs - need multiple methods
public void print(int a) { }
public void print(int a, int b) { }
public void print(int a, int b, int c) { }

// With varargs - ONE method handles all!
public void print(int... numbers) {
    // Can accept 0, 1, 2, 3... any number of arguments
}

// Internally converted to array
public void print(int[] numbers) { }
```

**Usage:**
```java
print();              // 0 arguments
print(1);            // 1 argument
print(1, 2);         // 2 arguments
print(1, 2, 3, 4);   // 4 arguments
```

---

#### Understanding Heap Pollution

**What is Heap Pollution?**

```
Heap Pollution = Object of Type A storing reference to object of Type B

Example:
Variable declared as: List<Integer>  (should hold Integer lists)
Actually holds:       List<String>   (holds String list instead!)
                      ↑
                   HEAP POLLUTION!
```

**Simple Example:**
```java
List<Integer> numbers = new ArrayList<>();  // Declared as Integer list
numbers = (List<Integer>)(Object) stringList;  // Actually String list!
// Type A object pointing to Type B object = Heap Pollution
```

---

#### How Varargs Causes Heap Pollution

```java
public class HeapPollutionExample {
    
    // Method accepting varargs of List<Integer>
    public static void printLogValues(List<Integer>... logNumbers) {
        // varargs converted to array: List<Integer>[]
        
        // Store in Object array (Object is parent of all)
        Object[] objectArray = logNumbers;
        
        // Create a List<String> (different type!)
        List<String> stringList = new ArrayList<>();
        stringList.add("Hello");
        stringList.add("World");
        
        // Assign String list to position meant for Integer list
        objectArray[0] = stringList;  // HEAP POLLUTION!
        
        // logNumbers declared as List<Integer>[]
        // But logNumbers[0] actually contains List<String>
        
        // Trying to get Integer from what's actually String list
        Integer num = logNumbers[0].get(0);  // ClassCastException!
    }
    
    public static void main(String[] args) {
        List<Integer> list1 = Arrays.asList(1, 2, 3);
        List<Integer> list2 = Arrays.asList(4, 5, 6);
        
        printLogValues(list1, list2);  
        // ⚠️ Warning: Possible heap pollution from varargs
    }
}
```

**What Happened:**
```
1. logNumbers declared as: List<Integer>[]
2. Internally stored as: Object[]
3. Inserted: List<String> into Object[]
4. logNumbers[0] now points to: List<String>
5. But compiler thinks it's: List<Integer>
6. Result: HEAP POLLUTION!
```

---

#### Using @SafeVarargs

**When to use:**
- You're SURE heap pollution won't happen in your method
- Want to suppress the warning

```java
@SafeVarargs  // Suppress heap pollution warning
public static void printLogValues(List<Integer>... logNumbers) {
    // If you're confident no heap pollution occurs
    for (List<Integer> list : logNumbers) {
        for (Integer num : list) {
            System.out.println(num);
        }
    }
}
```

---

#### Rules for @SafeVarargs

**Can only be used on:**
```
✓ static methods
✓ final methods
✓ private methods (Java 9+)
```

**Why these restrictions?**
- These methods cannot be overridden
- Prevents child class from removing @SafeVarargs
- Ensures safety is maintained

```java
// ✓ Allowed
@SafeVarargs
public static void method1(List<String>... lists) { }

@SafeVarargs
public final void method2(List<String>... lists) { }

@SafeVarargs  // Java 9+
private void method3(List<String>... lists) { }

// ❌ Not allowed
@SafeVarargs
public void method4(List<String>... lists) { }  // Not static/final/private
```

---

#### Where @SafeVarargs Can Be Used

```
✓ Methods (static, final, or private)
✓ Constructors (that have varargs parameters)
```

---

## Summary: Predefined Annotations on Java Code

| Annotation | Purpose | Where Used |
|------------|---------|------------|
| **@Deprecated** | Mark as outdated | Class, method, field, etc. |
| **@Override** | Verify method override | Methods only |
| **@SuppressWarnings** | Ignore specific warnings | Class, method, field, etc. |
| **@FunctionalInterface** | Mark functional interface | Interfaces |
| **@SafeVarargs** | Suppress heap pollution warning | static/final/private methods with varargs |

---

## Part 2: Meta-Annotations

### What are Meta-Annotations?

**Meta-Annotations** = Annotations used ON other annotations

```java
@Retention(RetentionPolicy.RUNTIME)  // ← Meta-annotation
@Target(ElementType.METHOD)          // ← Meta-annotation
public @interface MyAnnotation {
    // Custom annotation
}
```

**Purpose:** Configure how and where annotations can be used

---

### Common Meta-Annotations

```
Meta-Annotations:
├── @Retention  → When annotation information is available
├── @Target     → Where annotation can be applied
├── @Documented → Include in JavaDoc
├── @Inherited  → Inherited by subclasses
└── @Repeatable → Can be used multiple times
```

**Note:** Detailed coverage of meta-annotations typically continues in next part of tutorial.

---

## Quick Comparison

### Annotations vs Regular Code

| Aspect | Without Annotation | With Annotation |
|--------|-------------------|-----------------|
| **Override check** | No compile-time check | @Override checks at compile-time |
| **Deprecation** | No warning | @Deprecated shows warning |
| **Warnings** | All warnings shown | @SuppressWarnings hides them |
| **Functional Interface** | No enforcement | @FunctionalInterface enforces rule |

---

## Best Practices

### ✅ DO:

1. **Use @Override** for all override methods
```java
@Override
public void method() { }  // Always add it
```

2. **Document deprecations** clearly
```java
/**
 * @deprecated Use {@link #newMethod()} instead
 */
@Deprecated
public void oldMethod() { }
```

3. **Be specific** with @SuppressWarnings
```java
@SuppressWarnings("deprecation")  // ✓ Specific
// Better than
@SuppressWarnings("all")  // ✗ Too broad
```

---

### ❌ DON'T:

1. **Don't suppress** all warnings blindly
```java
@SuppressWarnings("all")  // ❌ Dangerous
```

2. **Don't use @SafeVarargs** if not sure
```java
@SafeVarargs
public void method(List<String>... lists) {
    // ❌ Only use if you're CERTAIN no heap pollution
}
```

3. **Don't forget @Override**
```java
public void toString() {  // ❌ Missing @Override
    return "...";
}
```

---

## Common Interview Questions

### Q1: What is an annotation?

**Answer:** Metadata added to Java code that provides information to the compiler, runtime, or frameworks. Denoted by @ symbol. Usage is optional.

---

### Q2: Difference between @Override and normal method?

**Answer:**

| Aspect | Without @Override | With @Override |
|--------|------------------|----------------|
| Compilation | Compiles even if method doesn't exist in parent | Compilation error if method not in parent |
| Purpose | No indication of intent | Clearly shows override intent |
| Safety | Can have typos | Catches typos at compile-time |

---

### Q3: Why use @Deprecated instead of just removing the code?

**Answer:**
- **Backward compatibility:** Existing code using it won't break
- **Migration time:** Give users time to switch to new alternative
- **Warning:** Alert users to stop using it
- **Documentation:** Shows method is no longer maintained

---

### Q4: What is heap pollution?

**Answer:** When an object of one type stores reference to object of different type. Common with varargs and generics.

```java
List<Integer> list = ...;  // Declared as Integer
list = (List) stringList;   // Actually String - HEAP POLLUTION!
```

---

### Q5: Can annotations affect program behavior?

**Answer:**
- **Compile-time:** Yes (@Override causes compile errors)
- **Runtime:** Yes (via Reflection - frameworks use this)
- **Performance:** Minimal to no impact
- **Logic:** Frameworks add logic based on annotations

---

## Key Takeaways

### Essential Points

1. ✅ **Annotations are metadata** - optional but useful
2. ✅ **Start with @ symbol** - easy to identify
3. ✅ **Accessed via Reflection** - must understand reflection first
4. ✅ **Two types:** Predefined (Java provides) and Custom (you create)
5. ✅ **Can be used anywhere** - classes, methods, fields, etc.

---

### Five Main Predefined Annotations

```
1. @Deprecated     → Mark as outdated
2. @Override       → Verify method override  
3. @SuppressWarnings → Hide specific warnings
4. @FunctionalInterface → Enforce one abstract method
5. @SafeVarargs    → Suppress varargs heap pollution warning
```

---

### Remember

> **Annotations don't change code behavior directly.**  
> **They provide information that compiler/JVM/frameworks use to add behavior.**

---

## What's Next?

**Coming in Part 2:**
1. Meta-Annotations (@Retention, @Target, etc.)
2. Creating Custom Annotations
3. Reading Annotations using Reflection
4. Real-world annotation examples
5. How frameworks use annotations

---

## Practice Exercises

### Exercise 1: Basic Annotations
```java
// Add appropriate annotations
interface Animal {
    void sound();
}

class Dog implements Animal {
    // Add annotation here
    public void sound() {
        System.out.println("Bark");
    }
    
    // Add annotation here
    public void oldFeature() {
        System.out.println("Old");
    }
}
```

---

### Exercise 2: @SuppressWarnings
```java
// Suppress only necessary warnings
public class Test {
    public static void main(String[] args) {
        Mobile m = new Mobile();
        m.deprecatedMethod();  // Shows warning
        
        int unused = 10;  // Shows warning
    }
}
```

---

### Exercise 3: Varargs
```java
// Create a safe varargs method
// Should accept multiple String arrays
// Print all strings
```

---

**End of Part 1. Continue to Part 2 for Meta-Annotations and Custom Annotations! 🚀**
