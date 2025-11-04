# Graphs - Comprehensive DSA Notes

## Core Concepts

### What are Graphs?
**Simple Definition**: A data structure consisting of nodes (vertices) connected by edges, where connections can go in any direction.

**Real-world Analogy**: Like a map of cities. Each city is a node, and roads connecting them are edges. You can travel along any road in any direction (depending on the road type).

```python
# Graph representation
class GraphNode:
    def __init__(self, val: int):
        self.val = val
        self.neighbors = []  # List of connected nodes

# Visual:
#     1 --- 2
#     |     |
#     3 --- 4
```

### Types of Graphs

| Type | Edges | Use Case | Example |
|------|-------|----------|---------|
| Undirected | No direction | Social networks | Friendship |
| Directed | Have direction | Web links | Website A → B |
| Weighted | Have weight | Distances | Distance between cities |
| Acyclic (DAG) | No cycles | Dependencies | Task scheduling |
| Tree | Special graph | Hierarchy | File system |
| Cyclic | Has cycles | General graphs | Network topology |

### Time and Space Complexity

| Operation | Adjacency List | Adjacency Matrix |
|-----------|----------------|------------------|
| Add vertex | O(1) | O(n²) |
| Add edge | O(1) | O(1) |
| Remove vertex | O(n + m) | O(n²) |
| Remove edge | O(m/n) avg | O(1) |
| Query edge | O(m/n) avg | O(1) |
| Space | O(n + m) | O(n²) |
| Sparse graphs | Better | Worse |
| Dense graphs | Similar | Better |

Where n = vertices, m = edges

### When to Use Graphs
✅ Networks (social, computer, transportation)
✅ Dependency management
✅ Maze/pathfinding problems
✅ Recommendation systems
✅ State space exploration

### When to Use Different Representations
- **Adjacency List**: Sparse graphs, most interview problems
- **Adjacency Matrix**: Dense graphs, edge queries
- **Edge List**: Minimum spanning trees, shortest paths

---

## Essential Patterns & Techniques

### 1. Graph Representations

**Template Code**:
```python
# Adjacency List (most common in interviews)
from typing import Optional
from collections import defaultdict, deque

class GraphAdjacencyList:
    def __init__(self):
        self.graph = defaultdict(list)

    def add_edge(self, u: int, v: int, weight: int = 1) -> None:
        """Add edge from u to v"""
        self.graph[u].append((v, weight))
        # For undirected: self.graph[v].append((u, weight))

    def get_neighbors(self, u: int) -> list[tuple]:
        """Get neighbors of vertex u"""
        return self.graph[u]

# Adjacency Matrix
class GraphAdjacencyMatrix:
    def __init__(self, n: int):
        self.n = n
        self.matrix = [[0] * n for _ in range(n)]

    def add_edge(self, u: int, v: int, weight: int = 1) -> None:
        """Add edge from u to v"""
        self.matrix[u][v] = weight
        # For undirected: self.matrix[v][u] = weight

    def get_neighbors(self, u: int) -> list[int]:
        """Get neighbors of vertex u"""
        return [v for v in range(self.n) if self.matrix[u][v] != 0]

# Using node objects
class Node:
    def __init__(self, val: int):
        self.val = val
        self.neighbors = []
```

---

### 2. Depth First Search (DFS)

**Description**: Explore as far as possible along each branch before backtracking.

**Template Code**:
```python
def dfs_recursive(node: Optional[Node], visited: set) -> list:
    """
    DFS traversal using recursion.

    Approach:
    1. Mark node as visited
    2. Process node
    3. Recursively visit unvisited neighbors
    """
    if not node or node in visited:
        return []

    visited.add(node)
    result = [node.val]

    for neighbor in node.neighbors:
        if neighbor not in visited:
            result.extend(dfs_recursive(neighbor, visited))

    return result

def dfs_iterative(start: Optional[Node]) -> list:
    """
    DFS traversal using explicit stack.

    Stack-based approach simulates recursion
    """
    if not start:
        return []

    visited = set()
    stack = [start]
    result = []

    while stack:
        node = stack.pop()

        if node in visited:
            continue

        visited.add(node)
        result.append(node.val)

        # Add neighbors to stack (reversed for left-to-right order)
        for neighbor in reversed(node.neighbors):
            if neighbor not in visited:
                stack.append(neighbor)

    return result

def count_connected_components(adj_list: dict) -> int:
    """Count number of connected components in graph"""
    visited = set()
    count = 0

    def dfs(node):
        visited.add(node)
        for neighbor in adj_list[node]:
            if neighbor not in visited:
                dfs(neighbor)

    for node in adj_list:
        if node not in visited:
            dfs(node)
            count += 1

    return count
```

