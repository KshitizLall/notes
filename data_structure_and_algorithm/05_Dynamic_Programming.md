# Dynamic Programming - Comprehensive DSA Notes

## Core Concepts

### What is Dynamic Programming?
**Simple Definition**: Breaking a complex problem into overlapping subproblems, solving each once, and storing results to avoid recomputation.

**Real-world Analogy**: Like remembering which routes took least time in your city. Instead of recalculating the shortest path every day, you store your findings and reuse them.

```python
# Example: Fibonacci

# ❌ Inefficient - recalculates same subproblems
def fib_naive(n):
    if n <= 1:
        return n
    return fib_naive(n-1) + fib_naive(n-2)
# fib(5) calculates fib(3) twice, fib(2) three times...

# ✅ DP - stores results
def fib_dp(n):
    dp = [0] * (n + 1)
    dp[1] = 1
    for i in range(2, n + 1):
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]
```

### Two Essential Conditions for DP
1. **Overlapping Subproblems**: Same subproblems solved multiple times
2. **Optimal Substructure**: Optimal solution built from optimal subsolutions

### DP Approaches

| Approach | Style | How It Works |
|----------|-------|------------|
| Top-Down (Memoization) | Recursive | Start from main problem, recurse and cache results |
| Bottom-Up (Tabulation) | Iterative | Build solutions from smallest subproblems up |
| 1D DP | Space Optimized | Optimize space by keeping only necessary previous states |
| 2D DP | Matrix problems | Used for grid/string problems |

### Time and Space Complexity

| Problem | States | Transitions | Time | Space |
|---------|--------|-------------|------|-------|
| Fibonacci | O(n) | O(1) | O(n) | O(n) |
| Coin Change | O(n) | O(m) | O(n*m) | O(n) |
| Longest Common Subsequence | O(m*n) | O(1) | O(m*n) | O(m*n) |
| Knapsack | O(n*w) | O(1) | O(n*w) | O(n*w) |
| Edit Distance | O(m*n) | O(1) | O(m*n) | O(m*n) |

---

## Essential Patterns & Techniques

### 1. 1D DP - Simple Sequences

**Template Code**:
```python
def climb_stairs(n: int) -> int:
    """
    Climb n stairs, 1 or 2 steps at a time. How many ways?

    Approach: 1D DP
    - dp[i] = number of ways to reach step i
    - dp[i] = dp[i-1] + dp[i-2]
    - (reach i from i-1 or i-2)

    Time: O(n), Space: O(n)
    """
    if n <= 2:
        return n

    dp = [0] * (n + 1)
    dp[1] = 1  # 1 way to climb 1 step
    dp[2] = 2  # 2 ways to climb 2 steps (1+1 or 2)

    for i in range(3, n + 1):
        dp[i] = dp[i-1] + dp[i-2]

    return dp[n]

# Space optimization: only need last 2 values
def climb_stairs_optimized(n: int) -> int:
    """Space-optimized Fibonacci-like solution"""
    if n <= 2:
        return n

    prev1 = 2  # dp[i-1]
    prev2 = 1  # dp[i-2]

    for i in range(3, n + 1):
        current = prev1 + prev2
        prev2 = prev1
        prev1 = current

    return prev1

def max_rob(nums: list[int]) -> int:
    """
    Rob houses getting max money. Can't rob adjacent.

    Approach: 1D DP
    - dp[i] = max money robbing up to house i
    - Choice: rob i (get nums[i] + dp[i-2]) or skip i (get dp[i-1])
    - dp[i] = max(dp[i-1], nums[i] + dp[i-2])

    Time: O(n), Space: O(1) optimized
    """
    if not nums:
        return 0
    if len(nums) == 1:
        return nums[0]

    prev2 = 0  # dp[i-2]
    prev1 = nums[0]  # dp[i-1]

    for i in range(1, len(nums)):
        current = max(prev1, nums[i] + prev2)
        prev2 = prev1
        prev1 = current

    return prev1
```

**Visual Example**:
```
Climb stairs n=4:
- 1 step: [1]
- 2 steps: [1+1], [2]
- 3 steps: [1+1+1], [1+2], [2+1]
- 4 steps: [1+1+1+1], [1+1+2], [1+2+1], [2+1+1], [2+2]

dp: [0, 1, 2, 3, 5]
      0  1  2  3  4
```

---

### 2. 2D DP - Strings & Sequences

