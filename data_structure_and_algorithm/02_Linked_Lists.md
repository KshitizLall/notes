# Linked Lists - Comprehensive DSA Notes

## Core Concepts

### What are Linked Lists?
**Simple Definition**: A linear data structure where each element (node) contains data and a link to the next node.

**Real-world Analogy**: Like a treasure hunt where each clue points to the next location. You can't jump to the end; you must follow the chain from start to finish.

```python
# Node structure
class Node:
    def __init__(self, data: int):
        self.data = data
        self.next = None  # Points to next node

# Linked List visualization:
# Head -> [1|next] -> [2|next] -> [3|next] -> None
#         Node      Node       Node
```

### Singly Linked List vs Doubly Linked List

| Aspect | Singly | Doubly |
|--------|--------|--------|
| Navigation | Forward only | Both directions |
| Memory | Less (1 pointer) | More (2 pointers) |
| Insert/Delete | O(1) with pointer | O(1) with pointer |
| Reverse Traversal | O(n) | O(1) |
| Use Case | Memory tight | Need backward access |

### Time and Space Complexity

| Operation | Time | Space | Notes |
|-----------|------|-------|-------|
| Access by index | O(n) | O(1) | Must traverse from head |
| Search | O(n) | O(1) | Linear search |
| Insert at head | O(1) | O(1) | Update head pointer |
| Insert at tail | O(n) | O(1) | Must find tail first |
| Insert with pointer | O(1) | O(1) | If pointer provided |
| Delete at head | O(1) | O(1) | Update head |
| Delete by value | O(n) | O(1) | Must find node first |
| Delete with pointer | O(1) | O(1) | If pointer provided |

### When to Use Linked Lists
✅ Frequent insertions/deletions at beginning
✅ Don't need random access
✅ Memory allocation unpredictable
✅ Need dynamic size

### When to Avoid Linked Lists
❌ Need random access
❌ Lots of searching
❌ Cache locality important
❌ Extra memory overhead (pointers) matters

---

## Essential Patterns & Techniques

### 1. Basic Linked List Operations

**Template Code - Node Definition**:
```python
from typing import Optional

class ListNode:
    def __init__(self, val: int = 0, next: Optional['ListNode'] = None):
        self.val = val
        self.next = next

class LinkedList:
    def __init__(self):
        self.head = None

    def append(self, val: int) -> None:
        """Add node at end - O(n) time"""
        new_node = ListNode(val)

        if not self.head:
            self.head = new_node
            return

        current = self.head
        while current.next:
            current = current.next
        current.next = new_node

    def prepend(self, val: int) -> None:
        """Add node at beginning - O(1) time"""
        new_node = ListNode(val)
        new_node.next = self.head
        self.head = new_node

    def insert_after(self, prev_node: ListNode, val: int) -> None:
        """Insert after specific node - O(1) time"""
        if not prev_node:
            return

        new_node = ListNode(val)
        new_node.next = prev_node.next
        prev_node.next = new_node

    def delete(self, val: int) -> None:
        """Delete first node with value - O(n) time"""
        if not self.head:
            return

        # If head needs to be deleted
        if self.head.val == val:
            self.head = self.head.next
            return

        current = self.head
        while current.next:
            if current.next.val == val:
                current.next = current.next.next
                return
            current = current.next

    def display(self) -> None:
        """Print linked list"""
        current = self.head
        while current:
            print(current.val, end=" -> ")
            current = current.next
        print("None")
```

**Complexity Analysis**:
- Append: O(n) - must traverse to end
- Prepend: O(1) - update head
- Insert after: O(1) - with pointer
- Delete: O(n) - must find node
- Space: O(1) - no extra space (except new node)

---

### 2. Two Pointers Technique

**Description**: Use slow and fast pointers to detect cycles, find middle, etc.