**Complexity Analysis**:
- Time: O(V + E) - visit each vertex and edge once
- Space: O(V) - visited set + recursion/stack

---

### 3. Breadth First Search (BFS)

**Description**: Explore nodes level by level.

**Template Code**:
```python
def bfs(start: Optional[Node]) -> list:
    """
    BFS traversal using queue.

    Approach:
    1. Add start to queue
    2. While queue not empty:
       - Dequeue node
       - Process node
       - Enqueue unvisited neighbors
    """
    if not start:
        return []

    from collections import deque

    visited = {start}
    queue = deque([start])
    result = []

    while queue:
        node = queue.popleft()
        result.append(node.val)

        for neighbor in node.neighbors:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)

    return result

def shortest_path_bfs(start: Optional[Node], target: Optional[Node]) -> int:
    """
    Find shortest path between two nodes using BFS.

    Returns distance or -1 if unreachable
    """
    if start == target:
        return 0

    visited = {start}
    queue = deque([(start, 0)])  # (node, distance)

    while queue:
        node, dist = queue.popleft()

        for neighbor in node.neighbors:
            if neighbor == target:
                return dist + 1

            if neighbor not in visited:
                visited.add(neighbor)
                queue.append((neighbor, dist + 1))

    return -1  # Target unreachable
```

**Complexity Analysis**:
- Time: O(V + E) - same as DFS
- Space: O(V) - queue can have at most V nodes

---

### 4. Topological Sort (Directed Acyclic Graphs)

**Description**: Order vertices so all directed edges point forward.

**Template Code**:
```python
def topological_sort_kahn(adj_list: dict, in_degree: dict) -> list:
    """
    Topological sort using Kahn's algorithm (BFS-based).

    Approach:
    1. Start with nodes having 0 in-degree
    2. Remove node, reduce in-degree of neighbors
    3. When neighbor in-degree becomes 0, add to queue

    Time: O(V + E), Space: O(V)
    """
    from collections import deque

    queue = deque([node for node in in_degree if in_degree[node] == 0])
    result = []

    while queue:
        node = queue.popleft()
        result.append(node)

        for neighbor in adj_list[node]:
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                queue.append(neighbor)

    return result

def topological_sort_dfs(adj_list: dict) -> list:
    """
    Topological sort using DFS (post-order).

    DFS order nodes by finish time (reverse finish time is topo sort)
    """
    visited = set()
    stack = []

    def dfs(node):
        visited.add(node)

        for neighbor in adj_list[node]:
            if neighbor not in visited:
                dfs(neighbor)

        stack.append(node)  # Add after visiting all neighbors

    for node in adj_list:
        if node not in visited:
            dfs(node)

    return stack[::-1]  # Reverse for topological order

def detect_cycle_dfs(adj_list: dict) -> bool:
    """
    Detect cycle in directed graph using DFS.

    Approach:
    - WHITE (0): unvisited
    - GRAY (1): visiting (in current path)
    - BLACK (2): visited (finished)

    If we encounter GRAY node, cycle exists
    """
    state = {node: 0 for node in adj_list}

    def dfs(node):
        if state[node] == 1:  # GRAY - cycle found
            return True
        if state[node] == 2:  # BLACK - already processed
            return False

        state[node] = 1  # Mark as GRAY

        for neighbor in adj_list[node]:
            if dfs(neighbor):
                return True

        state[node] = 2  # Mark as BLACK
        return False

    for node in adj_list:
        if state[node] == 0:
            if dfs(node):
                return True

    return False
```

**Visual Example**:
```
Directed Acyclic Graph:
    0 --> 1 --> 3
    |           ^
    v           |
    2 ----------

Topological order: [0, 1, 2, 3] or [0, 2, 1, 3]
(All edges point forward)
```

---

### 5. Union-Find (Disjoint Set Union)

