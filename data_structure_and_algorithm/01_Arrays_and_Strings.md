# Arrays & Strings - Comprehensive DSA Notes

## Core Concepts

### What are Arrays?
**Simple Definition**: A container that holds multiple items of the same type in a fixed sequence.

**Real-world Analogy**: Think of an array like a row of numbered mailboxes. Each mailbox has a unique address (index), and you can quickly access any mailbox if you know its number.

```python
# Array (List in Python)
numbers = [10, 20, 30, 40, 50]
#          0   1   2   3   4  <- indices
```

### What are Strings?
**Simple Definition**: A sequence of characters stored together.

**Real-world Analogy**: A string is like a chain of beads, where each bead is a character. You can access individual characters by their position in the chain.

```python
# String
text = "HELLO"
#       01234  <- indices
```

### Time and Space Complexity Fundamentals

| Operation | Time Complexity | Space Complexity | Why? |
|-----------|-----------------|------------------|------|
| Access by index | O(1) | O(1) | Direct memory access |
| Search (unsorted) | O(n) | O(1) | Must check each element |
| Insert at end | O(1) | O(1) | Just append |
| Insert at beginning | O(n) | O(1) | Must shift all elements |
| Delete at end | O(1) | O(1) | Just remove last |
| Delete at beginning | O(n) | O(1) | Must shift all elements |

### When to Use Arrays
✅ Random access needed frequently
✅ Cache-friendly (contiguous memory)
✅ Fixed size or predictable growth
✅ Need fast append operations

### When to Avoid Arrays
❌ Frequent insertions/deletions at beginning
❌ Size unknown and varies wildly
❌ Need dynamic resizing frequently

---

## Essential Patterns & Techniques

### 1. Two Pointers Technique

**Description**: Use two pointers moving from opposite ends or same direction to solve problems efficiently.

**Template Code**:
```python
def two_pointers_opposite(arr: list[int]) -> None:
    """Two pointers starting from opposite ends"""
    left = 0
    right = len(arr) - 1

    while left < right:
        # Do something with arr[left] and arr[right]
        if some_condition:
            left += 1
        else:
            right -= 1

def two_pointers_same_direction(arr: list[int]) -> None:
    """Two pointers moving in same direction"""
    slow = 0
    fast = 0

    while fast < len(arr):
        if some_condition:
            slow += 1
        fast += 1
```

**Complexity Analysis**:
- Time: O(n) - each pointer moves at most n times
- Space: O(1) - no extra space

**Visual Example**:
```
Reverse a string: "HELLO" -> "OLLEH"

H E L L O
↑       ↑
H E L L O  -> Swap
↑       ↑

O E L L H
  ↑   ↑
O E L L H  -> Swap
  ↑   ↑

O L L E H
   ↑ ↑
Result: "OLLEH"
```

**Common Variations**:
- Same direction (slow/fast pointers for removing duplicates)
- Opposite direction (for palindromes, merge sorted arrays)

---

### 2. Sliding Window Technique

**Description**: Maintain a window of elements and slide it across the array to solve problems.

**Template Code**:
```python
def sliding_window_fixed_size(arr: list[int], k: int) -> list[int]:
    """Fixed size sliding window"""
    if len(arr) < k:
        return []

    result = []
    window_sum = sum(arr[:k])  # Initial window
    result.append(window_sum)

    for i in range(k, len(arr)):
        window_sum = window_sum - arr[i - k] + arr[i]
        result.append(window_sum)

    return result

def sliding_window_variable_size(s: str, t: str) -> str:
    """Variable size sliding window (e.g., minimum window)"""
    if not s or not t:
        return ""

    need = {}  # Characters we need
    for char in t:
        need[char] = need.get(char, 0) + 1

    required = len(need)
    formed = 0

    window_counts = {}
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
```

**Complexity Analysis**:
- Time: O(n) - each element visited at most twice
- Space: O(k) for fixed, O(alphabet) for variable

**Visual Example**:
```
Find max sum of subarray of size 3: [1, 3, 2, 6, -1, 4, 1, 8]

Window: [1, 3, 2] = 6
        [3, 2, 6] = 11
            [2, 6, -1] = 7
                [6, -1, 4] = 9
                    [-1, 4, 1] = 4
                        [4, 1, 8] = 13  <- Maximum
```

---

### 3. Prefix Sum Technique

