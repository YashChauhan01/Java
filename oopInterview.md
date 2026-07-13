# UBS OOP Interview — 50 Complete Answers

**Prep target: July 17 technical interview**
Format note: each answer is written the way you should *say* it out loud — banking-anchored, precise, and short enough to deliver in 60–90 seconds unless a follow-up probes deeper.

---

## Category 1: The 4 Pillars & Core Foundations

### 1. Explain the 4 pillars of OOP using a banking example
- **Encapsulation** — an `Account` class keeps `balance` private and exposes it only through `deposit()`/`withdraw()` methods that enforce business rules (no negative balance).
- **Abstraction** — a `PaymentGateway` interface exposes `processPayment()` without revealing whether it's routed through Visa, SWIFT, or an internal ledger.
- **Inheritance** — `SavingsAccount` and `CurrentAccount` both extend a base `Account` class, reusing common fields like `accountNumber` and `owner`.
- **Polymorphism** — calling `calculateInterest()` on a list of `Account` references executes different logic depending on whether the actual object is a `SavingsAccount` or `FixedDepositAccount`.

### 2. What is Encapsulation, and how do getters/setters enforce data hiding and validation?
Encapsulation bundles data and the methods that operate on it into one unit, and restricts direct access to internal state. In an `Account` class, `balance` is `private`; the only way to change it is through a `setBalance()` (or better, `deposit()`/`withdraw()`) method that validates the input — e.g., rejecting a withdrawal that would push the balance negative. Without encapsulation, any part of the codebase could set `balance = -500` directly, which is unacceptable in a financial system.

### 3. Exact difference between Abstraction and Encapsulation
- **Abstraction** is a *design-level* concept — it hides **complexity** by exposing only "what" an object does (via interfaces/abstract classes), not "how."
- **Encapsulation** is an *implementation-level* mechanism — it hides **data** using access modifiers (`private`, `protected`).
One-liner interviewers like: "Abstraction hides implementation, encapsulation hides data." Abstraction is achieved via interfaces/abstract classes; encapsulation is achieved via access modifiers + getters/setters.

### 4. Compile-time vs runtime polymorphism
- **Compile-time (static) polymorphism** = method **overloading**. The compiler resolves which method to call based on method signature at compile time (e.g., `transfer(Account a)` vs `transfer(Account a, String note)`).
- **Runtime (dynamic) polymorphism** = method **overriding**. The JVM resolves the actual method to invoke at runtime based on the object's real type, via a mechanism called dynamic method dispatch (using the vtable). E.g., `Account acc = new SavingsAccount(); acc.calculateInterest();` calls `SavingsAccount`'s version even though the reference type is `Account`.

### 5. What is Inheritance? Why doesn't Java support multiple inheritance via classes, and how does that avoid the Diamond Problem?
Inheritance lets a class acquire fields/methods of another class (`extends`), enabling code reuse and establishing an IS-A relationship. Java disallows multiple class inheritance because if class `C extends A, B` and both `A` and `B` define a method `foo()`, the compiler can't unambiguously decide which `foo()` `C` inherits — this is the **Diamond Problem**. Java sidesteps it by allowing only single class inheritance, while permitting multiple **interface** implementation (where ambiguity from default methods must be explicitly resolved by the implementing class — see Q12).

### 6. IS-A (Inheritance) vs HAS-A (Composition/Aggregation)
- **IS-A**: `SavingsAccount` IS-A `Account` → modeled with `extends`.
- **HAS-A**: `Account` HAS-A `Customer` → modeled by holding a reference to a `Customer` object as a field.
IS-A implies substitutability (an `Account` reference can point to a `SavingsAccount`); HAS-A implies ownership/collaboration without that substitutability. Composition (strong ownership, e.g., `Engine` inside `Car`, lifecycle-bound) is distinguished from aggregation (weaker ownership, e.g., `Department` has `Employees` who can exist independently).

### 7. Why "favor composition over inheritance"?
Inheritance creates **tight coupling** — a change in the parent class can silently break subclasses (the "fragile base class" problem), and it locks you into a single fixed hierarchy decided at compile time. Composition lets you assemble behavior from independent, swappable objects at runtime — e.g., instead of `FraudCheckingAccount extends Account`, you inject a `FraudDetector` strategy into `Account`. This is more flexible, more testable (you can mock the injected component), and avoids deep, brittle hierarchies — which is exactly why Spring's Dependency Injection model is built around composition, not inheritance.