**Template Code**:
```python
def find_middle(head: Optional[ListNode]) -> Optional[ListNode]:
    """
    Find middle node using slow/fast pointers.

    Approach:
    - Slow moves 1 step, fast moves 2 steps
    - When fast reaches end, slow is at middle
    - Works for both odd and even length lists
    """
    if not head or not head.next:
        return head

    slow = fast = head

    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

    return slow  # For odd length, returns middle
               # For even length, returns second middle

def has_cycle(head: Optional[ListNode]) -> bool:
    """
    Detect cycle in linked list (Floyd's Cycle Detection).

    Approach:
    - If fast and slow meet, there's a cycle
    - If fast reaches None, no cycle
    """
    if not head or not head.next:
        return False

    slow = fast = head

    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

        if slow == fast:
            return True

    return False

def find_cycle_start(head: Optional[ListNode]) -> Optional[ListNode]:
    """
    Find the node where cycle begins.

    Approach:
    1. Detect cycle using slow/fast pointers
    2. Move one pointer to head, keep other at meeting point
    3. Move both one step at a time
    4. Where they meet is cycle start
    """
    if not head:
        return None

    slow = fast = head

    # Find meeting point
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

        if slow == fast:
            break
    else:
        return None  # No cycle

    # Move slow to head
    slow = head

    # Move both one step at a time
    while slow != fast:
        slow = slow.next
        fast = fast.next

    return slow  # Start of cycle
```

**Complexity Analysis**:
- Time: O(n) - visit each node at most twice
- Space: O(1) - no extra space

**Visual Example**:
```
Find Middle: [1 -> 2 -> 3 -> 4 -> 5]
Step 0: slow=1, fast=1
Step 1: slow=2, fast=3
Step 2: slow=3, fast=5
Step 3: slow=4, fast=None
Result: slow=4 (not exactly middle, but close)

For [1 -> 2 -> 3 -> 4]:
Step 0: slow=1, fast=1
Step 1: slow=2, fast=3
Step 2: slow=3, fast=None
Result: slow=3
```

---

### 3. Reversal Techniques

**Description**: Reverse linked list or parts of it.

**Template Code - Iterative**:
```python
def reverseList(head: Optional[ListNode]) -> Optional[ListNode]:
    """
    Reverse entire linked list iteratively.

    Approach:
    1. Keep track of previous node
    2. Reverse current node's pointer
    3. Move to next node

    [1 -> 2 -> 3 -> None]
    [None <- 1 <- 2 <- 3]

    Time: O(n), Space: O(1)
    """
    prev = None
    current = head

    while current:
        # Store next node
        next_temp = current.next

        # Reverse the pointer
        current.next = prev

        # Move prev and current one step
        prev = current
        current = next_temp

    return prev  # New head
```

**Visual Example**:
```
Original:    1 -> 2 -> 3 -> None
Step 1:      1 <- 2    3 -> None (prev=1, current=2)
Step 2:      1 <- 2 <- 3    None (prev=2, current=3)
Step 3:      1 <- 2 <- 3 <- None (prev=3, current=None)
Result:      3 -> 2 -> 1 -> None
```

**Template Code - Recursive**:
```python
def reverseListRecursive(head: Optional[ListNode]) -> Optional[ListNode]:
    """
    Reverse linked list recursively.

    Approach:
    1. Recurse to end
    2. Reverse pointers while returning

    Time: O(n), Space: O(n) - recursion stack
    """
    if not head or not head.next:
        return head

    # Recurse to end
    new_head = reverseListRecursive(head.next)

    # Reverse pointers
    head.next.next = head  # Make next node point back
    head.next = None       # Break old pointer

    return new_head
```

---

### 4. Merge Technique

**Description**: Merge two sorted linked lists.

**Template Code**:
```python
def mergeTwoLists(list1: Optional[ListNode], list2: Optional[ListNode]) -> Optional[ListNode]:
    """
    Merge two sorted linked lists.

    Approach:
    1. Create dummy node as anchor
    2. Compare nodes from both lists
    3. Attach smaller one
    4. Handle remaining nodes

    Time: O(n + m), Space: O(1) - no extra space
    """
    # Dummy node helps avoid special case for head
    dummy = ListNode(0)
    current = dummy

    while list1 and list2:
        if list1.val <= list2.val:
            current.next = list1
            list1 = list1.next
        else:
            current.next = list2
            list2 = list2.next

        current = current.next

    # Attach remaining nodes
    if list1:
        current.next = list1
    if list2:
        current.next = list2

    return dummy.next
```