**Description**: Precompute cumulative sums to answer range queries in O(1).

**Template Code**:
```python
def prefix_sum(arr: list[int]) -> list[int]:
    """Build prefix sum array"""
    n = len(arr)
    prefix = [0] * (n + 1)

    for i in range(n):
        prefix[i + 1] = prefix[i] + arr[i]

    return prefix

def range_sum(prefix: list[int], left: int, right: int) -> int:
    """Query sum in range [left, right]"""
    return prefix[right + 1] - prefix[left]

# Example usage
arr = [1, 2, 3, 4, 5]
prefix = prefix_sum(arr)
# prefix = [0, 1, 3, 6, 10, 15]

print(range_sum(prefix, 1, 3))  # sum(arr[1:4]) = 2+3+4 = 9
```

**Complexity Analysis**:
- Build: O(n) time, O(n) space
- Query: O(1) time per query

---

### 4. String Pattern Matching (KMP Algorithm)

**Description**: Efficiently find pattern in text using failure function.

**Template Code**:
```python
def build_failure_function(pattern: str) -> list[int]:
    """Build KMP failure function"""
    m = len(pattern)
    failure = [0] * m
    j = 0

    for i in range(1, m):
        while j > 0 and pattern[i] != pattern[j]:
            j = failure[j - 1]
        if pattern[i] == pattern[j]:
            j += 1
        failure[i] = j

    return failure

def kmp_search(text: str, pattern: str) -> int:
    """Find first occurrence of pattern in text"""
    n = len(text)
    m = len(pattern)

    if m == 0:
        return 0

    failure = build_failure_function(pattern)
    j = 0

    for i in range(n):
        while j > 0 and text[i] != pattern[j]:
            j = failure[j - 1]
        if text[i] == pattern[j]:
            j += 1
        if j == m:
            return i - m + 1

    return -1

# Example
print(kmp_search("ABABDABACDABABCABAB", "ABABCABAB"))  # 10
```

**Complexity Analysis**:
- Time: O(n + m) where n = text length, m = pattern length
- Space: O(m) for failure function

---

## Problem-Solving Framework

### Step 1: Understand the Problem
- **Clarify constraints**: Array size? String encoding? Duplicates?
- **Identify input/output**: What needs to be returned?
- **Ask examples**: Work through small examples

### Step 2: Recognize the Pattern
```
Is this a...?
├─ Search problem? → Two pointers or Binary search
├─ Substring problem? → Sliding window or KMP
├─ Range query? → Prefix sum
├─ Sorting related? → Check constraints
└─ Optimization? → Dynamic programming
```

### Step 3: Design Your Approach
1. Start with brute force (understand the problem)
2. Optimize using pattern recognition
3. Consider trade-offs (time vs space)

### Step 4: Code & Test
- Use type hints
- Handle edge cases
- Test with examples

### Step 5: Analyze Complexity
- Calculate time and space
- Compare with alternatives

---

## Common Pitfalls & Edge Cases

### 1. Off-by-One Errors
```python
# ❌ Wrong - doesn't include last element
for i in range(len(arr)):
    if i == len(arr):  # This never happens!
        pass

# ✅ Correct
for i in range(len(arr)):
    pass  # Already includes last element
```

### 2. Empty Input
```python
def find_max(arr: list[int]) -> int:
    if not arr:  # Handle empty array!
        return None
    return max(arr)
```

### 3. String Index Out of Bounds
```python
# ❌ Wrong
text = "HELLO"
print(text[5])  # IndexError

# ✅ Correct
if i < len(text):
    print(text[i])
```

### 4. Mutating While Iterating
```python
# ❌ Wrong - modifies list while iterating
arr = [1, 2, 3, 4, 5]
for i in range(len(arr)):
    arr.pop(i)  # Skips elements!

# ✅ Correct - use two pointers or new list
arr = [x for x in arr if should_keep(x)]
```

### 5. String Immutability
```python
# ❌ Wrong - strings are immutable in Python
s = "hello"
s[0] = 'H'  # TypeError

# ✅ Correct - convert to list
s = list("hello")
s[0] = 'H'
s = ''.join(s)  # "Hello"
```

---

## Optimization Techniques