**Template Code**:
```python
def longest_common_subsequence(text1: str, text2: str) -> int:
    """
    Find longest common subsequence length.

    Approach: 2D DP
    - dp[i][j] = LCS length of text1[0..i-1] and text2[0..j-1]
    - If chars match: dp[i][j] = dp[i-1][j-1] + 1
    - Else: dp[i][j] = max(dp[i-1][j], dp[i][j-1])

    Time: O(m*n), Space: O(m*n)
    """
    m, n = len(text1), len(text2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]

    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if text1[i-1] == text2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])

    return dp[m][n]

def edit_distance(word1: str, word2: str) -> int:
    """
    Minimum edits (insert, delete, replace) to transform word1 -> word2.

    Approach: 2D DP (Levenshtein Distance)
    - dp[i][j] = min edits to transform word1[0..i-1] -> word2[0..j-1]
    - If chars match: dp[i][j] = dp[i-1][j-1]
    - Else: dp[i][j] = 1 + min(
        dp[i-1][j],     # delete from word1
        dp[i][j-1],     # insert into word1
        dp[i-1][j-1]    # replace
      )

    Time: O(m*n), Space: O(m*n)
    """
    m, n = len(word1), len(word2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]

    # Base case: empty word1, need j insertions
    for j in range(n + 1):
        dp[0][j] = j

    # Base case: empty word2, need i deletions
    for i in range(m + 1):
        dp[i][0] = i

    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if word1[i-1] == word2[j-1]:
                dp[i][j] = dp[i-1][j-1]
            else:
                dp[i][j] = 1 + min(
                    dp[i-1][j],      # delete
                    dp[i][j-1],      # insert
                    dp[i-1][j-1]     # replace
                )

    return dp[m][n]

def longest_increasing_subsequence(nums: list[int]) -> int:
    """
    Find length of longest strictly increasing subsequence.

    Approach: 1D DP with binary search optimization
    - dp[i] = length of LIS ending at index i
    - dp[i] = 1 + max(dp[j]) for all j < i where nums[j] < nums[i]

    Time: O(n log n) with binary search, O(n²) basic
    Space: O(n)
    """
    if not nums:
        return 0

    dp = [1] * len(nums)

    for i in range(1, len(nums)):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)

    return max(dp)
```

**Visual Example**:
```
LCS("abcde", "ace"):
    ""  a   c   e
""   0  0   0   0
a    0  1   1   1
b    0  1   1   1
c    0  1   2   2
d    0  1   2   2
e    0  1   2   3

Result: 3 (the subsequence "ace")
```

---

### 3. Grid DP - 2D Problems

**Template Code**:
```python
def unique_paths(m: int, n: int) -> int:
    """
    Paths from top-left to bottom-right (only move down/right).

    Approach: 2D DP
    - dp[i][j] = ways to reach (i,j)
    - dp[i][j] = dp[i-1][j] + dp[i][j-1]

    Time: O(m*n), Space: O(m*n)
    """
    dp = [[1] * n for _ in range(m)]

    for i in range(1, m):
        for j in range(1, n):
            dp[i][j] = dp[i-1][j] + dp[i][j-1]

    return dp[m-1][n-1]

def minimum_path_sum(grid: list[list[int]]) -> int:
    """
    Find minimum sum path from top-left to bottom-right.

    Approach: 2D DP
    - dp[i][j] = min sum to reach (i,j)
    - dp[i][j] = grid[i][j] + min(dp[i-1][j], dp[i][j-1])

    Time: O(m*n), Space: O(m*n)
    """
    if not grid:
        return 0

    m, n = len(grid), len(grid[0])
    dp = [[0] * n for _ in range(m)]

    # First cell
    dp[0][0] = grid[0][0]

    # First row
    for j in range(1, n):
        dp[0][j] = dp[0][j-1] + grid[0][j]

    # First column
    for i in range(1, m):
        dp[i][0] = dp[i-1][0] + grid[i][0]

    # Fill rest
    for i in range(1, m):
        for j in range(1, n):
            dp[i][j] = grid[i][j] + min(dp[i-1][j], dp[i][j-1])

    return dp[m-1][n-1]
```

---

### 4. Knapsack Problem

