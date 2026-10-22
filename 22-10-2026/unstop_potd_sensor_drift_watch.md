# Unstop Problem of the Day: Sensor Drift Watch

- **Platform:** [Unstop](https://unstop.com/)
- **Difficulty:** Medium
- **Topic Tags:** Fenwick Tree (Binary Indexed Tree), Coordinate Compression, Dynamic Rank Queries, Hash Map, Streaming Algorithms, Order Statistics
- **Target Audience:** POTD Solvers, Computer Science Students, Competitive Programmers, Technical Interview Preparation (FAANG / Tier-1 Systems)

---

## 1. Problem Statement

Dev oversees a regional weather monitoring network constructed from sensor stations scattered across the countryside. Station identifiers are assigned by the equipment vendor and are neither small nor sequential ($1 \le \text{id} \le 10^9$). The stations report readings at irregular intervals throughout the day. Every report names a station and its newly measured temperature, replacing whatever that station last reported.

### The Drift Event Rule
Occasionally, a station's new temperature reading jumps sharply away from its own previous reading. This usually signals either an abrupt weather front or a sensor glitch worth flagging. Dev defines a report as a **drift event** whenever:
1. The station has **reported at least once before** (first-time reports are never drift events, as there is no historical reading to compare against).
2. Its new reading differs from its own immediately preceding reading by **strictly more than** a fixed tolerance $D$:
   $$|\text{temp}_{\text{new}} - \text{temp}_{\text{prev}}| > D$$

### The Network Query Specification
Whenever a drift event occurs, Dev wants to immediately quantify how unusual the new reading is relative to the current state of the entire network at that exact moment. Specifically:
- **Query Objective:** Count how many **other** currently active stations have a latest reading **strictly below** this new temperature:
  $$\text{Count} = \big| \{ s \in \text{Active Stations} \setminus \{\text{drifting station}\} \mid \text{latest\_temp}[s] < \text{temp}_{\text{new}} \} \big|$$
- **Pre-update Exclusion Principle:** When computing this count, the drifting station's own outdated reading ($\text{temp}_{\text{prev}}$) must be set aside first.
- **State Transition:** Following the query, the station's latest reading in the active network is updated to $\text{temp}_{\text{new}}$.

Because reports arrive in a continuous stream ($q \le 2 \times 10^5$), Dev needs every drift event resolved instantly as it occurs, maintaining a running picture of "most recent reading per station" under continuous mutation.

---

## 2. Input & Output Formats

### Input Format
- **Line 1:** Two space-separated integers:
  $$q \quad D$$
  where $q$ is the total number of sensor reports and $D$ is the drift tolerance ($0 \le D \le 2 \times 10^9$).
- **Next $q$ lines:** Each line contains two space-separated integers:
  $$\text{id} \quad \text{temp}$$
  representing a station identifier ($1 \le \text{id} \le 10^9$) and its newly measured temperature ($-10^9 \le \text{temp} \le 10^9$) in chronological order.

### Output Format
For every drift event, in the exact chronological order it occurs, print a line containing three space-separated integers:
$$\text{id} \quad \text{temp} \quad \text{count}$$
where:
- `id` is the drifting station identifier.
- `temp` is the newly reported temperature.
- `count` is the number of other currently reporting stations whose latest reading is strictly below `temp`.

### Constraints
- $1 \le q \le 2 \times 10^5$
- $0 \le D \le 2 \times 10^9$
- $1 \le \text{id} \le 10^9$
- $-10^9 \le \text{temp} \le 10^9$

---

## 3. Examples & Detailed Explanations

### Example 1 (Sample Testcase 0)

**Input:**
```text
5 5
1 10
2 20
1 17
3 5
2 8
```

**Step-by-Step Execution Trace ($D = 5$):**

1. **Report 1: `1 10`**
   - Station `1` has never reported before $\implies$ **Not a drift event**.
   - Network State: Station `1` registered with temperature $10$.
   - Active Multiset: $\{10\}$.

2. **Report 2: `2 20`**
   - Station `2` has never reported before $\implies$ **Not a drift event**.
   - Network State: Station `2` registered with temperature $20$.
   - Active Multiset: $\{10, 20\}$.

3. **Report 3: `1 17`**
   - Station `1` previous reading = $10$.
   - Absolute Jump: $|17 - 10| = 7$.
   - Check Drift: $7 > 5$ (Tolerance $D$) $\implies$ **DRIFT EVENT!**
   - **Exclusion:** Temporarily remove Station `1`'s old reading ($10$). Active other readings: $\{20\}$.
   - **Query:** How many other active readings are strictly below $17$?
     - Reading $20 < 17$ is False. Count = **0**.
   - **Output:** `1 17 0`
   - **Update:** Insert new reading $17$ for Station `1`.
   - Active Multiset: $\{17, 20\}$.

4. **Report 4: `3 5`**
   - Station `3` has never reported before $\implies$ **Not a drift event**.
   - Network State: Station `3` registered with temperature $5$.
   - Active Multiset: $\{5, 17, 20\}$.

5. **Report 5: `2 8`**
   - Station `2` previous reading = $20$.
   - Absolute Jump: $|8 - 20| = 12$.
   - Check Drift: $12 > 5$ (Tolerance $D$) $\implies$ **DRIFT EVENT!**
   - **Exclusion:** Temporarily remove Station `2`'s old reading ($20$). Active other readings: $\{5, 17\}$.
   - **Query:** How many other active readings are strictly below $8$?
     - $5 < 8 \implies$ True.
     - $17 < 8 \implies$ False.
     - Count = **1**.
   - **Output:** `2 8 1`
   - **Update:** Insert new reading $8$ for Station `2`.
   - Active Multiset: $\{5, 8, 17\}$.

**Final Output:**
```text
1 17 0
2 8 1
```

---

### Example 2 (Sample Testcase 1)

**Input:**
```text
5 3
1 0
1 1
1 10
2 2
2 2
```

**Step-by-Step Execution Trace ($D = 3$):**

1. **Report 1: `1 0`**
   - Station `1` first report $\implies$ Not a drift event. Active: $\{0\}$.
2. **Report 2: `1 1`**
   - Station `1` prev = $0$. Jump: $|1 - 0| = 1 \le 3 \implies$ **No drift event**.
   - Old reading $0$ is replaced by $1$. Active: $\{1\}$.
3. **Report 3: `1 10`**
   - Station `1` prev = $1$. Jump: $|10 - 1| = 9 > 3 \implies$ **DRIFT EVENT!**
   - Exclude old reading $1$. Other active stations: $\emptyset$.
   - Count of readings $< 10$ among empty set: **0**.
   - **Output:** `1 10 0`
   - Update Station `1` to $10$. Active: $\{10\}$.
4. **Report 4: `2 2`**
   - Station `2` first report $\implies$ Not a drift event. Active: $\{2, 10\}$.
5. **Report 5: `2 2`**
   - Station `2` prev = $2$. Jump: $|2 - 2| = 0 \le 3 \implies$ Not a drift event.
   - Active remains $\{2, 10\}$.

**Final Output:**
```text
1 10 0
```

---

## 4. Mathematical Invariants & Theoretical Foundations

```
                           +---------------------------+
                           |  New Report: (id, temp)   |
                           +-------------+-------------+
                                         |
                                         v
                         /-------------------------------\
                        <   Has Station 'id' reported?   >
                         \-------------------------------/
                                    /         \
                              NO   /           \  YES
                                  v             v
                    +--------------------+   +------------------------------------+
                    | First-Time Report  |   | Compute: diff = |temp - prev_temp| |
                    | (Never a drift)    |   +-----------------+------------------+
                    +---------+----------+                     |
                              |                  /-----------------------------\
                              |                 <      Is diff > Tolerance D?   >
                              |                  \-----------------------------/
                              |                             /         \
                              |                       NO   /           \  YES
                              |                           v             v
                              |              +----------------+   +-------------------------------+
                              |              | Normal Update: |   | DRIFT EVENT FLAGGED!          |
                              |              | Remove prev    |   | 1. Remove prev from BIT       |
                              |              | Add temp       |   | 2. Query count strictly < temp|
                              |              +-------+--------+   | 3. Output: id temp count      |
                              |                      |            | 4. Add temp to BIT            |
                              |                      |            +---------------+---------------+
                              v                      v                            v
                           +---------------------------------------------------------+
                           | Update Station Table: prev_temp[id] = temp              |
                           +---------------------------------------------------------+
```

### Invariant 1: 64-Bit Arithmetic Safety
The temperature values lie in $[-10^9, 10^9]$ and tolerance $D \in [0, 2 \times 10^9]$.
- The absolute difference $|\text{temp}_{\text{new}} - \text{temp}_{\text{prev}}|$ can reach:
  $$10^9 - (-10^9) = 2 \times 10^9$$
- In standard 32-bit signed integers, subtracting $-10^9$ from $10^9$ can trigger arithmetic overflow if intermediate values exceed $2^{31}-1 \approx 2.147 \times 10^9$.
- **Theorem:** All differences and tolerance comparisons MUST be evaluated using **64-bit signed integers** (`long long` in C/C++, `long` in Java/C#, `int64`).

### Invariant 2: Dynamic Multiset Rank Problem
At any moment, we maintain a multiset $\mathcal{S}$ of size at most $q$. We require:
1. `Insert(x)`: Add temperature $x$ to $\mathcal{S}$.
2. `Delete(x)`: Remove one instance of temperature $x$ from $\mathcal{S}$.
3. `CountStrictlyLessThan(x)`: Return $|\{ y \in \mathcal{S} \mid y < x \}|$.

Because all queries occur online over an arbitrary stream of numbers in $[-10^9, 10^9]$, a static direct array of size $2 \times 10^9$ is impossible ($\approx 8 \text{ GB}$). However, the total number of distinct temperature values appearing in all $q$ reports cannot exceed $q \le 2 \times 10^5$.

### Invariant 3: Coordinate Compression & Fenwick Tree Order Statistics
By pre-collecting all $q$ temperatures, sorting them, and discarding duplicates, we map each unique temperature to an integer rank:
$$\text{Rank}(t) \in [1, M], \quad \text{where } M \le q \le 2 \times 10^5$$

Let a Fenwick Tree (Binary Indexed Tree) of size $M$ track the frequency of active temperatures at each rank:
- **Insert $t$:** $\text{BIT.add}(\text{Rank}(t), +1)$ in $\mathcal{O}(\log M)$ time.
- **Delete $t$:** $\text{BIT.add}(\text{Rank}(t), -1)$ in $\mathcal{O}(\log M)$ time.
- **Count Strictly Less Than $t$:**
  Because the compression preserves strict inequality ($a < b \iff \text{Rank}(a) < \text{Rank}(b)$):
  $$\text{Count}(< t) = \sum_{r = 1}^{\text{Rank}(t) - 1} \text{Freq}[r] = \text{BIT.query}(\text{Rank}(t) - 1)$$
  which executes in $\mathcal{O}(\log M)$ time!

---

## 5. System Architecture & Visual Diagram

### Dual-Layer Dynamic Architecture

```
Reports Stream: (id, temp)
    │
    ▼
┌────────────────────────────────────────────────────────┐
│               Hash Map / Station Index                 │
│         id -> last_known_temperature_rank              │
│                                                        │
│   Station 1: Rank(17) ──┐                              │
│   Station 2: Rank(8)  ──┼── Keeps track of which rank  │
│   Station 3: Rank(5)  ──┘   each active station holds  │
└────────────────────────────────────────────────────────┘
    │
    │  (Insert / Delete operations on Rank values)
    ▼
┌──────────────────────────────────────────────────────────────────┐
│                   Fenwick Tree (BIT Array)                       │
│                                                                  │
│  Rank:     1       2       3       4             M               │
│  Temp:   [-40°C] [ -5°C] [ +5°C] [ +8°C] ...  [ +50°C]           │
│  Freq:   [  0  ] [  1  ] [  1  ] [  1  ] ...  [  0  ]            │
│                                                                  │
│  Prefix Sum: query(Rank - 1) calculates exact count < Temp       │
│  All updates and prefix queries run in O(log M) operations!      │
└──────────────────────────────────────────────────────────────────┘
```

---

## 6. Step-by-Step Simulation Trace Table

Tracing Sample Testcase 0 ($D = 5$):
- Temperatures observed: $[10, 20, 17, 5, 8]$
- Unique Sorted Temperatures: $[5, 8, 10, 17, 20]$
- Rank Map:
  - $5 \to 1$
  - $8 \to 2$
  - $10 \to 3$
  - $17 \to 4$
  - $20 \to 5$

| Step | Report $(id, temp)$ | Previous Temp | Jump $\|\Delta\|$ | Drift? ($> 5$) | BIT Removal | BIT Query ($< temp$) | BIT Addition | Active BIT Counts $[r_1..r_5]$ | Output |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **1** | `1, 10` | None | — | **No** (1st time) | — | — | Add $r_3 (10)$ | $[0, 0, 1, 0, 0]$ | — |
| **2** | `2, 20` | None | — | **No** (1st time) | — | — | Add $r_5 (20)$ | $[0, 0, 1, 0, 1]$ | — |
| **3** | `1, 17` | $10$ | $\|17 - 10\| = 7$ | **YES** | Del $r_3 (10)$ | Query $(< 17) \implies \text{query}(r_3)$ on $[0,0,0,0,1] = 0$ | Add $r_4 (17)$ | $[0, 0, 0, 1, 1]$ | `1 17 0` |
| **4** | `3, 5` | None | — | **No** (1st time) | — | — | Add $r_1 (5)$ | $[1, 0, 0, 1, 1]$ | — |
| **5** | `2, 8` | $20$ | $\|8 - 20\| = 12$ | **YES** | Del $r_5 (20)$ | Query $(< 8) \implies \text{query}(r_1)$ on $[1,0,0,1,0] = 1$ | Add $r_2 (8)$ | $[1, 1, 0, 1, 0]$ | `2 8 1` |

---

## 7. Mermaid Flowchart

```mermaid
flowchart TD
    Start([Start Streaming Sensor Reports]) --> PreProcess[Pre-process & Coordinate Compress all Temperatures]
    PreProcess --> InitDS[Initialize Fenwick Tree & Station Map]
    InitDS --> LoopReports{Has More Reports?}
    
    LoopReports -- No --> EndProg([Finish Execution])
    LoopReports -- Yes --> ReadReport[Read Station ID & New Temperature]
    
    ReadReport --> CheckSeen{Has Station Reported Before?}
    CheckSeen -- No --> RegFirst[Register Station in Map<br/>Add New Temp to Fenwick Tree]
    RegFirst --> LoopReports
    
    CheckSeen -- Yes --> CalcJump[Calculate Absolute Jump<br/>diff = |new_temp - prev_temp|]
    CalcJump --> CheckDrift{Is diff > D?}
    
    CheckDrift -- No --> NormalUpdate[Remove prev_temp from Fenwick Tree<br/>Add new_temp to Fenwick Tree<br/>Update Station Map]
    NormalUpdate --> LoopReports
    
    CheckDrift -- Yes --> DriftAction[1. Remove prev_temp from Fenwick Tree<br/>2. Query Fenwick Tree for count strictly < new_temp<br/>3. Print: ID new_temp count<br/>4. Add new_temp to Fenwick Tree<br/>5. Update Station Map]
    DriftAction --> LoopReports
```

---

## 8. Algorithmic Approaches: From Naive to Pro

### Approach 1: Naive Dynamic Array Scanning (Brute Force)
- **Concept:** Store station readings in an unordered dictionary or hash table `stations[id] = temp`. On a drift event, iterate over all other $(s, t)$ pairs currently in the dictionary and count how many have $t < \text{new\_temp}$.
- **Time Complexity:**
  - Update: $\mathcal{O}(1)$ average.
  - Drift Query: $\mathcal{O}(N_{\text{active}}) = \mathcal{O}(q)$ in the worst case.
  - Overall Time: $\mathcal{O}(q^2)$.
- **Limitation:** For $q = 2 \times 10^5$, $q^2 = 4 \times 10^{10}$ operations, resulting in a severe **Time Limit Exceeded (TLE)**.

```python
# Naive Pseudo-code (TLE)
stations = {}
for id, temp in reports:
    if id in stations:
        prev = stations[id]
        if abs(temp - prev) > D:
            # Linear scan over all active stations
            cnt = sum(1 for sid, t in stations.items() if sid != id and t < temp)
            print(f"{id} {temp} {cnt}")
    stations[id] = temp
```

---

### Approach 2: Balanced Binary Search Tree / Policy-Based PBDS
- **Concept:** Maintain an Order Statistic Tree (like GNU C++ PBDS `tree<pair<long long, int>, ...>`) storing pairs `(temp, station_id)` to handle duplicate temperatures.
- **Operations:**
  - Deletion: $\mathcal{O}(\log N)$.
  - Rank Query (`order_of_key({temp, -1})`): $\mathcal{O}(\log N)$.
  - Insertion: $\mathcal{O}(\log N)$.
- **Limitation:** While asymptotically $\mathcal{O}(q \log q)$, PBDS is specific to GCC C++ and tree rebalancing involves heavy pointer overhead and constant-factor memory penalties in languages like Java (`TreeMap` does not provide $\mathcal{O}(\log N)$ order statistics without augmenting subtree sizes).

---

### Approach 3: Coordinate Compression + Fenwick Tree (Pro / Optimal)
- **Concept:**
  1. Offline compression of all temperatures into contiguous ranks $1 \dots M$ where $M \le q$.
  2. A Fenwick Tree (Binary Indexed Tree) stores frequencies of temperatures across ranks.
  3. Every temperature insertion and deletion is a single point update `add(rank, ±1)` taking $\mathcal{O}(\log M)$ operations with pure cache-friendly array indexing.
  4. Querying temperatures strictly less than `temp` is `query(rank - 1)`, which computes the exact prefix sum in $\mathcal{O}(\log M)$ time.
- **Station ID Resolution:**
  - In languages with fast hash maps (Java, Python, C++, C#, JS), map `id -> prev_temp`.
  - In C, coordinate-compress `id` values as well to obtain direct static array lookups with zero collisions!
- **Time Complexity:** $\mathcal{O}(q \log q)$ time.
- **Space Complexity:** $\mathcal{O}(q)$ auxiliary memory.

---

## 9. Complexity Comparison Matrix

| Approach | Preprocessing Time | Per-Query Time | Total Time ($q = 2 \cdot 10^5$) | Auxiliary Space | Cache Friendliness | Verdict |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **Approach 1: Naive Scan** | $\mathcal{O}(1)$ | $\mathcal{O}(q)$ | $\mathcal{O}(q^2) \approx 4 \times 10^{10}$ ops | $\mathcal{O}(q)$ | Poor | **TLE** |
| **Approach 2: Augmented BST / PBDS** | $\mathcal{O}(1)$ | $\mathcal{O}(\log q)$ | $\mathcal{O}(q \log q) \approx 3.6 \times 10^6$ ops | $\mathcal{O}(q)$ | Moderate (Node pointers) | **Pass** (C++ only) |
| **Approach 3: Coordinate Compression + BIT** | $\mathcal{O}(q \log q)$ | $\mathcal{O}(\log q)$ | $\mathcal{O}(q \log q) \approx 3.6 \times 10^6$ ops | $\mathcal{O}(q)$ | **High (Flat Arrays)** | **Optimal (All Languages)** |

---

## 10. Corner Cases & Defensive Engineering

1. **Station IDs up to $10^9$:** Station IDs cannot be used directly as array indices. Use hash maps or secondary coordinate compression.
2. **Negative Temperatures:** Temperatures can be negative (down to $-10^9$). Coordinate compression gracefully handles negative numbers because sorting naturally orders negative values before positive ones.
3. **Identical Consecutive Readings:** A station reporting the exact same temperature back-to-back ($|temp - prev| = 0 \le D$) is never a drift event, but its state must be maintained consistently.
4. **Duplicate Temperatures across Multiple Stations:** Multiple stations can have the exact same temperature simultaneously. The Fenwick Tree maintains a frequency count (multiplicity) at each rank, seamlessly handling identical readings.
5. **No Previous Report:** Stations reporting for the first time must simply be recorded and inserted without drift evaluation.
6. **Large Lookahead / Tolerance $D = 0$:** When $D = 0$, any report that changes the temperature ($|\Delta| > 0$) is classified as a drift event.
7. **Fast I/O Requirement:** With $q = 2 \times 10^5$, standard console I/O (`cin`/`cout` without synchronization, `Scanner`, `Console.WriteLine` in loops) will bottleneck. Buffered I/O is mandatory.

---

## 11. Multi-Language Source Code Implementations

### Java (OpenJDK 21.0)

```java
import java.io.*;
import java.util.*;

public class Main {

    // Fast I/O Reader
    static class FastReader {
        private final InputStream in;
        private final byte[] buffer = new byte[1 << 16];
        private int ptr = 0, len = 0;

        public FastReader(InputStream in) {
            this.in = in;
        }

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

        public long nextLong() {
            int b = readByte();
            while (b <= ' ' && b != -1) b = readByte();
            if (b == -1) return Long.MIN_VALUE;
            boolean neg = false;
            if (b == '-') {
                neg = true;
                b = readByte();
            }
            long res = 0;
            while (b >= '0' && b <= '9') {
                res = res * 10 + (b - '0');
                b = readByte();
            }
            return neg ? -res : res;
        }

        public int nextInt() {
            return (int) nextLong();
        }
    }

    // Binary Indexed Tree (Fenwick Tree)
    static class FenwickTree {
        private final int size;
        private final int[] tree;

        public FenwickTree(int size) {
            this.size = size;
            this.tree = new int[size + 2];
        }

        public void add(int i, int delta) {
            for (; i <= size; i += i & -i) {
                tree[i] += delta;
            }
        }

        public int query(int i) {
            int sum = 0;
            for (; i > 0; i -= i & -i) {
                sum += tree[i];
            }
            return sum;
        }
    }

    public static void main(String[] args) throws IOException {
        FastReader scanner = new FastReader(System.in);
        int q = scanner.nextInt();
        if (q <= 0) return;
        long D = scanner.nextLong();

        long[] ids = new long[q];
        long[] temps = new long[q];
        long[] sortedTemps = new long[q];

        for (int i = 0; i < q; i++) {
            ids[i] = scanner.nextLong();
            temps[i] = scanner.nextLong();
            sortedTemps[i] = temps[i];
        }

        // Coordinate Compression for Temperatures
        Arrays.sort(sortedTemps);
        int uniqueCount = 0;
        for (int i = 0; i < q; i++) {
            if (i == 0 || sortedTemps[i] != sortedTemps[i - 1]) {
                sortedTemps[uniqueCount++] = sortedTemps[i];
            }
        }

        FenwickTree bit = new FenwickTree(uniqueCount);
        Map<Long, Long> stationLatest = new HashMap<>(q);
        StringBuilder out = new StringBuilder();

        for (int i = 0; i < q; i++) {
            long stationId = ids[i];
            long currentTemp = temps[i];
            int currentRank = Arrays.binarySearch(sortedTemps, 0, uniqueCount, currentTemp) + 1;

            if (stationLatest.containsKey(stationId)) {
                long prevTemp = stationLatest.get(stationId);
                int prevRank = Arrays.binarySearch(sortedTemps, 0, uniqueCount, prevTemp) + 1;
                long diff = Math.abs(currentTemp - prevTemp);

                if (diff > D) {
                    // Drift Event!
                    // 1. Set aside the station's own outdated reading
                    bit.add(prevRank, -1);

                    // 2. Query count of active readings strictly below currentTemp
                    int countBelow = bit.query(currentRank - 1);

                    // 3. Output the drift event
                    out.append(stationId).append(' ')
                       .append(currentTemp).append(' ')
                       .append(countBelow).append('\n');

                    // 4. Insert new temperature
                    bit.add(currentRank, +1);
                } else {
                    // Not a drift event, but replace the reading in the active network
                    bit.add(prevRank, -1);
                    bit.add(currentRank, +1);
                }
            } else {
                // First-time report: never a drift event
                bit.add(currentRank, +1);
            }

            // Update latest known temperature for this station
            stationLatest.put(stationId, currentTemp);
        }

        System.out.print(out);
    }
}
```

---

### Python (3.12.11)

```python
import sys
from bisect import bisect_left

def main():
    # Read all input from standard input efficiently
    input_data = sys.stdin.read().split()
    if not input_data:
        return

    q = int(input_data[0])
    D = int(input_data[1])

    reports = []
    idx = 2
    all_temps = []
    for _ in range(q):
        station_id = int(input_data[idx])
        temp = int(input_data[idx + 1])
        reports.append((station_id, temp))
        all_temps.append(temp)
        idx += 2

    # Coordinate Compression on unique temperatures
    unique_temps = sorted(list(set(all_temps)))
    total_unique = len(unique_temps)

    # Fenwick Tree (1-based index)
    bit = [0] * (total_unique + 2)

    def bit_add(r, delta):
        while r <= total_unique:
            bit[r] += delta
            r += r & -r

    def bit_query(r):
        total = 0
        while r > 0:
            total += bit[r]
            r -= r & -r
        return total

    def get_rank(t):
        return bisect_left(unique_temps, t) + 1

    station_latest = {}
    output = []

    for station_id, current_temp in reports:
        curr_rank = get_rank(current_temp)

        if station_id in station_latest:
            prev_temp = station_latest[station_id]
            prev_rank = get_rank(prev_temp)
            diff = abs(current_temp - prev_temp)

            if diff > D:
                # Drift Event!
                # 1. Remove previous reading from active pool
                bit_add(prev_rank, -1)

                # 2. Query count strictly below current_temp
                count_below = bit_query(curr_rank - 1)

                # 3. Record drift event
                output.append(f"{station_id} {current_temp} {count_below}")

                # 4. Add new reading
                bit_add(curr_rank, 1)
            else:
                # Not a drift, simply update temperature in active pool
                bit_add(prev_rank, -1)
                bit_add(curr_rank, 1)
        else:
            # First-time report: register in active pool
            bit_add(curr_rank, 1)

        station_latest[station_id] = current_temp

    if output:
        sys.stdout.write("\n".join(output) + "\n")

if __name__ == '__main__':
    main()
```

---

### C (GCC 13.2.0)

```c
#include <stdio.h>
#include <stdlib.h>
#include <math.h>

// Binary Indexed Tree functions
static inline void bit_add(int *bit, int size, int idx, int val) {
    for (; idx <= size; idx += idx & -idx) {
        bit[idx] += val;
    }
}

static inline int bit_query(const int *bit, int idx) {
    int sum = 0;
    for (; idx > 0; idx -= idx & -idx) {
        sum += bit[idx];
    }
    return sum;
}

// Comparator for 64-bit integers
static int cmp_long(const void *a, const void *b) {
    long long va = *(const long long *)a;
    long long vb = *(const long long *)b;
    if (va < vb) return -1;
    if (va > vb) return 1;
    return 0;
}

// Binary search to find 1-based rank in sorted array
static int find_rank(const long long *arr, int n, long long val) {
    int low = 0, high = n - 1;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (arr[mid] == val) return mid + 1;
        if (arr[mid] < val) low = mid + 1;
        else high = mid - 1;
    }
    return low + 1;
}

int main() {
    int q;
    long long D;
    if (scanf("%d %lld", &q, &D) != 2) return 0;

    long long *ids = (long long *)malloc(q * sizeof(long long));
    long long *temps = (long long *)malloc(q * sizeof(long long));
    long long *sorted_temps = (long long *)malloc(q * sizeof(long long));
    long long *sorted_ids = (long long *)malloc(q * sizeof(long long));

    for (int i = 0; i < q; i++) {
        scanf("%lld %lld", &ids[i], &temps[i]);
        sorted_temps[i] = temps[i];
        sorted_ids[i] = ids[i];
    }

    // Coordinate Compress Temperatures
    qsort(sorted_temps, q, sizeof(long long), cmp_long);
    int unique_temps_count = 0;
    for (int i = 0; i < q; i++) {
        if (i == 0 || sorted_temps[i] != sorted_temps[i - 1]) {
            sorted_temps[unique_temps_count++] = sorted_temps[i];
        }
    }

    // Coordinate Compress Station IDs for O(log U) array indexing
    qsort(sorted_ids, q, sizeof(long long), cmp_long);
    int unique_ids_count = 0;
    for (int i = 0; i < q; i++) {
        if (i == 0 || sorted_ids[i] != sorted_ids[i - 1]) {
            sorted_ids[unique_ids_count++] = sorted_ids[i];
        }
    }

    // Fenwick Tree over compressed temperature ranks
    int *bit = (int *)calloc(unique_temps_count + 2, sizeof(int));

    // Arrays to maintain per-station latest reading
    char *has_reported = (char *)calloc(unique_ids_count, sizeof(char));
    long long *station_temp = (long long *)malloc(unique_ids_count * sizeof(long long));

    for (int i = 0; i < q; i++) {
        long long sid = ids[i];
        long long curr_temp = temps[i];

        int id_idx = find_rank(sorted_ids, unique_ids_count, sid) - 1;
        int curr_rank = find_rank(sorted_temps, unique_temps_count, curr_temp);

        if (has_reported[id_idx]) {
            long long prev_temp = station_temp[id_idx];
            int prev_rank = find_rank(sorted_temps, unique_temps_count, prev_temp);
            long long diff = llabs(curr_temp - prev_temp);

            if (diff > D) {
                // Drift Event!
                // 1. Evict outdated reading
                bit_add(bit, unique_temps_count, prev_rank, -1);

                // 2. Query count strictly below current temperature
                int count_below = bit_query(bit, curr_rank - 1);

                // 3. Print result
                printf("%lld %lld %d\n", sid, curr_temp, count_below);

                // 4. Insert new reading
                bit_add(bit, unique_temps_count, curr_rank, +1);
            } else {
                // Not a drift: update reading in BIT
                bit_add(bit, unique_temps_count, prev_rank, -1);
                bit_add(bit, unique_temps_count, curr_rank, +1);
            }
        } else {
            // First-time report: register in BIT
            has_reported[id_idx] = 1;
            bit_add(bit, unique_temps_count, curr_rank, +1);
        }

        station_temp[id_idx] = curr_temp;
    }

    free(ids);
    free(temps);
    free(sorted_temps);
    free(sorted_ids);
    free(bit);
    free(has_reported);
    free(station_temp);

    return 0;
}
```

---

### C++ (GCC++ 13.2.0)

```cpp
#include <iostream>
#include <vector>
#include <cmath>
#include <algorithm>
#include <unordered_map>

using namespace std;

// Fenwick Tree (Binary Indexed Tree)
class FenwickTree {
private:
    int n;
    vector<int> tree;

public:
    FenwickTree(int size) : n(size), tree(size + 2, 0) {}

    void add(int idx, int delta) {
        for (; idx <= n; idx += idx & -idx) {
            tree[idx] += delta;
        }
    }

    int query(int idx) const {
        int sum = 0;
        for (; idx > 0; idx -= idx & -idx) {
            sum += tree[idx];
        }
        return sum;
    }
};

int main() {
    // Fast I/O
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);

    int q;
    long long D;
    if (!(cin >> q >> D)) return 0;

    vector<long long> ids(q);
    vector<long long> temps(q);
    vector<long long> sorted_temps(q);

    for (int i = 0; i < q; i++) {
        cin >> ids[i] >> temps[i];
        sorted_temps[i] = temps[i];
    }

    // Coordinate Compression
    sort(sorted_temps.begin(), sorted_temps.end());
    sorted_temps.erase(unique(sorted_temps.begin(), sorted_temps.end()), sorted_temps.end());
    int m = sorted_temps.size();

    auto get_rank = [&](long long val) -> int {
        return lower_bound(sorted_temps.begin(), sorted_temps.end(), val) - sorted_temps.begin() + 1;
    };

    FenwickTree bit(m);
    unordered_map<long long, long long> station_latest;
    station_latest.reserve(q * 2);

    for (int i = 0; i < q; i++) {
        long long sid = ids[i];
        long long curr_temp = temps[i];
        int curr_rank = get_rank(curr_temp);

        auto it = station_latest.find(sid);
        if (it != station_latest.end()) {
            long long prev_temp = it->second;
            int prev_rank = get_rank(prev_temp);
            long long diff = llabs(curr_temp - prev_temp);

            if (diff > D) {
                // Drift Event!
                // 1. Remove outdated reading from active set
                bit.add(prev_rank, -1);

                // 2. Query count of active readings strictly below current reading
                int count_below = bit.query(curr_rank - 1);

                // 3. Output drift event
                cout << sid << " " << curr_temp << " " << count_below << "\n";

                // 4. Insert new reading into active set
                bit.add(curr_rank, +1);
            } else {
                // Standard update
                bit.add(prev_rank, -1);
                bit.add(curr_rank, +1);
            }
        } else {
            // First-time report: register into active set
            bit.add(curr_rank, +1);
        }

        station_latest[sid] = curr_temp;
    }

    return 0;
}
```

---

### C# (mcs 5.4.0.201)

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Text;

class Solution {
    // Binary Indexed Tree
    class FenwickTree {
        private readonly int size;
        private readonly int[] tree;

        public FenwickTree(int size) {
            this.size = size;
            this.tree = new int[size + 2];
        }

        public void Add(int idx, int delta) {
            for (; idx <= size; idx += idx & -idx) {
                tree[idx] += delta;
            }
        }

        public int Query(int idx) {
            int sum = 0;
            for (; idx > 0; idx -= idx & -idx) {
                sum += tree[idx];
            }
            return sum;
        }
    }

    static void Main(string[] args) {
        // High-performance token reader
        using (var reader = new StreamReader(Console.OpenStandardInput(), Encoding.UTF8, false, 1 << 16)) {
            string content = reader.ReadToEnd();
            if (string.IsNullOrWhiteSpace(content)) return;

            string[] tokens = content.Split(new char[] { ' ', '\t', '\r', '\n' }, StringSplitOptions.RemoveEmptyEntries);
            if (tokens.Length < 2) return;

            int q = int.Parse(tokens[0]);
            long D = long.Parse(tokens[1]);

            long[] ids = new long[q];
            long[] temps = new long[q];
            long[] sortedTemps = new long[q];

            int tIdx = 2;
            for (int i = 0; i < q; i++) {
                ids[i] = long.Parse(tokens[tIdx++]);
                temps[i] = long.Parse(tokens[tIdx++]);
                sortedTemps[i] = temps[i];
            }

            // Coordinate Compression
            Array.Sort(sortedTemps);
            int uniqueCount = 0;
            for (int i = 0; i < q; i++) {
                if (i == 0 || sortedTemps[i] != sortedTemps[i - 1]) {
                    sortedTemps[uniqueCount++] = sortedTemps[i];
                }
            }

            FenwickTree bit = new FenwickTree(uniqueCount);
            Dictionary<long, long> stationLatest = new Dictionary<long, long>(q);
            StringBuilder sb = new StringBuilder();

            for (int i = 0; i < q; i++) {
                long sid = ids[i];
                long currTemp = temps[i];
                int currRank = Array.BinarySearch(sortedTemps, 0, uniqueCount, currTemp) + 1;

                if (stationLatest.TryGetValue(sid, out long prevTemp)) {
                    int prevRank = Array.BinarySearch(sortedTemps, 0, uniqueCount, prevTemp) + 1;
                    long diff = Math.Abs(currTemp - prevTemp);

                    if (diff > D) {
                        // Drift Event!
                        bit.Add(prevRank, -1);
                        int countBelow = bit.Query(currRank - 1);
                        sb.Append(sid).Append(' ').Append(currTemp).Append(' ').Append(countBelow).Append('\n');
                        bit.Add(currRank, +1);
                    } else {
                        bit.Add(prevRank, -1);
                        bit.Add(currRank, +1);
                    }
                } else {
                    bit.Add(currRank, +1);
                }

                stationLatest[sid] = currTemp;
            }

            Console.Write(sb.ToString());
        }
    }
}
```

---

### JavaScript (Node 24.4.1)

```javascript
function processData(input) {
    const tokens = input.trim().split(/\s+/);
    if (!tokens || tokens.length < 2) return;

    let ptr = 0;
    const q = parseInt(tokens[ptr++], 10);
    const D = Number(tokens[ptr++]);

    const ids = new Array(q);
    const temps = new Array(q);
    const sortedTemps = new Array(q);

    for (let i = 0; i < q; i++) {
        ids[i] = tokens[ptr++];
        temps[i] = Number(tokens[ptr++]);
        sortedTemps[i] = temps[i];
    }

    // Coordinate Compression
    sortedTemps.sort((a, b) => a - b);
    const uniqueTemps = [];
    for (let i = 0; i < q; i++) {
        if (i === 0 || sortedTemps[i] !== sortedTemps[i - 1]) {
            uniqueTemps.push(sortedTemps[i]);
        }
    }
    const totalUnique = uniqueTemps.length;

    // Binary search for 1-based rank
    function getRank(val) {
        let low = 0, high = totalUnique - 1;
        while (low <= high) {
            const mid = (low + high) >> 1;
            if (uniqueTemps[mid] === val) return mid + 1;
            if (uniqueTemps[mid] < val) low = mid + 1;
            else high = mid - 1;
        }
        return low + 1;
    }

    // Fenwick Tree using Int32Array for maximum V8 performance
    const bit = new Int32Array(totalUnique + 2);

    function bitAdd(idx, val) {
        for (; idx <= totalUnique; idx += idx & -idx) {
            bit[idx] += val;
        }
    }

    function bitQuery(idx) {
        let sum = 0;
        for (; idx > 0; idx -= idx & -idx) {
            sum += bit[idx];
        }
        return sum;
    }

    const stationLatest = new Map();
    const result = [];

    for (let i = 0; i < q; i++) {
        const sid = ids[i];
        const currTemp = temps[i];
        const currRank = getRank(currTemp);

        if (stationLatest.has(sid)) {
            const prevTemp = stationLatest.get(sid);
            const prevRank = getRank(prevTemp);
            const diff = Math.abs(currTemp - prevTemp);

            if (diff > D) {
                // Drift Event!
                // 1. Evict outdated reading
                bitAdd(prevRank, -1);

                // 2. Query count strictly below currTemp
                const countBelow = bitQuery(currRank - 1);

                // 3. Record drift event
                result.push(`${sid} ${currTemp} ${countBelow}`);

                // 4. Insert new reading
                bitAdd(currRank, 1);
            } else {
                // Standard reading replacement
                bitAdd(prevRank, -1);
                bitAdd(currRank, 1);
            }
        } else {
            // First-time report: register reading
            bitAdd(currRank, 1);
        }

        stationLatest.set(sid, currTemp);
    }

    if (result.length > 0) {
        console.log(result.join('\n'));
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

---

## 12. Key Takeaways & Interview Cheat-Sheet

1. **Order Statistics over Dynamic Streams:**
   Whenever a streaming system requires calculating ranks or counts under dynamic insertions and deletions, think **Fenwick Tree / Segment Tree + Coordinate Compression**.
2. **Offline vs. Online Coordinate Compression:**
   When all values that will ever be queried are known upfront ($q$ reports), sort and deduplicate them in $\mathcal{O}(q \log q)$ before streaming. If queries were fully online with unknown ranges, a dynamic balanced BST (like Treap, Splay Tree, or PBDS) or a Dynamic Segment Tree with pointer-based node creation would be required.
3. **Strict Inequality vs. Inclusive Inequality in Fenwick Trees:**
   - Elements strictly less than $x$: `query(rank(x) - 1)`
   - Elements less than or equal to $x$: `query(rank(x))`
   - Elements strictly greater than $x$: `total_active - query(rank(x))`
4. **Pre-Update Exclusion Axiom:**
   Notice the problem specifically requires excluding the drifting station's own outdated reading *before* taking the count, then adding the new reading *afterwards*. Always sequence:
   $$\text{Evict Old} \longrightarrow \text{Query Rank} \longrightarrow \text{Insert New}$$
5. **Memory and Cache Efficiency:**
   A Fenwick Tree over compressed coordinates uses a flat continuous array of $2 \times 10^5$ integers ($\approx 800\text{ KB}$), fitting entirely inside modern L2/L3 CPU cache, yielding orders of magnitude faster execution than pointer-heavy tree structures.
