# Java Collection Framework - Complete Notes

## Table of Contents
1. [Introduction](#introduction)
2. [What is Java Collection Framework?](#what-is-java-collection-framework)
3. [Why We Need Collection Framework](#why-we-need-collection-framework)
4. [Collection Framework Hierarchy](#collection-framework-hierarchy)
5. [Iterable Interface](#iterable-interface)
6. [Collection Interface](#collection-interface)
7. [Collection vs Collections](#collection-vs-collections)

---

## Introduction

- **Added in**: Java 1.2
- **Package**: `java.util`
- Collection Framework provides a unified architecture to manage groups of objects

---

## What is Java Collection Framework?

### Collection
- A **group of objects** (also known as elements)
- Example: `int[] a = {1, 2, 3, 4}` is a collection of integers

### Framework
- Provides **architecture** to manage groups of objects
- Includes pre-built classes, interfaces, and methods
- Supports operations like: add, update, delete, search
- **Extensible**: Can build custom functionality on top of it

---

## Why We Need Collection Framework

### Problem Before Java 1.2

Before the Collection Framework, Java had:
- Arrays
- Vector
- Hashtable

**Major Problem**: **No Common Interface**

#### Example of the Problem:

```java
// Array - read/write
int[] array = new int[4];
array[0] = 1;           // Write
int value = array[0];    // Read

// Vector - read/write
Vector<Integer> vector = new Vector<>();
vector.add(1);          // Write (different method!)
int value = vector.get(0); // Read (different method!)
```

**Issues**:
- Different methods for different collections
- Difficult to remember syntax for each collection type
- No standardization

### Solution: Collection Framework

✅ Provides **common interface** for all collections  
✅ Same methods (`add()`, `get()`, `remove()`) work across different collection types  
✅ Focus on **choosing the right collection** rather than memorizing methods

---

## Collection Framework Hierarchy

```
                    Iterable (Interface)
                        |
                   Collection (Interface)
                   /      |      \
                  /       |       \
              List      Set      Queue
               |         |         |
        ┌──────┴────┐    |    ┌────┴─────┐
        |           |    |    |          |
    ArrayList   LinkedList  HashSet  PriorityQueue
        |                    |
     Vector              LinkedHashSet
        |
      Stack
```

### Key Points:
- **Light Blue** = Interface
- **Pink** = Concrete Class
- **Iterable** → Parent of Collection (added in Java 1.5)
- **Collection** → Parent of List, Set, Queue (added in Java 1.2)
- **Map** is separate (NOT part of Iterable/Collection hierarchy)

---

## Iterable Interface

**Purpose**: Used to **traverse** (iterate through) collections

**Added in**: Java 1.5

### Methods:

#### 1. `iterator()` Method (Java 1.5)
Returns an Iterator object with three methods:

| Method | Description |
|--------|-------------|
| `hasNext()` | Returns `true` if more elements exist |
| `next()` | Returns the next element |
| `remove()` | Removes the last returned element |

#### Example:
```java
List<Integer> values = new ArrayList<>();
values.add(1);
values.add(2);
values.add(3);
values.add(4);

// Method 1: Using Iterator
Iterator<Integer> iterator = values.iterator();
while(iterator.hasNext()) {
    int value = iterator.next();
    System.out.println(value);
    
    if(value == 3) {
        iterator.remove(); // Removes 3 from list
    }
}
// Output: 1, 2, 3, 4
// List now contains: [1, 2, 4]
```

#### 2. `forEach()` Method (Java 1.8)
Uses **lambda expressions** to iterate

```java
// Method 2: Enhanced for loop
for(int value : values) {
    System.out.println(value);
}

// Method 3: forEach with Lambda Expression
values.forEach(value -> System.out.println(value));
```

### Three Ways to Iterate Collections:
1. **Iterator** → `iterator()`, `hasNext()`, `next()`
2. **Enhanced for loop** → `for(int val : collection)`
3. **forEach method** → `collection.forEach(val -> ...)`

---

## Collection Interface

**Purpose**: Represents a group of objects and provides methods to work with them

**Added in**: Java 1.2

### Common Methods:

| Method | Description | Example |
|--------|-------------|---------|
| `size()` | Returns total elements | `values.size()` → 3 |
| `isEmpty()` | Checks if collection is empty | `values.isEmpty()` → false |
| `contains(Object)` | Checks if element exists | `values.contains(5)` → false |
| `add(Object)` | Adds an element | `values.add(5)` |
| `remove(Object)` | Removes an element | `values.remove(3)` |
| `remove(index)` | Removes by index | `values.remove(2)` |
| `toArray()` | Converts to array | `Object[] arr = values.toArray()` |
| `addAll(Collection)` | Adds another collection | `values.addAll(stackValues)` |
| `removeAll(Collection)` | Removes all elements from another collection | `values.removeAll(stackValues)` |
| `containsAll(Collection)` | Checks if all elements exist | `values.containsAll(stackValues)` |
| `clear()` | Removes all elements | `values.clear()` |
| `equals(Collection)` | Checks equality | `values.equals(other)` |
| `stream()` | Returns a stream (Java 1.8) | `values.stream()` |

### Complete Example:

```java
List<Integer> values = new ArrayList<>();
values.add(2);
values.add(3);
values.add(4);

// Size
System.out.println(values.size()); // 3

// Is Empty
System.out.println(values.isEmpty()); // false

// Contains
System.out.println(values.contains(5)); // false
values.add(5);
System.out.println(values.contains(5)); // true

// Remove by index
values.remove(3); // Removes element at index 3 (which is 5)
System.out.println(values.contains(5)); // false

// Remove by object
values.remove(Integer.valueOf(3)); // Removes the value 3
System.out.println(values.contains(3)); // false

// Add another collection
Stack<Integer> stackValues = new Stack<>();
stackValues.add(6);
stackValues.add(7);
stackValues.add(8);

values.addAll(stackValues); // values = [2, 4, 6, 7, 8]
System.out.println(values.containsAll(stackValues)); // true

// Remove one element
values.remove(Integer.valueOf(7));
System.out.println(values.containsAll(stackValues)); // false (7 is missing)

// Remove all
values.removeAll(stackValues); // Removes 6, 8
System.out.println(values.contains(8)); // false

// Clear
values.clear();
System.out.println(values.isEmpty()); // true
```

---

## Collection vs Collections

### Collection (Interface)
- **Type**: Interface
- **Purpose**: Part of Collection Framework
- **Role**: Defines common methods for all collection types
- **Usage**: Implemented by ArrayList, LinkedList, HashSet, etc.

```java
Collection<Integer> list = new ArrayList<>();
list.add(1);
```

### Collections (Utility Class)
- **Type**: Class (Utility)
- **Purpose**: Provides static helper methods
- **Methods**: All methods are **static**
- **Operations**: Sort, search, reverse, shuffle, min, max, etc.

```java
List<Integer> list = new ArrayList<>();
list.add(3);
list.add(1);
list.add(2);
list.add(4);

// Using Collections utility methods
Collections.max(list);      // Returns 4
Collections.min(list);      // Returns 1
Collections.sort(list);     // Sorts: [1, 2, 3, 4]
Collections.reverse(list);  // Reverses: [4, 3, 2, 1]
Collections.shuffle(list);  // Randomly shuffles
```

### Common Collections Methods:
- `sort(List)` - Sorts the list
- `reverse(List)` - Reverses the list
- `shuffle(List)` - Randomly shuffles
- `max(Collection)` - Returns maximum element
- `min(Collection)` - Returns minimum element
- `binarySearch(List, key)` - Binary search
- `copy(dest, src)` - Copies one list to another
- `swap(List, i, j)` - Swaps two elements
- `rotate(List, distance)` - Rotates elements

---

## Key Takeaways

✅ **Collection Framework** = Collections + Framework  
✅ Collections have **common interface** → Easy to use  
✅ **Iterable** enables iteration (3 ways: iterator, for-each loop, forEach method)  
✅ **Collection interface** provides standard methods for all collections  
✅ **Collection** (interface) vs **Collections** (utility class)  
✅ Choose collection based on **use case** (Stack, Queue, List, Set)  

---

## Coming Next

- Detailed coverage of each collection type:
  - ArrayList
  - LinkedList
  - Vector
  - Stack
  - PriorityQueue
  - HashSet
  - TreeSet
  - LinkedHashSet
- Map Interface and implementations
- **Streams API** (Separate detailed topic)

---

## Practice Tips

1. Try creating different collections and use common methods
2. Practice all three iteration methods
3. Understand when to use Collection vs Collections
4. Focus on **choosing the right collection** for your use case
5. Remember: Iterator object was present since Java 1.2, but Iterable interface was added in Java 1.5

---

*Happy Coding! 🚀*