---

## Problem-Solving Framework

### Step 1: Draw It Out
- Visualize the linked list
- Mark what's changing (pointers, nodes)
- Trace through small examples

### Step 2: Identify the Pattern
```
Is this about...?
├─ Traversal? → Basic iteration, two pointers
├─ Reversal? → Iterative or recursive reversal
├─ Cycle? → Floyd's cycle detection
├─ Merging? → Two pointer merge
├─ Reordering? → Multiple passes or recursion
└─ Modification? → Dummy node technique
```

### Step 3: Handle Edge Cases
- Empty list: `if not head: return None`
- Single node: `if not head.next: return head`
- Two nodes: Handle specially if needed
- Cycles: Use visited set or fast/slow pointers

### Step 4: Be Careful With Pointers
- Don't lose references while modifying
- Use temporary variables to store next node
- Test with pen and paper first

---

## Common Pitfalls & Edge Cases

### 1. Lost Pointer References
```python
# ❌ Wrong - loses reference to node.next
current.next = new_node
current = current.next  # This is new_node, not what we want

# ✅ Correct - save reference first
next_temp = current.next
current.next = new_node
current = next_temp
```

### 2. Infinite Loops
```python
# ❌ Wrong - could loop forever if cycle exists
current = head
while current:
    print(current.val)
    current = current.next  # If cycle, never reaches None

# ✅ Correct - use counter or cycle detection
visited = set()
current = head
while current and id(current) not in visited:
    visited.add(id(current))
    current = current.next
```

### 3. Modifying While Iterating
```python
# ❌ Wrong - deleting while iterating
current = head
while current:
    if current.val == target:
        current = current.next  # Loses reference to node
    current = current.next

# ✅ Correct - keep track of previous
prev = None
current = head
while current:
    if current.val == target:
        if prev:
            prev.next = current.next
        else:
            head = current.next
    else:
        prev = current
    current = current.next
```

### 4. Dummy Node Importance
```python
# ❌ Wrong - special case for head deletion
if head.val == target:
    head = head.next
# ... then regular deletion code

# ✅ Correct - use dummy node
dummy = ListNode(0)
dummy.next = head
current = dummy
while current.next:
    if current.next.val == target:
        current.next = current.next.next
    else:
        current = current.next
return dummy.next
```

---

## Optimization Techniques

### Technique 1: Dummy Node
```python
# Makes head deletion easy
dummy = ListNode(0)
dummy.next = head
```

### Technique 2: Fast/Slow Pointers
```python
# Find middle, detect cycles, find start of cycle
slow = fast = head
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
```

### Technique 3: Recursion
```python
# Can reverse, merge, or transform recursively
# Trade time efficiency for code elegance
```

### Technique 4: In-place Reversal
```python
# Don't create new nodes
# Just reverse existing pointers
```

---

## Comparison Table

| Operation | Singly LL | Doubly LL | Array | Notes |
|-----------|-----------|-----------|-------|-------|
| Access | O(n) | O(n) | O(1) | LL needs traversal |
| Search | O(n) | O(n) | O(n) | Same complexity |
| Insert front | O(1) | O(1) | O(n) | LL advantage |
| Insert middle | O(n) | O(n) | O(n) | Same when found |
| Delete | O(n) | O(n) | O(n) | Same when found |
| Memory | Less | More | Fixed | LL overhead |
| Cache | Bad | Bad | Good | Array better |

---

## Curated LeetCode Problems

### Beginner Level

#### 1. Reverse Linked List (LeetCode #206)
**Pattern**: Iterative/Recursive Reversal
**Key Insight**: Reverse pointers while traversing

