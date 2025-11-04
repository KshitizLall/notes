# Sliding Window - Comprehensive DSA Notes

## Core Concepts

### What is Sliding Window?
**Simple Definition**: A technique using two pointers to track a contiguous window of elements, sliding it across the array.

**Real-world Analogy**: Like moving a magnifying glass across text - examining a fixed range at a time, shifting the view as you progress.

### Window Types

| Type | Movement | Use Case |
|------|----------|----------|
| Fixed Size | Slide right, maintain size | Max sum in window |
| Variable Size | Expand/contract as needed | Minimum window containing substring |
| Two Pointer | Pointers from both ends | Two sum in sorted array |

---

## Essential Patterns & Techniques

### 1. Fixed Size Window

**Template Code**:
```python
def maxSumSubarray(arr: list[int], k: int) -> int:
    """
    Find maximum sum of subarray of size k.

    Approach: Fixed-size sliding window
    1. Create window of first k elements
    2. Slide right, remove left, add right
    3. Track maximum

    Time: O(n), Space: O(1)
    """
    if k > len(arr):
        return 0

    # Initial window sum
    window_sum = sum(arr[:k])
    max_sum = window_sum

    for i in range(k, len(arr)):
        window_sum = window_sum - arr[i - k] + arr[i]
        max_sum = max(max_sum, window_sum)

    return max_sum

def avgSubarray(arr: list[int], k: int) -> list[float]:
    """Average of subarray of size k"""
    if k > len(arr):
        return []

    result = []
    window_sum = sum(arr[:k])
    result.append(window_sum / k)

    for i in range(k, len(arr)):
        window_sum = window_sum - arr[i - k] + arr[i]
        result.append(window_sum / k)

    return result
```

---

### 2. Variable Size Window

**Template Code**:
```python
def minWindow(s: str, t: str) -> str:
    """
    Find minimum window substring containing all characters in t.

    Approach: Variable-size sliding window
    1. Expand window until all characters found
    2. Contract window while all characters still present
    3. Track minimum window

    Time: O(|s| + |t|), Space: O(alphabet_size)
    """
    if not s or not t:
        return ""

    need = {}
    for char in t:
        need[char] = need.get(char, 0) + 1

    required = len(need)
    window_counts = {}
    formed = 0
    left = right = 0
    result = (float('inf'), None, None)

    while right < len(s):
        # Expand window
        char = s[right]
        window_counts[char] = window_counts.get(char, 0) + 1

        if char in need and window_counts[char] == need[char]:
            formed += 1

        # Contract window
        while left <= right and formed == required:
            if right - left + 1 < result[0]:
                result = (right - left + 1, left, right)

            char = s[left]
            window_counts[char] -= 1
            if char in need and window_counts[char] < need[char]:
                formed -= 1

            left += 1

        right += 1

    return "" if result[0] == float('inf') else s[result[1]:result[2] + 1]

def lengthOfLongestSubstring(s: str) -> int:
    """
    Longest substring without repeating characters.

    Approach: Variable window with hash map
    - Expand right, track character positions
    - When duplicate found, move left past previous occurrence
    """
    char_index = {}
    max_len = 0
    left = 0

    for right in range(len(s)):
        char = s[right]

        if char in char_index and char_index[char] >= left:
            left = char_index[char] + 1

        char_index[char] = right
        max_len = max(max_len, right - left + 1)

    return max_len
```

---

### 3. Pattern Matching Window

**Template Code**:
```python
def findAnagrams(s: str, p: str) -> list[int]:
    """
    Find all positions where anagram of p starts in s.

    Approach: Fixed-size window with frequency matching
    1. Window size = len(p)
    2. Compare character frequencies
    3. When match, add to result

    Time: O(n), Space: O(1) constant alphabet
    """
    if len(p) > len(s):
        return []

    from collections import Counter

    p_count = Counter(p)
    window_count = Counter(s[:len(p)])
    result = []

    if p_count == window_count:
        result.append(0)

    for i in range(len(p), len(s)):
        # Add right character
        window_count[s[i]] = window_count.get(s[i], 0) + 1

        # Remove left character
        left_char = s[i - len(p)]
        window_count[left_char] -= 1
        if window_count[left_char] == 0:
            del window_count[left_char]

        # Check if match
        if p_count == window_count:
            result.append(i - len(p) + 1)

    return result
```

---

## Problem-Solving Framework

### Step 1: Identify Window Type
- **Fixed Size**: Known window length
- **Variable Size**: Find optimal window

### Step 2: Choose Pointer Strategy
- **Expansion Phase**: Grow window to include necessary elements
- **Contraction Phase**: Shrink window to find optimal

### Step 3: Track Window State
- Use hash map for character frequencies
- Use set for unique elements
- Update state efficiently

### Step 4: Move Pointers Correctly
- **Right pointer**: Always moves right (expand)
- **Left pointer**: Moves right when needed (contract)

---

## Common Pitfalls & Edge Cases

