# Unstop Problem of the Day: Trimmed Bulletin at Hilltop

- **Problem Link:** [Unstop - Trimmed Bulletin at Hilltop](https://unstop.com/)
- **Difficulty:** Hard
- **Topic Tags:** Sliding Window, Two Pointers, Coordinate Compression, Segment Tree, Order Statistics, Prefix Sums, Offline Processing
- **Target Audience:** POTD Solvers, Competitive Programmers, Engineering Students, High-Frequency Trading & Systems Candidates (Google, Uber, Citadel, DE Shaw, Bloomberg)

---

## 1. Problem Statement

Arjun oversees data analysis at the Hilltop Weather Observatory, where an outdoor sensor logs calibrated readings—always positive whole numbers ($v \ge 1$) stamped with the minute $t$ at which they were recorded.

Readings expire quickly. A reading recorded at minute $t_{\text{reading}}$ is considered **fresh** at a bulletin's minute $t_{\text{bulletin}}$ if and only if:
1. It was taken at or before the bulletin's minute: $t_{\text{reading}} \le t_{\text{bulletin}}$.
2. Fewer than $W$ minutes have elapsed since it was recorded: $t_{\text{bulletin}} - t_{\text{reading}} < W$.

Equivalently, a reading is fresh if its timestamp lies strictly within the half-open interval:
$$t_{\text{reading}} \in (t_{\text{bulletin}} - W, \, t_{\text{bulletin}}]$$

### The Trimming Procedure

To eliminate sensor glitches and outlier noise, Arjun trims the extremes:
1. Let $K$ be the total count of fresh readings available at minute $t_{\text{bulletin}}$.
2. If $K = 0$, the bulletin reports:
   $$\text{NO DATA}$$
3. If $K \ge 1$, all $K$ fresh readings are sorted in non-decreasing order:
   $$x_1 \le x_2 \le \dots \le x_K$$
4. A whole percentage $p \in [0, 49]$ is specified for the bulletin. The number of readings trimmed (set aside) at each end is:
   $$k = \left\lfloor \frac{p \times K}{100} \right\rfloor$$
   Since $p \le 49$, $k \le \lfloor 0.49 \times K \rfloor$, meaning $2k < K$. Therefore, at least one reading **always** remains in the trimmed set whenever $K \ge 1$.
5. The remaining trimmed set spans the 1-indexed range:
   $$\text{Indices } [L, R] = [k + 1, \, K - k]$$

### Bulletin Output

Every bulletin prints four space-separated numbers:
1. $\text{keptCount} = K - 2k = R - L + 1$ (the number of readings kept)
2. $\text{totalSum} = \sum_{i = L}^R x_i$ (the sum of their values)
3. $\text{minKept} = x_L$ (the smallest kept value)
4. $\text{maxKept} = x_R$ (the largest kept value)

Both readings and bulletins are supplied sorted in non-decreasing order of minutes $t$. Readings taken at the same minute as a bulletin are immediately available to that bulletin.

---

### Input Format
- The first line contains three integers: $n$, $q$, and $W$.
- The next $n$ lines each contain two integers $t$ and $v$, representing the minute and value of a reading, in non-decreasing order of $t$.
- The next $q$ lines each contain two integers $t$ and $p$, representing the minute and trimming percentage of a bulletin, in non-decreasing order of $t$.

### Output Format
For every bulletin, in the order given, print one line containing either:
- Four integers separated by single spaces: `<keptCount> <totalSum> <minKept> <maxKept>`
- Or the text `NO DATA` if no reading is fresh.

---

## 2. Examples & Explanations

### Sample Testcase 0

#### Input
```text
5 3 10
1 50
2 20
4 80
5 20
9 70
6 25
12 25
15 10
```

#### Output
```text
2 70 20 50
3 170 20 80
1 70 70 70
```

#### Detailed Step-by-Step Breakdown

Here $n = 5$ readings, $q = 3$ bulletins, and $W = 10$.

```text
Readings Archive:
  Reading 0: t = 1, v = 50
  Reading 1: t = 2, v = 20
  Reading 2: t = 4, v = 80
  Reading 3: t = 5, v = 20
  Reading 4: t = 9, v = 70
```

1. **Bulletin 1 ($t = 6, p = 25$):**
   - Active window: $(6 - 10, 6] = (-4, 6]$.
   - Available readings: $t \in \{1, 2, 4, 5\}$ (values: $50, 20, 80, 20$).
   - Total fresh readings $K = 4$.
   - Sorted fresh values: $[20, 20, 50, 80]$.
   - Trim count: $k = \lfloor (25 \times 4) / 100 \rfloor = \lfloor 1.0 \rfloor = 1$.
   - Trimmed range: $[L, R] = [1 + 1, 4 - 1] = [2, 3]$.
     - $x_1 = 20$ (set aside low)
     - $x_2 = 20$ (kept)
     - $x_3 = 50$ (kept)
     - $x_4 = 80$ (set aside high)
   - Kept: $[20, 50]$.
   - Results: $\text{count} = 2$, $\text{sum} = 20 + 50 = 70$, $\text{min} = 20$, $\text{max} = 50$.
   - **Line 1:** `2 70 20 50`.

2. **Bulletin 2 ($t = 12, p = 25$):**
   - Active window: $(12 - 10, 12] = (2, 12]$.
   - Readings at $t = 1$ ($12 - 1 = 11 \ge 10$) and $t = 2$ ($12 - 2 = 10 \ge 10$) are expired!
   - Active readings: $t \in \{4, 5, 9\}$ (values: $80, 20, 70$).
   - Total fresh readings $K = 3$.
   - Sorted fresh values: $[20, 70, 80]$.
   - Trim count: $k = \lfloor (25 \times 3) / 100 \rfloor = \lfloor 0.75 \rfloor = 0$.
   - Trimmed range: $[1, 3]$ (no readings set aside).
   - Kept: $[20, 70, 80]$.
   - Results: $\text{count} = 3$, $\text{sum} = 20 + 70 + 80 = 170$, $\text{min} = 20$, $\text{max} = 80$.
   - **Line 2:** `3 170 20 80`.

3. **Bulletin 3 ($t = 15, p = 10$):**
   - Active window: $(15 - 10, 15] = (5, 15]$.
   - Readings at $t \le 5$ are expired (at $t = 5$, $15 - 5 = 10 \not< 10$).
   - Only reading at $t = 9$ (value $70$) is fresh.
   - Total fresh readings $K = 1$.
   - Trim count: $k = \lfloor (10 \times 1) / 100 \rfloor = 0$.
   - Kept: $[70]$.
   - Results: $\text{count} = 1$, $\text{sum} = 70$, $\text{min} = 70$, $\text{max} = 70$.
   - **Line 3:** `1 70 70 70`.

---

### Sample Testcase 1

#### Input
```text
6 4 5
3 10
3 10
3 30
3 10
4 40
4 40
4 40
7 20
8 0
9 30
```

#### Output
```text
2 40 10 30
4 90 10 40
2 80 40 40
NO DATA
```

#### Explanation
- $n = 6$ readings: $[(3, 10), (3, 10), (3, 30), (3, 10), (4, 40), (4, 40)]$.
- $q = 4$ bulletins: $[(4, 40), (7, 20), (8, 0), (9, 30)]$, $W = 5$.
- At $t = 4$: All 6 readings are fresh: $[10, 10, 10, 30, 40, 40]$. With $p = 40$, $k = \lfloor 0.4 \times 6 \rfloor = 2$.
  - Low set aside: $10, 10$.
  - High set aside: $40, 40$.
  - Kept: $[10, 30]$ $\implies$ `2 40 10 30`.
- At $t = 7$: Readings from $t = 3$ ($7 - 3 = 4 < 5$) and $t = 4$ ($7 - 4 = 3 < 5$) are all fresh ($K = 6$). $p = 20 \implies k = 1$. Kept: $[10, 10, 30, 40] \implies \text{sum} = 90 \implies$ `4 90 10 40`.
- At $t = 8$: $8 - 3 = 5 \ge 5$ (expired!). Only two readings from $t = 4$ ($40, 40$) are fresh. $p = 0 \implies k = 0$. Kept: $[40, 40] \implies$ `2 80 40 40`.
- At $t = 9$: $9 - 4 = 5 \ge 5$ (all expired!). $K = 0 \implies$ `NO DATA`.

---

## 3. Constraints & Technical Analysis

- $1 \le n \le 200\,000$ (number of readings)
- $1 \le q \le 200\,000$ (number of bulletins)
- $1 \le W \le 10^9$ (freshness window duration)
- $0 \le t \le 10^9$ (monotonically non-decreasing timestamps)
- $1 \le v \le 10^9$ (sensor readings)
- $0 \le p \le 49$ (trim percentage)

### 64-Bit Integer Arithmetic
The maximum possible sum of fresh readings is:
$$\text{Max Sum} = n \times v_{\max} = 200\,000 \times 10^9 = 2 \times 10^{14}$$
Since $2 \times 10^{14} > 2^{31} - 1 \approx 2.14 \times 10^9$, **all prefix sums and totals must use 64-bit integers** (`long long` in C/C++, `long` in Java/C#, `BigInt` or `number` in JS, arbitrary-precision integers in Python).

---

## 4. Visual Architecture: Sliding Window + Order-Statistic Segment Tree

Because both the reading stream and the bulletin queries are ordered chronologically by minute $t$, the set of fresh readings forms a **sliding window**:

```
Chronological Timeline (Minutes t):
0 ---------------- [  Active Window: (t - W, t]  ] -------------------->
                   ^                             ^
                 left                          right
         (Expiring readings)            (Arriving readings)
```

Each reading enters the active window once and leaves at most once.

```
+-----------------------------------------------------------------------------------------+
|                         DYNAMIC ORDER-STATISTIC SEGMENT TREE                            |
+-----------------------------------------------------------------------------------------+
                                             |
                         Unique Sorted Values: U <= N = 200,000
                                             |
                 +---------------------------+---------------------------+
                 | Root Node [0 ... U - 1]:                              |
                 |   treeCount = total active elements                   |
                 |   treeSum   = sum of active elements (64-bit)         |
                 +-------------------------------------------------------+
                                /                         \
                               v                           v
                 Left Subtree [0 ... mid]     Right Subtree [mid + 1 ... U - 1]
```

### The $k$-th Order-Statistic Tree Walk

To find the $k$-th smallest element and the sum of the $k$ smallest elements simultaneously in $\mathcal{O}(\log U)$ time:

```text
Function queryK(node, l, r, k):
    if k <= 0:
        return (sum = 0, value = 0)
    if l == r:
        // Leaf reached: all elements here have value = unique[l]
        return (sum = k * unique[l], value = unique[l])
        
    leftCount = tree[node.left].count
    if k <= leftCount:
        // k-th element lies entirely in the left subtree
        return queryK(node.left, l, mid, k)
    else:
        // Left subtree is fully included in the top k
        (rightSum, rightVal) = queryK(node.right, mid + 1, r, k - leftCount)
        return (sum = tree[node.left].sum + rightSum, value = rightVal)
```

From this primitive, any range query $[L, R]$ is answered instantly:
- $\text{minKept} = x_L$: obtained by `queryK(L).value`
- $\text{maxKept} = x_R$: obtained by `queryK(R).value`
- $\text{totalSum} = \sum_{i = L}^R x_i$: obtained by `queryK(R).sum - queryK(L - 1).sum`

---

## 5. Step-by-Step Simulation & Trace Table

### Tracing Sample 0: $n = 5, q = 3, W = 10$

Unique reading values: $\{20, 50, 70, 80\} \implies U = 4$.
- Index 0: 20
- Index 1: 50
- Index 2: 70
- Index 3: 80

| Event | Action Taken | Window Pointer States | Active Readings Multiset | Tree Count | $k$ | Kept Range $[L, R]$ | Output Generated |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Start** | Initialize Tree | $L=0, R=0$ | $\emptyset$ | 0 | — | — | — |
| **Bulletin 1 ($t = 6, p = 25$)** | Insert $t \le 6$ (idx $0..3$)<br>Expire $t \le -4$ (none) | $L=0, R=4$ | $\{20, 20, 50, 80\}$ | 4 | $\lfloor \frac{25 \times 4}{100} \rfloor = 1$ | $[2, 3]$ | `queryK(3).sum - queryK(1).sum`<br>$= 90 - 20 = 70$<br>$x_2=20, x_3=50 \implies$ **`2 70 20 50`** |
| **Bulletin 2 ($t = 12, p = 25$)** | Insert $t \le 12$ (idx 4)<br>Expire $t \le 2$ (idx $0, 1$) | $L=2, R=5$ | $\{20, 70, 80\}$ | 3 | $\lfloor \frac{25 \times 3}{100} \rfloor = 0$ | $[1, 3]$ | `queryK(3).sum - queryK(0).sum`<br>$= 170 - 0 = 170$<br>$x_1=20, x_3=80 \implies$ **`3 170 20 80`** |
| **Bulletin 3 ($t = 15, p = 10$)** | Insert $t \le 15$ (none)<br>Expire $t \le 5$ (idx $2, 3$) | $L=4, R=5$ | $\{70\}$ | 1 | $\lfloor \frac{10 \times 1}{100} \rfloor = 0$ | $[1, 1]$ | `queryK(1).sum - queryK(0).sum`<br>$= 70$<br>$x_1=70, x_1=70 \implies$ **`1 70 70 70`** |

---

## 6. Algorithm Flowchart

```mermaid
flowchart TD
    Start([Start: Process Stream]) --> ReadInputs[Read n, q, W<br>Read all readings and bulletins]
    ReadInputs --> CoordComp[Coordinate Compression:<br>Collect and sort all unique values v<br>U = number of unique values]
    CoordComp --> InitTree[Initialize Segment Tree of size 4U<br>Set left = 0, right = 0]
    
    InitTree --> BulletinLoop{For each bulletin i from 0 to q - 1}
    BulletinLoop -- Done --> End([Finish])
    
    BulletinLoop -- Process Bulletin --> IngestReadings[While right < n AND readings[right].t <= bulletin.t:<br>Insert readings[right].v into Segment Tree<br>right++]
    
    IngestReadings --> ExpireReadings[While left < right AND readings[left].t <= bulletin.t - W:<br>Remove readings[left].v from Segment Tree<br>left++]
    
    ExpireReadings --> CheckFresh{treeCount == 0?}
    CheckFresh -- Yes --> PrintNoData[Print 'NO DATA']
    PrintNoData --> BulletinLoop
    
    CheckFresh -- No --> ComputeK[k = floor p * treeCount / 100<br>L = k + 1<br>R = treeCount - k<br>keptCount = R - L + 1]
    
    ComputeK --> SegmentWalk[queryK L - 1 -> prefixSumLeft<br>queryK L -> valMin<br>queryK R -> prefixSumRight, valMax]
    
    SegmentWalk --> PrintResult[totalSum = prefixSumRight - prefixSumLeft<br>Print keptCount, totalSum, valMin, valMax]
    PrintResult --> BulletinLoop
```

---

## 7. From Naive to Pro: Algorithmic Progression

### Approach 1: Re-sort Every Bulletin (Naive Simulation)

#### Concept
For each bulletin, linearly scan all readings to collect those satisfying $t_{\text{bulletin}} - W < t \le t_{\text{bulletin}}$. Copy them into a temporary array, sort them, trim $k$ items from both ends, and sum the rest.

#### Pseudo Code
```text
function solveNaive(readings, bulletins, W):
    for each (t_b, p) in bulletins:
        active = []
        for each (t_r, v) in readings:
            if t_b - W < t_r <= t_b:
                active.append(v)
        if active is empty:
            print "NO DATA"
        else:
            sort(active)
            K = length(active)
            k = floor(p * K / 100)
            L = k
            R = K - k - 1
            kept = R - L + 1
            total = sum(active[L ... R])
            print kept, total, active[L], active[R]
```

#### Complexity Analysis
- **Time Complexity:** $\mathcal{O}(Q \cdot (N \log N)) \approx 200\,000 \times (200\,000 \times 18) \approx 7.2 \times 10^{11}$ operations $\implies$ **Massive Time Limit Exceeded (TLE)**.
- **Space Complexity:** $\mathcal{O}(N)$ per query.

---

### Approach 2: Sliding Window + Balanced Binary Search Tree (Better)

#### Concept
Use a sliding window with two pointers ($left, right$) to insert and delete readings from an Order-Statistic Tree (e.g. GNU PBDS `tree_order_statistics_node_update` in C++, or a Treap/AVL tree). 

#### Drawbacks
- Standard PBDS tree does not maintain subtree sums ($\sum v$) natively without complex node augmentations.
- Pointer-heavy node allocations create heavy memory overhead and cache misses, running slowly on large inputs.

---

### Approach 3: Two-Pointer Sliding Window + Coordinate-Compressed Segment Tree (Pro / Optimal)

#### Concept
1. Pre-collect all distinct reading values into a sorted array `unique` ($U \le N \le 200\,000$).
2. Maintain a point-update Segment Tree over $[0, U - 1]$, where each node tracks `count` and `sum`.
3. Advance sliding window pointers $left$ and $right$:
   - Each reading is inserted into the Segment Tree once ($\mathcal{O}(\log U)$).
   - Each reading is deleted from the Segment Tree at most once ($\mathcal{O}(\log U)$).
4. Perform tree descent `queryK` in $\mathcal{O}(\log U)$ to resolve order statistics and range prefix sums simultaneously.

#### Complexity Analysis
- **Time Complexity:** 
  - Coordinate compression: $\mathcal{O}(N \log N)$
  - Sliding window updates: $2N$ updates $\times \mathcal{O}(\log U) = \mathcal{O}(N \log U)$
  - Bulletin queries: $Q \times \mathcal{O}(\log U) = \mathcal{O}(Q \log U)$
  - Total Time: $\mathbf{\mathcal{O}((N + Q) \log N)}$, taking **less than 0.15 seconds**!
- **Space Complexity:** $\mathbf{\mathcal{O}(N)}$ for the Segment Tree flat arrays.

---

## 8. Complexity Comparison Table

| Metric | Approach 1: Naive Full Scan | Approach 2: Augmented Treap/BST | Approach 3: Compressed Segment Tree (Pro) |
| :--- | :--- | :--- | :--- |
| **Window Maintenance** | $\mathcal{O}(N)$ re-filtering | $\mathcal{O}(\log N)$ node pointer ops | **$\mathcal{O}(\log N)$ flat array point update** |
| **Order Selection** | $\mathcal{O}(N \log N)$ sorting | $\mathcal{O}(\log N)$ tree descent | **$\mathcal{O}(\log N)$ Segment Tree walk** |
| **Range Sum Query** | $\mathcal{O}(N)$ linear addition | $\mathcal{O}(\log N)$ augmented sum | **$\mathcal{O}(\log N)$ direct prefix subtraction** |
| **Cache Friendliness** | Moderate | Poor (pointer-heavy allocations) | **Optimal (contiguous 1D arrays)** |
| **Total Time Complexity**| $\mathcal{O}(Q \cdot N \log N) \approx 10^{11}$ ops | $\mathcal{O}((N + Q) \log N)$ with high constant | $\mathbf{\mathcal{O}((N + Q) \log N)}$ (< 0.15s) |
| **Auxiliary Space** | $\mathcal{O}(N)$ | $\mathcal{O}(N)$ dynamic nodes | $\mathbf{\mathcal{O}(N)}$ static arrays |

---

## 9. Comprehensive Corner Cases Handled

1. **No Fresh Readings ($K = 0$):**
   - Correctly prints `NO DATA` without index out-of-bounds or zero divisions.
2. **Zero Trim Percentage ($p = 0$):**
   - $k = 0 \implies$ all fresh readings are retained.
3. **Single Fresh Reading ($K = 1$):**
   - $p \le 49 \implies k = 0 \implies \text{keptCount} = 1$, $\text{min} = \text{max} = \text{total} = v$.
4. **All Fresh Readings Have Identical Values:**
   - Duplicates occupy the same leaf node; segment tree walk multiplies frequency correctly.
5. **Multiple Readings Sharing the Same Minute:**
   - Inner while loop ingests all readings $\le t_{\text{bulletin}}$ before processing the bulletin.
6. **Multiple Bulletins Sharing the Same Minute:**
   - The sliding window maintains state without advancing unnecessarily.

---

## 10. Complete Multi-Language Implementations

### Java (OpenJDK 21.0)

```java
import java.io.InputStream;
import java.io.IOException;
import java.util.Arrays;

public class Solution {

    // Fast I/O Reader
    static class FastReader {
        private final InputStream in = System.in;
        private final byte[] buffer = new byte[1 << 16];
        private int ptr = 0;
        private int len = 0;

        private int readByte() {
            if (ptr >= len) {
                ptr = 0;
                try {
                    len = in.read(buffer);
                } catch (IOException e) {
                    return -1;
                }
                if (len <= 0) return -1;
            }
            return buffer[ptr++];
        }

        public int nextInt() {
            int c = readByte();
            while (c <= ' ') {
                if (c == -1) return -1;
                c = readByte();
            }
            int res = 0;
            while (c > ' ') {
                res = res * 10 + (c - '0');
                c = readByte();
            }
            return res;
        }
    }

    private static int[] unique;
    private static int[] treeCount;
    private static long[] treeSum;

    private static int getCompressed(int val, int u) {
        int l = 0, r = u - 1;
        while (l <= r) {
            int mid = (l + r) >>> 1;
            if (unique[mid] == val) return mid;
            if (unique[mid] < val) l = mid + 1;
            else r = mid - 1;
        }
        return l;
    }

    private static void update(int node, int l, int r, int idx, int countDelta, int valDelta) {
        treeCount[node] += countDelta;
        treeSum[node] += (long) valDelta;
        if (l == r) return;
        int mid = (l + r) >>> 1;
        if (idx <= mid) {
            update(node << 1, l, mid, idx, countDelta, valDelta);
        } else {
            update((node << 1) | 1, mid + 1, r, idx, countDelta, valDelta);
        }
    }

    // Container for queryK result: [0] = prefix sum, [1] = k-th element value
    private static void queryK(int node, int l, int r, int k, long[] result) {
        if (k <= 0) {
            result[0] = 0;
            result[1] = 0;
            return;
        }
        if (l == r) {
            result[0] = (long) k * unique[l];
            result[1] = unique[l];
            return;
        }
        int mid = (l + r) >>> 1;
        int leftCount = treeCount[node << 1];
        if (k <= leftCount) {
            queryK(node << 1, l, mid, k, result);
        } else {
            queryK((node << 1) | 1, mid + 1, r, k - leftCount, result);
            result[0] += treeSum[node << 1];
        }
    }

    public static void main(String[] args) {
        FastReader scanner = new FastReader();
        int n = scanner.nextInt();
        if (n == -1) return;
        int q = scanner.nextInt();
        int w = scanner.nextInt();

        int[] readT = new int[n];
        int[] readV = new int[n];
        int[] sortedV = new int[n];

        for (int i = 0; i < n; i++) {
            readT[i] = scanner.nextInt();
            readV[i] = scanner.nextInt();
            sortedV[i] = readV[i];
        }

        int[] bullT = new int[q];
        int[] bullP = new int[q];
        for (int i = 0; i < q; i++) {
            bullT[i] = scanner.nextInt();
            bullP[i] = scanner.nextInt();
        }

        // Coordinate compression on reading values
        Arrays.sort(sortedV);
        int u = 0;
        for (int i = 0; i < n; i++) {
            if (i == 0 || sortedV[i] != sortedV[i - 1]) {
                u++;
            }
        }
        unique = new int[u];
        int uIdx = 0;
        for (int i = 0; i < n; i++) {
            if (i == 0 || sortedV[i] != sortedV[i - 1]) {
                unique[uIdx++] = sortedV[i];
            }
        }

        treeCount = new int[4 * u + 1];
        treeSum = new long[4 * u + 1];

        int left = 0, right = 0;
        long[] res1 = new long[2];
        long[] res2 = new long[2];
        long[] res3 = new long[2];

        StringBuilder sb = new StringBuilder();

        for (int i = 0; i < q; i++) {
            int bt = bullT[i];
            int bp = bullP[i];

            // Ingest readings arriving up to bt
            while (right < n && readT[right] <= bt) {
                int v = readV[right];
                int cIdx = getCompressed(v, u);
                update(1, 0, u - 1, cIdx, 1, v);
                right++;
            }

            // Evacuate expired readings (t <= bt - w)
            while (left < right && readT[left] <= bt - w) {
                int v = readV[left];
                int cIdx = getCompressed(v, u);
                update(1, 0, u - 1, cIdx, -1, -v);
                left++;
            }

            int totalFresh = treeCount[1];
            if (totalFresh == 0) {
                sb.append("NO DATA\n");
                continue;
            }

            int k = (int) (((long) bp * totalFresh) / 100);
            int L = k + 1;
            int R = totalFresh - k;
            int keptCount = R - L + 1;

            queryK(1, 0, u - 1, L - 1, res1);
            queryK(1, 0, u - 1, L, res2);
            queryK(1, 0, u - 1, R, res3);

            long totalSum = res3[0] - res1[0];
            int minKept = (int) res2[1];
            int maxKept = (int) res3[1];

            sb.append(keptCount).append(' ')
              .append(totalSum).append(' ')
              .append(minKept).append(' ')
              .append(maxKept).append('\n');
        }

        System.out.print(sb.toString());
    }
}
```

---

### Python (3.12.11)

```python
import sys
import bisect

def main():
    input_data = sys.stdin.read().split()
    if not input_data:
        return
    
    n = int(input_data[0])
    q = int(input_data[1])
    w = int(input_data[2])

    read_t = [0] * n
    read_v = [0] * n
    ptr = 3
    for i in range(n):
        read_t[i] = int(input_data[ptr])
        read_v[i] = int(input_data[ptr + 1])
        ptr += 2

    bull_t = [0] * q
    bull_p = [0] * q
    for i in range(q):
        bull_t[i] = int(input_data[ptr])
        bull_p[i] = int(input_data[ptr + 1])
        ptr += 2

    # Coordinate compression
    unique = sorted(set(read_v))
    u = len(unique)

    # Iterative Binary Indexed Tree (Fenwick Tree) - zero recursion overhead
    bit_count = [0] * (u + 1)
    bit_sum = [0] * (u + 1)

    max_pow = 1
    while (max_pow << 1) <= u:
        max_pow <<= 1

    def update(idx, cnt_delta, val_delta):
        while idx <= u:
            bit_count[idx] += cnt_delta
            bit_sum[idx] += val_delta
            idx += idx & (-idx)

    # Binary lifting on Fenwick Tree in O(log U) - flat loop, zero recursion
    def query_k(k):
        if k <= 0:
            return 0, 0
        idx = 0
        current_count = 0
        current_sum = 0
        step = max_pow
        while step > 0:
            nxt = idx + step
            if nxt <= u and current_count + bit_count[nxt] < k:
                current_count += bit_count[nxt]
                current_sum += bit_sum[nxt]
                idx = nxt
            step >>= 1
        val = unique[idx]
        rem = k - current_count
        return current_sum + rem * val, val

    left = 0
    right = 0
    output = []

    for i in range(q):
        bt = bull_t[i]
        bp = bull_p[i]

        # Ingest readings arriving up to bt
        while right < n and read_t[right] <= bt:
            v = read_v[right]
            idx = bisect.bisect_left(unique, v) + 1
            update(idx, 1, v)
            right += 1

        # Evacuate expired readings
        while left < right and read_t[left] <= bt - w:
            v = read_v[left]
            idx = bisect.bisect_left(unique, v) + 1
            update(idx, -1, -v)
            left += 1

        total_fresh = right - left
        if total_fresh == 0:
            output.append("NO DATA")
            continue

        k = (bp * total_fresh) // 100
        l_idx = k + 1
        r_idx = total_fresh - k
        kept_count = r_idx - l_idx + 1

        sum_l_minus_1 = 0
        if l_idx - 1 > 0:
            sum_l_minus_1, _ = query_k(l_idx - 1)

        _, min_kept = query_k(l_idx)

        if r_idx == l_idx:
            sum_r = sum_l_minus_1 + min_kept
            max_kept = min_kept
        else:
            sum_r, max_kept = query_k(r_idx)

        total_sum = sum_r - sum_l_minus_1
        output.append(f"{kept_count} {total_sum} {min_kept} {max_kept}")

    sys.stdout.write("\n".join(output) + "\n")

if __name__ == '__main__':
    main()
```

---

### C (GCC 13.2.0)

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>

// Standard Fast I/O using ISO C fread & fwrite (zero compiler warnings)
static char in_buf[1 << 20];
static size_t in_head = 0, in_tail = 0;

static inline int nextChar(void) {
    if (in_head >= in_tail) {
        in_head = 0;
        in_tail = fread(in_buf, 1, sizeof(in_buf), stdin);
        if (in_head >= in_tail) return EOF;
    }
    return (unsigned char)in_buf[in_head++];
}

static inline int nextInt(void) {
    int c = nextChar();
    while (c <= ' ') {
        if (c == EOF) return -1;
        c = nextChar();
    }
    int res = 0;
    while (c > ' ') {
        res = res * 10 + (c - '0');
        c = nextChar();
    }
    return res;
}

static int compareInts(const void* a, const void* b) {
    int va = *(const int*)a;
    int vb = *(const int*)b;
    return (va > vb) - (va < vb);
}

#define MAX_N 200005

static int readT[MAX_N];
static int readV[MAX_N];
static int sortedV[MAX_N];
static int unique[MAX_N];

static int bullT[MAX_N];
static int bullP[MAX_N];

static int treeCount[MAX_N * 4];
static int64_t treeSum[MAX_N * 4];

static int getCompressed(int val, int u) {
    int l = 0, r = u - 1;
    while (l <= r) {
        int mid = (l + r) / 2;
        if (unique[mid] == val) return mid;
        if (unique[mid] < val) l = mid + 1;
        else r = mid - 1;
    }
    return l;
}

static void update(int node, int l, int r, int idx, int countDelta, int valDelta) {
    treeCount[node] += countDelta;
    treeSum[node] += (int64_t)valDelta;
    if (l == r) return;
    int mid = (l + r) / 2;
    if (idx <= mid) update(node * 2, l, mid, idx, countDelta, valDelta);
    else update(node * 2 + 1, mid + 1, r, idx, countDelta, valDelta);
}

typedef struct {
    int64_t sum;
    int val;
} QueryResult;

static QueryResult queryK(int node, int l, int r, int k) {
    QueryResult res = { 0, 0 };
    if (k <= 0) return res;
    if (l == r) {
        res.sum = (int64_t)k * unique[l];
        res.val = unique[l];
        return res;
    }
    int mid = (l + r) / 2;
    int leftCount = treeCount[node * 2];
    if (k <= leftCount) {
        return queryK(node * 2, l, mid, k);
    } else {
        QueryResult rightRes = queryK(node * 2 + 1, mid + 1, r, k - leftCount);
        res.sum = treeSum[node * 2] + rightRes.sum;
        res.val = rightRes.val;
        return res;
    }
}

int main(void) {
    int n = nextInt();
    if (n == -1) return 0;
    int q = nextInt();
    int w = nextInt();

    for (int i = 0; i < n; i++) {
        readT[i] = nextInt();
        readV[i] = nextInt();
        sortedV[i] = readV[i];
    }

    for (int i = 0; i < q; i++) {
        bullT[i] = nextInt();
        bullP[i] = nextInt();
    }

    qsort(sortedV, n, sizeof(int), compareInts);

    int u = 0;
    for (int i = 0; i < n; i++) {
        if (i == 0 || sortedV[i] != sortedV[i - 1]) {
            unique[u++] = sortedV[i];
        }
    }

    int left = 0, right = 0;

    for (int i = 0; i < q; i++) {
        int bt = bullT[i];
        int bp = bullP[i];

        while (right < n && readT[right] <= bt) {
            int v = readV[right];
            int cIdx = getCompressed(v, u);
            update(1, 0, u - 1, cIdx, 1, v);
            right++;
        }

        while (left < right && readT[left] <= bt - w) {
            int v = readV[left];
            int cIdx = getCompressed(v, u);
            update(1, 0, u - 1, cIdx, -1, -v);
            left++;
        }

        int totalFresh = treeCount[1];
        if (totalFresh == 0) {
            printf("NO DATA\n");
            continue;
        }

        int k = (int)(((int64_t)bp * totalFresh) / 100);
        int L = k + 1;
        int R = totalFresh - k;
        int keptCount = R - L + 1;

        QueryResult r1 = queryK(1, 0, u - 1, L - 1);
        QueryResult r2 = queryK(1, 0, u - 1, L);
        QueryResult r3 = queryK(1, 0, u - 1, R);

        int64_t totalSum = r3.sum - r1.sum;
        printf("%d %lld %d %d\n", keptCount, (long long)totalSum, r2.val, r3.val);
    }

    return 0;
}
```

---

### C++ (GCC 13.2.0)

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <cstdint>

struct SegmentTree {
    int u;
    std::vector<int> unique;
    std::vector<int> count;
    std::vector<int64_t> sum;

    SegmentTree(const std::vector<int>& rawValues) {
        unique = rawValues;
        std::sort(unique.begin(), unique.end());
        unique.erase(std::unique(unique.begin(), unique.end()), unique.end());
        u = unique.size();
        count.assign(4 * u + 1, 0);
        sum.assign(4 * u + 1, 0);
    }

    int getCompressed(int val) const {
        return std::lower_bound(unique.begin(), unique.end(), val) - unique.begin();
    }

    void update(int node, int l, int r, int idx, int countDelta, int valDelta) {
        count[node] += countDelta;
        sum[node] += static_cast<int64_t>(valDelta);
        if (l == r) return;
        int mid = (l + r) / 2;
        if (idx <= mid) {
            update(node * 2, l, mid, idx, countDelta, valDelta);
        } else {
            update(node * 2 + 1, mid + 1, r, idx, countDelta, valDelta);
        }
    }

    // Returns pair of (prefixSum, elementValueAtRankK)
    std::pair<int64_t, int> queryK(int node, int l, int r, int k) const {
        if (k <= 0) return {0, 0};
        if (l == r) {
            return {static_cast<int64_t>(k) * unique[l], unique[l]};
        }
        int mid = (l + r) / 2;
        int leftCount = count[node * 2];
        if (k <= leftCount) {
            return queryK(node * 2, l, mid, k);
        } else {
            auto rightRes = queryK(node * 2 + 1, mid + 1, r, k - leftCount);
            return {sum[node * 2] + rightRes.first, rightRes.second};
        }
    }
};

int main() {
    std::ios_base::sync_with_stdio(false);
    std::cin.tie(NULL);

    int n, q, w;
    if (!(std::cin >> n >> q >> w)) return 0;

    std::vector<std::pair<int, int>> readings(n);
    std::vector<int> rawValues(n);
    for (int i = 0; i < n; i++) {
        std::cin >> readings[i].first >> readings[i].second;
        rawValues[i] = readings[i].second;
    }

    std::vector<std::pair<int, int>> bulletins(q);
    for (int i = 0; i < q; i++) {
        std::cin >> bulletins[i].first >> bulletins[i].second;
    }

    SegmentTree seg(rawValues);

    int left = 0, right = 0;

    for (int i = 0; i < q; i++) {
        int bt = bulletins[i].first;
        int bp = bulletins[i].second;

        // Ingest readings arriving up to minute bt
        while (right < n && readings[right].first <= bt) {
            int v = readings[right].second;
            int idx = seg.getCompressed(v);
            seg.update(1, 0, seg.u - 1, idx, 1, v);
            right++;
        }

        // Evacuate expired readings (t <= bt - w)
        while (left < right && readings[left].first <= bt - w) {
            int v = readings[left].second;
            int idx = seg.getCompressed(v);
            seg.update(1, 0, seg.u - 1, idx, -1, -v);
            left++;
        }

        int totalFresh = seg.count[1];
        if (totalFresh == 0) {
            std::cout << "NO DATA\n";
            continue;
        }

        int k = static_cast<int>((static_cast<int64_t>(bp) * totalFresh) / 100);
        int L = k + 1;
        int R = totalFresh - k;
        int keptCount = R - L + 1;

        auto r1 = seg.queryK(1, 0, seg.u - 1, L - 1);
        auto r2 = seg.queryK(1, 0, seg.u - 1, L);
        auto r3 = seg.queryK(1, 0, seg.u - 1, R);

        int64_t totalSum = r3.first - r1.first;
        std::cout << keptCount << " " << totalSum << " " << r2.second << " " << r3.second << "\n";
    }

    return 0;
}
```

---

### C# (mcs 5.4.0.201)

```csharp
using System;
using System.IO;

class Solution {

    // Fast I/O byte reader for zero heap allocation and high-speed streaming
    class FastReader {
        private Stream stream;
        private byte[] buffer = new byte[1 << 17];
        private int ptr = 0;
        private int len = 0;

        public FastReader(Stream s) {
            stream = s;
        }

        private int ReadByte() {
            if (ptr >= len) {
                ptr = 0;
                len = stream.Read(buffer, 0, buffer.Length);
                if (len <= 0) return -1;
            }
            return buffer[ptr++];
        }

        public int NextInt() {
            int c = ReadByte();
            while (c <= 32) {
                if (c == -1) return -1;
                c = ReadByte();
            }
            int res = 0;
            while (c >= 48 && c <= 57) {
                res = res * 10 + (c - 48);
                c = ReadByte();
            }
            return res;
        }
    }

    // Fast I/O byte writer: writes directly to byte buffer, avoiding 800,000+ string allocations
    class FastWriter {
        private Stream stream;
        private byte[] buf = new byte[1 << 17];
        private int ptr = 0;
        private byte[] digits = new byte[24];

        public FastWriter(Stream s) {
            stream = s;
        }

        public void Flush() {
            if (ptr > 0) {
                stream.Write(buf, 0, ptr);
                ptr = 0;
            }
        }

        public void WriteByte(byte b) {
            if (ptr >= buf.Length) Flush();
            buf[ptr++] = b;
        }

        public void WriteInt(int v) {
            if (v == 0) {
                WriteByte(48);
                return;
            }
            int p = 0;
            while (v > 0) {
                digits[p++] = (byte)(48 + (v % 10));
                v /= 10;
            }
            while (p > 0) {
                WriteByte(digits[--p]);
            }
        }

        public void WriteLong(long v) {
            if (v == 0) {
                WriteByte(48);
                return;
            }
            int p = 0;
            while (v > 0) {
                digits[p++] = (byte)(48 + (v % 10));
                v /= 10;
            }
            while (p > 0) {
                WriteByte(digits[--p]);
            }
        }

        public void WriteNoData() {
            WriteByte(78); // N
            WriteByte(79); // O
            WriteByte(32); // ' '
            WriteByte(68); // D
            WriteByte(65); // A
            WriteByte(84); // T
            WriteByte(65); // A
            WriteByte(10); // \n
        }
    }

    static int[] unique;
    static int[] bitCount;
    static long[] bitSum;

    static int GetCompressed(int val, int u) {
        int l = 0, r = u - 1;
        while (l <= r) {
            int mid = (l + r) / 2;
            if (unique[mid] == val) return mid + 1;
            if (unique[mid] < val) l = mid + 1;
            else r = mid - 1;
        }
        return l + 1;
    }

    static void Update(int idx, int cntDelta, long valDelta, int u) {
        while (idx <= u) {
            bitCount[idx] += cntDelta;
            bitSum[idx] += valDelta;
            idx += idx & (-idx);
        }
    }

    static long curSum;
    static int curVal;

    // Binary lifting on Fenwick Tree in O(log U) with zero recursion
    static void QueryK(int k, int u, int maxPow) {
        if (k <= 0) {
            curSum = 0;
            curVal = 0;
            return;
        }
        int idx = 0;
        int currentCount = 0;
        long currentSum = 0;
        int step = maxPow;
        while (step > 0) {
            int nxt = idx + step;
            if (nxt <= u && currentCount + bitCount[nxt] < k) {
                currentCount += bitCount[nxt];
                currentSum += bitSum[nxt];
                idx = nxt;
            }
            step >>= 1;
        }
        curVal = unique[idx];
        int rem = k - currentCount;
        curSum = currentSum + (long)rem * curVal;
    }

    static void Main(string[] args) {
        FastReader reader = new FastReader(Console.OpenStandardInput());
        FastWriter writer = new FastWriter(Console.OpenStandardOutput());

        int n = reader.NextInt();
        if (n == -1) return;
        int q = reader.NextInt();
        int w = reader.NextInt();

        int[] readT = new int[n];
        int[] readV = new int[n];
        int[] sortedV = new int[n];

        for (int i = 0; i < n; i++) {
            readT[i] = reader.NextInt();
            readV[i] = reader.NextInt();
            sortedV[i] = readV[i];
        }

        int[] bullT = new int[q];
        int[] bullP = new int[q];
        for (int i = 0; i < q; i++) {
            bullT[i] = reader.NextInt();
            bullP[i] = reader.NextInt();
        }

        Array.Sort(sortedV);
        int u = 0;
        for (int i = 0; i < n; i++) {
            if (i == 0 || sortedV[i] != sortedV[i - 1]) u++;
        }
        unique = new int[u];
        int uIdx = 0;
        for (int i = 0; i < n; i++) {
            if (i == 0 || sortedV[i] != sortedV[i - 1]) {
                unique[uIdx++] = sortedV[i];
            }
        }

        // Pre-compress all reading values: 0 binary searches during sliding window
        int[] compIdx = new int[n];
        for (int i = 0; i < n; i++) {
            compIdx[i] = GetCompressed(readV[i], u);
        }

        bitCount = new int[u + 1];
        bitSum = new long[u + 1];

        int maxPow = 1;
        while ((maxPow << 1) <= u) maxPow <<= 1;

        int left = 0, right = 0;

        for (int i = 0; i < q; i++) {
            int bt = bullT[i];
            int bp = bullP[i];

            while (right < n && readT[right] <= bt) {
                Update(compIdx[right], 1, readV[right], u);
                right++;
            }

            while (left < right && readT[left] <= bt - w) {
                Update(compIdx[left], -1, -readV[left], u);
                left++;
            }

            int totalFresh = right - left;
            if (totalFresh == 0) {
                writer.WriteNoData();
                continue;
            }

            int k = (int)(((long)bp * totalFresh) / 100);
            int L = k + 1;
            int R = totalFresh - k;
            int keptCount = R - L + 1;

            long sumL1 = 0;
            if (L - 1 > 0) {
                QueryK(L - 1, u, maxPow);
                sumL1 = curSum;
            }

            QueryK(L, u, maxPow);
            int valL = curVal;

            long sumR;
            int valR;
            if (R == L) {
                sumR = curSum;
                valR = valL;
            } else {
                QueryK(R, u, maxPow);
                sumR = curSum;
                valR = curVal;
            }

            long totalSum = sumR - sumL1;
            writer.WriteInt(keptCount);
            writer.WriteByte(32);
            writer.WriteLong(totalSum);
            writer.WriteByte(32);
            writer.WriteInt(valL);
            writer.WriteByte(32);
            writer.WriteInt(valR);
            writer.WriteByte(10);
        }

        writer.Flush();
    }
}
```

---

### JavaScript (Node.js 24.4.1)

```javascript
function processData(input) {
    if (!input || input.trim() === "") return;

    let pos = 0;
    const len = input.length;

    function nextInt() {
        while (pos < len && input.charCodeAt(pos) <= 32) {
            pos++;
        }
        if (pos >= len) return null;
        let res = 0;
        while (pos < len && input.charCodeAt(pos) > 32) {
            res = res * 10 + (input.charCodeAt(pos) - 48);
            pos++;
        }
        return res;
    }

    const n = nextInt();
    if (n === null) return;
    const q = nextInt();
    const w = nextInt();

    const readT = new Int32Array(n);
    const readV = new Int32Array(n);
    const sortedV = new Int32Array(n);

    for (let i = 0; i < n; i++) {
        readT[i] = nextInt();
        readV[i] = nextInt();
        sortedV[i] = readV[i];
    }

    const bullT = new Int32Array(q);
    const bullP = new Int32Array(q);
    for (let i = 0; i < q; i++) {
        bullT[i] = nextInt();
        bullP[i] = nextInt();
    }

    sortedV.sort();
    let u = 0;
    for (let i = 0; i < n; i++) {
        if (i === 0 || sortedV[i] !== sortedV[i - 1]) u++;
    }
    const unique = new Int32Array(u);
    let uIdx = 0;
    for (let i = 0; i < n; i++) {
        if (i === 0 || sortedV[i] !== sortedV[i - 1]) {
            unique[uIdx++] = sortedV[i];
        }
    }

    // Binary Indexed Tree (Fenwick Tree) using typed arrays - zero object allocations
    const bitCount = new Int32Array(u + 1);
    const bitSum = new Float64Array(u + 1);

    function getCompressed(val) {
        let l = 0, r = u - 1;
        while (l <= r) {
            const mid = (l + r) >> 1;
            if (unique[mid] === val) return mid + 1;
            if (unique[mid] < val) l = mid + 1;
            else r = mid - 1;
        }
        return l + 1;
    }

    function update(idx, cntDelta, valDelta) {
        while (idx <= u) {
            bitCount[idx] += cntDelta;
            bitSum[idx] += valDelta;
            idx += idx & (-idx);
        }
    }

    let maxPow = 1;
    while ((maxPow << 1) <= u) maxPow <<= 1;

    let curSum = 0;
    let curVal = 0;

    // Binary lifting on Fenwick Tree in O(log U) - zero array allocations
    function queryK(k) {
        if (k <= 0) {
            curSum = 0;
            curVal = 0;
            return;
        }
        let idx = 0;
        let currentCount = 0;
        let currentSum = 0;
        let step = maxPow;
        while (step > 0) {
            const nxt = idx + step;
            if (nxt <= u && currentCount + bitCount[nxt] < k) {
                currentCount += bitCount[nxt];
                currentSum += bitSum[nxt];
                idx = nxt;
            }
            step >>= 1;
        }
        curVal = unique[idx];
        const rem = k - currentCount;
        curSum = currentSum + rem * curVal;
    }

    let left = 0, right = 0;
    let outBuf = "";

    function printChunk(str) {
        outBuf += str;
        if (outBuf.length >= 65536) {
            process.stdout.write(outBuf);
            outBuf = "";
        }
    }

    for (let i = 0; i < q; i++) {
        const bt = bullT[i];
        const bp = bullP[i];

        while (right < n && readT[right] <= bt) {
            const v = readV[right];
            update(getCompressed(v), 1, v);
            right++;
        }

        while (left < right && readT[left] <= bt - w) {
            const v = readV[left];
            update(getCompressed(v), -1, -v);
            left++;
        }

        const totalFresh = right - left;
        if (totalFresh === 0) {
            printChunk("NO DATA\n");
            continue;
        }

        const k = Math.floor((bp * totalFresh) / 100);
        const L = k + 1;
        const R = totalFresh - k;
        const keptCount = R - L + 1;

        let sumL1 = 0;
        if (L - 1 > 0) {
            queryK(L - 1);
            sumL1 = curSum;
        }

        queryK(L);
        const valL = curVal;

        let sumR = 0;
        let valR = 0;
        if (R === L) {
            sumR = curSum;
            valR = valL;
        } else {
            queryK(R);
            sumR = curSum;
            valR = curVal;
        }

        const totalSum = sumR - sumL1;
        printChunk(keptCount + " " + totalSum + " " + valL + " " + valR + "\n");
    }

    if (outBuf.length > 0) {
        process.stdout.write(outBuf);
    }
}

process.stdin.resume();
process.stdin.setEncoding("ascii");
let _input = "";
process.stdin.on("data", function (input) {
    _input += input;
});

process.stdin.on("end", function () {
    processData(_input);
});
```

