# GeeksforGeeks Problem of the Day: Minimum Days

- **Problem Link:** [GeeksforGeeks - Minimum Days](https://www.geeksforgeeks.org/problems/minimum-days--170646/1)
- **Difficulty:** Medium
- **Topic Tags:** Strings, Arrays, Greedy, State Tracking, Two Pointers / Neighbor Invariants, Simulation
- **Target Audience:** POTD Solvers, Computer Science Students, Competitive Programmers, Interview Preparation (Amazon, Google, Microsoft, Tier-1 Tech)

---

## 1. Problem Statement

Given a string `s` of length `n` containing lowercase English letters, and a permutation array `p[]` of length `n` containing integers from `0` to `n - 1`.

On the $(i + 1)^{\text{th}}$ day (for $0 \le i < n$), the character at index `p[i]` in the string `s` is replaced with the question mark character `'?'`.

Find the **minimum number of days** required so that no two adjacent characters in `s` are equal, unless at least one of them is `'?'`. If the string already satisfies this condition from day $0$, return `0`.

### Formal Definition

For every index $i$ ($0 \le i < n - 1$):
$$\text{Conflict}(i) \iff \Big(s[i] \neq \text{'?'} \;\land\; s[i+1] \neq \text{'?'} \;\land\; s[i] = s[i+1]\Big)$$

The condition is satisfied on day $D$ if and only if:
$$\sum_{i=0}^{n-2} [\text{Conflict}(i)] = 0$$

You need to find the smallest day $D \in \{0, 1, 2, \dots, n\}$ such that the total number of conflicts is zero.

---

## 2. Examples & Explanations

### Example 1
- **Input:** `s = "aabb"`, `p = [2, 1, 3, 0]`
- **Output:** `2`
- **Explanation:**
  - **Initial State (Day 0):** `s = "aabb"`.  
    - Pair $(0, 1)$: `'a' == 'a'` $\implies$ **Conflict 1**
    - Pair $(1, 2)$: `'a' != 'b'` $\implies$ No conflict
    - Pair $(2, 3)$: `'b' == 'b'` $\implies$ **Conflict 2**  
    Total conflicts = $2$. Condition not satisfied.
  - **Day 1:** Replace character at index `p[0] = 2` with `'?'`.  
    `s` becomes `"aa?b"`.  
    - Pair $(0, 1)$: `'a' == 'a'` $\implies$ **Conflict 1**
    - Pair $(1, 2)$: `'a'` and `'?'` $\implies$ No conflict
    - Pair $(2, 3)$: `'?'` and `'b'` $\implies$ No conflict  
    Total conflicts = $1$.
  - **Day 2:** Replace character at index `p[1] = 1` with `'?'`.  
    `s` becomes `"a??b"`.  
    - Pair $(0, 1)$: `'a'` and `'?'` $\implies$ No conflict
    - Pair $(1, 2)$: `'?'` and `'?'` $\implies$ No conflict
    - Pair $(2, 3)$: `'?'` and `'b'` $\implies$ No conflict  
    Total conflicts = $0$. All adjacent equal conflicts are resolved!
  - Therefore, the minimum number of days required is **$2$**.

```text
Day 0:   [ a | a | b | b ]   --> Conflicts: (0,1) and (2,3) = 2
Day 1:   [ a | a | ? | b ]   --> Replace index 2 ('b') -> 1 conflict remains: (0,1)
Day 2:   [ a | ? | ? | b ]   --> Replace index 1 ('a') -> 0 conflicts remain! -> Return 2
```

---

### Example 2
- **Input:** `s = "abca"`, `p = [3, 0, 2, 1]`
- **Output:** `0`
- **Explanation:**
  - Let us check all adjacent pairs in the initial string:
    - Pair $(0, 1)$: `'a' != 'b'`
    - Pair $(1, 2)$: `'b' != 'c'`
    - Pair $(2, 3)$: `'c' != 'a'`
  - There are $0$ adjacent equal pairs in the original string.
  - No replacements are needed; the answer is **$0$**.

---

### Example 3
- **Input:** `s = "aaaa"`, `p = [1, 2, 0, 3]`
- **Output:** `2`
- **Explanation:**
  - **Initial State:** `s = "aaaa"`, length $n = 4$.
    - Pairs $(0, 1)$, $(1, 2)$, $(2, 3)$ are all identical.  
    Total conflicts = $3$.
  - **Day 1 ($p[0] = 1$):** `s` becomes `"a?aa"`.  
    - Replacing index $1$ breaks pair $(0, 1)$ AND pair $(1, 2)$ simultaneously!
    - Remaining conflicts = $1$ (between index $2$ and $3$).
  - **Day 2 ($p[1] = 2$):** `s` becomes `"a??a"`.  
    - Replacing index $2$ breaks pair $(2, 3)$.
    - Remaining conflicts = $0$.
  - Result: **$2$** days.

---

### Example 4 (Simultaneous Dual Conflict Elimination)
- **Input:** `s = "abbba"`, `p = [2, 1, 3, 0, 4]`
- **Output:** `1`
- **Explanation:**
  - Initial conflicts: index $1$ with index $2$ (`'b' == 'b'`) and index $2$ with index $3$ (`'b' == 'b'`). Total conflicts = $2$.
  - **Day 1 ($p[0] = 2$):** Replacing index $2$ (the middle `'b'`) with `'?'` produces `"ab?ba"`.
  - Both conflict pairs $(1, 2)$ and $(2, 3)$ are eliminated in a single day!
  - Remaining conflicts = $0$. Result: **$1$** day.

---

## 3. Constraints

- $1 \le n \le 10^5$
- $s$ consists only of lowercase English letters (`'a'` - `'z'`)
- $p$ is a valid permutation of integers from $0$ to $n - 1$
- **Expected Time Complexity:** $\mathcal{O}(n)$
- **Expected Auxiliary Space:** $\mathcal{O}(1)$ (in languages allowing in-place mutations) or $\mathcal{O}(n)$ (for string to array conversion)

---

## 4. Visual Architecture & Key Theoretical Foundations (The Core Theorem)

### 1. The Conflict Graph Model

We can view the string $s$ as a linear path graph of $n$ vertices ($0 \dots n-1$), connected by $n-1$ undirected edges:
$$E = \{(0, 1), (1, 2), \dots, (n-2, n-1)\}$$

An edge $(i, i+1)$ is **active / in conflict** if and only if $s[i] = s[i+1]$ and neither character is `'?'`.

```text
Linear Chain Graph Representation:

Index:     0       1       2       3       4
Char:     [a] === [a] --- [b] === [b] --- [c]
           ^       ^       ^       ^       ^
           |=======|       |=======|
          Conflict 1      Conflict 2
```

### 2. The Locality of Mutations Invariant

When index $k = p[\text{day}]$ is replaced with `'?'`:
- The character at index $k$ changes from an English letter to `'?'`.
- **Crucial Observation:** This change can **ONLY** alter the state of at most **two edges**:
  1. The left edge: $(k-1, k)$, if $k > 0$.
  2. The right edge: $(k+1, k)$, if $k < n - 1$.
- No other edge $(i, i+1)$ in the entire graph (where $i \neq k-1$ and $i \neq k$) is affected in any way!

```text
Before Mutation at index k:
... [s[k-1]] ======= [s[k]] ======= [s[k+1]] ...
     Left Edge         ^         Right Edge
                   Target (k)

After Mutation s[k] = '?':
... [s[k-1]] ------- [ ? ] ------- [s[k+1]] ...
   (Defused)                   (Defused)
```

### 3. Monotonicity of Conflicts

Because `'?'` is a wildcard that **never** conflicts with any character (not even another `'?'`), replacing any character with `'?'`:
- Can **only decrease** the number of active conflicts (by $0$, $1$, or $2$).
- Can **never create** a new conflict!

Thus, the total conflict count $\text{conflicts}(t)$ is **monotonically non-increasing** with respect to days $t$:
$$\text{conflicts}(0) \ge \text{conflicts}(1) \ge \text{conflicts}(2) \ge \dots \ge \text{conflicts}(n) = 0$$

This guarantees:
1. We can maintain a single integer variable `equalPairs` tracking the active conflicts in $\mathcal{O}(1)$ time per day.
2. The very first day on which `equalPairs == 0` is guaranteed to be the global minimum answer.

---

## 5. Step-by-Step Simulation & Trace Table

Let us trace `s = "aabb"`, `p = [2, 1, 3, 0]` step-by-step:

| Day $t$ | Index Replaced $p[t-1]$ | Left Neighbor Check | Right Neighbor Check | Conflict Delta | Active Conflicts | String State | Terminate? |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **0** (Initial) | — | — | — | — | **$2$** | `"aabb"` | No ($2 > 0$) |
| **1** | $2$ (was `'b'`) | `s[1]=='a' != s[2]=='b'` (No) | `s[3]=='b' == s[2]=='b'` (Yes, $-1$) | $-1$ | **$1$** | `"aa?b"` | No ($1 > 0$) |
| **2** | $1$ (was `'a'`) | `s[0]=='a' == s[1]=='a'` (Yes, $-1$) | `s[2]=='?' != s[1]=='a'` (No) | $-1$ | **$0$** | `"a??b"` | **YES! Output: 2** |

---

## 6. Algorithm Flowchart

```mermaid
flowchart TD
    Start(["Start: getMinimumDays(s, p)"]) --> Init["Count initial adjacent equal pairs<br/>equalPairs = count(s[i] == s[i+1])"]
    Init --> CheckZero{"equalPairs == 0 ?"}
    CheckZero -- Yes --> RetZero["Return 0 (Already valid)"]
    CheckZero -- No --> LoopStart["Loop day from 0 to n - 1<br/>idx = p[day]"]
    
    LoopStart --> LeftCheck{"idx > 0 &&<br/>s[idx - 1] == s[idx] ?"}
    LeftCheck -- Yes --> DecLeft["equalPairs--"]
    LeftCheck -- No --> RightCheck
    DecLeft --> RightCheck
    
    RightCheck{"idx < n - 1 &&<br/>s[idx + 1] == s[idx] ?"}
    RightCheck -- Yes --> DecRight["equalPairs--"]
    RightCheck -- No --> MarkQ
    DecRight --> MarkQ
    
    MarkQ["Mutate: s[idx] = '?'"] --> ConflictCheck{"equalPairs == 0 ?"}
    ConflictCheck -- Yes --> RetDay["Return day + 1 (1-indexed Day)"]
    ConflictCheck -- No --> NextDay{"day < n - 1 ?"}
    NextDay -- Yes --> LoopStart
    NextDay -- No --> RetN["Return n"]
```

---

## 7. From Naive to Pro: Algorithmic Progression

### Approach 1: Naive Simulation (Day-by-Day Full Re-scan)

#### Concept
Simulate every day in chronological order. On day $t$, replace `s[p[t]] = '?'` and scan the entire string of length $n$ to check whether any two adjacent identical non-`'?'` characters still exist.

#### Pseudo Code
```text
ALGORITHM NaiveMinimumDays(s, p):
    n = length(s)
    IF isValid(s) THEN RETURN 0
    FOR day FROM 0 TO n - 1:
        s[p[day]] = '?'
        IF isValid(s) THEN:
            RETURN day + 1
    RETURN n

FUNCTION isValid(s):
    FOR i FROM 0 TO length(s) - 2:
        IF s[i] != '?' AND s[i] == s[i + 1]:
            RETURN FALSE
    RETURN TRUE
```

#### Complexity Analysis
- **Time Complexity:** $\mathcal{O}(n^2)$ — Up to $n$ days, each performing an $\mathcal{O}(n)$ verification scan. For $n = 10^5$, $n^2 = 10^{10}$ operations $\implies$ **Time Limit Exceeded (TLE)**.
- **Auxiliary Space:** $\mathcal{O}(n)$ to hold the mutable string.

---

### Approach 2: Binary Search on Number of Days

#### Concept
Because the validity predicate $P(D)$ ("Is the string valid after $D$ days?") is monotonic:
$$[F, F, \dots, F, T, T, \dots, T]$$
We can binary search for the minimum $D \in [0, n]$. To evaluate $P(D)$, create a deleted set of indices $\{p[0], \dots, p[D-1]\}$ and do a single $\mathcal{O}(n)$ pass over the string.

#### Pseudo Code
```text
ALGORITHM BinarySearchMinimumDays(s, p):
    low = 0, high = n, ans = n
    WHILE low <= high:
        mid = (low + high) / 2
        IF checkValidAfterDays(s, p, mid):
            ans = mid
            high = mid - 1
        ELSE:
            low = mid + 1
    RETURN ans
```

#### Complexity Analysis
- **Time Complexity:** $\mathcal{O}(n \log n)$ — $\mathcal{O}(\log n)$ binary search steps, each performing an $\mathcal{O}(n)$ scan.
- **Auxiliary Space:** $\mathcal{O}(n)$ to mark deleted indices.
- **Verdict:** Passes within time limit, but unnecessarily introduces $\mathcal{O}(n \log n)$ overhead.

---

### Approach 3: Incremental State Tracking (Optimal / Pro)

#### Concept
Notice that replacing a character at index `p[day]` only alters conflicts with its immediate left and right neighbors. By maintaining a single counter `equalPairs`:
1. We count initial conflicts once in $\mathcal{O}(n)$ upfront.
2. On each day, we inspect only the left neighbor `idx - 1` and right neighbor `idx + 1` in $\mathcal{O}(1)$ time.
3. If `s[idx - 1] == s[idx]`, decrement `equalPairs`.
4. If `s[idx + 1] == s[idx]`, decrement `equalPairs`.
5. Update `s[idx] = '?'`.
6. The instant `equalPairs == 0`, return `day + 1`.

#### Pseudo Code
```text
ALGORITHM OptimalMinimumDays(s, p):
    n = length(s)
    equalPairs = 0
    FOR i FROM 0 TO n - 2:
        IF s[i] == s[i + 1]:
            equalPairs = equalPairs + 1
            
    IF equalPairs == 0:
        RETURN 0
        
    FOR day FROM 0 TO n - 1:
        idx = p[day]
        IF idx > 0 AND s[idx - 1] == s[idx]:
            equalPairs = equalPairs - 1
        IF idx < n - 1 AND s[idx + 1] == s[idx]:
            equalPairs = equalPairs - 1
            
        s[idx] = '?'
        
        IF equalPairs == 0:
            RETURN day + 1
            
    RETURN n
```

#### Complexity Analysis
- **Time Complexity:** $\mathbf{\mathcal{O}(n)}$ single pass! Upfront count takes $\mathcal{O}(n)$, and the daily loop executes at most $n$ iterations with $\mathcal{O}(1)$ work per iteration.
- **Auxiliary Space:** $\mathbf{\mathcal{O}(1)}$ in C++ (in-place modification), $\mathcal{O}(n)$ in Java/Python/JS for string-to-array conversion.

---

## 8. Complexity Comparison Table

| Metric | Approach 1 (Naive Simulation) | Approach 2 (Binary Search) | Approach 3 (Incremental State Tracking - Pro) |
| :--- | :---: | :---: | :---: |
| **Initial Precomputation** | None ($\mathcal{O}(1)$) | None ($\mathcal{O}(1)$) | $\mathcal{O}(n)$ single pass |
| **Per-Day Inspection** | $\mathcal{O}(n)$ full scan | $\mathcal{O}(n)$ per check | $\mathbf{\mathcal{O}(1)}$ immediate neighbors |
| **Total Time Complexity** | $\mathcal{O}(n^2)$ | $\mathcal{O}(n \log n)$ | $\mathbf{\mathcal{O}(n)}$ |
| **Auxiliary Space** | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ | $\mathbf{\mathcal{O}(1)}$ (C++) / $\mathcal{O}(n)$ (Java/JS/Py) |
| **Operations ($n = 10^5$)** | $\approx 10^{10}$ ops | $\approx 1.7 \times 10^6$ ops | $\mathbf{\approx 2 \times 10^5}$ **ops** |
| **Verdict** | ❌ TLE | ⚠️ Accepted ($\approx 120\text{ ms}$) | 🚀 **Optimal (~15-30 ms)** |

---

## 9. Comprehensive Corner Cases Handled

1. **Initial String Already Conflict-Free ($s = \text{"abcdef"}$ or $s = \text{"abca"}$):**  
   - Handled: Initial scan finds `equalPairs == 0` and exits immediately with `0` without running the mutation loop.
2. **String of Length 1 ($n = 1$, $s = \text{"a"}$):**  
   - Handled: The initial loop `0 to n - 2` never executes. `equalPairs = 0` $\implies$ returns `0`.
3. **All Characters Identical ($s = \text{"aaaaa"}$):**  
   - Handled: Starts with $n - 1$ conflicts. Each deletion decrements by either $1$ or $2$ until all conflicts are eradicated.
4. **Boundary Mutations ($p[\text{day}] = 0$ or $p[\text{day}] = n - 1$):**  
   - Handled: Safe boundary guards `idx > 0` and `idx < n - 1` prevent out-of-bounds indexing.
5. **Neighbor Already Replaced by `'?'`:**  
   - Handled: If `s[idx - 1]` is `'?'`, `s[idx - 1] == s[idx]` evaluates to `false` because `s[idx]` is a lowercase English letter. No duplicate decrement occurs.
6. **Simultaneous Double Conflict Elimination (Middle Element in `"aaa"`):**  
   - Handled: If index $1$ is deleted from `"aaa"`, both left and right checks evaluate to true, decrementing `equalPairs` by $2$ in a single day.

---

## 10. Complete Multi-Language Implementations

Each implementation below supports both GFG driver signatures: `(s, p)` and `(n, s, p)`.

### Java (21)

```java
class Solution {
    /**
     * Calculates the minimum number of days to eliminate all adjacent equal characters.
     * 
     * @param s The input string
     * @param p The 0-indexed permutation array of replacement indices
     * @return The minimum days required (0 if already valid)
     * 
     * Time Complexity:  O(n)
     * Space Complexity: O(n) for character array representation
     */
    public int getMinimumDays(String s, int[] p) {
        int n = s.length();
        char[] arr = s.toCharArray();
        int equalPairs = 0;

        // Step 1: Count initial adjacent identical pairs
        for (int i = 0; i < n - 1; i++) {
            if (arr[i] == arr[i + 1]) {
                equalPairs++;
            }
        }

        // Base case: already valid from day 0
        if (equalPairs == 0) {
            return 0;
        }

        // Step 2: Process each day in order
        for (int day = 0; day < n; day++) {
            int idx = p[day];

            // Check conflict with left neighbor
            if (idx > 0 && arr[idx - 1] == arr[idx]) {
                equalPairs--;
            }

            // Check conflict with right neighbor
            if (idx < n - 1 && arr[idx + 1] == arr[idx]) {
                equalPairs--;
            }

            // Replace character with '?'
            arr[idx] = '?';

            // If all adjacent conflicts are resolved
            if (equalPairs == 0) {
                return day + 1; // 1-indexed day count
            }
        }

        return n;
    }

    // Overload supporting GFG test harness signature passing length n
    public int getMinimumDays(int n, String s, int[] p) {
        return getMinimumDays(s, p);
    }
}
```

---

### Python3

```python
class Solution:
    def getMinimumDays(self, *args, **kwargs) -> int:
        """
        Calculates the minimum number of days to eliminate all adjacent equal characters.
        
        Supports both signatures:
          getMinimumDays(s, p)
          getMinimumDays(n, s, p)
          
        Time Complexity:  O(n)
        Space Complexity: O(n) for list representation of mutable string
        """
        if len(args) == 2:
            s, p = args
        elif len(args) == 3:
            _, s, p = args
        else:
            s = kwargs.get('s')
            p = kwargs.get('p')

        arr = list(s)
        n = len(arr)
        equal_pairs = 0

        # Step 1: Count initial adjacent identical pairs
        for i in range(n - 1):
            if arr[i] == arr[i + 1]:
                equal_pairs += 1

        # Base case: already valid from day 0
        if equal_pairs == 0:
            return 0

        # Step 2: Process each day in order
        for day in range(n):
            idx = p[day]

            # Check conflict with left neighbor
            if idx > 0 and arr[idx - 1] == arr[idx]:
                equal_pairs -= 1

            # Check conflict with right neighbor
            if idx < n - 1 and arr[idx + 1] == arr[idx]:
                equal_pairs -= 1

            # Replace character with '?'
            arr[idx] = '?'

            # If all conflicts are resolved
            if equal_pairs == 0:
                return day + 1

        return n
```

---

### C++ (17)

```cpp
#include <string>
#include <vector>

using namespace std;

class Solution {
  public:
    /**
     * Calculates the minimum number of days to eliminate all adjacent equal characters.
     * 
     * @param s The input string (modified in-place)
     * @param p The 0-indexed permutation array of replacement indices
     * @return The minimum days required (0 if already valid)
     * 
     * Time Complexity:  O(n)
     * Space Complexity: O(1) Auxiliary Space
     */
    int getMinimumDays(string &s, vector<int> &p) {
        int n = static_cast<int>(s.length());
        int equalPairs = 0;

        // Step 1: Count initial adjacent identical pairs
        for (int i = 0; i < n - 1; i++) {
            if (s[i] == s[i + 1]) {
                equalPairs++;
            }
        }

        // Base case: already valid from day 0
        if (equalPairs == 0) {
            return 0;
        }

        // Step 2: Process each day in order
        for (int day = 0; day < n; day++) {
            int idx = p[day];

            // Check conflict with left neighbor
            if (idx > 0 && s[idx - 1] == s[idx]) {
                equalPairs--;
            }

            // Check conflict with right neighbor
            if (idx < n - 1 && s[idx + 1] == s[idx]) {
                equalPairs--;
            }

            // Replace character with '?' in-place
            s[idx] = '?';

            // If all conflicts are resolved
            if (equalPairs == 0) {
                return day + 1;
            }
        }

        return n;
    }

    // Overload supporting GFG test harness signature passing length n
    int getMinimumDays(int n, string &s, vector<int> &p) {
        return getMinimumDays(s, p);
    }
};
```

---

### C#

```csharp
public class Solution {
    /// <summary>
    /// Calculates the minimum number of days to eliminate all adjacent equal characters.
    /// </summary>
    /// <param name="s">The input string</param>
    /// <param name="p">The 0-indexed permutation array of replacement indices</param>
    /// <returns>The minimum days required (0 if already valid)</returns>
    /// <remarks>
    /// Time Complexity:  O(n)
    /// Space Complexity: O(n)
    /// </remarks>
    public int getMinimumDays(string s, int[] p) {
        char[] arr = s.ToCharArray();
        int n = arr.Length;
        int equalPairs = 0;

        // Step 1: Count initial adjacent identical pairs
        for (int i = 0; i < n - 1; i++) {
            if (arr[i] == arr[i + 1]) {
                equalPairs++;
            }
        }

        // Base case: already valid from day 0
        if (equalPairs == 0) {
            return 0;
        }

        // Step 2: Process each day in order
        for (int day = 0; day < n; day++) {
            int idx = p[day];

            // Check conflict with left neighbor
            if (idx > 0 && arr[idx - 1] == arr[idx]) {
                equalPairs--;
            }

            // Check conflict with right neighbor
            if (idx < n - 1 && arr[idx + 1] == arr[idx]) {
                equalPairs--;
            }

            // Replace character with '?'
            arr[idx] = '?';

            // If all conflicts are resolved
            if (equalPairs == 0) {
                return day + 1;
            }
        }

        return n;
    }

    // Overload supporting GFG test harness signature passing length n
    public int GetMinimumDays(int n, string s, int[] p) {
        return getMinimumDays(s, p);
    }
}
```

---

### Javascript (Node v22)

```javascript
/**
 * @param {string} s
 * @param {number[]} p
 * @return {number}
 */
class Solution {
    /**
     * Calculates the minimum number of days to eliminate all adjacent equal characters.
     * 
     * Time Complexity:  O(n)
     * Space Complexity: O(n)
     */
    getMinimumDays(...args) {
        let s, p;
        if (args.length === 2) {
            [s, p] = args;
        } else if (args.length === 3) {
            [, s, p] = args;
        } else {
            s = args[0]?.s;
            p = args[0]?.p;
        }

        const arr = typeof s === 'string' ? s.split('') : s;
        const n = arr.length;
        let equalPairs = 0;

        // Step 1: Count initial adjacent identical pairs
        for (let i = 0; i < n - 1; i++) {
            if (arr[i] === arr[i + 1]) {
                equalPairs++;
            }
        }

        // Base case: already valid from day 0
        if (equalPairs === 0) {
            return 0;
        }

        // Step 2: Process each day in order
        for (let day = 0; day < n; day++) {
            const idx = p[day];

            // Check conflict with left neighbor
            if (idx > 0 && arr[idx - 1] === arr[idx]) {
                equalPairs--;
            }

            // Check conflict with right neighbor
            if (idx < n - 1 && arr[idx + 1] === arr[idx]) {
                equalPairs--;
            }

            // Replace character with '?'
            arr[idx] = '?';

            // If all conflicts are resolved
            if (equalPairs === 0) {
                return day + 1;
            }
        }

        return n;
    }
}
```

---

## 11. Key Takeaways & Interview Preparation Tips

1. **Delta Tracking vs. Full Re-computation:**  
   In competitive programming and technical interviews, whenever an operation modifies a small localized part of a sequence, **always avoid full re-scans**. Instead of re-verifying the whole string in $\mathcal{O}(n)$, ask: *"Which invariants or relations are impacted by mutating index $k$?"* Here, only the 2 adjacent edges can change state, transforming an $\mathcal{O}(n^2)$ algorithm into an optimal $\mathcal{O}(n)$ algorithm.

2. **The "Conflict Counter" Pattern:**  
   Tracking the count of violating constraints (e.g. `equalPairs`) reduces the termination check from an $\mathcal{O}(n)$ predicate to an $\mathcal{O}(1)$ integer comparison (`equalPairs == 0`). This pattern frequently appears in sliding window problems (e.g., LeetCode 76: Minimum Window Substring) and graph connectivity maintenance.

3. **In-Place Mutation Space Advantage:**  
   In languages with mutable strings like C++, modifying `s[idx] = '?'` avoids allocating any extra buffers, yielding true **$\mathcal{O}(1)$ Auxiliary Space**. In languages where strings are immutable (Java, Python, C#, JavaScript), converting the string to an array once upfront takes $\mathcal{O}(n)$ space but avoids costly substring allocations inside the loop.

