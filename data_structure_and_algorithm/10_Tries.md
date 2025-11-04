# Tries - Comprehensive DSA Notes

## Core Concepts

### What is a Trie (Prefix Tree)?
**Simple Definition**: A tree data structure where each node represents a character, used for efficient string searches.

**Real-world Analogy**: Like a dictionary organization where you navigate letter by letter to find words.

```python
class TrieNode:
    def __init__(self):
        self.children = {}  # Maps char to TrieNode
        self.is_word = False  # Marks end of word
```

### When to Use Trie
✅ Autocomplete/search suggestions
✅ Dictionary word validation
✅ IP routing (longest matching prefix)
✅ Phone directory
✅ Spell checker

### Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Insert | O(m) | O(m) |
| Search | O(m) | O(1) |
| StartsWith | O(m) | O(1) |
| Space for k words | O(k*m) | - |

Where m = word length, k = number of words

---

## Essential Patterns & Techniques

### 1. Basic Trie Operations

**Template Code**:
```python
class Trie:
    """
    Trie (Prefix Tree) implementation.

    Time: Insert/Search/Delete - O(m) where m = word length
    Space: O(alphabet_size * number_of_words)
    """

    def __init__(self):
        self.root = TrieNode()

    def insert(self, word: str) -> None:
        """Insert word into trie"""
        node = self.root

        for char in word:
            if char not in node.children:
                node.children[char] = TrieNode()
            node = node.children[char]

        node.is_word = True

    def search(self, word: str) -> bool:
        """Check if exact word exists"""
        node = self.root

        for char in word:
            if char not in node.children:
                return False
            node = node.children[char]

        return node.is_word

    def startsWith(self, prefix: str) -> bool:
        """Check if prefix exists"""
        node = self.root

        for char in prefix:
            if char not in node.children:
                return False
            node = node.children[char]

        return True

    def delete(self, word: str) -> bool:
        """Delete word from trie"""
        def _delete(node, word, index):
            if index == len(word):
                if not node.is_word:
                    return False
                node.is_word = False
                return len(node.children) == 0

            char = word[index]
            if char not in node.children:
                return False

            child = node.children[char]
            should_delete_child = _delete(child, word, index + 1)

            if should_delete_child:
                del node.children[char]

            return len(node.children) == 0 and not node.is_word

        return _delete(self.root, word, 0)

class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_word = False
```

---

### 2. Pattern Matching with Wildcards

**Template Code**:
```python
def search(board: list[list[str]], word: str) -> bool:
    """
    Search word with wildcards ('.' matches any char).

    Approach: DFS from each position
    - Use Trie for efficient matching
    - '.' matches any character

    Time: O(m*n*w) worst case
    Space: O(w) recursion
    """
    trie = Trie()
    trie.insert(word.replace('.', ''))  # Build trie for pattern

    def dfs(node, word, index):
        if index == len(word):
            return node.is_word

        char = word[index]

        if char == '.':
            for child in node.children.values():
                if dfs(child, word, index + 1):
                    return True
            return False
        else:
            if char not in node.children:
                return False
            return dfs(node.children[char], word, index + 1)

    return dfs(trie.root, word, 0)
```

---

### 3. Word Break/Dictionary

**Template Code**:
```python
def wordBreak(s: str, wordDict: list[str]) -> bool:
    """
    Check if string can be segmented using dictionary.

    Approach: DP with Trie
    1. Build Trie from dictionary
    2. dp[i] = can segment s[0:i]
    3. For each position, try matching from Trie

    Time: O(n²), Space: O(n + total_word_length)
    """
    trie = Trie()
    for word in wordDict:
        trie.insert(word)

    n = len(s)
    dp = [False] * (n + 1)
    dp[0] = True

    for i in range(1, n + 1):
        node = trie.root

        for j in range(i - 1, -1, -1):
            if not dp[j]:
                continue

            char = s[j]
            if char not in node.children:
                break

            node = node.children[char]

            if node.is_word and j < i:
                if j == 0 or dp[j]:
                    dp[i] = True
                    break

    return dp[n]

def findWords(board: list[list[str]], words: list[str]) -> list[str]:
    """
    Find all words from list in board (similar to boggle).

    Approach: Build Trie + DFS
    1. Build Trie from word list
    2. DFS from each board position
    3. Search through Trie paths

    Time: O(n*m*4^l), Space: O(total_word_length)
    """
    trie = Trie()
    for word in words:
        trie.insert(word)

    result = set()
    visited = set()

    def dfs(i, j, node, path):
        if node.is_word:
            result.add(path)

        for di, dj, char in [(0, 1, 'next'), (1, 0, 'down'),
                              (0, -1, 'prev'), (-1, 0, 'up')]:
            ni, nj = i + di, j + dj

            if 0 <= ni < len(board) and 0 <= nj < len(board[0]) and \
               (ni, nj) not in visited:
                char = board[ni][nj]

                if char in node.children:
                    visited.add((ni, nj))
                    dfs(ni, nj, node.children[char], path + char)
                    visited.remove((ni, nj))

    for i in range(len(board)):
        for j in range(len(board[0])):
            if board[i][j] in trie.root.children:
                visited.add((i, j))
                dfs(i, j, trie.root.children[board[i][j]], board[i][j])
                visited.remove((i, j))

    return list(result)
```

