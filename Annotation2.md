# Java Annotations Part 2 - Meta-Annotations & Custom Annotations

## Recap from Part 1

**Annotations** = Metadata added to Java code
- Start with `@` symbol
- Optional but useful
- Accessed via Reflection
- Two types: Predefined and Custom

---

## Meta-Annotations

### What are Meta-Annotations?

**Meta-Annotations** = Annotations used **ON other annotations**

```java
@Target(ElementType.METHOD)  // ← Meta-annotation
@Retention(RetentionPolicy.RUNTIME)  // ← Meta-annotation
public @interface MyAnnotation {
    // Custom annotation
}
```

**Purpose:** Configure behavior and usage of annotations

---

## Five Main Meta-Annotations

```
Meta-Annotations:
├── 1. @Target      → Where annotation can be applied
├── 2. @Retention   → When annotation information is available
├── 3. @Documented  → Include in JavaDoc
├── 4. @Inherited   → Inherited by subclasses
└── 5. @Repeatable  → Can be used multiple times
```

---

## 1. @Target

### Purpose

**Defines WHERE an annotation can be applied** (class, method, field, etc.)

### How It Works

```java
// Java's @Override definition
@Target(ElementType.METHOD)  // Can only be used on methods
public @interface Override {
}

// Usage
class Bird {
    @Override  // ✓ Valid - used on method
    public String toString() {
        return "Bird";
    }
}

@Override  // ❌ Error - cannot use on class
class Eagle { }
```

---

### ElementType Values

| ElementType | Where It Can Be Applied | Example |
|-------------|------------------------|---------|
| **TYPE** | Class, Interface, Enum | `@Deprecated class MyClass` |
| **FIELD** | Field (member variable) | `@Deprecated private int value;` |
| **METHOD** | Method | `@Override public void method()` |
| **PARAMETER** | Method parameter | `void method(@Param String s)` |
| **CONSTRUCTOR** | Constructor | `@Deprecated public MyClass()` |
| **LOCAL_VARIABLE** | Local variable inside method | `@SuppressWarnings int x = 10;` |
| **ANNOTATION_TYPE** | Another annotation (Meta) | `@Target(ElementType.METHOD)` |
| **PACKAGE** | Package declaration | In `package-info.java` |
| **TYPE_PARAMETER** | Generic type parameter | `class Box<@TypeParam T>` |
| **TYPE_USE** | Any use of type | `List<@NonNull String>` |

---

### Example: @SafeVarargs Definition

```java
// How Java defined @SafeVarargs
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD})
public @interface SafeVarargs {
}

// Usage
class Example {
    @SafeVarargs  // ✓ Valid - method
    public static void method(List<String>... lists) { }
    
    @SafeVarargs  // ✓ Valid - constructor
    public Example(List<String>... lists) { }
    
    @SafeVarargs  // ❌ Error - cannot use on field
    private int field;
}
```

---

### Multiple Targets

```java
@Target({ElementType.METHOD, ElementType.FIELD})
public @interface MyAnnotation {
}

// Can use on both
class Example {
    @MyAnnotation  // ✓ Valid - field
    private int value;
    
    @MyAnnotation  // ✓ Valid - method
    public void method() { }
}
```

---

### ANNOTATION_TYPE Example

```java
// @Target itself uses ANNOTATION_TYPE
@Target(ElementType.ANNOTATION_TYPE)  // Can be used on annotations
public @interface Target {
    ElementType[] value();
}

// Therefore, we can use @Target on other annotations
@Target(ElementType.METHOD)  // ✓ Valid - using on annotation
public @interface MyAnnotation {
}
```

---

### TYPE_USE (Java 8+)

**Allows annotation on ANY type usage**

```java
@Target(ElementType.TYPE_USE)
public @interface NonNull {
}

// Usage
class Example {
    // Before variable type
    @NonNull String name;
    
    // In generic type
    List<@NonNull String> names;
    
    // In cast
    String s = (@NonNull String) obj;
    
    // In new instance
    new @NonNull String();
}
```

---

## 2. @Retention

### Purpose

**Defines HOW LONG annotation information is retained**

### Three Retention Policies

```
SOURCE  →  Compile Time  →  Runtime
   ↓           ↓              ↓
Discarded   In .class    Available
by compiler  but not     via
             at runtime  Reflection
```

---

### RetentionPolicy.SOURCE

**Annotation discarded after compilation**

```java
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```

**What happens:**
```
Eagle.java (source):
class Eagle {
    @Override
    public String toString() { ... }
}

↓ Compilation

Eagle.class (bytecode):
class Eagle {
    // @Override is GONE!
    public String toString() { ... }
}
```