```python
def reverseList(head: Optional[ListNode]) -> Optional[ListNode]:
    """
    Reverse a singly linked list.

    Approach: Iterative pointer reversal
    - Track previous node
    - Reverse current's next pointer
    - Move forward

    Time: O(n), Space: O(1)
    """
    prev = None
    current = head

    while current:
        next_temp = current.next  # Save next
        current.next = prev        # Reverse pointer
        prev = current             # Move prev forward
        current = next_temp        # Move current forward

    return prev

# Test
list1 = ListNode(1, ListNode(2, ListNode(3)))
result = reverseList(list1)
# Result: 3 -> 2 -> 1
```

---

#### 2. Merge Two Sorted Lists (LeetCode #21)
**Pattern**: Two Pointer Merge
**Key Insight**: Use dummy node to avoid head special case

```python
def mergeTwoLists(list1: Optional[ListNode], list2: Optional[ListNode]) -> Optional[ListNode]:
    """
    Merge two sorted linked lists into one sorted list.

    Approach: Two pointer comparison
    1. Create dummy node
    2. Compare heads of both lists
    3. Attach smaller one
    4. Attach remaining

    Time: O(n + m), Space: O(1)
    """
    dummy = ListNode(0)
    current = dummy

    # Compare and merge
    while list1 and list2:
        if list1.val <= list2.val:
            current.next = list1
            list1 = list1.next
        else:
            current.next = list2
            list2 = list2.next
        current = current.next

    # Attach remaining
    current.next = list1 if list1 else list2

    return dummy.next

# Test
l1 = ListNode(1, ListNode(2, ListNode(4)))
l2 = ListNode(1, ListNode(3, ListNode(4)))
result = mergeTwoLists(l1, l2)
# Result: 1 -> 1 -> 2 -> 3 -> 4 -> 4
```

---

#### 3. Linked List Cycle (LeetCode #141)
**Pattern**: Floyd's Cycle Detection
**Key Insight**: Fast pointer catches slow pointer if cycle exists

```python
def hasCycle(head: Optional[ListNode]) -> bool:
    """
    Detect if linked list has a cycle.

    Approach: Floyd's cycle detection
    - Slow pointer moves 1 step
    - Fast pointer moves 2 steps
    - If they meet, cycle exists

    Why? In a cycle, fast always catches up
    Time: O(n), Space: O(1)
    """
    if not head or not head.next:
        return False

    slow = fast = head

    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

        if slow == fast:
            return True

    return False

# Test
node1 = ListNode(3)
node2 = ListNode(2)
node3 = ListNode(0)
node4 = ListNode(-4)
node1.next = node2
node2.next = node3
node3.next = node4
node4.next = node2  # Cycle: points back to node2
assert hasCycle(node1) == True
```

---

### Intermediate Level

#### 4. Find the Duplicate Number (LeetCode #287)
**Pattern**: Floyd's Cycle Detection
**Key Insight**: Treat array indices as linked list

```python
def findDuplicate(nums: list[int]) -> int:
    """
    Find duplicate in array where all numbers in range [1, n].

    Approach: Floyd's cycle detection on implicit linked list
    - Treat array as linked list: nums[i] points to nums[nums[i]]
    - First duplicate = cycle start

    Why? If duplicate exists, must have cycle
    Time: O(n), Space: O(1)
    """
    slow = fast = nums[0]

    # Find meeting point in cycle
    while True:
        slow = nums[slow]
        fast = nums[nums[fast]]
        if slow == fast:
            break

    # Find cycle start
    slow = nums[0]
    while slow != fast:
        slow = nums[slow]
        fast = nums[fast]

    return slow

# Test
assert findDuplicate([1, 3, 4, 2, 2]) == 2
assert findDuplicate([3, 1, 3, 4, 2]) == 3
```

---

#### 5. Middle of the Linked List (LeetCode #876)
**Pattern**: Two Pointers
**Key Insight**: Slow moves 1, fast moves 2 steps

