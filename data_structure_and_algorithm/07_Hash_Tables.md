# Hash Tables - Comprehensive DSA Notes

## Core Concepts

### What are Hash Tables?
**Simple Definition**: A data structure that uses a hash function to map keys to values for fast lookup.

**Real-world Analogy**: Like a phone directory indexed alphabetically. Instead of searching linearly, you jump to the right section immediately.

```python
# Hash Table (Dictionary in Python)
hash_table = {}
hash_table['key'] = 'value'  # O(1) average
value = hash_table['key']     # O(1) average
```

### Hash Function
Converts a key into an array index:
- **Good hash function**: Distributes keys uniformly, deterministic
- **Bad hash function**: Clusters keys, collisions

### Collision Handling

| Method | How It Works | Pros | Cons |
|--------|------------|------|------|
| Chaining | Store list at each bucket | Simple, good load factor | Extra space |
| Open Addressing | Find next empty slot | Memory efficient | Clustering |
| Double Hashing | Use second hash if collision | Reduces clustering | More complex |

### Time Complexity

| Operation | Average | Worst | Notes |
|-----------|---------|-------|-------|
| Insert | O(1) | O(n) | Good hash function |
| Search | O(1) | O(n) | Collision handling |
| Delete | O(1) | O(n) | Depends on hash |
| Space | O(n) | O(n) | Always linear |

---

## Essential Patterns & Techniques

### 1. Basic Hash Table Operations

**Template Code**:
```python
from collections import defaultdict, Counter

# Count frequencies
def count_frequencies(arr: list) -> dict:
    """Count occurrences of each element"""
    freq = {}
    for item in arr:
        freq[item] = freq.get(item, 0) + 1
    return freq

    # Or using Counter
    return dict(Counter(arr))

# Check for duplicates
def has_duplicates(arr: list) -> bool:
    """Check if array has duplicates"""
    seen = set()
    for item in arr:
        if item in seen:
            return True
        seen.add(item)
    return False

# Find common elements
def find_common(arr1: list, arr2: list) -> set:
    """Find common elements"""
    set1 = set(arr1)
    return set1 & set(arr2)

# Anagram check
def is_anagram(s1: str, s2: str) -> bool:
    """Check if strings are anagrams"""
    from collections import Counter
    return Counter(s1) == Counter(s2)
```

---

### 2. Two-Sum Pattern

**Template Code**:
```python
def two_sum(nums: list[int], target: int) -> list[int]:
    """
    Find two numbers that sum to target.

    Approach: Hash map for O(n)
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

def three_sum(nums: list[int], target: int) -> set:
    """Find all triplets summing to target"""
    nums.sort()
    result = set()

    for i in range(len(nums) - 2):
        left, right = i + 1, len(nums) - 1
        while left < right:
            current_sum = nums[i] + nums[left] + nums[right]
            if current_sum == target:
                result.add((nums[i], nums[left], nums[right]))
                left += 1
                right -= 1
            elif current_sum < target:
                left += 1
            else:
                right -= 1

    return result
```

---

### 3. LRU Cache

**Template Code**:
```python
from collections import OrderedDict

class LRUCache:
    """
    Least Recently Used Cache with O(1) operations.

    Uses OrderedDict to maintain access order
    """

    def __init__(self, capacity: int):
        self.cache = OrderedDict()
        self.capacity = capacity

    def get(self, key: int) -> int:
        """Get value and mark as recently used"""
        if key not in self.cache:
            return -1

        # Move to end (mark as most recently used)
        self.cache.move_to_end(key)
        return self.cache[key]

    def put(self, key: int, value: int) -> None:
        """Add/update key-value pair"""
        if key in self.cache:
            self.cache.move_to_end(key)

        self.cache[key] = value

        if len(self.cache) > self.capacity:
            # Remove oldest (first) item
            self.cache.popitem(last=False)

# Or using doubly linked list + hash map
class LRUCache_Manual:
    def __init__(self, capacity: int):
        self.capacity = capacity
        self.cache = {}
        self.head = Node(0, 0)
        self.tail = Node(0, 0)
        self.head.next = self.tail
        self.tail.prev = self.head

    def get(self, key: int) -> int:
        if key not in self.cache:
            return -1
        node = self.cache[key]
        self._remove(node)
        self._add(node)
        return node.val

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            self._remove(self.cache[key])
        node = Node(key, value)
        self.cache[key] = node
        self._add(node)
        if len(self.cache) > self.capacity:
            node = self.head.next
            self._remove(node)
            del self.cache[node.key]

    def _add(self, node):
        """Add to most recent (before tail)"""
        prev = self.tail.prev
        node.next = self.tail
        node.prev = prev
        prev.next = node
        self.tail.prev = node

    def _remove(self, node):
        """Remove from linked list"""
        prev, next = node.prev, node.next
        prev.next = next
        next.prev = prev

class Node:
    def __init__(self, key, val):
        self.key = key
        self.val = val
        self.prev = None
        self.next = None
```

---

### 4. Group Anagrams

**Template Code**:
```python
def groupAnagrams(strs: list[str]) -> list[list[str]]:
    """
    Group anagrams together.

    Approach: Use sorted string as key
    - Anagrams have same sorted form
    - Group by sorted form

    Time: O(n * k log k) where k = max string length
    Space: O(n * k)
    """
    from collections import defaultdict

    groups = defaultdict(list)

    for s in strs:
        # Sorted string is key
        key = ''.join(sorted(s))
        groups[key].append(s)

    return list(groups.values())
```

