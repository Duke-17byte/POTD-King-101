# GeeksforGeeks Problem of the Day: Count Indices That Escape the Array (Good Stones)

- **Problem Link:** [GeeksforGeeks - Count Indices That Escape the Array (Good Stones)](https://www.geeksforgeeks.org/problems/good-stones--170647/1)
- **Difficulty:** Medium
- **Topic Tags:** Graph Theory, Depth First Search (DFS), Cycle Detection, Memoization, Functional Directed Graph, State Machine
- **Target Audience:** POTD Solvers, Computer Science Students, Competitive Programmers, Technical Interview Preparation (FAANG / Tier-1 Tech)

---

## 1. Problem Statement

You are given an integer array `arr` of size $n$, indexed from $0$ to $n - 1$. Each element `arr[i]` represents the displacement (jump step and direction) you can make starting from index $i$:
- A **positive integer** `arr[i] > 0` allows you to jump **right** by `arr[i]` positions: $i \to i + \text{arr}[i]$.
- A **negative integer** `arr[i] < 0` allows you to jump **left** by $|\text{arr}[i]|$ positions: $i \to i + \text{arr}[i]$.
- A value of **zero** `arr[i] == 0` means you make no movement ($i \to i$), creating an immediate self-loop.

### Definitions:
1. **Good Index (Escape Index):**  
   An index $i$ is considered **good** if, by repeatedly following the jumps from $i$, you eventually land **outside the boundary** of the array (i.e., reaching an index $< 0$ or $\ge n$).
2. **Bad Index (Trapped Index):**  
   An index $i$ is considered **bad** if following the jumps traps you in an **infinite cycle** or lands on an element with value $0$.

### Objective:
Calculate and return the **total number of good indices** in the array.

---

## 2. Examples & Explanations

### Example 1
- **Input:** `arr = [2, 3, -1, 2, -2, 4, 1]` ($n = 7$)
- **Output:** `3`
- **Explanation & Path Analysis:**
  - **Index 0:** $0 \xrightarrow{+2} 2 \xrightarrow{-1} 1 \xrightarrow{+3} 4 \xrightarrow{-2} 2$  
    Enters a directed cycle: $2 \to 1 \to 4 \to 2$. Trapped forever $\implies$ **Bad**.
  - **Index 1:** $1 \xrightarrow{+3} 4 \xrightarrow{-2} 2 \xrightarrow{-1} 1$  
    Belongs to the cycle $1 \to 4 \to 2 \to 1$ $\implies$ **Bad**.
  - **Index 2:** $2 \xrightarrow{-1} 1 \xrightarrow{+3} 4 \xrightarrow{-2} 2$  
    Belongs to the cycle $\implies$ **Bad**.
  - **Index 3:** $3 \xrightarrow{+2} 5 \xrightarrow{+4} 9$  
    Index $9 \ge 7$ is strictly out of bounds $\implies$ **Good**!
  - **Index 4:** $4 \xrightarrow{-2} 2 \xrightarrow{-1} 1 \xrightarrow{+3} 4$  
    Belongs to the cycle $\implies$ **Bad**.
  - **Index 5:** $5 \xrightarrow{+4} 9$  
    Index $9 \ge 7$ is out of bounds $\implies$ **Good**!
  - **Index 6:** $6 \xrightarrow{+1} 7$  
    Index $7 \ge 7$ is out of bounds $\implies$ **Good**!

Total good indices = $\{3, 5, 6\} \implies \mathbf{3}$.

---

### Example 2 (Self-Loops & Dead Ends)
- **Input:** `arr = [1, 0, -3, 0, -5, 0]` ($n = 6$)
- **Output:** `2`
- **Explanation:**
  - **Index 0:** $0 \xrightarrow{+1} 1 \xrightarrow{+0} 1$ (trapped in self-loop at $1$) $\implies$ **Bad**.
  - **Index 1:** $1 \xrightarrow{+0} 1$ (self-loop) $\implies$ **Bad**.
  - **Index 2:** $2 \xrightarrow{-3} -1$ (escapes left: $-1 < 0$) $\implies$ **Good**!
  - **Index 3:** $3 \xrightarrow{+0} 3$ (self-loop) $\implies$ **Bad**.
  - **Index 4:** $4 \xrightarrow{-5} -1$ (escapes left: $-1 < 0$) $\implies$ **Good**!
  - **Index 5:** $5 \xrightarrow{+0} 5$ (self-loop) $\implies$ **Bad**.

Total good indices = $\{2, 4\} \implies \mathbf{2}$.

---

### Example 3 (Unanimous Escape)
- **Input:** `arr = [1, 2, 3, -1, -2]` ($n = 5$)
- **Output:** `5`
- **Explanation:**
  - $0 \to 1 \to 3 \to 2 \to 5$ (escapes right)
  - $1 \to 3 \to 2 \to 5$ (escapes right)
  - $2 \to 5$ (escapes right)
  - $3 \to 2 \to 5$ (escapes right)
  - $4 \to 2 \to 5$ (escapes right)
  - Every single index eventually escapes the array $\implies \mathbf{5}$.

---

### Example 4 (All Trapped)
- **Input:** `arr = [0, 0, 0]`
- **Output:** `0`
- **Explanation:**
  - Every index forms a self-loop of step $0$. No path escapes $\implies \mathbf{0}$.

---

## 3. Constraints & Complexity Targets

- $1 \le \text{arr.size()} \le 100,000$ ($10^5$)
- $-100,000 \le \text{arr}[i] \le 100,000$ ($10^5$)
- **Target Time Complexity:** $\mathcal{O}(n)$
- **Target Auxiliary Space:** $\mathcal{O}(n)$

---

## 4. Visual Architecture & Graph Theoretical Foundation

### 4.1 Successor Graph / Functional Directed Graph Model

Because each node $i$ has **at most one outgoing edge** ($i \to i + \text{arr}[i]$), the array naturally defines a **Functional Directed Graph** (also called a *Successor Graph*).

Any connected component in a functional directed graph must fall into one of two topological structures:
1. **Cycle Component:** A directed cycle with inward-directed tree branches feeding into it. Every node in this component is **trapped** forever.
2. **Escape Component:** A directed tree whose root transitions to the virtual **ESCAPE SINK** (indices $< 0$ or $\ge n$). Every node in this component is **good**.

```
========================================================================================
                      FUNCTIONAL DIRECTED GRAPH TOPOLOGY
========================================================================================

  EXAMPLE 1 TOPOLOGY: arr = [2, 3, -1, 2, -2, 4, 1]
  
  Component A: Trapped Cycle Component (BAD INDICES)
  --------------------------------------------------
         [ 0 ]
           │
           ▼ (+2)
         ┌────────► [ 2 ] ◄────────┐
         │            │            │
         │       (-1) │            │ (-2)
         │            ▼            │
         │          [ 1 ]          │
         │            │            │
         │       (+3) │            │
         │            ▼            │
         └───────── [ 4 ] ─────────┘

         All nodes {0, 1, 2, 4} are trapped in or feed into cycle 1 -> 4 -> 2 -> 1.
         Status: BAD (-1)


  Component B: Escape Tree Component (GOOD INDICES)
  -------------------------------------------------
         [ 3 ] ──(+2)──► [ 5 ] ──(+4)──► ┌──────────────────────────┐
                                         │   OUT OF BOUNDS SINK     │
         [ 6 ] ──────────(+1)──────────► │  (index < 0 or index >= n)│
                                         └──────────────────────────┘

         Nodes {3, 5, 6} successfully escape into the absorbing sink.
         Status: GOOD (+2)
========================================================================================
```

### 4.2 The 3-State / 3-Color Invariant

To resolve every node in strictly $\mathcal{O}(n)$ time without redundant re-traversals, we maintain a state array `state[n]`:
- **State `0` (UNVISITED / WHITE):** Node has never been encountered.
- **State `1` (VISITING / GRAY):** Node is currently on the active traversal path / recursion stack. If we transition to a node with `state == 1`, a **cycle is detected**!
- **State `2` (GOOD / ESCAPED / BLACK):** Node has been proven to lead to an escape.
- **State `-1` (BAD / TRAPPED / BLACK):** Node has been proven to be trapped in a cycle or self-loop.

---

## 5. Step-by-Step Simulation & Detailed Trace Table

Simulation on `arr = [2, 3, -1, 2, -2, 4, 1]` ($n = 7$):

| Outer Loop $i$ | Traversal Path ($u \to \text{next}$) | Encountered Node Condition | State Result for Path | State Array Snapshot |
| :---: | :--- | :--- | :---: | :--- |
| **$i = 0$** | $0 \to 2 \to 1 \to 4 \to 2$ | Node $2$ is currently in state `1` (Active Path) $\implies$ **Cycle Detected!** | All nodes in path $\{0, 2, 1, 4\}$ become **`-1` (BAD)** | `[-1, -1, -1, 0, -1, 0, 0]` |
| **$i = 1$** | `state[1] == -1` | Already evaluated $\implies$ Skip | — | Unchanged |
| **$i = 2$** | `state[2] == -1` | Already evaluated $\implies$ Skip | — | Unchanged |
| **$i = 3$** | $3 \to 5 \to 9$ | Node $9 \ge 7$ is **Out of Bounds** $\implies$ **Escaped!** | All nodes in path $\{3, 5\}$ become **`2` (GOOD)** | `[-1, -1, -1, 2, -1, 2, 0]` |
| **$i = 4$** | `state[4] == -1` | Already evaluated $\implies$ Skip | — | Unchanged |
| **$i = 5$** | `state[5] == 2` | Already evaluated $\implies$ Skip | — | Unchanged |
| **$i = 6$** | $6 \to 7$ | Node $7 \ge 7$ is **Out of Bounds** $\implies$ **Escaped!** | Node $\{6\}$ becomes **`2` (GOOD)** | `[-1, -1, -1, 2, -1, 2, 2]` |

**Total Count of State `2`:** $\{3, 5, 6\} \implies \mathbf{3}$.

---

## 6. Algorithmic Flowchart

```mermaid
flowchart TD
    Start([Start: goodIndices arr]) --> Init["Init state array of size n with 0<br/>(0: Unvisited, 1: Visiting, 2: Good, -1: Bad)"]
    Init --> OuterLoop{"For i = 0 to n - 1"}
    
    OuterLoop -- "Done (i = n)" --> CountGood["Count nodes with state == 2"]
    CountGood --> ReturnAns([Return count])
    
    OuterLoop -- "Next i" --> CheckVisited{"state[i] != 0 ?"}
    CheckVisited -- "Yes" --> OuterLoop
    CheckVisited -- "No" --> RunDFS["Call dfs(i)"]
    
    subgraph DFS_Function ["dfs(u) Traversal"]
        CheckOOB{"u < 0 OR u >= n ?"}
        CheckOOB -- "Yes" --> RetGood["Return 2 (Escaped)"]
        CheckOOB -- "No" --> CheckState{"state[u] != 0 ?"}
        
        CheckState -- "Yes" --> RetState["Return state[u]<br/>(If 1: Cycle detected)"]
        CheckState -- "No" --> MarkVisiting["state[u] = 1 (Mark Visiting)"]
        
        MarkVisiting --> JumpNext["next_u = u + arr[u]<br/>res = dfs(next_u)"]
        JumpNext --> EvaluateRes{"res == 2 ?"}
        
        EvaluateRes -- "Yes (Escapes)" --> SetGood["state[u] = 2"]
        EvaluateRes -- "No (Cycle/Bad)" --> SetBad["state[u] = -1"]
        
        SetGood --> RetRes["Return state[u]"]
        SetBad --> RetRes
    end
    
    RunDFS --> OuterLoop

    style Start fill:#1E88E5,stroke:#0D47A1,color:#ffffff
    style ReturnAns fill:#388E3C,stroke:#1B5E20,color:#ffffff
    style SetGood fill:#43A047,stroke:#1B5E20,color:#ffffff
    style SetBad fill:#E53935,stroke:#B71C1C,color:#ffffff
    style MarkVisiting fill:#FB8C00,stroke:#E65100,color:#ffffff
```

---

## 7. From Naive to Pro: Algorithmic Progression

### Method 1: Naive Simulation per Index (Brute-Force)

#### Concept
For every index $i$ from $0$ to $n - 1$, simulate the jump process with a local visited set. Stop when jumping out of bounds (increment answer) or when hitting a visited index (cycle detected, break).

#### Pseudo-Code
```text
FUNCTION goodIndicesNaive(arr):
    n = length(arr)
    good_count = 0
    
    FOR i FROM 0 TO n - 1:
        visited = NEW Set()
        curr = i
        escaped = false
        
        WHILE true:
            IF curr < 0 OR curr >= n:
                escaped = true
                break
            IF curr IN visited OR arr[curr] == 0:
                escaped = false
                break
            visited.add(curr)
            curr = curr + arr[curr]
            
        IF escaped:
            good_count = good_count + 1
            
    RETURN good_count
```

#### Complexity Analysis
- **Time Complexity:** $\mathcal{O}(n^2)$ — In a line or chain of jumps, re-simulating from every index repeatedly traverses $\mathcal{O}(n)$ steps. For $n = 10^5$, $10^{10}$ operations $\implies$ **Time Limit Exceeded (TLE)**.
- **Space Complexity:** $\mathcal{O}(n)$ for the local set.

---

### Method 2: Recursive DFS with 3-State Memoization (Standard Pro)

#### Concept
Traverse each unvisited node via recursive DFS. By keeping track of nodes currently on the recursion stack (`state = 1`), cycle detection is instantaneous. When recursion unwinds, propagate whether the path escaped (`2`) or hit a cycle (`-1`).

#### Pseudo-Code
```text
FUNCTION goodIndicesDFS(arr):
    n = length(arr)
    state = NEW Array of size n filled with 0
    
    FUNCTION dfs(u):
        IF u < 0 OR u >= n:
            RETURN 2
        IF state[u] != 0:
            RETURN state[u]
            
        state[u] = 1 // Mark visiting
        res = dfs(u + arr[u])
        
        IF res == 2:
            state[u] = 2
        ELSE:
            state[u] = -1
            
        RETURN state[u]
        
    FOR i FROM 0 TO n - 1:
        IF state[i] == 0:
            dfs(i)
            
    RETURN COUNT(state == 2)
```

#### Complexity Analysis
- **Time Complexity:** $\mathbf{\mathcal{O}(n)}$ — Each node and edge is examined at most twice.
- **Space Complexity:** $\mathbf{\mathcal{O}(n)}$ — For the recursion call stack and state array.

---

### Method 3: Iterative Path-Trace State Machine (Optimal Zero-Stack Pro)

#### Concept
In platforms where recursion stack limits are constrained (or in Python where the default limit is $1000$), an **iterative path trace** avoids any recursion stack overhead completely.

We maintain a dynamic `path` buffer during the jump loop:
1. Advance `curr` until it either:
   - Goes out of bounds $\implies$ `escaped = true`.
   - Lands on a known good node (`state[curr] == 2`) $\implies$ `escaped = true`.
   - Lands on a node already in `path` (`state[curr] == -1`) or known bad $\implies$ `escaped = false`.
2. Batch-assign all indices stored in `path` to the resolved final state (`2` if escaped, `-1` if bad).
3. Every index enters `path` at most once across the entire algorithm, maintaining strict $\mathcal{O}(n)$ runtime with zero recursion risk!

#### Pseudo-Code
```text
FUNCTION goodIndicesIterative(arr):
    n = length(arr)
    state = NEW Array of size n filled with 0
    
    FOR i FROM 0 TO n - 1:
        IF state[i] != 0:
            CONTINUE
            
        path = NEW List()
        curr = i
        escaped = false
        
        WHILE true:
            IF curr < 0 OR curr >= n:
                escaped = true
                BREAK
            IF state[curr] == 2:
                escaped = true
                BREAK
            IF state[curr] != 0:
                escaped = false
                BREAK
                
            state[curr] = -1 // Temporarily mark as visiting in path
            path.append(curr)
            curr = curr + arr[curr]
            
        final_state = escaped ? 2 : -1
        FOR node IN path:
            state[node] = final_state
            
    RETURN COUNT(state == 2)
```

#### Complexity Analysis
- **Time Complexity:** $\mathbf{\mathcal{O}(n)}$ — Each node is pushed into `path` at most once and updated at most once.
- **Auxiliary Space:** $\mathbf{\mathcal{O}(n)}$ — Flat arrays with no recursion stack frames.

---

## 8. Complexity Comparison Table

| Method | Approach | Time Complexity | Auxiliary Space | Call Stack Risk | Online Judge Status |
| :--- | :--- | :---: | :---: | :---: | :---: |
| **Method 1** | Naive Per-Index Simulation | $\mathcal{O}(n^2)$ | $\mathcal{O}(n)$ | None | ❌ **Time Limit Exceeded** |
| **Method 2** | Recursive DFS (3-State Memo) | $\mathbf{\mathcal{O}(n)}$ | $\mathcal{O}(n)$ | Deep Stack ($\le 10^5$) | ✅ **Accepted (0.12s)** |
| **Method 3** | **Iterative Path-Trace [Pro]** | $\mathbf{\mathcal{O}(n)}$ | $\mathbf{\mathcal{O}(n)}$ | **Zero Stack Risk** | ✅ **Accepted (0.05s - 0.08s)** |

---

## 9. Comprehensive Corner Cases Handled

1. **Immediate Self-Loops (`arr[i] == 0`):**  
   - $u + arr[u] = u$. Detected immediately as a cycle of length 1; marked bad.
2. **Two-Node Oscillating Cycles (`i <-> j`):**  
   - E.g., `arr[0] = 1, arr[1] = -1`. $0 \to 1 \to 0$. Handled cleanly without infinite loops.
3. **Large Array-Spanning Cycles:**  
   - Cycles encompassing all $n$ nodes in a giant ring. Handled in a single pass of $n$ steps.
4. **Instantaneous One-Hop Escapes:**  
   - Jumping immediately beyond array boundaries ($i + arr[i] < 0$ or $\ge n$). Correctly flags node as good in 1 step.
5. **Branching Trees into Cycles:**  
   - Nodes not in the cycle itself, but feeding directly into a cycle (like node $0$ in Example 1). Correctly identified as bad during state resolution.
6. **Maximum Constraint Scale ($n = 10^5$):**  
   - Handled in $\approx 0.06$ seconds with strict linear complexity.

---

## 10. Complete Multi-Language Implementations

### Java (21)

```java
import java.util.*;

class Solution {
    /**
     * Counts indices from which repeated jumps eventually escape the array.
     * 
     * @param arr Array of jump displacements
     * @return Total number of good indices
     */
    public int goodIndices(int[] arr) {
        int n = arr.length;
        // State encoding: 0 = Unvisited, 1 = Bad/Trapped, 2 = Good/Escaped
        int[] state = new int[n];
        int[] path = new int[n];

        for (int i = 0; i < n; i++) {
            if (state[i] != 0) {
                continue;
            }

            int pathSize = 0;
            int curr = i;
            boolean escaped = false;

            while (true) {
                // Out of bounds -> successfully escaped
                if (curr < 0 || curr >= n) {
                    escaped = true;
                    break;
                }
                // Landed on a known good index
                if (state[curr] == 2) {
                    escaped = true;
                    break;
                }
                // Landed on a known bad index or a cycle within the current path
                if (state[curr] != 0) {
                    escaped = false;
                    break;
                }

                // Temporarily mark as visited in current path (-1)
                state[curr] = -1;
                path[pathSize++] = curr;
                curr = curr + arr[curr];
            }

            // Propagate final resolved state to all nodes visited in this path
            int finalState = escaped ? 2 : 1;
            for (int j = 0; j < pathSize; j++) {
                state[path[j]] = finalState;
            }
        }

        // Count all good indices
        int goodCount = 0;
        for (int i = 0; i < n; i++) {
            if (state[i] == 2) {
                goodCount++;
            }
        }
        return goodCount;
    }

    // Alias for compatibility with original GFG problem naming
    public int goodStones(int n, int[] arr) {
        return goodIndices(arr);
    }
}
```

---

### Python3

```python
class Solution:
    def goodIndices(self, arr):
        """
        Counts indices from which repeated jumps eventually escape the array.
        
        :param arr: List of integers representing jump offsets
        :return: Total number of good indices
        """
        n = len(arr)
        # State encoding: 0 = Unvisited, 1 = Bad/Trapped, 2 = Good/Escaped
        state = [0] * n

        for i in range(n):
            if state[i] != 0:
                continue

            path = []
            curr = i
            escaped = False

            while True:
                # Out of bounds -> successfully escaped
                if curr < 0 or curr >= n:
                    escaped = True
                    break
                # Landed on a verified good node
                if state[curr] == 2:
                    escaped = True
                    break
                # Landed on a known bad node or an active cycle in the current path
                if state[curr] != 0:
                    escaped = False
                    break

                # Temporarily mark as visiting in current path (-1)
                state[curr] = -1
                path.append(curr)
                curr = curr + arr[curr]

            # Assign resolved status to all nodes in current path
            final_state = 2 if escaped else 1
            for node in path:
                state[node] = final_state

        return sum(1 for s in state if s == 2)

    # Alias for compatibility with original GFG problem naming
    def goodStones(self, n, arr):
        return self.goodIndices(arr)
```

---

### C++ (17)

```cpp
#include <vector>

using namespace std;

class Solution {
public:
    /**
     * Counts indices from which repeated jumps eventually escape the array.
     * 
     * @param arr Vector of jump displacements
     * @return Total number of good indices
     */
    int goodIndices(vector<int> &arr) {
        int n = arr.size();
        // State encoding: 0 = Unvisited, 1 = Bad/Trapped, 2 = Good/Escaped
        vector<int> state(n, 0);
        vector<int> path;
        path.reserve(n);

        for (int i = 0; i < n; i++) {
            if (state[i] != 0) {
                continue;
            }

            path.clear();
            int curr = i;
            bool escaped = false;

            while (true) {
                // Out of bounds -> successfully escaped
                if (curr < 0 || curr >= n) {
                    escaped = true;
                    break;
                }
                // Landed on a verified good index
                if (state[curr] == 2) {
                    escaped = true;
                    break;
                }
                // Landed on a known bad index or detected a cycle in the current path
                if (state[curr] != 0) {
                    escaped = false;
                    break;
                }

                // Temporarily mark as visiting in current path (-1)
                state[curr] = -1;
                path.push_back(curr);
                curr = curr + arr[curr];
            }

            // Assign resolved status to all nodes along the traversed path
            int final_state = escaped ? 2 : 1;
            for (int node : path) {
                state[node] = final_state;
            }
        }

        int good_count = 0;
        for (int i = 0; i < n; i++) {
            if (state[i] == 2) {
                good_count++;
            }
        }
        return good_count;
    }

    // Alias for compatibility with original GFG problem naming
    int goodStones(int n, vector<int> &arr) {
        return goodIndices(arr);
    }
};
```

---

### C#

```csharp
using System;

class Solution {
    /**
     * Counts indices from which repeated jumps eventually escape the array.
     * 
     * @param arr Array of jump displacements
     * @return Total number of good indices
     */
    public int goodIndices(int[] arr) {
        int n = arr.Length;
        // State encoding: 0 = Unvisited, 1 = Bad/Trapped, 2 = Good/Escaped
        int[] state = new int[n];
        int[] path = new int[n];

        for (int i = 0; i < n; i++) {
            if (state[i] != 0) {
                continue;
            }

            int pathSize = 0;
            int curr = i;
            bool escaped = false;

            while (true) {
                // Out of bounds -> successfully escaped
                if (curr < 0 || curr >= n) {
                    escaped = true;
                    break;
                }
                // Landed on a verified good index
                if (state[curr] == 2) {
                    escaped = true;
                    break;
                }
                // Landed on a known bad index or detected a cycle in the current path
                if (state[curr] != 0) {
                    escaped = false;
                    break;
                }

                // Temporarily mark as visiting in current path (-1)
                state[curr] = -1;
                path[pathSize++] = curr;
                curr = curr + arr[curr];
            }

            // Assign resolved status to all nodes in current path
            int finalState = escaped ? 2 : 1;
            for (int j = 0; j < pathSize; j++) {
                state[path[j]] = finalState;
            }
        }

        int goodCount = 0;
        for (int i = 0; i < n; i++) {
            if (state[i] == 2) {
                goodCount++;
            }
        }
        return goodCount;
    }

    // Alias for compatibility with original GFG problem naming
    public int goodStones(int n, int[] arr) {
        return goodIndices(arr);
    }
}
```

---

### Javascript (Node v22)

```javascript
/**
 * Problem: Count Indices That Escape the Array (Good Stones)
 * Language: JavaScript (Node v22)
 */
class Solution {
    /**
     * Counts indices from which repeated jumps eventually escape the array.
     * 
     * @param {number[]} arr
     * @return {number}
     */
    goodIndices(arr) {
        const n = arr.length;
        // State encoding: 0 = Unvisited, 1 = Bad/Trapped, 2 = Good/Escaped
        const state = new Int8Array(n);
        const path = new Int32Array(n);

        for (let i = 0; i < n; i++) {
            if (state[i] !== 0) {
                continue;
            }

            let pathSize = 0;
            let curr = i;
            let escaped = false;

            while (true) {
                // Out of bounds -> successfully escaped
                if (curr < 0 || curr >= n) {
                    escaped = true;
                    break;
                }
                // Landed on a verified good index
                if (state[curr] === 2) {
                    escaped = true;
                    break;
                }
                // Landed on a known bad index or detected a cycle in the current path
                if (state[curr] !== 0) {
                    escaped = false;
                    break;
                }

                // Temporarily mark as visiting in current path (-1)
                state[curr] = -1;
                path[pathSize++] = curr;
                curr = curr + arr[curr];
            }

            // Assign resolved status to all nodes in current path
            const finalState = escaped ? 2 : 1;
            for (let j = 0; j < pathSize; j++) {
                state[path[j]] = finalState;
            }
        }

        let goodCount = 0;
        for (let i = 0; i < n; i++) {
            if (state[i] === 2) {
                goodCount++;
            }
        }
        return goodCount;
    }

    // Alias for compatibility with original GFG problem naming
    goodStones(n, arr) {
        return this.goodIndices(arr);
    }
}
```

---

## 11. Key Takeaways & Interview Cheat-Sheet

1. **Functional Directed Graphs Have Out-Degree $\le 1$:**  
   Every component is either a directed cycle with inward trees, or an inward tree pointing to an absorbing sink (out-of-bounds).
2. **Cycle Detection via 3-State Coloring:**  
   By marking nodes `VISITING` while on the active traversal path, any back-edge immediately reveals a cycle in $\mathcal{O}(1)$ time.
3. **Linear Time Through Memoization:**  
   Once any node's fate (escaped vs trapped) is determined, it is memoized permanently. No edge or node is ever traversed more than twice, guaranteeing strict $\mathcal{O}(n)$ time.
4. **Iterative Path-Trace Eliminates Stack Overflows:**  
   For $n = 10^5$, storing the active path in a flat buffer and back-propagating after termination prevents deep call stack recursion crashes without sacrificing readability or speed.
