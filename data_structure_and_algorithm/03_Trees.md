# Trees - Comprehensive DSA Notes

## Core Concepts

### What are Trees?
**Simple Definition**: A hierarchical data structure with a root node and nodes connected by edges, where each node has 0 or more children.

**Real-world Analogy**: Like an organizational chart or family tree. The CEO is the root, and each person reports to someone above them. No cycles - everyone has a clear chain upward to the CEO.

```python
#      1 (root)
#     / \
#    2   3
#   / \
#  4   5

class TreeNode:
    def __init__(self, val: int = 0, left: 'TreeNode' = None, right: 'TreeNode' = None):
        self.val = val
        self.left = left
        self.right = right
```

### Types of Trees

| Type | Definition | Example |
|------|-----------|---------|
| Binary Tree | Max 2 children per node | [1, 2, 3, 4, None, None, 5] |
| Binary Search Tree (BST) | Left < Parent < Right | [4, 2, 6, 1, 3, 5, 7] |
| Balanced Tree | Height-balanced | AVL, Red-Black trees |
| Complete Tree | Filled level by level | Heap structure |
| Full Tree | Every node has 0 or 2 children | - |
| Perfect Tree | All levels filled | Like complete pyramid |

### Time and Space Complexity

| Operation | Best | Average | Worst | Space |
|-----------|------|---------|-------|-------|
| **BST** |
| Search | O(log n) | O(log n) | O(n) | O(log n) recursion |
| Insert | O(log n) | O(log n) | O(n) | O(log n) recursion |
| Delete | O(log n) | O(log n) | O(n) | O(log n) recursion |
| **Unbalanced Binary Tree** |
| Search | O(log n) | O(n) | O(n) | O(h) recursion |
| Insert | O(log n) | O(n) | O(n) | O(h) recursion |
| **Traversal** |
| DFS (all) | O(n) | O(n) | O(n) | O(h) stack |
| BFS (all) | O(n) | O(n) | O(n) | O(w) queue (w=width) |

Where h = height, n = number of nodes

### When to Use Trees
✅ Hierarchical data (file system, DOM)
✅ Need efficient searching
✅ Range queries (BST)
✅ Sorted data representation
✅ Autocomplete (Trie)

### When to Avoid Trees
❌ Simple linear data (use arrays)
❌ Need cache locality (arrays better)
❌ Extra memory overhead matters (vs arrays)

---

## Essential Patterns & Techniques

### 1. Tree Traversal Techniques

**Description**: Ways to visit all nodes in a tree.

**Template Code - DFS (Depth First Search)**:
```python
# DFS - Preorder (Root, Left, Right)
def preorder(root: Optional[TreeNode]) -> list[int]:
    """Visit root before children"""
    result = []

    def dfs(node):
        if not node:
            return
        result.append(node.val)      # Process root
        dfs(node.left)               # Process left
        dfs(node.right)              # Process right

    dfs(root)
    return result

# DFS - Inorder (Left, Root, Right)
def inorder(root: Optional[TreeNode]) -> list[int]:
    """Visit root between children"""
    result = []

    def dfs(node):
        if not node:
            return
        dfs(node.left)               # Process left
        result.append(node.val)      # Process root
        dfs(node.right)              # Process right

    dfs(root)
    return result

# DFS - Postorder (Left, Right, Root)
def postorder(root: Optional[TreeNode]) -> list[int]:
    """Visit root after children"""
    result = []

    def dfs(node):
        if not node:
            return
        dfs(node.left)               # Process left
        dfs(node.right)              # Process right
        result.append(node.val)      # Process root

    dfs(root)
    return result

# Iterative DFS using stack
def inorder_iterative(root: Optional[TreeNode]) -> list[int]:
    """Iterative inorder traversal"""
    result = []
    stack = []
    current = root

    while current or stack:
        # Go to leftmost node
        while current:
            stack.append(current)
            current = current.left

        # Current is None, pop from stack
        current = stack.pop()
        result.append(current.val)

        # Visit right subtree
        current = current.right

    return result
```

