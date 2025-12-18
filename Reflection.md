# Java Reflection API - Complete Guide

## What is Reflection?

**Reflection** is a powerful feature in Java that allows you to **examine and modify** classes, methods, fields, and interfaces **at runtime**.

### Core Capabilities

```
Reflection allows you to:
├─ Examine classes, methods, fields at runtime
├─ Change behavior of classes (modify field values)
├─ Invoke methods dynamically
├─ Access private members (break encapsulation)
└─ Create objects dynamically
```

### What You Can Do with Reflection

| Capability | Description |
|------------|-------------|
| **Examine** | Get class name, methods, fields, constructors |
| **Inspect** | Check modifiers, return types, parameters |
| **Invoke** | Call methods dynamically |
| **Modify** | Change public AND private field values |
| **Create** | Instantiate objects dynamically |

---

## The `Class` Class

### Understanding the Class Class

**Key Concept:** Every class in Java has an associated `Class` object created by JVM.

```
Your Code:
├─ class Bird { }
├─ class Animal { }
├─ class Lion { }

JVM Creates (automatically):
├─ Class object for Bird
├─ Class object for Animal  
├─ Class object for Lion
```

### What is the Class Object?

```java
// When you write:
public class Bird {
    private String breed;
    public void fly() { }
}

// JVM creates:
Class<?> birdClass = // ... metadata container
    ├─ Contains: All methods
    ├─ Contains: All fields
    ├─ Contains: All constructors
    ├─ Contains: Modifiers
    └─ Contains: Return types, parameters, etc.
```

**Purpose:** The `Class` object stores **metadata** about your class.

### When is Class Object Created?

**JVM creates one `Class` object for each class when:**
1. Class is loaded into memory
2. First time class is used in program

```
Program Flow:
1. Your code uses Bird class
2. JVM loads Bird class
3. JVM automatically creates Class object for Bird
4. Class object contains all metadata about Bird
```

---

## Three Ways to Get Class Object

### Method 1: Using `Class.forName()`

```java
// For class named Bird
Class<?> birdClass = Class.forName("Bird");

// Full package path
Class<?> cls = Class.forName("com.example.Bird");
```

**Use Case:** When you have class name as a String

---

### Method 2: Using `.class` Literal

```java
// Direct access
Class<?> birdClass = Bird.class;
```

**Use Case:** When you know the class at compile time

---

### Method 3: Using `getClass()` Method

```java
// Create object first
Bird bird = new Bird();

// Get its Class object
Class<?> birdClass = bird.getClass();
```

**Use Case:** When you already have an object instance

---

### Comparison of Three Methods

| Method | Code | When to Use |
|--------|------|-------------|
| **forName()** | `Class.forName("Bird")` | Class name is a String |
| **.class** | `Bird.class` | Know class at compile time |
| **getClass()** | `object.getClass()` | Already have object instance |

---

## Reflection of Classes

### Example Class

```java
public class Eagle {
    // Fields
    public String breed;
    private boolean canSwim;
    
    // Methods
    public void fly() {
        System.out.println("Eagle is flying");
    }
    
    private void eat() {
        System.out.println("Eagle is eating");
    }
}
```

### Getting Class Metadata

```java
// Step 1: Get Class object
Class<?> eagleClass = Eagle.class;

// Step 2: Get metadata
String className = eagleClass.getName();        // "Eagle"
int modifiers = eagleClass.getModifiers();      // public
Field[] fields = eagleClass.getFields();        // All public fields
Method[] methods = eagleClass.getMethods();     // All public methods
Constructor[] constructors = eagleClass.getConstructors();
```

### Available Class Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `getName()` | String | Class name |
| `getModifiers()` | int | Modifier (public, private, etc.) |
| `getFields()` | Field[] | All **public** fields |
| `getDeclaredFields()` | Field[] | **All** fields (public + private) |
| `getMethods()` | Method[] | All **public** methods |
| `getDeclaredMethods()` | Method[] | **All** methods (public + private) |
| `getConstructors()` | Constructor[] | All **public** constructors |
| `getDeclaredConstructors()` | Constructor[] | **All** constructors |

**Key Difference:**
- `getXxx()` → Only **public** members
- `getDeclaredXxx()` → **All** members (public + private)

---

## Reflection of Methods

### Example: Get Public Methods Only

