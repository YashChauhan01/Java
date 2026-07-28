# C++ STL Crash Course: Containers, Iterators & Core Algorithms

## Executive Summary
This lesson introduces the **Standard Template Library (STL)** — the pre-built collection of containers, algorithms, and iterators that lets you skip writing data structures from scratch in C++. It walks through the full container lineup used in DSA (pair, vector, list, deque, stack, queue, priority_queue, set/multiset/unordered_set, map/multimap/unordered_map), how iterators traverse memory, and the handful of algorithms (sort with custom comparators, `__builtin_popcount`, `next_permutation`, `max_element`) that appear constantly in competitive programming.

## Core Concepts

### STL (Standard Template Library)
- **The Layman's Definition:** STL is a ready-made toolbox of containers (ways to store data), algorithms (ways to process data), functions, and iterators (ways to move through data), so you don't rewrite a stack or a sorting routine every time you need one.
- **How it Works / The Logic:** Instead of `#include`-ing individual libraries (`math.h`, `string.h`, etc.) one by one, competitive programmers typically include `#include <bits/stdc++.h>`, a bundle header that pulls in essentially all standard libraries at once. Pairing this with `using namespace std;` avoids having to prefix every call with `std::` (e.g., `std::cin` becomes just `cin`).
- **Example:** Without `using namespace std;`, printing a variable requires `std::cout << a;`. With it, you simply write `cout << a;`.

STL is organized into four pillars:
1. **Algorithms** – e.g., `sort`, `next_permutation`
2. **Containers** – e.g., `vector`, `map`, `stack`
3. **Functions** – helper utilities
4. **Iterators** – pointers that traverse containers

---

### Pair
- **The Layman's Definition:** A `pair` is a container from the `<utility>` library that bundles exactly two values together, which may be of different data types.
- **How it Works / The Logic:** Declare with `pair<int, int> p = {1, 3};`. Access the first value with `p.first` and the second with `p.second`. To store more than two values, nest pairs inside each other — the second slot of an outer pair can itself be a pair.
- **Example:** For `pair<int, pair<int,int>> p = {1, {3, 4}};`:
  - `p.first` → `1`
  - `p.second.first` → `3`
  - `p.second.second` → `4`

You can also build arrays of pairs: `pair<int,int> arr[3];`, where `arr[1].second` accesses the second element of the pair at index 1.

---

### Vector
- **The Layman's Definition:** A **vector** is a dynamic (resizable) array — unlike a plain C++ array, its size can grow after declaration.
- **How it Works / The Logic:**
  - Declare: `vector<int> v;`
  - Add elements: `v.push_back(1);` or `v.emplace_back(2);` — both append to the end; `emplace_back` is generally faster since it constructs the element in place rather than creating a temporary copy.
  - Pre-size with a default value: `vector<int> v(5, 100);` creates 5 elements, all `100`. `vector<int> v(5);` creates 5 elements initialized to `0` (or garbage, compiler-dependent).
  - Copy: `vector<int> v2(v1);` creates an independent copy, not a shared reference.
  - Access: `v[i]`, same syntax as an array.
  - **Iterators**: `vector<int>::iterator it = v.begin();` — `it` points to the *memory address* of the first element, not the value itself. Dereference with `*it` to get the value. `it++` moves the iterator to the next memory location.
  - `v.end()` points to the memory location *right after* the last element (not the last element itself).
  - `rbegin()`/`rend()` are reverse iterators (rarely used in practice, but useful to recognize).
  - Shortcut for iteration: `for (auto it : v)` — `auto` lets the compiler infer the data type automatically instead of you spelling it out.
  - **Erase**: `v.erase(v.begin() + 1)` deletes a single element at that position. `v.erase(v.begin() + 1, v.begin() + 4)` deletes a range — start is inclusive, end is exclusive.
  - **Insert**: `v.insert(v.begin(), 300)` inserts a single value at a position. `v.insert(v.begin() + 1, 2, 5)` inserts two copies of `5` starting at position 1. You can also insert an entire other vector's contents via `v.insert(pos, otherVector.begin(), otherVector.end())`.
  - Other utility functions: `v.size()`, `v.pop_back()` (removes last element), `v.swap(v2)`, `v.clear()` (empties the vector), `v.empty()` (returns `true`/`false`).
- **Example:** `vector<int> v; v.push_back(1); v.emplace_back(2);` → `v` now holds `{1, 2}`.

---

### List
- **The Layman's Definition:** A **list** behaves like a vector but additionally supports fast operations at the *front*, not just the back.
- **How it Works / The Logic:** Internally, a `list` is a **doubly linked list**, while a `vector` is backed by a contiguous (effectively singly-traversable) array. This makes `push_front()` and `emplace_front()` cheap for a list, whereas inserting at the front of a vector is expensive (it must shift every subsequent element). All other operations (`begin`, `end`, `size`, `clear`, `empty`, etc.) mirror the vector's.
- **Example:** `list<int> l; l.push_back(4); l.push_front(5);` → `l` holds `{5, 4}`.

---