**Template Code - BFS (Breadth First Search)**:
```python
from collections import deque

def levelorder(root: Optional[TreeNode]) -> list[list[int]]:
    """
    Visit nodes level by level.

    Approach: Use queue
    - Add root to queue
    - While queue not empty:
      - Process all nodes at current level
      - Add children to queue
    """
    if not root:
        return []

    result = []
    queue = deque([root])

    while queue:
        level_size = len(queue)
        current_level = []

        # Process all nodes at current level
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

**Complexity Analysis**:
- Time: O(n) - visit each node once
- Space: O(h) for DFS (recursion stack), O(w) for BFS (queue width)

**Visual Example**:
```
       1
      / \
     2   3
    / \
   4   5

Preorder:   [1, 2, 4, 5, 3]  (Root first)
Inorder:    [4, 2, 5, 1, 3]  (Root middle - sorted if BST)
Postorder:  [4, 5, 2, 3, 1]  (Root last)
Level:      [[1], [2, 3], [4, 5]]
```

---

### 2. Binary Search Tree (BST) Operations

**Template Code**:
```python
class BST:
    def __init__(self, val: int):
        self.root = TreeNode(val)

    def search(self, root: Optional[TreeNode], val: int) -> Optional[TreeNode]:
        """
        Search for node with value in BST.

        Approach: Compare with root
        - If val < root.val, go left
        - If val > root.val, go right
        - If equal, found it

        Time: O(log n) average, O(n) worst
        """
        if not root:
            return None

        if val == root.val:
            return root
        elif val < root.val:
            return self.search(root.left, val)
        else:
            return self.search(root.right, val)

    def insert(self, root: Optional[TreeNode], val: int) -> TreeNode:
        """
        Insert value into BST maintaining BST property.

        Time: O(log n) average, O(n) worst
        """
        if not root:
            return TreeNode(val)

        if val < root.val:
            root.left = self.insert(root.left, val)
        elif val > root.val:
            root.right = self.insert(root.right, val)
        # If val == root.val, don't insert (no duplicates)

        return root

    def delete(self, root: Optional[TreeNode], val: int) -> Optional[TreeNode]:
        """
        Delete node with value from BST.

        Three cases:
        1. No children (leaf): just remove
        2. One child: replace with child
        3. Two children: replace with inorder successor

        Time: O(log n) average, O(n) worst
        """
        if not root:
            return None

        if val < root.val:
            root.left = self.delete(root.left, val)
        elif val > root.val:
            root.right = self.delete(root.right, val)
        else:
            # Found node to delete

            # Case 1: No left child
            if not root.left:
                return root.right

            # Case 2: No right child
            if not root.right:
                return root.left

            # Case 3: Two children
            # Find inorder successor (smallest in right subtree)
            min_larger_node = self._find_min(root.right)
            root.val = min_larger_node.val
            root.right = self.delete(root.right, min_larger_node.val)

        return root

    def _find_min(self, root: TreeNode) -> TreeNode:
        """Find node with minimum value"""
        current = root
        while current.left:
            current = current.left
        return current

    def is_valid_bst(self, root: Optional[TreeNode]) -> bool:
        """
        Check if binary tree is valid BST.

        Approach: Track min/max bounds
        - Left children must be < parent
        - Right children must be > parent
        """
        def validate(node, min_val, max_val):
            if not node:
                return True

            if node.val <= min_val or node.val >= max_val:
                return False

            return (validate(node.left, min_val, node.val) and
                    validate(node.right, node.val, max_val))

        return validate(root, float('-inf'), float('inf'))
```

**Visual Example**:
```
BST Insert [4, 2, 6, 1, 3, 5, 7]:
        4
       / \
      2   6
     / \ / \
    1  3 5  7