```java
public class Eagle {
    public void fly() {
        System.out.println("Flying");
    }
    
    private void eat() {
        System.out.println("Eating");
    }
}

// Get only PUBLIC methods
Class<?> eagleClass = Eagle.class;
Method[] methods = eagleClass.getMethods();  // Returns public methods only

for (Method method : methods) {
    System.out.println(method.getName());           // Method name
    System.out.println(method.getReturnType());     // Return type
    System.out.println(method.getDeclaringClass()); // Declaring class
}

// Output:
// fly
// void
// class Eagle
// wait      ← From Object class (parent)
// notify    ← From Object class
// ... (all public methods from Object)
```

**Important:** `getMethods()` returns:
- All public methods in the class
- All public methods from parent classes (including Object)

---

### Example: Get ALL Methods (Public + Private)

```java
// Get ALL methods (public + private) in THIS class only
Method[] allMethods = eagleClass.getDeclaredMethods();

for (Method method : allMethods) {
    System.out.println(method.getName());
}

// Output:
// fly
// eat
// (Does NOT include Object class methods)
```

**Important:** `getDeclaredMethods()` returns:
- All methods (public + private)
- Only from THIS class (not parent classes)

---

### Method Metadata

```java
Method method = eagleClass.getMethod("fly");

// Get metadata
String name = method.getName();                    // "fly"
Class<?> returnType = method.getReturnType();      // void.class
int modifiers = method.getModifiers();             // public
Class<?>[] params = method.getParameterTypes();    // []
Class<?>[] exceptions = method.getExceptionTypes(); // []
Class<?> declaringClass = method.getDeclaringClass(); // Eagle.class
```

---

## Invoking Methods Using Reflection

### Example: Method with Parameters

```java
public class Eagle {
    public void fly(int speed, boolean high, String direction) {
        System.out.println("Flying at " + speed + " km/h, "
                         + "high: " + high + ", "
                         + "direction: " + direction);
    }
}
```

### Invoking the Method

```java
// Step 1: Get Class object
Class<?> eagleClass = Class.forName("Eagle");

// Step 2: Create instance of Eagle
Object eagleInstance = eagleClass.newInstance();  // Calls constructor

// Step 3: Get the method
Method flyMethod = eagleClass.getMethod(
    "fly",                    // Method name
    int.class,               // First parameter type
    boolean.class,           // Second parameter type
    String.class             // Third parameter type
);

// Step 4: Invoke the method
flyMethod.invoke(
    eagleInstance,           // Object to invoke on
    100,                     // First argument (speed)
    true,                    // Second argument (high)
    "North"                  // Third argument (direction)
);

// Output: Flying at 100 km/h, high: true, direction: North
```

### Method Invocation Breakdown

```java
method.invoke(objectInstance, arg1, arg2, arg3, ...);
              │               │
              │               └─ Arguments to pass
              └─ Object to invoke method on
```

---

## Reflection of Fields

### Example Class

```java
public class Eagle {
    public String breed;
    private boolean canSwim;
}
```

### Get Public Fields Only

```java
Class<?> eagleClass = Eagle.class;

// Get only PUBLIC fields
Field[] fields = eagleClass.getFields();

for (Field field : fields) {
    System.out.println("Name: " + field.getName());       // breed
    System.out.println("Type: " + field.getType());       // String
    System.out.println("Modifier: " + field.getModifiers()); // public
}

// Output shows only: breed (public field)
```

---

### Get ALL Fields (Public + Private)

```java
// Get ALL fields (public + private)
Field[] allFields = eagleClass.getDeclaredFields();

for (Field field : allFields) {
    System.out.println("Name: " + field.getName());
    System.out.println("Type: " + field.getType());
    System.out.println("Modifier: " + field.getModifiers());
}

// Output shows:
// breed - String - public
// canSwim - boolean - private
```

---

### Field Metadata Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `getName()` | String | Field name |
| `getType()` | Class<?> | Field data type |
| `getModifiers()` | int | Access modifier |
| `get(object)` | Object | Get field value |
| `set(object, value)` | void | Set field value |

---

## Changing Field Values

### Setting Public Field Value

