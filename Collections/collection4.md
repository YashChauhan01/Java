# Java HashMap Internals: A Deep Dive for Interview Preparation

## TL;DR
This lesson covers the `Map` interface and provides an in-depth look at how `HashMap` works internally — the array-of-nodes data structure, how `put()` and `get()` actually compute indices and store data, and the performance-tuning mechanisms (**load factor**, **rehashing**, **treeification**) that keep operations fast. It closes with the `hashCode()`/`equals()` contract and a comparison between `HashMap` and `Hashtable` — all frequently asked interview topics.

## Map Basics vs. Collections

**Why `Map` is not a child of `Collection`:** Every type under the `Collection` interface (`List`, `Set`, `Queue`, etc.) deals with a **single stream of values** — value one, value two, value three — and all of `Collection`'s methods are designed around manipulating that flat list of values. `Map`, however, deals with **key-value pairs**, requiring a fundamentally different set of operations (`get(key)`, `put(key, value)`, `containsKey()`, etc.) that don't make sense for a plain value collection. Since the functionality doesn't overlap, `Map` was deliberately kept as a separate interface rather than folded into the `Collection` hierarchy.

**Basic properties of `Map`:**
- It's an **interface**, with concrete implementations including `HashMap`, `Hashtable`, `LinkedHashMap`, and `TreeMap`.
- Stores data as **key-value pairs**.
- **Keys must be unique** — no duplicate keys are allowed. Inserting a duplicate key **overwrites** the existing value for that key.
- **Values can be duplicated** freely across different keys.

## HashMap Internal Architecture (Deep Dive)