BST Property: Left < Parent < Right
Inorder traversal gives sorted: [1, 2, 3, 4, 5, 6, 7]
```

---

### 3. Path Finding in Trees

**Template Code**:
```python
def path_sum(root: Optional[TreeNode], targetSum: int) -> bool:
    """
    Check if any root-to-leaf path sums to target.

    Approach: DFS with accumulated sum
    - Subtract current node value from target
    - If reach leaf with sum = 0, found path
    """
    def dfs(node, remaining):
        if not node:
            return False

        remaining -= node.val

        # Check if leaf node and sum is correct
        if not node.left and not node.right:
            return remaining == 0

        # Recurse to children
        return dfs(node.left, remaining) or dfs(node.right, remaining)

    return dfs(root, targetSum)

def find_all_paths(root: Optional[TreeNode], targetSum: int) -> list[list[int]]:
    """Find all root-to-leaf paths that sum to target"""
    result = []

    def dfs(node, remaining, path):
        if not node:
            return

        remaining -= node.val
        path.append(node.val)

        # Check if leaf and sum is correct
        if not node.left and not node.right and remaining == 0:
            result.append(path.copy())

        # Recurse
        dfs(node.left, remaining, path)
        dfs(node.right, remaining, path)

        # Backtrack
        path.pop()

    dfs(root, targetSum, [])
    return result
```

---

### 4. Lowest Common Ancestor (LCA)

**Template Code**:
```python
def lowestCommonAncestor(root: Optional[TreeNode], p: TreeNode, q: TreeNode) -> Optional[TreeNode]:
    """
    Find lowest common ancestor of two nodes in BST.

    Approach: Use BST property
    - If both p and q < root, go left
    - If both p and q > root, go right
    - Otherwise, root is LCA

    Time: O(log n) average, O(n) worst
    """
    if not root:
        return None

    # Both nodes in left subtree
    if p.val < root.val and q.val < root.val:
        return lowestCommonAncestor(root.left, p, q)

    # Both nodes in right subtree
    if p.val > root.val and q.val > root.val:
        return lowestCommonAncestor(root.right, p, q)

    # Root is LCA (one on each side or root is one of them)
    return root

def lowestCommonAncestor_generic(root: Optional[TreeNode], p: TreeNode, q: TreeNode) -> Optional[TreeNode]:
    """
    Find LCA in generic binary tree (not necessarily BST).

    Approach: Postorder traversal
    - If both p and q found in subtree, subtree root is LCA
    - Return node if p or q found
    """
    if not root or root == p or root == q:
        return root

    left = lowestCommonAncestor_generic(root.left, p, q)
    right = lowestCommonAncestor_generic(root.right, p, q)

    # Both children have valid results
    if left and right:
        return root

    # One child has result
    return left if left else right
```

---

## Problem-Solving Framework

### Step 1: Identify Tree Type
- Binary Tree? BST? N-ary? Graph-like?
- Balanced? Complete? Perfect?

### Step 2: Choose Traversal
```
Need to...?
├─ Process in order? → Inorder
├─ Process parents first? → Preorder
├─ Process children first? → Postorder
├─ Level by level? → BFS
├─ Find paths? → DFS
└─ Use BST property? → Binary search path
```

### Step 3: Recursive vs Iterative
- **Recursive**: Cleaner code, stack space
- **Iterative**: Explicit control, extra space for stack

### Step 4: Common Patterns
- **Return values**: Boolean, node, list
- **Track state**: Parent pointers, visited set
- **Backtrack**: Restore state when returning

---

## Common Pitfalls & Edge Cases

### 1. Not Checking Leaf Nodes Correctly
```python
# ❌ Wrong - null can be a child
if node.left and node.right:
    # This is not necessarily a leaf

# ✅ Correct
if not node.left and not node.right:
    # This is definitely a leaf
```

### 2. Incorrect BST Validation
```python
# ❌ Wrong - only checks immediate children
if node.left.val < node.val and node.right.val > node.val:
    # Left subtree might have values > node.val

# ✅ Correct - track min/max bounds
def validate(node, min_val, max_val):
    if not node:
        return True
    if node.val <= min_val or node.val >= max_val:
        return False
    return validate(node.left, min_val, node.val) and \
           validate(node.right, node.val, max_val)