```java
public class Eagle {
    public String breed;
}

// Step 1: Get Class object
Class<?> eagleClass = Eagle.class;

// Step 2: Get the field
Field breedField = eagleClass.getDeclaredField("breed");

// Step 3: Create instance (or use existing)
Object eagleInstance = eagleClass.newInstance();

// Step 4: Set field value
breedField.set(eagleInstance, "Brown Eagle");

// Step 5: Verify
Eagle eagle = (Eagle) eagleInstance;
System.out.println(eagle.breed);  // Output: Brown Eagle
```

---

### Setting Private Field Value

#### ❌ Direct Access Throws Exception

```java
public class Eagle {
    private boolean canSwim;
}

Class<?> eagleClass = Eagle.class;
Field swimField = eagleClass.getDeclaredField("canSwim");
Object eagleInstance = eagleClass.newInstance();

// This will FAIL!
swimField.set(eagleInstance, true);

// Exception: IllegalAccessException
// Main class cannot access private member of Eagle class
```

---

#### ✅ Correct Way: Use `setAccessible(true)`

```java
public class Eagle {
    private boolean canSwim;
}

// Step 1: Get Class object
Class<?> eagleClass = Eagle.class;

// Step 2: Get private field
Field swimField = eagleClass.getDeclaredField("canSwim");

// Step 3: Make it accessible (IMPORTANT!)
swimField.setAccessible(true);  // Bypass private access check

// Step 4: Create instance
Object eagleInstance = eagleClass.newInstance();

// Step 5: Set value (now works!)
swimField.set(eagleInstance, true);

// Step 6: Verify
Eagle eagle = (Eagle) eagleInstance;
// Note: Can't access directly (still private)
// But value IS changed internally!
```

**Key Point:** `setAccessible(true)` grants access to private members!

---

## Reflection of Constructors

### Example: Private Constructor

```java
public class Eagle {
    // Private constructor
    private Eagle() {
        System.out.println("Eagle created");
    }
    
    public void fly() {
        System.out.println("Flying");
    }
}
```

### Accessing Private Constructor

```java
// Step 1: Get Class object
Class<?> eagleClass = Eagle.class;

// Step 2: Get ALL constructors (including private)
Constructor<?>[] constructors = eagleClass.getDeclaredConstructors();

for (Constructor<?> constructor : constructors) {
    System.out.println("Modifier: " + constructor.getModifiers());
    
    // Step 3: Make private constructor accessible
    constructor.setAccessible(true);
    
    // Step 4: Create new instance using constructor
    Object eagleInstance = constructor.newInstance();
    
    // Step 5: Cast and use
    Eagle eagle = (Eagle) eagleInstance;
    eagle.fly();
}

// Output:
// Modifier: private
// Eagle created
// Flying
```

---

## How Reflection Breaks Singleton

### Singleton Pattern (Normal)

```java
public class Singleton {
    private static Singleton instance;
    
    // Private constructor - no direct instantiation
    private Singleton() { }
    
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}

// Usage
Singleton s1 = Singleton.getInstance();
Singleton s2 = Singleton.getInstance();
// s1 == s2 (same object) ✓
```

---

### Breaking Singleton with Reflection

```java
// Get first instance (normal way)
Singleton instance1 = Singleton.getInstance();

// Use reflection to create second instance
Class<?> singletonClass = Singleton.class;
Constructor<?> constructor = singletonClass.getDeclaredConstructor();
constructor.setAccessible(true);  // Bypass private!

Singleton instance2 = (Singleton) constructor.newInstance();

// Now we have TWO instances!
System.out.println(instance1 == instance2);  // false ❌
// Singleton pattern broken!
```

**Problem:** Reflection can access private constructor and create multiple instances!

---

### Protecting Singleton from Reflection

```java
public class Singleton {
    private static Singleton instance;
    
    private Singleton() {
        // Protection against reflection
        if (instance != null) {
            throw new RuntimeException("Use getInstance() method!");
        }
    }
    
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}

// Now if reflection tries to create second instance:
Constructor<?> constructor = Singleton.class.getDeclaredConstructor();
constructor.setAccessible(true);
Singleton instance2 = (Singleton) constructor.newInstance();
// Throws RuntimeException! ✓
```

---

## Package Information

### Where Reflection Classes Are Located

All reflection-related classes are in: `java.lang.reflect` package

```java
import java.lang.reflect.Method;
import java.lang.reflect.Field;
import java.lang.reflect.Constructor;
import java.lang.reflect.Modifier;
```

**Exception:** `Class` is in `java.lang` (imported by default)

---

