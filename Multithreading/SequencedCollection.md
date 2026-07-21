# Java 21's Sequenced Collections: SequencedCollection, SequencedSet & SequencedMap Explained

## Executive Summary
This lesson explains the three new interfaces introduced in Java 21 — **SequencedCollection**, **SequencedSet**, and **SequencedMap** — which unify how ordered collections handle first/last element access, manipulation, and reversal. Rather than memorizing which collection extends what, the lesson teaches the underlying *criteria* so the entire hierarchy becomes self-evident and easy to reconstruct from first principles.

## Core Concepts

### The Pre-Java 21 Problem (Why These Interfaces Were Needed)
**The Layman's Definition:** Before Java 21, every ordered collection type (`List`, `Deque`, `LinkedHashSet`, `TreeSet`, `LinkedHashMap`, `TreeMap`) had its **own unique set of method names** for doing the exact same conceptual operations — get the first/last element, add/remove at either end, or view the collection in reverse.

**How it Works / The Logic:** There was no shared interface enforcing consistency, so developers had to remember a different API per collection type for identical intent:

| Collection | Get First/Last | Add First/Last | Remove First/Last | Reverse |
|---|---|---|---|---|
| `List` | `.get(0)`, `.get(size-1)` | `.add(0, x)`, `.add(x)` | manual removal | `Collections.reverse(list)` |
| `Deque` | `getFirst()`, `getLast()` | `addFirst()`, `addLast()` | `removeFirst()`, `removeLast()` | no built-in method |
| `LinkedHashSet` | manual iteration | not exposed (only `add()` at end) | manual iteration | not exposed |
| `TreeSet`/Sorted Set | `.first()`, `.last()` | N/A — position is sort-determined | `.pollFirst()`, `.pollLast()` | `.descendingIterator()` |
| `LinkedHashMap` | manual iteration | not exposed | manual iteration | not exposed |
| `TreeMap`/Sorted Map | `.firstKey()`, `.lastKey()` | N/A — position is sort-determined | `.pollFirstEntry()`, `.pollLastEntry()` | `.descendingMap()` |

This inconsistency made code harder to maintain and forced engineers to memorize collection-specific quirks. Java 21 fixes this by introducing **one common interface** with standardized method names (`getFirst()`, `getLast()`, `addFirst()`, `addLast()`, `removeFirst()`, `removeLast()`, `reversed()`) that every qualifying collection now implements identically.

**Example:**
```java
// Before Java 21 — every type needs different syntax
list.get(0);              // List
deque.getFirst();         // Deque
// LinkedHashSet had no first/last access at all without manual iteration

// After Java 21 — same method names everywhere
list.getFirst();
deque.getFirst();
linkedHashSet.getFirst();
```

---

### The Three Qualifying Conditions ("What Makes a Collection Sequenced")
**The Layman's Definition:** A collection can only be classified as "sequenced" — and thus extend one of the new interfaces — if it satisfies **three specific conditions**. These three rules explain every placement decision in the new hierarchy.

**How it Works / The Logic:**

1. **Predictable Iteration** — Elements are returned in a consistent, well-defined order every time you iterate, whether that order is **insertion order** (e.g., `List`) or **sorted order** (e.g., `TreeSet`). If the order is unpredictable or unspecified (like `HashSet` or `HashMap`), this fails immediately.
2. **Access/Manipulation of First and Last Elements** — The collection must support getting, adding, and removing elements specifically at the first and last positions.
3. **Reversible View** — It must be possible to obtain a **reversed view** of the collection (not a new copy — changes to the view reflect in the underlying collection) without creating a separate reversed structure.

**Only if all three conditions are met** can a collection type be classified as "sequenced" and placed under `SequencedCollection`, `SequencedSet`, or `SequencedMap`.

**Example (Synthesized Example):** Think of it like a checklist for a librarian deciding whether a shelf of books qualifies as "sequenced": Can you predict the order books sit in every time you scan the shelf? Can you grab the first and last book directly? Can you view the shelf back-to-front without physically rearranging the books? Only shelves passing all three checks get the "sequenced" label.

---