### Deque
- **The Layman's Definition:** A **double-ended queue** — supports fast push/pop from both the front and back.
- **How it Works / The Logic:** Declared and used the same way as `list`/`vector`, with `push_back`, `push_front`, `pop_back`, `pop_front`, `front()`, and `back()` all available directly.
- **Example:** *(Synthesized Example)* `deque<int> dq; dq.push_back(1); dq.push_front(2);` → `dq` holds `{2, 1}`.

---

### Stack (LIFO)
- **The Layman's Definition:** A **stack** follows **LIFO** (Last In, First Out) — the most recently added element is the first one removed.
- **How it Works / The Logic:** Only three core operations exist: `push()` (add to top), `pop()` (remove the top element, without returning it), and `top()` (peek at the top element without removing it). Random/index-based access is not allowed. All operations run in constant time, O(1).
- **Example:** Pushing `1, 2, 3, 5` in order means `stack.top()` returns `5`. After `stack.pop()`, `stack.top()` returns `3`.

---

### Queue (FIFO)
- **The Layman's Definition:** A **queue** follows **FIFO** (First In, First Out) — like a ticket line, the first person to join is the first one served.
- **How it Works / The Logic:** `push()` adds to the back; `pop()` removes from the front; `front()` peeks the front element; `back()` peeks/modifies the back element. All operations are O(1).
- **Example:** Pushing `1, 2, 4` then modifying the back (`q.back() += 5` → back becomes `9`): `q.front()` returns `1`; after `q.pop()`, `q.front()` returns `2`.

---

### Priority Queue (Heap)
- **The Layman's Definition:** A **priority_queue** always keeps the "highest priority" (by default, the largest) element instantly accessible at the top — it is not stored in simple linear order internally, but backed by a tree-based structure called a **heap**.
- **How it Works / The Logic:**
  - Max-heap (default): `priority_queue<int> pq;` — largest element always at `pq.top()`.
  - Min-heap: `priority_queue<int, vector<int>, greater<int>> pq;` — smallest element always at `pq.top()`.
  - `push()` and `pop()` run in **O(log n)**; `top()` runs in **O(1)**.
- **Example:** Pushing `5, 2, 8, 10` into a default (max) priority queue → `pq.top()` is `10`. Pushing `5, 2, 8, 10` into a min priority queue → `pq.top()` is `2`.

| Type | Declaration | Top Element |
|---|---|---|
| Max-Heap (default) | `priority_queue<int> pq;` | Largest value |
| Min-Heap | `priority_queue<int, vector<int>, greater<int>> pq;` | Smallest value |

---

### Set, Multiset, and Unordered Set
- **The Layman's Definition:** A **set** stores elements in **sorted order** with **no duplicates**. A **multiset** is the same but allows duplicates. An **unordered_set** stores unique elements but in no guaranteed order.
- **How it Works / The Logic:**
  - Internally backed by a tree (for `set`/`multiset`) rather than a simple linear list, which is why insert/erase/find run in **O(log n)**.
  - `insert()`/`emplace()` add elements.
  - `find(x)` returns an iterator pointing to `x` if present, or `.end()` if absent.
  - `erase(x)` removes element `x` (in a `multiset`, removing by value deletes **all** occurrences — to remove just one occurrence, `erase(set.find(x))` using the iterator instead).
  - `count(x)` returns `1` or `0` for a `set` (existence check); for a `multiset`, it returns the number of occurrences.
  - `unordered_set` typically offers O(1) average-case operations but degrades to O(n) worst case (a rare occurrence); it also does **not** support `lower_bound`/`upper_bound`, unlike ordered `set`.
- **Example:** Inserting `1, 2, 2, 4, 3` into a `set` results in stored order `{1, 2, 3, 4}` — the duplicate `2` is discarded. The same insertions into a `multiset` result in `{1, 2, 2, 3, 4}`.

| Container | Sorted? | Duplicates Allowed? | Typical Complexity |
|---|---|---|---|
| `set` | Yes | No | O(log n) |
| `multiset` | Yes | Yes | O(log n) |
| `unordered_set` | No | No | O(1) average |

---

### Map, Multimap, and Unordered Map
- **The Layman's Definition:** A **map** stores **key-value pairs** where every key is unique, sorted by key — analogous to looking someone up by a unique roll number to find their name.
- **How it Works / The Logic:**
  - Declare: `map<int, string> m;` (key type, then value type).
  - Insert via assignment `m[1] = "two"`, via `m.emplace(3, "one")`, or via `m.insert({2, "four"})`.
  - Traversal (e.g., with `for (auto it : m)`) yields pairs in ascending order of **key**; access with `it.first` (key) and `it.second` (value).
  - Accessing a non-existent key (e.g., `m[5]`) returns a default/zero value rather than an error.
  - `find(key)` returns an iterator to that key-value pair, or `.end()` if not found.
  - **Multimap** allows duplicate keys, still sorted.
  - **Unordered_map** allows only unique keys but stores them unsorted; it trades the O(log n) of `map` for O(1) average-case lookups (with rare O(n) worst case).
- **Example:** `map<int,int> m; m[1] = 2; m.emplace(3, 1); m.insert({2, 4});` stores (in sorted-by-key order) `{1:2, 2:4, 3:1}`.

