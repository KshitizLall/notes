# Recursion & Backtracking - Comprehensive DSA Notes

## Core Concepts

### What is Recursion?
**Simple Definition**: A function calling itself with a simpler version of the problem until reaching a base case.

**Real-world Analogy**: Looking for your keys - first check this room, if not found, check next room (recursive call). When you find them (base case), you return.

```python
# Simple recursion
def factorial(n):
    if n <= 1:  # Base case
        return 1
    return n * factorial(n - 1)  # Recursive case
```

### What is Backtracking?
**Simple Definition**: Recursion that explores all possibilities and "backs up" when hitting dead ends.

**Real-world Analogy**: Solving a maze - try one path, if dead end, backtrack to last intersection and try another.

```python
# Backtracking pattern
def solve(path, remaining):
    if is_complete(path):  # Base case - found solution
        return path

    for choice in get_choices(remaining):
        path.append(choice)
        remaining.remove(choice)

        result = solve(path, remaining)
        if result:
            return result

        # Backtrack - undo choice
        path.pop()
        remaining.add(choice)

    return None
```

---

## Essential Patterns & Techniques

### 1. Recursion Fundamentals

**Template Code**:
```python
def recursion_template(n: int) -> int:
    """
    Basic recursion structure.

    Key components:
    1. Base case(s) - when to stop
    2. Recursive case - call with simpler input
    3. Combine results
    """

    # Base case
    if n <= 0:
        return 1

    # Recursive case + combine
    return n * recursion_template(n - 1)

def tree_traversal(node):
    """DFS using recursion"""
    if not node:
        return []

    result = [node.val]  # Process node
    result.extend(tree_traversal(node.left))  # Left subtree
    result.extend(tree_traversal(node.right))  # Right subtree

    return result

def calculate_depth(node):
    """Calculate tree depth recursively"""
    if not node:
        return 0

    left_depth = calculate_depth(node.left)
    right_depth = calculate_depth(node.right)

    return 1 + max(left_depth, right_depth)
```

---

### 2. Backtracking - Permutations

**Template Code**:
```python
def permute(nums: list[int]) -> list[list[int]]:
    """
    Generate all permutations.

    Approach: Backtracking
    - Try each unused number at current position
    - Recurse with remaining numbers
    - Backtrack by undoing the choice

    Time: O(n!), Space: O(n)
    """
    result = []

    def backtrack(path, remaining):
        # Base case: no more choices
        if not remaining:
            result.append(path.copy())
            return

        # Try each choice
        for i in range(len(remaining)):
            choice = remaining[i]

            # Choose
            path.append(choice)
            new_remaining = remaining[:i] + remaining[i+1:]

            # Explore
            backtrack(path, new_remaining)

            # Unchoose (backtrack)
            path.pop()

    backtrack([], nums)
    return result

# Alternative: with visited set
def permute_visited(nums: list[int]) -> list[list[int]]:
    """Using visited set instead of creating new list"""
    result = []
    visited = [False] * len(nums)

    def backtrack(path):
        if len(path) == len(nums):
            result.append(path.copy())
            return

        for i in range(len(nums)):
            if not visited[i]:
                visited[i] = True
                path.append(nums[i])

                backtrack(path)

                path.pop()
                visited[i] = False

    backtrack([])
    return result
```

---

### 3. Backtracking - Combinations/Subsets

**Template Code**:
```python
def combinations(n: int, k: int) -> list[list[int]]:
    """
    Generate all k-combinations of [1..n].

    Approach: Backtracking without replacement
    - Choose element or skip it
    - Only look at larger elements (avoid duplicates)

    Time: O(C(n,k)), Space: O(k)
    """
    result = []

    def backtrack(start, path):
        # Base case: combination complete
        if len(path) == k:
            result.append(path.copy())
            return

        # Try each number starting from 'start'
        for i in range(start, n + 1):
            path.append(i)
            backtrack(i + 1, path)  # Only consider larger numbers
            path.pop()

    backtrack(1, [])
    return result

def subsets(nums: list[int]) -> list[list[int]]:
    """
    Generate all subsets (power set).

    Approach: Include or exclude each element
    """
    result = []

    def backtrack(index, path):
        # Add current subset
        result.append(path.copy())

        # Try adding each remaining element
        for i in range(index, len(nums)):
            path.append(nums[i])
            backtrack(i + 1, path)
            path.pop()

    backtrack(0, [])
    return result
```

