# Java LinkedHashMap & TreeMap - Complete Notes

## Table of Contents
1. [LinkedHashMap Overview](#linkedhashmap-overview)
2. [LinkedHashMap Internal Structure](#linkedhashmap-internal-structure)
3. [Insertion Order vs Access Order](#insertion-order-vs-access-order)
4. [LinkedHashMap Thread Safety](#linkedhashmap-thread-safety)
5. [TreeMap Overview](#treemap-overview)
6. [TreeMap Internal Structure](#treemap-internal-structure)
7. [SortedMap & NavigableMap Methods](#sortedmap--navigablemap-methods)
8. [Comparison Tables](#comparison-tables)

---

## LinkedHashMap Overview

### What is LinkedHashMap?

- **Extends**: HashMap
- **Key Feature**: Maintains **order** (unlike HashMap)
- **Two Types of Order**:
  1. **Insertion Order** (default)
  2. **Access Order** (configurable)

### HashMap vs LinkedHashMap

| Feature | HashMap | LinkedHashMap |
|---------|---------|---------------|
| **Order** | ❌ No order maintained | ✅ Maintains order |
| **Structure** | Array + Linked List/Tree | Array + Doubly Linked List |
| **Performance** | Faster | Slightly slower (extra pointers) |
| **Use Case** | When order doesn't matter | When order is important |

---

## LinkedHashMap Internal Structure

### Node Structure Comparison

#### HashMap Node
```java
static class Node<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;  // Only next pointer
}
```

#### LinkedHashMap Node (Extends HashMap Node)
```java
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before;  // 👈 Extra: Previous node
    Entry<K,V> after;   // 👈 Extra: Next node
}
```

**Key Difference**: LinkedHashMap adds **two extra pointers** (before & after) for doubly linked list.

---

## How LinkedHashMap Works

### Visual Example

**Inserting**: `{1: "A", 21: "B", 23: "C", 141: "D", 25: "E"}`

#### Step 1: HashMap Array Structure (Same as HashMap)
```
Array Structure (size 3):
[0] → [1:"A"] → [21:"B"]
[1] → [23:"C"] → [25:"E"]
[2] → [141:"D"]
```

#### Step 2: Doubly Linked List (Additional in LinkedHashMap)
```
Insertion Order Linked List:

null ← [1:"A"] ⇄ [21:"B"] ⇄ [23:"C"] ⇄ [141:"D"] ⇄ [25:"E"] → null
       ↑ head                                        ↑ tail
       before/after pointers maintain order
```

### Detailed Node Structure
```
Node for key=1:
┌─────────────────────────┐
│ hash: 12345             │
│ key: 1                  │
│ value: "A"              │
│ next: → [21:"B"]        │ (HashMap collision chain)
│ before: null            │ (LinkedHashMap doubly linked list)
│ after: → [21:"B"]       │ (LinkedHashMap doubly linked list)
└─────────────────────────┘

Node for key=21:
┌─────────────────────────┐
│ hash: 56789             │
│ key: 21                 │
│ value: "B"              │
│ next: null              │
│ before: → [1:"A"]       │
│ after: → [23:"C"]       │
└─────────────────────────┘
```

---

## Insertion Order vs Access Order

### 1. Insertion Order (Default)

Elements retrieved in the **same order** they were inserted.

```java
// Default: Insertion Order
Map<Integer, String> map = new LinkedHashMap<>();

map.put(1, "A");
map.put(21, "B");
map.put(23, "C");
map.put(141, "D");
map.put(25, "E");

// Iteration order: 1, 21, 23, 141, 25 (same as insertion)
for (Map.Entry<Integer, String> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " = " + entry.getValue());
}
// Output:
// 1 = A
// 21 = B
// 23 = C
// 141 = D
// 25 = E
```

**Why?** Iterates through the doubly linked list starting from `head`:
```
head → [1] → [21] → [23] → [141] → [25] → null
```

---

### 2. Access Order (LRU Cache)

**Least Recently Used (LRU)** → **Most Recently Used (MRU)**

```java
// Enable Access Order
Map<Integer, String> map = new LinkedHashMap<>(16, 0.75f, true);
//                                              capacity, loadFactor, accessOrder

map.put(1, "A");
map.put(21, "B");
map.put(23, "C");
map.put(141, "D");
map.put(25, "E");

// Initial order: 1, 21, 23, 141, 25

// Access key 23
map.get(23);  // Moves 23 to end

// New order: 1, 21, 141, 25, 23
for (Map.Entry<Integer, String> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " = " + entry.getValue());
}
// Output:
// 1 = A
// 21 = B
// 141 = D
// 25 = E
// 23 = C  👈 Moved to end (most recently used)
```

### How Access Order Works Internally

```java
// Inside LinkedHashMap.get()
public V get(Object key) {
    Node<K,V> e = getNode(hash(key), key);
    if (e != null && accessOrder) {
        moveNodeToLast(e);  // 👈 Move accessed node to end
    }
    return e.value;
}
```

**Before `get(23)`**:
```
[1] ⇄ [21] ⇄ [23] ⇄ [141] ⇄ [25]
```

**After `get(23)`**:
```
[1] ⇄ [21] ⇄ [141] ⇄ [25] ⇄ [23]
                              ↑
                         Moved to end
```

---

## LinkedHashMap Use Case: LRU Cache

```java
import java.util.LinkedHashMap;
import java.util.Map;

class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;

    public LRUCache(int capacity) {
        super(capacity, 0.75f, true);  // accessOrder = true
        this.capacity = capacity;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;  // Remove least recently used
    }
}

// Usage
LRUCache<Integer, String> cache = new LRUCache<>(3);
cache.put(1, "A");
cache.put(2, "B");
cache.put(3, "C");
// Cache: [1, 2, 3]

cache.get(1);  // Access 1
// Cache: [2, 3, 1]

cache.put(4, "D");  // Evicts 2 (least recently used)
// Cache: [3, 1, 4]
```

---

## LinkedHashMap Thread Safety

### Not Thread-Safe by Default

```java
Map<Integer, String> map = new LinkedHashMap<>();
// ❌ Not safe for concurrent access
```

### Making it Thread-Safe

```java
// Option 1: Collections.synchronizedMap (Recommended)
Map<Integer, String> syncMap = Collections.synchronizedMap(
    new LinkedHashMap<>()
);

// Usage
syncMap.put(1, "A");  // Thread-safe
syncMap.get(1);       // Thread-safe

// Option 2: Manual synchronization
Map<Integer, String> map = new LinkedHashMap<>();
synchronized(map) {
    map.put(1, "A");
    map.get(1);
}
```

### How synchronizedMap Works

```java
// Inside Collections.synchronizedMap()
public V put(K key, V value) {
    synchronized(mutex) {  // 👈 Wraps with synchronized block
        return map.put(key, value);
    }
}

public V get(Object key) {
    synchronized(mutex) {
        return map.get(key);
    }
}
```

---

## LinkedHashMap Time Complexity

| Operation | Time Complexity | Notes |
|-----------|----------------|-------|
| `put(key, value)` | O(1) average | Same as HashMap |
| `get(key)` | O(1) average | Same as HashMap |
| `remove(key)` | O(1) average | Same as HashMap |
| `containsKey(key)` | O(1) average | Same as HashMap |
| **Worst Case** | O(log n) | After treeify threshold |
| Space | O(n) | Extra space for before/after pointers |

---

## TreeMap Overview

### What is TreeMap?

- **Implements**: NavigableMap (extends SortedMap, extends Map)
- **Key Feature**: Stores entries in **sorted order**
- **Sorting**: By natural ordering or custom Comparator
- **Internal Structure**: **Red-Black Tree** (self-balancing BST)

### Map Hierarchy

```
Map (Interface)
 ├── SortedMap (Interface)
 │    └── NavigableMap (Interface)
 │         └── TreeMap (Concrete Class)
 ├── HashMap
 └── LinkedHashMap
```

---

## TreeMap Internal Structure

### Node Structure

```java
static final class Entry<K,V> implements Map.Entry<K,V> {
    K key;
    V value;
    Entry<K,V> left;    // Left child
    Entry<K,V> right;   // Right child
    Entry<K,V> parent;  // Parent node
    boolean color;      // Red or Black (for Red-Black Tree)
}
```

### Visual Structure

```
Red-Black Tree (Binary Search Tree):

         [4:"SJ"]
         /      \
    [1:"PJ"]   [5:"KJ"]
```

**Properties**:
- Left child < Parent
- Right child > Parent
- Self-balancing (Red-Black Tree)

---

## TreeMap Sorting

### 1. Natural Ordering (Default)

```java
// Default: Natural ordering (ascending for integers)
Map<Integer, String> map = new TreeMap<>();

map.put(21, "B");
map.put(5, "A");
map.put(13, "C");
map.put(11, "D");

// Iteration order: 5, 11, 13, 21 (sorted ascending)
for (Map.Entry<Integer, String> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " = " + entry.getValue());
}
// Output:
// 5 = A
// 11 = D
// 13 = C
// 21 = B
```

### 2. Custom Comparator (Descending)

```java
// Custom comparator: Descending order
Map<Integer, String> map = new TreeMap<>((k1, k2) -> k2 - k1);

map.put(21, "B");
map.put(5, "A");
map.put(13, "C");
map.put(11, "D");

// Iteration order: 21, 13, 11, 5 (sorted descending)
for (Map.Entry<Integer, String> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " = " + entry.getValue());
}
// Output:
// 21 = B
// 13 = C
// 11 = D
// 5 = A
```

---

## TreeMap Insertion Example

### Building a Red-Black Tree

```java
TreeMap<Integer, String> map = new TreeMap<>();

map.put(4, "SJ");
map.put(1, "PJ");
map.put(5, "KJ");
```

**Step-by-Step Construction**:

```
Step 1: Insert (4, "SJ")
        [4:"SJ"]
        parent: null
        left: null
        right: null

Step 2: Insert (1, "PJ")
        [4:"SJ"]
        /
    [1:"PJ"]
    parent: [4:"SJ"]

Step 3: Insert (5, "KJ")
        [4:"SJ"]
        /      \
    [1:"PJ"]  [5:"KJ"]
```

---

## SortedMap & NavigableMap Methods

### SortedMap Methods (4 methods)

```java
TreeMap<Integer, String> map = new TreeMap<>();
map.put(5, "A");
map.put(11, "B");
map.put(13, "C");
map.put(21, "D");

// Sorted order: [5, 11, 13, 21]

// 1. headMap(toKey) - Returns keys < toKey
SortedMap<Integer, String> head = map.headMap(13);
// Returns: {5=A, 11=B}  (13 is exclusive)

// 2. tailMap(fromKey) - Returns keys >= fromKey
SortedMap<Integer, String> tail = map.tailMap(13);
// Returns: {13=C, 21=D}  (13 is inclusive)

// 3. firstKey() - Returns first (smallest) key
Integer first = map.firstKey();  // 5

// 4. lastKey() - Returns last (largest) key
Integer last = map.lastKey();    // 21
```

---

### NavigableMap Methods (17 methods)

```java
TreeMap<Integer, String> map = new TreeMap<>();
map.put(1, "A");
map.put(21, "B");
map.put(23, "C");
map.put(25, "D");
map.put(141, "E");

// Sorted order: [1, 21, 23, 25, 141]
```

#### Lower (Strictly Less Than)
```java
// lowerEntry(key) - Returns entry < key
Map.Entry<Integer, String> entry = map.lowerEntry(23);
// Returns: 21=B  (21 < 23)

// lowerKey(key) - Returns key < key
Integer key = map.lowerKey(23);
// Returns: 21
```

#### Floor (Less Than or Equal)
```java
// floorEntry(key) - Returns entry <= key
Map.Entry<Integer, String> entry1 = map.floorEntry(24);
// Returns: 23=C  (23 <= 24, equal doesn't exist)

Map.Entry<Integer, String> entry2 = map.floorEntry(23);
// Returns: 23=C  (23 == 23, equal exists)

// floorKey(key) - Returns key <= key
Integer key = map.floorKey(24);
// Returns: 23
```

#### Ceiling (Greater Than or Equal)
```java
// ceilingEntry(key) - Returns entry >= key
Map.Entry<Integer, String> entry1 = map.ceilingEntry(23);
// Returns: 23=C  (23 == 23)

Map.Entry<Integer, String> entry2 = map.ceilingEntry(24);
// Returns: 25=D  (25 >= 24, equal doesn't exist)

// ceilingKey(key) - Returns key >= key
Integer key = map.ceilingKey(24);
// Returns: 25
```

#### Higher (Strictly Greater Than)
```java
// higherEntry(key) - Returns entry > key
Map.Entry<Integer, String> entry = map.higherEntry(23);
// Returns: 25=D  (25 > 23)

// higherKey(key) - Returns key > key
Integer key = map.higherKey(23);
// Returns: 25
```

#### First and Last
```java
// firstEntry() - Returns first entry
Map.Entry<Integer, String> first = map.firstEntry();
// Returns: 1=A

// lastEntry() - Returns last entry
Map.Entry<Integer, String> last = map.lastEntry();
// Returns: 141=E
```

#### Poll (Remove and Return)
```java
// pollFirstEntry() - Remove and return first entry
Map.Entry<Integer, String> removed1 = map.pollFirstEntry();
// Returns: 1=A  (removed from map)

// pollLastEntry() - Remove and return last entry
Map.Entry<Integer, String> removed2 = map.pollLastEntry();
// Returns: 141=E  (removed from map)

// Map now: [21, 23, 25]
```

#### Reverse Views
```java
// descendingMap() - Reverse order map
NavigableMap<Integer, String> reversed = map.descendingMap();
// Order: [25, 23, 21]

// descendingKeySet() - Reverse order keys
NavigableSet<Integer> reversedKeys = map.descendingKeySet();
// Order: [25, 23, 21]
```

#### Range Views
```java
// headMap(toKey, inclusive) - Keys < toKey (or <=)
NavigableMap<Integer, String> head1 = map.headMap(23, false);
// Returns: {21=B}  (exclusive)

NavigableMap<Integer, String> head2 = map.headMap(23, true);
// Returns: {21=B, 23=C}  (inclusive)

// tailMap(fromKey, inclusive) - Keys >= fromKey (or >)
NavigableMap<Integer, String> tail1 = map.tailMap(23, true);
// Returns: {23=C, 25=D}  (inclusive)

NavigableMap<Integer, String> tail2 = map.tailMap(23, false);
// Returns: {25=D}  (exclusive)

// subMap(fromKey, fromInclusive, toKey, toInclusive)
NavigableMap<Integer, String> sub = map.subMap(21, true, 25, false);
// Returns: {21=B, 23=C}  (21 inclusive, 25 exclusive)
```

---

## TreeMap Time Complexity

| Operation | Time Complexity | Reason |
|-----------|----------------|--------|
| `put(key, value)` | O(log n) | Red-Black Tree insertion |
| `get(key)` | O(log n) | Binary search in tree |
| `remove(key)` | O(log n) | Tree rebalancing |
| `containsKey(key)` | O(log n) | Binary search |
| `firstKey()` / `lastKey()` | O(log n) | Tree traversal to leftmost/rightmost |
| Space | O(n) | Tree nodes |

**Why O(log n)?**
- Binary Search Tree property
- At each step, eliminates half the tree
- Height of balanced tree = log(n)

---

## Comparison Tables

### HashMap vs LinkedHashMap vs TreeMap

| Feature | HashMap | LinkedHashMap | TreeMap |
|---------|---------|---------------|---------|
| **Order** | ❌ No order | ✅ Insertion/Access order | ✅ Sorted order |
| **Data Structure** | Array + Linked List/Tree | Array + Doubly Linked List | Red-Black Tree |
| **Performance** | ⭐⭐⭐ Fastest | ⭐⭐ Medium | ⭐ Slowest |
| **put/get** | O(1) average | O(1) average | O(log n) |
| **Null Keys** | ✅ One null key | ✅ One null key | ❌ No null keys |
| **Null Values** | ✅ Allowed | ✅ Allowed | ✅ Allowed |
| **Thread-Safe** | ❌ No | ❌ No | ❌ No |
| **Use Case** | General purpose | Order matters, LRU cache | Sorted keys needed |
| **Since** | Java 1.2 | Java 1.4 | Java 1.2 |

---

### Method Comparison

| Method Category | HashMap | LinkedHashMap | TreeMap |
|----------------|---------|---------------|---------|
| **Basic Map Methods** | ✅ All | ✅ All | ✅ All |
| **Order Maintenance** | ❌ | ✅ Insertion/Access | ✅ Sorted |
| **SortedMap Methods** | ❌ | ❌ | ✅ (4 methods) |
| **NavigableMap Methods** | ❌ | ❌ | ✅ (17 methods) |

---

## When to Use What?

### Use HashMap When:
✅ Order doesn't matter  
✅ Need **fastest** performance (O(1))  
✅ General-purpose key-value storage  
✅ Most common use case  

### Use LinkedHashMap When:
✅ Need to maintain **insertion order**  
✅ Implementing **LRU cache** (access order)  
✅ Need predictable iteration order  
✅ Slightly slower than HashMap acceptable  

### Use TreeMap When:
✅ Need **sorted keys** (ascending/descending)  
✅ Need **range queries** (headMap, tailMap, subMap)  
✅ Need **navigation methods** (lower, floor, ceiling, higher)  
✅ O(log n) performance acceptable  
✅ Natural ordering or custom comparator  

---

## Complete Examples

### LinkedHashMap: Insertion Order
```java
Map<Integer, String> map = new LinkedHashMap<>();
map.put(3, "C");
map.put(1, "A");
map.put(2, "B");

map.forEach((k, v) -> System.out.println(k + " = " + v));
// Output: 3=C, 1=A, 2=B (insertion order)
```

### LinkedHashMap: Access Order (LRU)
```java
Map<Integer, String> map = new LinkedHashMap<>(16, 0.75f, true);
map.put(1, "A");
map.put(2, "B");
map.put(3, "C");

map.get(1);  // Access 1

map.forEach((k, v) -> System.out.println(k + " = " + v));
// Output: 2=B, 3=C, 1=A (1 moved to end)
```

### TreeMap: Natural Ordering
```java
Map<Integer, String> map = new TreeMap<>();
map.put(3, "C");
map.put(1, "A");
map.put(2, "B");

map.forEach((k, v) -> System.out.println(k + " = " + v));
// Output: 1=A, 2=B, 3=C (sorted ascending)
```

### TreeMap: Custom Comparator
```java
Map<Integer, String> map = new TreeMap<>((a, b) -> b - a);
map.put(3, "C");
map.put(1, "A");
map.put(2, "B");

map.forEach((k, v) -> System.out.println(k + " = " + v));
// Output: 3=C, 2=B, 1=A (sorted descending)
```

---

## Key Takeaways

### LinkedHashMap
✅ **HashMap + Doubly Linked List**  
✅ Adds `before` and `after` pointers to each node  
✅ Two modes: **Insertion Order** (default) & **Access Order**  
✅ Perfect for **LRU Cache** implementation  
✅ O(1) average time complexity (same as HashMap)  
✅ Not thread-safe (use `Collections.synchronizedMap()`)  

### TreeMap
✅ Uses **Red-Black Tree** (self-balancing BST)  
✅ Always maintains **sorted order**  
✅ O(log n) time complexity for all operations  
✅ Supports **SortedMap** (4 methods) & **NavigableMap** (17 methods)  
✅ Natural ordering or custom **Comparator**  
✅ No null keys allowed (null values okay)  
✅ Not thread-safe  

---

## Quick Reference

```java
// HashMap - No order, O(1)
Map<Integer, String> hashMap = new HashMap<>();

// LinkedHashMap - Insertion order, O(1)
Map<Integer, String> linkedHashMap = new LinkedHashMap<>();

// LinkedHashMap - Access order (LRU), O(1)
Map<Integer, String> lruMap = new LinkedHashMap<>(16, 0.75f, true);

// TreeMap - Sorted ascending, O(log n)
Map<Integer, String> treeMap = new TreeMap<>();

// TreeMap - Sorted descending, O(log n)
Map<Integer, String> reverseTreeMap = new TreeMap<>((a, b) -> b - a);

// Thread-safe LinkedHashMap
Map<Integer, String> syncMap = Collections.synchronizedMap(
    new LinkedHashMap<>()
);
```

---

*Happy Coding! 🚀*