```

### 3. Modifying Tree While Traversing
```python
# ❌ Wrong - might break traversal
current = root
while current:
    if current.val == target:
        del current  # Breaks structure

# ✅ Correct - rebuild tree or mark for deletion
# Option 1: Mark and process after
# Option 2: Rebuild and return new root
```

### 4. Off-by-One in Height Calculation
```python
# ❌ Wrong - single node height = 1?
height = 1 + max(left_height, right_height)

# ✅ Correct - single node height = 0 OR 1 (be consistent)
# Common: leaf = 0, so:
height = 1 + max(left_height or -1, right_height or -1)
# Or: leaf = 1, so:
height = 1 + max(left_height, right_height)
```

---

## Comparison Table

| Operation | Binary Tree | BST | AVL | Hash Table |
|-----------|-------------|-----|-----|-----------|
| Search | O(n) | O(log n) avg | O(log n) | O(1) avg |
| Insert | O(n) | O(log n) avg | O(log n) | O(1) avg |
| Delete | O(n) | O(log n) avg | O(log n) | O(1) avg |
| Range Query | O(n) | O(k+log n) | O(k+log n) | O(n) |
| Sorted Order | O(n) | O(n) | O(n) | O(n log n) |
| Memory | Minimal | With pointers | With pointers | Hash table |

---

## Curated LeetCode Problems

### Beginner Level

#### 1. Maximum Depth of Binary Tree (LeetCode #104)
**Pattern**: DFS/Recursion
**Key Insight**: Max depth = 1 + max(left depth, right depth)

```python
def maxDepth(root: Optional[TreeNode]) -> int:
    """
    Find maximum depth (height) of binary tree.

    Approach: Recursive DFS
    1. If node is null, depth = 0
    2. Otherwise, depth = 1 + max(left, right)

    Time: O(n), Space: O(h)
    """
    if not root:
        return 0

    left_depth = maxDepth(root.left)
    right_depth = maxDepth(root.right)

    return 1 + max(left_depth, right_depth)

# Test
root = TreeNode(3)
root.left = TreeNode(9)
root.right = TreeNode(20)
root.right.left = TreeNode(15)
root.right.right = TreeNode(7)
assert maxDepth(root) == 3
```

---

#### 2. Invert Binary Tree (LeetCode #226)
**Pattern**: DFS/Recursion
**Key Insight**: Swap left and right children

```python
def invertTree(root: Optional[TreeNode]) -> Optional[TreeNode]:
    """
    Mirror/invert a binary tree.

    Approach: Swap children at each node
    - Process node
    - Recursively invert left and right

    Time: O(n), Space: O(h)
    """
    if not root:
        return None

    # Swap children
    root.left, root.right = root.right, root.left

    # Recursively invert subtrees
    invertTree(root.left)
    invertTree(root.right)

    return root

# Test
root = TreeNode(4)
root.left = TreeNode(2)
root.right = TreeNode(7)
root.left.left = TreeNode(1)
root.left.right = TreeNode(3)
invertTree(root)
# Result:       4
#              / \
#             7   2
#            / \ / \
#           -  3 1
```

---

#### 3. Balanced Binary Tree (LeetCode #110)
**Pattern**: DFS with Height Tracking
**Key Insight**: Check balance at each node

```python
def isBalanced(root: Optional[TreeNode]) -> bool:
    """
    Check if binary tree is height-balanced.

    Definition: For every node, |height(left) - height(right)| <= 1

    Approach: DFS returning (is_balanced, height)
    - If unbalanced, return False early
    - Check all nodes in single pass

    Time: O(n), Space: O(h)
    """
    def check(node):
        # Returns (is_balanced, height)
        if not node:
            return True, 0

        left_balanced, left_height = check(node.left)
        if not left_balanced:
            return False, 0

        right_balanced, right_height = check(node.right)
        if not right_balanced:
            return False, 0

        # Check if current node is balanced
        is_balanced = abs(left_height - right_height) <= 1
        height = 1 + max(left_height, right_height)

        return is_balanced, height

    return check(root)[0]

