# Two Pointers - Comprehensive DSA Notes

## Core Concepts

### What is Two Pointers?
**Simple Definition**: Using two pointers moving in the same or opposite directions to solve problems efficiently.

**Real-world Analogy**: Like two people starting from opposite ends of a meeting line - they walk towards each other and meet somewhere in the middle.

### Pointer Movement Strategies

| Strategy | Direction | Use Case | Example |
|----------|-----------|----------|---------|
| Opposite | Left ← → Right | Palindromes, target sums | Valid palindrome |
| Same | Both → | Removing duplicates, window | Remove duplicates |
| Slow/Fast | Slow → Fast ↦ | Cycle detection, middle | Find middle |

---

## Essential Patterns & Techniques

### 1. Opposite Direction (Convergent)

**Template Code**:
```python
def twoSum(arr: list[int], target: int) -> tuple:
    """
    Find two numbers summing to target (sorted array).

    Approach: Two pointers from opposite ends
    - If sum too small, move left right
    - If sum too large, move right left
    - If equal, found pair

    Time: O(n), Space: O(1)
    """
    left, right = 0, len(arr) - 1

    while left < right:
        current_sum = arr[left] + arr[right]

        if current_sum == target:
            return (arr[left], arr[right])
        elif current_sum < target:
            left += 1
        else:
            right -= 1

    return None

def isPalindrome(s: str) -> bool:
    """
    Check if string is palindrome (ignoring non-alphanumeric).

    Approach: Compare from both ends
    - Skip non-alphanumeric
    - Compare characters
    """
    left, right = 0, len(s) - 1

    while left < right:
        # Skip non-alphanumeric on left
        while left < right and not s[left].isalnum():
            left += 1

        # Skip non-alphanumeric on right
        while left < right and not s[right].isalnum():
            right -= 1

        if s[left].lower() != s[right].lower():
            return False

        left += 1
        right -= 1

    return True

def reverseArray(arr: list) -> None:
    """Reverse array in-place"""
    left, right = 0, len(arr) - 1

    while left < right:
        arr[left], arr[right] = arr[right], arr[left]
        left += 1
        right -= 1
```

---

### 2. Same Direction (Divergent)

**Template Code**:
```python
def removeDuplicates(nums: list[int]) -> int:
    """
    Remove duplicates in-place from sorted array.

    Approach: Two pointers same direction
    - k tracks position to insert unique
    - i scans through array
    - Only advance k when new element found

    Time: O(n), Space: O(1)
    """
    if not nums:
        return 0

    k = 1
    for i in range(1, len(nums)):
        if nums[i] != nums[i - 1]:
            nums[k] = nums[i]
            k += 1

    return k

def moveZeroes(nums: list[int]) -> None:
    """
    Move all zeros to end (in-place).

    Approach: Two pointer, one for position, one for scan
    """
    left = 0

    for right in range(len(nums)):
        if nums[right] != 0:
            nums[left], nums[right] = nums[right], nums[left]
            left += 1

def sortArrayByParity(nums: list[int]) -> list[int]:
    """
    Move all even numbers before odd numbers.

    Approach: Two pointers, swap when even/odd meet
    """
    left, right = 0, len(nums) - 1

    while left < right:
        # Find odd from left
        while left < right and nums[left] % 2 == 0:
            left += 1

        # Find even from right
        while left < right and nums[right] % 2 == 1:
            right -= 1

        # Swap
        nums[left], nums[right] = nums[right], nums[left]

    return nums
```

---

### 3. Slow/Fast Pointers

**Template Code**:
```python
def hasCycle(head: Optional[ListNode]) -> bool:
    """
    Detect cycle in linked list (Floyd's algorithm).

    Approach: Slow moves 1 step, fast moves 2 steps
    - If cycle, they eventually meet
    - If no cycle, fast reaches end

    Time: O(n), Space: O(1)
    """
    if not head:
        return False

    slow = fast = head

    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

        if slow == fast:
            return True

    return False

def findMiddle(head: Optional[ListNode]) -> Optional[ListNode]:
    """Find middle node of linked list"""
    slow = fast = head

    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

    return slow
```

---

## Problem-Solving Framework

### Pattern Recognition

```
Is this about...?
├─ Finding pairs? → Opposite pointers
├─ In-place modification? → Same direction
├─ Linked list property? → Slow/fast
├─ Palindrome check? → Opposite pointers
└─ Sorted container? → Likely two pointers
```

### Decision Tree

```
Are pointers moving...?
├─ Towards each other? → Opposite
├─ In same direction? → Same (slow/fast or window)
└─ Both start same? → Same direction
```

---

## Common Pitfalls & Edge Cases

### 1. Incorrect Loop Condition
```python
# ❌ Wrong - might miss last element
while left < right:  # left == right not processed

# ✅ Correct
while left <= right:  # Include when pointers meet
```

