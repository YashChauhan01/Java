# 🔄 Polymorphic Reference in Java

> Understanding how parent class references can point to child class objects

---

## What is a Polymorphic Reference?

```java
A obj = new B();
```

This statement creates a **polymorphic reference** where:
- **Left side** (`A obj`): The reference variable type is the parent class `A`
- **Right side** (`new B()`): The actual object created is of child class `B`

---

## Key Rules

### 1. Access Restriction

You can **only access members** (methods and fields) that are defined in class `A` through the `obj` reference.

```java
// ✅ Valid - if methodA() exists in class A
obj.methodA();

// ❌ Compilation Error - even if methodB() exists in class B
obj.methodB();
```

### 2. Method Overriding Behavior

If class `B` **overrides** a method from class `A`, calling that method through `obj` will execute the **class B version** (runtime polymorphism).

```java
// If methodXyz() is overridden in class B
obj.methodXyz(); // Executes class B's version
```

---

## Complete Example

```java
// Parent class A
class A {
    public void methodA() {
        System.out.println("Method A from class A");
    }
    
    public void methodXyz() {
        System.out.println("Method Xyz from class A");
    }
}

// Child class B extends A
class B extends A {
    public void methodB() {
        System.out.println("Method B from class B");
    }
    
    @Override
    public void methodXyz() {
        System.out.println("Method Xyz from class B (Overridden)");
    }
}

// Main method
public class Main {
    public static void main(String[] args) {
        A obj = new B(); // Polymorphic reference
        
        // Case 1: Calling method defined in class A
        obj.methodA();
        // Output: Method A from class A
        
        // Case 2: Trying to call method specific to class B
        // obj.methodB(); 
        // ❌ Compilation Error: Cannot find symbol methodB()
        
        // Case 3: Calling overridden method
        obj.methodXyz();
        // Output: Method Xyz from class B (Overridden)
        // ✅ Calls the overridden version from class B
    }
}
```

---

## Summary Table

| Method Call | Result | Reason |
|------------|--------|--------|
| `obj.methodA()` | ✅ Executes class A's `methodA()` | Method exists in class A |
| `obj.methodB()` | ❌ Compilation Error | Method doesn't exist in class A |
| `obj.methodXyz()` | ✅ Executes class B's `methodXyz()` | Method is overridden in class B |

---

## Visual Representation

```
Reference Type: A          Object Type: B
     ↓                          ↓
  A obj          =          new B();
     ↓                          ↓
Can only access            Actual object
methods in A               with B's behavior
```

---

## Key Takeaways

1. **Reference type determines accessibility** - You can only call methods that exist in the reference type (class A)

2. **Object type determines behavior** - For overridden methods, the actual object's version (class B) is executed

3. **Compile-time vs Runtime** - Access checking happens at compile-time, but method selection for overridden methods happens at runtime

4. **Type Casting** - If you need to access class B specific methods, you must cast:
   ```java
   ((B) obj).methodB(); // Now you can access methodB()
   ```

---

## Why Use Polymorphic References?

- **Flexibility**: Write code that works with parent types but can handle any child type
- **Loose Coupling**: Depend on abstractions (parent class/interface) rather than concrete implementations
- **Extensibility**: Add new child classes without changing existing code

```java
// Example: Processing different types of animals
Animal animal1 = new Dog();
Animal animal2 = new Cat();

animal1.makeSound(); // Outputs: Bark
animal2.makeSound(); // Outputs: Meow
```

This is the foundation of **polymorphism** in Java!