### Technique 1: Space-Time Trade-off
```python
# Approach 1: O(1) space, O(n²) time (brute force)
def find_duplicate_slow(arr: list[int]) -> int:
    for i in range(len(arr)):
        for j in range(i + 1, len(arr)):
            if arr[i] == arr[j]:
                return arr[i]
    return -1

# Approach 2: O(n) space, O(n) time (hash set)
def find_duplicate_fast(arr: list[int]) -> int:
    seen = set()
    for num in arr:
        if num in seen:
            return num
        seen.add(num)
    return -1
```

### Technique 2: Early Termination
```python
def is_sorted(arr: list[int]) -> bool:
    for i in range(len(arr) - 1):
        if arr[i] > arr[i + 1]:
            return False  # Early exit
    return True
```

### Technique 3: Preprocessing
```python
def two_sum(arr: list[int], target: int) -> tuple:
    # Preprocess: store in hash map
    num_map = {num: i for i, num in enumerate(arr)}

    for i, num in enumerate(arr):
        complement = target - num
        if complement in num_map and num_map[complement] != i:
            return (i, num_map[complement])

    return (-1, -1)
```

---

## Comparison Table

| Approach | Time | Space | Best For | Worst For |
|----------|------|-------|----------|-----------|
| Brute Force | O(n²) | O(1) | Learning | Large inputs |
| Two Pointers | O(n) | O(1) | Sorted arrays | Unsorted |
| Hash Map | O(n) | O(n) | Fast lookup | Memory limited |
| Sliding Window | O(n) | O(k) | Subarray/substring | Fixed patterns |
| Prefix Sum | O(n) + O(1) | O(n) | Range queries | Space limited |
| Binary Search | O(log n) | O(1) | Sorted arrays | Unsorted |

---

## Curated LeetCode Problems

### Beginner Level

#### 1. Two Sum (LeetCode #1)
**Pattern**: Hash Map / Two Pointers (if sorted)
**Key Insight**: Use hash map to store complement of each number

```python
def twoSum(nums: list[int], target: int) -> list[int]:
    """
    Find two numbers that add up to target.

    Approach: Hash map for O(n) solution
    - For each number, check if (target - number) exists
    - Store numbers we've seen

    Time: O(n), Space: O(n)
    """
    num_map = {}

    for i, num in enumerate(nums):
        complement = target - num
        if complement in num_map:
            return [num_map[complement], i]
        num_map[num] = i

    return []

# Test cases
assert twoSum([2, 7, 11, 15], 9) == [0, 1]
assert twoSum([3, 2, 4], 6) == [1, 2]
```

**Alternative Approach**: Two Pointers (if array sorted)
```python
def twoSum_sorted(nums: list[int], target: int) -> list[int]:
    """Two pointers approach for sorted array"""
    nums_sorted = sorted(enumerate(nums), key=lambda x: x[1])
    left, right = 0, len(nums_sorted) - 1

    while left < right:
        current_sum = nums_sorted[left][1] + nums_sorted[right][1]
        if current_sum == target:
            return sorted([nums_sorted[left][0], nums_sorted[right][0]])
        elif current_sum < target:
            left += 1
        else:
            right -= 1

    return []
```

---

#### 2. Valid Palindrome (LeetCode #125)
**Pattern**: Two Pointers
**Key Insight**: Compare characters from both ends, skip non-alphanumeric

```python
def isPalindrome(s: str) -> bool:
    """
    Check if string is palindrome, ignoring non-alphanumeric characters.

    Approach: Two pointers
    - Move from both ends
    - Skip non-alphanumeric characters
    - Compare lowercase characters

    Time: O(n), Space: O(1)
    """
    left = 0
    right = len(s) - 1

    while left < right:
        # Skip non-alphanumeric on left
        while left < right and not s[left].isalnum():
            left += 1

        # Skip non-alphanumeric on right
        while left < right and not s[right].isalnum():
            right -= 1

        # Compare characters (case-insensitive)
        if s[left].lower() != s[right].lower():
            return False

        left += 1
        right -= 1

    return True

# Test cases
assert isPalindrome("A man, a plan, a canal: Panama") == True
assert isPalindrome("race a car") == False
assert isPalindrome(" ") == True
```

---

#### 3. Remove Duplicates from Sorted Array (LeetCode #26)
**Pattern**: Two Pointers
**Key Insight**: Keep pointer at position to insert unique element