### 8. Object cloning — deep copy vs shallow copy
Cloning creates a duplicate object, typically via `Object.clone()` after implementing the `Cloneable` marker interface.
- **Shallow copy**: copies primitive fields directly but copies **references** for object fields — so the clone and original share the same nested objects. Mutating a nested object in the clone affects the original.
- **Deep copy**: recursively clones nested objects too, so the clone is fully independent.
In a banking context: shallow-cloning a `Transaction` object that holds a `List<LineItem>` means both the original and clone share the same list — a dangerous bug if one is later modified. Deep copy avoids that by cloning the list and its contents too.

### 9. What is a constructor? Rules around default vs parameterized constructors
A constructor initializes an object's state at creation time; it has the same name as the class and no return type. Key rules:
- If you define **no constructor**, Java auto-generates a no-arg default constructor.
- The moment you define **any** parameterized constructor, Java stops auto-generating the default no-arg one — if you still need a no-arg constructor, you must write it explicitly.
- Constructors can be overloaded (multiple constructors, different parameter lists).
- Constructors cannot be `final`, `static`, or `abstract`.

### 10. Can you override private, static, or final methods?
- **`private`**: No — private methods aren't visible to subclasses at all, so there's nothing to override. If you define a same-signature method in the subclass, it's a completely new method, not an override.
- **`static`**: No — static methods belong to the class, not the instance, and are resolved at compile time based on reference type. A same-signature static method in a subclass **hides** the parent's version (method hiding), it doesn't override it.
- **`final`**: No — `final` explicitly forbids overriding; that's its purpose. Attempting to override a final method is a compile-time error.

### 11. Can `main()` be overloaded? How does the JVM execute it?
Yes, `main()` can be overloaded like any other method — you can have `main(String[] args)`, `main(int x)`, `main(String[] args, int y)`, etc., all in the same class. However, the **JVM only ever invokes** the exact signature `public static void main(String[] args)` as the program entry point. Any overloaded versions are just regular methods that must be called explicitly from within the code (typically from the real `main`) — the JVM will never call them automatically.

### 12. How does Java resolve the Diamond Problem for default methods in interfaces?
If a class implements two interfaces that both declare the same `default` method, Java doesn't pick one automatically — it forces the implementing class to **explicitly override** that method and resolve the conflict, optionally calling a specific interface's version using `InterfaceName.super.methodName()`. This keeps the resolution explicit and unambiguous rather than relying on implicit precedence rules, which is exactly what Java avoided with multiple class inheritance.

---

## Category 2: Advanced Language Syntax & Keyword Traps

### 13. Significance of `super`; `super()` vs `super.variable`
`super` refers to the immediate parent class. `super()` (a constructor call) invokes the parent class's constructor and **must be the first statement** in the child constructor — used to ensure the parent's state is initialized before the child adds its own. `super.variable` or `super.method()` accesses a specific field/method from the parent class, typically used when the child has a field/method with the same name and you need to disambiguate (e.g., calling the overridden parent version inside an override).

### 14. `this()` vs `super()` — can both be in the same constructor?
`this()` calls another constructor **in the same class** (constructor chaining); `super()` calls a constructor **in the parent class**. **No, both cannot appear in the same constructor** — Java only allows one explicit constructor-call statement, and it must be the first line. This is because both are trying to occupy the same "first statement" slot responsible for guaranteeing proper initialization order; allowing both would create ambiguity about which chain runs first.

### 15. What happens in memory if an exception is thrown inside a constructor during instantiation?
Memory is allocated on the heap for the object before the constructor body runs (the reference is not yet returned to the caller). If an exception is thrown mid-constructor, object construction is aborted, the reference is never returned to the caller, and the partially-constructed object becomes **eligible for garbage collection** since no live reference exists. The caller's variable is never assigned — it's as if the object never came into existence from the program's point of view, though the memory briefly existed on the heap until GC reclaims it.

### 16. `final` on a variable, method, class, and object reference
- **Variable**: value can be assigned once; cannot be reassigned after initialization.
- **Method**: cannot be overridden by subclasses.
- **Class**: cannot be extended/subclassed (e.g., `String`, `Integer`).
- **Object reference**: the reference itself can't be reassigned to point to a different object, but the object's **internal state can still change** if it's mutable — e.g., `final List<String> list` prevents reassigning `list`, but you can still `list.add(...)`.

