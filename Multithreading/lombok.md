# Lombok Essentials: Eliminating Java Boilerplate with Annotations

## Executive Summary
This lesson introduces **Lombok**, a widely-used Java library that eliminates repetitive boilerplate code (getters, setters, constructors, `toString()`, etc.) through compile-time annotation processing. It walks through setup (Maven dependency + IDE plugin) and covers the ten most industry-relevant Lombok annotations, each with its generated-code behavior explained via decompiled `.class` output.

## Core Concepts

### Lombok (Overview & Setup)
**The Layman's Definition:** **Lombok** is a Java library that lets you write annotations instead of repetitive "plumbing" code (getters, setters, constructors, etc.); during compilation, it automatically injects the actual Java code for you.

**How it Works / The Logic:**
- Lombok processes annotations **during compilation** and injects generated code directly into the compiled `.class` files — the source `.java` file stays clean and short.
- Compatible with Java 6 and all later versions.
- **Setup requires two steps:**
  1. Add the Lombok dependency to your `pom.xml`.
  2. (Recommended, not mandatory) Install the **Lombok IDE plugin** and enable **annotation processing** in your IDE's build settings — otherwise the IDE will show false "compilation error" red lines, even though the code compiles and runs fine, because the IDE doesn't understand what Lombok will generate at compile time.

**Example:** Using Lombok's `val` keyword in source code makes IntelliJ show a red squiggly error since it doesn't recognize `val` as valid Java syntax — but the code compiles and runs correctly because Lombok rewrites it into proper typed code (e.g., `String`) behind the scenes during compilation.

---

### `val` and `var`
**The Layman's Definition:** Shortcuts that let you declare a **local variable** without explicitly writing its type — Lombok infers the type from whatever value you assign.

**How it Works / The Logic:**
- Type is determined from the **initializer expression** (the value assigned).
- Works **only for local variables** inside a method/block — not for class fields or method parameters.
- **`val`** marks the variable as effectively **final/immutable** (cannot be reassigned).
- **`var`** behaves the same but does **not** make the variable final (it can be reassigned).

| Keyword | Type Inference | Reassignable? |
|---|---|---|
| `val` | Yes, from initializer | No (immutable/final) |
| `var` | Yes, from initializer | Yes |

**Example:**
```java
val a = 10;
a = 20; // Compile error: cannot assign a value to final variable 'a'

var b = 10;
b = 20; // Works fine — var is not final
```

---

### `@NonNull`
**The Layman's Definition:** An annotation that automatically inserts a **null check** on a method or constructor parameter, throwing an exception if a `null` is passed in.

**How it Works / The Logic:** Applied to a parameter of a method or constructor. Lombok injects an `if (param == null) throw new NullPointerException(...)` check at the very start of the method body.

**Example:**
```java
// Source code
public void greet(@NonNull String name) {
    System.out.println("Hello " + name);
}

// Generated (decompiled) behavior
public void greet(String name) {
    if (name == null) {
        throw new NullPointerException("name is marked non-null but is null");
    }
    System.out.println("Hello " + name);
}
```

---

### `@Getter` and `@Setter`
**The Layman's Definition:** Annotations that auto-generate standard **getter and setter methods** for class fields, removing the need to hand-write them.

