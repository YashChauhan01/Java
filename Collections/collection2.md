# Java Queue, PriorityQueue, Comparator & Comparable - Complete Notes

## Table of Contents
1. [Queue Interface](#queue-interface)
2. [PriorityQueue](#priorityqueue)
3. [Comparator vs Comparable](#comparator-vs-comparable)
4. [Comparator Deep Dive](#comparator-deep-dive)
5. [Comparable Deep Dive](#comparable-deep-dive)
6. [Key Differences](#key-differences)

---

## Queue Interface

### What is Queue?
- **Interface** that extends Collection
- Represents a collection designed for holding elements prior to processing
- Generally follows **FIFO** (First In First Out) approach
- **Exceptions**: PriorityQueue has different behavior

### Queue Structure
```
Front                    Rear
  ↓                       ↓
[1] → [2] → [3] → [4] → [5]
  ↑                       ↑
Remove                   Add
```

- **Add**: Elements added at the rear
- **Remove**: Elements removed from the front

---

## Queue Methods

Queue provides **6 key methods** (3 pairs with different exception handling):

### Method Comparison Table

| Operation | Exception on Failure | Returns Special Value |
|-----------|---------------------|----------------------|
| **Insert** | `add(e)` - throws exception | `offer(e)` - returns false |
| **Remove** | `remove()` - throws exception | `poll()` - returns null |
| **Examine** | `element()` - throws exception | `peek()` - returns null |

### Detailed Method Descriptions

#### 1. `add(E e)` vs `offer(E e)`
```java
Queue<Integer> queue = new LinkedList<>();

// add() - throws exception on failure
queue.add(5);           // Returns true
queue.add(null);        // Throws NullPointerException

// offer() - returns false on failure
queue.offer(10);        // Returns true
queue.offer(null);      // Returns false (no exception)
```

#### 2. `remove()` vs `poll()`
```java
// remove() - throws exception if queue is empty
int value = queue.remove();  // Returns and removes head
// If empty: throws NoSuchElementException

// poll() - returns null if queue is empty
Integer value = queue.poll(); // Returns and removes head
// If empty: returns null
```

#### 3. `element()` vs `peek()`
```java
// element() - throws exception if queue is empty
int value = queue.element();  // Returns head (doesn't remove)
// If empty: throws NoSuchElementException

// peek() - returns null if queue is empty
Integer value = queue.peek(); // Returns head (doesn't remove)
// If empty: returns null
```

---

## PriorityQueue

### What is PriorityQueue?
- Implements the Queue interface
- Elements ordered by **priority**, not insertion order
- Uses **Heap** data structure internally
- Two types:
  - **Min PriorityQueue** (Min Heap) - smallest element at front
  - **Max PriorityQueue** (Max Heap) - largest element at front

### Natural Ordering (Default Behavior)

By default, PriorityQueue uses **natural ordering**:
- **Integer**: Ascending order (Min Heap)
- **String**: Lexicographical order
- **Custom objects**: Must implement Comparable

---

## Min PriorityQueue Example

```java
// Creating Min Heap (default)
PriorityQueue<Integer> minPQ = new PriorityQueue<>();

// Adding elements
minPQ.add(5);
minPQ.add(2);
minPQ.add(8);
minPQ.add(1);

// Internal structure (Min Heap):
//       1
//      / \
//     2   8
//    /
//   5

// Printing elements (level order traversal)
System.out.println(minPQ); // Output: [1, 2, 8, 5]

// Polling elements (removes in priority order)
while(!minPQ.isEmpty()) {
    System.out.print(minPQ.poll() + " "); // Output: 1 2 5 8
}
```

**Key Point**: Even though we added `5, 2, 8, 1`, the output is `1, 2, 5, 8` because PriorityQueue maintains min heap property.

---

## Max PriorityQueue Example

```java
// Creating Max Heap using Comparator
PriorityQueue<Integer> maxPQ = new PriorityQueue<>(Collections.reverseOrder());
// OR
PriorityQueue<Integer> maxPQ = new PriorityQueue<>((a, b) -> b - a);

// Adding elements
maxPQ.add(5);
maxPQ.add(2);
maxPQ.add(8);
maxPQ.add(1);

// Internal structure (Max Heap):
//       8
//      / \
//     5   2
//    /
//   1

// Printing elements
System.out.println(maxPQ); // Output: [8, 5, 2, 1]

// Polling elements
while(!maxPQ.isEmpty()) {
    System.out.print(maxPQ.poll() + " "); // Output: 8 5 2 1
}
```

---

## PriorityQueue Time Complexity

| Operation | Time Complexity | Reason |
|-----------|----------------|--------|
| `add(e)` / `offer(e)` | O(log n) | Heapify after insertion |
| `peek()` | O(1) | Just look at root |
| `poll()` / `remove()` | O(log n) | Remove root + heapify |
| `remove(Object)` | O(n) | Search + remove + heapify |

---

## Comparator vs Comparable

### Why Do We Need Them?

#### Problem 1: Sorting Primitive Arrays
```java
int[] array = {5, 1, 8, 10};
Arrays.sort(array); // Works fine - ascending order: [1, 5, 8, 10]
```
✅ Works because Java knows how to compare integers

**But what about descending order?** 🤔

#### Problem 2: Sorting Custom Objects
```java
class Car {
    String name;
    String type;
    
    Car(String name, String type) {
        this.name = name;
        this.type = type;
    }
}

Car[] cars = new Car[3];
cars[0] = new Car("SUV", "Petrol");
cars[1] = new Car("Sedan", "Diesel");
cars[2] = new Car("Hatchback", "CNG");

Arrays.sort(cars); // ❌ ERROR: Car cannot be cast to Comparable
```

**Why the error?**
- Java doesn't know how to compare `Car` objects
- Should it compare by `name` or `type`?
- Should it sort ascending or descending?

**Solution**: Use **Comparator** or **Comparable**

---

## Comparator Deep Dive

### What is Comparator?
- **Functional Interface** with one abstract method: `compare(T o1, T o2)`
- Used to define **custom sorting logic**
- Can create **multiple sorting strategies** for the same class

### Comparator Interface
```java
@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
}
```

### Compare Method Contract
```java
compare(Object o1, Object o2):
    - Returns POSITIVE (>0) if o1 > o2  → SWAP
    - Returns ZERO (0) if o1 == o2     → NO SWAP
    - Returns NEGATIVE (<0) if o1 < o2 → NO SWAP
```

### How Sorting Algorithms Use Comparator

```java
// Inside sorting algorithm (simplified)
if (comparator.compare(obj1, obj2) > 0) {
    swap(obj1, obj2);
}
```

---

## Comparator Examples

### Example 1: Sorting Integer Array in Descending Order

```java
Integer[] array = {17, 3, 5, 1, 10};

// Method 1: Lambda Expression (Recommended)
Arrays.sort(array, (val1, val2) -> val2 - val1);

// Method 2: Explicit Comparator
Arrays.sort(array, new Comparator<Integer>() {
    @Override
    public int compare(Integer val1, Integer val2) {
        return val2 - val1; // Descending order
    }
});

System.out.println(Arrays.toString(array)); // [17, 10, 5, 3, 1]
```

**Why `val2 - val1` for descending?**

Let's trace with values `6` and `9`:

**Ascending Order** (`val1 - val2`):
```
compare(6, 9) → 6 - 9 = -3 (negative)
-3 > 0? NO → Don't swap → [6, 9] ✅ (smaller first)
```

**Descending Order** (`val2 - val1`):
```
compare(6, 9) → 9 - 6 = 3 (positive)
3 > 0? YES → Swap → [9, 6] ✅ (larger first)
```

---

### Example 2: Sorting Custom Objects

```java
class Car {
    String name;
    String type;
    
    Car(String name, String type) {
        this.name = name;
        this.type = type;
    }
    
    @Override
    public String toString() {
        return name + "(" + type + ")";
    }
}

Car[] cars = new Car[3];
cars[0] = new Car("SUV", "Petrol");
cars[1] = new Car("Sedan", "Diesel");
cars[2] = new Car("Hatchback", "CNG");
```

#### Sort by Type (Descending)
```java
Arrays.sort(cars, (c1, c2) -> c2.type.compareTo(c1.type));
// Output: [SUV(Petrol), Sedan(Diesel), Hatchback(CNG)]
```

#### Sort by Name (Ascending)
```java
Arrays.sort(cars, (c1, c2) -> c1.name.compareTo(c2.name));
// Output: [Hatchback(CNG), Sedan(Diesel), SUV(Petrol)]
```

#### Sort by Name (Descending)
```java
Arrays.sort(cars, (c1, c2) -> c2.name.compareTo(c1.name));
// Output: [SUV(Petrol), Sedan(Diesel), Hatchback(CNG)]
```

---

## Three Ways to Use Comparator

### 1. Lambda Expression (Modern Way) ⭐
```java
Arrays.sort(array, (a, b) -> b - a);
```

### 2. Separate Comparator Class
```java
class CarNameComparator implements Comparator<Car> {
    @Override
    public int compare(Car c1, Car c2) {
        return c2.name.compareTo(c1.name); // Descending
    }
}

// Usage
Arrays.sort(cars, new CarNameComparator());
```

### 3. Anonymous Inner Class
```java
Arrays.sort(cars, new Comparator<Car>() {
    @Override
    public int compare(Car c1, Car c2) {
        return c1.name.compareTo(c2.name);
    }
});
```

---

## Comparable Deep Dive

### What is Comparable?
- Interface with one method: `compareTo(T o)`
- Defines **natural ordering** for a class
- Object compares itself with another object
- **Only ONE sorting logic** per class

### Comparable Interface
```java
public interface Comparable<T> {
    int compareTo(T o);
}
```

### CompareTo Method Contract
```java
compareTo(Object o):
    - Returns POSITIVE (>0) if this > o
    - Returns ZERO (0) if this == o
    - Returns NEGATIVE (<0) if this < o
```

---

## Comparable Example

### Integer Class (Built-in)
```java
Integer[] array = {1, 7, 6, 3};
Arrays.sort(array); // Uses Integer's compareTo method
// Output: [1, 3, 6, 7]
```

**How it works internally:**
```java
// Inside Integer class
public class Integer implements Comparable<Integer> {
    public int compareTo(Integer anotherInteger) {
        return this.value - anotherInteger.value; // Ascending
    }
}
```

### Custom Class with Comparable

```java
class Car implements Comparable<Car> {
    String name;
    String type;
    
    Car(String name, String type) {
        this.name = name;
        this.type = type;
    }
    
    @Override
    public int compareTo(Car other) {
        // Sort by type in ascending order
        return this.type.compareTo(other.type);
    }
    
    @Override
    public String toString() {
        return name + "(" + type + ")";
    }
}

// Usage
List<Car> carList = new ArrayList<>();
carList.add(new Car("SUV", "Petrol"));
carList.add(new Car("Sedan", "Diesel"));
carList.add(new Car("Hatchback", "CNG"));

Collections.sort(carList); // Uses Car's compareTo method
// Output: [Hatchback(CNG), Sedan(Diesel), SUV(Petrol)]
```

**Key Point**: No need to pass Comparator - sorting logic is in the class itself!

---

## Key Differences: Comparator vs Comparable

| Aspect | Comparator | Comparable |
|--------|-----------|-----------|
| **Interface Method** | `compare(T o1, T o2)` | `compareTo(T o)` |
| **Parameters** | Two objects | One object (compares with `this`) |
| **Location** | External class or lambda | Inside the class being compared |
| **Sorting Logic** | Multiple strategies possible | Only ONE strategy |
| **Flexibility** | ✅ High - can change without modifying class | ❌ Low - requires class modification |
| **Natural Order** | ❌ Doesn't define | ✅ Defines natural ordering |
| **Package** | `java.util.Comparator` | `java.lang.Comparable` |
| **Usage** | `Arrays.sort(arr, comparator)` | `Arrays.sort(arr)` |

---

## When to Use What?

### Use Comparable When:
✅ You have **one clear natural ordering** for the class  
✅ You control the source code of the class  
✅ The sorting logic won't change frequently  

**Example**: Person sorted by age, Student sorted by roll number

### Use Comparator When:
✅ You need **multiple sorting strategies**  
✅ You don't control the source code  
✅ You want to sort in different ways at different times  
✅ You want to override natural ordering  

**Example**: Sort employees by name, salary, department, joining date

---

## Practical Scenarios

### Scenario 1: Sorting with Comparable Only
```java
class Student implements Comparable<Student> {
    String name;
    int rollNo;
    
    @Override
    public int compareTo(Student other) {
        return this.rollNo - other.rollNo; // Always by roll number
    }
}

List<Student> students = new ArrayList<>();
Collections.sort(students); // Sorted by roll number
```

**Problem**: What if you want to sort by name sometimes? 🤔

### Scenario 2: Sorting with Comparator (Flexible)
```java
class Student {
    String name;
    int rollNo;
}

List<Student> students = new ArrayList<>();

// Sort by roll number
Collections.sort(students, (s1, s2) -> s1.rollNo - s2.rollNo);

// Sort by name
Collections.sort(students, (s1, s2) -> s1.name.compareTo(s2.name));

// Sort by name descending
Collections.sort(students, (s1, s2) -> s2.name.compareTo(s1.name));
```

**Solution**: Multiple sorting options without modifying Student class! ✅

---

## Summary

### Queue
- FIFO data structure with special methods
- Six methods: add/offer, remove/poll, element/peek
- Use offer/poll/peek to avoid exceptions

### PriorityQueue
- Orders elements by priority (not insertion order)
- Uses Heap internally (Min Heap by default)
- Use Comparator to create Max Heap
- Essential for solving heap-based problems

### Comparator
- Functional interface for custom sorting
- `compare(o1, o2)` - compares two objects
- Multiple sorting strategies possible
- More flexible

### Comparable
- Interface for natural ordering
- `compareTo(o)` - compares with itself
- Only one sorting strategy
- Built into the class

---

## Quick Reference

### Ascending vs Descending Order

```java
// Ascending: val1 - val2 (or a - b)
Arrays.sort(array, (a, b) -> a - b);

// Descending: val2 - val1 (or b - a)
Arrays.sort(array, (a, b) -> b - a);

// String Ascending
Arrays.sort(array, (s1, s2) -> s1.compareTo(s2));

// String Descending
Arrays.sort(array, (s1, s2) -> s2.compareTo(s1));
```

### PriorityQueue Quick Setup

```java
// Min Heap (default)
PriorityQueue<Integer> minHeap = new PriorityQueue<>();

// Max Heap
PriorityQueue<Integer> maxHeap = new PriorityQueue<>((a, b) -> b - a);
// OR
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
```

---

## Practice Tips

1. ✅ Practice lambda expressions with Comparator
2. ✅ Understand the `compare()` return values (+, 0, -)
3. ✅ Remember: `val2 - val1` for descending
4. ✅ Use PriorityQueue for heap problems
5. ✅ Choose Comparable for single natural ordering
6. ✅ Choose Comparator for multiple sorting strategies

---

*Happy Coding! 🚀*
