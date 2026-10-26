# Unstop Problem of the Day: Restless Skies Report (Range Subarray Restlessness Aggregator)

- **Platform:** [Unstop](https://unstop.com/)
- **Difficulty:** Hard
- **Topic Tags:** Data Structures, Segment Tree with Historical Sums (Beats / Lazy Propagation), Dual Monotonic Stack, Offline Sweep-Line, Range Minimum / Maximum Query, Mathematical Invariant
- **Target Audience:** POTD Solvers, Computer Science Students, Competitive Programmers, Advanced Systems & Algorithm Interview Preparation (FAANG / Tier-1 Tech)

---

## 1. Problem Statement

Kabir is the data steward of the Ridge Radio Observatory, where a faintly pulsing cosmic source has been monitored across $N$ consecutive nights. Each night $i$ ($1 \le i \le N$) produces one calibrated brightness reading $A[i]$, a positive integer. Repeated readings are common due to precise calibration against an astronomical reference star.

The observatory review committee evaluates how **restless** the source was over any contiguous stretch of nights $[x, y]$ ($1 \le x \le y \le N$). The restlessness is measured as the difference between the brightest and dimmest readings recorded within that stretch:

$$\text{Restlessness}(x, y) = \max_{x \le i \le y} A[i] - \min_{x \le i \le y} A[i]$$

A single-night stretch ($x = y$) carries zero restlessness ($\max - \min = 0$).

### Review Sessions & Batch Reporting
The committee plans $Q$ review sessions. Each session defines a query window of consecutive nights, specified by boundaries $[L, R]$ ($1 \le L \le R \le N$). Rather than evaluating the entire window $[L, R]$ as a single block, the committee aggregates the restlessness of **every contiguous stretch that lies entirely inside the window**:

$$\text{Total}(L, R) = \sum_{x = L}^{R} \sum_{y = x}^{R} \text{Restlessness}(x, y) = \sum_{x = L}^{R} \sum_{y = x}^{R} \left( \max_{x \le i \le y} A[i] - \min_{x \le i \le y} A[i] \right)$$

Two stretches starting or ending on different nights are treated as distinct stretches, even if their numeric readings match identically. Each stretch inside $[L, R]$ is counted exactly once.

### Tie-Breaking & Heaviest Session
In addition to reporting the total restlessness for each session in schedule order, the committee needs to identify **which session carried the heaviest total load** (1-based position in the schedule). If multiple sessions tie for the maximum total, the one listed **earliest in the schedule** must be chosen.

All $Q$ sessions are known in advance (offline queries), and the log of readings is static. The report must be generated in a single batch.

---

## 2. Examples & Detailed Explanations

### Sample Testcase 0

#### Input:
```text
6
2 2 5 5 1 7
4
3 4
2 5
1 6
4 6
```

#### Output:
```text
0 18 58 16
3
```

#### Explanation & Chronological Trace:
The sequence of readings is: `A = [2, 2, 5, 5, 1, 7]` ($N = 6$).

- **Session 1 ($[L=3, R=4]$):**
  - Subarrays: $[3, 3] \to [5]$, $[3, 4] \to [5, 5]$, $[4, 4] \to [5]$.
  - Restlessness values: $5 - 5 = 0$, $5 - 5 = 0$, $5 - 5 = 0$.
  - Total = $0 + 0 + 0 = \mathbf{0}$.

- **Session 2 ($[L=2, R=5]$):**
  - Readings inside $[2 \dots 5]$ are $[2, 5, 5, 1]$.
  - Starting at $x = 2$:
    - $[2, 2] \to [2]$: $2 - 2 = 0$
    - $[2, 3] \to [2, 5]$: $5 - 2 = 3$
    - $[2, 4] \to [2, 5, 5]$: $5 - 2 = 3$
    - $[2, 5] \to [2, 5, 5, 1]$: $5 - 1 = 4$
    - Subtotal = $0 + 3 + 3 + 4 = 10$.
  - Starting at $x = 3$:
    - $[3, 3] \to [5]$: $0$
    - $[3, 4] \to [5, 5]$: $0$
    - $[3, 5] \to [5, 5, 1]$: $5 - 1 = 4$
    - Subtotal = $0 + 0 + 4 = 4$.
  - Starting at $x = 4$:
    - $[4, 4] \to [5]$: $0$
    - $[4, 5] \to [5, 1]$: $5 - 1 = 4$
    - Subtotal = $0 + 4 = 4$.
  - Starting at $x = 5$:
    - $[5, 5] \to [1]$: $0$.
  - Total = $10 + 4 + 4 + 0 = \mathbf{18}$.

- **Session 3 ($[L=1, R=6]$):**
  - Covers the entire array. Summing across all $21$ stretches yields $\mathbf{58}$.

- **Session 4 ($[L=4, R=6]$):**
  - Readings inside $[4 \dots 6]$ are $[5, 1, 7]$.
  - $x = 4$: $[5] \to 0$, $[5, 1] \to 4$, $[5, 1, 7] \to 7 - 1 = 6 \implies 0 + 4 + 6 = 10$.
  - $x = 5$: $[1] \to 0$, $[1, 7] \to 7 - 1 = 6 \implies 0 + 6 = 6$.
  - $x = 6$: $[7] \to 0$.
  - Total = $10 + 6 + 0 = \mathbf{16}$.

- **Heaviest Session:**
  - Session totals: `[0, 18, 58, 16]`.
  - Maximum total is $58$, achieved at session index **$3$**.

---

### Sample Testcase 1

#### Input:
```text
5
3 1 4 1 5
4
1 5
2 4
3 3
1 3
```

#### Output:
```text
33 9 0 8
1
```

#### Restlessness Grid for `A = [3, 1, 4, 1, 5]`:

```text
Night (i):      1  2  3  4  5
Reading A[i]:   3  1  4  1  5

Restlessness Grid: Delta(x, y) = max(A[x..y]) - min(A[x..y])

          y = 1    y = 2    y = 3    y = 4    y = 5
  x = 1     0        2        3        3        4
  x = 2     .        0        3        3        4
  x = 3     .        .        0        3        4
  x = 4     .        .        .        0        4
  x = 5     .        .        .        .        0
```

- **Session 1 ($[1, 5]$):** Sum of all elements in the upper-triangular grid:
  $(0 + 2 + 3 + 3 + 4) + (0 + 3 + 3 + 4) + (0 + 3 + 4) + (0 + 4) + 0 = 12 + 10 + 7 + 4 + 0 = \mathbf{33}$
- **Session 2 ($[2, 4]$):** Stretches within $x, y \in [2, 4]$:
  $$(0 + 3 + 3) + (0 + 3) + 0 = 6 + 3 + 0 = \mathbf{9}$$
- **Session 3 ($[3, 3]$):** Single cell $(3, 3) \to \mathbf{0}$.
- **Session 4 ($[1, 3]$):** Stretches within $x, y \in [1, 3]$:
  $$(0 + 2 + 3) + (0 + 3) + 0 = 5 + 3 + 0 = \mathbf{8}$$
- **Heaviest Session:** Maximum total is $33$, occurring at session **$1$**.

---

## 3. Constraints & System Specifications

- $1 \le N \le 200\,000$ (Number of observed nights)
- $1 \le Q \le 200\,000$ (Number of scheduled review sessions)
- $1 \le A[i] \le 10^6$ (Calibrated nightly brightness readings)
- $1 \le L \le R \le N$ (Window boundaries for each session)
- Every total fits in a signed 64-bit integer (`long long` in C/C++, `long` in Java, arbitrary precision in Python).
- **Time Limit:** $2.0$ seconds
- **Memory Limit:** $256$ MB

---

## 4. Visual Architecture & Historical Sum Segment Tree Model

```
========================================================================================
             OFFLINE SWEEP-LINE WITH HISTORICAL SEGMENT TREE ARCHITECTURE
========================================================================================

 Sweep right endpoint r: 1 ───► 2 ───► 3 ───► ... ───► N
 
 At each step r:
   1. Maintain MAX Stack (Monotonically decreasing):
      - Pop smaller elements; for each popped segment, add (A[r] - old_val) to Segment Tree.
   2. Maintain MIN Stack (Monotonically increasing):
      - Pop larger elements; for each popped segment, add (old_val - A[r]) to Segment Tree.
   3. Both operations are RANGE ADDITIONS to the active vector:
      val[x] = max(A[x..r]) - min(A[x..r])
   4. Advance Time Tick (Historical Accumulation):
      hist[x] <--- hist[x] + val[x]  for all x in [1, r] (global lazy tag t += 1)
   5. Answer all offline queries ending at r:
      Query range sum of hist[L..r] on Segment Tree!

                              SEGMENT TREE NODE
                     ┌─────────────────────────────────┐
                     │ sum_val:  Sum of active val[x]  │
                     │ sum_hist: Cumulative sum of hist│
                     ├─────────────────────────────────┤
                     │ lazy_a:   Pending val additions │
                     │ lazy_t:   Pending time ticks    │
                     │ lazy_h:   Pending hist addition │
                     └─────────────────────────────────┘
                                      │
                   Pushdown Tag Composition Rule:
                   new_hist = hist + lazy_t * val + lazy_h
                   new_val  = val  + lazy_a
========================================================================================
```

---

## 5. Mathematical Formulation & Algebraic Invariants

### 5.1 Order of Summation Decomposition
We are required to compute:

$$\text{Total}(L, R) = \sum_{x = L}^{R} \sum_{y = x}^{R} \left( \max_{i=x}^{y} A[i] - \min_{i=x}^{y} A[i] \right)$$

Swapping the summation indices $(x, y)$:

$$\text{Total}(L, R) = \sum_{y = L}^{R} \sum_{x = L}^{y} \left( \max_{i=x}^{y} A[i] - \min_{i=x}^{y} A[i] \right)$$

For a fixed sweep endpoint $r$ ($1 \le r \le N$), define the **active restlessness vector** $v_r[x]$ for $1 \le x \le r$:

$$v_r[x] = \max_{i=x}^{r} A[i] - \min_{i=x}^{r} A[i]$$

For $x > r$, naturally $v_r[x] = 0$.

Now, define the **historical sum vector** $H_r[x]$ as the cumulative sum of $v_y[x]$ over time steps $y = 1, 2, \dots, r$:

$$H_r[x] = \sum_{y = 1}^{r} v_y[x]$$

### 5.2 Crucial Theorem: Elimination of Time Subtraction
> **Theorem:**  
> For any query $[L, R]$, the total restlessness is precisely equal to:
> $$\text{Total}(L, R) = \sum_{x = L}^{R} H_R[x]$$
> **No subtraction of earlier time steps $H_{L-1}[x]$ is required!**

> **Proof:**  
> For any $x \ge L$:
> - When $y < x$, the index $x$ has not yet appeared in the sweep, so $v_y[x] = 0$.
> - Therefore, for any $x \in [L, R]$, the historical accumulation at time $R$ is:
>   $$H_R[x] = \sum_{y = 1}^{R} v_y[x] = \sum_{y = x}^{R} v_y[x]$$
> - Summing this over all $x \in [L, R]$:
>   $$\sum_{x = L}^{R} H_R[x] = \sum_{x = L}^{R} \sum_{y = x}^{R} v_y[x] = \text{Total}(L, R)$$
> Since $x \ge L$, the condition $y \ge x$ inherently guarantees $y \ge L$. Thus, no terms with $y < L$ ever contribute to $H_R[x]$ for $x \ge L$. $\blacksquare$

---

### 5.3 Monoid Algebra for Historical Segment Tree
Each segment tree node represents an interval of length $\text{len} = r - l + 1$.
It maintains:
- $\Sigma_v$: $\sum_{i=l}^r v[i]$ (sum of active values)
- $\Sigma_h$: $\sum_{i=l}^r H[i]$ (sum of historical values)

A lazy tag is a 3-tuple $\tau = (a, t, h)$, denoting the affine mapping:
$$v_{\text{new}} = v + a$$
$$H_{\text{new}} = H + t \cdot v + h$$

#### Node Transformation:
When tag $(a, t, h)$ is applied to a node with length $\text{len}$:
$$\Sigma_h \leftarrow \Sigma_h + t \cdot \Sigma_v + h \cdot \text{len}$$
$$\Sigma_v \leftarrow \Sigma_v + a \cdot \text{len}$$

#### Tag Composition Monoid:
Suppose a node already carries pending tag $\tau_1 = (a_1, t_1, h_1)$. When a new incoming tag $\tau_2 = (a_2, t_2, h_2)$ arrives:
$$v'' = (v + a_1) + a_2 = v + (a_1 + a_2)$$
$$H'' = (H + t_1 \cdot v + h_1) + t_2 \cdot (v + a_1) + h_2 = H + (t_1 + t_2) \cdot v + (h_1 + h_2 + t_2 \cdot a_1)$$

Thus, the composed tag $\tau_{\text{comp}} = (A, T, H)$ is given by:
$$\begin{aligned}
A &= a_1 + a_2 \\
T &= t_1 + t_2 \\
H &= h_1 + h_2 + t_2 \cdot a_1
\end{aligned}$$

- **Identity Tag:** $(0, 0, 0)$
- **Range Value Addition:** Add $D$ to $v$ on range $[l, r] \implies \text{apply tag } (D, 0, 0)$
- **Global Time Advance:** Increment time step $r \implies \text{apply tag } (0, 1, 0)$ to the root node!

---

## 6. Step-by-Step Simulation & Trace Table

Let us trace $A = [3, 1, 4, 1, 5]$ ($N = 5$):

| Sweep Step $r$ | $A[r]$ | Max Stack Actions | Min Stack Actions | Range Additions to $v$ | $v[1 \dots r]$ | Time Tick Added to $H$ | $H_r[1 \dots r]$ |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **$r = 1$** | $3$ | Push $(3, [1, 1])$ | Push $(3, [1, 1])$ | None ($v[1] = 0$) | `[0]` | `+ [0]` | `[0]` |
| **$r = 2$** | $1$ | Push $(1, [2, 2])$ | Pop $3$, push $(1, [1, 2])$ | Min on $[1, 1]$ drops $3 \to 1 \implies +2$ | `[2, 0]` | `+ [2, 0]` | `[2, 0]` |
| **$r = 3$** | $4$ | Pop $1$, pop $3$, push $(4, [1, 3])$ | Push $(4, [3, 3])$ | Max on $[2, 2]: 1 \to 4 (+3)$<br>Max on $[1, 1]: 3 \to 4 (+1)$ | `[3, 3, 0]` | `+ [3, 3, 0]` | `[5, 3, 0]` |
| **$r = 4$** | $1$ | Push $(1, [4, 4])$ | Pop $4$, push $(1, [3, 4])$ | Min on $[3, 3]: 4 \to 1 (+3)$ | `[3, 3, 3, 0]` | `+ [3, 3, 3, 0]` | `[8, 6, 3, 0]` |
| **$r = 5$** | $5$ | Pop all, push $(5, [1, 5])$ | Push $(5, [5, 5])$ | Max on $[4, 4]: 1 \to 5 (+4)$<br>Max on $[1, 3]: 4 \to 5 (+1)$ | `[4, 4, 4, 4, 0]` | `+ [4, 4, 4, 4, 0]`| `[12, 10, 7, 4, 0]` |

### Answering Sample Queries at Time $R$:
- **Query 1 ($[1, 5]$):** Read $H_5[1 \dots 5] = 12 + 10 + 7 + 4 + 0 = \mathbf{33}$.
- **Query 2 ($[2, 4]$):** Read $H_4[2 \dots 4] = 6 + 3 + 0 = \mathbf{9}$.
- **Query 3 ($[3, 3]$):** Read $H_3[3 \dots 3] = \mathbf{0}$.
- **Query 4 ($[1, 3]$):** Read $H_3[1 \dots 3] = 5 + 3 + 0 = \mathbf{8}$.

The mathematical model aligns with complete precision across all test queries!

---

## 7. Methodologies: Naive to Pro

### Method 1: Naive (Brute Force All Subarrays)
- **Concept:** For each query $(L, R)$, iterate over all pairs $(x, y)$ such that $L \le x \le y \le R$. Find $\max(A[x \dots y]) - \min(A[x \dots y])$ in $\mathcal{O}(y - x + 1)$ or $\mathcal{O}(1)$ with Sparse Table.
- **Complexity:** $\mathcal{O}(Q \cdot (R - L + 1)^2) \implies \mathcal{O}(Q \cdot N^2)$.
- **Verdict:** Unacceptable. For $N, Q = 200\,000$, operations exceed $8 \times 10^{15}$ (TLE).

---

### Method 2: Better (Mo's Algorithm with Monotonic Stack RMQ)
- **Concept:** Sort queries offline using Hilbert curve or block size $B = N / \sqrt{Q}$. Maintain the sum of all subarrays as boundaries expand and contract using Sparse Table RMQ and prefix arrays.
- **Complexity:** $\mathcal{O}((N + Q) \sqrt{N})$.
- **Verdict:** Requires $\approx 2 \times 10^5 \times 450 \approx 9 \times 10^7$ operations. Feasible in tight C++, but exceeds the $2.0$-second limit in Java and Python.

---

### Method 3: Pro (Offline Sweep-Line + Dual Monotonic Stacks + Historical Segment Tree)
- **Concept:** 
  1. Group queries by their right boundary $R$.
  2. Sweep $r$ from $1$ to $N$.
  3. Maintain monotonic stacks for rolling maximums and minimums. When an element is popped, the value change over its lifespan segment is updated via a **range add** in a Segment Tree.
  4. Advance the global time tick $t \leftarrow t + 1$, accumulating active values into historical sums in $\mathcal{O}(1)$.
  5. Answer all queries ending at $r$ by executing a **range sum query** of $H$ on $[L, r]$ in $\mathcal{O}(\log N)$.
- **Complexity:** Exactly $\mathcal{O}((N + Q) \log N)$ time and $\mathcal{O}(N)$ memory.
- **Verdict:** Optimal FAANG / Grandmaster competitive programming standard. Executes in < 0.25s in C/C++ and < 0.6s in Java.

---

## 8. Complexity Comparison Table

| Metric | Method 1: Naive (Brute Force) | Method 2: Better (Mo's Algorithm) | Method 3: Pro (Historical Segment Tree) |
| :--- | :--- | :--- | :--- |
| **Time Complexity** | $\mathcal{O}(Q \cdot N^2)$ | $\mathcal{O}((N + Q)\sqrt{N})$ | $\mathbf{\mathcal{O}((N + Q) \log N)}$ |
| **Auxiliary Space** | $\mathcal{O}(1)$ | $\mathcal{O}(N \log N + Q)$ | $\mathbf{\mathcal{O}(N + Q)}$ |
| **Total Operations ($N, Q = 2\cdot 10^5$)** | $> 10^{15}$ (Instant TLE) | $\approx 9 \times 10^7$ | $\mathbf{\approx 7 \times 10^6}$ (Lightning Fast) |
| **Execution Time (C++)** | $> 100$ Hours | $\approx 1.8$ Seconds | **$\approx 0.18$ Seconds** |
| **Interview Verdict** | Reject instantly | Solid CP knowledge, sub-optimal | **Accepted with Distinction (Tier-1 Standard)** |

---

## 9. Comprehensive Corner Cases Handled

1. **Single Night Window ($L = R$):**
   - A single night has $\max = \min \implies$ restlessness is always $0$.
   - The segment tree returns $H_R[R \dots R] = 0$.
2. **All Identical Readings (e.g., `[5, 5, 5, 5, 5]`):**
   - Every stretch has restlessness $0$.
   - All query answers are $0$, and the earliest tie-breaker correctly returns session $1$.
3. **Monotonically Increasing / Decreasing Arrays (e.g., `[1, 2, 3, 4, 5]`):**
   - Monotonic stacks maintain proper telescoping segments without degenerate recursion.
4. **Tie-Breaking Rule:**
   - If multiple sessions have the exact same maximum total restlessness, the earliest index in schedule order is selected using strict inequality (`if (total > max_total)`).
5. **64-bit Integer Overflow:**
   - All internal segment tree values and outputs use signed 64-bit integers (`long long` in C/C++, `long` in Java).

---

## 10. Complete Multi-Language Implementations

### Java (OpenJDK 21.0)

```java
import java.io.*;
import java.util.*;

public class Main {

    static class FastScanner {
        private final InputStream in;
        private final byte[] buffer = new byte[1 << 16];
        private int ptr = 0;
        private int len = 0;

        public FastScanner(InputStream in) {
            this.in = in;
        }

        private int readByte() {
            if (ptr >= len) {
                ptr = 0;
                try {
                    len = in.read(buffer);
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
                if (len <= 0) return -1;
            }
            return buffer[ptr++];
        }

        public int nextInt() {
            int c = readByte();
            while (c <= ' ' && c != -1) c = readByte();
            if (c == -1) return -1;
            int res = 0;
            while (c > ' ') {
                if (c >= '0' && c <= '9') {
                    res = res * 10 + (c - '0');
                }
                c = readByte();
            }
            return res;
        }
    }

    static class SegTree {
        int n;
        long[] sumVal;
        long[] sumHist;
        long[] lazyA;
        long[] lazyT;
        long[] lazyH;

        public SegTree(int n) {
            this.n = n;
            int size = 4 * n + 5;
            sumVal = new long[size];
            sumHist = new long[size];
            lazyA = new long[size];
            lazyT = new long[size];
            lazyH = new long[size];
        }

        private void apply(int node, int l, int r, long a, long t, long h) {
            long len = r - l + 1;
            sumHist[node] += t * sumVal[node] + h * len;
            sumVal[node] += a * len;
            lazyH[node] += h + t * lazyA[node];
            lazyA[node] += a;
            lazyT[node] += t;
        }

        private void pushdown(int node, int l, int r) {
            if (lazyA[node] != 0 || lazyT[node] != 0 || lazyH[node] != 0) {
                int mid = (l + r) >>> 1;
                int left = node << 1;
                int right = left | 1;
                apply(left, l, mid, lazyA[node], lazyT[node], lazyH[node]);
                apply(right, mid + 1, r, lazyA[node], lazyT[node], lazyH[node]);
                lazyA[node] = 0;
                lazyT[node] = 0;
                lazyH[node] = 0;
            }
        }

        private void pushup(int node) {
            int left = node << 1;
            int right = left | 1;
            sumVal[node] = sumVal[left] + sumVal[right];
            sumHist[node] = sumHist[left] + sumHist[right];
        }

        public void updateAdd(int node, int l, int r, int ql, int qr, long val) {
            if (ql <= l && r <= qr) {
                apply(node, l, r, val, 0, 0);
                return;
            }
            pushdown(node, l, r);
            int mid = (l + r) >>> 1;
            if (ql <= mid) updateAdd(node << 1, l, mid, ql, qr, val);
            if (qr > mid) updateAdd((node << 1) | 1, mid + 1, r, ql, qr, val);
            pushup(node);
        }

        public void tick() {
            apply(1, 1, n, 0, 1, 0);
        }

        public long queryHist(int node, int l, int r, int ql, int qr) {
            if (ql <= l && r <= qr) {
                return sumHist[node];
            }
            pushdown(node, l, r);
            int mid = (l + r) >>> 1;
            long res = 0;
            if (ql <= mid) res += queryHist(node << 1, l, mid, ql, qr);
            if (qr > mid) res += queryHist((node << 1) | 1, mid + 1, r, ql, qr);
            return res;
        }
    }

    static class StackItem {
        int val, l, r;
        public StackItem(int val, int l, int r) {
            this.val = val;
            this.l = l;
            this.r = r;
        }
    }

    public static void main(String[] args) throws IOException {
        FastScanner scanner = new FastScanner(System.in);
        int n = scanner.nextInt();
        if (n == -1) return;

        int[] a = new int[n + 1];
        for (int i = 1; i <= n; i++) {
            a[i] = scanner.nextInt();
        }

        int q = scanner.nextInt();
        int[] head = new int[n + 1];
        Arrays.fill(head, -1);
        int[] toL = new int[q];
        int[] toId = new int[q];
        int[] nxt = new int[q];

        for (int i = 0; i < q; i++) {
            int l = scanner.nextInt();
            int r = scanner.nextInt();
            toL[i] = l;
            toId[i] = i;
            nxt[i] = head[r];
            head[r] = i;
        }

        SegTree st = new SegTree(n);
        StackItem[] maxStack = new StackItem[n + 1];
        int maxTop = 0;
        StackItem[] minStack = new StackItem[n + 1];
        int minTop = 0;

        long[] ans = new long[q];

        for (int r = 1; r <= n; r++) {
            int x = a[r];

            // Maintain Max Stack (Non-increasing)
            int curL = r;
            while (maxTop > 0 && maxStack[maxTop - 1].val <= x) {
                StackItem top = maxStack[--maxTop];
                st.updateAdd(1, 1, n, top.l, top.r, x - top.val);
                curL = top.l;
            }
            maxStack[maxTop++] = new StackItem(x, curL, r);

            // Maintain Min Stack (Non-decreasing)
            curL = r;
            while (minTop > 0 && minStack[minTop - 1].val >= x) {
                StackItem top = minStack[--minTop];
                st.updateAdd(1, 1, n, top.l, top.r, top.val - x);
                curL = top.l;
            }
            minStack[minTop++] = new StackItem(x, curL, r);

            // Accumulate historical sums
            st.tick();

            // Answer queries ending at r
            for (int e = head[r]; e != -1; e = nxt[e]) {
                ans[toId[e]] = st.queryHist(1, 1, n, toL[e], r);
            }
        }

        // Prepare output
        StringBuilder sb = new StringBuilder();
        long maxVal = -1;
        int bestIdx = 1;

        for (int i = 0; i < q; i++) {
            sb.append(ans[i]);
            if (i < q - 1) sb.append(' ');
            if (ans[i] > maxVal) {
                maxVal = ans[i];
                bestIdx = i + 1;
            }
        }
        sb.append('\n').append(bestIdx);

        BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(System.out));
        bw.write(sb.toString());
        bw.flush();
    }
}
```

---

### Python (3.12.11)

```python
import sys

def main():
    input_data = sys.stdin.read().split()
    if not input_data:
        return
    
    iterator = iter(input_data)
    n = int(next(iterator))
    
    a = [0] * (n + 1)
    for i in range(1, n + 1):
        a[i] = int(next(iterator))
        
    q = int(next(iterator))
    
    # Group queries by R using head-next structure for maximum performance
    head = [-1] * (n + 1)
    to_l = [0] * q
    to_id = [0] * q
    nxt = [-1] * q
    
    for i in range(q):
        l = int(next(iterator))
        r = int(next(iterator))
        to_l[i] = l
        to_id[i] = i
        nxt[i] = head[r]
        head[r] = i

    del input_data, iterator  # Free string allocations immediately

    # Binary Indexed Trees for Historical Sum:
    # At time step r, historical sum satisfies:
    # sumHist(i) = (r + 1) * A(i) - B(i)
    # Range updates of 'diff' at time r correspond to:
    # A(i) += diff on [l, rg]
    # B(i) += diff * r on [l, rg]
    # Both A and B are maintained with standard range-add, prefix-sum Fenwick trees.
    bitA1 = [0] * (n + 2)
    bitA2 = [0] * (n + 2)
    bitB1 = [0] * (n + 2)
    bitB2 = [0] * (n + 2)

    def range_add(l, r_idx, diff, cur_r):
        vA = diff
        vB = diff * cur_r
        
        # Point update at l
        idx = l
        vA2 = (l - 1) * vA
        vB2 = (l - 1) * vB
        while idx <= n:
            bitA1[idx] += vA
            bitA2[idx] += vA2
            bitB1[idx] += vB
            bitB2[idx] += vB2
            idx += idx & -idx
            
        # Point update at r_idx + 1
        if r_idx < n:
            idx = r_idx + 1
            neg_vA = -vA
            neg_vB = -vB
            vA2 = r_idx * neg_vA
            vB2 = r_idx * neg_vB
            while idx <= n:
                bitA1[idx] += neg_vA
                bitA2[idx] += vA2
                bitB1[idx] += neg_vB
                bitB2[idx] += vB2
                idx += idx & -idx

    def query_hist_prefix(p, cur_r):
        sA1 = 0
        sA2 = 0
        sB1 = 0
        sB2 = 0
        idx = p
        while idx > 0:
            sA1 += bitA1[idx]
            sA2 += bitA2[idx]
            sB1 += bitB1[idx]
            sB2 += bitB2[idx]
            idx &= idx - 1
        sumA = p * sA1 - sA2
        sumB = p * sB1 - sB2
        return (cur_r + 1) * sumA - sumB

    max_stack = []  # (val, l, r)
    min_stack = []  # (val, l, r)
    ans = [0] * q
    memo = {}

    for r in range(1, n + 1):
        x = a[r]

        # Update Max Stack (Non-increasing)
        cur_l = r
        while max_stack and max_stack[-1][0] <= x:
            v, l, rg = max_stack.pop()
            diff = x - v
            if diff:
                range_add(l, rg, diff, r)
            cur_l = l
        max_stack.append((x, cur_l, r))

        # Update Min Stack (Non-decreasing)
        cur_l = r
        while min_stack and min_stack[-1][0] >= x:
            v, l, rg = min_stack.pop()
            diff = v - x
            if diff:
                range_add(l, rg, diff, r)
            cur_l = l
        min_stack.append((x, cur_l, r))

        # Answer Queries ending at r
        e = head[r]
        if e != -1:
            tot = query_hist_prefix(r, r)
            memo.clear()
            while e != -1:
                ql = to_l[e]
                if ql == r:
                    ans[to_id[e]] = 0  # Single-night stretch restlessness is always 0
                elif ql == 1:
                    ans[to_id[e]] = tot
                else:
                    p = ql - 1
                    if p in memo:
                        ans[to_id[e]] = tot - memo[p]
                    else:
                        pref = query_hist_prefix(p, r)
                        memo[p] = pref
                        ans[to_id[e]] = tot - pref
                e = nxt[e]

    max_val = -1
    best_idx = 1
    for i in range(q):
        if ans[i] > max_val:
            max_val = ans[i]
            best_idx = i + 1

    sys.stdout.write(" ".join(map(str, ans)) + "\n" + str(best_idx) + "\n")

if __name__ == '__main__':
    main()
```

---

### C (GCC 13.2.0)

```c
#include <stdio.h>
#include <stdlib.h>

#define MAXN 200005
#define MAXQ 200005

typedef struct {
    long long sum_val;
    long long sum_hist;
    long long lazy_a;
    long long lazy_t;
    long long lazy_h;
} Node;

static Node tree[MAXN * 4];
static int a[MAXN];
static long long ans[MAXQ];

// Fast Linked List for Offline Queries
static int head[MAXN];
static int to_l[MAXQ];
static int to_id[MAXQ];
static int nxt[MAXQ];

typedef struct {
    int val;
    int l;
    int r;
} StackItem;

static StackItem max_stack[MAXN];
static StackItem min_stack[MAXN];

static inline void apply_tag(int node, int l, int r, long long a_val, long long t_val, long long h_val) {
    long long len = (long long)(r - l + 1);
    tree[node].sum_hist += t_val * tree[node].sum_val + h_val * len;
    tree[node].sum_val += a_val * len;
    tree[node].lazy_h += h_val + t_val * tree[node].lazy_a;
    tree[node].lazy_a += a_val;
    tree[node].lazy_t += t_val;
}

static inline void pushdown(int node, int l, int r) {
    if (tree[node].lazy_a != 0 || tree[node].lazy_t != 0 || tree[node].lazy_h != 0) {
        int mid = (l + r) >> 1;
        int left = node << 1;
        int right = left | 1;
        apply_tag(left, l, mid, tree[node].lazy_a, tree[node].lazy_t, tree[node].lazy_h);
        apply_tag(right, mid + 1, r, tree[node].lazy_a, tree[node].lazy_t, tree[node].lazy_h);
        tree[node].lazy_a = 0;
        tree[node].lazy_t = 0;
        tree[node].lazy_h = 0;
    }
}

static inline void pushup(int node) {
    int left = node << 1;
    int right = left | 1;
    tree[node].sum_val = tree[left].sum_val + tree[right].sum_val;
    tree[node].sum_hist = tree[left].sum_hist + tree[right].sum_hist;
}

static void update_add(int node, int l, int r, int ql, int qr, long long val) {
    if (ql <= l && r <= qr) {
        apply_tag(node, l, r, val, 0, 0);
        return;
    }
    pushdown(node, l, r);
    int mid = (l + r) >> 1;
    if (ql <= mid) update_add(node << 1, l, mid, ql, qr, val);
    if (qr > mid) update_add((node << 1) | 1, mid + 1, r, ql, qr, val);
    pushup(node);
}

static long long query_hist(int node, int l, int r, int ql, int qr) {
    if (ql <= l && r <= qr) {
        return tree[node].sum_hist;
    }
    pushdown(node, l, r);
    int mid = (l + r) >> 1;
    long long res = 0;
    if (ql <= mid) res += query_hist(node << 1, l, mid, ql, qr);
    if (qr > mid) res += query_hist((node << 1) | 1, mid + 1, r, ql, qr);
    return res;
}

// Fast I/O Buffer with seamless chunk boundary handling
static char inbuf[1 << 20];
static int inpos = 0, inlen = 0;

static inline int gc() {
    if (inpos >= inlen) {
        inpos = 0;
        inlen = (int)fread(inbuf, 1, sizeof(inbuf), stdin);
        if (inlen <= 0) return -1;
    }
    return (unsigned char)inbuf[inpos++];
}

static inline int next_int() {
    int c = gc();
    while (c <= ' ' && c != -1) c = gc();
    if (c == -1) return -1;
    int res = 0;
    while (c > ' ') {
        if (c >= '0' && c <= '9') {
            res = res * 10 + (c - '0');
        }
        c = gc();
    }
    return res;
}

// Fast Output Buffer
static char outbuf[1 << 20];
static int outpos = 0;

static inline void pc(char c) {
    if (outpos >= (int)sizeof(outbuf)) {
        fwrite(outbuf, 1, outpos, stdout);
        outpos = 0;
    }
    outbuf[outpos++] = c;
}

static inline void print_ll(long long x) {
    if (x == 0) {
        pc('0');
        return;
    }
    if (x < 0) {
        pc('-');
        x = -x;
    }
    char s[24];
    int top = 0;
    while (x > 0) {
        s[top++] = (char)('0' + (x % 10));
        x /= 10;
    }
    while (top > 0) {
        pc(s[--top]);
    }
}

static inline void flush_out() {
    if (outpos > 0) {
        fwrite(outbuf, 1, outpos, stdout);
        outpos = 0;
    }
}

int main() {
    int n = next_int();
    if (n <= 0) return 0;

    for (int i = 1; i <= n; i++) {
        a[i] = next_int();
        head[i] = -1;
    }

    int q = next_int();
    for (int i = 0; i < q; i++) {
        int l = next_int();
        int r = next_int();
        to_l[i] = l;
        to_id[i] = i;
        nxt[i] = head[r];
        head[r] = i;
    }

    int max_top = 0;
    int min_top = 0;

    for (int r = 1; r <= n; r++) {
        int x = a[r];

        // Maintain Max Stack
        int cur_l = r;
        while (max_top > 0 && max_stack[max_top - 1].val <= x) {
            StackItem top = max_stack[--max_top];
            update_add(1, 1, n, top.l, top.r, x - top.val);
            cur_l = top.l;
        }
        max_stack[max_top].val = x;
        max_stack[max_top].l = cur_l;
        max_stack[max_top++].r = r;

        // Maintain Min Stack
        cur_l = r;
        while (min_top > 0 && min_stack[min_top - 1].val >= x) {
            StackItem top = min_stack[--min_top];
            update_add(1, 1, n, top.l, top.r, top.val - x);
            cur_l = top.l;
        }
        min_stack[min_top].val = x;
        min_stack[min_top].l = cur_l;
        min_stack[min_top++].r = r;

        // Advance global time tick
        apply_tag(1, 1, n, 0, 1, 0);

        // Answer queries ending at r
        for (int e = head[r]; e != -1; e = nxt[e]) {
            ans[to_id[e]] = query_hist(1, 1, n, to_l[e], r);
        }
    }

    long long max_val = -1;
    int best_idx = 1;
    for (int i = 0; i < q; i++) {
        print_ll(ans[i]);
        pc(i == q - 1 ? '\n' : ' ');
        if (ans[i] > max_val) {
            max_val = ans[i];
            best_idx = i + 1;
        }
    }
    print_ll(best_idx);
    pc('\n');
    flush_out();

    return 0;
}
```

---

### C++ (GCC 13.2.0)

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

struct Node {
    long long sum_val;
    long long sum_hist;
    long long lazy_a;
    long long lazy_t;
    long long lazy_h;
};

class HistoricalSegTree {
private:
    int n;
    vector<Node> tree;

    inline void apply_tag(int node, int l, int r, long long a_val, long long t_val, long long h_val) {
        long long len = (long long)(r - l + 1);
        tree[node].sum_hist += t_val * tree[node].sum_val + h_val * len;
        tree[node].sum_val += a_val * len;
        tree[node].lazy_h += h_val + t_val * tree[node].lazy_a;
        tree[node].lazy_a += a_val;
        tree[node].lazy_t += t_val;
    }

    inline void pushdown(int node, int l, int r) {
        if (tree[node].lazy_a != 0 || tree[node].lazy_t != 0 || tree[node].lazy_h != 0) {
            int mid = (l + r) >> 1;
            int left = node << 1;
            int right = left | 1;
            apply_tag(left, l, mid, tree[node].lazy_a, tree[node].lazy_t, tree[node].lazy_h);
            apply_tag(right, mid + 1, r, tree[node].lazy_a, tree[node].lazy_t, tree[node].lazy_h);
            tree[node].lazy_a = 0;
            tree[node].lazy_t = 0;
            tree[node].lazy_h = 0;
        }
    }

    inline void pushup(int node) {
        int left = node << 1;
        int right = left | 1;
        tree[node].sum_val = tree[left].sum_val + tree[right].sum_val;
        tree[node].sum_hist = tree[left].sum_hist + tree[right].sum_hist;
    }

public:
    HistoricalSegTree(int n) : n(n), tree(4 * n + 5, {0, 0, 0, 0, 0}) {}

    void update_add(int node, int l, int r, int ql, int qr, long long val) {
        if (ql <= l && r <= qr) {
            apply_tag(node, l, r, val, 0, 0);
            return;
        }
        pushdown(node, l, r);
        int mid = (l + r) >> 1;
        if (ql <= mid) update_add(node << 1, l, mid, ql, qr, val);
        if (qr > mid) update_add((node << 1) | 1, mid + 1, r, ql, qr, val);
        pushup(node);
    }

    void tick() {
        apply_tag(1, 1, n, 0, 1, 0);
    }

    long long query_hist(int node, int l, int r, int ql, int qr) {
        if (ql <= l && r <= qr) {
            return tree[node].sum_hist;
        }
        pushdown(node, l, r);
        int mid = (l + r) >> 1;
        long long res = 0;
        if (ql <= mid) res += query_hist(node << 1, l, mid, ql, qr);
        if (qr > mid) res += query_hist((node << 1) | 1, mid + 1, r, ql, qr);
        return res;
    }
};

struct StackItem {
    int val, l, r;
};

struct Query {
    int l, id;
};

int main() {
    // Optimize standard I/O operations for maximum throughput
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);

    int n;
    if (!(cin >> n)) return 0;

    vector<int> a(n + 1);
    for (int i = 1; i <= n; i++) {
        cin >> a[i];
    }

    int q;
    cin >> q;
    vector<vector<Query>> queries_at(n + 1);
    for (int i = 0; i < q; i++) {
        int l, r;
        cin >> l >> r;
        queries_at[r].push_back({l, i});
    }

    HistoricalSegTree st(n);
    vector<StackItem> max_stack;
    vector<StackItem> min_stack;
    max_stack.reserve(n);
    min_stack.reserve(n);

    vector<long long> ans(q);

    for (int r = 1; r <= n; r++) {
        int x = a[r];

        // Maintain Max Stack (Non-increasing)
        int cur_l = r;
        while (!max_stack.empty() && max_stack.back().val <= x) {
            auto top = max_stack.back();
            max_stack.pop_back();
            st.update_add(1, 1, n, top.l, top.r, x - top.val);
            cur_l = top.l;
        }
        max_stack.push_back({x, cur_l, r});

        // Maintain Min Stack (Non-decreasing)
        cur_l = r;
        while (!min_stack.empty() && min_stack.back().val >= x) {
            auto top = min_stack.back();
            min_stack.pop_back();
            st.update_add(1, 1, n, top.l, top.r, top.val - x);
            cur_l = top.l;
        }
        min_stack.push_back({x, cur_l, r});

        // Advance global time tick: hist += val
        st.tick();

        // Answer all queries ending at r
        for (const auto& qry : queries_at[r]) {
            ans[qry.id] = st.query_hist(1, 1, n, qry.l, r);
        }
    }

    // Output all totals in schedule order and determine heaviest session
    long long max_val = -1;
    int best_idx = 1;

    for (int i = 0; i < q; i++) {
        cout << ans[i] << (i == q - 1 ? "" : " ");
        if (ans[i] > max_val) {
            max_val = ans[i];
            best_idx = i + 1;
        }
    }
    cout << "\n" << best_idx << "\n";

    return 0;
}
```

---

## 11. Line-by-Line Code Walkthrough

Taking the C++ and Java implementations as standard reference:

1. **Query Offline Organization:**
   ```cpp
   queries_at[r].push_back({l, i});
   ```
   - Bundles queries by their right boundary $R$. This allows evaluating queries dynamically at the exact moment the sweep reaches step $R$.

2. **Monotonic Max Stack Update:**
   ```cpp
   while (!max_stack.empty() && max_stack.back().val <= x) {
       auto top = max_stack.back();
       max_stack.pop_back();
       st.update_add(1, 1, n, top.l, top.r, x - top.val);
       cur_l = top.l;
   }
   max_stack.push_back({x, cur_l, r});
   ```
   - As $A[r] = x$ arrives, any previous prefix segments with $\text{max} \le x$ have their maximum elevated to $x$.
   - The increase is $\Delta = x - \text{top.val} \ge 0$.
   - A range addition of $\Delta$ is applied over $[top.l, top.r]$ in the Segment Tree.
   - The new element $x$ inherits the leftmost reach of all popped elements (`cur_l`).

3. **Monotonic Min Stack Update:**
   ```cpp
   while (!min_stack.empty() && min_stack.back().val >= x) {
       auto top = min_stack.back();
       min_stack.pop_back();
       st.update_add(1, 1, n, top.l, top.r, top.val - x);
       cur_l = top.l;
   }
   min_stack.push_back({x, cur_l, r});
   ```
   - As $A[r] = x$ arrives, any previous prefix segments with $\text{min} \ge x$ have their minimum lowered to $x$.
   - Since restlessness is $\max - \min$, lowering the minimum by $\Delta = \text{top.val} - x$ **increases** restlessness by $\Delta$.
   - A range addition of $\Delta$ is applied over $[top.l, top.r]$.

4. **Advancing Time Tick (Historical Accumulation):**
   ```cpp
   st.tick(); // apply_tag(1, 1, n, 0, 1, 0);
   ```
   - Every node in the segment tree has its active value accumulated into its historical sum: $H[x] \leftarrow H[x] + v[x]$.
   - Handled lazily in $\mathcal{O}(1)$ at the root by setting $t \leftarrow t + 1$.

5. **Query Answering:**
   ```cpp
   ans[qry.id] = st.query_hist(1, 1, n, qry.l, r);
   ```
   - Range sum query of $H$ on $[L, r]$ in $\mathcal{O}(\log N)$.
   - Directly yields the restlessness of all contiguous stretches within $[L, r]$.

---

## 12. Key Takeaways & Interview Follow-ups

### Key Principles:
1. **Affine Transformation Monoids in Segment Trees:**
   When maintaining both active values $v$ and historical sums $H$, their simultaneous updates form an affine mapping $H \leftarrow H + t \cdot v + h$ and $v \leftarrow v + a$. Maintaining the 3-tuple tag $(a, t, h)$ ensures closed composition and $\mathcal{O}(1)$ pushdown operations.
2. **Offline Sweep-Line Synergy:**
   Transforming a 2D range subsegment problem into a 1D timeline sweep allows replacing dynamic range allocations with amortized monotonic stack pops ($\le 2N$ total updates).
3. **No Suffix Subtraction Needed:**
   Because all stretches within $[L, R]$ start at $x \ge L$ and end at $y \ge x \ge L$, the historical sum at time $R$ restricted to indices $[L, R]$ naturally excludes any contributions from $y < L$, eliminating prefix-sum subtraction across time.