**Why?** @Override only needed during compilation to check validity. Not needed at runtime.

---

### RetentionPolicy.CLASS (Default)

**Annotation recorded in .class file but NOT available at runtime**

```java
@Retention(RetentionPolicy.CLASS)  // Default if not specified
public @interface MyAnnotation {
}
```

**What happens:**
```
MyClass.java:
@MyAnnotation
class MyClass { }

↓ Compilation

MyClass.class:
@MyAnnotation  // Present in bytecode
class MyClass { }

↓ Runtime

Cannot access via Reflection ❌
```

**Use Case:** Tools that process bytecode (not runtime)

---

### RetentionPolicy.RUNTIME

**Annotation available at runtime via Reflection**

```java
@Retention(RetentionPolicy.RUNTIME)  // IMPORTANT!
public @interface SafeVarargs {
}
```

**What happens:**
```
MyClass.java:
@SafeVarargs
class MyClass { }

↓ Compilation

MyClass.class:
@SafeVarargs  // Present
class MyClass { }

↓ Runtime

CAN access via Reflection ✓
```

---

### Practical Example: Accessing Annotations

#### With RUNTIME Retention

```java
// Define annotation with RUNTIME
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface MyAnnotation {
    String value();
}

// Use annotation
@MyAnnotation("TestValue")
class TestClass { }

// Access at runtime
public class Main {
    public static void main(String[] args) {
        TestClass obj = new TestClass();
        Class<?> clazz = obj.getClass();
        
        // Get annotation
        MyAnnotation annotation = clazz.getAnnotation(MyAnnotation.class);
        
        System.out.println(annotation.value());  // Output: TestValue ✓
    }
}
```

---

#### Without RUNTIME Retention

```java
// Define annotation WITHOUT retention (defaults to CLASS)
@Target(ElementType.TYPE)
public @interface MyAnnotation {
    String value();
}

// Use annotation
@MyAnnotation("TestValue")
class TestClass { }

// Try to access at runtime
public class Main {
    public static void main(String[] args) {
        TestClass obj = new TestClass();
        Class<?> clazz = obj.getClass();
        
        MyAnnotation annotation = clazz.getAnnotation(MyAnnotation.class);
        
        System.out.println(annotation);  // Output: null ❌
        // Annotation not available at runtime!
    }
}
```

---

### Retention Summary

| Policy | In Source | In .class | At Runtime | Use Case |
|--------|-----------|-----------|------------|----------|
| **SOURCE** | ✓ | ✗ | ✗ | Compile-time checks (@Override) |
| **CLASS** | ✓ | ✓ | ✗ | Bytecode processing tools |
| **RUNTIME** | ✓ | ✓ | ✓ | Frameworks using Reflection |

**Rule of Thumb:** If you need Reflection → Use RUNTIME

---

## 3. @Documented

### Purpose

**Include annotation in JavaDoc documentation**

### Without @Documented

```java
// @Override does NOT have @Documented
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}

// Usage
class Eagle {
    @Override
    public String toString() {
        return "Eagle";
    }
}

// Generate JavaDoc
```

**JavaDoc Output:**
```
Method: toString
Returns: String

// No mention of @Override annotation
```

---

### With @Documented

```java
// @SafeVarargs HAS @Documented
@Documented
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface SafeVarargs {
}

// Usage
class Example {
    @SafeVarargs
    public static void method(List<String>... lists) {
    }
}

// Generate JavaDoc
```

**JavaDoc Output:**
```
Method: method
Annotations: @SafeVarargs  ← Shows in documentation!
Parameters: List<String>... lists
```

---

### How to Generate JavaDoc

**In IntelliJ IDEA:**
```
Tools → Generate JavaDoc...
```

**In Eclipse:**
```
Project → Generate JavaDoc...
```

**Command Line:**
```bash
javadoc -d docs MyClass.java
```

---

### When to Use @Documented

**Use @Documented when:**
- Annotation provides important information for API users
- Want developers to see annotation in documentation
- Annotation affects behavior users should know about

**Don't use when:**
- Internal implementation detail
- Not relevant to API users
- Clutters documentation

---

## 4. @Inherited

### Purpose

**Allows annotation to be inherited by subclasses**

**Default:** Annotations are NOT inherited

---

### Without @Inherited