---

## Problem-Solving Framework

### When to Use Trie
1. **String searching** - Check prefixes/words
2. **Autocomplete** - Find all words with prefix
3. **Spell checking** - Validate dictionary
4. **IP routing** - Longest prefix matching
5. **Boggle/Word Search** - Find words in grid

### When to Avoid Trie
- Simple string checks → Hash set faster
- Need sorting → Different approach
- Space critical → Alternative better

---

## Common Pitfalls & Edge Cases

### 1. Not Handling End of Word
```python
# ❌ Wrong - can't distinguish words vs prefixes
class TrieNode:
    def __init__(self):
        self.children = {}

# ✅ Correct - mark word boundaries
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_word = False
```

### 2. Empty Prefix
```python
# ❌ Wrong - doesn't handle empty
def startsWith(self, prefix):
    if not prefix:
        return False  # Wrong!

# ✅ Correct - empty prefix always matches
def startsWith(self, prefix):
    if not prefix:
        return True
    # ... rest of logic
```

### 3. Not Clearing Visited in DFS
```python
# ❌ Wrong - visited persists
visited.add(node)
dfs(...)
# Not removing, affects next path

# ✅ Correct - backtrack properly
visited.add(node)
dfs(...)
visited.remove(node)
```

---

## Comparison Table

| Structure | Insert | Search | Prefix | Space | Best For |
|-----------|--------|--------|--------|-------|----------|
| Trie | O(m) | O(m) | O(m) | O(k*m) | Prefixes |
| Hash Set | O(m) | O(m) | O(n*m) | O(k*m) | Exact match |
| Binary Tree | O(log n) | O(log n) | No | O(n) | Sorted |
| Hash Map | O(m) | O(m) | O(n*m) | O(k*m) | Key-value |

---

## Curated LeetCode Problems

### Beginner Level

#### 1. Implement Trie (LeetCode #208)
```python
class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word: str) -> None:
        node = self.root
        for char in word:
            if char not in node.children:
                node.children[char] = TrieNode()
            node = node.children[char]
        node.is_word = True

    def search(self, word: str) -> bool:
        node = self.root
        for char in word:
            if char not in node.children:
                return False
            node = node.children[char]
        return node.is_word

    def startsWith(self, prefix: str) -> bool:
        node = self.root
        for char in prefix:
            if char not in node.children:
                return False
            node = node.children[char]
        return True
```

---

#### 2. Search Suggestions System (LeetCode #1268)
**Pattern**: Trie with Prefix Matching
**Key Insight**: Efficient prefix search with Trie

```python
def suggestedProducts(products: list[str], searchWord: str) -> list[list[str]]:
    products.sort()
    trie = Trie()
    for product in products:
        trie.insert(product)

    result = []
    prefix = ""

    for char in searchWord:
        prefix += char
        result.append(trie.get_top_3(prefix))

    return result
```

---

### Intermediate Level

#### 3. Word Search II (LeetCode #212)
```python
# See implementation above in "3. Word Break/Dictionary"
```

---

### Advanced Level

#### 4. Implement Magic Dictionary (LeetCode #676)
```python
class MagicDictionary:
    def __init__(self):
        self.trie = Trie()

    def buildDict(self, dictionary: list[str]) -> None:
        for word in dictionary:
            self.trie.insert(word)

    def search(self, searchWord: str) -> bool:
        # Search for word with exactly 1 char different
        def dfs(node, word, index, differences):
            if index == len(word):
                return differences == 1 and node.is_word

            char = word[index]

            for child_char in node.children:
                if child_char == char:
                    if dfs(node.children[child_char], word, index + 1, differences):
                        return True
                else:
                    if differences == 0:
                        if dfs(node.children[child_char], word, index + 1, 1):
                            return True

            return False

        return dfs(self.trie.root, searchWord, 0, 0)
```

---

## Practice Roadmap

### Beginner
- LeetCode #208: Implement Trie
- LeetCode #1268: Search Suggestions System
- LeetCode #1032: Stream of Characters

### Intermediate
- LeetCode #212: Word Search II
- LeetCode #139: Word Break
- LeetCode #720: Longest Word in Dictionary

### Advanced
- LeetCode #676: Implement Magic Dictionary
- LeetCode #1416: Restore The Array
- LeetCode #3043: Find the Length of the Longest Valid Parentheses

---

## Quick Reference

```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_word = False

def insert(word):
    node = root
    for char in word:
        if char not in node.children:
            node.children[char] = TrieNode()
        node = node.children[char]
    node.is_word = True

def search(word):
    node = root
    for char in word:
        if char not in node.children:
            return False
        node = node.children[char]
    return node.is_word
```

---

## Key Takeaways

1. **Trie for prefix problems** - Efficient prefix matching
2. **Mark word boundaries** - Use is_word flag
3. **Space-time tradeoff** - Extra space for fast lookup
4. **DFS for word search** - Traverse board with Trie guidance
5. **Backtracking needed** - Restore state in DFS
6. **Compare with hash set** - Sometimes simpler solution works
7. **Autocomplete use case** - Classic Trie application

