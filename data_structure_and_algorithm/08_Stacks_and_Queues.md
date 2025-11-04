# Stacks & Queues - Comprehensive DSA Notes

## Core Concepts

### Stack (LIFO - Last In First Out)
**Real-world Analogy**: Stack of plates - you take from top, add to top.

```python
class Stack:
    def __init__(self):
        self.items = []

    def push(self, item):
        """Add to top - O(1)"""
        self.items.append(item)

    def pop(self):
        """Remove from top - O(1)"""
        return self.items.pop() if self.items else None

    def peek(self):
        """View top without removing - O(1)"""
        return self.items[-1] if self.items else None

    def is_empty(self):
        return len(self.items) == 0
```

### Queue (FIFO - First In First Out)
**Real-world Analogy**: Queue at bank - first to come is first to serve.

```python
from collections import deque

class Queue:
    def __init__(self):
        self.items = deque()

    def enqueue(self, item):
        """Add to back - O(1)"""
        self.items.append(item)

    def dequeue(self):
        """Remove from front - O(1)"""
        return self.items.popleft() if self.items else None

    def is_empty(self):
        return len(self.items) == 0
```

### Operations

| Operation | Stack | Queue | Time |
|-----------|-------|-------|------|
| Push/Enqueue | append | append | O(1) |
| Pop/Dequeue | pop | popleft | O(1) |
| Peek | [-1] | [0] | O(1) |
| Is Empty | len == 0 | len == 0 | O(1) |
| Space | O(n) | O(n) | - |

---

## Essential Patterns & Techniques

### 1. Stack - Parentheses Matching

**Template Code**:
```python
def isValid(s: str) -> bool:
    """
    Check if brackets are balanced.

    Approach: Stack
    - Push opening brackets
    - Match closing brackets with top
    - All matched = valid

    Time: O(n), Space: O(n)
    """
    stack = []
    pairs = {'(': ')', '[': ']', '{': '}'}

    for char in s:
        if char in pairs:  # Opening bracket
            stack.append(char)
        else:  # Closing bracket
            if not stack or pairs[stack.pop()] != char:
                return False

    return len(stack) == 0
```

---

### 2. Stack - Monotonic Stack

**Template Code**:
```python
def nextGreaterElement(nums: list[int]) -> list[int]:
    """
    Find next greater element for each number.

    Approach: Monotonic decreasing stack
    - Process numbers right to left
    - Pop smaller elements (they can't be answers)
    - Top of stack is next greater
    - Push current number

    Time: O(n), Space: O(n)
    """
    result = [-1] * len(nums)
    stack = []

    for i in range(len(nums) - 1, -1, -1):
        # Pop smaller elements
        while stack and stack[-1] <= nums[i]:
            stack.pop()

        if stack:
            result[i] = stack[-1]

        stack.append(nums[i])

    return result

def largestRectangleArea(heights: list[int]) -> int:
    """
    Largest rectangle in histogram.

    Approach: Monotonic increasing stack
    - Track indices of increasing heights
    - When height decreases, calculate areas

    Time: O(n), Space: O(n)
    """
    stack = []
    max_area = 0
    index = 0

    while index < len(heights):
        if not stack or heights[index] >= heights[stack[-1]]:
            stack.append(index)
            index += 1
        else:
            top = stack.pop()
            width = index if not stack else index - stack[-1] - 1
            area = heights[top] * width
            max_area = max(max_area, area)

    while stack:
        top = stack.pop()
        width = index if not stack else index - stack[-1] - 1
        area = heights[top] * width
        max_area = max(max_area, area)

    return max_area
```

---

### 3. Queue - BFS Level Traversal

**Template Code**:
```python
from collections import deque

def levelOrder(root: Optional[TreeNode]) -> list[list[int]]:
    """
    Level-order traversal using queue.

    Approach: BFS
    - Add root to queue
    - For each level, process all current nodes
    - Add children to queue

    Time: O(n), Space: O(w) where w = width
    """
    if not root:
        return []

    result = []
    queue = deque([root])

    while queue:
        level_size = len(queue)
        current_level = []

        for _ in range(level_size):
            node = queue.popleft()
            current_level.append(node.val)

            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)

        result.append(current_level)

    return result
```

---

### 4. Stack - Expression Evaluation

**Template Code**:
```python
def evalRPN(tokens: list[str]) -> int:
    """
    Evaluate reverse Polish notation (postfix).

    Example: ["2", "1", "+", "3", "*"] = (2+1)*3 = 9

    Approach: Stack
    - Push numbers
    - On operator, pop two operands, compute, push result

    Time: O(n), Space: O(n)
    """
    stack = []
    operators = {'+', '-', '*', '/'}

    for token in tokens:
        if token in operators:
            b = stack.pop()
            a = stack.pop()

            if token == '+':
                result = a + b
            elif token == '-':
                result = a - b
            elif token == '*':
                result = a * b
            elif token == '/':
                result = int(a / b)  # Truncate towards zero

            stack.append(result)
        else:
            stack.append(int(token))

    return stack[0]
```

---

## Problem-Solving Framework

### When to Use Stack
- Matching pairs (brackets, tags)
- Undo/Redo functionality
- Parsing expressions
- DFS traversal
- Monotonic problems (next greater/smaller)

### When to Use Queue
- Level-order traversal
- BFS (shortest path)
- Job scheduling
- Print queue

---

## Common Pitfalls & Edge Cases