```java
// Custom annotation WITHOUT @Inherited
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface MyAnnotation {
}

// Parent class with annotation
@MyAnnotation
class Parent { }

// Child class WITHOUT annotation
class Child extends Parent { }

// Check at runtime
public class Main {
    public static void main(String[] args) {
        Class<?> childClass = Child.class;
        MyAnnotation annotation = childClass.getAnnotation(MyAnnotation.class);
        
        System.out.println(annotation);  // Output: null ❌
        // Child does NOT inherit annotation
    }
}
```

---

### With @Inherited

```java
// Custom annotation WITH @Inherited
@Inherited  // IMPORTANT!
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface MyAnnotation {
}

// Parent class with annotation
@MyAnnotation
class Parent { }

// Child class WITHOUT annotation (but inherits it!)
class Child extends Parent { }

// Check at runtime
public class Main {
    public static void main(String[] args) {
        Class<?> childClass = Child.class;
        MyAnnotation annotation = childClass.getAnnotation(MyAnnotation.class);
        
        System.out.println(annotation);  // Output: @MyAnnotation() ✓
        // Child INHERITS annotation from Parent!
    }
}
```

---

### Important Notes

**Only works for:**
- **Classes** (not interfaces)
- **TYPE** level annotations

**Does NOT work for:**
- Methods
- Fields
- Constructors

```java
@Inherited
@Target(ElementType.TYPE)  // ✓ Works - class level
public @interface ClassAnnotation { }

@Inherited
@Target(ElementType.METHOD)  // ✗ No effect - methods not inherited
public @interface MethodAnnotation { }
```

---

## 5. @Repeatable (Java 8+)

### Purpose

**Allow same annotation to be used multiple times in same place**

### Without @Repeatable

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Category {
    String name();
}

// Try to use multiple times
@Category(name = "Bird")
@Category(name = "Animal")  // ❌ Compilation Error!
// Duplicate annotation error
class Eagle { }
```

---

### With @Repeatable - Two Steps

#### Step 1: Create Container Annotation

```java
// Container annotation to hold multiple Category annotations
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Categories {
    Category[] value();  // Array of Category annotations
}
```

---

#### Step 2: Mark Original Annotation as @Repeatable

```java
@Repeatable(Categories.class)  // Specify container
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Category {
    String name();
}
```

---

#### Step 3: Use Multiple Times

```java
@Category(name = "Bird")
@Category(name = "Animal")
@Category(name = "Carnivorous")
class Eagle { }
```

---

### Complete Working Example

```java
// 1. Original annotation
@Repeatable(Categories.class)
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Category {
    String name();
}

// 2. Container annotation
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Categories {
    Category[] value();
}

// 3. Use multiple times
@Category(name = "Bird")
@Category(name = "LivingThing")
@Category(name = "Carnivorous")
class Eagle { }

// 4. Access at runtime
public class Main {
    public static void main(String[] args) {
        Eagle eagle = new Eagle();
        Category[] categories = eagle.getClass()
                                     .getAnnotationsByType(Category.class);
        
        for (Category category : categories) {
            System.out.println(category.name());
        }
        // Output:
        // Bird
        // LivingThing
        // Carnivorous
    }
}
```

---

### @Repeatable Requirements

**Must have:**
1. Container annotation with array of original annotation
2. @Repeatable on original annotation pointing to container
3. Same @Target on both
4. Same or broader @Retention on container

```java
// ✓ Correct setup
@Repeatable(Categories.class)
@Retention(RetentionPolicy.RUNTIME)  // Same
@Target(ElementType.TYPE)            // Same
public @interface Category { }

@Retention(RetentionPolicy.RUNTIME)  // Same or broader
@Target(ElementType.TYPE)            // Same
public @interface Categories {
    Category[] value();
}
```

---

## Creating Custom Annotations

### Basic Syntax

```java
@interface AnnotationName {
    // annotation body
}
```

**Note:** Use `@interface` (not `interface`)

---

### 1. Empty Annotation

```java
public @interface MyAnnotation {
    // No members
}

// Usage
@MyAnnotation
class MyClass { }
```

**Characteristics:**
- No metadata stored
- Just acts as a marker
- Can still use meta-annotations

---

### 2. Annotation with Members

**Members look like methods but act like fields**

```java
public @interface MyAnnotation {
    String name();      // Member (looks like method)
    int value();        // Member
}

// Usage
@MyAnnotation(name = "Test", value = 10)
class MyClass { }
```

---

### Member Rules

| Rule | Description | Example |
|------|-------------|---------|
| **No parameters** | Members cannot have parameters | `String name();` ✓<br>`String name(int x);` ✗ |
| **No body** | Members cannot have body | `String name();` ✓<br>`String name() { }` ✗ |
| **Return type** | Limited types only | Primitive, String, Class, Enum, Annotation, Array of these |

---

### Allowed Return Types

```java
public @interface AllTypes {
    // Primitives
    int intValue();
    boolean boolValue();
    double doubleValue();
    
