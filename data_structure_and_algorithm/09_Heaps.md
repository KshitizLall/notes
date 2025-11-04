# Heaps - Comprehensive DSA Notes

## Core Concepts

### What is a Heap?
**Simple Definition**: A complete binary tree where parent is always <= (min-heap) or >= (max-heap) than children.

**Real-world Analogy**: Priority queue at hospital - urgent patients are served first regardless of arrival order.

```python
import heapq

# Min-heap (default)
min_heap = []
heapq.heappush(min_heap, 5)
heapq.heappush(min_heap, 3)
min_val = heapq.heappop(min_heap)  # 3

# Max-heap (negate values)
max_heap = []
heapq.heappush(max_heap, -5)
heapq.heappush(max_heap, -3)
max_val = -heapq.heappop(max_heap)  # 5
```

### Heap Properties
- **Complete Binary Tree**: All levels filled except possibly last
- **Heap Order**: Parent >= children (max-heap) or <= children (min-heap)
- **Array Representation**: index i has left child at 2i+1, right at 2i+2

### Operations & Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Push (insert) | O(log n) | O(1) |
| Pop (extract) | O(log n) | O(1) |
| Peek | O(1) | O(1) |
| Heapify | O(n) | O(1) |
| Build heap | O(n) | O(1) |

---

## Essential Patterns & Techniques

### 1. Basic Heap Operations

**Template Code**:
```python
import heapq

def heap_operations():
    """Basic heap operations"""
    # Create heap
    heap = [5, 3, 7, 1]
    heapq.heapify(heap)  # O(n)

    # Insert
    heapq.heappush(heap, 2)  # O(log n)

    # Extract min
    min_val = heapq.heappop(heap)  # O(log n)

    # Peek
    min_val = heap[0]  # O(1)

    # K largest
    largest_k = heapq.nlargest(2, heap)

    # K smallest
    smallest_k = heapq.nsmallest(2, heap)

# Max-heap implementation
class MaxHeap:
    def __init__(self):
        self.heap = []

    def push(self, val: int) -> None:
        heapq.heappush(self.heap, -val)

    def pop(self) -> int:
        return -heapq.heappop(self.heap)

    def peek(self) -> int:
        return -self.heap[0]
```

---

### 2. Heap Sort

**Template Code**:
```python
def heapSort(arr: list[int]) -> list[int]:
    """
    Sort using heap.

    Approach:
    1. Build max-heap
    2. Repeatedly extract max

    Time: O(n log n), Space: O(1) in-place
    """
    import heapq

    # Build min-heap of negatives (simulates max-heap)
    heap = [-x for x in arr]
    heapq.heapify(heap)

    result = []
    while heap:
        result.append(-heapq.heappop(heap))

    return result
```

---

### 3. K Largest/Smallest Elements

**Template Code**:
```python
def findKthLargest(nums: list[int], k: int) -> int:
    """
    Find kth largest element.

    Approach: Min-heap of size k
    - Keep k largest elements
    - Root is kth largest

    Time: O(n log k), Space: O(k)
    """
    import heapq

    # Min-heap of k largest
    heap = nums[:k]
    heapq.heapify(heap)

    for i in range(k, len(nums)):
        if nums[i] > heap[0]:
            heapq.heapreplace(heap, nums[i])

    return heap[0]

def topKFrequent(nums: list[int], k: int) -> list[int]:
    """
    Find k most frequent elements.

    Approach: Heap with frequency count
    1. Count frequencies
    2. Use min-heap of k frequent
    3. Return elements

    Time: O(n log k), Space: O(n)
    """
    from collections import Counter
    import heapq

    freq = Counter(nums)

    # Min-heap by frequency
    heap = []
    for num, count in freq.items():
        if len(heap) < k:
            heapq.heappush(heap, (count, num))
        elif count > heap[0][0]:
            heapq.heapreplace(heap, (count, num))

    return [x[1] for x in heap]
```

---

### 4. Merge K Sorted Lists

**Template Code**:
```python
def mergeKLists(lists: list[Optional[ListNode]]) -> Optional[ListNode]:
    """
    Merge k sorted linked lists.

    Approach: Min-heap of list heads
    1. Push head of each list to min-heap
    2. Pop minimum, add to result, push next
    3. Repeat until heap empty

    Time: O(n log k) where n = total nodes, k = lists
    Space: O(k)
    """
    import heapq

    min_heap = []
    dummy = ListNode(0)
    current = dummy

    # Add first node of each list
    for i, lst in enumerate(lists):
        if lst:
            heapq.heappush(min_heap, (lst.val, i, lst))

    while min_heap:
        val, idx, node = heapq.heappop(min_heap)
        current.next = node
        current = current.next

        if node.next:
            heapq.heappush(min_heap, (node.next.val, idx, node.next))

    return dummy.next
```