---

### 4. Backtracking - NQueens Problem

**Template Code**:
```python
def solveNQueens(n: int) -> list[list[str]]:
    """
    Place n queens on n×n board, no conflicts.

    Approach: Backtracking
    - Place one queen per row
    - Check if placement safe (no column/diagonal conflicts)
    - Try next row or backtrack

    Time: O(n!), Space: O(n)
    """
    result = []
    board = [['.' for _ in range(n)] for _ in range(n)]
    cols = set()
    diag1 = set()  # row - col
    diag2 = set()  # row + col

    def backtrack(row):
        # Base case: all queens placed
        if row == n:
            result.append([
                ''.join(board[r])
                for r in range(n)
            ])
            return

        # Try each column
        for col in range(n):
            # Check if safe
            if col in cols or (row - col) in diag1 or (row + col) in diag2:
                continue

            # Place queen
            board[row][col] = 'Q'
            cols.add(col)
            diag1.add(row - col)
            diag2.add(row + col)

            backtrack(row + 1)

            # Remove queen (backtrack)
            board[row][col] = '.'
            cols.remove(col)
            diag1.remove(row - col)
            diag2.remove(row + col)

    backtrack(0)
    return result
```

---

### 5. Memoization (Top-Down DP)

**Template Code**:
```python
def fib_memo(n: int, memo: dict = None) -> int:
    """
    Fibonacci with memoization (recursive DP).

    Approach: Store results to avoid recomputation
    """
    if memo is None:
        memo = {}

    if n in memo:
        return memo[n]

    if n <= 1:
        return n

    memo[n] = fib_memo(n - 1, memo) + fib_memo(n - 2, memo)
    return memo[n]

# Using decorator
from functools import lru_cache

@lru_cache(maxsize=None)
def fib_cached(n: int) -> int:
    """Using Python's built-in caching"""
    if n <= 1:
        return n
    return fib_cached(n - 1) + fib_cached(n - 2)
```

---

## Problem-Solving Framework

### Pattern Recognition

```
Is this a...?
├─ Permutation problem? → Generate all orderings
├─ Combination problem? → Select k from n
├─ Subset problem? → All possible selections
├─ Constraint satisfaction? → Backtrack when violated
├─ Tree/Graph problem? → DFS recursion
└─ Optimization problem? → Memoization
```

### Backtracking Template

```python
result = []

def backtrack(path, remaining):
    # Base case
    if is_complete(path):
        result.append(path.copy())
        return

    # Try each choice
    for choice in get_choices(remaining):
        # Choose
        path.append(choice)
        new_remaining = remove(remaining, choice)

        # Explore
        backtrack(path, new_remaining)

        # Unchoose
        path.pop()

return result
```

---

## Common Pitfalls & Edge Cases

### 1. Not Backtracking Properly
```python
# ❌ Wrong - doesn't undo the choice
path.append(choice)
backtrack(path, remaining)  # Path still modified!

# ✅ Correct - undo after exploring
path.append(choice)
backtrack(path, remaining)
path.pop()  # Undo the choice
```

### 2. Duplicate Results
```python
# ❌ Wrong - generates duplicates
for choice in get_choices():
    backtrack(path + [choice], ...)

# ✅ Correct - only consider larger indices
for i in range(start, len(choices)):
    backtrack(..., i + 1)
```

### 3. Too Much Memory (Memoization)
```python
# ❌ Wrong - stores every intermediate result
memo = {}
def fib(n):
    if n not in memo:
        memo[n] = ...
    # memo grows very large!

# ✅ Correct - only store necessary
from functools import lru_cache
@lru_cache(maxsize=128)  # Limit cache size
def fib(n):
    ...
```

### 4. Modifying Original
```python
# ❌ Wrong - original list modified
result.append(path)  # path is reference!

# ✅ Correct - copy the list
result.append(path.copy())
```

---

## Comparison Table

| Pattern | Use Case | Complexity | Space |
|---------|----------|-----------|-------|
| Simple Recursion | Tree traversal, divide-and-conquer | O(2^n) avg | O(n) recursion |
| Permutations | Generate all orderings | O(n!) | O(n) |
| Combinations | Select without order | O(2^n) | O(k) |
| Subsets | All subsets | O(2^n) | O(n) |
| Memoization | Optimization with overlap | Variable | O(n) or more |

---