### 1. Empty Stack Pop
```python
# ❌ Wrong - crashes on empty
top = stack.pop()

# ✅ Correct
if not stack:
    return None
top = stack.pop()
```

### 2. Stack vs List Operations
```python
# ❌ Wrong - list operations aren't O(1)
stack = []
stack.pop(0)  # O(n)! Not O(1)

# ✅ Correct - use deque for efficient popleft
from collections import deque
stack = deque()
stack.popleft()  # O(1)
```

### 3. Monotonic Stack Direction
```python
# ❌ Wrong - stack order unclear
while stack and stack[-1] < nums[i]:
    # Is this decreasing or increasing?

# ✅ Correct - comment clarifies
# Monotonic decreasing stack (maintain decreasing order)
while stack and stack[-1] < nums[i]:
    stack.pop()
```

---

## Comparison Table

| Operation | Stack | Queue | Notes |
|-----------|-------|-------|-------|
| Insert | O(1) | O(1) | append |
| Remove | O(1) | O(1) | pop vs popleft |
| Order | LIFO | FIFO | Different semantics |
| Use Case | DFS | BFS | Different algorithms |

---

## Curated LeetCode Problems

### Beginner Level

#### 1. Valid Parentheses (LeetCode #20)
**Pattern**: Stack Matching
**Key Insight**: Match closing with opening brackets

```python
def isValid(s: str) -> bool:
    stack = []
    pairs = {'(': ')', '[': ']', '{': '}'}
    for c in s:
        if c in pairs:
            stack.append(c)
        else:
            if not stack or pairs[stack.pop()] != c:
                return False
    return not stack
```

---

#### 2. Implement Queue using Stacks (LeetCode #232)
**Pattern**: Multiple Data Structures
**Key Insight**: Use two stacks to reverse order

```python
class MyQueue:
    def __init__(self):
        self.input = []
        self.output = []

    def push(self, x: int) -> None:
        self.input.append(x)

    def pop(self) -> int:
        self.peek()
        return self.output.pop()

    def peek(self) -> int:
        if not self.output:
            while self.input:
                self.output.append(self.input.pop())
        return self.output[-1]

    def empty(self) -> bool:
        return not self.input and not self.output
```

---

### Intermediate Level

#### 3. Evaluate Reverse Polish Notation (LeetCode #150)
**Pattern**: Stack with Operations
**Key Insight**: Process operands then operators

```python
def evalRPN(tokens: list[str]) -> int:
    stack = []
    for token in tokens:
        if token in '+-*/':
            b = stack.pop()
            a = stack.pop()
            if token == '+':
                stack.append(a + b)
            elif token == '-':
                stack.append(a - b)
            elif token == '*':
                stack.append(a * b)
            else:
                stack.append(int(a / b))
        else:
            stack.append(int(token))
    return stack[0]
```

---

#### 4. Daily Temperatures (LeetCode #739)
**Pattern**: Monotonic Stack
**Key Insight**: Find next greater element with distances

```python
def dailyTemperatures(temps: list[int]) -> list[int]:
    result = [0] * len(temps)
    stack = []
    for i in range(len(temps)):
        while stack and temps[i] > temps[stack[-1]]:
            prev_idx = stack.pop()
            result[prev_idx] = i - prev_idx
        stack.append(i)
    return result
```

---

### Advanced Level

#### 5. Largest Rectangle in Histogram (LeetCode #84)
**Pattern**: Monotonic Stack with Calculation
**Key Insight**: Track increasing heights, calculate areas on decrease

```python
def largestRectangleArea(heights: list[int]) -> int:
    stack = []
    max_area = 0
    index = 0

    while index < len(heights):
        if not stack or heights[index] >= heights[stack[-1]]:
            stack.append(index)
            index += 1
        else:
            top = stack.pop()
            width = index if not stack else index - stack[-1] - 1
            area = heights[top] * width
            max_area = max(max_area, area)

    while stack:
        top = stack.pop()
        width = len(heights) if not stack else len(heights) - stack[-1] - 1
        area = heights[top] * width
        max_area = max(max_area, area)

    return max_area
```

---

## Practice Roadmap

### Beginner
- LeetCode #20: Valid Parentheses
- LeetCode #232: Implement Queue using Stacks
- LeetCode #225: Implement Stack using Queues

### Intermediate
- LeetCode #150: Evaluate Reverse Polish Notation
- LeetCode #739: Daily Temperatures
- LeetCode #155: Min Stack

### Advanced
- LeetCode #84: Largest Rectangle in Histogram
- LeetCode #42: Trapping Rain Water
- LeetCode #316: Remove Duplicate Letters

---

## Quick Reference

```python
# Stack (LIFO)
stack = []
stack.append(x)      # Push
x = stack.pop()      # Pop
x = stack[-1]        # Peek

# Queue (FIFO)
from collections import deque
queue = deque()
queue.append(x)      # Enqueue
x = queue.popleft()  # Dequeue

# Monotonic stack pattern
for i in range(n):
    while stack and condition:
        stack.pop()  # Remove violating elements
    # Process current
    stack.append(i)
```

---

## Key Takeaways

1. **Stack for LIFO problems** - Parentheses, DFS, undo
2. **Queue for FIFO problems** - BFS, scheduling
3. **Monotonic stack is powerful** - Solves many problems efficiently
4. **Use deque not list for queue** - O(1) popleft
5. **Two stacks can simulate queue** - And vice versa
6. **Expression evaluation** - RPN is easier with stacks
7. **Always handle empty cases** - Don't crash on empty
8. **Consider space for level-order** - Queue space = tree width

