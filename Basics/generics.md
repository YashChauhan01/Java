# Java Generic Classes
## Complete Guide

---

## 1. Why Generics? The Problem with Object Class

Every class in Java implicitly extends `Object`. Since `Object` is the parent of all, it can hold any reference — but this causes a **typecasting problem**.

### Without Generics (Object-based approach)

```java
public class Print {
    private Object value;

    public void setPrintValue(Object value) {
        this.value = value;
    }

    public Object getPrintValue() {
        return value;
    }
}
```

**Usage — messy typecasting required:**
```java
Print obj = new Print();
obj.setPrintValue(1);          // accepts Integer ✅
obj.setPrintValue("Hello");    // accepts String too ✅

// To use the value, you MUST typecast
Object val = obj.getPrintValue();

if (val instanceof Integer) {
    int i = (Integer) val;     // typecast required
} else if (val instanceof String) {
    String s = (String) val;   // typecast required
}
```

**Problems:**
- Don't know what type is stored at runtime
- Manual `instanceof` checks needed everywhere
- Runtime `ClassCastException` risk
- No compile-time safety

---

## 2. Generic Class — The Solution

### Syntax: Diamond `<T>` notation

```java
public class Print<T> {      // T = type parameter
    private T value;

    public void setPrintValue(T value) {
        this.value = value;
    }

    public T getPrintValue() {     // returns T, not Object
        return value;
    }
}
```

> `T` can be any capital letter — `T`, `A`, `B`, `K`, `V`, `E` etc. Convention: `T` for Type, `K` for Key, `V` for Value, `E` for Element.

### Usage — no typecasting needed!

```java
// Create a Print for Integer
Print<Integer> intObj = new Print<>();
intObj.setPrintValue(1);          // ✅ Only Integer allowed
// intObj.setPrintValue("Hello"); // ❌ Compile-time error!

Integer val = intObj.getPrintValue();  // No typecast needed ✅

// Create a Print for String
Print<String> strObj = new Print<>();
strObj.setPrintValue("Hello");    // ✅ Only String allowed
String s = strObj.getPrintValue(); // No typecast needed ✅
```

> ⚠️ **T can only be non-primitive types.**
> Use wrapper classes: `Integer` (not `int`), `Double` (not `double`), etc.

---

## 3. Inheritance with Generic Classes

### Case 1: Non-Generic Subclass

When the subclass is NOT generic, you must specify the type at the time of `extends`.

```java
// Parent: generic
public class Print<T> { ... }

// Child: non-generic → must specify type at extends
public class ColorPrint extends Print<String> {
    // ColorPrint is fixed to String only
}
```

```java
ColorPrint cp = new ColorPrint();
cp.setPrintValue("Red");    // ✅ Only String
// cp.setPrintValue(100);   // ❌ Compile-time error
```

### Case 2: Generic Subclass

When the subclass IS also generic, type is specified at object creation time.

```java
// Child: also generic
public class ColorPrint<T> extends Print<T> {
    // passes T through to parent
}
```

```java
ColorPrint<String> cp = new ColorPrint<>();
cp.setPrintValue("Blue");   // ✅

ColorPrint<Integer> cp2 = new ColorPrint<>();
cp2.setPrintValue(42);      // ✅
```

---

## 4. Multiple Type Parameters

A generic class can have **any number** of type parameters.

```java
public class Pair<K, V> {    // K = key, V = value
    private K key;
    private V value;

    public void put(K key, V value) {
        this.key = key;
        this.value = value;
    }

    public K getKey()   { return key; }
    public V getValue() { return value; }
}
```

```java
// K = String, V = Integer
Pair<String, Integer> pair = new Pair<>();
pair.put("Age", 25);

String k = pair.getKey();    // "Age"
Integer v = pair.getValue(); // 25

// Both syntaxes are valid:
Pair<String, Integer> p1 = new Pair<String, Integer>(); // explicit
Pair<String, Integer> p2 = new Pair<>();                 // diamond (preferred)
```

---

## 5. Generic Methods

You can make **only one method** generic without making the whole class generic.

### Syntax

```java
public class GenericMethodClass {            // NOT a generic class

    // Generic method syntax:
    // <TypeParam> ReturnType methodName(TypeParam param)
    public <T> void printValue(T value) {
        System.out.println(value);
    }
}
```

> **Rule**: Type parameter `<T>` must appear **before the return type** in the method declaration.
> **Scope**: The `T` is limited to this method only.

```java
GenericMethodClass obj = new GenericMethodClass();

obj.printValue(new Bus());  // ✅ accepts Bus
obj.printValue(new Car());  // ✅ accepts Car
obj.printValue("Hello");    // ✅ accepts String
obj.printValue(100);        // ✅ accepts Integer
```

### Generic Method vs Generic Class Scope