## Curated LeetCode Problems

### Beginner Level

#### 1. Factorial (LeetCode #172 Trailing Zeros)
**Pattern**: Simple recursion
**Key Insight**: Decompose into smaller problems

```python
def factorial(n: int) -> int:
    if n <= 1:
        return 1
    return n * factorial(n - 1)
```

---

#### 2. Climbing Stairs (LeetCode #70)
**Pattern**: Recursion with memoization
**Key Insight**: f(n) = f(n-1) + f(n-2)

```python
from functools import lru_cache

@lru_cache(maxsize=None)
def climbStairs(n: int) -> int:
    if n <= 2:
        return n
    return climbStairs(n - 1) + climbStairs(n - 2)
```

---

### Intermediate Level

#### 3. Permutations (LeetCode #46)
**Pattern**: Backtracking
**Key Insight**: Try each element at current position

```python
def permute(nums: list[int]) -> list[list[int]]:
    result = []
    def backtrack(path):
        if len(path) == len(nums):
            result.append(path[:])
            return
        for i in range(len(nums)):
            if nums[i] not in path:
                path.append(nums[i])
                backtrack(path)
                path.pop()
    backtrack([])
    return result
```

---

#### 4. Combinations (LeetCode #77)
**Pattern**: Backtracking without replacement
**Key Insight**: Only consider larger numbers

```python
def combine(n: int, k: int) -> list[list[int]]:
    result = []
    def backtrack(start, path):
        if len(path) == k:
            result.append(path[:])
            return
        for i in range(start, n + 1):
            path.append(i)
            backtrack(i + 1, path)
            path.pop()
    backtrack(1, [])
    return result
```

---

### Advanced Level

#### 5. N-Queens (LeetCode #51)
**Pattern**: Backtracking with constraints
**Key Insight**: Track conflicts with sets

```python
# See full implementation above in "4. Backtracking - NQueens"
```

---

#### 6. Word Search II (LeetCode #212)
**Pattern**: Backtracking with Trie
**Key Insight**: Combine Trie and DFS

```python
# Requires Trie + DFS backtracking combination
```

---

## Practice Roadmap

### Beginner
- LeetCode #70: Climbing Stairs
- LeetCode #100: Same Tree
- LeetCode #226: Invert Binary Tree

### Intermediate
- LeetCode #46: Permutations
- LeetCode #77: Combinations
- LeetCode #78: Subsets

### Advanced
- LeetCode #51: N-Queens
- LeetCode #212: Word Search II
- LeetCode #37: Sudoku Solver

---

## Quick Reference

```python
# Basic recursion
def solve(n):
    if n <= 0:  # Base case
        return 1
    return n * solve(n - 1)

# Backtracking template
def backtrack(path, remaining):
    if is_complete(path):
        result.append(path.copy())
        return
    for choice in choices:
        path.append(choice)
        backtrack(path, remaining)
        path.pop()

# Memoization
from functools import lru_cache
@lru_cache(maxsize=None)
def solve(n):
    # ... recursive logic
```

---

## Key Takeaways

1. **Recursion basics** - Base case + recursive case
2. **Backtracking pattern** - Choose, explore, unchoose
3. **Memoization prevents recomputation** - Top-down DP
4. **Copy lists before storing** - Avoid reference issues
5. **Only look forward** - Avoid duplicate combinations
6. **Track constraints** - Use sets for efficiency
7. **Test base cases** - Single element, empty input
8. **Understand time complexity** - O(n!), O(2^n) growth

---

## Advanced Tips

### 1. Optimize with Pruning
```python
# Bad: explores all branches
for choice in choices:
    backtrack(choice)

# Good: skip invalid choices early
for choice in choices:
    if is_valid(choice):
        backtrack(choice)
```

### 2. Use Multiple Data Structures
```python
# Track visited with set for O(1) lookup
visited = set()
for elem in elements:
    if elem not in visited:
        backtrack(elem)
```

### 3. Space Optimization in Recursion
```python
# Instead of creating new arrays
backtrack(path + [choice])

# Modify and restore
path.append(choice)
backtrack(path)
path.pop()
```

---

## Debugging Backtracking

1. **Trace execution** - Print path at each step
2. **Check base case** - Ensure it's reached
3. **Verify backtracking** - State restored properly
4. **Count results** - Expected vs actual
5. **Test small inputs** - Debug easier with fewer elements