**Description**: Track and merge connected components efficiently.

**Template Code**:
```python
class UnionFind:
    """
    Union-Find data structure for detecting cycles and components.

    Time:
    - Union: O(α(n)) - nearly O(1)
    - Find: O(α(n)) - nearly O(1)
    Space: O(n)
    """

    def __init__(self, n: int):
        self.parent = list(range(n))
        self.rank = [0] * n

    def find(self, x: int) -> int:
        """Find root of set containing x (with path compression)"""
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])  # Path compression
        return self.parent[x]

    def union(self, x: int, y: int) -> bool:
        """
        Union two sets.

        Returns True if merged, False if already in same set
        """
        root_x = self.find(x)
        root_y = self.find(y)

        if root_x == root_y:
            return False  # Already in same set

        # Union by rank - attach smaller tree to larger
        if self.rank[root_x] < self.rank[root_y]:
            self.parent[root_x] = root_y
        elif self.rank[root_x] > self.rank[root_y]:
            self.parent[root_y] = root_x
        else:
            self.parent[root_y] = root_x
            self.rank[root_x] += 1

        return True

    def is_connected(self, x: int, y: int) -> bool:
        """Check if two nodes are in same set"""
        return self.find(x) == self.find(y)

# Use case: Detect cycle in undirected graph
def has_cycle_undirected(n: int, edges: list[tuple]) -> bool:
    """
    Detect cycle in undirected graph using Union-Find.

    If edge connects two nodes already in same set, cycle exists
    """
    uf = UnionFind(n)

    for u, v in edges:
        if not uf.union(u, v):  # Union fails = already connected = cycle
            return True

    return False
```

---

## Problem-Solving Framework

### Step 1: Represent the Graph
- Identify vertices and edges
- Choose representation (list vs matrix)
- Check if directed/undirected, weighted

### Step 2: Recognize Pattern
```
Is this about...?
├─ Traversal? → DFS or BFS
├─ Shortest path? → BFS (unweighted) or Dijkstra (weighted)
├─ Connected components? → DFS/BFS or Union-Find
├─ Topological order? → Kahn or DFS
├─ Cycle detection? → DFS colors or Union-Find
├─ All paths? → DFS with backtracking
└─ Minimum spanning tree? → Kruskal or Prim
```

### Step 3: Handle Edge Cases
- Disconnected graphs
- Self-loops
- Multiple edges
- Empty graph

### Step 4: Optimize if Needed
- Path compression in Union-Find
- Memoization in DFS
- Early termination if possible

---

## Common Pitfalls & Edge Cases

### 1. Forgetting to Mark as Visited
```python
# ❌ Wrong - infinite loop in cycles
def dfs(node):
    for neighbor in node.neighbors:
        dfs(neighbor)  # Never marks as visited

# ✅ Correct
def dfs(node, visited):
    visited.add(node)
    for neighbor in node.neighbors:
        if neighbor not in visited:
            dfs(neighbor, visited)
```

### 2. Wrong Space Complexity Calculation
```python
# For adjacency list:
# Space = O(V + E) not O(V²)
# For sparse graphs (E << V²), much better than matrix

# For adjacency matrix:
# Space always O(V²) regardless of edge count
```

### 3. Not Handling Disconnected Graphs
```python
# ❌ Wrong - assumes all nodes reachable from start
dfs(start_node)

# ✅ Correct - process all unvisited nodes
visited = set()
for node in all_nodes:
    if node not in visited:
        dfs(node, visited)
```

### 4. Topological Sort on Graph with Cycles
```python
# ❌ Wrong - topological sort undefined if cycle exists
result = topological_sort(adj_list)

# ✅ Correct - check for cycles first
if detect_cycle(adj_list):
    return []  # No valid topological order
result = topological_sort(adj_list)
```

---

## Comparison Table

| Algorithm | Use Case | Time | Space |
|-----------|----------|------|-------|
| DFS | Traversal, cycles | O(V+E) | O(V) |
| BFS | Shortest path, levels | O(V+E) | O(V) |
| Dijkstra | Shortest path (weighted) | O(E log V) | O(V) |
| Kruskal | MST | O(E log E) | O(V) |
| Prim | MST | O(E log V) | O(V) |
| Topological Sort | Task scheduling | O(V+E) | O(V) |
| Union-Find | Components, cycles | O(α(n)) | O(V) |
| Bellman-Ford | Shortest path (negative) | O(VE) | O(V) |

