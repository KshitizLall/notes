# Sorting & Searching - Comprehensive DSA Notes

## Core Concepts

### What is Sorting?
**Simple Definition**: Arranging elements in a specific order (ascending/descending).

**Real-world Analogy**: Like organizing books on a shelf by title or arranging students by height.

### Searching Algorithms
- **Linear Search**: Check each element - O(n)
- **Binary Search**: On sorted data, eliminate half each step - O(log n)

### Time Complexity Comparison

| Algorithm | Best | Average | Worst | Space | Stable |
|-----------|------|---------|-------|-------|--------|
| Bubble Sort | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Selection Sort | O(n²) | O(n²) | O(n²) | O(1) | No |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) | No |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | No |
| Counting Sort | O(n+k) | O(n+k) | O(n+k) | O(k) | Yes |
| Radix Sort | O(nk) | O(nk) | O(nk) | O(n+k) | Yes |

---

## Essential Patterns & Techniques

### 1. Binary Search

**Template Code**:
```python
def binary_search(arr: list[int], target: int) -> int:
    """
    Find target in sorted array, return index or -1.

    Approach: Eliminate half at each step
    - Set left=0, right=n-1
    - While left <= right:
      - mid = (left + right) // 2
      - If arr[mid] == target, found
      - If arr[mid] < target, go right
      - If arr[mid] > target, go left

    Time: O(log n), Space: O(1)
    """
    left, right = 0, len(arr) - 1

    while left <= right:
        mid = (left + right) // 2

        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1

    return -1

def binary_search_left(arr: list[int], target: int) -> int:
    """Find leftmost position of target"""
    left, right = 0, len(arr)

    while left < right:
        mid = (left + right) // 2
        if arr[mid] < target:
            left = mid + 1
        else:
            right = mid

    return left

def binary_search_right(arr: list[int], target: int) -> int:
    """Find rightmost position of target"""
    left, right = 0, len(arr)

    while left < right:
        mid = (left + right) // 2
        if arr[mid] <= target:
            left = mid + 1
        else:
            right = mid

    return left - 1
```

---

### 2. Quick Sort

**Template Code**:
```python
def quick_sort(arr: list[int], low: int = 0, high: int = None) -> None:
    """
    In-place quick sort using partition.

    Average: O(n log n), Worst: O(n²)
    Space: O(log n) recursion
    """
    if high is None:
        high = len(arr) - 1

    if low < high:
        pi = partition(arr, low, high)
        quick_sort(arr, low, pi - 1)
        quick_sort(arr, pi + 1, high)

def partition(arr: list[int], low: int, high: int) -> int:
    """Partition around pivot"""
    pivot = arr[high]
    i = low - 1

    for j in range(low, high):
        if arr[j] < pivot:
            i += 1
            arr[i], arr[j] = arr[j], arr[i]

    arr[i + 1], arr[high] = arr[high], arr[i + 1]
    return i + 1
```

---

### 3. Merge Sort

**Template Code**:
```python
def merge_sort(arr: list[int]) -> list[int]:
    """
    Stable sort using divide-and-conquer.

    Time: O(n log n), Space: O(n)
    """
    if len(arr) <= 1:
        return arr

    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])

    return merge(left, right)

def merge(left: list[int], right: list[int]) -> list[int]:
    """Merge two sorted arrays"""
    result = []
    i = j = 0

    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1

    result.extend(left[i:])
    result.extend(right[j:])
    return result
```

---

### 4. Finding Kth Element

**Template Code**:
```python
def find_kth_largest(nums: list[int], k: int) -> int:
    """
    Find kth largest element (k=1 is max).

    Approach: Min-heap of size k
    - Keep k largest elements
    - Top of heap is kth largest

    Time: O(n log k), Space: O(k)
    """
    import heapq
    return heapq.nlargest(k, nums)[-1]

def find_kth_quickselect(arr: list[int], k: int) -> int:
    """
    Using quickselect (similar to quicksort partition).

    Average: O(n), Worst: O(n²)
    Space: O(1)
    """
    def select(low, high, k_smallest):
        if low == high:
            return arr[low]

        pi = partition(arr, low, high)

        if k_smallest == pi:
            return arr[pi]
        elif k_smallest < pi:
            return select(low, pi - 1, k_smallest)
        else:
            return select(pi + 1, high, k_smallest)

    return select(0, len(arr) - 1, len(arr) - k)
```

---

## Problem-Solving Framework

### Choose Algorithm By:
1. **Is data sorted?**
   - Yes → Binary search
   - No → Need to sort first?

2. **Space constraints?**
   - Limited → Quick sort (in-place)
   - Enough → Merge sort (stable)

3. **Stability matters?**
   - Yes → Merge sort, insertion sort
   - No → Quick sort, heap sort

4. **What's the data?**
   - Integers with range → Counting/radix sort
   - Generic → Merge/quick sort

---

## Common Pitfalls & Edge Cases

### 1. Binary Search Boundary Errors
```python
# ❌ Wrong boundary
mid = (left + right) // 2  # Can overflow in some languages
left = mid  # Infinite loop if arr[mid] == target

# ✅ Correct
mid = left + (right - left) // 2  # Avoids overflow
left = mid + 1  # Move past checked element
```