### The Underlying Data Structure
Internally, `HashMap` is backed by an **array of nodes**, where each node implements the `Map.Entry<K, V>` sub-interface (Java's actual implementation class is called `Node<K, V>`).

```java
// Conceptual structure
class Node<K, V> {
    int hash;
    K key;
    V value;
    Node<K, V> next; // pointer to the next node (for collision chaining)
}

Node<K, V>[] table; // the underlying array — default size 16
```

- **Default initial capacity**: 16 (indices 0–15) — used automatically if no size is specified during construction.
- You can also explicitly specify an initial capacity: `new HashMap<>(3)`.

### How `put(key, value)` Works — Step by Step
1. **Compute the hash** of the key using a hashing algorithm (via `hashCode()`).
2. **Modulo the hash by the table size** to compute an array index: `index = hash % table.length`.
3. **Check the target index:**
   - If it's empty, create a new `Node` (with `hash`, `key`, `value`, `next = null`) and store it there.
   - If a node already exists at that index (a **collision**), check whether the existing key matches the new key:
     - If the key **matches**, overwrite the existing value.
     - If the key **doesn't match**, append a new node to the end of the chain via the `next` pointer (forming a linked list at that index).

**Example (table size = 3, for simplicity):**
```java
Map<Integer, String> map = new HashMap<>(3);
map.put(1, "SJ");
// hash(1) → some value, e.g., 1234567
// 1234567 % 3 → index 1
// table[1] = Node(hash=1234567, key=1, value="SJ", next=null)

map.put(5, "PJ");
// hash(5) → some value, e.g., 984120
// 984120 % 3 → index 2
// table[2] = Node(hash=984120, key=5, value="PJ", next=null)

map.put(10, "KJ");
// hash(10) → some value, e.g., 515100
// 515100 % 3 → index 1 (collision! key=1 already occupies index 1)
// key 10 ≠ key 1, so append: table[1] → Node(key=1,...) → Node(hash=515100, key=10, value="KJ", next=null)
```

### How `get(key)` Works — Step by Step
1. **Compute the hash** of the key being searched for — using the exact same hashing algorithm, so it always produces the **same hash** for the same key.
2. **Modulo by table size** to find the target index.
3. **Traverse the chain** (linked list or tree) at that index, comparing each node's key with the target key using `equals()`.
4. **Return the value** of the first node whose key matches; if the chain is exhausted with no match, return `null`.

**Example:**
```java
map.get(5);
// hash(5) → 984120 (same as during put)
// 984120 % 3 → index 2
// Traverse table[2]: key == 5? Yes → return "PJ"
```

## Collision Resolution & Performance Tuning

### Collisions
A **collision** occurs when two different keys hash to the **same array index**. `HashMap` resolves this by **chaining** — storing multiple nodes at the same index as a **linked list**, connected via each node's `next` pointer. During insertion, the map checks each node in the chain to see if the key already exists (to overwrite) before appending a new node at the end.

### Load Factor & Rehashing
| Property | Default Value | Purpose |
|---|---|---|
| **Initial Capacity** | 16 | The starting size of the internal array |
| **Load Factor** | 0.75 | The fraction of capacity that triggers a resize |
| **Threshold** | `capacity × loadFactor` = 12 (for size 16) | The number of entries at which rehashing occurs |

**How it works:** Once the number of key-value mappings in the table reaches the threshold (e.g., 12 out of 16), inserting the **next** element (the 13th) triggers **rehashing**:
- The table's capacity **doubles** (16 → 32 → 64 → 128, always a power of two).
- Every existing entry is **recomputed and redistributed** into the new, larger table (since the modulo calculation changes with the new table size).

**Why this matters:** A smaller table with many entries forces longer collision chains, which slows down search/insert/delete operations (since each requires traversing the chain). By growing the table proactively, the map keeps collision chains short, preserving fast average-case performance.

### Treeify Threshold
Even with a properly sized table (thanks to the load factor), it's still theoretically possible for many keys to hash to the **same index** — creating one very long linked list ("bucket") even though other parts of the table remain sparse.

**Solution — Treeification:**
- Once a single bucket's linked-list chain reaches a length of **8** (the **treeify threshold**), `HashMap` converts that bucket from a **linked list** into a **balanced binary search tree** (specifically a **Red-Black Tree** internally).
- In a balanced BST, searching only requires traversing **one branch** (left or right) at each step, rather than checking every node sequentially.

**Example (Synthesized Example):** Picture a filing cabinet drawer (an index/bucket) so overstuffed with folders (nodes) that flipping through them one by one to find a name takes forever. Once that drawer holds more than 8 folders, the system reorganizes them into an alphabetized binary tree structure instead — so finding a folder now takes a handful of quick left/right decisions rather than checking every single folder.

## The `hashCode()` and `equals()` Contract
There are two essential rules governing the relationship between these two methods:

1. **If two objects are equal (`object1.equals(object2)` is `true`), their hash codes must also be equal.** This guarantees that calling `hashCode()` on the same key value will *always* produce the same hash — which is essential, since `put()` and `get()` must compute the identical index for the identical key every time.
2. **If two objects have the same hash code, it does NOT mean the objects are equal.** Different keys can coincidentally produce the same hash (a collision). This is precisely why each node also stores the actual `key` — during lookup, `HashMap` always confirms a match using **both** the hash *and* an `equals()` comparison on the actual key, never relying on the hash alone.

## Time Complexity Breakdown

| Scenario | Time Complexity | Reasoning |
|---|---|---|
| **Average case** (get, put, remove) | **O(1)** | Well-distributed hashing means each bucket holds very few (often 0–1) entries |
| **Worst case — before treeification existed conceptually** | O(N) | If all keys collide into one bucket, it degrades into a single linked list requiring full traversal |
| **Worst case — with treeification (actual `HashMap` behavior)** | **O(log N)** | Once a bucket's chain exceeds the treeify threshold (8), it converts to a balanced Red-Black Tree, where lookups only traverse one branch per step instead of the whole chain |

**Key interview point:** `HashMap`'s true worst-case complexity is **O(log N)**, not O(N) — because once a bucket's linked list grows past the treeify threshold, it is automatically converted into a balanced binary search tree, bounding the worst-case search time logarithmically rather than linearly.

## HashMap vs. Hashtable

| Feature | HashMap | Hashtable |
|---|---|---|
| **Thread Safety** | Not thread-safe | Thread-safe (synchronized) |
| **Null Keys** | Allows one `null` key | Does **not** allow `null` keys |
| **Null Values** | Allows `null` values | Does **not** allow `null` values |
| **Order Guarantee** | Does not maintain insertion order | Does not maintain insertion order |

**Note:** For a thread-safe alternative to `Hashtable` with generally better performance, `ConcurrentHashMap` is the modern recommended choice — it is also thread-safe, unlike plain `HashMap`.

## Key Methods Cheatsheet
- **`size()`** — returns the number of key-value mappings currently stored.
- **`isEmpty()`** — returns `true` if the map has zero mappings.
- **`containsKey(key)`** — returns `true` if the specified key exists in the map.
- **`get(key)`** — returns the value mapped to the given key (or `null` if not found).
- **`put(key, value)`** — inserts a new mapping, or overwrites the value if the key already exists.
- **`putIfAbsent(key, value)`** — inserts the value **only if** the key doesn't currently exist, **or** if the key exists but its current value is `null`; otherwise, it leaves the existing value untouched.
- **`getOrDefault(key, defaultValue)`** — returns the value for the key if present; otherwise returns the specified default value instead of `null`.
- **`remove(key)`** — removes the mapping for the given key and returns the removed value.
- **`entrySet()`** — returns a `Set` view of all key-value pairs (each as a `Map.Entry`), useful for iterating and accessing both `getKey()` and `getValue()`.
- **`keySet()`** — returns a `Set` view containing only the keys in the map.
- **`values()`** — returns a `Collection` view containing only the values in the map.