---

## Problem-Solving Framework

### When to Use Hash Table
1. Need fast lookup/search
2. Counting frequencies
3. Checking existence
4. Finding pairs/groups
5. Caching results

### Pattern Recognition
```
Is this about...?
├─ Counting? → Use Counter or dict
├─ Checking existence? → Use set
├─ Pairing? → Use hash map
├─ Grouping? → Use defaultdict
└─ Caching? → Use dict or LRU cache
```

---

## Common Pitfalls & Edge Cases

### 1. Empty Collections
```python
# ❌ Wrong - crashes on empty
return max(freq.values())

# ✅ Correct
if not freq:
    return 0
return max(freq.values())
```

### 2. Hash Function Consistency
```python
# ❌ Wrong - mutable keys
d = {}
d[[1, 2]] = 'value'  # TypeError: list not hashable

# ✅ Correct - use tuples
d = {}
d[(1, 2)] = 'value'
```

### 3. Collision Handling
```python
# ❌ Wrong - ignores collisions
# Assume hash function distributes perfectly

# ✅ Correct - handle collisions
# Use chaining (lists) or open addressing
```

---

## Comparison Table

| Structure | Insert | Search | Delete | Space | Notes |
|-----------|--------|--------|--------|-------|-------|
| Hash Table | O(1) | O(1) | O(1) | O(n) | Average case |
| Hash Set | O(1) | O(1) | O(1) | O(n) | No duplicates |
| TreeMap | O(log n) | O(log n) | O(log n) | O(n) | Ordered |
| Array | O(n) | O(n) | O(n) | O(n) | Sequential |
| Linked List | O(1) | O(n) | O(1) | O(n) | With pointer |

---

## Curated LeetCode Problems

### Beginner Level

#### 1. Two Sum (LeetCode #1)
**Pattern**: Hash Map Pair
**Key Insight**: Store complement of each number

```python
def twoSum(nums: list[int], target: int) -> list[int]:
    num_map = {}
    for i, num in enumerate(nums):
        complement = target - num
        if complement in num_map:
            return [num_map[complement], i]
        num_map[num] = i
    return []
```

---

#### 2. Valid Anagram (LeetCode #242)
**Pattern**: Frequency Counting
**Key Insight**: Same characters, same frequency

```python
def isAnagram(s: str, t: str) -> bool:
    from collections import Counter
    return Counter(s) == Counter(t)
```

---

### Intermediate Level

#### 3. Group Anagrams (LeetCode #49)
**Pattern**: Hash Map with Grouping
**Key Insight**: Use sorted string as key

```python
def groupAnagrams(strs: list[str]) -> list[list[str]]:
    from collections import defaultdict
    groups = defaultdict(list)
    for s in strs:
        key = ''.join(sorted(s))
        groups[key].append(s)
    return list(groups.values())
```

---

#### 4. LRU Cache (LeetCode #146)
**Pattern**: Hash Map + Doubly Linked List
**Key Insight**: Maintain insertion/access order

```python
# See full implementation above in "3. LRU Cache"
```

---

#### 5. Contains Duplicate II (LeetCode #219)
**Pattern**: Hash Map with Window
**Key Insight**: Maintain set of k recent elements

```python
def containsNearbyDuplicate(nums: list[int], k: int) -> bool:
    window = set()
    for i in range(len(nums)):
        if nums[i] in window:
            return True
        window.add(nums[i])
        if len(window) > k:
            window.remove(nums[i - k])
    return False
```

---

### Advanced Level

#### 6. LFU Cache (LeetCode #460)
**Pattern**: Hash Map + Min-Heap + Frequency Tracking
**Key Insight**: Track both frequency and recency

```python
# Complex implementation combining hash maps with frequency tracking
```

---

## Practice Roadmap

### Beginner
- LeetCode #1: Two Sum
- LeetCode #242: Valid Anagram
- LeetCode #217: Contains Duplicate

### Intermediate
- LeetCode #49: Group Anagrams
- LeetCode #146: LRU Cache
- LeetCode #219: Contains Duplicate II

### Advanced
- LeetCode #460: LFU Cache
- LeetCode #356: Line Reflection
- LeetCode #1010: Pairs of Songs With Total Durations Divisible by 60

---

## Quick Reference

```python
# Frequency counting
from collections import Counter
freq = Counter(items)
most_common = freq.most_common(k)

# Two sum pattern
num_map = {}
for num in nums:
    if target - num in num_map:
        return True
    num_map[num] = True

# Grouping with defaultdict
from collections import defaultdict
groups = defaultdict(list)
for item in items:
    groups[key].append(item)
```

---

## Key Takeaways

1. **Hash tables are fast** - O(1) average time
2. **Use dict for mapping** - Key-value pairs
3. **Use set for lookup** - Check existence quickly
4. **Counter for frequencies** - Built-in frequency counting
5. **defaultdict for grouping** - Avoid KeyError
6. **Handle collisions** - Hash collisions possible
7. **Consider space** - Trade-off for speed
8. **Practice key design** - Proper keys make solutions elegant