### 17. Rules to create a custom immutable class (like `String`)
1. Declare the class as `final` (prevents subclassing that could break immutability).
2. Make all fields `private final`.
3. Provide no setters.
4. Initialize all fields via constructor only.
5. For mutable field types (e.g., `Date`, `List`), never expose the actual reference — return a **defensive copy** in the getter, and accept a defensive copy in the constructor too, so external code can't mutate internal state through a shared reference.

```java
public final class TransactionRecord {
    private final String id;
    private final List<String> tags;

    public TransactionRecord(String id, List<String> tags) {
        this.id = id;
        this.tags = new ArrayList<>(tags); // defensive copy in
    }

    public List<String> getTags() {
        return new ArrayList<>(tags); // defensive copy out
    }
}
```

### 18. Why is `String` immutable, and how does the String Pool save memory?
`String` is immutable so it can be safely shared, cached, and used as a key in hash-based collections without risk of its hashcode changing after insertion, and so it's inherently thread-safe. The **String Pool** is a special memory region (part of the heap, in Metaspace-adjacent area since Java 7) where string literals are stored; when you write `String a = "UBS";`, the JVM checks the pool first — if `"UBS"` already exists, `a` just points to the existing object instead of allocating a new one. In a financial app processing millions of repeated string values (currency codes, account types), this dramatically reduces memory overhead. Note: `new String("UBS")` bypasses the pool and forces a new heap allocation.

### 19. Method hiding vs method overriding (static methods)
**Overriding** applies to instance methods and is resolved at runtime via dynamic dispatch based on the actual object type. **Method hiding** applies to `static` methods — if a subclass declares a static method with the same signature as the parent's static method, it doesn't override it; it **hides** it. Which version runs is determined by the **reference type at compile time**, not the object's actual type. E.g., `Account.staticMethod()` vs `SavingsAccount.staticMethod()` called via an `Account`-typed reference will always call `Account`'s version, even if the object is actually a `SavingsAccount`.

### 20. Covariant return types
When overriding a method, Java allows the overriding method in the subclass to return a **subtype** of the return type declared in the parent method, instead of the exact same type. E.g., if `Account.copy()` returns `Account`, `SavingsAccount.copy()` can override it to return `SavingsAccount` directly — callers get a more specific type without needing to cast, while still satisfying the override contract.

### 21. Can an abstract class have a constructor? When is it invoked?
Yes. Even though an abstract class can't be instantiated directly with `new`, its constructor exists to initialize the common state that subclasses rely on. It's invoked implicitly (or explicitly via `super()`) whenever a **concrete subclass is instantiated** — the subclass constructor always calls the abstract parent's constructor first as part of the constructor chaining process, ensuring shared fields (e.g., `accountNumber` in an abstract `Account`) are properly set up.

### 22. Can interfaces declare variables? What modifiers are implicit?
Yes, but any variable declared in an interface is implicitly `public static final` — i.e., a compile-time constant, not instance state. You cannot declare a plain mutable field in an interface. This is why interfaces are used for behavior contracts and shared constants (e.g., `MAX_TRANSFER_LIMIT`), not for holding per-object data.

### 23. Inner class vs static nested class
- **Inner (non-static) class**: tied to an instance of the outer class; holds an implicit reference to that outer instance, so it can access the outer object's instance members directly. Created via `outerInstance.new InnerClass()`.
- **Static nested class**: does not hold a reference to any outer instance; behaves like a top-level class that's just namespaced inside another class for organization. Created via `new Outer.StaticNested()`.
Memory-wise, every inner class instance carries a hidden reference to its enclosing instance (which can also prevent that outer object from being garbage collected while the inner instance is alive) — static nested classes don't have this overhead.

### 24. Anonymous inner class — what and where used
An anonymous inner class is a class defined and instantiated in a single expression, without a name, typically used to provide a one-off implementation of an interface or abstract class. Common in event handling and concurrency, e.g., implementing `Runnable` inline for a thread task, or providing a custom `Comparator` for sorting a list of transactions by amount without creating a separate named class. Largely superseded by lambdas for functional interfaces since Java 8, but still used when you need to override multiple methods or maintain state.