# Test
root = TreeNode(3)
root.left = TreeNode(9)
root.right = TreeNode(20)
root.right.left = TreeNode(15)
root.right.right = TreeNode(7)
assert isBalanced(root) == True
```

---

### Intermediate Level

#### 4. Validate Binary Search Tree (LeetCode #98)
**Pattern**: DFS with Min/Max Bounds
**Key Insight**: Track allowed range for each node

```python
def isValidBST(root: Optional[TreeNode]) -> bool:
    """
    Check if binary tree is valid binary search tree.

    Approach: DFS with min/max constraints
    - Left subtree: must be < parent.val
    - Right subtree: must be > parent.val
    - NOT just compare with immediate parent

    Time: O(n), Space: O(h)
    """
    def validate(node, min_val, max_val):
        if not node:
            return True

        # Current node must be within bounds
        if node.val <= min_val or node.val >= max_val:
            return False

        # Left children must be < node.val
        # Right children must be > node.val
        return (validate(node.left, min_val, node.val) and
                validate(node.right, node.val, max_val))

    return validate(root, float('-inf'), float('inf'))

# Test
root = TreeNode(2)
root.left = TreeNode(1)
root.right = TreeNode(3)
assert isValidBST(root) == True

# Counter-example: node values don't respect BST
bad_root = TreeNode(5)
bad_root.left = TreeNode(1)
bad_root.right = TreeNode(4)
bad_root.right.left = TreeNode(3)
bad_root.right.right = TreeNode(6)
assert isValidBST(bad_root) == False  # 3 violates BST property
```

---

#### 5. Path Sum (LeetCode #112)
**Pattern**: DFS with Accumulation
**Key Insight**: Track running sum from root to leaf

```python
def hasPathSum(root: Optional[TreeNode], targetSum: int) -> bool:
    """
    Check if any root-to-leaf path sums to target.

    Approach: DFS subtracting target as we go
    - At leaf: check if remaining sum = node.val
    - At non-leaf: recurse with updated sum

    Time: O(n), Space: O(h)
    """
    def dfs(node, remaining):
        if not node:
            return False

        remaining -= node.val

        # Leaf node check
        if not node.left and not node.right:
            return remaining == 0

        # Recurse to children
        return dfs(node.left, remaining) or dfs(node.right, remaining)

    return dfs(root, targetSum)

# Test
root = TreeNode(5)
root.left = TreeNode(4)
root.right = TreeNode(8)
root.left.left = TreeNode(11)
root.left.left.left = TreeNode(7)
root.left.left.right = TreeNode(2)
root.right.left = TreeNode(13)
root.right.right = TreeNode(4)
root.right.right.right = TreeNode(1)

assert hasPathSum(root, 22) == True  # 5->4->11->2
assert hasPathSum(root, 26) == False
```

---

#### 6. Lowest Common Ancestor of a BST (LeetCode #235)
**Pattern**: BST Binary Search Path
**Key Insight**: Use BST ordering to find LCA

```python
def lowestCommonAncestor(root: Optional[TreeNode], p: TreeNode, q: TreeNode) -> Optional[TreeNode]:
    """
    Find lowest common ancestor in BST.

    Approach: Use BST property
    - If both < root, go left
    - If both > root, go right
    - Otherwise, root is LCA

    Why? In BST, LCA is first node where paths diverge

    Time: O(log n) avg, O(n) worst, Space: O(1) iterative
    """
    current = root

    while current:
        if p.val < current.val and q.val < current.val:
            # Both in left subtree
            current = current.left
        elif p.val > current.val and q.val > current.val:
            # Both in right subtree
            current = current.right
        else:
            # Found LCA (one on each side or current is one of them)
            return current

    return None