| Container | Unique Keys? | Sorted by Key? | Typical Complexity |
|---|---|---|---|
| `map` | Yes | Yes | O(log n) |
| `multimap` | No | Yes | O(log n) |
| `unordered_map` | Yes | No | O(1) average |

---

### `sort` with Custom Comparators
- **The Layman's Definition:** The STL `sort` algorithm sorts a range in one line instead of hand-writing bubble sort or selection sort, and can be customized to sort by any rule you define.
- **How it Works / The Logic:**
  - Basic ascending sort: `sort(a, a + n);` — the first argument is the starting iterator, the second is the position right after the last element to sort (exclusive), following the same "start inclusive, end exclusive" rule seen in `erase`.
  - Descending sort: `sort(a, a + n, greater<int>());` uses the built-in `greater<int>()` **comparator**, a function that decides ordering.
  - Custom sort: pass a self-written boolean function, e.g., sorting pairs by ascending second element, and — when tied — by descending first element:

```cpp
bool comparator(pair<int,int> p1, pair<int,int> p2) {
    if (p1.second != p2.second)
        return p1.second < p2.second;   // ascending by second
    return p1.first > p2.first;         // descending by first, if tied
}
sort(arr, arr + n, comparator);
```
  - The comparator must return `true` if `p1` should come before `p2` (i.e., they are already "in order"), and `false` if they should be swapped.
- **Example:** For pairs `{1,2}, {2,1}, {4,1}`, sorting by ascending second element (ties broken by descending first) produces `{4,1}, {2,1}, {1,2}`.

---

### `__builtin_popcount`
- **The Layman's Definition:** A built-in function that counts how many bits are set to `1` in a number's binary representation.
- **How it Works / The Logic:** `__builtin_popcount(x)` works for `int`; for `long long` values, use `__builtin_popcountll(x)` instead, since a plain `int` version can't handle the larger range.
- **Example:** `__builtin_popcount(7)` → `7` is `111` in binary → returns `3`. `__builtin_popcount(6)` → `6` is `110` in binary → returns `2`.

---

### `next_permutation`
- **The Layman's Definition:** Given a sequence, this function rearranges it into the *next* lexicographically greater arrangement — useful for generating all permutations of a set.
- **How it Works / The Logic:** Call `next_permutation(s.begin(), s.end())` in a loop. It returns `false` once there is no next permutation (i.e., the sequence is in fully descending order), which is the natural loop-termination condition. **Critically, to print all permutations, the sequence must start already sorted in ascending order** — otherwise some permutations that come "before" the starting arrangement will be skipped.
- **Example:** Starting from `"123"`, repeated calls to `next_permutation` walk through `123 → 132 → 213 → 231 → 312 → 321`, after which the function returns `false` and the loop ends.

```cpp
sort(s.begin(), s.end()); // must start sorted
do {
    cout << s << endl;
} while (next_permutation(s.begin(), s.end()));
```

---

### `max_element` / `min_element`
- **The Layman's Definition:** Utility functions that find the largest or smallest value in a range without writing a manual loop.
- **How it Works / The Logic:** `max_element(a, a + n)` returns an **iterator** pointing to the maximum value's memory address; dereference with `*` to get the actual value. `min_element` works identically for the minimum.
- **Example:** For array `{1, 8, 5, 6}`, `*max_element(a, a + 4)` returns `8`.

---

## Key Takeaways & Quick Reference
- **STL = Standard Template Library**: pre-built containers, algorithms, functions, and iterators to avoid rewriting common data structures.
- `emplace_back()` is generally preferred over `push_back()` for efficiency (avoids an extra copy).
- Iterators point to **memory addresses**; dereference with `*` to get the value. `end()` always points *just past* the last element, never to it.
- In `erase`/`sort`/range operations, the **start is inclusive and the end is exclusive** — a consistent STL-wide convention.
- **Stack = LIFO**, **Queue = FIFO**, both with O(1) operations.
- **priority_queue** is a max-heap by default; use `greater<int>` for a min-heap. Push/pop are O(log n).
- **set/map** = sorted + unique keys, O(log n) operations. **multiset/multimap** allow duplicates. **unordered_** versions trade sorted order for average O(1) speed but lose `lower_bound`/`upper_bound` support.
- Custom sorting requires a **comparator** function returning `true` when two elements are already in the desired relative order.
- `next_permutation` requires the sequence to start **sorted ascending** to enumerate every permutation.

## Glossary of Terms
- **STL**: Standard Template Library — C++'s built-in library of containers, algorithms, and iterators.
- **Iterator**: A pointer-like object that references a position (memory address) within a container.
- **LIFO**: Last In, First Out — the ordering principle of a stack.
- **FIFO**: First In, First Out — the ordering principle of a queue.
- **Comparator**: A boolean function that defines a custom ordering rule for sorting.
- **Heap**: A tree-based data structure underlying `priority_queue` that keeps the max (or min) element quickly accessible.
- **O(1) / O(log n) / O(n)**: Big-O time complexity notations describing how an operation's cost scales with input size (constant, logarithmic, linear).