```python
def removeDuplicates(nums: list[int]) -> int:
    """
    Remove duplicates in-place from sorted array.

    Approach: Two pointers
    - 'k' points to position of next unique element
    - 'i' scans through array
    - Only advance 'k' when we find a new unique element

    Time: O(n), Space: O(1)
    """
    k = 1  # At least one unique element

    for i in range(1, len(nums)):
        if nums[i] != nums[i - 1]:
            nums[k] = nums[i]
            k += 1

    return k

# Test cases
nums1 = [1, 1, 2]
assert removeDuplicates(nums1) == 2
assert nums1[:2] == [1, 2]

nums2 = [0, 0, 1, 1, 1, 2, 2, 3, 3, 4]
assert removeDuplicates(nums2) == 5
assert nums2[:5] == [0, 1, 2, 3, 4]
```

---

### Intermediate Level

#### 4. Container with Most Water (LeetCode #11)
**Pattern**: Two Pointers
**Key Insight**: Start from ends, move inward based on smaller height

```python
def maxArea(height: list[int]) -> int:
    """
    Find two lines that can hold most water.

    Approach: Two pointers
    - Start from both ends (maximum width)
    - Area = width × min(height[left], height[right])
    - Move the pointer with smaller height inward
    - Why? Moving taller pointer reduces area, moving smaller has chance to increase

    Time: O(n), Space: O(1)
    """
    max_area = 0
    left = 0
    right = len(height) - 1

    while left < right:
        # Calculate current area
        width = right - left
        current_height = min(height[left], height[right])
        current_area = width * current_height
        max_area = max(max_area, current_area)

        # Move the pointer pointing to shorter line
        if height[left] < height[right]:
            left += 1
        else:
            right -= 1

    return max_area

# Test cases
assert maxArea([1, 8, 6, 2, 5, 4, 8, 3, 7]) == 49
assert maxArea([1, 1]) == 1
```

---

#### 5. Longest Substring Without Repeating Characters (LeetCode #3)
**Pattern**: Sliding Window
**Key Insight**: Maintain window of unique characters using hash map

```python
def lengthOfLongestSubstring(s: str) -> int:
    """
    Find length of longest substring without repeating characters.

    Approach: Sliding window with hash map
    - Keep track of last seen index of each character
    - Expand window by moving right pointer
    - When we see duplicate, move left pointer to exclude previous occurrence

    Time: O(n), Space: O(min(n, alphabet_size))
    """
    char_index = {}  # Maps character to its last seen index
    max_length = 0
    left = 0

    for right in range(len(s)):
        char = s[right]

        # If character seen before and is in current window
        if char in char_index and char_index[char] >= left:
            left = char_index[char] + 1  # Move left past previous occurrence

        # Update last seen index
        char_index[char] = right

        # Update max length
        max_length = max(max_length, right - left + 1)

    return max_length

# Test cases
assert lengthOfLongestSubstring("abcabcbb") == 3  # "abc"
assert lengthOfLongestSubstring("bbbbb") == 1     # "b"
assert lengthOfLongestSubstring("pwwkew") == 3    # "wke"
```

---

#### 6. Minimum Window Substring (LeetCode #76)
**Pattern**: Sliding Window
**Key Insight**: Expand window until all characters found, then contract

```python
def minWindow(s: str, t: str) -> str:
    """
    Find minimum window substring containing all characters in t.

    Approach: Variable-size sliding window
    1. Build frequency map of characters needed (t)
    2. Expand right pointer to include characters
    3. When window has all characters, contract left
    4. Track minimum window found

    Time: O(|s| + |t|), Space: O(|t|)
    """
    if not s or not t or len(s) < len(t):
        return ""

    # Build frequency map for characters we need
    dict_t = {}
    for char in t:
        dict_t[char] = dict_t.get(char, 0) + 1

    required = len(dict_t)  # Number of unique characters we need

    # Left and right pointers
    l = r = 0
    formed = 0  # How many unique characters have desired frequency

    # Dictionary to keep count of all characters in current window
    window_counts = {}

    # Answer tuple: (window length, left, right)
    ans = float('inf'), None, None

    while r < len(s):
        # Add character from right to window
        char = s[r]
        window_counts[char] = window_counts.get(char, 0) + 1

        # If frequency of character equals desired frequency
        if char in dict_t and window_counts[char] == dict_t[char]:
            formed += 1

        # Try to contract the window until we lose a required character
        while l <= r and formed == required:
            char = s[l]

            # Save the smallest window
            if r - l + 1 < ans[0]:
                ans = (r - l + 1, l, r)

            # Remove from left of window
            window_counts[char] -= 1
            if char in dict_t and window_counts[char] < dict_t[char]:
                formed -= 1

            l += 1

        r += 1

    # Return the smallest window or empty string
    return "" if ans[0] == float('inf') else s[ans[1]:ans[2] + 1]

# Test cases
assert minWindow("ADOBECODEBANC", "ABC") == "BANC"
assert minWindow("a", "a") == "a"
assert minWindow("a", "aa") == ""
```

