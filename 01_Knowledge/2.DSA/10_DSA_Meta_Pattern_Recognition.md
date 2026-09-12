# DSA Meta: Pattern Recognition — Which Technique for Which Problem?

## The Problem

You face an interview question. You recognize it *could* be solved with recursion, or DP, or backtracking, or iteration. **Which one is the right tool?**

This note is a **decision tree** to help you match problem **features** to the **technique** that solves it best.

---

## Quick Decision Tree (Start Here)

```
Does the problem ask for:
│
├─ "All subsets / all combinations / all permutations"?
│  └─→ BACKTRACKING (with pruning) or DP
│
├─ "Count / enumerate valid arrangements with constraints"?
│  └─→ BACKTRACKING (with constraint checks)
│
├─ "Find if a path / solution exists"?
│  ├─→ BFS/DFS (for graph/tree traversal)
│  └─→ RECURSION + memoization (for overlapping subproblems)
│
├─ "Optimize a choice: max/min with dependencies"?
│  └─→ DYNAMIC PROGRAMMING
│
├─ "Process data level-by-level / layer-by-layer"?
│  └─→ BFS (breadth-first search)
│
├─ "Process data branch-by-branch / deeply"?
│  └─→ DFS (depth-first search) or RECURSION
│
├─ "Fast search in sorted data"?
│  └─→ BINARY SEARCH
│
└─ "Find optimal subarray / substring / window"?
   └─→ SLIDING WINDOW or TWO POINTERS
```

---

## Problem Patterns & Techniques

### Pattern 1: "All Subsets / Combinations / Permutations"

**Indicators:**
- "Generate all..." / "List all..." / "Count all..."
- "Find all valid..."
- "All possible arrangements"

**Technique:** BACKTRACKING

**Time Complexity:**
- All subsets of $n$ elements: $O(2^n)$ (there are $2^n$ subsets)
- All permutations of $n$ elements: $O(n!)$ (there are $n!$ permutations)
- All combinations of $k$ from $n$: $O(\binom{n}{k})$

**Why:** Backtracking naturally explores all branches and prunes dead-ends.

**Template:**
```python
def backtrack(path, choices):
    if is_valid(path):  # Base case: found a solution
        result.append(path)
        return
    
    for choice in choices:
        if is_feasible(choice, path):  # Pruning
            path.append(choice)
            backtrack(path, remaining_choices)
            path.pop()  # Backtrack
```

**Examples:**
- N-Queens, Sudoku, Letter Combinations of a Phone Number, Permutations, Combinations

---

### Pattern 2: "Count / Find Valid Arrangements with Constraints"

**Indicators:**
- "How many ways..."
- "Find all strings that..."
- "Valid parentheses / paths / etc."
- Constraints eliminate many branches

**Technique:** BACKTRACKING (aggressive pruning) or DP (if constraints allow overlapping subproblems)

**Choice:**
- **Backtracking:** When you need to explore many branches but constraints prune heavily
- **DP:** When the problem has optimal substructure (problem decomposes into smaller versions of itself)

**Example:** 
- Generate valid parentheses: BACKTRACKING (explore, prune if invalid)
- Count valid parentheses of length $2n$: DP (overlapping subproblems; Catalan numbers)

---

### Pattern 3: "Find if a Path / Solution Exists"

**Indicators:**
- "Does there exist a path..."
- "Can you reach..."
- "Is there a way to..."
- Often traverses a graph or tree

**Technique:** BFS or DFS (depending on structure)

**BFS vs. DFS:**

| Aspect | BFS | DFS |
|--------|-----|-----|
| **Order** | Level-by-level (breadth) | Branch-by-branch (depth) |
| **Data Structure** | Queue | Recursion stack (or explicit stack) |
| **Find shortest path?** | Yes (unweighted) | No |
| **Detect cycle?** | Yes | Yes |
| **Space** | $O(\text{width of tree})$ | $O(\text{height of tree})$ |

**When to use:**
- **BFS:** Find shortest path, level-order traversal, word ladder problems
- **DFS:** Simpler code, detect cycles, topological sort, connected components

---

### Pattern 4: "Optimize a Choice: Maximize/Minimize with Dependencies"

**Indicators:**
- "Maximum / minimum..."
- "Best way to..."
- Earlier decisions affect later options
- Often can be solved multiple ways

**Technique:** DYNAMIC PROGRAMMING (if overlapping subproblems)

**Key insight:** A problem has **optimal substructure** if an optimal solution can be built from optimal solutions to subproblems.

**Template:**
```python
# DP approach
dp[n] = max(choice1(dp[n-1]), choice2(dp[n-2]), ...)
# or for 2D:
dp[i][j] = max(dp[i-1][j], dp[i][j-1], dp[i-1][j-1]) + cost[i][j]
```

**Examples:**
- Coin change (minimum coins), house robber, longest common subsequence, 0/1 knapsack

---

### Pattern 5: "Process Data Level-by-Level / Layer-by-Layer"

**Indicators:**
- "Traverse a tree level by level"
- "Process each layer" / "Visit by distance"
- "Shortest path in unweighted graph"

**Technique:** BFS

**Why:** BFS naturally explores all nodes at distance $d$ before distance $d+1$.

**Template:**
```python
from collections import deque

def bfs(start):
    queue = deque([start])
    visited = {start}
    while queue:
        node = queue.popleft()
        for neighbor in node.neighbors:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
```

