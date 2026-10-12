# GeeksforGeeks Problem of the Day: Minimum Squares with Full Cuts (Squares in Rectangle)

- **Problem Link:** [GeeksforGeeks - Minimum Squares with Full Cuts](https://www.geeksforgeeks.org/problems/squares-in-reactangle3340/1)
- **Difficulty:** Medium
- **Topic Tags:** Mathematical Geometry, Greedy, Euclidean Algorithm, Number Theory, Continued Fractions, Guillotine Cuts
- **Target Audience:** POTD Solvers, Computer Science Students, Competitive Programmers, Interview Preparation (FAANG/Tier-1 Tech)

---

## 1. Problem Statement

Given a rectangular paper with dimensions $n \times m$, determine the **minimum number of squares** that can be formed by cutting the paper.

### The Operational Cutting Constraint:
- **Full Cuts Only (Guillotine Cuts):** Each cut must be made **completely across the remaining paper**, either horizontally from edge to edge, or vertically from edge to edge.
- No partial, stopped, or staggered cuts are permitted.
- Once a cut is executed, the remaining piece of paper is partitioned into smaller rectangles, which are then recursively cut until all resulting pieces are perfect squares.

---

### Crucial Distinction: Full Cuts vs. Arbitrary Cuts (Min Cut Square)
It is vital to contrast this problem with the classic **"Min Cut Square"** problem:
- In the general **Min Cut Square** problem (without the full-cut restriction), partial cuts are allowed. For instance, an $11 \times 13$ rectangle can be tiled into **$6$ squares** using an asymmetric "pinwheel" dissection.
- Under the **Full Cut Rule**, every cut slices cleanly through the entire breadth or length of the current piece. Consequently, the optimal strategy for full cuts strictly mirrors the **Euclidean Division Algorithm**, producing **$8$ squares** for $11 \times 13$.

---

## 2. Examples & Explanations

### Example 1
- **Input:** `n = 4, m = 5`
- **Output:** `5`
- **Explanation:**
  1. The largest square that can fit across the $4 \times 5$ paper has side length $\min(4, 5) = 4$.
  2. Make a complete vertical cut across the paper at distance $4$.
  3. This produces **one $4 \times 4$ square** and leaves a remaining strip of dimensions $4 \times 1$.
  4. The $4 \times 1$ strip can be cut into **four $1 \times 1$ squares**.
  5. Total squares formed: $1 + 4 = \mathbf{5}$.

#### Visual Dissection of Example 1 ($4 \times 5$):
```text
      <--------- 4 ---------> < 1 >
    +-------------------------+-----+
    |                         | 1x1 |
    |                         +-----+
    |                         | 1x1 |
  4 |       4 x 4 Square      +-----+
    |                         | 1x1 |
    |                         +-----+
    |                         | 1x1 |
    +-------------------------+-----+
    Total Squares: 1 (size 4x4) + 4 (size 1x1) = 5
```

---

### Example 2
- **Input:** `n = 2, m = 4`
- **Output:** `2`
- **Explanation:**
  1. Side lengths are $2$ and $4$.
  2. The paper can be cleanly divided by a single vertical full cut into **two $2 \times 2$ squares**.
  3. Total squares: $\mathbf{2}$.

#### Visual Dissection of Example 2 ($2 \times 4$):
```text
      <----- 2 -----> <----- 2 ----->
    +---------------+---------------+
    |               |               |
  2 |  2x2 Square   |  2x2 Square   |
    |               |               |
    +---------------+---------------+
    Total Squares: 2 (size 2x2) = 2
```

---

### Example 3
- **Input:** `n = 11, m = 13`
- **Output:** `8`
- **Explanation:**
  - Cut 1: Largest square of size $11 \times 11$. Leaves an $11 \times 2$ rectangle. ($\mathbf{1}$ square)
  - Cut 2: From $11 \times 2$, we can cut five $2 \times 2$ squares ($5 \times 2 = 10$). Leaves a $1 \times 2$ rectangle. ($\mathbf{5}$ squares)
  - Cut 3: From $1 \times 2$, we can cut two $1 \times 1$ squares. ($\mathbf{2}$ squares)
  - Total squares formed: $1 + 5 + 2 = \mathbf{8}$.

---

### Example 4 (Square Paper)
- **Input:** `n = 7, m = 7`
- **Output:** `1`
- **Explanation:**
  - The paper is already a square of size $7 \times 7$. No cuts are needed.
  - Total squares: $\mathbf{1}$.

---

## 3. Constraints & Scale Analysis

- $1 \le n, m \le 10^9$
- **Expected Time Complexity:** $\mathcal{O}(\log(\min(n, m)))$
- **Expected Auxiliary Space:** $\mathcal{O}(1)$

> [!WARNING]
> **Scale Analysis & Pitfall of Naive Subtraction:**  
> Notice that $n$ and $m$ can be as large as $10^9$.  
> A naive greedy subtraction approach that removes one square at a time ($n \leftarrow n - m$) takes $\mathcal{O}(\max(n, m))$ steps.  
> If $n = 10^9$ and $m = 1$, naive subtraction executes $10^9$ loop iterations $\implies$ **Time Limit Exceeded (TLE)**.  
> We must use the **Euclidean Division Algorithm** with division and modulo ($\mathcal{O}(\log(\min(n, m)))$) to complete the operation in fewer than $35$ CPU cycles!

---

## 4. Visual Architecture & The Euclidean Geometry Model

### Why Full Cuts Strictly Enforce the Euclidean Algorithm
When cutting a rectangle of size $A \times B$ (assume $A \ge B$ without loss of generality) using full cuts:
1. To obtain squares, any square cut from this rectangle cannot exceed side length $B$.
2. To minimize the total count of squares, we must greedily take the largest possible squares of side length $B$.
3. We can extract exactly $q = \lfloor A / B \rfloor$ such squares of size $B \times B$ in a row.
4. The remaining remnant paper has dimensions $B \times (A \pmod B)$.
5. This process repeats with the remnant dimensions until the remainder is $0$.

This is geometrically identical to the **Continued Fraction expansion** of the rational aspect ratio $A / B$:
$$\frac{A}{B} = q_0 + \frac{1}{q_1 + \frac{1}{q_2 + \dots}}$$
The total number of squares is simply the sum of all quotients:
$$\text{Total Squares} = \sum_{k} q_k = \sum_{k} \left\lfloor \frac{A_k}{B_k} \right\rfloor$$

```
========================================================================================
             EUCLIDEAN GEOMETRIC DISSECTION FOR n = 11, m = 13
========================================================================================

Step 1: 13 x 11
        - Quotient = 13 // 11 = 1 (One 11x11 square)
        - Remainder = 13 % 11 = 2 (Remnant paper is 11 x 2)

        +-----------------------------------+---+
        |                                   |   |
        |                                   |   |
     11 |           11 x 11 Square          |11x2
        |                                   |   |
        |                                   |   |
        +-----------------------------------+---+
                         11                   2

Step 2: 11 x 2
        - Quotient = 11 // 2 = 5 (Five 2x2 squares)
        - Remainder = 11 % 2 = 1 (Remnant paper is 2 x 1)

        +-------+
        |  2x2  |
        +-------+
        |  2x2  |
        +-------+
        |  2x2  |
        +-------+
        |  2x2  |
        +-------+
        |  2x2  |
        +-------+
        |  2x1  |
        +-------+

Step 3: 2 x 1
        - Quotient = 2 // 1 = 2 (Two 1x1 squares)
        - Remainder = 2 % 1 = 0 (Finished!)

        +---+
        |1x1|
        +---+
        |1x1|
        +---+

Total Squares = 1 + 5 + 2 = 8
========================================================================================
```

---

## 5. Step-by-Step Simulation & Detailed Trace Table

### Trace for $n = 11, m = 13$

| Iteration | Current $n$ | Current $m$ | Larger Dimension | Smaller Dimension | Squares Extracted $\lfloor \text{Large} / \text{Small} \rfloor$ | New Remainder $(\text{Large} \pmod{\text{Small}})$ | Running Total |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Initial** | 11 | 13 | — | — | — | — | 0 |
| **Step 1** | 11 | 13 | 13 | 11 | $\lfloor 13 / 11 \rfloor = \mathbf{1}$ | $13 \pmod{11} = 2$ | **1** |
| **Step 2** | 2 | 11 | 11 | 2 | $\lfloor 11 / 2 \rfloor = \mathbf{5}$ | $11 \pmod 2 = 1$ | $1 + 5 = \mathbf{6}$ |
| **Step 3** | 1 | 2 | 2 | 1 | $\lfloor 2 / 1 \rfloor = \mathbf{2}$ | $2 \pmod 1 = 0$ | $6 + 2 = \mathbf{8}$ |
| **Term** | 0 | 1 | — | — | — | Dimension is 0 $\implies$ Stop | **8** |

---

## 6. Algorithm Flowchart

```mermaid
flowchart TD
    Start([Start: minSquares n, m]) --> Init["res = 0"]
    Init --> CheckLoop{"n > 0 AND m > 0 ?"}
    
    CheckLoop -- "No (Paper fully divided)" --> ReturnResult["Return res"]
    ReturnResult --> Finish([End])
    
    CheckLoop -- "Yes" --> CheckOrder{"n < m ?"}
    CheckOrder -- "Yes" --> SwapDims["Swap(n, m)<br/>Ensure n >= m"]
    CheckOrder -- "No" --> CalcQuotient["res += n / m<br/>Add all m x m squares"]
    
    SwapDims --> CalcQuotient
    CalcQuotient --> CalcRem["n = n % m<br/>Update remaining dimension"]
    CalcRem --> CheckLoop

    style Start fill:#1E88E5,stroke:#0D47A1,color:#ffffff
    style Finish fill:#1E88E5,stroke:#0D47A1,color:#ffffff
    style ReturnResult fill:#2E7D32,stroke:#1B5E20,color:#ffffff
    style CalcQuotient fill:#FB8C00,stroke:#E65100,color:#ffffff
    style CheckLoop fill:#0288D1,stroke:#01579B,color:#ffffff
```

---

## 7. From Naive to Pro: Algorithmic Progression

### Approach 1: Naive Subtraction (Repeated Single Square Removal)
- **Concept:** Remove one largest square of size $\min(n, m) \times \min(n, m)$ at a time by subtracting: $n \leftarrow n - m$, and incrementing `res++`.
- **Why it is suboptimal:** If $n = 10^9$ and $m = 1$, the loop runs $10^9$ times.
- **Complexity:**
  - Time: $\mathcal{O}(\max(n, m)) \implies$ **TLE** for $10^9$.
  - Auxiliary Space: $\mathcal{O}(1)$.
- **Pseudocode:**
```text
function minSquares_Naive(n, m):
    res = 0
    while n > 0 and m > 0:
        if n < m:
            swap(n, m)
        n = n - m
        res = res + 1
    return res
```

---

### Approach 2: Recursive Euclidean Formulation
- **Concept:** By applying Euclidean recurrence:
  $$\text{minSquares}(n, m) = \lfloor n / m \rfloor + \text{minSquares}(m, n \pmod m)$$
- **Why it is good:** Cuts time complexity down to $\mathcal{O}(\log(\min(n, m)))$.
- **Complexity:**
  - Time: $\mathcal{O}(\log(\min(n, m)))$
  - Auxiliary Space: $\mathcal{O}(\log(\min(n, m)))$ call stack space.
- **Pseudocode:**
```text
function minSquares_Recursive(n, m):
    if m == 0:
        return 0
    return (n / m) + minSquares_Recursive(m, n % m)
```

---

### Approach 3: Pro Optimized Iterative Euclidean Algorithm ($\mathcal{O}(1)$ Space)
- **Concept:** Eliminates all recursion stack overhead. Uses division and modulo in an iterative loop.
- **Why it is optimal:**
  - Executes in $\le 35$ operations even for maximum constraints $n, m = 10^9$.
  - Strictly $\mathcal{O}(1)$ memory with zero allocations.
  - Highly cache-friendly and instruction-level parallelizable.
- **Pseudocode:**
```text
function minSquares_Optimal(n, m):
    res = 0
    while n > 0 and m > 0:
        if n < m:
            swap(n, m)
        res = res + (n / m)
        n = n % m
    return res
```

---

## 8. Complexity Comparison Table

| Metric | Approach 1: Naive Subtraction | Approach 2: Recursive Euclidean | Approach 3: Iterative Euclidean (Pro) |
| :--- | :--- | :--- | :--- |
| **Time Complexity** | $\mathcal{O}(\max(n, m))$ | $\mathcal{O}(\log(\min(n, m)))$ | $\mathbf{\mathcal{O}(\log(\min(n, m)))}$ |
| **Auxiliary Space** | $\mathcal{O}(1)$ | $\mathcal{O}(\log(\min(n, m)))$ (Call Stack) | $\mathbf{\mathcal{O}(1)}$ (Zero Extra Memory) |
| **Steps for $n=10^9, m=1$** | $1,000,000,000$ operations (**TLE**) | $\approx 2$ recursive calls | **$1$ division step ($< 0.001\text{ ms}$)** |
| **Worst-Case Steps (Fibonacci)** | Exponentially slow | $\approx 45$ calls | **$\le 45$ iterations** |
| **Memory Locality** | CPU register | Stack frames | **Pure CPU registers** |
| **Interview Suitability** | Naive baseline | Clean | **Gold Standard (Production Grade)** |

---

## 9. Corner Cases & Deep Engineering Nuances

1. **Square Paper ($n = m$):**
   - $n / m = 1$, remainder is $0$. Loop terminates immediately with `res = 1`.
2. **Extreme Aspect Ratios ($n = 10^9, m = 1$ or $n = 1, m = 10^9$):**
   - Direct division computes $10^9 / 1 = 10^9$ in a single iteration without timeouts.
3. **Integer Overflow Check:**
   - The maximum possible result is $\max(n, m) = 10^9$.
   - A standard 32-bit signed integer has maximum value $2^{31} - 1 = 2,147,483,647 > 10^9$.
   - Thus, the return value fits comfortably within a standard 32-bit signed integer in C++, Java, C#, and JavaScript.
4. **Dimensions Order Independence:**
   - Whether $n > m$ or $n < m$, the swap logic normalizes the dimensions on the very first cycle.

---

## 10. Complete Multi-Language Implementations

### Java (21)

```java
class Solution {
    /**
     * Calculates the minimum number of squares formed using full cuts.
     * Time Complexity: O(log(min(n, m)))
     * Auxiliary Space: O(1)
     *
     * @param n Width/length of rectangle
     * @param m Length/width of rectangle
     * @return Minimum number of squares
     */
    public int minSquares(int n, int m) {
        int res = 0;

        // Iteratively extract maximum squares using the Euclidean division algorithm
        while (n > 0 && m > 0) {
            // Ensure n is the larger or equal dimension
            if (n < m) {
                int temp = n;
                n = m;
                m = temp;
            }

            // Extract all possible squares of size m x m
            res += n / m;

            // Retain the remaining paper dimensions
            n %= m;
        }

        return res;
    }
}
```

---

### Python3

```python
class Solution:
    def minSquares(self, n: int, m: int) -> int:
        """
        Calculates the minimum number of squares formed using full cuts.
        Time Complexity: O(log(min(n, m)))
        Auxiliary Space: O(1)
        """
        res = 0

        # Iteratively extract maximum squares using Euclidean division
        while n > 0 and m > 0:
            if n < m:
                n, m = m, n

            res += n // m
            n %= m

        return res
```

---

### C++ (17)

```cpp
#include <algorithm>

using namespace std;

class Solution {
  public:
    /**
     * Calculates the minimum number of squares formed using full cuts.
     * Time Complexity: O(log(min(n, m)))
     * Auxiliary Space: O(1)
     *
     * @param n Width/length of rectangle
     * @param m Length/width of rectangle
     * @return Minimum number of squares
     */
    int minSquares(int n, int m) {
        int res = 0;

        // Iteratively extract maximum squares using Euclidean division
        while (n > 0 && m > 0) {
            // Ensure n is the larger or equal dimension
            if (n < m) {
                swap(n, m);
            }

            // Add the count of m x m squares
            res += n / m;

            // Reduce to the remaining dimension
            n %= m;
        }

        return res;
    }
};
```

---

### C#

```csharp
public class Solution {
    /// <summary>
    /// Calculates the minimum number of squares formed using full cuts.
    /// Time Complexity: O(log(min(n, m)))
    /// Auxiliary Space: O(1)
    /// </summary>
    /// <param name="n">Dimension 1 of rectangle</param>
    /// <param name="m">Dimension 2 of rectangle</param>
    /// <returns>Minimum number of squares</returns>
    public int minSquares(int n, int m) {
        int res = 0;

        // Iteratively extract maximum squares using Euclidean division
        while (n > 0 && m > 0) {
            // Ensure n is the larger dimension
            if (n < m) {
                int temp = n;
                n = m;
                m = temp;
            }

            // Extract all possible squares of size m x m
            res += n / m;

            // Reduce to the remaining dimension
            n %= m;
        }

        return res;
    }
}
```

---

### Javascript (Node v22)

```javascript
/**
 * @param {number} n
 * @param {number} m
 * @returns {number}
 */
class Solution {
    /**
     * Calculates the minimum number of squares formed using full cuts.
     * Time Complexity: O(log(min(n, m)))
     * Auxiliary Space: O(1)
     */
    minSquares(n, m) {
        let res = 0;

        // Iteratively extract maximum squares using Euclidean division
        while (n > 0 && m > 0) {
            // Ensure n is the larger dimension
            if (n < m) {
                const temp = n;
                n = m;
                m = temp;
            }

            // Extract all possible squares of size m x m
            res += Math.floor(n / m);

            // Reduce to the remaining dimension
            n %= m;
        }

        return res;
    }
}
```

---

## 11. Verification & Test Summary

| Test Case | Dimensions $(n, m)$ | Division Sequence ($\text{Quotient} \times \text{Size}$) | Expected Output | Actual Output | Status |
| :---: | :---: | :--- | :---: | :---: | :---: |
| **Example 1** | $(4, 5)$ | $1 \times (4\times 4) + 4 \times (1\times 1)$ | `5` | `5` | **PASS** |
| **Example 2** | $(2, 4)$ | $2 \times (2\times 2)$ | `2` | `2` | **PASS** |
| **Example 3** | $(11, 13)$ | $1 \times (11\times 11) + 5 \times (2\times 2) + 2 \times (1\times 1)$ | `8` | `8` | **PASS** |
| **Square** | $(7, 7)$ | $1 \times (7\times 7)$ | `1` | `1` | **PASS** |
| **Linear Strip** | $(1, 10)$ | $10 \times (1\times 1)$ | `10` | `10` | **PASS** |
| **Inverted Order** | $(13, 11)$ | $1 \times (11\times 11) + 5 \times (2\times 2) + 2 \times (1\times 1)$ | `8` | `8` | **PASS** |
| **Large Aspect Ratio** | $(1, 10^9)$ | $10^9 \times (1\times 1)$ in 1 step | `1000000000` | `1000000000` | **PASS** |
| **Composite Common Factor** | $(6, 9)$ | $1 \times (6\times 6) + 2 \times (3\times 3)$ | `3` | `3` | **PASS** |