    // String
    String stringValue();
    
    // Class
    Class<?> classValue();
    
    // Enum
    MyEnum enumValue();
    
    // Another annotation
    MyOtherAnnotation annotationValue();
    
    // Arrays
    int[] intArray();
    String[] stringArray();
    Class<?>[] classArray();
}
```

---

### NOT Allowed Return Types

```java
public @interface Invalid {
    // ❌ Custom objects
    MyCustomClass customValue();  // ERROR!
    
    // ❌ Collections
    List<String> listValue();  // ERROR!
    
    // ❌ Interfaces (non-annotation)
    MyInterface interfaceValue();  // ERROR!
}
```

---

### 3. Annotation with Default Values

```java
public @interface MyAnnotation {
    String name() default "DefaultName";
    int value() default 0;
}

// Usage - with values
@MyAnnotation(name = "Custom", value = 10)
class Class1 { }

// Usage - without values (uses defaults)
@MyAnnotation  // name = "DefaultName", value = 0
class Class2 { }

// Usage - partial values
@MyAnnotation(name = "Custom")  // value = 0
class Class3 { }
```

---

### Single-Value Annotations

**Special case:** If annotation has only one member named `value`, you can omit the name

```java
public @interface MyAnnotation {
    String value();  // Member named 'value'
}

// Normal usage
@MyAnnotation(value = "Test")
class Class1 { }

// Shorthand (omit 'value =')
@MyAnnotation("Test")  // Same as above
class Class2 { }
```

**Multiple members:** Must specify names

```java
public @interface MyAnnotation {
    String value();
    int number();
}

// Must use names
@MyAnnotation(value = "Test", number = 10)  // ✓ Correct
@MyAnnotation("Test", 10)  // ❌ Error
```

---

## Complete Custom Annotation Example

### Step 1: Define Annotation

```java
@Retention(RetentionPolicy.RUNTIME)  // Available at runtime
@Target(ElementType.TYPE)            // Can use on classes
@Documented                          // Include in JavaDoc
public @interface Author {
    String name();
    String date();
    int version() default 1;
}
```

---

### Step 2: Use Annotation

```java
@Author(
    name = "John Doe",
    date = "2024-01-15",
    version = 2
)
public class MyClass {
    public void method() {
        System.out.println("My method");
    }
}
```

---

### Step 3: Access at Runtime

```java
public class Main {
    public static void main(String[] args) {
        // Get class object
        Class<?> clazz = MyClass.class;
        
        // Check if annotation present
        if (clazz.isAnnotationPresent(Author.class)) {
            // Get annotation
            Author author = clazz.getAnnotation(Author.class);
            
            // Access members
            System.out.println("Author: " + author.name());
            System.out.println("Date: " + author.date());
            System.out.println("Version: " + author.version());
        }
    }
}

// Output:
// Author: John Doe
// Date: 2024-01-15
// Version: 2
```

---

## Real-World Custom Annotation Example

### Test Framework Annotation

```java
// Define annotation
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
    String description() default "";
    boolean enabled() default true;
    int timeout() default 1000;
}

// Use annotation
public class MyTests {
    
    @Test(description = "Tests addition", timeout = 500)
    public void testAddition() {
        // Test code
    }
    
    @Test(enabled = false)
    public void disabledTest() {
        // This test will be skipped
    }
}

// Test runner
public class TestRunner {
    public static void runTests(Class<?> testClass) {
        Method[] methods = testClass.getDeclaredMethods();
        
        for (Method method : methods) {
            if (method.isAnnotationPresent(Test.class)) {
                Test test = method.getAnnotation(Test.class);
                
                if (test.enabled()) {
                    System.out.println("Running: " + test.description());
                    // Execute test method
                } else {
                    System.out.println("Skipping: " + method.getName());
                }
            }
        }
    }
}
```

---

## Meta-Annotations Summary Table

| Meta-Annotation | Purpose | Example Value |
|-----------------|---------|---------------|
| **@Target** | Where annotation can be used | `ElementType.METHOD` |
| **@Retention** | How long annotation is kept | `RetentionPolicy.RUNTIME` |
| **@Documented** | Include in JavaDoc | No value needed |
| **@Inherited** | Inherit to subclasses | No value needed |
| **@Repeatable** | Use multiple times | Container class |

---

## Common Annotation Patterns

### Pattern 1: Marker Annotation

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Entity {
    // No members - just marks a class
}
```