**Template Code**:
```python
def knapsack_01(weights: list[int], values: list[int], capacity: int) -> int:
    """
    0/1 Knapsack: maximize value with weight constraint.

    Approach: 2D DP
    - dp[i][w] = max value using first i items with weight limit w
    - For each item: take it or leave it
    - dp[i][w] = max(
        dp[i-1][w],              # leave item
        values[i] + dp[i-1][w - weights[i]]  # take item
      )

    Time: O(n*w), Space: O(n*w)
    """
    n = len(weights)
    dp = [[0] * (capacity + 1) for _ in range(n + 1)]

    for i in range(1, n + 1):
        weight = weights[i-1]
        value = values[i-1]

        for w in range(capacity + 1):
            # Can't include this item
            if weight > w:
                dp[i][w] = dp[i-1][w]
            else:
                # Choose max: include or exclude
                dp[i][w] = max(
                    dp[i-1][w],              # exclude
                    value + dp[i-1][w - weight]  # include
                )

    return dp[n][capacity]

# Space optimization: 1D DP
def knapsack_01_optimized(weights: list[int], values: list[int], capacity: int) -> int:
    """1D space-optimized knapsack"""
    dp = [0] * (capacity + 1)

    for i in range(len(weights)):
        # Iterate backwards to avoid using same item twice
        for w in range(capacity, weights[i] - 1, -1):
            dp[w] = max(dp[w], values[i] + dp[w - weights[i]])

    return dp[capacity]

def coin_change(coins: list[int], amount: int) -> int:
    """
    Minimum coins to make amount (unbounded knapsack).

    Approach: 1D DP
    - dp[i] = min coins to make amount i
    - dp[i] = min(dp[i - coin] + 1) for all coins

    Time: O(amount * len(coins)), Space: O(amount)
    """
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0

    for i in range(1, amount + 1):
        for coin in coins:
            if coin <= i:
                dp[i] = min(dp[i], dp[i - coin] + 1)

    return dp[amount] if dp[amount] != float('inf') else -1
```

**Visual Example**:
```
0/1 Knapsack: weights=[2,3,4,5], values=[3,4,5,6], capacity=5

       0   1   2   3   4   5
    0  0   0   0   0   0   0
w=2,v=3 0   0   3   3   3   3
w=3,v=4 0   0   3   4   4   7
w=4,v=5 0   0   3   4   5   7
w=5,v=6 0   0   3   4   5   7

Best: take items with weights 2,3 for value 7
```

---

## Problem-Solving Framework

### Step 1: Can This Be Solved with DP?
- Does it have overlapping subproblems?
- Does it have optimal substructure?
- If yes to both → DP might work

### Step 2: Define State
```
What does dp[i] or dp[i][j] represent?
- Must capture all info needed to solve subproblem
- Must be able to compute from smaller states
```

### Step 3: State Transition
```
How to compute state from previous states?
- Analyze all choices at each step
- Pick the one that optimizes objective
- dp[state] = optimal_choice(dp[related_states])
```

### Step 4: Base Cases
```
What are the smallest subproblems?
- dp[0] = ?
- dp[0][0] = ?
- Must be explicitly initialized
```

### Step 5: Implementation
- Top-down (recursion + memoization) or
- Bottom-up (iterative tabulation)

### Step 6: Space Optimization
- Can we reduce dimensions?
- Do we need all previous rows/states?

---

## Common Pitfalls & Edge Cases

### 1. Off-by-One in DP Array
```python
# ❌ Wrong - array too small
dp = [0] * n  # Should be n+1 for 1-indexed

# ✅ Correct
dp = [0] * (n + 1)
for i in range(1, n + 1):
    dp[i] = ...
```

### 2. Not Initializing Base Cases
```python
# ❌ Wrong - dp[0] not set
dp = [0] * (n + 1)
for i in range(1, n + 1):
    dp[i] = dp[i-1] + ...  # dp[0] might be wrong

# ✅ Correct
dp = [0] * (n + 1)
dp[0] = base_case
for i in range(1, n + 1):
    dp[i] = ...
```

### 3. Wrong Iteration Order (1D Array)
```python
# ❌ Wrong - might use updated value
for w in range(capacity + 1):
    dp[w] = max(dp[w], value + dp[w - weight])

# ✅ Correct - iterate backwards to avoid reusing
for w in range(capacity, weight - 1, -1):
    dp[w] = max(dp[w], value + dp[w - weight])
```