---

## Curated LeetCode Problems

### Beginner Level

#### 1. Number of Islands (LeetCode #200)
**Pattern**: DFS/BFS Connected Components
**Key Insight**: Treat grid as graph, count connected 1s

```python
def numIslands(grid: list[list[str]]) -> int:
    """
    Count islands (connected 1s) in grid.

    Approach: DFS from each unvisited land cell
    1. For each '1' not visited
    2. DFS to mark all connected '1's as visited
    3. Increment island count

    Time: O(m*n), Space: O(m*n)
    """
    if not grid:
        return 0

    rows, cols = len(grid), len(grid[0])
    visited = set()
    island_count = 0

    def dfs(r, c):
        if r < 0 or r >= rows or c < 0 or c >= cols or \
           grid[r][c] == '0' or (r, c) in visited:
            return

        visited.add((r, c))

        # Explore 4 directions
        dfs(r + 1, c)
        dfs(r - 1, c)
        dfs(r, c + 1)
        dfs(r, c - 1)

    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == '1' and (r, c) not in visited:
                dfs(r, c)
                island_count += 1

    return island_count

# Test
grid = [
    ["1","1","1","1","0"],
    ["1","1","0","1","0"],
    ["1","1","0","0","0"],
    ["0","0","0","0","0"]
]
assert numIslands(grid) == 1
```

---

#### 2. Clone Graph (LeetCode #133)
**Pattern**: DFS with Node Mapping
**Key Insight**: Map original nodes to clones

```python
def cloneGraph(node: Optional['Node']) -> Optional['Node']:
    """
    Deep copy a connected graph.

    Approach: DFS with mapping
    1. Keep map of original -> clone
    2. For each node, create clone if not exists
    3. Recursively clone all neighbors

    Time: O(V + E), Space: O(V)
    """
    if not node:
        return None

    visited = {}

    def dfs(original):
        if original in visited:
            return visited[original]

        # Create clone
        clone = Node(original.val)
        visited[original] = clone

        # Clone neighbors
        for neighbor in original.neighbors:
            clone.neighbors.append(dfs(neighbor))

        return clone

    return dfs(node)

# Alternative: BFS approach
def cloneGraph_bfs(node: Optional['Node']) -> Optional['Node']:
    """BFS approach for graph cloning"""
    if not node:
        return None

    from collections import deque

    visited = {node: Node(node.val)}
    queue = deque([node])

    while queue:
        original = queue.popleft()

        for neighbor in original.neighbors:
            if neighbor not in visited:
                visited[neighbor] = Node(neighbor.val)
                queue.append(neighbor)

            visited[original].neighbors.append(visited[neighbor])

    return visited[node]
```

---

#### 3. Detect Cycle in Directed Graph (LeetCode #207)
**Pattern**: DFS with Color States
**Key Insight**: Use colors to detect back edges

```python
def canFinish(numCourses: int, prerequisites: list[list[int]]) -> bool:
    """
    Determine if all courses can be completed (no cycle in prerequisites).

    Approach: Detect cycle using DFS colors
    - 0 (white): unvisited
    - 1 (gray): visiting (in current DFS path)
    - 2 (black): finished

    Cycle exists if we reach GRAY node

    Time: O(V + E), Space: O(V)
    """
    # Build adjacency list
    adj_list = [[] for _ in range(numCourses)]
    for course, prerequisite in prerequisites:
        adj_list[course].append(prerequisite)

    # 0: white, 1: gray, 2: black
    state = [0] * numCourses

    def has_cycle(course):
        if state[course] == 1:  # Gray - cycle detected
            return True
        if state[course] == 2:  # Black - already processed
            return False

        state[course] = 1  # Mark gray

        for prerequisite in adj_list[course]:
            if has_cycle(prerequisite):
                return True

        state[course] = 2  # Mark black
        return False

    # Check each course
    for course in range(numCourses):
        if state[course] == 0:
            if has_cycle(course):
                return False

    return True

# Test
assert canFinish(2, [[1, 0]]) == True
assert canFinish(2, [[1, 0], [0, 1]]) == False  # Cycle
```

---