**How it Works / The Logic:**
- Can be applied **per-field** or at the **class level** (applies to all applicable fields).
- Default generated methods are **public**, but access level can be overridden (`private`, `protected`, `public`).
- At the class level:
  - `@Getter` applies to all **non-static** fields.
  - `@Setter` applies to all **non-static AND non-final** fields (final fields can't be reassigned, so no setter makes sense).
  - Use `@Setter(AccessLevel.NONE)` (or similarly for `@Getter`) on an individual field to **exclude** it from the class-level generation.

**Example:**
```java
@Getter
@Setter
public class Employee {
    private String name;
    private boolean committeeMember;
    private static String company; // skipped — static fields excluded
}
// Generates: getName(), setName(), isCommitteeMember(), setCommitteeMember()
// No getter/setter generated for 'company' (static)
```

**Example — custom access levels & exclusion:**
```java
public class Employee {
    @Getter(AccessLevel.PRIVATE)
    @Setter(AccessLevel.PROTECTED)
    private String name;

    @Getter
    private boolean committeeMember; // public getter, no setter override needed
}
```

---

### `@ToString`
**The Layman's Definition:** Auto-generates a `toString()` method — commonly used for logging and debugging — that prints the class name and field values.

**How it Works / The Logic:**
- Default output format: `ClassName(field1=value1, field2=value2, ...)`.
- **Customization options:**
  - `@ToString.Exclude` on a field — omit that field from the output.
  - `@ToString(includeFieldNames = false)` — print only values, not field names (reduces log size).
  - `@ToString(onlyExplicitlyIncluded = true)` combined with `@ToString.Include` on specific fields — only explicitly marked fields appear.

**Example:**
```java
@ToString
public class Employee {
    private String name;
    @ToString.Exclude
    private boolean committeeMember;
}
// toString() output: Employee(name=John)  — committeeMember is excluded
```

---

### `@NoArgsConstructor`, `@AllArgsConstructor`, `@RequiredArgsConstructor`
**The Layman's Definition:** Three annotations that generate different flavors of constructors automatically, based on which fields they include.

**How it Works / The Logic:**

| Annotation | Generated Constructor Includes |
|---|---|
| `@NoArgsConstructor` | No parameters — empty constructor |
| `@AllArgsConstructor` | **All** fields as parameters |
| `@RequiredArgsConstructor` | Only **final** fields and fields marked `@NonNull` |

- When a `@NonNull` field is included in a generated constructor, Lombok also inserts the null-check logic for it automatically.

**Example:**
```java
@NoArgsConstructor
@AllArgsConstructor
@RequiredArgsConstructor
public class Employee {
    private String name;
    private boolean committeeMember;
    @NonNull
    private Integer age;
}
// Generates:
// Employee() {}
// Employee(String name, boolean committeeMember, Integer age) { ...null check on age... }
// Employee(Integer age) { ...null check on age... }  — only the @NonNull field
```

---

### `@EqualsAndHashCode`
**The Layman's Definition:** Auto-generates the `equals()` and `hashCode()` methods that follow Java's official contract between the two — no manual implementation needed.

**How it Works / The Logic:**
- By default, uses **all non-static and non-transient** fields for comparison.
- Individual fields can be excluded with `@EqualsAndHashCode.Exclude`.
- The generated code correctly follows the equals/hashCode contract (consistent hashing, symmetric equality, etc.).

**Example:**
```java
@EqualsAndHashCode
public class Employee {
    private String name;
    @EqualsAndHashCode.Exclude
    private boolean committeeMember; // excluded from equals()/hashCode()
    private Integer age;
}
```

---

### `@Data`
**The Layman's Definition:** A convenience "bundle" annotation equivalent to stacking several annotations at once — the all-in-one shortcut for standard POJO boilerplate.

**How it Works / The Logic:** `@Data` is equivalent to combining:
- `@ToString`
- `@EqualsAndHashCode`
- `@Getter` on all fields
- `@Setter` on all non-final fields
- `@RequiredArgsConstructor`

**Example:**
```java
@Data
public class Employee {
    private String name;
    private final int age;
    @NonNull
    private String address;
}
// Generates: toString(), equals(), hashCode(), getName(), getAge(), getAddress(),
// setName(), setAddress() (with null check) — NO setAge() since 'age' is final,
// plus a constructor for (age, address) — the non-null/final required fields
```

---

### `@Value`
**The Layman's Definition:** The **immutable version** of `@Data` — designed specifically to produce fully immutable, thread-safe classes.

**How it Works / The Logic:**
- Makes **all fields private and final**.
- Makes the **class itself final** (cannot be subclassed).
- **No setters** are generated (since all fields are final).
- Still generates `toString()`, `equals()`/`hashCode()`, and getters for all fields.
- Since all fields become final, the generated constructor effectively becomes an **all-args constructor** (equivalent to `@RequiredArgsConstructor` behavior, but now covering every field because every field is final).

**Example:**
```java
@Value
public class Employee {
    String name;
    int age;
    String address;
}
// Generates: final class Employee with all fields private final,
// getName(), getAge(), getAddress(), toString(), equals(), hashCode(),
// and a constructor: Employee(String name, int age, String address)
// No setters at all.
```

---

### `@Builder`
**The Layman's Definition:** Auto-generates a **Builder design pattern** implementation, letting you construct objects step-by-step and ensuring the final object is immutable (no setters).

**How it Works / The Logic:**
- Generates an internal static `Builder` class with a method per field, each returning the builder itself (fluent chaining) until `.build()` is called to produce the final immutable object.
- The target class ends up with **no setter methods**, reinforcing immutability.

**Example:**
```java
@Builder
public class TestPojo {
    private String name;
    private int age;
}

// Usage:
TestPojo obj = TestPojo.builder()
    .name("John")
    .age(30)
    .build();
// obj has no setName()/setAge() — it's immutable once built
```

---

### `@Cleanup`
**The Layman's Definition:** Ensures a resource (like a file stream) is **automatically closed** once execution leaves the current scope — no manual `try-finally` needed.

**How it Works / The Logic:** Lombok wraps the annotated variable's usage in a `try { ... } finally { resource.close(); }` block automatically at compile time.

**Example:**
```java
@Cleanup
FileInputStream inputStream = new FileInputStream("file.txt");
// read from inputStream...
// Lombok automatically generates: try { ... } finally { inputStream.close(); }
```

## Key Takeaways & Quick Reference
- Lombok reduces Java **boilerplate code** by injecting generated code at **compile time** via annotations — the source stays short, but the compiled `.class` file contains the full generated logic.
- Setup requires adding the dependency to `pom.xml`; the IDE plugin + annotation processing setting is optional but prevents false "error" highlighting.
- `val`/`var` infer types for **local variables only**; `val` is effectively final, `var` is not.
- `@Getter`/`@Setter` can target individual fields or an entire class; `@Setter` never applies to final fields.
- `@ToString` and `@EqualsAndHashCode` support fine-grained field inclusion/exclusion.
- Constructor annotations differ by scope: `@NoArgsConstructor` (empty), `@AllArgsConstructor` (every field), `@RequiredArgsConstructor` (final + `@NonNull` fields only).
- `@Data` = shortcut bundle of `toString` + `equals/hashCode` + getters + non-final setters + required-args constructor.
- `@Value` = the **immutable** counterpart of `@Data` — all fields private/final, class itself final, no setters.
- `@Builder` generates a fluent builder pattern for step-by-step, immutable object construction.
- `@Cleanup` auto-generates try-finally resource cleanup, avoiding manual stream/connection closing code.

## Glossary of Terms
- **Boilerplate Code**: Repetitive, standard code (like getters/setters) that must be written in every class but carries no unique business logic.
- **POJO (Plain Old Java Object)**: A simple Java class with private fields and public getters/setters, with no special framework restrictions.
- **Builder Pattern**: A design pattern for constructing complex objects step-by-step while keeping the final object immutable.
- **Annotation Processing**: The compiler mechanism that reads annotations and can generate or modify code during compilation — the technical basis for how Lombok works.