```
Generic Class:          Generic Method:
┌─────────────────┐    ┌─────────────────────────┐
│ class Print<T>  │    │ class Print {            │
│                 │    │                          │
│  method1(T)  ✅ │    │  <T> void method1(T) ✅ │
│  method2(T)  ✅ │    │                          │
│  field: T    ✅ │    │  method2(T)  ❌ (T not   │
│                 │    │  available here)         │
└─────────────────┘    └─────────────────────────┘
T is class-scoped        T is method-scoped only
```

---

## 6. Raw Type

When you create an object of a generic class **without specifying the type parameter**, it's called a **raw type**.

```java
// Parameterized type (correct way):
Print<String> p1 = new Print<>();

// Raw type (avoid in production!):
Print rawObj = new Print();   // ← no type specified
```

**What happens internally with raw type:**
```java
// You write:
Print rawObj = new Print();

// Compiler internally treats it as:
Print<Object> rawObj = new Print<Object>();
// → accepts anything (back to the original problem!)
```

```java
rawObj.setPrintValue(1);       // ✅ no error (accepts Object)
rawObj.setPrintValue("Hello"); // ✅ no error (accepts Object)
// But you'll need typecasting again — defeats the purpose!
```

> ⚠️ Raw types exist for backward compatibility with pre-Java-5 code. **Avoid raw types in new code.**

---

## 7. Bounded Generics

Restrict which types can be passed as the type parameter.

### 7.1 Upper Bound — `extends`

**Syntax**: `<T extends ClassName>`

Means: T can be `ClassName` **or any of its subclasses**.

```
Object
  └── Number
        ├── Integer  ✅
        ├── Double   ✅
        ├── Float    ✅
        ├── Long     ✅
        └── BigDecimal ✅
  └── String         ❌ (not a child of Number)
```

```java
public class Print<T extends Number> {   // upper bound
    private T value;
    // ...
}
```

```java
Print<Integer> p1 = new Print<>();    // ✅ Integer extends Number
Print<Double>  p2 = new Print<>();    // ✅ Double extends Number
// Print<String> p3 = new Print<>();  // ❌ Compile-time error!
```

> ⚠️ Even for interfaces, use `extends` (not `implements`) in type bounds.

---

### 7.2 Multi-Bound — `extends ClassA & InterfaceB & InterfaceC`

Restrict the type to something that **extends a class AND implements multiple interfaces**.

```java
// Structure required:
class A extends ParentClass implements Interface1, Interface2 {}

// Multi-bound type parameter:
public class Print<T extends ParentClass & Interface1 & Interface2> {
    // T must be ParentClass (or child) AND implement both interfaces
}
```

**Rules:**
- First bound must be a **class** (or can be omitted)
- All subsequent bounds must be **interfaces**
- Use `&` to separate multiple bounds

```java
// Class A: extends ParentClass + implements both interfaces ✅
class A extends ParentClass implements Interface1, Interface2 {}
Print<A> p1 = new Print<>();   // ✅ Works

// Class B: extends ParentClass + implements only one interface ❌
class B extends ParentClass implements Interface1 {}
// Print<B> p2 = new Print<>();  // ❌ Compile-time error! Missing Interface2
```

---

## 8. Wildcards (`?`)

Wildcards solve a key limitation: **`List<Vehicle>` is NOT a parent of `List<Bus>`**, even though `Vehicle` is a parent of `Bus`.

```java
Vehicle vehicleObj = new Bus();    // ✅ valid (polymorphism)

List<Vehicle> vehicleList = new ArrayList<>();
List<Bus> busList = new ArrayList<>();

// vehicleList = busList;          // ❌ INVALID! Not polymorphic for generics
// busList = vehicleList;          // ❌ INVALID!
```

**Why?**
```java
// If it were allowed:
List<Vehicle> vl = busList;   // hypothetically...
vl.add(new Car());            // Car is a Vehicle → allowed on List<Vehicle>
// But busList now contains a Car! That breaks type safety!
```

### 8.1 Upper Bound Wildcard — `? extends ClassName`

Accepts `ClassName` **and all its subclasses**.

```java
// WITHOUT wildcard — only List<Vehicle> accepted
public void setValues(List<Vehicle> list) { ... }

// WITH upper bound wildcard — List<Vehicle>, List<Bus>, List<Car> all accepted
public void setValues(List<? extends Vehicle> list) { ... }
```

```java
List<Vehicle> vl = new ArrayList<>();
List<Bus>     bl = new ArrayList<>();
List<Car>     cl = new ArrayList<>();

obj.setValues(vl);   // ✅
obj.setValues(bl);   // ✅ (Bus extends Vehicle)
obj.setValues(cl);   // ✅ (Car extends Vehicle)
```

### 8.2 Lower Bound Wildcard — `? super ClassName`

Accepts `ClassName` **and all its superclasses**.

