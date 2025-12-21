# Java Map & HashMap Internals - Complete Notes

## Table of Contents
1. [Why Map is Not Part of Collection](#why-map-is-not-part-of-collection)
2. [Map Interface](#map-interface)
3. [HashMap Internal Structure](#hashmap-internal-structure)
4. [How put() Works](#how-put-works)
5. [How get() Works](#how-get-works)
6. [Collision Handling](#collision-handling)
7. [Load Factor & Rehashing](#load-factor--rehashing)
8. [Treeify Threshold](#treeify-threshold)
9. [HashMap Methods](#hashmap-methods)
10. [Time Complexity](#time-complexity)
11. [HashMap vs Hashtable](#hashmap-vs-hashtable)

---

## Why Map is Not Part of Collection?

### Collection Interface
```
Collection stores: [value1, value2, value3, value4]
Methods work on: Single values
```

### Map Interface
```
Map stores: {key1: value1, key2: value2, key3: value3}
Methods work on: Key-Value pairs
```

**Key Reason**: 
- Collection deals with **single values**
- Map deals with **key-value pairs**
- Requires completely different methods
- No point in Map extending Collection

---

## Map Interface

### What is Map?
- **Interface** (not part of Collection hierarchy)
- Maps **keys to values**
- **No duplicate keys** allowed (values can be duplicated)
- Duplicate key insertion → **overwrites** existing value

### Map Hierarchy
```
Map (Interface)
├── HashMap
├── LinkedHashMap
├── TreeMap
└── Hashtable
```

### Visual Representation
```
Map Structure:
┌─────┬────────┐
│ Key │ Value  │
├─────┼────────┤
│  1  │  "SJ"  │
│  2  │  "CX"  │
│  3  │  "PJ"  │
│  4  │  "MJ"  │
└─────┴────────┘

Note: Keys are unique, values can repeat
```

---

## Map Basic Methods

| Method | Description |
|--------|-------------|
| `size()` | Returns number of key-value mappings |
| `isEmpty()` | Returns true if map is empty |
| `containsKey(key)` | Returns true if key exists |
| `get(key)` | Returns value for given key |
| `put(key, value)` | Inserts or updates key-value pair |
| `remove(key)` | Removes mapping for given key |
| `putIfAbsent(key, value)` | Puts only if key is absent or value is null |
| `entrySet()` | Returns Set of Map.Entry objects |
| `keySet()` | Returns Set of all keys |
| `values()` | Returns Collection of all values |

---

## HashMap Internal Structure

### Core Components

#### 1. Node Class (Inner Class)
```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;      // Hash value
    final K key;         // Key
    V value;             // Value
    Node<K,V> next;      // Pointer to next node
}
```

#### 2. Internal Array
```java
Node<K,V>[] table;
```

### Visual Structure
```
HashMap Internal Array (Default size: 16)

Index:  [0]  [1]  [2]  [3]  ... [15]
         ↓    ↓    ↓    ↓        ↓
       Node  Node Node Node     Node
```

Each Node contains:
```
┌──────────────────────────┐
│ hash:  1234567            │
│ key:   1                  │
│ value: "SJ"               │
│ next:  null               │
└──────────────────────────┘
```

---

## HashMap Default Values

```java
// Default initial capacity
static final int DEFAULT_INITIAL_CAPACITY = 16;

// Default load factor
static final float DEFAULT_LOAD_FACTOR = 0.75f;

// Threshold for converting linked list to tree
static final int TREEIFY_THRESHOLD = 8;
```

### Creating HashMap
```java
// Default capacity (16) and load factor (0.75)
Map<Integer, String> map = new HashMap<>();

// Custom capacity
Map<Integer, String> map = new HashMap<>(32);

// Custom capacity and load factor
Map<Integer, String> map = new HashMap<>(32, 0.8f);
```

---

## How put() Works

### Algorithm Steps

```java
map.put(1, "SJ");
```

**Step 1: Calculate Hash**
```java
hash = hash(key);  // For key=1, let's say hash=1234567
```

**Step 2: Find Index**
```java
index = hash % table.length;
// 1234567 % 16 = 7 (for example)
```

**Step 3: Insert at Index**
```java
table[7] = new Node(hash=1234567, key=1, value="SJ", next=null);
```

### Visual Example

```java
Map<Integer, String> map = new HashMap<>(3); // Size 3 for simplicity

map.put(1, "SJ");
map.put(5, "PJ");
map.put(10, "KJ");
```

**Execution**:

```
Step 1: put(1, "SJ")
hash(1) = 1234567
1234567 % 3 = 1

[0] → null
[1] → [hash:1234567, key:1, value:"SJ", next:null]
[2] → null

Step 2: put(5, "PJ")
hash(5) = 984120
984120 % 3 = 2

[0] → null
[1] → [hash:1234567, key:1, value:"SJ", next:null]
[2] → [hash:984120, key:5, value:"PJ", next:null]

Step 3: put(10, "KJ") - COLLISION!
hash(10) = 515100
515100 % 3 = 1 (same as key 1!)

[0] → null
[1] → [hash:1234567, key:1, value:"SJ", next:→]
       ↓
      [hash:515100, key:10, value:"KJ", next:null]
[2] → [hash:984120, key:5, value:"PJ", next:null]
```

---

## Collision Handling

### What is Collision?
When two different keys hash to the same index.

### Solution: Chaining (Linked List)

```
Index 1 collision example:

[1] → [key:1, value:"SJ"] → [key:10, value:"KJ"] → [key:2, value:"J"] → null
       ↑                      ↑                       ↑
     First node           Second node             Third node
```

### Collision Resolution Algorithm
```java
// Pseudocode
if (index already has node) {
    if (key is same) {
        // Overwrite value
        oldValue = node.value;
        node.value = newValue;
    } else {
        // Add to end of linked list
        node.next = new Node(key, value);
    }
}
```

---

## How get() Works

### Algorithm Steps

```java
String value = map.get(5);
```

**Step 1: Calculate Hash**
```java
hash = hash(5);  // Same hash as during put()
```

**Step 2: Find Index**
```java
index = hash % table.length;
```

**Step 3: Search in Linked List**
```java
Node node = table[index];
while (node != null) {
    if (node.key.equals(5)) {
        return node.value;
    }
    node = node.next;
}
return null;  // Key not found
```

### Visual Example

```
Looking for key 5:

hash(5) = 616100
616100 % 3 = 1

Go to index 1:
[1] → [key:1] → [key:10] → [key:2] → [key:5, value:"P"]
       ↓          ↓          ↓           ↓
      Not 5      Not 5      Not 5      FOUND! Return "P"
```

---

## hashCode() and equals() Contract

### Two Critical Contracts

#### Contract 1: Equal Objects → Same Hash
```
If obj1.equals(obj2) == true
Then obj1.hashCode() == obj2.hashCode()
```

**Example**:
```java
Integer a = 5;
Integer b = 5;

a.equals(b);     // true
a.hashCode();    // 5
b.hashCode();    // 5 (MUST be same!)
```

**Why Important**: Ensures `get()` finds the same index as `put()`

#### Contract 2: Same Hash ≠ Equal Objects
```
If obj1.hashCode() == obj2.hashCode()
Does NOT mean obj1.equals(obj2) == true
```

**Example**:
```java
hash(6) = 51510
hash(8) = 51510  // Collision!

// But 6 != 8
6.equals(8);  // false
```

**Why Important**: That's why we compare both hash AND key using `equals()`

---

## Load Factor & Rehashing

### What is Load Factor?

**Default**: 0.75

**Formula**:
```
Threshold = Initial Capacity × Load Factor
          = 16 × 0.75
          = 12
```

**Meaning**: When HashMap has 12 entries, it will **resize**.

### Rehashing Process

```
Initial State (Capacity: 16):
[0] [1] [2] [3] ... [15]
 ↓   ↓   ↓   ↓      ↓
Node Node Node ...

After 12 insertions → Resize!

New State (Capacity: 32):
[0] [1] [2] [3] ... [31]
```

**Steps**:
1. Create new array (double size: 16 → 32)
2. Recalculate index for all nodes
3. Copy nodes to new positions
4. Point to new array

### Why Load Factor?

**Problem Without Rehashing**:
```
Small array (size 3) with 100 elements
→ Many collisions
→ Long linked lists
→ Slow operations (O(n))
```

**Solution With Rehashing**:
```
Array grows dynamically
→ Fewer collisions
→ Shorter chains
→ Fast operations (O(1))
```

---

## Treeify Threshold

### What is Treeify Threshold?

**Default**: 8

**Meaning**: When a single bucket's linked list has **8+ nodes**, convert to **balanced tree**.

### Why Convert to Tree?

#### Linked List Search: O(n)
```
[key:1] → [key:10] → [key:2] → [key:5] → [key:7] → ...
  ↓         ↓          ↓          ↓          ↓
Check    Check      Check      Check      Check
```

#### Balanced Tree Search: O(log n)
```
         [key:5]
        /       \
    [key:2]    [key:10]
      /           \
  [key:1]       [key:7]
```

### Tree Conversion Algorithm

```java
// Pseudocode
if (bucket.size >= TREEIFY_THRESHOLD) {
    convertToBalancedTree(bucket);
}
```

**Tree Used**: Red-Black Tree (self-balancing BST)

---

## Time Complexity

### Average Case (Amortized)

| Operation | Time Complexity |
|-----------|----------------|
| `put(key, value)` | O(1) |
| `get(key)` | O(1) |
| `remove(key)` | O(1) |
| `containsKey(key)` | O(1) |

**Why O(1)?**
- Direct index calculation via hashing
- Load factor prevents long chains
- Rehashing keeps array sparse

### Worst Case

| Scenario | Time Complexity |
|----------|----------------|
| All keys hash to same index (Linked List) | O(n) |
| All keys hash to same index (Tree after threshold) | O(log n) |

**Note**: Worst case O(n) is **rare** in practice due to:
- Good hash function
- Load factor (0.75)
- Treeify threshold (8)

---

## HashMap Methods Examples

### Complete Example

```java
Map<Integer, String> map = new HashMap<>();

// 1. put() - Insert key-value pairs
map.put(null, "test");    // null key allowed
map.put(0, null);          // null value allowed
map.put(1, "A");
map.put(2, "B");

// 2. putIfAbsent() - Insert only if absent or null
map.putIfAbsent(null, "new");  // Won't insert (key exists, value != null)
map.putIfAbsent(0, "Zero");     // Will insert (value was null)
map.putIfAbsent(3, "C");        // Will insert (key absent)

// Result: {null=test, 0=Zero, 1=A, 2=B, 3=C}

// 3. get() - Retrieve value
String value = map.get(1);  // "A"

// 4. getOrDefault() - Get with default value
String value2 = map.get(9);                  // null (key doesn't exist)
String value3 = map.getOrDefault(9, "N/A");  // "N/A"

// 5. containsKey() - Check if key exists
boolean exists = map.containsKey(3);  // true
boolean exists2 = map.containsKey(9); // false

// 6. size() - Number of entries
int size = map.size();  // 5

// 7. isEmpty() - Check if empty
boolean empty = map.isEmpty();  // false

// 8. remove() - Remove entry
String removed = map.remove(null);  // Returns "test"

// 9. entrySet() - Iterate over entries
for (Map.Entry<Integer, String> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " = " + entry.getValue());
}
// Output:
// 0 = Zero
// 1 = A
// 2 = B
// 3 = C

// 10. keySet() - Get all keys
Set<Integer> keys = map.keySet();
for (Integer key : keys) {
    System.out.println(key);
}
// Output: 0, 1, 2, 3

// 11. values() - Get all values
Collection<String> values = map.values();
for (String val : values) {
    System.out.println(val);
}
// Output: Zero, A, B, C
```

---

## HashMap vs Hashtable

### Comparison Table

| Feature | HashMap | Hashtable |
|---------|---------|-----------|
| **Thread-Safe** | ❌ No | ✅ Yes (Synchronized) |
| **Null Key** | ✅ Allowed (one) | ❌ Not allowed |
| **Null Values** | ✅ Allowed | ❌ Not allowed |
| **Performance** | Faster | Slower (locking overhead) |
| **Since** | Java 1.2 | Java 1.0 (Legacy) |
| **Iteration** | Fail-fast | Enumeration |
| **Recommended** | ✅ Yes | ❌ Legacy |

### Code Examples

#### HashMap (Not Thread-Safe)
```java
Map<Integer, String> hashMap = new HashMap<>();
hashMap.put(null, "test");   // ✅ Allowed
hashMap.put(1, null);         // ✅ Allowed
// Not thread-safe - use in single-threaded environment
```

#### Hashtable (Thread-Safe)
```java
Map<Integer, String> hashtable = new Hashtable<>();
hashtable.put(null, "test");  // ❌ NullPointerException
hashtable.put(1, null);        // ❌ NullPointerException
// Thread-safe but slower
```

#### ConcurrentHashMap (Modern Thread-Safe)
```java
Map<Integer, String> concurrentMap = new ConcurrentHashMap<>();
concurrentMap.put(1, "A");    // ✅ Thread-safe
concurrentMap.put(null, "B"); // ❌ NullPointerException
concurrentMap.put(2, null);   // ❌ NullPointerException
// Better performance than Hashtable
```

---

## Thread-Safe Alternatives

| Non-Thread-Safe | Thread-Safe Alternative |
|-----------------|------------------------|
| HashMap | ConcurrentHashMap (Recommended) |
| HashMap | Hashtable (Legacy) |
| HashMap | Collections.synchronizedMap(new HashMap()) |

### Example: Making HashMap Thread-Safe
```java
// Option 1: ConcurrentHashMap (Best)
Map<Integer, String> map1 = new ConcurrentHashMap<>();

// Option 2: Hashtable (Legacy)
Map<Integer, String> map2 = new Hashtable<>();

// Option 3: Synchronized wrapper
Map<Integer, String> map3 = Collections.synchronizedMap(new HashMap<>());
```

---

## Key Takeaways

### HashMap Internal Structure
✅ Uses **array of Node objects** (default size: 16)  
✅ Each Node has: hash, key, value, next  
✅ Collision handled by **chaining** (linked list)  
✅ Linked list converts to **tree** after 8 nodes  

### How Operations Work
✅ **put()**: hash(key) → index → insert/update  
✅ **get()**: hash(key) → index → search in chain  
✅ **Both hash and equals() used** for comparison  

### Performance Optimization
✅ **Load Factor (0.75)**: Triggers resize at 75% capacity  
✅ **Rehashing**: Doubles capacity, redistributes nodes  
✅ **Treeify (8)**: Converts long chains to balanced tree  

### Time Complexity
✅ **Average**: O(1) for put/get/remove  
✅ **Worst**: O(log n) with tree conversion  
✅ **Worst (no tree)**: O(n) with long chains  

### HashMap Characteristics
✅ **Not thread-safe** (use ConcurrentHashMap)  
✅ **Allows null key** (one) and null values  
✅ **No order maintained** (use LinkedHashMap)  
✅ **Keys must be unique**  

---

## Interview Questions

### Q1: How does HashMap work internally?
**Answer**: HashMap uses an array of Node objects. Each Node contains hash, key, value, and next pointer. When we put(key, value):
1. Calculate hash of key
2. Find index: hash % array.length
3. If no collision, insert directly
4. If collision, append to linked list
5. If list length ≥ 8, convert to balanced tree

### Q2: What is the time complexity of HashMap?
**Answer**: 
- **Average**: O(1) for get/put/remove
- **Worst**: O(log n) due to tree conversion after 8 collisions
- Without tree conversion, worst case would be O(n)

### Q3: What is load factor and why is it important?
**Answer**: Load factor (default 0.75) determines when to resize the HashMap. When size reaches capacity × 0.75, HashMap doubles its size and rehashes all entries. This prevents long collision chains and maintains O(1) performance.

### Q4: Explain hashCode() and equals() contract
**Answer**: 
- If obj1.equals(obj2) is true, their hashCode() must be equal
- If two objects have same hashCode(), they may or may not be equal
- This ensures get() can find keys that were inserted with put()

### Q5: HashMap vs Hashtable?
**Answer**: 
- HashMap: Not synchronized, allows null, faster, Java 1.2
- Hashtable: Synchronized, no nulls, slower, legacy (Java 1.0)
- Use ConcurrentHashMap for thread-safe modern alternative

---

## Practice Tips

1. ✅ Draw the internal array structure on paper
2. ✅ Trace put() and get() operations step-by-step
3. ✅ Understand collision handling (linked list → tree)
4. ✅ Calculate index: hash % capacity
5. ✅ Remember: Load factor triggers resize
6. ✅ Practice explaining to others verbally

---

*Happy Coding! 🚀*