# Test
root = TreeNode(6)
root.left = TreeNode(2)
root.right = TreeNode(8)
root.left.left = TreeNode(0)
root.left.right = TreeNode(4)
root.left.right.left = TreeNode(3)
root.left.right.right = TreeNode(5)
root.right.left = TreeNode(7)
root.right.right = TreeNode(9)

p = root.left  # Node 2
q = root.left.right  # Node 4
result = lowestCommonAncestor(root, p, q)
assert result.val == 2  # 2 is LCA of 2 and 4
```

---

### Advanced Level

#### 7. Binary Tree Maximum Path Sum (LeetCode #124)
**Pattern**: DFS with Backtracking
**Key Insight**: Track global max while computing max from current node

```python
def maxPathSum(root: Optional[TreeNode]) -> int:
    """
    Find maximum path sum in binary tree (any path, not just root-to-leaf).

    Approach: DFS return max path from node downward
    - Track global maximum
    - For each node, consider all 4 paths:
      1. Left child only
      2. Right child only
      3. Left + right + node
      4. Just node

    Time: O(n), Space: O(h)
    """
    max_sum = [float('-inf')]

    def dfs(node):
        if not node:
            return 0

        # Get max sum from left and right (0 if negative)
        left_max = max(dfs(node.left), 0)
        right_max = max(dfs(node.right), 0)

        # Max path including current node
        current_max = node.val + left_max + right_max
        max_sum[0] = max(max_sum[0], current_max)

        # Return max path from current node downward
        return node.val + max(left_max, right_max)

    dfs(root)
    return max_sum[0]

# Test
root = TreeNode(1)
root.left = TreeNode(2)
root.right = TreeNode(3)
assert maxPathSum(root) == 6  # 2+1+3

root = TreeNode(-10)
root.left = TreeNode(9)
root.right = TreeNode(20)
root.right.left = TreeNode(15)
root.right.right = TreeNode(7)
assert maxPathSum(root) == 42  # 15+20+7
```

---

#### 8. Serialize and Deserialize Binary Tree (LeetCode #297)
**Pattern**: BFS/DFS with String Encoding
**Key Insight**: Include null markers to reconstruct

```python
class Codec:
    def serialize(self, root: Optional[TreeNode]) -> str:
        """
        Encode binary tree to string.

        Approach: Preorder traversal with null markers
        - Include None as 'null' in output
        - Allows full reconstruction

        Time: O(n), Space: O(n)
        """
        def dfs(node):
            if not node:
                return ['null']

            result = [str(node.val)]
            result.extend(dfs(node.left))
            result.extend(dfs(node.right))
            return result

        return ','.join(dfs(root))

    def deserialize(self, data: str) -> Optional[TreeNode]:
        """
        Decode string back to binary tree.

        Approach: Preorder reconstruction
        - Use list pointer to track position
        - Recursively build nodes

        Time: O(n), Space: O(h)
        """
        def dfs(nodes):
            val = next(nodes)

            if val == 'null':
                return None

            node = TreeNode(int(val))
            node.left = dfs(nodes)
            node.right = dfs(nodes)

            return node

        nodes = iter(data.split(','))
        return dfs(nodes)

# Test
root = TreeNode(1)
root.left = TreeNode(2)
root.right = TreeNode(3)
root.right.left = TreeNode(4)
root.right.right = TreeNode(5)

