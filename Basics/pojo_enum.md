# Java Classes: POJOs, Enums, and Final - Complete Notes

## 1. POJO (Plain Old Java Object)

### Definition
**POJO** stands for **Plain Old Java Object** - a simple Java class with minimal restrictions.

### Properties of POJO Classes
1. **Public class** - Must be declared as public
2. **Public default constructor** - Should have a public no-argument constructor
3. **Contains variables and getter/setter methods**
4. **No annotations** - No `@Entity`, `@Table`, or other annotations
5. **Does not extend any class**
6. **Does not implement any interface**
7. **Minimal restrictions** - Variables can have any access modifier (public, private, protected, default)

### Example

```java
public class Student {
    int defaultVar;        // default access
    private String name;   // private access
    protected int age;     // protected access
    public String email;   // public access
    
    // Public default constructor (implicit or explicit)
    public Student() {
    }
    
    // Getter and Setter methods
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
    
    public int getAge() {
        return age;
    }
    
    public void setAge(int age) {
        this.age = age;
    }
}
```

### Common Use Cases
1. **Request/Response objects in REST APIs**
   - Controller → Business Logic → Repository → DB
2. **Entity objects** for database tables
3. **Data Transfer Objects (DTOs)**

---

## 2. Enum Classes

### Definition
**Enum** is a special class that represents a collection of constants. All constants in an Enum are implicitly `static` and `final`.

### Key Properties

1. **Cannot extend any class** - Internally extends `java.lang.Enum`
2. **Can implement interfaces** - No restriction on number of interfaces
3. **Can have variables, constructors, and methods**
4. **Cannot be instantiated** - Constructor is always private (even if not specified)
5. **No other class can extend Enum**
6. **Can have abstract methods** - All constants must implement them

### Default Ordinal Values
- Constants are automatically assigned values starting from 0
- Monday = 0, Tuesday = 1, Wednesday = 2, etc.

---

### 2.1 Simple Enum Class

```java
public enum EnumSample {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
}
```

**Internal ordinal values:** Monday=0, Tuesday=1, Wednesday=2, Thursday=3, Friday=4, Saturday=5, Sunday=6

---

### 2.2 Common Enum Methods

#### a) `values()` - Returns array of all enum constants

```java
for (EnumSample sample : EnumSample.values()) {
    System.out.println(sample);
}
// Output: MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
```

#### b) `ordinal()` - Returns the default numbering (0-based index)

```java
for (EnumSample sample : EnumSample.values()) {
    System.out.println(sample.ordinal());
}
// Output: 0, 1, 2, 3, 4, 5, 6
```

#### c) `valueOf(String name)` - Returns enum constant by name

```java
EnumSample day = EnumSample.valueOf("FRIDAY");
System.out.println(day.name());  // Output: FRIDAY
```

#### d) `name()` - Returns the name of the constant

```java
EnumSample day = EnumSample.valueOf("FRIDAY");
System.out.println(day.name());  // Output: FRIDAY
```

---

### 2.3 Enum with Custom Values

```java
public enum EnumSample {
    MONDAY(101, "First day of the week"),
    TUESDAY(102, "Second day"),
    WEDNESDAY(103, "Mid week"),
    THURSDAY(104, "Almost there"),
    FRIDAY(105, "Last working day"),
    SATURDAY(106, "First week off"),
    SUNDAY(107, "Second week off");
    
    // Member variables (one set for each constant)
    private int val;
    private String comment;
    
    // Private constructor
    EnumSample(int val, String comment) {
        this.val = val;
        this.comment = comment;
    }
    
    // Getter methods
    public int getVal() {
        return val;
    }
    
    public String getComment() {
        return comment;
    }
    
    // Static method (class-level, not per constant)
    public static EnumSample getEnumFromValue(int value) {
        for (EnumSample sample : EnumSample.values()) {
            if (sample.val == value) {
                return sample;
            }
        }
        return null;
    }
}

// Usage
EnumSample day = EnumSample.getEnumFromValue(107);
System.out.println(day.getComment());  // Output: Second week off
```

---

### 2.4 Method Override by Constants

```java
public enum EnumSample {
    MONDAY {
        @Override
        public void dummyMethod() {
            System.out.println("Monday dummy method");
        }
    },
    TUESDAY {
        @Override
        public void dummyMethod() {
            System.out.println("Tuesday dummy method");
        }
    },
    WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
    
    // Default method implementation
    public void dummyMethod() {
        System.out.println("Default dummy method");
    }
}

// Usage
EnumSample.FRIDAY.dummyMethod();   // Output: Default dummy method
EnumSample.MONDAY.dummyMethod();   // Output: Monday dummy method
```