### 25. Variable shadowing vs variable hiding
- **Shadowing**: a local variable (e.g., a constructor/method parameter) has the same name as an instance field, temporarily "shadowing" it within that scope — resolved using `this.fieldName` to access the field explicitly.
- **Hiding**: a subclass declares a field with the same name as a field in the parent class. Unlike method overriding, field access is resolved at **compile time based on reference type**, not the runtime object — so `Account acc = new SavingsAccount(); acc.rate` accesses `Account`'s `rate` field even if `SavingsAccount` redeclares `rate`, because fields don't participate in dynamic dispatch.

---

## Category 3: Memory Architecture, Concurrency & Enterprise Java

### 26. Abstract class vs Interface (Java 8+) — when to choose which
| | Abstract Class | Interface |
|---|---|---|
| State | Can hold instance fields | Only `public static final` constants |
| Constructors | Yes | No |
| Method bodies | Yes | Yes, via `default`/`static` (Java 8+) |
| Multiple inheritance | No (single class extends) | Yes (implements multiple) |
| Access modifiers | Any | Implicitly `public` |

Choose an **abstract class** when subclasses share common state and implementation (e.g., a base `Account` with a `balance` field and common validation logic). Choose an **interface** when you're defining a **contract/capability** that unrelated classes might implement (e.g., `Auditable`, `Transferable`) — especially when a class needs to satisfy multiple such contracts simultaneously.