### 1. Off-by-One in Window Size
```python
# ❌ Wrong - wrong window calculation
window_sum = sum(arr[left:right])

# ✅ Correct - include both endpoints
window_sum = sum(arr[left:right + 1])
```

### 2. Forgetting to Update on Move
```python
# ❌ Wrong - removes wrong element
left += 1  # Move left
# Don't subtract arr[left - 1]

# ✅ Correct - update before moving
window_sum -= arr[left]
left += 1
```

### 3. Not Handling Empty Window
```python
# ❌ Wrong - crashes with empty
result = min(window_sizes)

# ✅ Correct - check if any valid window exists
if not valid_windows:
    return -1
result = min(window_sizes)
```

---

## Comparison Table

| Pattern | Window Type | Use Case | Example |
|---------|------------|----------|---------|
| Fixed | Fixed size | Max sum in window | Max sum of k elements |
| Variable expand | Expand until condition | Minimum window | Min window substring |
| Variable contract | Contract when condition | Maximum window | Max consecutive 1s |
| Two pointer | Both ends | Pairing | Two sum |

---

## Curated LeetCode Problems

### Beginner Level

#### 1. Maximum Average Subarray I (LeetCode #643)
**Pattern**: Fixed-size sliding window
**Key Insight**: Slide window, track sum

```python
def findMaxAverage(nums: list[int], k: int) -> float:
    window_sum = sum(nums[:k])
    max_sum = window_sum
    for i in range(k, len(nums)):
        window_sum += nums[i] - nums[i - k]
        max_sum = max(max_sum, window_sum)
    return max_sum / k
```

---

#### 2. Longest Substring Without Repeating Characters (LeetCode #3)
**Pattern**: Variable-size window with hash map
**Key Insight**: Move left when duplicate found

```python
def lengthOfLongestSubstring(s: str) -> int:
    char_index = {}
    max_len = 0
    left = 0
    for right in range(len(s)):
        if s[right] in char_index and char_index[s[right]] >= left:
            left = char_index[s[right]] + 1
        char_index[s[right]] = right
        max_len = max(max_len, right - left + 1)
    return max_len
```

---

### Intermediate Level

#### 3. Minimum Window Substring (LeetCode #76)
**Pattern**: Variable-size window with expansion/contraction
**Key Insight**: Expand to find, contract to minimize

```python
# See full implementation in "2. Variable Size Window"
```

---

#### 4. Find All Anagrams in a String (LeetCode #438)
**Pattern**: Fixed-size window with frequency matching
**Key Insight**: Compare character frequencies

```python
# See implementation in "3. Pattern Matching Window"
```

---

### Advanced Level

#### 5. Sliding Window Maximum (LeetCode #239)
**Pattern**: Window with Deque for maximum tracking
**Key Insight**: Use deque to track potential maximums

```python
def maxSlidingWindow(nums: list[int], k: int) -> list[int]:
    """
    Maximum in each sliding window of size k.

    Approach: Deque to track decreasing indices
    - Keep deque of indices in decreasing value order
    - Remove indices outside window
    - Front of deque is current maximum
    """
    from collections import deque

    deq = deque()
    result = []

    for i in range(len(nums)):
        # Remove indices outside window
        while deq and deq[0] < i - k + 1:
            deq.popleft()

        # Remove smaller elements (they can't be max)
        while deq and nums[deq[-1]] < nums[i]:
            deq.pop()

        deq.append(i)

        if i >= k - 1:
            result.append(nums[deq[0]])

    return result
```

---

## Practice Roadmap

### Beginner
- LeetCode #643: Maximum Average Subarray I
- LeetCode #1456: Maximum Number of Vowels
- LeetCode #1984: Minimum Difference Between Highest and Lowest

### Intermediate
- LeetCode #3: Longest Substring Without Repeating
- LeetCode #438: Find All Anagrams
- LeetCode #76: Minimum Window Substring

### Advanced
- LeetCode #239: Sliding Window Maximum
- LeetCode #480: Sliding Window Median
- LeetCode #1658: Minimum Operations to Reduce X to Zero

---

## Quick Reference

```python
# Fixed-size window
window_sum = sum(arr[:k])
for i in range(k, n):
    window_sum = window_sum - arr[i-k] + arr[i]
    # Process current window

# Variable window
left = 0
for right in range(n):
    # Add arr[right]
    while condition_not_met:
        # Remove arr[left]
        left += 1
    # Process current window

# Frequency tracking
freq = {}
for char in s:
    freq[char] = freq.get(char, 0) + 1
```

---

## Key Takeaways

1. **Two pointers pattern** - Efficient O(n) solution
2. **Fixed vs variable** - Different movement strategies
3. **Track window state** - Use appropriate data structure
4. **Expand and contract** - Right for expansion, left for contraction
5. **Handle edge cases** - Empty windows, duplicates
6. **Practice pointer movement** - Most common mistake area
7. **Consider alternatives** - Sometimes hash set simpler than map