### Intermediate Level

#### 4. Course Schedule II (LeetCode #210)
**Pattern**: Topological Sort
**Key Insight**: Order courses respecting prerequisites

```python
def findOrder(numCourses: int, prerequisites: list[list[int]]) -> list[int]:
    """
    Return course order respecting prerequisites (topological sort).

    Approach: Kahn's algorithm
    1. Build in-degree count for each course
    2. Start with courses having 0 prerequisites
    3. Process course, reduce in-degree of dependent courses
    4. Add courses with 0 in-degree to queue

    Time: O(V + E), Space: O(V)
    """
    from collections import deque, defaultdict

    # Build graph and in-degree
    adj_list = defaultdict(list)
    in_degree = [0] * numCourses

    for course, prerequisite in prerequisites:
        adj_list[prerequisite].append(course)
        in_degree[course] += 1

    # Find courses with no prerequisites
    queue = deque([i for i in range(numCourses) if in_degree[i] == 0])
    result = []

    while queue:
        course = queue.popleft()
        result.append(course)

        # Reduce in-degree of dependent courses
        for dependent in adj_list[course]:
            in_degree[dependent] -= 1
            if in_degree[dependent] == 0:
                queue.append(dependent)

    # If all courses processed, valid order exists
    return result if len(result) == numCourses else []

# Test
assert findOrder(2, [[1, 0]]) == [0, 1]
assert findOrder(4, [[1, 0], [2, 0], [3, 1], [3, 2]]) == [0, 1, 2, 3]
```

---

#### 5. Union Find - Redundant Connection (LeetCode #684)
**Pattern**: Cycle Detection with Union-Find
**Key Insight**: First edge creating cycle is answer

```python
def findRedundantConnection(edges: list[list[int]]) -> list[int]:
    """
    Find redundant edge that creates cycle in undirected graph.

    Approach: Union-Find
    - Process edges in order
    - If union fails (nodes already connected), edge is redundant
    - Return first redundant edge

    Time: O(n * α(n)), Space: O(n)
    """
    parent = list(range(len(edges) + 1))

    def find(x):
        if parent[x] != x:
            parent[x] = find(parent[x])
        return parent[x]

    def union(x, y):
        root_x, root_y = find(x), find(y)
        if root_x == root_y:
            return False  # Already connected - cycle
        parent[root_x] = root_y
        return True

    for u, v in edges:
        if not union(u, v):
            return [u, v]  # First edge creating cycle

    return []

# Test
assert findRedundantConnection([[1, 2], [1, 3], [2, 3]]) == [2, 3]
assert findRedundantConnection([[1, 2], [2, 3], [3, 4], [1, 4], [1, 5]]) == [1, 4]
```

---

### Advanced Level

#### 6. Word Ladder (LeetCode #127)
**Pattern**: BFS Shortest Path
**Key Insight**: Find shortest word transformation sequence

```python
def ladderLength(beginWord: str, endWord: str, wordList: list[str]) -> int:
    """
    Find shortest transformation sequence from beginWord to endWord.

    Each step change one letter. All intermediate words must be in wordList.

    Approach: BFS to find shortest path
    1. Start from beginWord
    2. For each word, find all neighbors (one letter difference)
    3. BFS to find shortest path to endWord

    Time: O(n * l² + l * 26) where n = words, l = length
    Space: O(n)
    """
    from collections import deque

    if endWord not in wordList:
        return 0

    # Build word set for O(1) lookup
    word_set = set(wordList)
    queue = deque([(beginWord, 1)])
    visited = {beginWord}

    while queue:
        word, steps = queue.popleft()

        if word == endWord:
            return steps

        # Try changing each character
        for i in range(len(word)):
            for c in 'abcdefghijklmnopqrstuvwxyz':
                neighbor = word[:i] + c + word[i+1:]

                if neighbor in word_set and neighbor not in visited:
                    visited.add(neighbor)
                    queue.append((neighbor, steps + 1))

    return 0

# Test
assert ladderLength("hit", "cog", ["hot", "dot", "dog", "lot", "log", "cog"]) == 5
# Path: "hit" -> "hot" -> "dot" -> "dog" -> "cog"
```

---

#### 7. Alien Dictionary (LeetCode #269)
**Pattern**: Topological Sort on Character Ordering
**Key Insight**: Build dependency graph from word ordering

