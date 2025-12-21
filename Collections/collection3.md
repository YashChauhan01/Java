# Java Deque, List & Implementations - Complete Notes

## Table of Contents
1. [Deque Interface](#deque-interface)
2. [ArrayDeque](#arraydeque)
3. [List Interface](#list-interface)
4. [ArrayList](#arraylist)
5. [LinkedList](#linkedlist)
6. [Vector](#vector)
7. [Stack](#stack)
8. [Comparison Tables](#comparison-tables)

---

## Deque Interface

### What is Deque?
- **Deque** = **Double Ended Queue**
- **Extends**: Queue interface
- **Key Feature**: Addition and removal can happen from **both ends**

### Queue vs Deque

| Queue | Deque |
|-------|-------|
| Add at rear only | Add at both ends |
| Remove from front only | Remove from both ends |
| FIFO structure | Flexible structure |

### Visual Representation
```
Regular Queue:
Add →  [1] [2] [3] [4] [5]  → Remove
       Rear            Front

Deque:
Add/Remove ← [1] [2] [3] [4] [5] → Add/Remove
           Front              Rear
```

---

## Deque Methods

Deque provides **12 new methods** (in addition to Queue methods):

### 1. Insertion Operations (4 methods)

| Method | Description | Exception Behavior |
|--------|-------------|-------------------|
| `addFirst(e)` | Insert at start | Throws exception on failure |
| `offerFirst(e)` | Insert at start | Returns false on failure |
| `addLast(e)` | Insert at end | Throws exception on failure |
| `offerLast(e)` | Insert at end | Returns false on failure |

### 2. Removal Operations (4 methods)

| Method | Description | Empty Queue Behavior |
|--------|-------------|---------------------|
| `removeFirst()` | Remove from start | Throws exception |
| `pollFirst()` | Remove from start | Returns null |
| `removeLast()` | Remove from end | Throws exception |
| `pollLast()` | Remove from end | Returns null |

### 3. Examination Operations (4 methods)

| Method | Description | Empty Queue Behavior |
|--------|-------------|---------------------|
| `getFirst()` | Get first element | Throws exception |
| `peekFirst()` | Get first element | Returns null |
| `getLast()` | Get last element | Throws exception |
| `peekLast()` | Get last element | Returns null |

---

## How Old Queue Methods Work in Deque

```java
// Queue method → Deque equivalent
add(e)      → addLast(e)
offer(e)    → offerLast(e)
remove()    → removeFirst()
poll()      → pollFirst()
element()   → getFirst()
peek()      → peekFirst()
```

**Key Point**: Using regular Queue methods maintains normal FIFO behavior!

---

## Using Deque as Stack

Deque can implement Stack (LIFO) behavior:

```java
Deque<Integer> stack = new ArrayDeque<>();

// Stack operations using Deque
stack.push(1);    // Internally calls addFirst()
stack.push(2);
stack.push(3);

stack.pop();      // Internally calls removeFirst()
// Returns: 3, 2, 1 (LIFO order)
```

### Stack Visualization
```
Push operations (addFirst):
[1] → [2][1] → [3][2][1]

Pop operations (removeFirst):
[3][2][1] → [2][1] → [1] → []
```

---

## ArrayDeque

### What is ArrayDeque?
- **Concrete class** implementing Deque
- Uses **resizable array** internally
- **Not thread-safe**
- **No null elements** allowed

### ArrayDeque as Queue Example

```java
Deque<Integer> queue = new ArrayDeque<>();

// Using as Queue (FIFO)
queue.addLast(1);     // [1]
queue.addLast(5);     // [1, 5]
queue.addLast(10);    // [1, 5, 10]

// Remove from front
System.out.println(queue.removeFirst()); // 1
System.out.println(queue.removeFirst()); // 5
System.out.println(queue.removeFirst()); // 10
```

### ArrayDeque as Stack Example

```java
Deque<Integer> stack = new ArrayDeque<>();

// Using as Stack (LIFO)
stack.addFirst(1);    // [1]
stack.addFirst(5);    // [5, 1]
stack.addFirst(10);   // [10, 5, 1]

// Remove from front
System.out.println(stack.removeFirst()); // 10
System.out.println(stack.removeFirst()); // 5
System.out.println(stack.removeFirst()); // 1
```

---

## ArrayDeque Time Complexity

| Operation | Time Complexity | Notes |
|-----------|----------------|-------|
| `addFirst()` / `addLast()` | O(1) amortized | O(n) when resizing needed |
| `removeFirst()` / `removeLast()` | O(1) | Always constant time |
| `peekFirst()` / `peekLast()` | O(1) | Just examine, no removal |
| Space | O(n) | n = number of elements |

### Why "Amortized" O(1)?

**Initial Capacity**: 8 elements

When full, ArrayDeque **doubles** its capacity:
1. Create new array (size × 2)
2. Copy all elements
3. Point to new array

**Example**:
- Insert 1000 elements
- Resize happens ~7 times (8→16→32→64→128→256→512→1024)
- Average insertion time ≈ O(1)

---

## ArrayDeque - Thread Safety

### ArrayDeque (Not Thread-Safe)
```java
Deque<Integer> deque = new ArrayDeque<>();
// ❌ Not safe with multiple threads
```

### ConcurrentLinkedDeque (Thread-Safe)
```java
Deque<Integer> deque = new ConcurrentLinkedDeque<>();
// ✅ Safe for concurrent access
deque.addFirst(1);
deque.addLast(10);
```

---

## List Interface

### What is List?
- **Interface** extending Collection
- **Ordered collection** with duplicates allowed
- **Index-based** access (0, 1, 2, ...)
- Uses **array** data structure internally

### List vs Queue

| Aspect | Queue | List |
|--------|-------|------|
| **Access** | Only at ends (front/rear) | Anywhere (by index) |
| **Structure** | FIFO/LIFO | Index-based (0, 1, 2...) |
| **Insert** | Only at ends | At any index |
| **Remove** | Only from ends | From any index |

### Visual Comparison
```
Queue: Add at rear, remove from front
[Front] ← [2] [3] [4] [5] ← [Rear]

List: Access anywhere by index
[0]  [1]  [2]  [3]  [4]
 2    3    4    5    6
 ↑    ↑    ↑    ↑    ↑
Access by index
```

---

## List Methods

List provides **Collection methods** + **new index-based methods**:

### Index-Based Methods

| Method | Description |
|--------|-------------|
| `add(index, element)` | Insert at index, shift right |
| `addAll(index, collection)` | Insert collection at index |
| `get(index)` | Get element at index |
| `set(index, element)` | **Replace** element at index |
| `remove(index)` | Remove element, shift left |
| `indexOf(object)` | First occurrence index |
| `lastIndexOf(object)` | Last occurrence index |
| `subList(from, to)` | Get sublist [from, to) |
| `sort(comparator)` | Sort using comparator |
| `replaceAll(operator)` | Apply function to all elements |

### Add vs Set - Key Difference

```java
List<Integer> list = new ArrayList<>(Arrays.asList(10, 20, 30, 40));

// add() - shifts elements right
list.add(2, 100);  // [10, 20, 100, 30, 40]

// set() - replaces element
list.set(2, 200);  // [10, 20, 200, 30, 40]
```

---

## ListIterator

### What is ListIterator?
- **Child** of Iterator
- Allows **bidirectional** traversal
- Can **modify** list while iterating

### Iterator vs ListIterator

| Feature | Iterator | ListIterator |
|---------|----------|--------------|
| Direction | Forward only | Forward + Backward |
| Methods | hasNext(), next(), remove() | + hasPrevious(), previous(), add(), set() |
| Modification | Remove only | Add, Remove, Set |

### ListIterator Methods

| Method | Description |
|--------|-------------|
| `hasNext()` | Check if next element exists |
| `next()` | Move forward, return element |
| `hasPrevious()` | Check if previous element exists |
| `previous()` | Move backward, return element |
| `nextIndex()` | Get next element index |
| `previousIndex()` | Get previous element index |
| `add(element)` | Add element at cursor position |
| `set(element)` | Replace last returned element |
| `remove()` | Remove last returned element |

---

## ListIterator Examples

### Forward Traversal
```java
List<Integer> list = Arrays.asList(10, 20, 30, 40);
ListIterator<Integer> iterator = list.listIterator();

while(iterator.hasNext()) {
    int value = iterator.next();
    System.out.println(value + " at index " + iterator.previousIndex());
}
// Output: 10 at index 0, 20 at index 1, 30 at index 2, 40 at index 3
```

### Backward Traversal
```java
List<Integer> list = Arrays.asList(10, 20, 30, 40);
ListIterator<Integer> iterator = list.listIterator(list.size()); // Start at end

while(iterator.hasPrevious()) {
    int value = iterator.previous();
    System.out.println(value);
}
// Output: 40, 30, 20, 10
```

### Modifying During Iteration
```java
List<Integer> list = new ArrayList<>(Arrays.asList(10, 20, 30, 40));
ListIterator<Integer> iterator = list.listIterator();

while(iterator.hasNext()) {
    int value = iterator.next();
    if(value == 20) {
        iterator.set(200);     // Replace 20 with 200
        iterator.add(25);      // Add 25 after cursor
    }
}
// Result: [10, 200, 25, 30, 40]
```

### Understanding Cursor Position

```
List: [10, 20, 30, 40]
      ↑
   Cursor (initial position)

After next():
      [10, 20, 30, 40]
           ↑
        Cursor

add(25) inserts BEFORE next element:
      [10, 20, 25, 30, 40]
              ↑
           Cursor
```

---

## ArrayList

### What is ArrayList?
- **Concrete class** implementing List
- Uses **resizable array** internally
- **Not thread-safe**
- **Maintains insertion order**

### ArrayList Complete Example

```java
List<Integer> list = new ArrayList<>();

// 1. add(index, element)
list.add(0, 100);  // [100]
list.add(1, 200);  // [100, 200]
list.add(2, 300);  // [100, 200, 300]

// 2. addAll(index, collection)
List<Integer> list2 = Arrays.asList(400, 500, 600);
list.addAll(2, list2);  // [100, 200, 400, 500, 600, 300]

// 3. replaceAll(operator)
list.replaceAll(val -> val * -1);  // [-100, -200, -400, -500, -600, -300]

// 4. sort(comparator)
list.sort((a, b) -> a - b);  // Ascending: [-600, -500, -400, -300, -200, -100]

// 5. get(index)
int value = list.get(2);  // -400

// 6. set(index, element)
list.set(2, -4000);  // [-600, -500, -4000, -300, -200, -100]

// 7. remove(index)
list.remove(2);  // [-600, -500, -300, -200, -100]

// 8. indexOf(object)
int index = list.indexOf(-200);  // 3

// 9. lastIndexOf(object)
int lastIndex = list.lastIndexOf(-200);  // 3 (if no duplicates)
```

---

## ArrayList Time Complexity

| Operation | Time Complexity | Reason |
|-----------|----------------|--------|
| `add(element)` | O(1) amortized | O(n) when resizing |
| `add(index, element)` | O(n) | Shift elements right |
| `get(index)` | O(1) | Direct index access |
| `set(index, element)` | O(1) | Direct index access |
| `remove(index)` | O(n) | Shift elements left |
| `contains(element)` | O(n) | Linear search |
| `indexOf(element)` | O(n) | Linear search |
| Space | O(n) | n = number of elements |

### Why O(n) for Insertion/Deletion?

**Insertion at index 2**:
```
Before: [1, 2, 3, 4, 5]
         0  1  2  3  4

Insert 100 at index 2:
Step 1: Shift [3, 4, 5] right
Step 2: Insert 100

After:  [1, 2, 100, 3, 4, 5]
         0  1   2   3  4  5
```

**Deletion at index 2**:
```
Before: [1, 2, 3, 4, 5]
         0  1  2  3  4

Remove index 2:
Step 1: Remove 3
Step 2: Shift [4, 5] left

After:  [1, 2, 4, 5]
         0  1  2  3
```

---

## ArrayList - Thread Safety

### ArrayList (Not Thread-Safe)
```java
List<Integer> list = new ArrayList<>();
// ❌ Not safe with multiple threads
```

### CopyOnWriteArrayList (Thread-Safe)
```java
List<Integer> list = new CopyOnWriteArrayList<>();
// ✅ Safe for concurrent access
list.add(1);
list.add(2);
```

**How it works**: Creates a new copy of the array for every write operation!

---

## LinkedList

### What is LinkedList?
- **Implements** both List and Deque
- Uses **doubly-linked list** internally
- **Faster** than ArrayList for frequent insertions/deletions
- **Not thread-safe**

### LinkedList Structure
```
[100] ⇄ [200] ⇄ [300] ⇄ [400]
  ↑                       ↑
Head                    Tail
```

### LinkedList Dual Functionality

#### 1. As List (Index-Based)
```java
LinkedList<Integer> list = new LinkedList<>();

// List operations
list.add(0, 100);     // [100]
list.add(1, 200);     // [100, 200]
list.add(2, 300);     // [100, 200, 300]

System.out.println(list.get(1));  // 200
```

#### 2. As Deque (Double-Ended)
```java
LinkedList<Integer> deque = new LinkedList<>();

// Deque operations
deque.addLast(200);   // [200]
deque.addLast(300);   // [200, 300]
deque.addLast(400);   // [200, 300, 400]
deque.addFirst(100);  // [100, 200, 300, 400]

System.out.println(deque.getFirst());  // 100
```

#### 3. Combined Example
```java
LinkedList<Integer> list = new LinkedList<>();
list.add(0, 100);      // [100]
list.add(1, 300);      // [100, 300]
list.add(2, 400);      // [100, 300, 400]
list.add(1, 200);      // [100, 200, 300, 400] - Insert at index 1

System.out.println(list.get(1));  // 200
System.out.println(list.get(2));  // 300
```

---

## LinkedList Time Complexity

| Operation | Time Complexity | Reason |
|-----------|----------------|--------|
| `addFirst()` / `addLast()` | O(1) | Direct pointer manipulation |
| `add(index, element)` | O(n) | O(n) lookup + O(1) insertion |
| `get(index)` | O(n) | Must traverse from head |
| `removeFirst()` / `removeLast()` | O(1) | Direct pointer manipulation |
| `remove(index)` | O(n) | O(n) lookup + O(1) deletion |
| `contains(element)` | O(n) | Linear search |
| Space | O(n) | Extra space for pointers |

### Why LinkedList is Faster for Insertions?

**ArrayList Insertion** (O(n)):
```
[1, 2, 3, 4, 5]
Insert 100 at index 2
→ Shift [3, 4, 5] right (expensive!)
[1, 2, 100, 3, 4, 5]
```

**LinkedList Insertion** (O(1) after reaching position):
```
[1] → [2] → [3] → [4] → [5]
Insert 100 after node 2
→ Change pointers only
[1] → [2] → [100] → [3] → [4] → [5]
```

---

## Vector

### What is Vector?
- **Synchronized** version of ArrayList
- **Thread-safe** (all methods synchronized)
- **Less efficient** than ArrayList due to locking overhead
- **Legacy class** (introduced in Java 1.0)

### Vector vs ArrayList

| Feature | ArrayList | Vector |
|---------|-----------|--------|
| Thread-Safe | ❌ No | ✅ Yes |
| Performance | Fast | Slower (locking overhead) |
| Synchronization | Manual needed | Built-in |
| Growth | 1.5x | 2x (doubles) |
| Since | Java 1.2 | Java 1.0 (legacy) |

### Vector Example

```java
Vector<Integer> vector = new Vector<>();

// All operations are synchronized
vector.add(10);         // Thread-safe
vector.add(20);         // Thread-safe
vector.remove(0);       // Thread-safe

System.out.println(vector.get(0));  // 20
```

### Vector Synchronization

```java
// Inside Vector class
public synchronized boolean add(E e) {
    // Implementation
}

public synchronized E remove(int index) {
    // Implementation
}

public synchronized E get(int index) {
    // Implementation
}
```

**Key Point**: Every method has `synchronized` keyword → Thread-safe but slower!

---

## Stack

### What is Stack?
- **Extends** Vector (inherits thread-safety)
- Implements **LIFO** (Last In First Out)
- **Thread-safe** (synchronized methods)
- **Legacy class**

### Stack vs Deque

| Feature | Stack | ArrayDeque (as Stack) |
|---------|-------|-----------------------|
| Thread-Safe | ✅ Yes | ❌ No |
| Performance | Slower | Faster |
| Methods | push(), pop(), peek() | addFirst(), removeFirst(), peekFirst() |
| Recommended | Legacy | ✅ Modern approach |

### Stack Example

```java
Stack<Integer> stack = new Stack<>();

// Push operations
stack.push(1);    // [1]
stack.push(2);    // [2, 1]
stack.push(3);    // [3, 2, 1]
stack.push(4);    // [4, 3, 2, 1]

// Pop operations
System.out.println(stack.pop());   // 4
System.out.println(stack.pop());   // 3
System.out.println(stack.pop());   // 2
System.out.println(stack.pop());   // 1

// Peek (without removal)
stack.push(10);
System.out.println(stack.peek());  // 10 (still in stack)
```

### Stack Visualization
```
Stack Growth (LIFO):
Push 1:  [1]
Push 2:  [2]
         [1]
Push 3:  [3]
         [2]
         [1]
Push 4:  [4]
         [3]
         [2]
         [1]

Stack Shrink:
Pop:     [3]  (4 removed)
         [2]
         [1]
```

---

## Stack Time Complexity

| Operation | Time Complexity |
|-----------|----------------|
| `push(element)` | O(1) |
| `pop()` | O(1) |
| `peek()` | O(1) |
| `search(element)` | O(n) |
| Space | O(n) |

---

## Comparison Tables

### ArrayList vs LinkedList vs Vector

| Feature | ArrayList | LinkedList | Vector |
|---------|-----------|------------|--------|
| **Data Structure** | Resizable Array | Doubly Linked List | Resizable Array |
| **Thread-Safe** | ❌ No | ❌ No | ✅ Yes |
| **Random Access** | ✅ O(1) | ❌ O(n) | ✅ O(1) |
| **Insertion (end)** | O(1) amortized | O(1) | O(1) amortized |
| **Insertion (index)** | O(n) | O(n) | O(n) |
| **Deletion (end)** | O(1) | O(1) | O(1) |
| **Deletion (index)** | O(n) | O(n) | O(n) |
| **Memory Overhead** | Low | High (pointers) | Low |
| **Growth Factor** | 1.5x | N/A | 2x |
| **Null Elements** | ✅ Allowed | ✅ Allowed | ✅ Allowed |
| **Duplicates** | ✅ Allowed | ✅ Allowed | ✅ Allowed |
| **Insertion Order** | ✅ Maintained | ✅ Maintained | ✅ Maintained |
| **Best For** | Random access | Frequent insert/delete | Thread-safe lists |
| **Thread-Safe Alt** | CopyOnWriteArrayList | - | N/A (already safe) |

---

### Queue Implementations Comparison

| Feature | PriorityQueue | ArrayDeque | LinkedList |
|---------|--------------|------------|------------|
| **Data Structure** | Heap | Resizable Array | Doubly Linked List |
| **Thread-Safe** | ❌ No | ❌ No | ❌ No |
| **Ordering** | Priority-based | Insertion order | Insertion order |
| **Null Elements** | ❌ Not allowed | ❌ Not allowed | ✅ Allowed |
| **Deque Support** | ❌ No | ✅ Yes | ✅ Yes |
| **Best For** | Priority-based processing | Stack/Queue operations | Deque operations |
| **Thread-Safe Alt** | PriorityBlockingQueue | ConcurrentLinkedDeque | - |

---

### Stack Implementations Comparison

| Feature | Stack (Legacy) | ArrayDeque | LinkedList |
|---------|---------------|------------|------------|
| **Thread-Safe** | ✅ Yes | ❌ No | ❌ No |
| **Performance** | Slower (locking) | ✅ Faster | Fast |
| **Methods** | push/pop/peek | addFirst/removeFirst | addFirst/removeFirst |
| **Recommended** | ❌ No (legacy) | ✅ Yes | ✅ Yes |
| **Parent** | Vector | - | - |

---

## When to Use What?

### Choose ArrayList When:
✅ Frequent **random access** by index  
✅ Rare insertions/deletions in middle  
✅ **Single-threaded** environment  
✅ Memory efficiency important  

### Choose LinkedList When:
✅ Frequent **insertions/deletions** at ends  
✅ Implementing **Deque** behavior  
✅ No random access needed  
✅ Don't mind extra memory for pointers  

### Choose Vector When:
✅ Need **thread-safe** list  
✅ Working with **legacy code**  
⚠️ Note: Prefer CopyOnWriteArrayList for modern code

### Choose ArrayDeque When:
✅ Implementing **Stack** (LIFO)  
✅ Implementing **Queue** (FIFO)  
✅ Need **double-ended** operations  
✅ **Faster** than Stack/LinkedList  

### Choose Stack When:
✅ Need **thread-safe** stack  
✅ Working with **legacy code**  
⚠️ Note: Prefer ArrayDeque for modern code

---

## Key Takeaways

### Deque
- Double-ended queue with operations on both ends
- Can implement both Queue and Stack
- ArrayDeque is the recommended implementation

### List
- Index-based ordered collection
- Three main implementations: ArrayList, LinkedList, Vector
- ArrayList is most commonly used

### ArrayList
- Fast random access (O(1))
- Slow insertions/deletions in middle (O(n))
- Not thread-safe (use CopyOnWriteArrayList)

### LinkedList
- Fast insertions/deletions at ends (O(1))
- Slow random access (O(n))
- Implements both List and Deque

### Vector
- Thread-safe ArrayList alternative
- Slower due to synchronization overhead
- Legacy class - prefer CopyOnWriteArrayList

### Stack
- Thread-safe LIFO structure
- Extends Vector
- Legacy class - prefer ArrayDeque

---

## Thread-Safe Alternatives Summary

| Non-Thread-Safe | Thread-Safe Alternative |
|-----------------|------------------------|
| ArrayList | CopyOnWriteArrayList |
| LinkedList | (Use synchronizedList) |
| ArrayDeque | ConcurrentLinkedDeque |
| PriorityQueue | PriorityBlockingQueue |
| Stack | (Already thread-safe) |

---

## Quick Reference Code

```java
// List - ArrayList
List<Integer> arrayList = new ArrayList<>();
arrayList.add(10);
arrayList.get(0);        // O(1)
arrayList.remove(0);     // O(n)

// List - LinkedList
List<Integer> linkedList = new LinkedList<>();
linkedList.add(0, 10);   // O(n)
linkedList.get(0);       // O(n)

// Deque - ArrayDeque (as Queue)
Deque<Integer> queue = new ArrayDeque<>();
queue.addLast(10);
queue.removeFirst();

// Deque - ArrayDeque (as Stack)
Deque<Integer> stack = new ArrayDeque<>();
stack.addFirst(10);
stack.removeFirst();

// Vector (Thread-safe)
Vector<Integer> vector = new Vector<>();
vector.add(10);

// Stack (Thread-safe)
Stack<Integer> stack = new Stack<>();
stack.push(10);
stack.pop();
```

---

*Happy Coding! 🚀*
