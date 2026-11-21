# LeetCode Problem of the Day: Find Subtree Sizes After Changes

- **Problem Link:** [LeetCode 3331 - Find Subtree Sizes After Changes](https://leetcode.com/problems/find-subtree-sizes-after-changes/description/)
- **Difficulty:** Medium
- **Topic Tags:** Tree, Depth-First Search (DFS), Breadth-First Search (BFS), String, Backtracking, Graph Theory
- **Target Audience:** POTD Solvers, Computer Science Students, Competitive Programmers, Interview Preparation (FAANG/Tier-1 Tech)

---

## 1. Problem Statement

You are given a tree rooted at node `0` consisting of $n$ nodes numbered from `0` to $n - 1$. The tree is represented by an integer array `parent` of length $n$, where `parent[i]` is the parent of node $i$. Since node `0` is the root, `parent[0] == -1`.

You are also given a string `s` of length $n$, where `s[i]` represents the lowercase English character assigned to node $i$.

We want to restructure the tree by applying the following rule **simultaneously** for every node $x$ ($1 \le x \le n - 1$):
- Search along the path from $x$ up to the root in the **original tree** for the **closest ancestor** $y$ ($y \neq x$) such that:
  $$s[x] == s[y]$$
- If such an ancestor $y$ exists, disconnect $x$ from its original parent and attach it to $y$ as its new parent ($parent[x] \gets y$).
- If no such ancestor exists, the parent of $x$ remains unchanged.

All changes happen **simultaneously** based strictly on the original tree structure.

**Goal:** Return an integer array `answer` of length $n$, where `answer[i]` is the **size of the subtree** rooted at node $i$ in the final restructured tree.

> **Subtree Size Definition:**  
> The subtree of node $i$ contains node $i$ itself and all of its descendants in the restructured tree. The size is the total count of nodes within this subtree.

---

## 2. Examples & Explanations

### Example 1

- **Input:** `parent = [-1, 0, 0, 1, 1, 1]`, `s = "abaabc"`
- **Output:** `[6, 3, 1, 1, 1, 1]`

#### Step-by-Step Breakdown:
- $n = 6$ nodes (from $0$ to $5$):
  - Node `0`: character `'a'`, parent `-1` (root)
  - Node `1`: character `'b'`, parent `0`
  - Node `2`: character `'a'`, parent `0`
  - Node `3`: character `'a'`, parent `1`
  - Node `4`: character `'b'`, parent `1`
  - Node `5`: character `'c'`, parent `1`

#### Ancestor Analysis (Original Tree):
1. **Node 1 ($s[1] = \text{'b'}$):** Ancestors are $\{0\}$ with char `'a'`. No ancestor has char `'b'`. Parent remains **`0`**.
2. **Node 2 ($s[2] = \text{'a'}$):** Ancestors are $\{0\}$ with char `'a'`. Closest ancestor with `'a'` is **`0`**. Parent remains **`0`**.
3. **Node 3 ($s[3] = \text{'a'}$):** Ancestors along path to root are $[1 (\text{'b'}), 0 (\text{'a'})]$. Closest matching ancestor with `'a'` is **`0`** (skipping node 1).  
   $\implies$ **Parent changes from $1 \to 0$!**
4. **Node 4 ($s[4] = \text{'b'}$):** Ancestors are $[1 (\text{'b'}), 0 (\text{'a'})]$. Closest ancestor with `'b'` is **`1`**. Parent remains **`1`**.
5. **Node 5 ($s[5] = \text{'c'}$):** Ancestors are $[1 (\text{'b'}), 0 (\text{'a'})]$. No ancestor has `'c'`. Parent remains **`1`**.

#### Tree Transformation:

```
        ORIGINAL TREE                            RESTRUCTURED TREE
         0 ('a')                                     0 ('a')
        /       \                                  /    |    \
     1 ('b')    2 ('a')                         1 ('b') 2('a') 3('a')
    /   |   \                                   /   \
 3('a') 4('b') 5('c')                        4('b') 5('c')
```

#### Final Subtree Sizes:
- $\text{size}(5) = 1$ (leaf)
- $\text{size}(4) = 1$ (leaf)
- $\text{size}(3) = 1$ (leaf)
- $\text{size}(2) = 1$ (leaf)
- $\text{size}(1) = 1 + \text{size}(4) + \text{size}(5) = 1 + 1 + 1 = \mathbf{3}$
- $\text{size}(0) = 1 + \text{size}(1) + \text{size}(2) + \text{size}(3) = 1 + 3 + 1 + 1 = \mathbf{6}$

$\implies \text{answer} = [6, 3, 1, 1, 1, 1]$.

---

### Example 2

- **Input:** `parent = [-1, 0, 4, 0, 1]`, `s = "abbba"`
- **Output:** `[5, 2, 1, 1, 1]`

#### Step-by-Step Breakdown:
- $n = 5$:
  - Node 0: $s[0] = \text{'a'}$, parent $-1$
  - Node 1: $s[1] = \text{'b'}$, parent $0$
  - Node 2: $s[2] = \text{'b'}$, parent $4$
  - Node 3: $s[3] = \text{'b'}$, parent $0$
  - Node 4: $s[4] = \text{'a'}$, parent $1$

#### Ancestor Traversal on Original Tree:
- Original path from node 2 to root: $2 \to 4 (\text{'a'}) \to 1 (\text{'b'}) \to 0 (\text{'a'})$.
  - Closest ancestor to 2 with `'b'` is **`1`**.  
  $\implies$ **Parent of 2 changes from $4 \to 1$!**
- Original path from node 4 to root: $4 \to 1 (\text{'b'}) \to 0 (\text{'a'})$.
  - Closest ancestor to 4 with `'a'` is **`0`**.  
  $\implies$ **Parent of 4 changes from $1 \to 0$!**
- Node 1: Ancestors $[0 (\text{'a'})]$. No `'b'`. Parent remains **`0`**.
- Node 3: Ancestors $[0 (\text{'a'})]$. No `'b'`. Parent remains **`0`**.

#### Restructured Tree Layout:
- Root `0` has children: `1`, `3`, `4`.
- Node `1` has child: `2`.
- Leaves: `2`, `3`, `4`.
- Subtree sizes: $\text{size}(2)=1, \text{size}(3)=1, \text{size}(4)=1, \text{size}(1)=1+1=2, \text{size}(0)=1+2+1+1=5$.

$\implies \text{answer} = [5, 2, 1, 1, 1]$.

---

## 3. Constraints & Scale Analysis

| Parameter | Constraint Range | Algorithmic Implications |
| :--- | :--- | :--- |
| **Number of Nodes ($n$)** | $1 \le n \le 10^5$ | An $\mathcal{O}(n^2)$ naive ancestor search will perform up to $10^{10}$ steps on a degenerate line graph and result in **Time Limit Exceeded (TLE)**. An $\mathcal{O}(n)$ or $\mathcal{O}(n \log n)$ algorithm is required. |
| **Alphabet Size ($|\Sigma|$)** | $26$ (lowercase `'a'` to `'z'`) | Extremely small alphabet size! We can maintain a static $26$-element array or stack of active ancestors along the DFS branch in $\mathcal{O}(1)$ time per node. |
| **Root Node** | `parent[0] == -1` | Exactly one component; acyclic directed rooted tree. |
| **Simultaneous Update** | Applied simultaneously | Ancestor discovery must reflect the **initial** tree topology, not incremental tree modifications. |

---

## 4. Visual Architecture & Two-Phase Pipeline

```mermaid
flowchart TD
    subgraph Phase1 ["Phase 1: Ancestor Discovery & Re-wiring (DFS 1)"]
        A1["Traverse Original Tree from Root 0"]
        A2["Maintain last_seen[26] array on DFS recursion stack"]
        A3["For node u with char c = s[u]:<br/>If last_seen[c] != -1 -> new_parent[u] = last_seen[c]<br/>Else -> new_parent[u] = parent[u]"]
        A4["Push u into last_seen[c], visit children, backtrack"]
        
        A1 --> A2 --> A3 --> A4
    end

    subgraph Phase2 ["Phase 2: Bottom-Up Subtree Aggregation (DFS 2)"]
        B1["Build New Adjacency List from new_parent"]
        B2["Traverse Restructured Tree from Root 0"]
        B3["For each child v of u:<br/>size[u] += DFS2(v)"]
        B4["size[u] += 1 (Count node u itself)"]
        
        B1 --> B2 --> B3 --> B4
    end

    Phase1 -->|"Construct Restructured Tree"| Phase2

    style Phase1 fill:#1E88E5,stroke:#0D47A1,color:#ffffff
    style Phase2 fill:#43A047,stroke:#1B5E20,color:#ffffff
```

### The Key Invariant: Maintaining Active Ancestors with Backtracking
Instead of climbing parent pointers from node $u$ upward to the root (which takes $\mathcal{O}(h)$ where $h \le n$), we invert the direction:
- As we traverse **down** from the root, the path from the root to the current node $u$ represents all of $u$'s ancestors in exact order of increasing depth.
- Since the alphabet consists of only $26$ lowercase English characters, we can maintain an array `last_seen[26]` where `last_seen[c]` records the **deepest (most recent) ancestor** on the current path with character $c$.
- **Lookup Cost:** $\mathcal{O}(1)$! Just read `last_seen[s[u] - 'a']`.
- **Backtracking:** Upon returning from visiting $u$'s subtree, we restore `last_seen[s[u] - 'a']` to its previous value so sibling branches do not see node $u$.

### Why Does the Restructured Graph Remain a Valid Tree?
1. Every node $x \ge 1$ changes its parent either to its original parent or to an ancestor $y$ of $x$ in the original tree.
2. In the original tree, an ancestor $y$ of $x$ has $\text{depth}(y) < \text{depth}(x)$.
3. Therefore, all new edges point from a strictly lower-depth node to a strictly higher-depth node.
4. It is impossible to form a cycle! Node $0$ remains the unique root with in-degree $0$, and every other node retains an in-degree of exactly $1$.
5. The restructured graph is strictly a valid rooted tree with $n$ nodes.

---

## 5. Step-by-Step Simulation & Trace Table

Tracing **Example 1**: `parent = [-1, 0, 0, 1, 1, 1]`, `s = "abaabc"`.

### Phase 1: DFS Ancestor Discovery

| Node $u$ | Char $s[u]$ | Previous `last_seen[c]` | Action Taken | `new_parent[u]` | Active `last_seen` during children calls |
| :---: | :---: | :---: | :--- | :---: | :--- |
| **0** | `'a'` | $-1$ | Root: `last_seen['a']` set to $0$ | $-1$ | `{'a': 0}` |
| **1** | `'b'` | $-1$ | `last_seen['b'] == -1` $\implies$ unchanged | **$0$** | `{'a': 0, 'b': 1}` |
| **3** | `'a'` | $0$ | `last_seen['a'] == 0` $\implies$ reattached to **$0$** | **$0$** | `{'a': 3, 'b': 1}` (Leaf, backtrack) |
| **4** | `'b'` | $1$ | `last_seen['b'] == 1` $\implies$ reattached to **$1$** | **$1$** | `{'a': 0, 'b': 4}` (Leaf, backtrack) |
| **5** | `'c'` | $-1$ | `last_seen['c'] == -1` $\implies$ unchanged | **$1$** | `{'a': 0, 'b': 1, 'c': 5}` (Leaf, backtrack) |
| — | — | — | Backtrack from Node 1 $\implies$ restore `last_seen['b'] = -1` | — | `{'a': 0}` |
| **2** | `'a'` | $0$ | `last_seen['a'] == 0` $\implies$ reattached to **$0$** | **$0$** | `{'a': 2}` (Leaf, backtrack) |

**Resulting `new_parent` array:** `[-1, 0, 0, 0, 1, 1]`

---

### Phase 2: Post-Order DFS Subtree Computation

```
Adjacency List of Restructured Tree:
0 -> [1, 2, 3]
1 -> [4, 5]
2 -> []
3 -> []
4 -> []
5 -> []
```

| Post-Order Visit | Children | Calculation | `answer[u]` |
| :---: | :---: | :--- | :---: |
| **Node 4** | None | Leaf $\implies 1$ | **1** |
| **Node 5** | None | Leaf $\implies 1$ | **1** |
| **Node 1** | $\{4, 5\}$ | $1 + \text{ans}[4] + \text{ans}[5] = 1 + 1 + 1$ | **3** |
| **Node 2** | None | Leaf $\implies 1$ | **1** |
| **Node 3** | None | Leaf $\implies 1$ | **1** |
| **Node 0** | $\{1, 2, 3\}$ | $1 + \text{ans}[1] + \text{ans}[2] + \text{ans}[3] = 1 + 3 + 1 + 1$ | **6** |

**Final Answer:** `[6, 3, 1, 1, 1, 1]`

---

## 6. Algorithm Flowchart

```mermaid
flowchart TD
    Start([Start: findSubtreeSizes]) --> BuildOriginalTree["Build adjacency list 'adj' from input 'parent'"]
    BuildOriginalTree --> InitPhase1["Initialize new_parent = copy of parent<br/>last_seen = array of 26 elements initialized to -1"]
    InitPhase1 --> CallDFS1["Invoke DFS1(node = 0)"]
    
    subgraph DFS1_Block ["DFS1(u) Execution"]
        D1["c = s[u] - 'a'"] --> D2{"last_seen[c] != -1 ?"}
        D2 -- Yes --> D3["new_parent[u] = last_seen[c]"]
        D2 -- No --> D4["Keep existing parent[u]"]
        D3 --> D5["old_val = last_seen[c]<br/>last_seen[c] = u"]
        D4 --> D5
        D5 --> D6["For each child v in adj[u]:<br/>DFS1(v)"]
        D6 --> D7["Backtrack: last_seen[c] = old_val"]
    end
    
    CallDFS1 --> DFS1_Block
    DFS1_Block --> BuildNewTree["Build restructured adjacency list 'new_adj' from 'new_parent'"]
    BuildNewTree --> InitPhase2["Initialize ans = array of size n with 0"]
    InitPhase2 --> CallDFS2["Invoke DFS2(node = 0)"]
    
    subgraph DFS2_Block ["DFS2(u) Execution"]
        E1["size = 1"] --> E2["For each child v in new_adj[u]:<br/>size += DFS2(v)"]
        E2 --> E3["ans[u] = size<br/>Return size"]
    end
    
    CallDFS2 --> DFS2_Block
    DFS2_Block --> ReturnAns["Return ans"]
    ReturnAns --> End([End])

    style Start fill:#2E7D32,stroke:#1B5E20,color:#ffffff
    style End fill:#C62828,stroke:#B71C1C,color:#ffffff
    style DFS1_Block fill:#E3F2FD,stroke:#1565C0,color:#000000
    style DFS2_Block fill:#E8F5E9,stroke:#2E7D32,color:#000000
```

---

## 7. Progression from Naive to Pro

### Method 1: Naive Pointer Climbing ($\mathcal{O}(n^2)$)
- **Concept:** For each node $u$ from $1$ to $n - 1$, climb upwards step-by-step using `curr = parent[curr]` until either reaching the root or finding an ancestor with $s[\text{curr}] == s[u]$. Then run standard DFS to count sizes.
- **Pseudo-Code:**
  ```text
  function findSubtreeSizesNaive(parent, s):
      n = length(parent)
      new_parent = copy(parent)
      
      for u from 1 to n - 1:
          curr = parent[u]
          while curr != -1:
              if s[curr] == s[u]:
                  new_parent[u] = curr
                  break
              curr = parent[curr]
              
      new_tree = build_adjacency(new_parent)
      ans = compute_subtree_sizes(new_tree)
      return ans
  ```
- **Complexity:**
  - **Time Complexity:** $\mathcal{O}(n \cdot h) = \mathcal{O}(n^2)$ when the tree is a long linear chain ($h = n$). For $n = 10^5$, operations $\approx 10^{10} \implies$ **TLE**.
  - **Auxiliary Space:** $\mathcal{O}(n)$.

---

### Method 2: Binary Lifting / Jump Tables ($\mathcal{O}(n \log n)$)
- **Concept:** Precompute $2^k$-th ancestors using binary lifting table `up[node][k]`. For each character $c \in ['a', 'z']$, precompute ancestor jump tables.
- **Complexity:**
  - **Time Complexity:** $\mathcal{O}(n \log n)$ time and space.
  - **Evaluation:** Overly complex for an alphabet of size $26$ when a simple DFS with backtracking achieves $\mathcal{O}(n)$ in just a few lines.

---

### Method 3: Optimal Single-Pass DFS with Backtracking ($\mathcal{O}(n)$) — Pro Method
- **Concept:** Traverse the original tree top-down with DFS. Maintain a fixed table `last_seen[26]` tracking the closest active ancestor for each letter. Update `new_parent` in $\mathcal{O}(1)$ time per node. Then run a second post-order DFS to compute subtree sizes in $\mathcal{O}(n)$.
- **Pseudo-Code:**
  ```text
  function findSubtreeSizesOptimal(parent, s):
      n = length(parent)
      adj = build_tree(parent)
      new_parent = copy(parent)
      last_seen = array of size 26 initialized to -1
      
      function dfs1(u):
          char_idx = ord(s[u]) - ord('a')
          old_ancestor = last_seen[char_idx]
          
          if old_ancestor != -1:
              new_parent[u] = old_ancestor
              
          last_seen[char_idx] = u
          for each child v in adj[u]:
              dfs1(v)
          last_seen[char_idx] = old_ancestor // Backtrack
          
      dfs1(0)
      
      new_adj = build_tree(new_parent)
      ans = array of size n
      
      function dfs2(u):
          sz = 1
          for each child v in new_adj[u]:
              sz += dfs2(v)
          ans[u] = sz
          return sz
          
      dfs2(0)
      return ans
  ```
- **Complexity:**
  - **Time Complexity:** $\mathcal{O}(n + |\Sigma|) = \mathbf{\mathcal{O}(n)}$.
  - **Auxiliary Space:** $\mathbf{\mathcal{O}(n)}$ for adjacency lists and recursion stacks.

---

## 8. Complexity Comparison Table

| Metric | Method 1: Naive Pointer Climbing | Method 2: Binary Lifting Jump Tables | Method 3: DFS with Backtracking (Optimal) |
| :--- | :---: | :---: | :---: |
| **Time Complexity** | $\mathcal{O}(n^2)$ worst-case | $\mathcal{O}(n \log n)$ | $\mathbf{\mathcal{O}(n)}$ |
| **Auxiliary Space** | $\mathcal{O}(n)$ | $\mathcal{O}(n \log n)$ | $\mathbf{\mathcal{O}(n)}$ |
| **Max Operations ($n=10^5$)** | $\sim 10^{10}$ | $\sim 1.7 \times 10^6$ | $\mathbf{\sim 2 \times 10^5}$ |
| **LeetCode Status** | Time Limit Exceeded (TLE) | Accepted | **Accepted (Top 99% Runtime)** |
| **Implementation Complexity** | Low | High | **Clean & Idiomatic** |

---

## 9. Production Implementations in 7 Languages

### Java (OpenJDK 21)

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

class Solution {
    public int[] findSubtreeSizes(int[] parent, String s) {
        int n = parent.length;

        // Step 1: Construct original tree adjacency list
        List<Integer>[] adj = new ArrayList[n];
        for (int i = 0; i < n; i++) {
            adj[i] = new ArrayList<>();
        }
        for (int i = 1; i < n; i++) {
            adj[parent[i]].add(i);
        }

        // Step 2: DFS with backtracking to find new parent for each node
        int[] newParent = Arrays.copyOf(parent, n);
        int[] lastSeen = new int[26];
        Arrays.fill(lastSeen, -1);

        dfsFindNewParents(0, adj, s, lastSeen, newParent);

        // Step 3: Construct restructured tree adjacency list
        List<Integer>[] newAdj = new ArrayList[n];
        for (int i = 0; i < n; i++) {
            newAdj[i] = new ArrayList<>();
        }
        for (int i = 1; i < n; i++) {
            newAdj[newParent[i]].add(i);
        }

        // Step 4: DFS to calculate subtree sizes in restructured tree
        int[] ans = new int[n];
        dfsComputeSizes(0, newAdj, ans);

        return ans;
    }

    private void dfsFindNewParents(int u, List<Integer>[] adj, String s, int[] lastSeen, int[] newParent) {
        int charIdx = s.charAt(u) - 'a';
        int oldAncestor = lastSeen[charIdx];

        if (oldAncestor != -1) {
            newParent[u] = oldAncestor;
        }

        // Temporarily record current node as closest ancestor for this character
        lastSeen[charIdx] = u;

        for (int v : adj[u]) {
            dfsFindNewParents(v, adj, s, lastSeen, newParent);
        }

        // Backtrack
        lastSeen[charIdx] = oldAncestor;
    }

    private int dfsComputeSizes(int u, List<Integer>[] newAdj, int[] ans) {
        int size = 1;
        for (int v : newAdj[u]) {
            size += dfsComputeSizes(v, newAdj, ans);
        }
        ans[u] = size;
        return size;
    }
}
```

---

### Python3

```python
class Solution:
    def findSubtreeSizes(self, parent: list[int], s: str) -> list[int]:
        import sys
        sys.setrecursionlimit(200000)

        n = len(parent)

        # Step 1: Build original tree adjacency list
        adj = [[] for _ in range(n)]
        for i in range(1, n):
            adj[parent[i]].append(i)

        # Step 2: DFS with backtracking to reassign parents
        new_parent = list(parent)
        last_seen = [-1] * 26

        def dfs_parents(u: int) -> None:
            char_idx = ord(s[u]) - ord('a')
            old_ancestor = last_seen[char_idx]

            if old_ancestor != -1:
                new_parent[u] = old_ancestor

            last_seen[char_idx] = u
            for v in adj[u]:
                dfs_parents(v)
            last_seen[char_idx] = old_ancestor  # Backtrack

        dfs_parents(0)

        # Step 3: Build restructured tree adjacency list
        new_adj = [[] for _ in range(n)]
        for i in range(1, n):
            new_adj[new_parent[i]].append(i)

        # Step 4: DFS to compute subtree sizes
        ans = [0] * n

        def dfs_sizes(u: int) -> int:
            total_size = 1
            for v in new_adj[u]:
                total_size += dfs_sizes(v)
            ans[u] = total_size
            return total_size

        dfs_sizes(0)
        return ans
```

---

### C (GCC 13.2)

```c
#include <stdlib.h>
#include <string.h>

// Forward star representation for memory efficiency
typedef struct {
    int *head;
    int *to;
    int *nxt;
    int edge_cnt;
} Graph;

static void init_graph(Graph *g, int n) {
    g->head = (int *)malloc(n * sizeof(int));
    g->to = (int *)malloc(n * sizeof(int));
    g->nxt = (int *)malloc(n * sizeof(int));
    g->edge_cnt = 0;
    memset(g->head, -1, n * sizeof(int));
}

static void add_edge(Graph *g, int u, int v) {
    g->to[g->edge_cnt] = v;
    g->nxt[g->edge_cnt] = g->head[u];
    g->head[u] = g->edge_cnt++;
}

static void free_graph(Graph *g) {
    free(g->head);
    free(g->to);
    free(g->nxt);
}

static void dfs_find_parents(int u, const Graph *g, const char *s, int *last_seen, int *new_parent) {
    int c = s[u] - 'a';
    int old_anc = last_seen[c];

    if (old_anc != -1) {
        new_parent[u] = old_anc;
    }

    last_seen[c] = u;

    for (int e = g->head[u]; e != -1; e = g->nxt[e]) {
        dfs_find_parents(g->to[e], g, s, last_seen, new_parent);
    }

    last_seen[c] = old_anc; // Backtrack
}

static int dfs_compute_sizes(int u, const Graph *g, int *ans) {
    int sz = 1;
    for (int e = g->head[u]; e != -1; e = g->nxt[e]) {
        sz += dfs_compute_sizes(g->to[e], g, ans);
    }
    ans[u] = sz;
    return sz;
}

/**
 * Note: The returned array must be malloced, assume caller calls free().
 */
int* findSubtreeSizes(int* parent, int parentSize, char* s, int* returnSize) {
    int n = parentSize;
    *returnSize = n;

    // Step 1: Build original tree
    Graph orig_tree;
    init_graph(&orig_tree, n);
    for (int i = 1; i < n; i++) {
        add_edge(&orig_tree, parent[i], i);
    }

    // Step 2: Find new parents
    int *new_parent = (int *)malloc(n * sizeof(int));
    memcpy(new_parent, parent, n * sizeof(int));

    int last_seen[26];
    for (int i = 0; i < 26; i++) last_seen[i] = -1;

    dfs_find_parents(0, &orig_tree, s, last_seen, new_parent);
    free_graph(&orig_tree);

    // Step 3: Build new tree
    Graph new_tree;
    init_graph(&new_tree, n);
    for (int i = 1; i < n; i++) {
        add_edge(&new_tree, new_parent[i], i);
    }
    free(new_parent);

    // Step 4: Compute subtree sizes
    int *ans = (int *)malloc(n * sizeof(int));
    dfs_compute_sizes(0, &new_tree, ans);
    free_graph(&new_tree);

    return ans;
}
```

---

### C++ (C++17)

```cpp
#include <vector>
#include <string>
#include <functional>

class Solution {
public:
    std::vector<int> findSubtreeSizes(std::vector<int>& parent, std::string s) {
        int n = static_cast<int>(parent.size());

        // Step 1: Build original tree adjacency list
        std::vector<std::vector<int>> adj(n);
        for (int i = 1; i < n; ++i) {
            adj[parent[i]].push_back(i);
        }

        // Step 2: DFS with backtracking to find closest ancestors with identical character
        std::vector<int> newParent = parent;
        std::vector<int> lastSeen(26, -1);

        auto dfsParents = [&](auto& self, int u) -> void {
            int charIdx = s[u] - 'a';
            int oldAncestor = lastSeen[charIdx];

            if (oldAncestor != -1) {
                newParent[u] = oldAncestor;
            }

            lastSeen[charIdx] = u;

            for (int v : adj[u]) {
                self(self, v);
            }

            lastSeen[charIdx] = oldAncestor; // Backtrack
        };

        dfsParents(dfsParents, 0);

        // Step 3: Build restructured tree adjacency list
        std::vector<std::vector<int>> newAdj(n);
        for (int i = 1; i < n; ++i) {
            newAdj[newParent[i]].push_back(i);
        }

        // Step 4: Post-order DFS to aggregate subtree sizes
        std::vector<int> ans(n, 0);
        auto dfsSizes = [&](auto& self, int u) -> int {
            int size = 1;
            for (int v : newAdj[u]) {
                size += self(self, v);
            }
            return ans[u] = size;
        };

        dfsSizes(dfsSizes, 0);
        return ans;
    }
};
```

---

### C#

```csharp
using System;
using System.Collections.Generic;

public class Solution {
    public int[] FindSubtreeSizes(int[] parent, string s) {
        int n = parent.Length;

        // Step 1: Build original tree adjacency list
        List<int>[] adj = new List<int>[n];
        for (int i = 0; i < n; i++) {
            adj[i] = new List<int>();
        }
        for (int i = 1; i < n; i++) {
            adj[parent[i]].Add(i);
        }

        // Step 2: Find new parents using DFS with active ancestor tracking
        int[] newParent = (int[])parent.Clone();
        int[] lastSeen = new int[26];
        for (int i = 0; i < 26; i++) {
            lastSeen[i] = -1;
        }

        DfsParents(0, adj, s, lastSeen, newParent);

        // Step 3: Build restructured tree
        List<int>[] newAdj = new List<int>[n];
        for (int i = 0; i < n; i++) {
            newAdj[i] = new List<int>();
        }
        for (int i = 1; i < n; i++) {
            newAdj[newParent[i]].Add(i);
        }

        // Step 4: Compute subtree sizes
        int[] ans = new int[n];
        DfsSizes(0, newAdj, ans);

        return ans;
    }

    private void DfsParents(int u, List<int>[] adj, string s, int[] lastSeen, int[] newParent) {
        int charIdx = s[u] - 'a';
        int oldAncestor = lastSeen[charIdx];

        if (oldAncestor != -1) {
            newParent[u] = oldAncestor;
        }

        lastSeen[charIdx] = u;

        foreach (int v in adj[u]) {
            DfsParents(v, adj, s, lastSeen, newParent);
        }

        lastSeen[charIdx] = oldAncestor; // Backtrack
    }

    private int DfsSizes(int u, List<int>[] newAdj, int[] ans) {
        int size = 1;
        foreach (int v in newAdj[u]) {
            size += DfsSizes(v, newAdj, ans);
        }
        ans[u] = size;
        return size;
    }
}
```

---

### JavaScript (Node.js v22)

```javascript
/**
 * Problem: Find Subtree Sizes After Changes
 * Platform: LeetCode 3331
 * Language: JavaScript (Node.js v22)
 * 
 * @param {number[]} parent
 * @param {string} s
 * @return {number[]}
 */
var findSubtreeSizes = function(parent, s) {
    const n = parent.length;

    // Step 1: Build original tree adjacency list
    const adj = Array.from({ length: n }, () => []);
    for (let i = 1; i < n; i++) {
        adj[parent[i]].push(i);
    }

    // Step 2: DFS to determine updated parents
    const newParent = [...parent];
    const lastSeen = new Array(26).fill(-1);

    function dfsParents(u) {
        const charIdx = s.charCodeAt(u) - 97;
        const oldAncestor = lastSeen[charIdx];

        if (oldAncestor !== -1) {
            newParent[u] = oldAncestor;
        }

        lastSeen[charIdx] = u;

        for (const v of adj[u]) {
            dfsParents(v);
        }

        lastSeen[charIdx] = oldAncestor; // Backtrack
    }

    dfsParents(0);

    // Step 3: Build restructured tree adjacency list
    const newAdj = Array.from({ length: n }, () => []);
    for (let i = 1; i < n; i++) {
        newAdj[newParent[i]].push(i);
    }

    // Step 4: Post-order DFS to calculate subtree sizes
    const ans = new Array(n).fill(0);

    function dfsSizes(u) {
        let size = 1;
        for (const v of newAdj[u]) {
            size += dfsSizes(v);
        }
        ans[u] = size;
        return size;
    }

    dfsSizes(0);
    return ans;
};
```

---

### TypeScript

```typescript
/**
 * Problem: Find Subtree Sizes After Changes
 * Platform: LeetCode 3331
 * Language: TypeScript
 */
function findSubtreeSizes(parent: number[], s: string): number[] {
    const n = parent.length;

    // Step 1: Construct original tree
    const adj: number[][] = Array.from({ length: n }, () => []);
    for (let i = 1; i < n; i++) {
        adj[parent[i]].push(i);
    }

    // Step 2: DFS with backtracking for new parents
    const newParent: number[] = [...parent];
    const lastSeen: number[] = new Array(26).fill(-1);

    function dfsParents(u: number): void {
        const charIdx = s.charCodeAt(u) - 97;
        const oldAncestor = lastSeen[charIdx];

        if (oldAncestor !== -1) {
            newParent[u] = oldAncestor;
        }

        lastSeen[charIdx] = u;

        for (const v of adj[u]) {
            dfsParents(v);
        }

        lastSeen[charIdx] = oldAncestor; // Backtrack
    }

    dfsParents(0);

    // Step 3: Construct restructured tree
    const newAdj: number[][] = Array.from({ length: n }, () => []);
    for (let i = 1; i < n; i++) {
        newAdj[newParent[i]].push(i);
    }

    // Step 4: DFS to compute subtree sizes
    const ans: number[] = new Array(n).fill(0);

    function dfsSizes(u: number): number {
        let size = 1;
        for (const v of newAdj[u]) {
            size += dfsSizes(v);
        }
        ans[u] = size;
        return size;
    }

    dfsSizes(0);
    return ans;
}
```

---

## 10. Edge Cases & Boundary Analysis

1. **Star Graph / Depth 1:**
   - Every node $i \ge 1$ is a direct child of root `0`.
   - Node $0$ is the only ancestor. If $s[i] == s[0]$, node $i$'s parent remains $0$. If $s[i] \neq s[0]$, no matching ancestor exists $\implies$ parent remains $0$. Tree structure is invariant.
2. **All Nodes Share the Same Character (`s = "aaaaa..."`):**
   - For every node $x \ge 1$, walking up its ancestor path encounters node `0` (or intermediate ancestors). But note that the problem requires the **closest** ancestor!
   - Under `last_seen[0]`, the parent of $x$ becomes its immediate parent $parent[x]$, so no reassignments actually alter the tree! The tree retains its original shape.
3. **All Nodes Have Distinct Characters (`s = "abcdef..."`):**
   - No node will find any ancestor with a matching character.
   - `last_seen[s[u] - 'a']` is always `-1` when visited.
   - All parents remain completely untouched. Output is identical to subtree sizes of the original tree.
4. **Deepest Ancestor Selection:**
   - If node $u$ has ancestors $A_1 \to A_2 \to A_3$ that all share the same character $c$, top-down DFS ensures that `last_seen[c]` is updated as we travel downward, so when we reach $u$, `last_seen[c]` contains the closest ancestor ($A_3$), strictly fulfilling the problem specification.
5. **Skewed Degenerate Line Graph ($n = 10^5$):**
   - Handled seamlessly in $\mathcal{O}(n)$ time without stack overflow using appropriate recursion depth limits or efficient forward star structures.

---

## 11. Key Takeaways & Interview Cheat-Sheet

| Strategy | When to Apply | Why It Works |
| :--- | :--- | :--- |
| **Inverting the Search Direction (Top-Down vs Bottom-Up)** | Whenever a problem asks for the "closest ancestor with property $P$" along paths from the root. | Traversing top-down means the active path is already an ordered ancestor chain. We can store properties in a small lookup array instead of climbing back up. |
| **Backtracking on Trees** | Whenever an active path state must only be visible to descendants, not siblings. | Restoring `last_seen[c] = old_val` upon leaving the DFS node ensures state isolation across subtrees. |
| **Simultaneous Mutations** | Whenever problem statements declare "simultaneous modifications". | Always separate the **discovery/planning phase** (on the immutable original state) from the **rebuilding/evaluation phase**. Never mutate the graph mid-traversal! |