```python
def alienOrder(words: list[str]) -> str:
    """
    Determine alien dictionary order from sorted word list.

    Approach: Build character dependency graph
    1. Compare adjacent words to find ordering
    2. Topological sort the characters
    3. Return sorted order

    Time: O(S + c²) where S = total chars, c = unique chars
    Space: O(c)
    """
    from collections import defaultdict, deque

    # Build graph
    graph = defaultdict(set)
    in_degree = {char: 0 for word in words for char in word}

    # Compare adjacent words
    for i in range(len(words) - 1):
        w1, w2 = words[i], words[i + 1]
        min_len = min(len(w1), len(w2))

        # Find first different character
        for j in range(min_len):
            if w1[j] != w2[j]:
                if w2[j] not in graph[w1[j]]:
                    graph[w1[j]].add(w2[j])
                    in_degree[w2[j]] += 1
                break
        else:
            # No difference found, check if w1 longer (invalid)
            if len(w1) > len(w2):
                return ""

    # Topological sort
    queue = deque([char for char in in_degree if in_degree[char] == 0])
    result = []

    while queue:
        char = queue.popleft()
        result.append(char)

        for neighbor in graph[char]:
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                queue.append(neighbor)

    return "".join(result) if len(result) == len(in_degree) else ""

# Test
assert alienOrder(["wrt", "wrf", "er", "ett", "rftt"]) == "wertf"
```

---

## Practice Roadmap

### Beginner (Foundation)
1. **Basic Traversal**
   - LeetCode #733: Flood Fill
   - LeetCode #200: Number of Islands
   - LeetCode #695: Max Area of Island

2. **Simple Paths**
   - LeetCode #559: Maximum Depth of N-ary Tree
   - LeetCode #997: Find Town Judge
   - LeetCode #101: Symmetric Tree (if graph)

### Intermediate (Pattern Mastery)
1. **Connected Components**
   - LeetCode #547: Number of Provinces
   - LeetCode #130: Surrounded Regions
   - LeetCode #1319: Number of Operations to Make Network Connected

2. **Topological Sort**
   - LeetCode #207: Course Schedule
   - LeetCode #210: Course Schedule II
   - LeetCode #269: Alien Dictionary

3. **Path Finding**
   - LeetCode #127: Word Ladder
   - LeetCode #505: The Maze II
   - LeetCode #433: Minimum Genetic Mutation

### Advanced (Mastery)
1. **Complex Algorithms**
   - LeetCode #269: Alien Dictionary
   - LeetCode #332: Reconstruct Itinerary
   - LeetCode #1192: Critical Connections in a Network

2. **Graph Theory**
   - LeetCode #1042: Minimum Cost to Connect Sticks
   - LeetCode #1168: Optimize Water Distribution in a Village
   - LeetCode #1489: Find Critical and Pseudo-Critical Edges

---

## Quick Reference

```python
# DFS
def dfs(node, visited):
    visited.add(node)
    for neighbor in node.neighbors:
        if neighbor not in visited:
            dfs(neighbor, visited)

# BFS
from collections import deque
queue = deque([start])
visited = {start}
while queue:
    node = queue.popleft()
    for neighbor in node.neighbors:
        if neighbor not in visited:
            visited.add(neighbor)
            queue.append(neighbor)

# Topological Sort (Kahn)
queue = deque([i for i in range(n) if in_degree[i] == 0])
for node in queue:
    for neighbor in adj_list[node]:
        in_degree[neighbor] -= 1
        if in_degree[neighbor] == 0:
            queue.append(neighbor)

# Union-Find
def find(x):
    if parent[x] != x:
        parent[x] = find(parent[x])
    return parent[x]

def union(x, y):
    parent[find(x)] = find(y)
```

---

## Key Takeaways

1. **Choose right representation** - List or matrix
2. **Master DFS and BFS** - Foundation of graph algorithms
3. **Understand Union-Find** - Efficient component tracking
4. **Know topological sort** - For DAGs and dependencies
5. **Cycle detection** - DFS colors or Union-Find
6. **Path finding** - BFS for unweighted, Dijkstra for weighted
7. **Handle disconnected graphs** - Process all components
8. **Think about time/space** - Choose optimal algorithm