### 4. Forgetting Memoization
```python
# ❌ Wrong - recalculates same subproblems
def fib(n, memo={}):
    if n <= 1:
        return n
    # Forgot to check memo!
    return fib(n-1, memo) + fib(n-2, memo)

# ✅ Correct
def fib(n, memo={}):
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    memo[n] = fib(n-1, memo) + fib(n-2, memo)
    return memo[n]
```

---

## Comparison Table

| Pattern | Example | State | Time | Space |
|---------|---------|-------|------|-------|
| 1D Linear | Fibonacci, Rob House | O(n) | O(n) | O(n) |
| 2D String | LCS, Edit Distance | O(m*n) | O(m*n) | O(m*n) |
| 2D Grid | Unique Paths, Min Sum | O(m*n) | O(m*n) | O(m*n) |
| Knapsack | 0/1, Unbounded | O(n*w) | O(n*w) | O(n*w) |
| Interval | Burst Balloons | O(n²) | O(n³) | O(n²) |
| Tree | House Robber III | O(n) | O(n) | O(h) |

---

## Curated LeetCode Problems

### Beginner Level

#### 1. Climbing Stairs (LeetCode #70)
**Pattern**: 1D DP
**Key Insight**: Each step from previous state

```python
def climbStairs(n: int) -> int:
    """Ways to climb n stairs (1 or 2 steps at a time)"""
    if n <= 2:
        return n
    prev2, prev1 = 1, 2
    for _ in range(3, n + 1):
        prev2, prev1 = prev1, prev1 + prev2
    return prev1
# Test: climbStairs(3) == 3, climbStairs(4) == 5
```

---

#### 2. House Robber (LeetCode #198)
**Pattern**: 1D DP with Choice
**Key Insight**: Rob current or previous, choose max

```python
def rob(nums: list[int]) -> int:
    """Max money robbing houses (can't rob adjacent)"""
    if not nums:
        return 0
    prev2 = 0
    prev1 = nums[0]
    for i in range(1, len(nums)):
        current = max(prev1, nums[i] + prev2)
        prev2 = prev1
        prev1 = current
    return prev1
# Test: rob([1,2,3,1]) == 4, rob([2,7,9,3,1]) == 12
```

---

### Intermediate Level

#### 3. Coin Change (LeetCode #322)
**Pattern**: Unbounded Knapsack
**Key Insight**: Min coins for each amount

```python
def coinChange(coins: list[int], amount: int) -> int:
    """Minimum coins to make amount"""
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0
    for i in range(1, amount + 1):
        for coin in coins:
            if coin <= i:
                dp[i] = min(dp[i], dp[i - coin] + 1)
    return dp[amount] if dp[amount] != float('inf') else -1
# Test: coinChange([1,2,5], 5) == 1, coinChange([2], 3) == -1
```

---

#### 4. Longest Common Subsequence (LeetCode #1143)
**Pattern**: 2D String DP
**Key Insight**: Match characters diagonal, skip if not

```python
def longestCommonSubsequence(text1: str, text2: str) -> int:
    """Length of longest common subsequence"""
    m, n = len(text1), len(text2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if text1[i-1] == text2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    return dp[m][n]
# Test: longestCommonSubsequence("abc", "abc") == 3
```

---

#### 5. Unique Paths (LeetCode #62)
**Pattern**: Grid DP
**Key Insight**: Sum paths from left and top

```python
def uniquePaths(m: int, n: int) -> int:
    """Paths from top-left to bottom-right (only right/down)"""
    dp = [[1] * n for _ in range(m)]
    for i in range(1, m):
        for j in range(1, n):
            dp[i][j] = dp[i-1][j] + dp[i][j-1]
    return dp[m-1][n-1]
# Test: uniquePaths(3, 7) == 28, uniquePaths(3, 2) == 3
```

---

### Advanced Level

#### 6. Edit Distance (LeetCode #72)
**Pattern**: 2D String DP with Transitions
**Key Insight**: Three operations - insert, delete, replace

```python
def minDistance(word1: str, word2: str) -> int:
    """Min edits to transform word1 to word2"""
    m, n = len(word1), len(word2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    for j in range(n + 1):
        dp[0][j] = j
    for i in range(m + 1):
        dp[i][0] = i
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if word1[i-1] == word2[j-1]:
                dp[i][j] = dp[i-1][j-1]
            else:
                dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])
    return dp[m][n]
# Test: minDistance("horse", "ros") == 3, minDistance("a", "a") == 0
```

---