### 27. Why were `default` and `static` methods added to interfaces in Java 8?
To allow interfaces to **evolve without breaking existing implementations**. Before Java 8, adding a new method to an interface would break every class that implemented it (compile error, since they wouldn't have implemented the new method). `default` methods let you add new behavior to an interface with a built-in implementation, so existing implementing classes keep compiling and working unchanged. This was largely driven by the need to add methods like `forEach()` to the `Collection` interface for the new Streams API without breaking the entire Java ecosystem.

### 28. What is a Functional Interface? Three examples
An interface with **exactly one abstract method** (it can have any number of `default`/`static` methods), enabling it to be implemented via a lambda expression. Often annotated with `@FunctionalInterface` for compile-time safety. Standard examples:
- `Runnable` — `void run()`, no args, no return.
- `Callable<V>` — `V call()`, can return a value and throw checked exceptions.
- `Comparator<T>` — `int compare(T o1, T o2)`, used for custom sorting logic.

### 29. How does the JVM handle object creation in memory?
When you call `new Account()`: the JVM allocates memory on the **heap** for the object's instance variables, initializes them to default values, then runs the constructor to set actual values. The **reference** to that heap object (e.g., variable `acc`) is stored on the **stack** if it's a local variable, pointing to the heap location. **Local (primitive) variables** live entirely on the stack within their method's stack frame. **Instance variables** always live on the heap, as part of the object. Class metadata (method bytecode, static variables) lives in **Metaspace**.

### 30. Heap (shared) vs Stack (thread-private) — impact on thread safety
Every thread gets its own **stack**, so local variables and method call frames are inherently thread-safe — no thread can see another thread's stack. The **heap** is shared across all threads in the JVM, so any object referenced by multiple threads (like a shared `Account` instance) is a potential race-condition site if multiple threads read/write its state concurrently without synchronization. This is exactly why a shared `BankAccount.balance` field needs synchronization (locks, `synchronized`, `AtomicLong`, etc.) while a purely local calculation inside one thread's method never does.

### 31. Why did Java replace PermGen with Metaspace?
PermGen (Permanent Generation) had a **fixed maximum size** set at JVM startup, and it lived within the main heap — this frequently caused `OutOfMemoryError: PermGen space` in applications that dynamically loaded many classes (e.g., app servers redeploying WARs repeatedly), since old class metadata couldn't always be garbage collected promptly and the space couldn't grow. **Metaspace** (introduced in Java 8) moved class metadata to **native (off-heap) memory**, which by default grows dynamically up to the limits of available system memory, largely eliminating that class of OOM errors. Class metadata and compiled bytecode for loaded classes are stored in Metaspace; actual objects still live on the heap.

### 32. Garbage Collection and object eligibility
The GC's job is to reclaim heap memory occupied by objects that are no longer reachable from any live thread's roots (stack variables, static references, active thread objects). An object becomes **eligible for GC** when there are zero reachable references to it — e.g., a local `Transaction` object goes out of scope after its method returns, or you explicitly set `account = null`. The JVM (typically using a generational collector) doesn't guarantee *when* collection happens — only that unreachable objects are eventually reclaimed. Circular references between two otherwise-unreachable objects are still collected, since Java's GC works on reachability from GC roots, not simple reference counting.

### 33. Contract between `equals()` and `hashCode()` — why override together
The contract: **if two objects are equal according to `equals()`, they must return the same `hashCode()`.** (The reverse isn't required — unequal objects *can* share a hash code, called a collision.) If you override `equals()` without overriding `hashCode()`, you break this contract — two "equal" `Account` objects could land in different hash buckets in a `HashMap`/`HashSet`, causing lookups to silently fail (you insert an object, then can't find it via an equal key). This is a classic, subtle bug in production systems using accounts or transactions as map keys or in sets for deduplication.

### 34. `==` vs `.equals()`
- `==` compares **reference identity** for objects — whether two variables point to the exact same memory location (for primitives, it compares actual values).
- `.equals()` compares **logical/value equality**, as defined by the class's own implementation. By default (from `Object`), `.equals()` behaves like `==` unless overridden.
Example: two separate `new Account("123")` objects with the same account number will be `false` under `==` (different objects in memory) but should be `true` under a properly overridden `.equals()` that compares `accountNumber`.

### 35. `instanceof` and pattern matching
`instanceof` checks whether an object is an instance of a given type at runtime, returning a boolean — commonly used before an explicit downcast to avoid a `ClassCastException`. Since Java 16, **pattern matching for `instanceof`** lets you combine the check and the cast in one step: instead of `if (acc instanceof SavingsAccount) { SavingsAccount sa = (SavingsAccount) acc; ... }`, you write `if (acc instanceof SavingsAccount sa) { ... }` and `sa` is automatically available and correctly typed inside that block — removing boilerplate and a common source of casting bugs.

### 36. How does Spring Boot use OOP for Dependency Injection?
Spring relies heavily on **polymorphism and interfaces**: you typically depend on an interface type (e.g., `PaymentService`) rather than a concrete class, and Spring's IoC container decides at runtime which concrete implementation (`StripePaymentService`, `RazorpayPaymentService`) to **inject** into that reference — this is polymorphism in action, resolved via configuration/annotations instead of `new`. This is also **composition over inheritance**: instead of subclassing to change behavior, you swap out injected collaborator objects. `@Autowired` fields/constructors are populated by Spring reflecting on interface types and matching them to registered bean implementations.

### 37. Inversion of Control (IoC)
Traditionally, an object is responsible for creating and managing its own dependencies (`this.paymentService = new StripePaymentService();`) — the object controls its own object graph. **IoC flips this**: the framework (Spring's container) takes over creating and wiring dependencies, and *injects* them into your class from the outside, typically via constructor or field injection. Your class simply declares what it needs (via an interface type), and control over "which implementation, when, and how it's constructed" is inverted — moved out of your class and into the container. This decouples classes from concrete instantiation logic, making them easier to test (you can inject mocks) and reconfigure without code changes.

### 38. Can two JVM instances share/modify a public static variable?
No. A `static` variable belongs to a class **within a single JVM instance's memory space** — it is not shared across separate JVM processes, even if they're running the exact same application code on the same physical server. Each JVM has its own isolated heap and Metaspace. If two JVM instances (e.g., two instances of a microservice) both have a `static int requestCount`, incrementing it in one process has zero effect on the other's value — they are entirely independent copies. This is exactly why distributed systems can't rely on in-memory static state for shared counters/locks across service instances — you need external shared state (Redis, a database, or a distributed lock) instead.

---

## Category 4: Design Patterns, SOLID Principles & Applied LLD

### 39. Checked vs Unchecked exceptions — when to design a custom one
- **Checked exceptions** (extend `Exception`, not `RuntimeException`) must be either caught or declared in the method signature (`throws`) — the compiler enforces handling. Used for **recoverable** conditions the caller should anticipate, e.g., `InsufficientFundsException` during a withdrawal.
- **Unchecked exceptions** (extend `RuntimeException`) don't require explicit handling — used for **programming errors** or conditions that generally shouldn't be recovered from at the call site, e.g., `NullPointerException`, `IllegalArgumentException`.
Design a **custom checked exception** when the caller has a meaningful, expected recovery path (e.g., catch `InsufficientFundsException` and prompt a retry); design a **custom unchecked exception** for invariant violations that indicate a bug (e.g., `InvalidAccountStateException` on corrupted internal state).

### 40. Exception propagation across the call stack
When an exception is thrown and not caught in the current method, the JVM **unwinds the stack** — it pops the current method's stack frame and re-raises the exception in the caller's frame, checking if *that* method catches it, and so on up the call chain until either a matching `catch` block is found or the exception reaches the top (crashing the thread, or in `main()`, terminating the program). All **local variables in each unwound frame are discarded** (they go out of scope as their frame is popped), and any `finally` blocks along the way are guaranteed to execute during this unwinding before the frame is removed.

### 41. Overriding rules for checked/unchecked exceptions from a superclass method
An overriding method **cannot throw new or broader checked exceptions** than the parent method declares — it can only throw the same checked exceptions, a subset of them, or narrower (subclass) versions. This preserves the **Liskov Substitution Principle**: code calling the parent method (and handling its declared exceptions) must still work safely if a subclass instance is substituted in. There's **no such restriction on unchecked exceptions** — an override can throw any unchecked exception freely, since callers aren't compiler-forced to handle those anyway.

### 42. Singleton pattern — thread-safe double-checked locking implementation
Singleton ensures a class has exactly one instance and provides a global access point to it (e.g., a single `ConnectionPoolManager` or `ConfigurationLoader` in a banking app). Double-checked locking avoids the performance cost of synchronizing on *every* call to `getInstance()` by only synchronizing the first time the instance is created:

```java
public class ConfigurationLoader {
    private static volatile ConfigurationLoader instance;

    private ConfigurationLoader() { }

    public static ConfigurationLoader getInstance() {
        if (instance == null) {                      // 1st check (no lock, fast path)
            synchronized (ConfigurationLoader.class) {
                if (instance == null) {               // 2nd check (inside lock)
                    instance = new ConfigurationLoader();
                }
            }
        }
        return instance;
    }
}
```
`volatile` is essential here: without it, due to instruction reordering, another thread could see a **partially constructed** object (the reference gets assigned before the constructor fully finishes), because object construction isn't atomic from the JVM's perspective. `volatile` establishes a happens-before relationship that prevents this reordering.

### 43. Factory design pattern
The Factory pattern centralizes object creation logic behind a method, so client code depends on an abstraction (interface/return type) rather than concrete constructors. E.g., a `PaymentGatewayFactory.getGateway("UPI")` returns the correct concrete `PaymentGateway` implementation (`UpiGateway`, `CardGateway`, `NetBankingGateway`) based on input, without the calling code ever writing `new UpiGateway()` directly. This decouples client code from knowing which concrete class to instantiate, makes adding new payment types a change localized to the factory, and supports the Open/Closed Principle.

### 44. Observer pattern — real-time stock price notification example
Observer defines a one-to-many dependency: when a **Subject** (the observable) changes state, all registered **Observers** are automatically notified. For a stock price system: a `Stock` class (Subject) maintains a list of `PriceObserver` implementations (e.g., `MobileAppNotifier`, `TradingDeskAlert`, `PriceChartUpdater`). When the stock's price changes via `setPrice()`, the `Stock` object loops through its registered observers and calls `update(newPrice)` on each — so the mobile app, trading desk, and chart all react independently without the `Stock` class needing to know anything about their specific behavior. This is the foundation of pub-sub systems and is close in spirit to Java's built-in listener patterns and reactive streams.

### 45. Thread-safe BankAccount design — preventing race conditions and deadlocks
To prevent **race conditions** on `balance`, synchronize the critical sections that read-then-modify state:

```java
public class BankAccount {
    private final Object lock = new Object();
    private double balance;

    public void deposit(double amount) {
        synchronized (lock) {
            balance += amount;
        }
    }

    public void withdraw(double amount) {
        synchronized (lock) {
            if (balance < amount) throw new InsufficientFundsException();
            balance -= amount;
        }
    }
}
```
For **transfers between two accounts** (locking two objects), naive locking causes **deadlocks** if Thread A locks Account1 then waits for Account2, while Thread B locks Account2 then waits for Account1. The standard fix is **lock ordering** — always acquire locks in a consistent global order (e.g., by account ID), regardless of transfer direction:

```java
public void transfer(BankAccount from, BankAccount to, double amount) {
    BankAccount first = from.id < to.id ? from : to;
    BankAccount second = from.id < to.id ? to : from;
    synchronized (first) {
        synchronized (second) {
            from.withdraw(amount);
            to.deposit(amount);
        }
    }
}
```

### 46. SRP and OCP with concrete examples
- **Single Responsibility Principle**: a class should have only **one reason to change**. A bloated `Account` class that handles balance logic, PDF statement generation, *and* email notifications violates SRP — a change in email formatting shouldn't risk breaking balance calculations. Split into `Account`, `StatementGenerator`, and `NotificationService`.
- **Open/Closed Principle**: classes should be **open for extension, closed for modification**. Instead of a `calculateInterest()` method with a giant `if/else` on account type (which you must edit every time a new account type is added), define an `InterestStrategy` interface and add new account-type behavior by creating a **new class**, without touching existing, tested code.

### 47. Liskov Substitution Principle — Square/Rectangle violation
LSP states that objects of a subclass must be **substitutable** for objects of the superclass without altering the correctness of the program. The classic violation: if `Square extends Rectangle` and overrides `setWidth()`/`setHeight()` to keep both sides equal (since a square must have equal sides), then code that does `rect.setWidth(5); rect.setHeight(10); assert rect.area() == 50;` **breaks** when `rect` is actually a `Square` — setting height also silently changes width, giving `area() == 100`. The subtype changes the *behavioral contract* of the parent, so it can't be safely substituted — even though mathematically a square "is a" rectangle, it violates LSP in an OOP model where width and height are independently mutable.

### 48. Dependency Inversion Principle
DIP states: high-level modules should not depend on low-level modules — **both should depend on abstractions**. In banking backend terms: your `TransactionService` (high-level business logic) should depend on a `TransactionRepository` **interface**, not directly on a `PostgresTransactionRepository` **implementation**. If business logic directly imports and instantiates the concrete database class, swapping databases, unit testing with a mock, or supporting multiple storage backends all become painful, invasive changes. By depending on an abstraction (with the concrete implementation injected via Spring/DI), the business logic stays stable regardless of what's underneath — the database becomes a swappable detail, not a structural dependency.

### 49. Structuring core OOP classes for an ATM or Parking Lot system
General LLD approach interviewers expect (using ATM as example):
- **Entities**: `Card`, `Account`, `Transaction`, `CashDispenser`.
- **Interfaces/Strategy**: `ATMState` (interface) with concrete states `IdleState`, `CardInsertedState`, `PinValidatedState`, `TransactionState`, `DispenseCashState` — implementing the **State design pattern** so the ATM's behavior changes based on its current state without giant conditional logic.
- **Core controller**: an `ATMMachine` class that holds the current `ATMState` and delegates operations (`insertCard()`, `enterPin()`, `withdraw()`) to that state object.
- **SOLID application**: `CashDispenser` is an interface so different physical dispenser hardware can be swapped in (DIP); each state class has one responsibility (SRP); adding a new transaction type extends the state machine without modifying existing states (OCP).
For a Parking Lot: `ParkingSpot` (abstract, subclassed by `CompactSpot`/`LargeSpot`/`HandicapSpot`), a `ParkingSpotAllocationStrategy` interface for pluggable allocation logic (nearest spot, by vehicle size), `Ticket`, `Vehicle` hierarchy, and a `ParkingLot` façade coordinating entry/exit. The key signal interviewers look for is: **interfaces for anything that varies, composition for relationships, and a clear single entry point coordinating the pieces.**

### 50. Why coding to an interface makes unit testing/mocking easier
When a class depends on a concrete implementation (e.g., `PostgresTransactionRepository` directly), a unit test for the business logic is forced to spin up a real database connection just to test unrelated logic — slow, fragile, and not a true *unit* test. When the class instead depends on an interface (`TransactionRepository`), tests can inject a lightweight **mock or stub** implementation (via Mockito, for example) that returns controlled, predictable data instantly, with zero infrastructure. This isolates the test to just the logic under examination, makes tests fast and deterministic, and is precisely why DIP and interface-based design are treated as prerequisites for testable code in high-load financial engines where test suites must run quickly and reliably on every build.

---

## Quick priority pass for the 17th
Per your notes, prioritize: **Category 3 (memory/concurrency: Q29–38)** and **Category 2 (keyword traps: Q13–25)** — these are the highest-yield for UBS's elimination criteria. Make sure you can *write* Q42 (Singleton) and Q45 (thread-safe BankAccount) on a whiteboard from memory without hesitation — these two get asked disproportionately often in banking-tech interviews.