## Disadvantages of Reflection

### 1. Breaks Encapsulation

**Problem:** Can access and modify private members

```java
public class BankAccount {
    private double balance = 1000.0;  // Should be protected!
}

// Reflection breaks it
Class<?> cls = BankAccount.class;
Field balanceField = cls.getDeclaredField("balance");
balanceField.setAccessible(true);

BankAccount account = new BankAccount();
balanceField.set(account, 999999.0);  // Changed private field!

// Encapsulation violated! ❌
```

**OOP Principle Violated:**
- Private fields should be accessible only through public methods
- Reflection bypasses this completely

---

### 2. Performance Issues - Slower Execution

**Why Reflection is Slow:**

```
Normal Method Call:
├─ Compile time: Method resolved
├─ Runtime: Direct call
└─ Fast! ✓

Reflection Method Call:
├─ Compile time: Nothing resolved
├─ Runtime: 
│   ├─ Search for method by name
│   ├─ Verify method exists
│   ├─ Check parameters match
│   ├─ Validate access permissions
│   └─ Then invoke
└─ Slow! ❌
```

**Comparison:**

```java
// Direct call (Fast)
Eagle eagle = new Eagle();
eagle.fly();  // Instant

// Reflection call (Slow)
Class<?> cls = Eagle.class;
Method method = cls.getMethod("fly");
method.invoke(eagle);  // Much slower
```

---

### 3. Increased Code Complexity

**Example:**

```java
// Direct access (Simple)
eagle.breed = "Golden Eagle";

// Reflection (Complex)
Class<?> cls = Eagle.class;
Field field = cls.getDeclaredField("breed");
field.setAccessible(true);
field.set(eagle, "Golden Eagle");

// Same result, but 4x more code!
```

---

### 4. Type Safety Lost at Compile Time

```java
// Direct call - compile-time error
eagle.flyy();  // Method doesn't exist - won't compile ✓

// Reflection - runtime error only
Method method = cls.getMethod("flyy");  // Compiles fine!
method.invoke(eagle);  // RuntimeException at execution ❌
```

---

## When to Use Reflection

### ✅ Use Reflection When:

| Scenario | Example |
|----------|---------|
| **Frameworks & Libraries** | Spring, Hibernate (dependency injection) |
| **Plugin Systems** | Loading classes dynamically |
| **Testing Frameworks** | JUnit, Mockito (accessing private methods) |
| **Serialization** | Jackson, Gson (object-JSON conversion) |
| **Annotation Processing** | Reading annotations at runtime |

### ❌ Avoid Reflection When:

| Scenario | Why Avoid |
|----------|-----------|
| **Regular Application Code** | Slower, complex, breaks encapsulation |
| **Performance Critical Code** | Reflection is slow |
| **Simple Object Access** | Use direct access instead |
| **Type Safety Important** | Lose compile-time checks |

---

## Complete Example - All Concepts

### Sample Class

```java
public class Eagle {
    // Fields
    public String breed = "Golden";
    private boolean canSwim = false;
    
    // Constructors
    public Eagle() { }
    
    private Eagle(String breed) {
        this.breed = breed;
    }
    
    // Methods
    public void fly(int speed) {
        System.out.println("Flying at " + speed);
    }
    
    private void eat(String food) {
        System.out.println("Eating " + food);
    }
}
```

---

### Complete Reflection Demo