codec = Codec()
serialized = codec.serialize(root)  # "1,2,null,null,3,4,null,null,5,null,null"
deserialized = codec.deserialize(serialized)
# Should reconstruct original tree
```

---

#### 9. Construct Binary Tree from Preorder and Inorder (LeetCode #105)
**Pattern**: Divide and Conquer
**Key Insight**: Preorder gives root, inorder partitions subtrees

```python
def buildTree(preorder: list[int], inorder: list[int]) -> Optional[TreeNode]:
    """
    Reconstruct binary tree from preorder and inorder traversals.

    Approach: Divide and conquer
    1. First element in preorder is root
    2. Find root in inorder - left part is left subtree, right part is right subtree
    3. Recursively build left and right subtrees

    Why? Preorder: root first, Inorder: root in middle

    Time: O(n²) naive, O(n) with hash map
    Space: O(h) recursion
    """
    if not preorder or not inorder:
        return None

    # Build hash map for O(1) lookups
    inorder_map = {val: i for i, val in enumerate(inorder)}

    def build(pre_start, pre_end, in_start, in_end):
        if pre_start > pre_end:
            return None

        # First element in preorder is root
        root_val = preorder[pre_start]
        root = TreeNode(root_val)

        # Find root in inorder
        root_idx = inorder_map[root_val]

        # Number of nodes in left subtree
        left_count = root_idx - in_start

        # Build left subtree
        root.left = build(pre_start + 1, pre_start + left_count,
                         in_start, root_idx - 1)

        # Build right subtree
        root.right = build(pre_start + left_count + 1, pre_end,
                          root_idx + 1, in_end)

        return root

    return build(0, len(preorder) - 1, 0, len(inorder) - 1)

# Test
preorder = [3, 9, 20, 15, 7]
inorder = [9, 3, 15, 20, 7]
root = buildTree(preorder, inorder)
#       3
#      / \
#     9  20
#       /  \
#      15   7
```

---

## Practice Roadmap

### Beginner (Foundation)
1. **Basic Traversal**
   - LeetCode #94: Binary Tree Inorder Traversal
   - LeetCode #144: Binary Tree Preorder Traversal
   - LeetCode #145: Binary Tree Postorder Traversal

2. **Depth/Height**
   - LeetCode #104: Maximum Depth of Binary Tree
   - LeetCode #111: Minimum Depth of Binary Tree
   - LeetCode #110: Balanced Binary Tree

3. **Simple Problems**
   - LeetCode #226: Invert Binary Tree
   - LeetCode #100: Same Tree
   - LeetCode #101: Symmetric Tree

### Intermediate (Pattern Mastery)
1. **BST Operations**
   - LeetCode #98: Validate Binary Search Tree
   - LeetCode #230: Kth Smallest Element in BST
   - LeetCode #235: Lowest Common Ancestor of BST

2. **Path Problems**
   - LeetCode #112: Path Sum
   - LeetCode #113: Path Sum II
   - LeetCode #257: Binary Tree Paths

3. **Modification**
   - LeetCode #105: Construct Binary Tree (Preorder + Inorder)
   - LeetCode #106: Construct Binary Tree (Inorder + Postorder)
   - LeetCode #114: Flatten Binary Tree to Linked List

### Advanced (Mastery)
1. **Complex Patterns**
   - LeetCode #124: Binary Tree Maximum Path Sum
   - LeetCode #236: Lowest Common Ancestor (Generic Tree)
   - LeetCode #297: Serialize and Deserialize Binary Tree

2. **Challenging**
   - LeetCode #173: Binary Search Tree Iterator
   - LeetCode #222: Count Complete Tree Nodes
   - LeetCode #337: House Robber III

---

## Quick Reference

```python
# Recursive DFS
def dfs(node):
    if not node:
        return
    # Process
    dfs(node.left)
    dfs(node.right)

# Iterative DFS (Preorder)
stack = [root]
while stack:
    node = stack.pop()
    # Process
    if node.right: stack.append(node.right)
    if node.left: stack.append(node.left)

# BFS
from collections import deque
queue = deque([root])
while queue:
    node = queue.popleft()
    # Process
    if node.left: queue.append(node.left)
    if node.right: queue.append(node.right)

# Inorder traversal (gives sorted for BST)
def inorder(node):
    if node:
        inorder(node.left)
        # Process
        inorder(node.right)
```

---

## Key Takeaways

1. **Master three traversals**: Inorder, Preorder, Postorder
2. **Know BFS and DFS** - Different use cases
3. **Understand BST property** - Left < Parent < Right
4. **Use min/max bounds** - For BST validation
5. **Backtracking is key** - For path and sum problems
6. **Height vs Depth** - Be clear on definitions
7. **Recursion is natural** - For tree problems
8. **Test with examples** - Small trees help visualize