---

### Advanced Level

#### 7. Trapping Rain Water (LeetCode #42)
**Pattern**: Two Pointers / Prefix/Suffix Max
**Key Insight**: Water trapped = min(max_left, max_right) - current_height

```python
def trap(height: list[int]) -> int:
    """
    Calculate total water trapped after raining on elevation map.

    Approach: Two pointers with dynamic maximums
    - Keep track of max height seen from left and right
    - For each position, water trapped = min(left_max, right_max) - height[i]
    - Move pointer pointing to smaller max inward

    Time: O(n), Space: O(1)
    """
    if not height or len(height) < 3:
        return 0

    left = 0
    right = len(height) - 1
    left_max = 0
    right_max = 0
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

# Visual example:
# Height: [0, 1, 0, 2, 1, 0, 1, 4, 3, 2, 1, 2, 1]
# Water:  [0, 0, 1, 0, 1, 2, 1, 0, 0, 0, 1, 0, 0] = 6

assert trap([0, 1, 0, 2, 1, 0, 1, 4, 3, 2, 1, 2, 1]) == 6
assert trap([4, 2, 0, 3, 2, 5]) == 9
```

---

#### 8. Longest Palindromic Substring (LeetCode #5)
**Pattern**: Expand Around Center
**Key Insight**: Check palindrome by expanding from center

```python
def longestPalindrome(s: str) -> str:
    """
    Find longest palindromic substring.

    Approach: Expand around center
    - For each possible center (character or between characters)
    - Expand outward while characters match
    - Track longest palindrome found

    Why? Palindromes are symmetric around center
    Time: O(n²), Space: O(1)
    """
    if not s or len(s) < 2:
        return s

    def expand_around_center(left: int, right: int) -> tuple[int, int]:
        """Expand around center and return start, end of palindrome"""
        while left >= 0 and right < len(s) and s[left] == s[right]:
            left -= 1
            right += 1
        # Return valid range (left+1 to right-1)
        return left + 1, right - 1

    start = 0
    max_len = 0

    for i in range(len(s)):
        # Odd length palindromes (center is a character)
        left1, right1 = expand_around_center(i, i)
        len1 = right1 - left1 + 1

        # Even length palindromes (center is between characters)
        left2, right2 = expand_around_center(i, i + 1)
        len2 = right2 - left2 + 1

        # Update if we found longer palindrome
        if len1 > max_len:
            start = left1
            max_len = len1
        if len2 > max_len:
            start = left2
            max_len = len2

    return s[start:start + max_len]

# Test cases
assert longestPalindrome("babad") in ["bab", "aba"]
assert longestPalindrome("cbbd") == "bb"
assert longestPalindrome("a") == "a"
```

---

#### 9. Word Search II (LeetCode #212)
**Pattern**: Backtracking + Trie
**Key Insight**: Use Trie to prune search space efficiently

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.word = None

def findWords(board: list[list[str]], words: list[str]) -> list[str]:
    """
    Find all words from list that exist in board (can move up/down/left/right).

    Approach: Backtracking with Trie
    1. Build Trie from word list
    2. For each cell, do DFS with Trie traversal
    3. If we find a complete word in Trie, add to results
    4. Backtrack by unmarking visited and removing from Trie

    Time: O(m*n*4^l) where l = max word length
    Space: O(k) where k = total characters in words
    """
    # Build Trie
    root = TrieNode()
    for word in words:
        node = root
        for char in word:
            if char not in node.children:
                node.children[char] = TrieNode()
            node = node.children[char]
        node.word = word

    # DFS with backtracking
    def dfs(i: int, j: int, node: TrieNode, visited: set) -> None:
        char = board[i][j]

        # Current path doesn't match any word
        if char not in node.children:
            return

        next_node = node.children[char]

        # Found a word
        if next_node.word:
            results.add(next_node.word)
            next_node.word = None  # Avoid duplicates

        # Mark as visited
        visited.add((i, j))

        # Explore all 4 directions
        for di, dj in [(0, 1), (1, 0), (0, -1), (-1, 0)]:
            ni, nj = i + di, j + dj
            if 0 <= ni < len(board) and 0 <= nj < len(board[0]) and (ni, nj) not in visited:
                dfs(ni, nj, next_node, visited)

        # Backtrack
        visited.remove((i, j))

    results = set()

    for i in range(len(board)):
        for j in range(len(board[0])):
            dfs(i, j, root, set())

    return list(results)