### 2. Partition in QuickSort
```python
# ❌ Wrong - might not partition correctly
pivot = arr[0]  # Can be bad choice

# ✅ Better - random or median pivot
pivot = arr[random.randint(0, len(arr) - 1)]
```

### 3. Off-by-One in Merge
```python
# ❌ Wrong - misses last element
result.extend(left[i:])  # Should also check right

# ✅ Correct
result.extend(left[i:])
result.extend(right[j:])
```

---

## Curated LeetCode Problems

### Beginner Level

#### 1. Binary Search (LeetCode #704)
**Pattern**: Standard Binary Search
**Key Insight**: Always search in sorted array

```python
def search(nums: list[int], target: int) -> int:
    left, right = 0, len(nums) - 1
    while left <= right:
        mid = (left + right) // 2
        if nums[mid] == target:
            return mid
        elif nums[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return -1
```

---

#### 2. First Bad Version (LeetCode #278)
**Pattern**: Binary Search with Condition
**Key Insight**: Find first version where condition is true

```python
def firstBadVersion(n: int) -> int:
    left, right = 1, n
    while left < right:
        mid = (left + right) // 2
        if isBadVersion(mid):
            right = mid
        else:
            left = mid + 1
    return left
```

---

### Intermediate Level

#### 3. Kth Largest Element in Array (LeetCode #215)
**Pattern**: Heap/QuickSelect
**Key Insight**: Use min-heap of size k

```python
def findKthLargest(nums: list[int], k: int) -> int:
    import heapq
    return heapq.nlargest(k, nums)[-1]
```

---

#### 4. Search in Rotated Sorted Array (LeetCode #33)
**Pattern**: Modified Binary Search
**Key Insight**: Identify which half is sorted

```python
def search(nums: list[int], target: int) -> int:
    left, right = 0, len(nums) - 1
    while left <= right:
        mid = (left + right) // 2
        if nums[mid] == target:
            return mid
        if nums[left] <= nums[mid]:  # Left half sorted
            if nums[left] <= target < nums[mid]:
                right = mid - 1
            else:
                left = mid + 1
        else:  # Right half sorted
            if nums[mid] < target <= nums[right]:
                left = mid + 1
            else:
                right = mid - 1
    return -1
```

---

### Advanced Level

#### 5. Median of Two Sorted Arrays (LeetCode #4)
**Pattern**: Binary Search on Solution Space
**Key Insight**: Find partition point

```python
def findMedianSortedArrays(nums1: list[int], nums2: list[int]) -> float:
    if len(nums1) > len(nums2):
        nums1, nums2 = nums2, nums1

    low, high = 0, len(nums1)
    total = len(nums1) + len(nums2)

    while low <= high:
        partition1 = (low + high) // 2
        partition2 = (total + 1) // 2 - partition1

        left1 = float('-inf') if partition1 == 0 else nums1[partition1 - 1]
        right1 = float('inf') if partition1 == len(nums1) else nums1[partition1]
        left2 = float('-inf') if partition2 == 0 else nums2[partition2 - 1]
        right2 = float('inf') if partition2 == len(nums2) else nums2[partition2]

        if left1 <= right2 and left2 <= right1:
            if total % 2 == 0:
                return (max(left1, left2) + min(right1, right2)) / 2
            else:
                return max(left1, left2)
        elif left1 > right2:
            high = partition1 - 1
        else:
            low = partition1 + 1

    return -1
```

---

## Practice Roadmap

### Beginner
- LeetCode #704: Binary Search
- LeetCode #278: First Bad Version
- LeetCode #35: Search Insert Position

### Intermediate
- LeetCode #33: Search in Rotated Sorted Array
- LeetCode #215: Kth Largest Element
- LeetCode #34: Find First and Last Position of Element

### Advanced
- LeetCode #4: Median of Two Sorted Arrays
- LeetCode #315: Count of Smaller Numbers After Self
- LeetCode #1157: Online Majority Element In Subarray

---

## Quick Reference

```python
# Binary search
left, right = 0, len(arr) - 1
while left <= right:
    mid = (left + right) // 2
    if arr[mid] == target:
        return mid
    elif arr[mid] < target:
        left = mid + 1
    else:
        right = mid - 1

# Quick sort partition
def partition(arr, low, high):
    pivot = arr[high]
    i = low - 1
    for j in range(low, high):
        if arr[j] < pivot:
            i += 1
            arr[i], arr[j] = arr[j], arr[i]
    arr[i + 1], arr[high] = arr[high], arr[i + 1]
    return i + 1
```

---

## Key Takeaways

1. **Binary search requires sorted data**
2. **Choose sorting algorithm by constraints** - Space, stability, best case
3. **Merge sort is stable** - Preserves original order for equal elements
4. **QuickSort is faster on average** - But worst case is O(n²)
5. **For finding Kth element** - Use heap for O(n log k) or quickselect for O(n)
6. **Be careful with boundaries** - Off-by-one errors common
7. **Consider custom comparators** - For sorting complex objects