---

### Pattern 2: Single-Value Annotation

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface Column {
    String value();  // Single member named 'value'
}

// Usage
@Column("user_name")
private String username;
```

---

### Pattern 3: Multi-Value Annotation

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface RequestMapping {
    String path();
    String method() default "GET";
    String[] params() default {};
}

// Usage
@RequestMapping(
    path = "/users",
    method = "POST",
    params = {"id", "name"}
)
public void createUser() { }
```

---

## Best Practices

### ✅ DO:

1. **Always specify @Target and @Retention**
```java
@Retention(RetentionPolicy.RUNTIME)  // Always specify
@Target(ElementType.METHOD)          // Always specify
public @interface MyAnnotation { }
```

2. **Provide default values** for optional members
```java
public @interface Config {
    String name();              // Required
    boolean enabled() default true;  // Optional with default
}
```

3. **Use meaningful names**
```java
@interface Author { }  // ✓ Clear
@interface A { }       // ✗ Unclear
```

---

### ❌ DON'T:

1. **Don't create annotations without clear purpose**
```java
// ❌ Bad
@interface RandomAnnotation { }
```

2. **Don't use complex return types**
```java
// ❌ Bad
@interface Bad {
    MyCustomClass custom();  // Not allowed!
}
```

3. **Don't forget RUNTIME retention** if using Reflection
```java
// ❌ Bad - can't access at runtime
@Retention(RetentionPolicy.CLASS)
public @interface MyAnnotation { }
```

---

## Quick Reference

### Creating Custom Annotation Checklist

```
☐ 1. Use @interface keyword
☐ 2. Add @Target (where it can be used)
☐ 3. Add @Retention (when it's available)
☐ 4. Add @Documented (if should appear in JavaDoc)
☐ 5. Add @Inherited (if should inherit to subclasses)
☐ 6. Define members (if needed)
☐ 7. Provide default values (if optional)
☐ 8. Test with Reflection (if RUNTIME)
```

---

### Template for Custom Annotation

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)  // Change as needed
@Documented
public @interface MyAnnotation {
    String name();
    int value() default 0;
}
```

---

## Interview Questions

### Q1: What's the difference between @Target and @Retention?

**Answer:**
- **@Target:** WHERE annotation can be applied (method, class, field, etc.)
- **@Retention:** WHEN annotation information is available (SOURCE, CLASS, RUNTIME)

---

### Q2: Can you use an annotation multiple times?

**Answer:** Yes, but only if annotated with @Repeatable (Java 8+). Must also create a container annotation.

---

### Q3: How to access custom annotations at runtime?

**Answer:**
```java
// 1. Must use RUNTIME retention
// 2. Use Reflection
Class<?> clazz = MyClass.class;
MyAnnotation annotation = clazz.getAnnotation(MyAnnotation.class);
```

---

### Q4: What are allowed return types for annotation members?

**Answer:**
- Primitives (int, boolean, etc.)
- String
- Class
- Enum
- Another annotation
- Arrays of above types

**NOT allowed:** Custom objects, Collections, Interfaces

---

### Q5: What's the purpose of @Inherited?

**Answer:** Makes annotation inheritable by subclasses. Only works for class-level annotations. Child class automatically inherits parent's annotation.

---

## Key Takeaways

### Essential Points

1. ✅ **Meta-annotations** configure other annotations
2. ✅ **@Target** controls WHERE annotation can be used
3. ✅ **@Retention** controls WHEN annotation is available
4. ✅ **Use RUNTIME** retention if accessing via Reflection
5. ✅ **@Repeatable** requires container annotation
6. ✅ **Custom annotations** use `@interface` keyword
7. ✅ **Limited return types** for annotation members

---

### Must Remember

> **@Retention(RetentionPolicy.RUNTIME)** is crucial for Reflection!  
> Without it, annotation won't be available at runtime.

> **@interface** creates annotation (not regular interface)

> **Meta-annotations** must have @Target(ElementType.ANNOTATION_TYPE)

---

## What's Next?

**Advanced Topics:**
1. Annotation processors (compile-time processing)
2. Custom annotation validation
3. Framework-specific annotations (Spring, JPA, etc.)
4. Annotation inheritance complexities
5. Performance considerations

---

**Congratulations! You now understand:**
- ✓ All predefined annotations
- ✓ All meta-annotations
- ✓ How to create custom annotations
- ✓ How to access annotations via Reflection
- ✓ Real-world annotation patterns

**Keep practicing and happy coding! 🚀**