**Examples:**
- Level-order tree traversal, word ladder, binary tree level-order traversal

---

### Pattern 6: "Process Data Branch-by-Branch / Deeply"

**Indicators:**
- "Traverse a tree" (not level-order)
- "Recursively process subtree"
- "DFS" explicitly mentioned

**Technique:** DFS (or recursion)

**Template:**
```python
def dfs(node, state):
    if is_leaf(node):
        process_leaf(node)
        return result
    
    for child in node.children:
        dfs(child, updated_state)
```

**Examples:**
- Inorder/preorder/postorder tree traversal, path sum, connected components

---

### Pattern 7: "Fast Search in Sorted Data"

**Indicators:**
- "Search in sorted array" / "sorted list"
- Need to find element / range quickly
- Data is explicitly sorted

**Technique:** BINARY SEARCH

**Time Complexity:** $O(\log n)$ (vs. $O(n)$ for linear search)

**When:** Data is sorted; need to find an element, a boundary, or answer a range query

**Template:**
```python
def binary_search(arr, target):
    left, right = 0, len(arr) - 1
    while left <= right:
        mid = (left + right) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return -1
```

**Variants:**
- Find first occurrence
- Find last occurrence
- Find range

---

### Pattern 8: "Find Optimal Subarray / Substring / Window"

**Indicators:**
- "Longest / shortest subarray..." subject to constraint
- "Maximum sum of contiguous elements"
- "Find all substrings that..." (with some property)

**Technique:** SLIDING WINDOW or TWO POINTERS

**Sliding Window:**
- Expand window while condition holds
- Contract window when violated
- Track best seen so far
- $O(n)$ time (two pointers move across array once)

**Template:**
```python
def max_subarray_length(arr, constraint):
    left = 0
    max_len = 0
    for right in range(len(arr)):
        # Expand window
        add_to_window(arr[right])
        
        # Contract while violated
        while not satisfies(constraint):
            remove_from_window(arr[left])
            left += 1
        
        max_len = max(max_len, right - left + 1)
    return max_len
```

**Examples:**
- Longest substring without repeating characters, max sum subarray, sliding window maximum

---

### Pattern 9: "Two Pointers / Converging Approach"

**Indicators:**
- "Remove duplicates in-place"
- "Partition array"
- "Reverse elements"
- Often involves sorted input and two-way traversal

**Technique:** TWO POINTERS

**Template:**
```python
def two_pointers(arr):
    left, right = 0, len(arr) - 1
    while left < right:
        # Process and decide which to move
        if condition:
            left += 1
        else:
            right -= 1
```

**Examples:**
- Remove duplicates, container with most water, trapping rain water, reverse string

---

### Pattern 10: "Tree / Graph Traversal with State Tracking"

**Indicators:**
- "Find path that satisfies a property"
- "Track sum / max / state along path"
- Tree structure with dependencies

**Technique:** DFS/RECURSION with state parameter

**Template:**
```python
def dfs(node, path_state):
    if is_leaf(node):
        if is_valid(path_state):
            result.append(path)
        return
    
    for child in node.children:
        new_state = update_state(path_state, child)
        dfs(child, new_state)
```

**Examples:**
- Path sum, paths with given sum, maximum path sum in binary tree

---

## Decision Table: Quick Reference

| Pattern | Technique | Time | Space | Example |
|---------|-----------|------|-------|---------|
| All subsets/combinations | Backtracking | $O(2^n / n!)$ | $O(n)$ recursion | Subsets, permutations |
| Optimize with dependencies | DP | $O(n^2)$ typical | $O(n)$ typical | Coin change, LCS |
| Level-order traversal | BFS | $O(n)$ | $O(w)$ width | Tree level traversal |
| Deep traversal | DFS | $O(n)$ | $O(h)$ height | Preorder, postorder |
| Search sorted | Binary search | $O(\log n)$ | $O(1)$ | Find element |
| Optimal subarray | Sliding window | $O(n)$ | $O(k)$ window size | Max subarray length |
| Array partitioning | Two pointers | $O(n)$ | $O(1)$ | Remove duplicates |

---

## Interview Workflow

1. **Read the problem carefully** — identify which pattern it matches
2. **State your approach** — "This is a backtracking problem because..."
3. **State your complexity** — "Time: $O(2^n)$, Space: $O(n)$"
4. **Code the solution** — match the template
5. **Walk through an example** — trace your code on paper
6. **Optimize if time permits** — Can you prune better? Use DP? Binary search?

---

## Red Flags & How to Fix Them

| Red Flag | Likely Issue | Fix |
|----------|-------------|-----|
| Only backtracking works; feels slow | Maybe use DP instead? | Check for overlapping subproblems |
| Code is deeply nested | Too complex; maybe refactor | Extract helper functions |
| Forgot to backtrack | Logic error | Add `path.pop()` or equivalent |
| Off-by-one error in binary search | Classic mistake | Test boundary cases carefully |
| Visited set missing in DFS/BFS | Infinite loop / revisiting nodes | Always track visited nodes |

---

## Related Notes

- [[1.Arrays & Strings]] — Common subarray/substring problems
- [[3.Trees & BSTs]] — Tree traversal patterns
- [[5.Graphs]] — Graph search patterns (BFS, DFS)
- [[8.Dynamic Programming]] — Detailed DP approach guide
- [[7.Recursion & Backtracking]] — Deep dive into backtracking

#category/dsa #topic/meta-patterns #topic/algorithm-selection