#### 7. Maximum Product Subarray (LeetCode #152)
**Pattern**: 1D DP with Two Tracks
**Key Insight**: Track both max and min (negative becomes max)

```python
def maxProduct(nums: list[int]) -> int:
    """Max product of contiguous subarray"""
    if not nums:
        return 0
    max_dp = nums[0]
    min_dp = nums[0]
    result = nums[0]
    for i in range(1, len(nums)):
        # Current number might be new start, max*current, or min*current
        max_dp, min_dp = (
            max(nums[i], max_dp * nums[i], min_dp * nums[i]),
            min(nums[i], max_dp * nums[i], min_dp * nums[i])
        )
        result = max(result, max_dp)
    return result
# Test: maxProduct([2,3,-2,4]) == 6, maxProduct([-2]) == -2
```

---

#### 8. Longest Increasing Subsequence (LeetCode #300)
**Pattern**: 1D DP or Binary Search
**Key Insight**: For each element, find longest before it

```python
def lengthOfLIS(nums: list[int]) -> int:
    """Length of longest strictly increasing subsequence"""
    if not nums:
        return 0
    dp = [1] * len(nums)
    for i in range(1, len(nums)):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)
    return max(dp)
# Test: lengthOfLIS([10,9,2,5,3,7,101,18]) == 4 ([2,3,7,101])

# Optimized O(n log n) using binary search
import bisect
def lengthOfLIS_optimized(nums: list[int]) -> int:
    """Binary search optimization"""
    tails = []
    for num in nums:
        pos = bisect.bisect_left(tails, num)
        if pos == len(tails):
            tails.append(num)
        else:
            tails[pos] = num
    return len(tails)
```

---

## Practice Roadmap

### Beginner (Foundation)
1. **Simple 1D**
   - LeetCode #70: Climbing Stairs
   - LeetCode #198: House Robber
   - LeetCode #121: Best Time to Buy and Sell Stock

2. **Easy Transitions**
   - LeetCode #509: Fibonacci Number
   - LeetCode #1137: N-th Tribonacci Number
   - LeetCode #746: Min Cost Climbing Stairs

### Intermediate (Pattern Mastery)
1. **String DP**
   - LeetCode #1143: Longest Common Subsequence
   - LeetCode #516: Longest Palindromic Subsequence
   - LeetCode #1092: Shortest Common Supersequence

2. **Grid DP**
   - LeetCode #62: Unique Paths
   - LeetCode #63: Unique Paths II
   - LeetCode #64: Minimum Path Sum

3. **Knapsack**
   - LeetCode #322: Coin Change
   - LeetCode #518: Coin Change II
   - LeetCode #416: Partition Equal Subset Sum

### Advanced (Mastery)
1. **Complex String**
   - LeetCode #72: Edit Distance
   - LeetCode #10: Regular Expression Matching
   - LeetCode #132: Palindrome Partitioning II

2. **Challenging Patterns**
   - LeetCode #152: Maximum Product Subarray
   - LeetCode #300: Longest Increasing Subsequence
   - LeetCode #188: Best Time to Buy and Sell Stock IV

3. **Tree & Multi-D**
   - LeetCode #337: House Robber III
   - LeetCode #312: Burst Balloons
   - LeetCode #1092: Shortest Common Supersequence

---

## Quick Reference

```python
# Base DP template (1D)
dp = [0] * (n + 1)
dp[0] = base_case
for i in range(1, n + 1):
    dp[i] = compute_from_previous(dp[i-1], ...)

# 2D DP template
dp = [[0] * (n + 1) for _ in range(m + 1)]
# Initialize base cases
for i in range(1, m + 1):
    for j in range(1, n + 1):
        dp[i][j] = compute_from_neighbors(...)

# Memoization template
memo = {}
def dp_func(state):
    if state in memo:
        return memo[state]
    if base_case(state):
        return base_value
    result = compute_from_subproblems(...)
    memo[state] = result
    return result
```

---

## Key Takeaways

1. **Recognize DP problems** - Overlapping subproblems + optimal substructure
2. **Define state clearly** - What does dp[i] represent?
3. **Find transitions** - How to compute from smaller states
4. **Handle base cases** - Explicitly initialize
5. **Choose approach** - Top-down (easier to code) vs bottom-up (more control)
6. **Optimize space** - Only keep necessary previous states
7. **Test edge cases** - Empty inputs, single elements
8. **Practice a lot** - DP patterns become intuitive with practice

