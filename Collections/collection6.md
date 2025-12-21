# Java Set - Complete Notes

## Table of Contents
1. [Set Interface Overview](#set-interface-overview)
2. [How Set Works Internally](#how-set-works-internally)
3. [Set Methods](#set-methods)
4. [HashSet](#hashset)
5. [LinkedHashSet](#linkedhashset)
6. [TreeSet](#treeset)
7. [Concurrent Modification Exception](#concurrent-modification-exception)
8. [Comparison Tables](#comparison-tables)

---

## Set Interface Overview

### What is Set?

- **Interface** extending Collection
- **Collection of unique objects** (no duplicates)
- **Not ordered** by default (except LinkedHashSet)
- **No index-based access** (unlike List)
- Can have **at most one null** element

### Set Hierarchy

```
Collection (Interface)
    ↓
  Set (Interface)
    ↓
  ┌─────────┬──────────────┬──────────┐
  ↓         ↓              ↓          ↓
HashSet  LinkedHashSet  TreeSet  (Others)
```

### Key Characteristics

| Feature | Description |
|---------|-------------|
| **Duplicates** | ❌ Not allowed |
| **Order** | ❌ Not guaranteed (except LinkedHashSet) |
| **Index Access** | ❌ Not supported |
| **Null Elements** | ✅ One null allowed (except TreeSet) |
| **Thread-Safe** | ❌ Not by default |

---

## How Set Works Internally

### The Secret: Set Uses Map! 🤫

**Key Concept**: Set internally uses Map (HashMap, LinkedHashMap, or TreeMap).

#### How?

```java
// When you do:
Set<Integer> set = new HashSet<>();
set.add(12);

// Internally, HashSet does:
Map<Integer, Object> map = new HashMap<>();
map.put(12, DUMMY_OBJECT);  // Dummy object as value
```

### Visual Representation

```
Set: [1, 5, 9, 13, 41]

Internally stored as Map:
┌─────┬─────────────┐
│ Key │   Value     │
├─────┼─────────────┤
│  1  │ DUMMY_OBJECT│
│  5  │ DUMMY_OBJECT│
│  9  │ DUMMY_OBJECT│
│ 13  │ DUMMY_OBJECT│
│ 41  │ DUMMY_OBJECT│
└─────┴─────────────┘

Key = Actual set element
Value = Just a placeholder (Object present)
```

### Code Proof: HashSet Implementation

```java
public class HashSet<E> {
    private transient HashMap<E,Object> map;
    
    // Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();
    
    public boolean add(E e) {
        return map.put(e, PRESENT) == null;
    }
}
```

---

## Why Set Doesn't Allow Duplicates

**Answer**: Because Map doesn't allow duplicate keys!

```java
Set<Integer> set = new HashSet<>();
set.add(12);  // map.put(12, DUMMY) → success
set.add(12);  // map.put(12, DUMMY) → key exists, returns false

// Map behavior:
// Duplicate key → overwrites value (but value is same dummy object)
// Set.add() returns false → duplicate not added
```

---

## Set Methods

### Basic Methods (Inherited from Collection)

| Method | Description |
|--------|-------------|
| `add(element)` | Adds element if not present |
| `remove(element)` | Removes element |
| `contains(element)` | Checks if element exists |
| `size()` | Returns number of elements |
| `isEmpty()` | Checks if set is empty |
| `clear()` | Removes all elements |

### Set-Specific Mathematical Operations

#### 1. Union (`addAll()`)

**Mathematics**: A ∪ B = All elements from both sets

```java
Set<Integer> set1 = new HashSet<>(Arrays.asList(1, 2, 11, 33, 4));
Set<Integer> set2 = new HashSet<>(Arrays.asList(11, 9, 8, 10, 5, 12));

set1.addAll(set2);  // Union operation

// Result: set1 = [1, 2, 4, 5, 8, 9, 10, 11, 12, 33]
// All unique elements from both sets
```

**Visual**:
```
set1: {1, 2, 11, 33, 4}
set2: {11, 9, 8, 10, 5, 12}
            ↓
set1.addAll(set2)
            ↓
Result: {1, 2, 4, 5, 8, 9, 10, 11, 12, 33}
```

#### 2. Difference (`removeAll()`)

**Mathematics**: A - B = Elements in A but not in B

```java
Set<Integer> set1 = new HashSet<>(Arrays.asList(1, 2, 11, 33, 4));
Set<Integer> set2 = new HashSet<>(Arrays.asList(11, 9, 8, 10, 5, 12));

set1.removeAll(set2);  // Difference operation

// Result: set1 = [1, 2, 33, 4]
// Only elements from set1 that are NOT in set2
```

**Visual**:
```
set1: {1, 2, 11, 33, 4}
set2: {11, 9, 8, 10, 5, 12}
            ↓
set1.removeAll(set2)
            ↓
Remove: 11, 12 (common elements)
            ↓
Result: {1, 2, 33, 4}
```

#### 3. Intersection (`retainAll()`)

**Mathematics**: A ∩ B = Common elements in both sets

```java
Set<Integer> set1 = new HashSet<>(Arrays.asList(1, 2, 11, 33, 4));
Set<Integer> set2 = new HashSet<>(Arrays.asList(11, 9, 8, 10, 5, 12));

set1.retainAll(set2);  // Intersection operation

// Result: set1 = [11]
// Only common elements
```

**Visual**:
```
set1: {1, 2, 11, 33, 4}
set2: {11, 9, 8, 10, 5, 12}
            ↓
set1.retainAll(set2)
            ↓
Keep only common: 11
            ↓
Result: {11}
```

---

## HashSet

### What is HashSet?

- **Implements**: Set interface
- **Backed by**: HashMap
- **Order**: ❌ Not maintained
- **Null**: ✅ One null allowed
- **Duplicates**: ❌ Not allowed

### Internal Structure

```java
public class HashSet<E> {
    private transient HashMap<E, Object> map;
    
    public HashSet() {
        map = new HashMap<>();
    }
}
```

### Complete Example

```java
Set<Integer> set = new HashSet<>();

// Adding elements
set.add(2);
set.add(77);
set.add(82);
set.add(63);
set.add(5);

// Iteration (no guaranteed order)
for (Integer num : set) {
    System.out.println(num);
}
// Possible output: 2, 82, 63, 5, 77 (any order)

// Size
System.out.println(set.size());  // 5

// Contains
System.out.println(set.contains(77));  // true

// Remove
set.remove(82);
System.out.println(set.size());  // 4

// Duplicates not allowed
set.add(5);  // Returns false, not added
System.out.println(set.size());  // Still 4
```

---

## HashSet - Thread Safety

### Not Thread-Safe by Default

```java
Set<Integer> set = new HashSet<>();
// ❌ Not safe for concurrent access
```

### Making it Thread-Safe

```java
// Option 1: ConcurrentHashMap.newKeySet() (Recommended)
Set<Integer> threadSafeSet = ConcurrentHashMap.newKeySet();
threadSafeSet.add(1);
threadSafeSet.add(2);

// Option 2: Collections.synchronizedSet()
Set<Integer> syncSet = Collections.synchronizedSet(new HashSet<>());
syncSet.add(1);
syncSet.add(2);
```

---

## HashSet Time Complexity

| Operation | Time Complexity | Reason |
|-----------|----------------|--------|
| `add(element)` | O(1) average | HashMap put() |
| `remove(element)` | O(1) average | HashMap remove() |
| `contains(element)` | O(1) average | HashMap containsKey() |
| Space | O(n) | HashMap storage |

**Note**: Same as HashMap because HashSet uses HashMap internally!

---

## LinkedHashSet

### What is LinkedHashSet?

- **Extends**: HashSet
- **Backed by**: LinkedHashMap
- **Order**: ✅ **Insertion order maintained**
- **Null**: ✅ One null allowed
- **Duplicates**: ❌ Not allowed

### Why Insertion Order?

**Answer**: Uses LinkedHashMap internally (doubly linked list)

```java
public class LinkedHashSet<E> extends HashSet<E> {
    public LinkedHashSet() {
        super(16, 0.75f, true);  // Calls HashSet constructor
    }
}

// HashSet constructor
HashSet(int capacity, float loadFactor, boolean dummy) {
    map = new LinkedHashMap<>(capacity, loadFactor);
}
```

### Internal Structure

```
LinkedHashMap structure:

HashMap Array + Doubly Linked List

Array:
[0] → [2] → [77]
[1] → [82]
[2] → [63] → [5]

Doubly Linked List (maintains order):
null ← [2] ⇄ [77] ⇄ [82] ⇄ [63] ⇄ [5] → null
       ↑                              ↑
      head                          tail
```

### Complete Example

```java
Set<Integer> set = new LinkedHashSet<>();

// Adding elements
set.add(2);
set.add(77);
set.add(82);
set.add(63);
set.add(5);

// Iteration (insertion order maintained)
for (Integer num : set) {
    System.out.println(num);
}
// Output: 2, 77, 82, 63, 5 (exact insertion order)
```

---

## LinkedHashSet - Access Order?

### Question: Can LinkedHashSet maintain access order (like LRU cache)?

**Answer**: ❌ No! Only **insertion order**.

### Why?

```java
// LinkedHashMap supports access order
Map<Integer, String> map = new LinkedHashMap<>(16, 0.75f, true);
//                                                         ↑
//                                               accessOrder = true

// But LinkedHashSet doesn't expose this
Set<Integer> set = new LinkedHashSet<>();
// Internal LinkedHashMap always has accessOrder = false (hardcoded)
```

**Code Evidence**:
```java
// In LinkedHashSet constructor
super(capacity, loadFactor, true);  // 'true' is just a dummy flag

// In HashSet
HashSet(int capacity, float loadFactor, boolean dummy) {
    map = new LinkedHashMap<>(capacity, loadFactor);
    // accessOrder not passed! Always defaults to false
}
```

---

## LinkedHashSet Thread Safety

### Not Thread-Safe

```java
Set<Integer> set = new LinkedHashSet<>();
// ❌ Not safe for concurrent access
```

### Making it Thread-Safe

```java
Set<Integer> syncSet = Collections.synchronizedSet(
    new LinkedHashSet<>()
);

// Usage
syncSet.add(1);
syncSet.add(2);
```

---

## LinkedHashSet Time Complexity

| Operation | Time Complexity | Reason |
|-----------|----------------|--------|
| `add(element)` | O(1) average | LinkedHashMap put() |
| `remove(element)` | O(1) average | LinkedHashMap remove() |
| `contains(element)` | O(1) average | LinkedHashMap containsKey() |
| Space | O(n) | LinkedHashMap storage + pointers |

**Note**: Same as LinkedHashMap!

---

## TreeSet

### What is TreeSet?

- **Implements**: NavigableSet (extends SortedSet, extends Set)
- **Backed by**: TreeMap (Red-Black Tree)
- **Order**: ✅ **Sorted order** (ascending by default)
- **Null**: ❌ No null allowed
- **Duplicates**: ❌ Not allowed

### Internal Structure

```java
public class TreeSet<E> {
    private transient NavigableMap<E, Object> m;
    
    public TreeSet() {
        this(new TreeMap<>());
    }
}
```

**Uses**: Red-Black Tree (self-balancing BST)

---

## TreeSet Sorting

### 1. Natural Ordering (Default)

```java
Set<Integer> set = new TreeSet<>();

// Adding elements
set.add(2);
set.add(77);
set.add(82);
set.add(63);
set.add(5);

// Iteration (sorted ascending)
for (Integer num : set) {
    System.out.println(num);
}
// Output: 2, 5, 63, 77, 82 (sorted!)
```

**Visual Tree**:
```
         [63]
        /    \
    [5]      [77]
    /           \
  [2]           [82]
```

### 2. Custom Comparator (Descending)

```java
// Descending order
Set<Integer> set = new TreeSet<>((a, b) -> b - a);

set.add(2);
set.add(77);
set.add(82);
set.add(63);
set.add(5);

// Iteration (sorted descending)
for (Integer num : set) {
    System.out.println(num);
}
// Output: 82, 77, 63, 5, 2 (reverse sorted!)
```

---

## TreeSet Time Complexity

| Operation | Time Complexity | Reason |
|-----------|----------------|--------|
| `add(element)` | O(log n) | TreeMap put() (Red-Black Tree) |
| `remove(element)` | O(log n) | TreeMap remove() |
| `contains(element)` | O(log n) | TreeMap containsKey() |
| Space | O(n) | TreeMap storage |

**Note**: Slower than HashSet/LinkedHashSet but maintains sorted order!

---

## Concurrent Modification Exception

### What is it?

**Exception thrown** when you modify a collection while iterating over it.

### Example: Problem

```java
Set<Integer> set = new HashSet<>(Arrays.asList(1, 2, 3, 4, 5));

Iterator<Integer> iterator = set.iterator();
while (iterator.hasNext()) {
    Integer num = iterator.next();
    System.out.println(num);
    
    if (num == 3) {
        set.add(8);  // ❌ ConcurrentModificationException!
    }
}
```

**Why?**
```
Set: [1, 2, 3, 4, 5]
            ↑
         Iterator here
            ↓
      Trying to add 8
            ↓
Thread 1: Reading (iterator)
Thread 2: Writing (add)
            ↓
    CONFLICT! ❌
```

### Solution 1: Use Thread-Safe Collection

```java
Set<Integer> set = ConcurrentHashMap.newKeySet();
set.addAll(Arrays.asList(1, 2, 3, 4, 5));

Iterator<Integer> iterator = set.iterator();
while (iterator.hasNext()) {
    Integer num = iterator.next();
    System.out.println(num);
    
    if (num == 3) {
        set.add(8);  // ✅ Works! Thread-safe
    }
}
```

### Solution 2: Use Iterator.remove()

```java
Set<Integer> set = new HashSet<>(Arrays.asList(1, 2, 3, 4, 5));

Iterator<Integer> iterator = set.iterator();
while (iterator.hasNext()) {
    Integer num = iterator.next();
    
    if (num == 3) {
        iterator.remove();  // ✅ Safe removal
    }
}
```

---

## Comparison Tables

### HashSet vs LinkedHashSet vs TreeSet

| Feature | HashSet | LinkedHashSet | TreeSet |
|---------|---------|---------------|---------|
| **Order** | ❌ No order | ✅ Insertion order | ✅ Sorted order |
| **Backed by** | HashMap | LinkedHashMap | TreeMap |
| **Data Structure** | Hash Table | Hash Table + Doubly LL | Red-Black Tree |
| **Null** | ✅ One null | ✅ One null | ❌ No null |
| **Duplicates** | ❌ No | ❌ No | ❌ No |
| **Performance** | ⭐⭐⭐ Fastest | ⭐⭐ Medium | ⭐ Slowest |
| **add()** | O(1) | O(1) | O(log n) |
| **contains()** | O(1) | O(1) | O(log n) |
| **Thread-Safe** | ❌ No | ❌ No | ❌ No |
| **Use Case** | General purpose | Need insertion order | Need sorted data |
| **Since** | Java 1.2 | Java 1.4 | Java 1.2 |

---

### Time Complexity Summary

| Operation | HashSet | LinkedHashSet | TreeSet |
|-----------|---------|---------------|---------|
| `add(element)` | O(1) avg | O(1) avg | O(log n) |
| `remove(element)` | O(1) avg | O(1) avg | O(log n) |
| `contains(element)` | O(1) avg | O(1) avg | O(log n) |
| `size()` | O(1) | O(1) | O(1) |
| `clear()` | O(n) | O(n) | O(n) |
| Space | O(n) | O(n) | O(n) |

---

### Thread-Safe Alternatives

| Non-Thread-Safe | Thread-Safe Alternative |
|-----------------|------------------------|
| HashSet | `ConcurrentHashMap.newKeySet()` |
| HashSet | `Collections.synchronizedSet(new HashSet<>())` |
| LinkedHashSet | `Collections.synchronizedSet(new LinkedHashSet<>())` |
| TreeSet | `Collections.synchronizedSet(new TreeSet<>())` |

---

## When to Use What?

### Use HashSet When:
✅ Order doesn't matter  
✅ Need **fastest** performance (O(1))  
✅ General-purpose unique collection  
✅ Most common use case  

### Use LinkedHashSet When:
✅ Need to maintain **insertion order**  
✅ Want predictable iteration order  
✅ Slight performance overhead acceptable  
✅ Need to track order of insertion  

### Use TreeSet When:
✅ Need **sorted** collection (ascending/descending)  
✅ Need **range queries** (headSet, tailSet, subSet)  
✅ Need **navigation** (lower, higher, floor, ceiling)  
✅ O(log n) performance acceptable  
✅ Cannot have null elements  

---

## Complete Examples

### HashSet Example
```java
Set<Integer> set = new HashSet<>();
set.add(5);
set.add(2);
set.add(8);
set.add(2);  // Duplicate, not added

System.out.println(set);  // [2, 5, 8] or any order
System.out.println(set.size());  // 3
System.out.println(set.contains(5));  // true
```

### LinkedHashSet Example
```java
Set<Integer> set = new LinkedHashSet<>();
set.add(5);
set.add(2);
set.add(8);

System.out.println(set);  // [5, 2, 8] (insertion order)

for (Integer num : set) {
    System.out.println(num);  // 5, 2, 8
}
```

### TreeSet Example
```java
Set<Integer> set = new TreeSet<>();
set.add(5);
set.add(2);
set.add(8);

System.out.println(set);  // [2, 5, 8] (sorted)

for (Integer num : set) {
    System.out.println(num);  // 2, 5, 8
}

// TreeSet-specific methods
TreeSet<Integer> treeSet = (TreeSet<Integer>) set;
System.out.println(treeSet.first());   // 2
System.out.println(treeSet.last());    // 8
System.out.println(treeSet.higher(5)); // 8
System.out.println(treeSet.lower(5));  // 2
```

---

## Set Mathematical Operations

### Union, Intersection, Difference

```java
Set<Integer> set1 = new HashSet<>(Arrays.asList(1, 2, 11, 33, 4));
Set<Integer> set2 = new HashSet<>(Arrays.asList(11, 9, 8, 10, 5, 12));

// Union (A ∪ B)
Set<Integer> union = new HashSet<>(set1);
union.addAll(set2);
System.out.println("Union: " + union);
// Output: [1, 2, 4, 5, 8, 9, 10, 11, 12, 33]

// Intersection (A ∩ B)
Set<Integer> intersection = new HashSet<>(set1);
intersection.retainAll(set2);
System.out.println("Intersection: " + intersection);
// Output: [11, 12]

// Difference (A - B)
Set<Integer> difference = new HashSet<>(set1);
difference.removeAll(set2);
System.out.println("Difference: " + difference);
// Output: [1, 2, 33, 4]
```

---

## Key Takeaways

### Internal Structure
✅ **Set uses Map internally**  
✅ Set elements = Map keys  
✅ Map values = Dummy object (PRESENT)  
✅ No duplicates because Map doesn't allow duplicate keys  

### Three Implementations
✅ **HashSet**: HashMap → No order, O(1)  
✅ **LinkedHashSet**: LinkedHashMap → Insertion order, O(1)  
✅ **TreeSet**: TreeMap → Sorted order, O(log n)  

### Key Differences
✅ **HashSet**: Fast, no order  
✅ **LinkedHashSet**: Predictable iteration (insertion order)  
✅ **TreeSet**: Sorted, navigation methods, slower  

### Thread Safety
✅ None are thread-safe by default  
✅ Use `ConcurrentHashMap.newKeySet()` or `Collections.synchronizedSet()`  
✅ Modifying during iteration → ConcurrentModificationException  

---

## Quick Reference

```java
// HashSet - No order, O(1)
Set<Integer> hashSet = new HashSet<>();

// LinkedHashSet - Insertion order, O(1)
Set<Integer> linkedHashSet = new LinkedHashSet<>();

// TreeSet - Sorted ascending, O(log n)
Set<Integer> treeSet = new TreeSet<>();

// TreeSet - Sorted descending, O(log n)
Set<Integer> reverseTreeSet = new TreeSet<>((a, b) -> b - a);

// Thread-safe HashSet
Set<Integer> threadSafeSet = ConcurrentHashMap.newKeySet();

// Thread-safe wrapper
Set<Integer> syncSet = Collections.synchronizedSet(new HashSet<>());
```

---

*Happy Coding! 🚀*