```java
public void setValues(List<? super Vehicle> list) { ... }
```

```java
List<Vehicle> vl = new ArrayList<>();
List<Object>  ol = new ArrayList<>();   // Object is parent of Vehicle
List<Bus>     bl = new ArrayList<>();   // Bus is child of Vehicle

obj.setValues(vl);   // ✅ (Vehicle itself)
obj.setValues(ol);   // ✅ (Object is above Vehicle)
// obj.setValues(bl); // ❌ Bus is below Vehicle, not above
```

### 8.3 Unbounded Wildcard — `?`

Accepts **any type**. Use when your method only uses `Object` class methods.

```java
public void printList(List<?> list) {
    for (Object obj : list) {     // only Object-level operations
        System.out.println(obj);  // .toString() from Object
    }
}
```

```java
printList(new ArrayList<Integer>());  // ✅
printList(new ArrayList<String>());   // ✅
printList(new ArrayList<Bus>());      // ✅
```

---

## 9. Wildcard vs Generic Type Method — When to Use Which?

```java
// Wildcard method
public void compute(List<? extends Number> src, List<? extends Number> dst) { }

// Generic type method
public <T extends Number> void compute(List<T> src, List<T> dst) { }
```

| Feature | Wildcard `?` | Generic Type `<T>` |
|---------|-------------|-------------------|
| Multiple parameter types | ✅ `src=List<Integer>`, `dst=List<Float>` | ❌ Both must be same type T |
| Lower bound (`super`) | ✅ `? super Vehicle` | ❌ Not supported |
| Multiple type params | ❌ Only one `?` | ✅ `<K, V, T>` |
| Type enforcement across params | ❌ No restriction | ✅ All params share same T |
| Best for | Flexible, mixed types | Strict, same-type enforcement |

```java
// Wildcard: accepts different types for src and dst
obj.compute(integerList, floatList);   // ✅ both are Number children

// Generic: BOTH must be same type T
obj.<Integer>compute(integerList, floatList);  // ❌ Compile error!
obj.<Integer>compute(integerList, integerList); // ✅ Same type
```

---

## 10. Type Erasure

At **compile time**, all generic type information is **erased** and replaced with actual types in the bytecode.

```
What you write              What bytecode sees
──────────────             ──────────────────
```

**Unbounded generic:**
```java
// You write:
class Print<T> {
    T value;
    T getValue() { return value; }
}

// Bytecode (after erasure):
class Print {
    Object value;                // T → Object
    Object getValue() { return value; }
}
```

**Bounded generic:**
```java
// You write:
class Print<T extends Number> {
    T value;
}

// Bytecode (after erasure):
class Print {
    Number value;               // T → Number (the upper bound)
}
```

**Generic method:**
```java
// You write:
public <T> void print(T val) { }

// Bytecode:
public void print(Object val) { }  // T → Object (unbounded)

// You write:
public <T extends Bus> void print(T val) { }

// Bytecode:
public void print(Bus val) { }     // T → Bus (the bound)
```

> **Key Insight**: Generics are a **compile-time feature** only. The JVM has no knowledge of generic types at runtime — they're purely for type safety during development.

---

## 11. Complete Summary

```
GENERICS IN JAVA
│
├── Generic Class          class Pair<K, V> { }
│
├── Generic Method         public <T> void print(T val) { }
│
├── Inheritance
│   ├── Non-generic child  class Child extends Parent<String> { }
│   └── Generic child      class Child<T> extends Parent<T> { }
│
├── Bounded Generics
│   ├── Upper bound        <T extends Number>
│   └── Multi bound        <T extends ClassA & InterfaceB & InterfaceC>
│
├── Wildcards
│   ├── Upper bound        List<? extends Vehicle>  (Vehicle and children)
│   ├── Lower bound        List<? super Vehicle>    (Vehicle and parents)
│   └── Unbounded          List<?>                  (any type)
│
├── Raw Type               Print raw = new Print();  ← avoid!
│
└── Type Erasure           All T's replaced at compile time
```

---

## Quick Interview Reference

| Concept | Key Point |
|---------|-----------|
| Why generics? | Avoid typecasting, compile-time type safety |
| T restriction | Cannot use primitive types (`int`, `double`) — use wrappers |
| Raw type | Generic class without type param — compiler uses `Object` |
| Upper bound | `extends` — class + all its subclasses |
| Multi bound | First = class, rest = interfaces, joined with `&` |
| `List<Vehicle>` vs `List<Bus>` | NOT related, even if `Vehicle` is parent of `Bus` |
| Wildcard `?` | Flexible but no type enforcement across params |
| Generic `<T>` method | Strict — all params sharing T must be same type |
| Lower bound | Only possible with wildcards (`super`), not generic type params |
| Type erasure | Generics exist only at compile time; bytecode has no generics |