---

## Problem-Solving Framework

### When to Use Heap
- Find kth largest/smallest
- Priority queue problems
- Merge sorted sequences
- Scheduling/load balancing
- Median finding

### Pattern Recognition
```
Is this about...?
├─ Kth element? → Heap of size k
├─ Top k items? → Heap by frequency
├─ Merging? → Min-heap of candidates
└─ Priority? → Priority queue (heap)
```

---

## Common Pitfalls & Edge Cases

### 1. Max-Heap in Python
```python
# ❌ Wrong - Python only has min-heap
heapq.heappush(heap, 5)

# ✅ Correct - negate for max-heap
heapq.heappush(heap, -5)
max_val = -heapq.heappop(heap)
```

### 2. Heap with Tuples
```python
# ❌ Wrong - tuple comparison might fail
heapq.heappush(heap, (value, node))  # Can't compare nodes

# ✅ Correct - add counter for tiebreaker
counter = 0
heapq.heappush(heap, (value, counter, node))
counter += 1
```

### 3. Out of Order Popping
```python
# ❌ Wrong - heap order not maintained
arr = [5, 3, 7]
arr[0] = 1  # Breaks heap property

# ✅ Correct - use heappop/heappush
heapq.heappop(arr)
heapq.heappush(arr, 1)
```

---

## Curated LeetCode Problems

### Beginner Level

#### 1. Kth Largest Element in Array (LeetCode #215)
```python
def findKthLargest(nums: list[int], k: int) -> int:
    import heapq
    heap = nums[:k]
    heapq.heapify(heap)
    for i in range(k, len(nums)):
        if nums[i] > heap[0]:
            heapq.heapreplace(heap, nums[i])
    return heap[0]
```

---

### Intermediate Level

#### 2. Top K Frequent Elements (LeetCode #347)
```python
def topKFrequent(nums: list[int], k: int) -> list[int]:
    from collections import Counter
    import heapq
    freq = Counter(nums)
    heap = [(-count, num) for num, count in freq.items()]
    heapq.heapify(heap)
    return [heapq.heappop(heap)[1] for _ in range(k)]
```

---

#### 3. Merge K Sorted Lists (LeetCode #23)
```python
# See implementation above
```

---

### Advanced Level

#### 4. Find Median from Data Stream (LeetCode #295)
```python
import heapq

class MedianFinder:
    def __init__(self):
        self.small = []  # Max heap (negate)
        self.large = []  # Min heap

    def addNum(self, num: int) -> None:
        heapq.heappush(self.small, -num)

        if self.small and self.large and (-self.small[0] > self.large[0]):
            val = -heapq.heappop(self.small)
            heapq.heappush(self.large, val)

        if len(self.small) > len(self.large) + 1:
            val = -heapq.heappop(self.small)
            heapq.heappush(self.large, val)

        if len(self.large) > len(self.small):
            val = heapq.heappop(self.large)
            heapq.heappush(self.small, -val)

    def findMedian(self) -> float:
        if len(self.small) > len(self.large):
            return float(-self.small[0])
        return (-self.small[0] + self.large[0]) / 2.0
```

---

## Practice Roadmap

### Beginner
- LeetCode #215: Kth Largest Element
- LeetCode #692: Top K Frequent Words
- LeetCode #1337: The K Weakest Rows in a Matrix

### Intermediate
- LeetCode #347: Top K Frequent Elements
- LeetCode #23: Merge K Sorted Lists
- LeetCode #1046: Last Stone Weight

### Advanced
- LeetCode #295: Find Median from Data Stream
- LeetCode #480: Sliding Window Median
- LeetCode #1439: Find the Kth Smallest Sum

---

## Quick Reference

```python
import heapq

# Min-heap
heap = []
heapq.heappush(heap, x)      # Insert
x = heapq.heappop(heap)      # Extract min
x = heap[0]                  # Peek
heapq.heapify(arr)           # Build heap

# Max-heap (negate)
heapq.heappush(heap, -x)
x = -heapq.heappop(heap)

# K largest/smallest
heapq.nlargest(k, arr)
heapq.nsmallest(k, arr)
```

---

## Key Takeaways

1. **Heap for top k problems** - O(n log k) is efficient
2. **Min-heap default in Python** - Negate for max-heap
3. **Use heappush/heappop** - Maintain heap property
4. **Complete binary tree** - All levels filled
5. **Array representation** - Parent at i, children at 2i+1, 2i+2
6. **Heapify for existing array** - O(n) build time
7. **Priority queue implementation** - Use heap underneath