# Test case
board = [["o","a","a"],["e","t","a"],["t","a","t"]]
words = ["oath","pea","eat","eat"]
assert set(findWords(board, words)) == {"eat", "oath"}
```

---

## Practice Roadmap

### Beginner (Foundation Building)
1. **Arrays - Fundamentals**
   - LeetCode #88: Merge Sorted Array
   - LeetCode #27: Remove Element
   - LeetCode #26: Remove Duplicates from Sorted Array

2. **Strings - Basics**
   - LeetCode #344: Reverse String
   - LeetCode #383: Ransom Note
   - LeetCode #205: Isomorphic Strings

3. **Two Pointers - Introduction**
   - LeetCode #167: Two Sum II (sorted array)
   - LeetCode #125: Valid Palindrome
   - LeetCode #977: Squares of a Sorted Array

### Intermediate (Pattern Mastery)
1. **Two Pointers - Advanced**
   - LeetCode #11: Container with Most Water
   - LeetCode #15: 3Sum
   - LeetCode #16: 3Sum Closest

2. **Sliding Window**
   - LeetCode #3: Longest Substring Without Repeating Characters
   - LeetCode #209: Minimum Size Subarray Sum
   - LeetCode #438: Find All Anagrams in a String

3. **Prefix Sum**
   - LeetCode #303: Range Sum Query Immutable
   - LeetCode #304: Range Sum Query 2D Immutable
   - LeetCode #560: Subarray Sum Equals K

### Advanced (Mastery & Optimization)
1. **Complex Patterns**
   - LeetCode #42: Trapping Rain Water
   - LeetCode #5: Longest Palindromic Substring
   - LeetCode #76: Minimum Window Substring

2. **String Algorithms**
   - LeetCode #28: Find the Index of the First Occurrence in a String (KMP)
   - LeetCode #214: Shortest Palindrome
   - LeetCode #212: Word Search II

3. **Combination Problems**
   - LeetCode #986: Interval List Intersections
   - LeetCode #844: Backspace String Compare
   - LeetCode #1679: Max Number of K-Sum Pairs

---

## Quick Reference Cheatsheet

```python
# Two Pointers
left, right = 0, len(arr) - 1
while left < right:
    if condition:
        left += 1
    else:
        right -= 1

# Sliding Window (variable)
left = 0
for right in range(len(arr)):
    # Expand window
    while condition:
        # Contract window
        left += 1

# Prefix Sum
prefix = [0] * (n + 1)
for i in range(n):
    prefix[i + 1] = prefix[i] + arr[i]
result = prefix[r + 1] - prefix[l]

# String matching
freq = {}
for char in s:
    freq[char] = freq.get(char, 0) + 1

# Common imports
from collections import defaultdict, Counter
from typing import list, tuple, dict, set
```

---

## Key Takeaways

1. **Master two pointers** - Solves many problems efficiently
2. **Understand sliding window** - Optimize substring/subarray problems
3. **Use prefix sums** - Answer range queries quickly
4. **Know string algorithms** - KMP for pattern matching, Trie for word search
5. **Practice problem recognition** - Identify which pattern applies
6. **Handle edge cases** - Empty inputs, single elements, duplicates
7. **Optimize space** - Prefer O(1) space when possible
8. **Write clean code** - Use type hints and descriptive variable names

---

## Additional Resources

- **Visualization**: https://visualgo.net/en
- **Problem Sets**: Start with "Easy", then "Medium", then "Hard"
- **Time Complexity**: Always analyze and compare approaches
- **Practice Consistency**: 15-30 minutes daily beats 3-hour cramming sessions