```java
public class ReflectionDemo {
    public static void main(String[] args) throws Exception {
        
        // ========================================
        // 1. GET CLASS OBJECT
        // ========================================
        Class<?> eagleClass = Eagle.class;
        System.out.println("Class Name: " + eagleClass.getName());
        
        // ========================================
        // 2. EXAMINE FIELDS
        // ========================================
        System.out.println("\n--- All Fields ---");
        Field[] fields = eagleClass.getDeclaredFields();
        for (Field field : fields) {
            System.out.println(field.getName() + " - " + field.getType());
        }
        
        // ========================================
        // 3. MODIFY PUBLIC FIELD
        // ========================================
        System.out.println("\n--- Modify Public Field ---");
        Eagle eagle = new Eagle();
        Field breedField = eagleClass.getDeclaredField("breed");
        breedField.set(eagle, "Bald Eagle");
        System.out.println("New breed: " + eagle.breed);
        
        // ========================================
        // 4. MODIFY PRIVATE FIELD
        // ========================================
        System.out.println("\n--- Modify Private Field ---");
        Field swimField = eagleClass.getDeclaredField("canSwim");
        swimField.setAccessible(true);  // Important!
        swimField.set(eagle, true);
        System.out.println("Can swim set to: " + swimField.get(eagle));
        
        // ========================================
        // 5. EXAMINE METHODS
        // ========================================
        System.out.println("\n--- All Methods ---");
        Method[] methods = eagleClass.getDeclaredMethods();
        for (Method method : methods) {
            System.out.println(method.getName());
        }
        
        // ========================================
        // 6. INVOKE PUBLIC METHOD
        // ========================================
        System.out.println("\n--- Invoke Public Method ---");
        Method flyMethod = eagleClass.getMethod("fly", int.class);
        flyMethod.invoke(eagle, 120);
        
        // ========================================
        // 7. INVOKE PRIVATE METHOD
        // ========================================
        System.out.println("\n--- Invoke Private Method ---");
        Method eatMethod = eagleClass.getDeclaredMethod("eat", String.class);
        eatMethod.setAccessible(true);  // Important!
        eatMethod.invoke(eagle, "Fish");
        
        // ========================================
        // 8. ACCESS PRIVATE CONSTRUCTOR
        // ========================================
        System.out.println("\n--- Private Constructor ---");
        Constructor<?> privateConstructor = 
            eagleClass.getDeclaredConstructor(String.class);
        privateConstructor.setAccessible(true);
        Eagle eagle2 = (Eagle) privateConstructor.newInstance("White Eagle");
        System.out.println("Created with breed: " + eagle2.breed);
        
        // ========================================
        // 9. EXAMINE CONSTRUCTORS
        // ========================================
        System.out.println("\n--- All Constructors ---");
        Constructor<?>[] constructors = eagleClass.getDeclaredConstructors();
        for (Constructor<?> constructor : constructors) {
            System.out.println("Parameters: " + 
                constructor.getParameterCount());
        }
    }
}
```

---

## Summary Table

### Reflection Operations

| Operation | Public Members | Private Members | Method |
|-----------|----------------|-----------------|--------|
| **Get Class** | - | - | `Class.forName()`, `.class`, `getClass()` |
| **Get Fields** | ✓ | ✗ | `getFields()` |
| **Get All Fields** | ✓ | ✓ | `getDeclaredFields()` |
| **Get Methods** | ✓ | ✗ | `getMethods()` |
| **Get All Methods** | ✓ | ✓ | `getDeclaredMethods()` |
| **Set Public Field** | ✓ | ✗ | `field.set(obj, value)` |
| **Set Private Field** | ✓ | ✓ | `setAccessible(true)` + `set()` |
| **Invoke Public Method** | ✓ | ✗ | `method.invoke(obj, args)` |
| **Invoke Private Method** | ✓ | ✓ | `setAccessible(true)` + `invoke()` |
| **Use Private Constructor** | ✓ | ✓ | `setAccessible(true)` + `newInstance()` |

---

## Key Takeaways

### ✅ Reflection Capabilities

1. **Examine** classes, methods, fields at runtime
2. **Invoke** methods dynamically
3. **Modify** field values (even private)
4. **Access** private members (break encapsulation)
5. **Create** objects from private constructors
6. **Break** Singleton pattern

### ❌ Why Avoid Reflection

1. **Breaks Encapsulation** - Access private members
2. **Slower Performance** - Runtime resolution
3. **More Complex Code** - Harder to maintain
4. **No Type Safety** - Errors at runtime, not compile-time

### ⚠️ When to Use

**Only use when absolutely necessary:**
- Building frameworks
- Dependency injection
- Plugin systems
- Testing private methods
- Annotation processing

### 🎯 Best Practice

> "If you can access it directly, don't use reflection!"

---

## Practice Exercises

### Exercise 1: Basic Reflection
```java
// Create a Person class with private name field
// Use reflection to:
// 1. Get the field
// 2. Change its value
// 3. Print the new value
```

### Exercise 2: Method Invocation
```java
// Create Calculator class with private add(int, int) method
// Use reflection to invoke it
```

### Exercise 3: Singleton Breaking
```java
// Create Singleton class
// Try to create two instances using reflection
// Then add protection against reflection
```

---

**Remember: Reflection is powerful but use it wisely! 🚀**