### Evaluating Each Collection Type Against the Three Conditions
**The Layman's Definition:** Applying the three-condition checklist to each existing Java collection type explains exactly why some collections were included in the new hierarchy and others were deliberately left out.

**How it Works / The Logic (per type):**

| Collection Type | Predictable Iteration? | First/Last Access & Manipulation? | Reversible View? | Included as Sequenced? |
|---|---|---|---|---|
| **List** | Yes (insertion order) | Yes — `get(0)`, `get(size-1)`, `add()`, `remove()` | Yes — `Collections.reverse()` | ✅ Yes |
| **Deque** | Yes (insertion order) | Yes — `getFirst/Last`, `addFirst/Last`, `removeFirst/Last` | Yes | ✅ Yes |
| **Queue** | Yes (insertion/FIFO order) | Partial — can `peek()`/`poll()` from front, but **cannot** access or remove the last element, and cannot add to the front (FIFO only) | Not supported | ❌ No |
| **PriorityQueue** | **No** — internally a heap; only guarantees the head is min/max, rest is unordered | N/A (order not maintained) | Not supported | ❌ No |
| **HashSet** | **No** — no insertion or sorted order guaranteed | N/A | Not supported | ❌ No |
| **LinkedHashSet** | Yes (insertion order, via internal doubly linked list) | Yes (achievable via manual iteration; structure supports it) | Yes (achievable via doubly linked list traversal) | ✅ Yes (under `SequencedSet`) |
| **TreeSet (SortedSet)** | Yes (sorted order) | Yes — `.first()`, `.last()`, `.pollFirst()`, `.pollLast()` (note: `addFirst`/`addLast` don't apply — position is sort-determined) | Yes — `.descendingIterator()` | ✅ Yes (under `SequencedSet`) |
| **HashMap** | **No** — no order maintained | N/A | Not supported | ❌ No |
| **Hashtable** | **No** — no order maintained | N/A | Not supported | ❌ No |
| **LinkedHashMap** | Yes (insertion order, via doubly linked list) | Yes (achievable) | Yes (achievable) | ✅ Yes (under `SequencedMap`) |
| **TreeMap (SortedMap)** | Yes (sorted order) | Yes — `.firstKey()`, `.lastKey()`, `.pollFirstEntry()`, `.pollLastEntry()` | Yes — `.descendingMap()` | ✅ Yes (under `SequencedMap`) |

**Key insight on `Queue` and `PriorityQueue`:** These were **excluded** because `Queue` structurally forbids first-end insertion and last-element access (strict FIFO), and `PriorityQueue` doesn't maintain any consistent order at all beyond guaranteeing the head — making both the first/last-access condition and the reversibility condition impossible to satisfy.

---

### Why SequencedSet Exists Separately from SequencedCollection
**The Layman's Definition:** `SequencedSet` is a distinct interface — not just direct use of `SequencedCollection` — because **Sets enforce uniqueness (no duplicates)**, a rule that `List` and `Deque` don't follow.

**How it Works / The Logic:**
- `List` and `Deque` allow duplicate values, so they extend `SequencedCollection` directly.
- `LinkedHashSet` and `TreeSet` (via `SortedSet`) must guarantee **no duplicates**, so a specialized `SequencedSet` interface was created — it extends `SequencedCollection` (inheriting all its first/last/reverse methods) while adding the uniqueness constraint.
- This is also why, when adding an already-existing value to a `LinkedHashSet` at the front (`addFirst`), the existing entry is simply **moved** to the front rather than duplicated.

**Example:**
```java
SequencedSet<Character> set = new LinkedHashSet<>(List.of('B', 'C', 'D'));
set.addFirst('A'); // [A, B, C, D]
set.addLast('Z');  // [A, B, C, D, Z]
set.addFirst('C'); // 'C' already exists — it's moved to the front, not duplicated
// Result: [C, A, B, D, Z]
```

---

### The New Hierarchy in Practice (Java 21+)
**The Layman's Definition:** With the new interfaces in place, every qualifying collection now shares the exact same method vocabulary for first/last access, manipulation, and reversal.

**How it Works / The Logic & Examples:**

**List (extends SequencedCollection):**
```java
List<Character> list = new ArrayList<>(List.of('B', 'C', 'D'));
list.getFirst(); // 'B'
list.getLast();  // 'D'
list.addFirst('A'); // [A, B, C, D]
list.addLast('Z');  // [A, B, C, D, Z]
list.removeFirst(); // removes 'A'
list.removeLast();  // removes 'Z'
List<Character> reversedView = list.reversed(); // [D, C, B]
```

**Deque (extends SequencedCollection):**
```java
Deque<Character> deque = new ArrayDeque<>(List.of('B', 'C', 'D'));
deque.getFirst(); deque.getLast();
deque.addFirst('A'); deque.addLast('Z');
deque.removeFirst(); deque.removeLast();
deque.reversed();
```

**SortedSet / TreeSet (extends SequencedSet) — `addFirst`/`addLast` throw exceptions:**
```java
SequencedSet<Integer> sortedSet = new TreeSet<>(List.of(5, 7, 14));
sortedSet.getFirst(); // 5
sortedSet.getLast();  // 14
sortedSet.addFirst(2); // throws UnsupportedOperationException — position is determined by sort order, not by explicit placement
sortedSet.add(2);      // correct way — automatically sorts to the right position
```

**LinkedHashMap (extends SequencedMap):**
```java
SequencedMap<Integer, Character> map = new LinkedHashMap<>();
map.put(100, 'B');
map.put(300, 'D');
map.firstEntry(); // 100=B
map.lastEntry();  // 300=D
map.putFirst(400, 'A'); // adds at the front
map.pollFirstEntry(); map.pollLastEntry();
map.reversed();
```

**SortedMap / TreeMap (extends SequencedMap) — `putFirst`/`putLast` throw exceptions:**
```java
SequencedMap<Integer, String> sortedMap = new TreeMap<>();
sortedMap.putFirst(50, "x"); // throws UnsupportedOperationException — position is key-order-determined
sortedMap.put(50, "x");      // correct way — automatically sorted by key
sortedMap.firstEntry(); sortedMap.lastEntry();
sortedMap.pollFirstEntry(); sortedMap.pollLastEntry();
sortedMap.reversed(); // or .descendingMap()
```

## Key Takeaways & Quick Reference
- Java 21 introduces three new interfaces: **SequencedCollection**, **SequencedSet**, and **SequencedMap**, fitting into the existing collection hierarchy above `List`, `Deque`, `LinkedHashSet`/`SortedSet`, and `LinkedHashMap`/`SortedMap`.
- A collection qualifies as "sequenced" only if it satisfies **all three conditions**: (1) predictable iteration order, (2) first/last element access & manipulation, (3) a reversible view.
- **Queue** and **PriorityQueue** are excluded — `Queue` is strict FIFO (no last-element access or front insertion), and `PriorityQueue` doesn't guarantee any consistent order beyond its head.
- **HashSet** and **HashMap**/**Hashtable** are excluded — they don't maintain insertion or sorted order at all.
- **SequencedSet** exists as a separate interface (extending `SequencedCollection`) specifically to enforce **no duplicates**, which `List`/`Deque` don't require.
- For **sorted structures** (`SortedSet`/`SortedMap`), calling `addFirst()`/`addLast()` or `putFirst()`/`putLast()` throws `UnsupportedOperationException` — position is always determined by sort order, never by explicit placement.
- The core motivation behind these interfaces: **eliminate inconsistent, collection-specific method names** for functionally identical operations (get/add/remove first-or-last, and reversal).
- The `reversed()` method returns a **live view**, not a copy — modifications through the reversed view affect the original underlying collection.

## Glossary of Terms
- **FIFO (First-In-First-Out)**: An ordering discipline (used by `Queue`) where elements are removed in the same order they were added.
- **Heap (data structure)**: The internal structure `PriorityQueue` uses, which only guarantees the minimum or maximum element is at the head — not a fully ordered sequence.
- **Doubly Linked List**: A linked list where each node has references to both its previous and next node, enabling efficient traversal and insertion from both ends — the internal structure behind `LinkedHashSet` and `LinkedHashMap`'s ordering.
- **Reversible/Reversed View**: A live, read-through representation of a collection in reverse order, without creating a separate copy of the data.
