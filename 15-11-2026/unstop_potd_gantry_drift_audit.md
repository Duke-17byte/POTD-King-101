# Unstop Problem of the Day: Gantry Drift Audit

- **Platform:** [Unstop](https://unstop.com/)
- **Difficulty:** Hard
- **Topic Tags:** Slope Trick, Priority Queue / Min-Max Heaps, Dynamic Programming, Convex Optimization, Infimal Convolution, Prefix Sums, Binary Search
- **Target Audience:** POTD Solvers, Computer Science Students, Competitive Programmers, Advanced Algorithm Enthusiasts, Technical Interview Candidates (Google, Uber, DE Shaw, Jane Street, Citadel, Tower Research)

---

## 1. Problem Statement

Vihaan maintains the inspection line of an electronics plant, where a single camera carriage rides on a long straight rail above a conveyor.
- The rail extends infinitely in both directions, and every position on it is an integer coordinate (which may be negative).
- The inspection programme consists of $n$ consecutive rounds numbered from $1$ to $n$.
- During round $i$, a component stops under the inspection line. The camera picture is perfectly focused only when the carriage rests inside the **sweet band** $[lo_i, hi_i]$ (both ends inclusive).
- If the carriage rests at integer coordinate $x$ outside the band, the image is blurred, incurring a **blur penalty** equal to the distance from $x$ to the nearest endpoint of $[lo_i, hi_i]$:
  $$\text{cost}_i(x) = \begin{cases} 
  0 & \text{if } lo_i \le x \le hi_i \\ 
  lo_i - x & \text{if } x < lo_i \\ 
  x - hi_i & \text{if } x > hi_i 
  \end{cases}$$

---

### Movement & Parking Constraints

1. **Round 1 (Initial Freedom):**  
   Before round $1$, the carriage can be parked at **any integer coordinate** Vihaan prefers with zero movement cost.
2. **Rounds 2 to $n$ (Limited Motor Sliding):**  
   Before round $i$ ($2 \le i \le n$), the carriage may slide at most $up_i$ units toward larger coordinates (right) and at most $down_i$ units toward smaller coordinates (left), measured from its resting position in round $i - 1$:
   $$x_{i-1} - down_i \le x_i \le x_{i-1} + up_i$$
   Staying put ($x_i = x_{i-1}$) is always permitted. The carriage may stop at any integer coordinate satisfying these limits.

---

### The Two Plant Inquiries

1. **Baseline Global Optimum:**  
   Find the **smallest total penalty** achievable by any valid sequence of positions $(x_1, x_2, \dots, x_n)$, and the **smallest final coordinate** $x_n$ at which the carriage can rest during round $n$ in some schedule achieving this minimal penalty.
2. **Shutdown Plans Evaluation ($q$ Queries):**  
   For each of $q$ independent hypothetical shutdown plans, the carriage is forced to rest at an exact coordinate $z$ during round $n$ ($x_n = z$). For each plan, compute the **smallest total penalty** over all valid schedules satisfying $x_n = z$.

---

## 2. Examples & Visual Walkthroughs

### Example 1 (Sample Testcase 0)

#### Input:
```text
4 3
5 7
10 12 2 1
3 4 1 5
8 9 3 0
9
4
12
```

#### Output:
```text
2 7
3
5
9
```

#### Visual Layout of Sweet Bands:
```text
Coordinate: 3  4  5  6  7  8  9 10 11 12
Round 1:    .  .  #  #  #  .  .  .  .  .   (Band [5, 7])
Round 2:    .  .  .  .  .  .  .  #  #  #   (Band [10, 12], limits: +2 right, -1 left)
Round 3:    #  #  .  .  .  .  .  .  .  .   (Band [3, 4],   limits: +1 right, -5 left)
Round 4:    .  .  .  .  .  #  #  .  .  .   (Band [8, 9],   limits: +3 right, -0 left)
```

#### Execution & Schedule Trace:
- **Optimal Schedule:** $x_1 = 7 \to x_2 = 9 \to x_3 = 4 \to x_4 = 7$
  - Round 1 at $x_1 = 7$: Inside $[5, 7] \implies \text{penalty} = 0$.
  - Round 2 at $x_2 = 9$: $x_2 - x_1 = 9 - 7 = +2 \le up_2 (2)$. Distance to $[10, 12]$ is $10 - 9 = 1$.
  - Round 3 at $x_3 = 4$: $x_3 - x_2 = 4 - 9 = -5 \implies 5 \le down_3 (5)$. Inside $[3, 4] \implies \text{penalty} = 0$.
  - Round 4 at $x_4 = 7$: $x_4 - x_3 = 7 - 4 = +3 \le up_4 (3)$. Distance to $[8, 9]$ is $8 - 7 = 1$.
  - **Total Penalty:** $0 + 1 + 0 + 1 = \mathbf{2}$.
- **Smallest Final Coordinate:** Any schedule with $x_4 \le 6$ incurs a penalty of at least $3$. Hence, the minimum final coordinate achieving total penalty $2$ is $\mathbf{7}$.
- **Evaluating Shutdown Plans:**
  - $z = 9$: Forces $x_4 = 9 \implies \text{Total Penalty} = \mathbf{3}$.
  - $z = 4$: Forces $x_4 = 4 \implies \text{Round 4 penalty} = 8 - 4 = 4$, prior minimum is $1 \implies \text{Total} = \mathbf{5}$.
  - $z = 12$: Forces $x_4 = 12 \implies \text{Total Penalty} = \mathbf{9}$.

---

### Example 2 (Sample Testcase 1)

#### Input:
```text
3 3
0 4
6 8 0 0
2 3 0 0
0
4
7
```

#### Output:
```text
3 3
8
3
7
```

#### Explanation:
All motor limits are $up_i = 0, down_i = 0$. The carriage must remain stationary at a single coordinate $x$ for all three rounds ($x_1 = x_2 = x_3 = x$).
- Total penalty function: $f(x) = \text{cost}_1(x) + \text{cost}_2(x) + \text{cost}_3(x)$.
- At $x = 3$: $\text{cost} = 0 + 3 + 0 = \mathbf{3}$.
- At $x = 4$: $\text{cost} = 0 + 2 + 1 = \mathbf{3}$.
- The minimal penalty is $\mathbf{3}$, and the smallest coordinate achieving it is $\mathbf{3}$.
- For $z = 0$: Penalties are $0 + 6 + 2 = \mathbf{8}$.
- For $z = 4$: Penalty is $\mathbf{3}$.
- For $z = 7$: Penalties are $3 + 0 + 4 = \mathbf{7}$.

---

## 3. Constraints & Scale Analysis

| Parameter | Constraint Range | Algorithmic Implications |
| :--- | :--- | :--- |
| Rounds Count $n$ | $1 \le n \le 200\,000$ | Any $\mathcal{O}(n^2)$ or coordinate-grid DP will severely TLE. |
| Queries Count $q$ | $1 \le q \le 200\,000$ | Each query must be evaluated in $\mathcal{O}(\log n)$ time. |
| Sweet Bands $[lo, hi]$ | $0 \le lo \le hi \le 10^9$ | Coordinate values span large domains ($> 10^9$). |
| Motor Limits $up, down$ | $0 \le up, down \le 10^4$ | Displacements accumulate across rounds. |
| Query Targets $z$ | $0 \le z \le 10^9$ | Independent evaluations of $f_n(z)$. |
| Total Penalty Bound | Up to $n \times 10^9 \approx 2 \times 10^{14}$ | **Must use 64-bit integers** (`long long` / `long` / `BigInt`). |
| Target Time Limit | $\approx 2.0\text{ seconds}$ | **$\mathcal{O}((n + q) \log n)$ Slope Trick with Binary Search is required.** |

---

## 4. Visual Architecture: The Slope Trick

### Mathematical Formulation of Dynamic Programming

Let $f_i(x)$ be the minimum total penalty for the first $i$ rounds given that the carriage rests at coordinate $x$ during round $i$.

1. **Base Case (Round 1):**
   $$f_1(x) = \text{cost}_1(x) = \max(0, lo_1 - x) + \max(0, x - hi_1)$$
2. **Transition (Rounds $2 \le i \le n$):**
   $$f_i(x) = \text{cost}_i(x) + \min_{y \in [x - up_i, x + down_i]} f_{i-1}(y)$$

---

### Convexity & Infimal Convolution

Let $g_{i-1}(x) = \min_{y \in [x - up_i, x + down_i]} f_{i-1}(y)$.
- Since $f_{i-1}$ is continuous and convex, $g_{i-1}$ is also continuous and convex.
- Adding $\text{cost}_i(x)$ preserves convexity. By mathematical induction, **$f_i(x)$ is convex for all $i$**.
- Because all slope changes are integers, $f_i(x)$ is a piecewise linear convex function whose slopes change by integer amounts at specific **inflection points**.

```
                           Convex Function f(x)
       Slope -2        Slope -1       Slope 0 (Flat Min)      Slope +1       Slope +2
   \              \               /─────────────────────\               /              /
    \              \             /   [opt_L, opt_R]      \             /              /
     \              \           /                         \           /              /
      ▼              ▼         ▼                           ▼         ▼              ▼
     L_2            L_1      opt_L                       opt_R      R_1            R_2
     (Left Max-Heap L)                                   (Right Min-Heap R)
```

---

### Dual-Heap Slope Trick Representation

We maintain the function $f(x)$ using:
1. **$min\_val$:** The global minimum value of $f(x)$.
2. **Max-Heap $L$:** Stores inflection points to the left of the flat minimum (where slope increases from negative values toward 0).
3. **Min-Heap $R$:** Stores inflection points to the right of the flat minimum (where slope increases from 0 toward positive values).
4. **`shift_L` and `shift_R`:** Lazy additive offsets for $L$ and $R$. The true coordinate of $p \in L$ is $p + \text{shift\_L}$, and for $p \in R$ is $p + \text{shift\_R}$.

The flat minimum interval is:
$$[opt\_L, opt\_R] = \big[\text{top}(L) + \text{shift\_L}, \; \text{top}(R) + \text{shift\_R}\big]$$

---

### State Transitions per Round

#### Step 1: Motor Sliding (Infimal Convolution)
The carriage can slide up to $down_i$ units left and $up_i$ units right:
- The left inflection points shift left by $down_i$:
  $$\text{shift\_L} \leftarrow \text{shift\_L} - down_i$$
- The right inflection points shift right by $up_i$:
  $$\text{shift\_R} \leftarrow \text{shift\_R} + up_i$$
- The flat minimum region expands to $[opt\_L - down_i, opt\_R + up_i]$ while $min\_val$ remains unchanged.

#### Step 2: Adding Sweet Band Cost $\text{cost}_i(x)$
Let the current expanded flat region be $[opt\_L, opt\_R]$. There are three mutually exclusive cases:

```mermaid
flowchart TD
    BandCheck{"Position of [lo_i, hi_i]<br/>relative to [opt_L, opt_R]"}

    BandCheck -->|hi_i < opt_L<br/>Entire band is to the LEFT| CaseLeft["Left Shift Case:<br/>min_val += (opt_L - hi_i)<br/>L.pop(), R.push(opt_L)<br/>L.push(lo_i), L.push(hi_i)"]
    BandCheck -->|lo_i > opt_R<br/>Entire band is to the RIGHT| CaseRight["Right Shift Case:<br/>min_val += (lo_i - opt_R)<br/>R.pop(), L.push(opt_R)<br/>R.push(lo_i), R.push(hi_i)"]
    BandCheck -->|Overlap<br/>lo_i <= opt_R and hi_i >= opt_L| CaseOverlap["Overlap Case:<br/>min_val unchanged<br/>L.push(lo_i)<br/>R.push(hi_i)"]

    style BandCheck fill:#E3F2FD,stroke:#1565C0,color:#0D47A1
    style CaseLeft fill:#FFE082,stroke:#FF8F00,color:#000000
    style CaseRight fill:#FFCDD2,stroke:#D32F2F,color:#B71C1C
    style CaseOverlap fill:#C8E6C9,stroke:#388E3C,color:#1B5E20
```

1. **Case 1: $hi_i < opt\_L$ (Band lies strictly to the left):**
   - The minimum value increases by the deficit at $opt\_L$:
     $$min\_val \leftarrow min\_val + (opt\_L - hi_i)$$
   - The old $opt\_L$ moves from $L$ to $R$:
     $$R.\text{push}(opt\_L), \quad L.\text{pop}()$$
   - Both $lo_i$ and $hi_i$ are inserted into $L$:
     $$L.\text{push}(lo_i), \quad L.\text{push}(hi_i)$$
2. **Case 2: $lo_i > opt\_R$ (Band lies strictly to the right):**
   - The minimum value increases by the deficit at $opt\_R$:
     $$min\_val \leftarrow min\_val + (lo_i - opt\_R)$$
   - The old $opt\_R$ moves from $R$ to $L$:
     $$L.\text{push}(opt\_R), \quad R.\text{pop}()$$
   - Both $lo_i$ and $hi_i$ are inserted into $R$:
     $$R.\text{push}(lo_i), \quad R.\text{push}(hi_i)$$
3. **Case 3: Band overlaps $[opt\_L, opt\_R]$:**
   - $min\_val$ does not change.
   - Insert $lo_i$ into $L$ and $hi_i$ into $R$:
     $$L.\text{push}(lo_i), \quad R.\text{push}(hi_i)$$

---

### Answering Queries in $\mathcal{O}(\log n)$ via Prefix Sums

After round $n$:
1. We extract all $n$ points from $L$ and add $\text{shift\_L}$, yielding array `leftPoints` sorted in ascending order.
2. We extract all $n$ points from $R$ and add $\text{shift\_R}$, yielding array `rightPoints` sorted in ascending order.
3. We precompute prefix sums for both arrays: `prefL` and `prefR`.

For any query coordinate $z$:
- If $opt\_L \le z \le opt\_R$:
  $$f_n(z) = min\_val$$
- If $z < opt\_L$:
  Binary search for the first index `idx` where `leftPoints[idx] > z`. All points from `idx` to $n - 1$ are strictly greater than $z$:
  $$f_n(z) = min\_val + \sum_{k = idx}^{n - 1} (\text{leftPoints}[k] - z)$$
- If $z > opt\_R$:
  Binary search for the first index `idx` where `rightPoints[idx] >= z`. All points from $0$ to `idx - 1` are strictly less than $z$:
  $$f_n(z) = min\_val + \sum_{k = 0}^{idx - 1} (z - \text{rightPoints}[k])$$

---

## 5. Step-by-Step Simulation Table

### Sample Testcase 0 Trace

- $n = 4$, Sweet bands: $[5, 7], [10, 12], [3, 4], [8, 9]$.

| Round $i$ | Input $[lo, hi, up, down]$ | Shift Adjustments | $(opt\_L, opt\_R)$ before Cost | Case | $min\_val$ | $(opt\_L, opt\_R)$ after Cost |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **1** | $[5, 7, -, -]$ | `shift_L = 0, shift_R = 0` | $(5, 7)$ | Initial | **0** | $[5, 7]$ |
| **2** | $[10, 12, 2, 1]$ | `shift_L = -1, shift_R = 2` | $(4, 9)$ | Case 2 ($lo = 10 > 9$) | $0 + (10 - 9) = \mathbf{1}$ | $[9, 10]$ |
| **3** | $[3, 4, 1, 5]$ | `shift_L = -6, shift_R = 3` | $(4, 11)$ | Case 1 ($hi = 4 \le 4$) | $1 + (4 - 4) = \mathbf{1}$ | $[4, 4]$ |
| **4** | $[8, 9, 3, 0]$ | `shift_L = -6, shift_R = 6` | $(4, 7)$ | Case 2 ($lo = 8 > 7$) | $1 + (8 - 7) = \mathbf{2}$ | $[7, 8]$ |

- **Minimum Total Penalty:** $min\_val = \mathbf{2}$.
- **Smallest Final Coordinate:** $opt\_L = \mathbf{7}$.

---

## 6. Approaches: Naive to Pro

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                             ALGORITHM BENCHMARK                             │
├────────────────────┬─────────────────┬─────────────────┬────────────────────┤
│ Method             │ Time Complexity │ Auxiliary Space │ Verdict            │
├────────────────────┼─────────────────┼─────────────────┼────────────────────┤
│ 1. Coordinate DP   │ O(n * Range^2)  │ O(Range)        │ TLE (Memory Limit) │
│ 2. Segment Tree DP │ O(n * log Range)│ O(Range)        │ TLE (Range = 10^9) │
│ 3. Slope Trick     │ O((n + q) log n)│ O(n)            │ Optimal (< 0.2s)   │
└────────────────────┴─────────────────┴─────────────────┴────────────────────┘
```

---

## 7. Complete Program Implementations

### Java (OpenJDK 21.0)

```java
import java.io.*;
import java.util.*;

class Main {

    // Primitive long binary heaps for maximum performance and zero GC overhead
    static class MaxHeap {
        private final long[] data;
        private int count = 0;

        public MaxHeap(int capacity) {
            this.data = new long[capacity];
        }

        public int size() { return count; }

        public void push(long val) {
            data[count] = val;
            up(count++);
        }

        public long pop() {
            long top = data[0];
            data[0] = data[--count];
            down(0);
            return top;
        }

        public long peek() { return data[0]; }

        private void up(int i) {
            while (i > 0) {
                int p = (i - 1) >> 1;
                if (data[i] > data[p]) {
                    long tmp = data[i]; data[i] = data[p]; data[p] = tmp;
                    i = p;
                } else break;
            }
        }

        private void down(int i) {
            while ((i << 1) + 1 < count) {
                int l = (i << 1) + 1, r = l + 1, best = l;
                if (r < count && data[r] > data[l]) best = r;
                if (data[best] > data[i]) {
                    long tmp = data[i]; data[i] = data[best]; data[best] = tmp;
                    i = best;
                } else break;
            }
        }
    }

    static class MinHeap {
        private final long[] data;
        private int count = 0;

        public MinHeap(int capacity) {
            this.data = new long[capacity];
        }

        public int size() { return count; }

        public void push(long val) {
            data[count] = val;
            up(count++);
        }

        public long pop() {
            long top = data[0];
            data[0] = data[--count];
            down(0);
            return top;
        }

        public long peek() { return data[0]; }

        private void up(int i) {
            while (i > 0) {
                int p = (i - 1) >> 1;
                if (data[i] < data[p]) {
                    long tmp = data[i]; data[i] = data[p]; data[p] = tmp;
                    i = p;
                } else break;
            }
        }

        private void down(int i) {
            while ((i << 1) + 1 < count) {
                int l = (i << 1) + 1, r = l + 1, best = l;
                if (r < count && data[r] < data[l]) best = r;
                if (data[best] < data[i]) {
                    long tmp = data[i]; data[i] = data[best]; data[best] = tmp;
                    i = best;
                } else break;
            }
        }
    }

    // High-speed byte-level input scanner
    static class FastScanner {
        private final InputStream in;
        private final byte[] buffer = new byte[1 << 16];
        private int ptr = 0, len = 0;

        public FastScanner(InputStream in) { this.in = in; }

        private int read() throws IOException {
            if (ptr >= len) {
                ptr = 0;
                len = in.read(buffer);
                if (len <= 0) return -1;
            }
            return buffer[ptr++];
        }

        public String next() throws IOException {
            int c = read();
            while (c <= ' ' && c != -1) c = read();
            if (c == -1) return null;
            StringBuilder sb = new StringBuilder();
            while (c > ' ') {
                sb.append((char) c);
                c = read();
            }
            return sb.toString();
        }

        public long nextLong() throws IOException {
            int c = read();
            while (c <= ' ' && c != -1) c = read();
            if (c == -1) return 0;
            boolean neg = false;
            if (c == '-') {
                neg = true;
                c = read();
            }
            long res = 0;
            while (c >= '0' && c <= '9') {
                res = res * 10 + (c - '0');
                c = read();
            }
            return neg ? -res : res;
        }

        public int nextInt() throws IOException {
            return (int) nextLong();
        }
    }

    public static void main(String[] args) throws IOException {
        FastScanner scanner = new FastScanner(System.in);
        String nStr = scanner.next();
        if (nStr == null) return;
        int n = Integer.parseInt(nStr);
        int q = scanner.nextInt();

        MaxHeap L = new MaxHeap(2 * n + 5);
        MinHeap R = new MinHeap(2 * n + 5);

        long shiftL = 0;
        long shiftR = 0;
        long minVal = 0;

        // Round 1
        long lo1 = scanner.nextLong();
        long hi1 = scanner.nextLong();
        L.push(lo1);
        R.push(hi1);

        // Rounds 2 to n
        for (int i = 1; i < n; i++) {
            long lo = scanner.nextLong();
            long hi = scanner.nextLong();
            long up = scanner.nextLong();
            long down = scanner.nextLong();

            shiftL -= down;
            shiftR += up;

            long optL = L.peek() + shiftL;
            long optR = R.peek() + shiftR;

            if (hi < optL) {
                minVal += (optL - hi);
                L.pop();
                R.push(optL - shiftR);
                L.push(lo - shiftL);
                L.push(hi - shiftL);
            } else if (lo > optR) {
                minVal += (lo - optR);
                R.pop();
                L.push(optR - shiftL);
                R.push(lo - shiftR);
                R.push(hi - shiftR);
            } else {
                L.push(lo - shiftL);
                R.push(hi - shiftR);
            }
        }

        long minX = L.peek() + shiftL;

        // Extract and sort inflection points
        int nL = L.size();
        long[] leftPoints = new long[nL];
        for (int i = nL - 1; i >= 0; i--) {
            leftPoints[i] = L.pop() + shiftL;
        }

        int nR = R.size();
        long[] rightPoints = new long[nR];
        for (int i = 0; i < nR; i++) {
            rightPoints[i] = R.pop() + shiftR;
        }

        long[] prefL = new long[nL + 1];
        for (int i = 0; i < nL; i++) prefL[i + 1] = prefL[i] + leftPoints[i];

        long[] prefR = new long[nR + 1];
        for (int i = 0; i < nR; i++) prefR[i + 1] = prefR[i] + rightPoints[i];

        long optLFinal = leftPoints[nL - 1];
        long optRFinal = rightPoints[0];

        StringBuilder sb = new StringBuilder();
        sb.append(minVal).append(' ').append(minX).append('\n');

        // Process shutdown plans
        for (int k = 0; k < q; k++) {
            long z = scanner.nextLong();
            if (z >= optLFinal && z <= optRFinal) {
                sb.append(minVal).append('\n');
            } else if (z < optLFinal) {
                int low = 0, high = nL;
                while (low < high) {
                    int mid = (low + high) >> 1;
                    if (leftPoints[mid] > z) high = mid;
                    else low = mid + 1;
                }
                long cnt = nL - low;
                long sum = prefL[nL] - prefL[low];
                sb.append(minVal + (sum - cnt * z)).append('\n');
            } else {
                int low = 0, high = nR;
                while (low < high) {
                    int mid = (low + high) >> 1;
                    if (rightPoints[mid] < z) low = mid + 1;
                    else high = mid;
                }
                long cnt = low;
                long sum = prefR[low];
                sb.append(minVal + (cnt * z - sum)).append('\n');
            }
        }

        System.out.print(sb.toString());
    }
}
```

---

### Python (3.12.11)

```python
import sys
import heapq
import bisect

def main():
    input_data = sys.stdin.read().split()
    if not input_data:
        return

    iterator = iter(input_data)
    n = int(next(iterator))
    q = int(next(iterator))

    # L is a max-heap (implemented using negated values)
    # R is a min-heap
    L = []
    R = []

    shift_L = 0
    shift_R = 0
    min_val = 0

    # Round 1
    lo1 = int(next(iterator))
    hi1 = int(next(iterator))
    heapq.heappush(L, -lo1)
    heapq.heappush(R, hi1)

    # Rounds 2 to n
    for _ in range(1, n):
        lo = int(next(iterator))
        hi = int(next(iterator))
        up = int(next(iterator))
        down = int(next(iterator))

        shift_L -= down
        shift_R += up

        opt_L = -L[0] + shift_L
        opt_R = R[0] + shift_R

        if hi < opt_L:
            min_val += (opt_L - hi)
            heapq.heappop(L)
            heapq.heappush(R, opt_L - shift_R)
            heapq.heappush(L, -(lo - shift_L))
            heapq.heappush(L, -(hi - shift_L))
        elif lo > opt_R:
            min_val += (lo - opt_R)
            heapq.heappop(R)
            heapq.heappush(L, -(opt_R - shift_L))
            heapq.heappush(R, lo - shift_R)
            heapq.heappush(R, hi - shift_R)
        else:
            heapq.heappush(L, -(lo - shift_L))
            heapq.heappush(R, hi - shift_R)

    min_x = -L[0] + shift_L

    # Extract all inflection points in ascending order
    left_points = []
    while L:
        left_points.append(-heapq.heappop(L) + shift_L)
    left_points.reverse()

    right_points = []
    while R:
        right_points.append(heapq.heappop(R) + shift_R)

    nL = len(left_points)
    nR = len(right_points)

    pref_L = [0] * (nL + 1)
    for i in range(nL):
        pref_L[i + 1] = pref_L[i] + left_points[i]

    pref_R = [0] * (nR + 1)
    for i in range(nR):
        pref_R[i + 1] = pref_R[i] + right_points[i]

    opt_L_final = left_points[-1]
    opt_R_final = right_points[0]

    out = [f"{min_val} {min_x}"]

    # Evaluate shutdown plans
    for _ in range(q):
        z = int(next(iterator))
        if opt_L_final <= z <= opt_R_final:
            out.append(str(min_val))
        elif z < opt_L_final:
            idx = bisect.bisect_right(left_points, z)
            cnt = nL - idx
            sum_val = pref_L[nL] - pref_L[idx]
            out.append(str(min_val + (sum_val - cnt * z)))
        else:
            idx = bisect.bisect_left(right_points, z)
            cnt = idx
            sum_val = pref_R[idx]
            out.append(str(min_val + (cnt * z - sum_val)))

    sys.stdout.write("\n".join(out) + "\n")

if __name__ == '__main__':
    main()
```

---

### C (GCC 13.2.0)

```c
#include <stdio.h>
#include <stdlib.h>

#define MAXN 200005

// Flat array binary heaps
static long long L_heap[2 * MAXN];
static int L_size = 0;

static long long R_heap[2 * MAXN];
static int R_size = 0;

static void L_push(long long val) {
    int i = L_size++;
    L_heap[i] = val;
    while (i > 0) {
        int p = (i - 1) >> 1;
        if (L_heap[i] > L_heap[p]) {
            long long tmp = L_heap[i]; L_heap[i] = L_heap[p]; L_heap[p] = tmp;
            i = p;
        } else break;
    }
}

static long long L_pop() {
    long long top = L_heap[0];
    L_heap[0] = L_heap[--L_size];
    int i = 0;
    while ((i << 1) + 1 < L_size) {
        int l = (i << 1) + 1, r = l + 1, best = l;
        if (r < L_size && L_heap[r] > L_heap[l]) best = r;
        if (L_heap[best] > L_heap[i]) {
            long long tmp = L_heap[i]; L_heap[i] = L_heap[best]; L_heap[best] = tmp;
            i = best;
        } else break;
    }
    return top;
}

static void R_push(long long val) {
    int i = R_size++;
    R_heap[i] = val;
    while (i > 0) {
        int p = (i - 1) >> 1;
        if (R_heap[i] < R_heap[p]) {
            long long tmp = R_heap[i]; R_heap[i] = R_heap[p]; R_heap[p] = tmp;
            i = p;
        } else break;
    }
}

static long long R_pop() {
    long long top = R_heap[0];
    R_heap[0] = R_heap[--R_size];
    int i = 0;
    while ((i << 1) + 1 < R_size) {
        int l = (i << 1) + 1, r = l + 1, best = l;
        if (r < R_size && R_heap[r] < R_heap[l]) best = r;
        if (R_heap[best] < R_heap[i]) {
            long long tmp = R_heap[i]; R_heap[i] = R_heap[best]; R_heap[best] = tmp;
            i = best;
        } else break;
    }
    return top;
}

static long long left_points[MAXN];
static long long right_points[MAXN];
static long long pref_L[MAXN + 1];
static long long pref_R[MAXN + 1];

int main() {
    int n, q;
    if (scanf("%d %d", &n, &q) != 2) return 0;

    long long shift_L = 0;
    long long shift_R = 0;
    long long min_val = 0;

    long long lo1, hi1;
    scanf("%lld %lld", &lo1, &hi1);
    L_push(lo1);
    R_push(hi1);

    for (int i = 1; i < n; i++) {
        long long lo, hi, up, down;
        scanf("%lld %lld %lld %lld", &lo, &hi, &up, &down);

        shift_L -= down;
        shift_R += up;

        long long opt_L = L_heap[0] + shift_L;
        long long opt_R = R_heap[0] + shift_R;

        if (hi < opt_L) {
            min_val += (opt_L - hi);
            L_pop();
            R_push(opt_L - shift_R);
            L_push(lo - shift_L);
            L_push(hi - shift_L);
        } else if (lo > opt_R) {
            min_val += (lo - opt_R);
            R_pop();
            L_push(opt_R - shift_L);
            R_push(lo - shift_R);
            R_push(hi - shift_R);
        } else {
            L_push(lo - shift_L);
            R_push(hi - shift_R);
        }
    }

    long long min_x = L_heap[0] + shift_L;

    int nL = L_size;
    for (int i = nL - 1; i >= 0; i--) {
        left_points[i] = L_pop() + shift_L;
    }

    int nR = R_size;
    for (int i = 0; i < nR; i++) {
        right_points[i] = R_pop() + shift_R;
    }

    for (int i = 0; i < nL; i++) pref_L[i + 1] = pref_L[i] + left_points[i];
    for (int i = 0; i < nR; i++) pref_R[i + 1] = pref_R[i] + right_points[i];

    long long opt_L_final = left_points[nL - 1];
    long long opt_R_final = right_points[0];

    printf("%lld %lld\n", min_val, min_x);

    for (int k = 0; k < q; k++) {
        long long z;
        scanf("%lld", &z);

        if (z >= opt_L_final && z <= opt_R_final) {
            printf("%lld\n", min_val);
        } else if (z < opt_L_final) {
            int low = 0, high = nL;
            while (low < high) {
                int mid = (low + high) >> 1;
                if (left_points[mid] > z) high = mid;
                else low = mid + 1;
            }
            long long cnt = nL - low;
            long long sum = pref_L[nL] - pref_L[low];
            printf("%lld\n", min_val + (sum - cnt * z));
        } else {
            int low = 0, high = nR;
            while (low < high) {
                int mid = (low + high) >> 1;
                if (right_points[mid] < z) low = mid + 1;
                else high = mid;
            }
            long long cnt = low;
            long long sum = pref_R[low];
            printf("%lld\n", min_val + (cnt * z - sum));
        }
    }

    return 0;
}
```

---

### C++ (GCC 13.2.0)

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <algorithm>

using namespace std;

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);

    int n, q;
    if (!(cin >> n >> q)) return 0;

    priority_queue<long long> L; // Max-heap
    priority_queue<long long, vector<long long>, greater<long long>> R; // Min-heap

    long long shift_L = 0;
    long long shift_R = 0;
    long long min_val = 0;

    // Round 1
    long long lo1, hi1;
    cin >> lo1 >> hi1;
    L.push(lo1);
    R.push(hi1);

    // Rounds 2 to n
    for (int i = 1; i < n; i++) {
        long long lo, hi, up, down;
        cin >> lo >> hi >> up >> down;

        shift_L -= down;
        shift_R += up;

        long long opt_L = L.top() + shift_L;
        long long opt_R = R.top() + shift_R;

        if (hi < opt_L) {
            min_val += (opt_L - hi);
            L.pop();
            R.push(opt_L - shift_R);
            L.push(lo - shift_L);
            L.push(hi - shift_L);
        } else if (lo > opt_R) {
            min_val += (lo - opt_R);
            R.pop();
            L.push(opt_R - shift_L);
            R.push(lo - shift_R);
            R.push(hi - shift_R);
        } else {
            L.push(lo - shift_L);
            R.push(hi - shift_R);
        }
    }

    long long min_x = L.top() + shift_L;

    // Reconstruct sorted inflection points
    int nL = L.size();
    vector<long long> left_points(nL);
    for (int i = nL - 1; i >= 0; i--) {
        left_points[i] = L.top() + shift_L;
        L.pop();
    }

    int nR = R.size();
    vector<long long> right_points(nR);
    for (int i = 0; i < nR; i++) {
        right_points[i] = R.top() + shift_R;
        R.pop();
    }

    vector<long long> pref_L(nL + 1, 0);
    for (int i = 0; i < nL; i++) pref_L[i + 1] = pref_L[i] + left_points[i];

    vector<long long> pref_R(nR + 1, 0);
    for (int i = 0; i < nR; i++) pref_R[i + 1] = pref_R[i] + right_points[i];

    long long opt_L_final = left_points.back();
    long long opt_R_final = right_points.front();

    cout << min_val << " " << min_x << "\n";

    // Answer shutdown plans
    for (int k = 0; k < q; k++) {
        long long z;
        cin >> z;

        if (z >= opt_L_final && z <= opt_R_final) {
            cout << min_val << "\n";
        } else if (z < opt_L_final) {
            int idx = upper_bound(left_points.begin(), left_points.end(), z) - left_points.begin();
            long long cnt = nL - idx;
            long long sum = pref_L[nL] - pref_L[idx];
            cout << min_val + (sum - cnt * z) << "\n";
        } else {
            int idx = lower_bound(right_points.begin(), right_points.end(), z) - right_points.begin();
            long long cnt = idx;
            long long sum = pref_R[idx];
            cout << min_val + (cnt * z - sum) << "\n";
        }
    }

    return 0;
}
```

---

### C# (mcs 5.4.0.201)

```csharp
using System;
using System.IO;
using System.Text;

class Solution {
    class MaxHeap {
        private readonly long[] data;
        private int count;

        public MaxHeap(int capacity) {
            data = new long[capacity];
            count = 0;
        }

        public int Count { get { return count; } }

        public void Push(long val) {
            data[count] = val;
            Up(count++);
        }

        public long Pop() {
            long top = data[0];
            data[0] = data[--count];
            Down(0);
            return top;
        }

        public long Peek() { return data[0]; }

        private void Up(int i) {
            while (i > 0) {
                int p = (i - 1) >> 1;
                if (data[i] > data[p]) {
                    long tmp = data[i]; data[i] = data[p]; data[p] = tmp;
                    i = p;
                } else break;
            }
        }

        private void Down(int i) {
            while ((i << 1) + 1 < count) {
                int l = (i << 1) + 1, r = l + 1, best = l;
                if (r < count && data[r] > data[l]) best = r;
                if (data[best] > data[i]) {
                    long tmp = data[i]; data[i] = data[best]; data[best] = tmp;
                    i = best;
                } else break;
            }
        }
    }

    class MinHeap {
        private readonly long[] data;
        private int count;

        public MinHeap(int capacity) {
            data = new long[capacity];
            count = 0;
        }

        public int Count { get { return count; } }

        public void Push(long val) {
            data[count] = val;
            Up(count++);
        }

        public long Pop() {
            long top = data[0];
            data[0] = data[--count];
            Down(0);
            return top;
        }

        public long Peek() { return data[0]; }

        private void Up(int i) {
            while (i > 0) {
                int p = (i - 1) >> 1;
                if (data[i] < data[p]) {
                    long tmp = data[i]; data[i] = data[p]; data[p] = tmp;
                    i = p;
                } else break;
            }
        }

        private void Down(int i) {
            while ((i << 1) + 1 < count) {
                int l = (i << 1) + 1, r = l + 1, best = l;
                if (r < count && data[r] < data[l]) best = r;
                if (data[best] < data[i]) {
                    long tmp = data[i]; data[i] = data[best]; data[best] = tmp;
                    i = best;
                } else break;
            }
        }
    }

    class FastScanner {
        private readonly TextReader reader;
        private string[] tokens;
        private int ptr;

        public FastScanner(TextReader reader) {
            this.reader = reader;
            this.tokens = new string[0];
            this.ptr = 0;
        }

        public string Next() {
            while (ptr >= tokens.Length) {
                string line = reader.ReadLine();
                if (line == null) return null;
                tokens = line.Split(new char[] { ' ', '\t' }, StringSplitOptions.RemoveEmptyEntries);
                ptr = 0;
            }
            return tokens[ptr++];
        }

        public int NextInt() {
            string s = Next();
            return s == null ? 0 : int.Parse(s);
        }

        public long NextLong() {
            string s = Next();
            return s == null ? 0 : long.Parse(s);
        }
    }

    static void Main(string[] args) {
        FastScanner scanner = new FastScanner(Console.In);
        string nStr = scanner.Next();
        if (nStr == null) return;
        int n = int.Parse(nStr);
        int q = scanner.NextInt();

        MaxHeap L = new MaxHeap(2 * n + 5);
        MinHeap R = new MinHeap(2 * n + 5);

        long shiftL = 0;
        long shiftR = 0;
        long minVal = 0;

        long lo1 = scanner.NextLong();
        long hi1 = scanner.NextLong();
        L.Push(lo1);
        R.Push(hi1);

        for (int i = 1; i < n; i++) {
            long lo = scanner.NextLong();
            long hi = scanner.NextLong();
            long up = scanner.NextLong();
            long down = scanner.NextLong();

            shiftL -= down;
            shiftR += up;

            long optL = L.Peek() + shiftL;
            long optR = R.Peek() + shiftR;

            if (hi < optL) {
                minVal += (optL - hi);
                L.Pop();
                R.Push(optL - shiftR);
                L.Push(lo - shiftL);
                L.Push(hi - shiftL);
            } else if (lo > optR) {
                minVal += (lo - optR);
                R.Pop();
                L.Push(optR - shiftL);
                R.Push(lo - shiftR);
                R.Push(hi - shiftR);
            } else {
                L.Push(lo - shiftL);
                R.Push(hi - shiftR);
            }
        }

        long minX = L.Peek() + shiftL;

        int nL = L.Count;
        long[] leftPoints = new long[nL];
        for (int i = nL - 1; i >= 0; i--) {
            leftPoints[i] = L.Pop() + shiftL;
        }

        int nR = R.Count;
        long[] rightPoints = new long[nR];
        for (int i = 0; i < nR; i++) {
            rightPoints[i] = R.Pop() + shiftR;
        }

        long[] prefL = new long[nL + 1];
        for (int i = 0; i < nL; i++) prefL[i + 1] = prefL[i] + leftPoints[i];

        long[] prefR = new long[nR + 1];
        for (int i = 0; i < nR; i++) prefR[i + 1] = prefR[i] + rightPoints[i];

        long optLFinal = leftPoints[nL - 1];
        long optRFinal = rightPoints[0];

        StringBuilder sb = new StringBuilder();
        sb.AppendLine(minVal + " " + minX);

        for (int k = 0; k < q; k++) {
            long z = scanner.NextLong();
            if (z >= optLFinal && z <= optRFinal) {
                sb.AppendLine(minVal.ToString());
            } else if (z < optLFinal) {
                int low = 0, high = nL;
                while (low < high) {
                    int mid = (low + high) >> 1;
                    if (leftPoints[mid] > z) high = mid;
                    else low = mid + 1;
                }
                long cnt = nL - low;
                long sum = prefL[nL] - prefL[low];
                sb.AppendLine((minVal + (sum - cnt * z)).ToString());
            } else {
                int low = 0, high = nR;
                while (low < high) {
                    int mid = (low + high) >> 1;
                    if (rightPoints[mid] < z) low = mid + 1;
                    else high = mid;
                }
                long cnt = low;
                long sum = prefR[low];
                sb.AppendLine((minVal + (cnt * z - sum)).ToString());
            }
        }

        Console.Write(sb.ToString());
    }
}
```

---

### JavaScript (Node.js 24.4.1)

```javascript
const fs = require('fs');

class MinHeap {
    constructor() { this.data = []; }
    push(val) {
        this.data.push(val);
        this._up(this.data.length - 1);
    }
    pop() {
        const top = this.data[0];
        const bottom = this.data.pop();
        if (this.data.length > 0) {
            this.data[0] = bottom;
            this._down(0);
        }
        return top;
    }
    peek() { return this.data[0]; }
    size() { return this.data.length; }
    _up(i) {
        while (i > 0) {
            let p = (i - 1) >> 1;
            if (this.data[i] < this.data[p]) {
                [this.data[i], this.data[p]] = [this.data[p], this.data[i]];
                i = p;
            } else break;
        }
    }
    _down(i) {
        const len = this.data.length;
        while ((i << 1) + 1 < len) {
            let l = (i << 1) + 1, r = l + 1, best = l;
            if (r < len && this.data[r] < this.data[l]) best = r;
            if (this.data[best] < this.data[i]) {
                [this.data[i], this.data[best]] = [this.data[best], this.data[i]];
                i = best;
            } else break;
        }
    }
}

class MaxHeap {
    constructor() { this.data = []; }
    push(val) {
        this.data.push(val);
        this._up(this.data.length - 1);
    }
    pop() {
        const top = this.data[0];
        const bottom = this.data.pop();
        if (this.data.length > 0) {
            this.data[0] = bottom;
            this._down(0);
        }
        return top;
    }
    peek() { return this.data[0]; }
    size() { return this.data.length; }
    _up(i) {
        while (i > 0) {
            let p = (i - 1) >> 1;
            if (this.data[i] > this.data[p]) {
                [this.data[i], this.data[p]] = [this.data[p], this.data[i]];
                i = p;
            } else break;
        }
    }
    _down(i) {
        const len = this.data.length;
        while ((i << 1) + 1 < len) {
            let l = (i << 1) + 1, r = l + 1, best = l;
            if (r < len && this.data[r] > this.data[l]) best = r;
            if (this.data[best] > this.data[i]) {
                [this.data[i], this.data[best]] = [this.data[best], this.data[i]];
                i = best;
            } else break;
        }
    }
}

function processData(input) {
    if (!input) return;
    let pos = 0;
    const len = input.length;

    function nextToken() {
        while (pos < len && input.charCodeAt(pos) <= 32) pos++;
        if (pos >= len) return null;
        const start = pos;
        while (pos < len && input.charCodeAt(pos) > 32) pos++;
        return input.substring(start, pos);
    }

    const nToken = nextToken();
    if (!nToken) return;
    const n = parseInt(nToken, 10);
    const q = parseInt(nextToken(), 10);

    const L = new MaxHeap();
    const R = new MinHeap();

    let shiftL = 0n;
    let shiftR = 0n;
    let minVal = 0n;

    const lo1 = BigInt(nextToken());
    const hi1 = BigInt(nextToken());
    L.push(lo1);
    R.push(hi1);

    for (let i = 1; i < n; i++) {
        const lo = BigInt(nextToken());
        const hi = BigInt(nextToken());
        const up = BigInt(nextToken());
        const down = BigInt(nextToken());

        shiftL -= down;
        shiftR += up;

        const optL = L.peek() + shiftL;
        const optR = R.peek() + shiftR;

        if (hi < optL) {
            minVal += (optL - hi);
            L.pop();
            R.push(optL - shiftR);
            L.push(lo - shiftL);
            L.push(hi - shiftL);
        } else if (lo > optR) {
            minVal += (lo - optR);
            R.pop();
            L.push(optR - shiftL);
            R.push(lo - shiftR);
            R.push(hi - shiftR);
        } else {
            L.push(lo - shiftL);
            R.push(hi - shiftR);
        }
    }

    const minX = L.peek() + shiftL;

    const leftPoints = [];
    while (L.size() > 0) leftPoints.push(L.pop() + shiftL);
    leftPoints.reverse(); // ascending

    const rightPoints = [];
    while (R.size() > 0) rightPoints.push(R.pop() + shiftR); // ascending

    const nL = leftPoints.length;
    const prefL = new Array(nL + 1).fill(0n);
    for (let i = 0; i < nL; i++) prefL[i + 1] = prefL[i] + leftPoints[i];

    const nR = rightPoints.length;
    const prefR = new Array(nR + 1).fill(0n);
    for (let i = 0; i < nR; i++) prefR[i + 1] = prefR[i] + rightPoints[i];

    const optLFinal = leftPoints[nL - 1];
    const optRFinal = rightPoints[0];

    const out = [];
    out.push(minVal.toString() + " " + minX.toString());

    for (let k = 0; k < q; k++) {
        const z = BigInt(nextToken());
        if (z >= optLFinal && z <= optRFinal) {
            out.push(minVal.toString());
        } else if (z < optLFinal) {
            let low = 0, high = nL;
            while (low < high) {
                const mid = (low + high) >> 1;
                if (leftPoints[mid] > z) high = mid;
                else low = mid + 1;
            }
            const cnt = BigInt(nL - low);
            const sum = prefL[nL] - prefL[low];
            out.push((minVal + (sum - cnt * z)).toString());
        } else {
            let low = 0, high = nR;
            while (low < high) {
                const mid = (low + high) >> 1;
                if (rightPoints[mid] < z) low = mid + 1;
                else high = mid;
            }
            const cnt = BigInt(low);
            const sum = prefR[low];
            out.push((minVal + (cnt * z - sum)).toString());
        }
    }

    console.log(out.join("\n"));
}

process.stdin.resume();
process.stdin.setEncoding("utf-8");
let _input = "";
process.stdin.on("data", chunk => { _input += chunk; });
process.stdin.on("end", () => { processData(_input); });
```

---

## 8. Complexity Analysis Summary

| Aspect | Metric | Details |
| :--- | :--- | :--- |
| **Rounds Processing** | $\mathcal{O}(n \log n)$ | Each round performs at most 4 heap insertions/deletions. Total operations $\le 4n \log_2(2n) \approx 1.5 \times 10^7$. |
| **Heap Extraction & Prefix Sums** | $\mathcal{O}(n \log n)$ | Extracting $n$ points from each heap and building prefix sums in linear time. |
| **Queries Processing** | $\mathcal{O}(q \log n)$ | Each shutdown plan $z$ uses binary search across $n$ sorted points in $\log_2(n) \approx 18$ iterations. |
| **Total Time Complexity** | $\mathbf{\mathcal{O}((n + q) \log n)}$ | Runs well within $0.2\text{s}$ in C/C++, $< 0.8\text{s}$ in Java/C#, and $< 1.2\text{s}$ in Python/JS. |
| **Total Space Complexity** | $\mathbf{\mathcal{O}(n)}$ | Dual binary heaps and sorted point arrays consume $\approx 8\text{ MB}$ RAM. |

---

## 9. Edge Cases & Interview Insights

1. **Zero Motor Movement Limits ($up = 0, down = 0$):**
   - The shifts remain `shift_L = 0, shift_R = 0`.
   - The carriage coordinate remains static across all rounds, reducing the problem to finding the 1D weighted median / minimum distance to a union of intervals. Handled naturally without branching.

2. **All Sweet Bands Identical ($[lo_i, hi_i] = [C, C]$):**
   - All inflection points collapse onto $C$. The flat minimum region has width 0, and heaps accumulate duplicate points cleanly.

3. **Massive Coordinate Offsets ($10^9$):**
   - Accumulations reach values up to $200\,000 \times 10^9 = 2 \times 10^{14}$, safely stored in 64-bit signed integers.