```python
def middleNode(head: Optional[ListNode]) -> Optional[ListNode]:
    """
    Find middle node of linked list.

    Approach: Slow/fast pointers
    - When fast reaches end, slow is at middle
    - For even length, returns second middle

    Time: O(n), Space: O(1)
    """
    slow = fast = head

    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

    return slow

# Test
head = ListNode(1, ListNode(2, ListNode(3, ListNode(4, ListNode(5)))))
result = middleNode(head)
assert result.val == 3  # Returns middle node
```

---

#### 6. Linked List Cycle II (LeetCode #142)
**Pattern**: Floyd's Cycle Detection
**Key Insight**: Find where cycle begins

```python
def detectCycle(head: Optional[ListNode]) -> Optional[ListNode]:
    """
    Find the node where cycle begins (or None if no cycle).

    Approach: Floyd's detection + two pointer technique
    1. Detect cycle meeting point
    2. One pointer to head, other to meeting point
    3. Move both forward, they meet at cycle start

    Why? Distance to cycle start is same from head and meeting point
    Time: O(n), Space: O(1)
    """
    if not head or not head.next:
        return None

    slow = fast = head

    # Find meeting point
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow == fast:
            break
    else:
        return None  # No cycle

    # Find cycle start
    slow = head
    while slow != fast:
        slow = slow.next
        fast = fast.next

    return slow

# Test
node1 = ListNode(3)
node2 = ListNode(2)
node3 = ListNode(0)
node4 = ListNode(-4)
node1.next = node2
node2.next = node3
node3.next = node4
node4.next = node2
assert detectCycle(node1).val == 2
```

---

### Advanced Level

#### 7. Reverse Linked List II (LeetCode #92)
**Pattern**: Partial Reversal
**Key Insight**: Reverse only part of list

```python
def reverseBetween(head: Optional[ListNode], left: int, right: int) -> Optional[ListNode]:
    """
    Reverse linked list from position left to right.

    Approach: Find boundary, reverse, reconnect
    1. Find node before reversal start
    2. Reverse nodes between left and right
    3. Reconnect with remaining list

    Time: O(n), Space: O(1)
    """
    if not head or left == right:
        return head

    # Dummy helps with edge case: reversing from head
    dummy = ListNode(0)
    dummy.next = head
    prev = dummy

    # Find node before reversal starts
    for _ in range(left - 1):
        prev = prev.next

    # Start of reversal section
    curr = prev.next

    # Reverse 'right - left' nodes
    for _ in range(right - left):
        next_temp = curr.next
        curr.next = next_temp.next
        next_temp.next = prev.next
        prev.next = next_temp

    return dummy.next

# Test
head = ListNode(1, ListNode(2, ListNode(3, ListNode(4, ListNode(5)))))
result = reverseBetween(head, 2, 4)
# Result: 1 -> 4 -> 3 -> 2 -> 5
```

---

#### 8. Copy List with Random Pointer (LeetCode #138)
**Pattern**: Node Cloning with Hash Map
**Key Insight**: Track mapping between original and copy

```python
class Node:
    def __init__(self, x: int, next: 'Node' = None, random: 'Node' = None):
        self.val = x
        self.next = next
        self.random = random

def copyRandomList(head: Optional[Node]) -> Optional[Node]:
    """
    Deep copy linked list with random pointers.

    Approach: Two-pass with hash map
    1. First pass: create all nodes, map original -> copy
    2. Second pass: set next and random pointers

    Time: O(n), Space: O(n)
    """
    if not head:
        return None

    # First pass: create copy nodes and map
    node_map = {}
    current = head
    while current:
        node_map[current] = Node(current.val)
        current = current.next

    # Second pass: set pointers
    current = head
    while current:
        copy = node_map[current]
        copy.next = node_map[current.next] if current.next else None
        copy.random = node_map[current.random] if current.random else None
        current = current.next

    return node_map[head]

# Alternative: O(1) space using interweaving
def copyRandomListSpaceOptimal(head: Optional[Node]) -> Optional[Node]:
    """Space-optimized version using interweaving."""
    if not head:
        return None

    # First pass: interweave original and copy
    current = head
    while current:
        copy = Node(current.val)
        copy.next = current.next
        current.next = copy
        current = copy.next

    # Second pass: set random pointers
    current = head
    while current:
        if current.random:
            current.next.random = current.random.next
        current = current.next.next

    # Third pass: separate lists
    dummy = Node(0)
    copy_prev = dummy
    current = head
    while current:
        copy_prev.next = current.next
        copy_prev = copy_prev.next
        current.next = current.next.next
        current = current.next

    return dummy.next
```