### 2. Moving Pointer at Wrong Time
```python
# ❌ Wrong - compares after moving
left += 1
if arr[left] == target:  # Wrong element!

# ✅ Correct - compare before or use adjusted index
if arr[left] == target:
    result = arr[left]
left += 1
```

### 3. Forgetting to Skip Non-Target Elements
```python
# ❌ Wrong - doesn't skip irrelevant
while left < right:
    result = arr[left]
    left += 1

# ✅ Correct - skip until condition met
while left < right and not is_valid(arr[left]):
    left += 1
```

---

## Comparison Table

| Pattern | Movement | Time | Space | Use |
|---------|----------|------|-------|-----|
| Opposite | ← → | O(n) | O(1) | Sorted arrays |
| Same | → → | O(n) | O(1) | In-place mods |
| Slow/Fast | → ↦ | O(n) | O(1) | Linked lists |
| Sliding | → → → | O(n) | O(k) | Subarray |

---

## Curated LeetCode Problems

### Beginner Level

#### 1. Two Sum II - Input Array Is Sorted (LeetCode #167)
**Pattern**: Opposite pointers, target sum
**Key Insight**: Array sorted, use opposite ends

```python
def twoSum(numbers: list[int], target: int) -> list[int]:
    left, right = 0, len(numbers) - 1
    while left < right:
        current_sum = numbers[left] + numbers[right]
        if current_sum == target:
            return [left + 1, right + 1]
        elif current_sum < target:
            left += 1
        else:
            right -= 1
    return []
```

---

#### 2. Valid Palindrome (LeetCode #125)
**Pattern**: Opposite pointers, character validation
**Key Insight**: Compare from both ends

```python
def isPalindrome(s: str) -> bool:
    left, right = 0, len(s) - 1
    while left < right:
        while left < right and not s[left].isalnum():
            left += 1
        while left < right and not s[right].isalnum():
            right -= 1
        if s[left].lower() != s[right].lower():
            return False
        left += 1
        right -= 1
    return True
```

---

### Intermediate Level

#### 3. Remove Element (LeetCode #27)
**Pattern**: Same direction pointers, in-place
**Key Insight**: k pointer for write position

```python
def removeElement(nums: list[int], val: int) -> int:
    k = 0
    for i in range(len(nums)):
        if nums[i] != val:
            nums[k] = nums[i]
            k += 1
    return k
```

---

#### 4. Container with Most Water (LeetCode #11)
**Pattern**: Opposite pointers, greedy
**Key Insight**: Move shorter height inward

```python
def maxArea(height: list[int]) -> int:
    left, right = 0, len(height) - 1
    max_area = 0
    while left < right:
        width = right - left
        current_area = width * min(height[left], height[right])
        max_area = max(max_area, current_area)
        if height[left] < height[right]:
            left += 1
        else:
            right -= 1
    return max_area
```

---

### Advanced Level

#### 5. Trapping Rain Water (LeetCode #42)
**Pattern**: Opposite pointers with dynamic computation
**Key Insight**: Track max from both sides

```python
def trap(height: list[int]) -> int:
    """Water trapped = min(left_max, right_max) - current_height"""
    left, right = 0, len(height) - 1
    left_max, right_max = 0, 0
    water = 0
    while left < right:
        if height[left] < height[right]:
            if height[left] >= left_max:
                left_max = height[left]
            else:
                water += left_max - height[left]
            left += 1
        else:
            if height[right] >= right_max:
                right_max = height[right]
            else:
                water += right_max - height[right]
            right -= 1
    return water
```

---

## Practice Roadmap

### Beginner
- LeetCode #167: Two Sum II
- LeetCode #125: Valid Palindrome
- LeetCode #27: Remove Element

### Intermediate
- LeetCode #11: Container with Most Water
- LeetCode #28: Find Index of Substring
- LeetCode #141: Linked List Cycle

### Advanced
- LeetCode #42: Trapping Rain Water
- LeetCode #86: Partition List
- LeetCode #844: Backspace String Compare

---

## Quick Reference

```python
# Opposite pointers (sorted array)
left, right = 0, len(arr) - 1
while left < right:
    if condition:
        left += 1
    else:
        right -= 1

# Same direction (in-place modification)
j = 0
for i in range(len(arr)):
    if condition(arr[i]):
        arr[j] = arr[i]
        j += 1

# Slow/Fast (linked list)
slow = fast = head
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
```

---

## Key Takeaways

1. **Sorted arrays** → Opposite pointers work great
2. **In-place modification** → Same direction pointers
3. **Linked lists** → Slow/fast for cycles, middle
4. **Greedy strategy** → Move pointer based on values
5. **Index tracking** → Keep write/read positions separate
6. **Space efficiency** → Two pointers = O(1) extra space
7. **Time efficiency** → O(n) single pass solutions
8. **Master all three patterns** → Different problem types