---

### 2.5 Enum with Abstract Method

```java
public enum EnumSample {
    MONDAY {
        @Override
        public void dummyMethod() {
            System.out.println("Dummy method in Monday");
        }
    },
    TUESDAY {
        @Override
        public void dummyMethod() {
            System.out.println("Dummy method in Tuesday");
        }
    },
    SUNDAY {
        @Override
        public void dummyMethod() {
            System.out.println("Dummy method in Sunday");
        }
    };
    
    // Abstract method - all constants MUST implement
    public abstract void dummyMethod();
}

// Usage
EnumSample.MONDAY.dummyMethod();  // Output: Dummy method in Monday
```

---

### 2.6 Enum Implementing Interface

```java
interface MyInterface {
    String toLowerCase();
}

public enum EnumSample implements MyInterface {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
    
    @Override
    public String toLowerCase() {
        return this.name().toLowerCase();
    }
}

// Usage
System.out.println(EnumSample.MONDAY.toLowerCase());  // Output: monday
```

---

### 2.7 Advantages of Enum over Static Final Constants

#### Using Static Final Constants (Old Way)
```java
class WeekConstant {
    public static final int MONDAY = 0;
    public static final int TUESDAY = 1;
    public static final int WEDNESDAY = 2;
    public static final int THURSDAY = 3;
    public static final int FRIDAY = 4;
    public static final int SATURDAY = 5;
    public static final int SUNDAY = 6;
}

public boolean isWeekend(int day) {
    if (day == WeekConstant.SATURDAY || day == WeekConstant.SUNDAY) {
        return true;
    }
    return false;
}

// Problems:
isWeekend(2);      // Not very readable
isWeekend(6);      // Works fine
isWeekend(100);    // No compilation error! Can cause unexpected behavior
```

#### Using Enum (Better Way)
```java
public enum EnumSample {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
}

public boolean isWeekend(EnumSample day) {
    if (day == EnumSample.SATURDAY || day == EnumSample.SUNDAY) {
        return true;
    }
    return false;
}

// Benefits:
isWeekend(EnumSample.WEDNESDAY);  // Very readable and self-documenting
isWeekend(EnumSample.SUNDAY);     // Clear intent
isWeekend(1001);                  // COMPILATION ERROR - type safety!
```

### Advantages Summary:
1. **Better Readability** - Self-documenting code
2. **Type Safety** - Only valid enum values can be passed
3. **Controlled Input** - Cannot pass arbitrary values
4. **Compile-time Checking** - Errors caught during compilation

---

## 3. Final Keyword with Classes

### Final Class

A **final class** cannot be inherited (extended) by any other class.

### Purpose
- Prevents inheritance
- Ensures class implementation cannot be modified through inheritance
- Used for security and design integrity

### Example

```java
// Final class declaration
public final class Test {
    public void display() {
        System.out.println("This is a final class");
    }
}

// Attempting to extend final class
public class MyAnotherClass extends Test {  // COMPILATION ERROR
    // Error: Cannot inherit from final 'Test'
}
```

### Use Cases
- Immutable classes (e.g., `String`, `Integer`)
- Security-sensitive classes
- Utility classes that shouldn't be extended

---

## Summary Table

| Feature | POJO | Enum | Final Class |
|---------|------|------|-------------|
| **Purpose** | Simple data holder | Collection of constants | Prevent inheritance |
| **Extends** | Optional (but POJO shouldn't) | Implicitly extends java.lang.Enum | Can extend one class |
| **Implements** | No (for pure POJO) | Yes, multiple interfaces | Yes, multiple interfaces |
| **Instantiation** | Yes, public constructor | No, private constructor | Yes, unless marked otherwise |
| **Inheritance** | Can be extended | Cannot be extended | Cannot be extended |
| **Common Use** | DTOs, Entities | Constants, State machines | Utility classes, Security |

---

## Key Takeaways

1. **POJOs** are simple, restriction-free classes perfect for data transfer
2. **Enums** provide type-safe constant collections with better readability than static final variables
3. **Final classes** prevent inheritance for security and design integrity
4. Enum constants are implicitly `static final`
5. Enum constructors are always private (implicit or explicit)
6. Use Enums over static final constants for better type safety and readability