---

#### 9. Reorder List (LeetCode #143)
**Pattern**: Find Middle + Reverse + Merge
**Key Insight**: Combine multiple techniques

```python
def reorderList(head: Optional[ListNode]) -> None:
    """
    Reorder list as: [L0 -> Ln -> L1 -> Ln-1 -> ...]

    Approach: Three-step process
    1. Find middle using slow/fast pointers
    2. Reverse second half
    3. Merge first and reversed second half

    Time: O(n), Space: O(1)
    """
    if not head or not head.next:
        return

    # Step 1: Find middle
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

    # Step 2: Reverse second half
    prev = None
    current = slow.next
    slow.next = None

    while current:
        next_temp = current.next
        current.next = prev
        prev = current
        current = next_temp

    # Step 3: Merge first and reversed second half
    first = head
    second = prev

    while first and second:
        next_temp1 = first.next
        next_temp2 = second.next

        first.next = second
        second.next = next_temp1

        first = next_temp1
        second = next_temp2

# Test
head = ListNode(1, ListNode(2, ListNode(3, ListNode(4))))
reorderList(head)
# Result: 1 -> 4 -> 2 -> 3
```

---

## Practice Roadmap

### Beginner (Foundation)
1. **Basic Operations**
   - LeetCode #206: Reverse Linked List
   - LeetCode #21: Merge Two Sorted Lists
   - LeetCode #203: Remove Linked List Elements

2. **Two Pointers**
   - LeetCode #876: Middle of the Linked List
   - LeetCode #141: Linked List Cycle
   - LeetCode #160: Intersection of Two Linked Lists

### Intermediate (Pattern Mastery)
1. **Cycle Detection**
   - LeetCode #142: Linked List Cycle II
   - LeetCode #287: Find the Duplicate Number

2. **Modification**
   - LeetCode #92: Reverse Linked List II
   - LeetCode #24: Swap Nodes in Pairs
   - LeetCode #25: Reverse Nodes in k-Group

3. **Merging**
   - LeetCode #23: Merge k Sorted Lists
   - LeetCode #148: Sort List

### Advanced (Mastery)
1. **Complex Patterns**
   - LeetCode #138: Copy List with Random Pointer
   - LeetCode #143: Reorder List
   - LeetCode #146: LRU Cache

2. **Challenging Problems**
   - LeetCode #430: Flatten a Multilevel Doubly Linked List
   - LeetCode #1721: Swapping Nodes in a Linked List
   - LeetCode #2130: Maximum Twin Sum of a Linked List

---

## Quick Reference

```python
# Basic node
class ListNode:
    def __init__(self, val, next=None):
        self.val = val
        self.next = next

# Reversal
prev = None
while current:
    next_temp = current.next
    current.next = prev
    prev = current
    current = next_temp

# Cycle detection
slow = fast = head
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
    if slow == fast: return True

# Find middle
slow = fast = head
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
```

---

## Key Takeaways

1. **Master pointer manipulation** - Core of linked list work
2. **Use dummy nodes** - Simplifies head special cases
3. **Slow/fast pointers** - Solves middle, cycle, start of cycle
4. **Always draw it out** - Visual understanding crucial
5. **Handle edge cases** - Empty, single node, cycles
6. **In-place operations** - Modify pointers, not values
7. **Space optimization** - Prefer O(1) space

