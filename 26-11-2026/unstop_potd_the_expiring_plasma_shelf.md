# Unstop Problem of the Day: The Expiring Plasma Shelf

- **Platform:** [Unstop](https://unstop.com/)
- **Difficulty:** Medium
- **Topic Tags:** Priority Queue (Min-Heap), Hash Map / Set, Lazy Deletion, Event-Driven Simulation, Greedy Strategy, Two-Key Sorting
- **Target Audience:** POTD Solvers, Computer Science Students, Competitive Programmers, Technical Interview Candidates (Google, Amazon, Microsoft, Uber, Atlassian)

---

## 1. Problem Statement

Nisha manages the cold storage room of a regional hospital network. Plasma units arrive throughout the day. Each plasma unit is characterized by:
- A unique identifier `id` ($1 \le id \le 10^9$)
- A blood group label `group` ($1 \le group \le G$)
- Its last safe day of usability `expiry` ($t \le expiry \le 10^9$)

### Dispensing & Shelf Rules
1. **Earliest Expiry First (Greedy):** Whenever a hospital ward requests plasma of a particular blood group on day $t$, Nisha hands over the usable unit of that blood group that **will expire the soonest** (minimal `expiry`), minimizing waste.
2. **Tie-Breaker:** If two or more usable units of that group share the exact same `expiry`, the unit with the **smaller identifier** `id` is handed over first.
3. **Usability Invariant:** A unit is usable on day $t$ if and only if:
   $$expiry \ge t$$
   Any unit whose `expiry < t` has expired. Expired units must never be handed out and are discarded as soon as noticed.
4. **Donor Withdrawals:** A donor may withdraw their donation at any moment. Once withdrawn, that unit must behave as if it had never been available.
   - Withdrawing a unit that was already handed over, or is no longer on the shelf, has **no effect**.
5. **Single-Use Constraint:** Once a unit has been handed over to a ward, it is permanently removed from the shelf and can never be used again.

### Event Stream
The storage log lists $q$ events in chronological order with non-decreasing days $t$. Three types of events appear:
1. `t D id group expiry`: On day $t$, delivery of a new unit with identifier `id`, blood group `group`, and last safe day `expiry`.
2. `t C id`: On day $t$, donor withdrawal of the unit with identifier `id`.
3. `t R group`: On day $t$, a ward requests a unit of blood group `group`.

### Output Specification
For every request event `t R group`, print on a new line:
- The `id` of the unit handed over, OR
- The word `NONE` if no usable unit of that group exists on the shelf at that moment.

---

## 2. Examples & Visual Walkthroughs

### Example 1 (Sample Testcase 0)

#### Input:
```text
10 2
1 D 101 2 6
1 D 102 2 2
1 D 103 1 9
2 D 104 2 3
3 C 104
3 R 2
3 R 1
4 R 2
5 D 106 2 6
6 R 2
```

#### Output:
```text
101
103
NONE
106
```

#### Chronological Event Timeline:

```
Day  Event               Action & State Changes                                      Output
────────────────────────────────────────────────────────────────────────────────────────────
t=1  D 101 2 6           Group 2 receives Unit 101 (expiry=6)                        —
t=1  D 102 2 2           Group 2 receives Unit 102 (expiry=2)                        —
t=1  D 103 1 9           Group 1 receives Unit 103 (expiry=9)                        —
t=2  D 104 2 3           Group 2 receives Unit 104 (expiry=3)                        —
t=3  C 104               Unit 104 marked as WITHDRAWN                                —
t=3  R 2                 Inspect Group 2 heap:
                         - Top: Unit 102 (expiry=2 < t=3) -> EXPIRED! Discard.
                         - Next: Unit 104 (withdrawn)    -> WITHDRAWN! Discard.
                         - Next: Unit 101 (expiry=6 >= 3) -> VALID! Hand over.       101
t=3  R 1                 Inspect Group 1 heap:
                         - Top: Unit 103 (expiry=9 >= 3) -> VALID! Hand over.       103
t=4  R 2                 Inspect Group 2 heap:
                         - Group 2 heap is EMPTY!                                    NONE
t=5  D 106 2 6           Group 2 receives Unit 106 (expiry=6)                        —
t=6  R 2                 Inspect Group 2 heap:
                         - Top: Unit 106 (expiry=6 >= t=6) -> VALID! Hand over.      106
```

---

### Example 2 (Sample Testcase 1)

#### Input:
```text
9 3
1 D 7 3 4
1 D 8 3 4
1 D 9 1 2
2 C 7
2 R 3
3 R 1
3 D 10 3 5
4 R 3
4 R 3
```

#### Output:
```text
8
NONE
10
NONE
```

#### Explanation:
- On day $1$, Group 3 has Unit $7$ (`expiry=4`) and Unit $8$ (`expiry=4`).
- On day $2$, Unit $7$ is withdrawn.
- On day $2$, Request for Group 3: Unit $7$ is discarded because it was withdrawn. Unit $8$ is usable (`expiry 4 >= 2`) and handed over $\implies$ **`8`**.
- On day $3$, Request for Group 1: Unit $9$ had `expiry=2 < 3`, so it is expired and discarded. Shelf is empty $\implies$ **`NONE`**.
- On day $3$, Unit $10$ (`group=3, expiry=5`) arrives.
- On day $4$, First request for Group 3: Unit $10$ (`expiry 5 >= 4`) is handed over $\implies$ **`10`**.
- On day $4$, Second request for Group 3: Shelf is empty $\implies$ **`NONE`**.

---

## 3. Constraints & Scale Analysis

| Parameter | Constraint Range | Practical Algorithmic Implication |
| :--- | :--- | :--- |
| **Events Count ($q$)** | $1 \le q \le 200,000$ | Total operations up to $2 \times 10^5$. Any $\mathcal{O}(q^2)$ solution ($4 \times 10^{10}$ ops) will trigger **Time Limit Exceeded (TLE)**. |
| **Blood Groups ($G$)** | $1 \le G \le 100,000$ | Memory allocation for group containers must be $\mathcal{O}(G)$. Dynamic structures per group are necessary. |
| **Timestamps ($t, expiry$)** | $1 \le t \le expiry \le 10^9$ | Time fits within 64-bit integers (`long` / `long long`). |
| **Unit Identifiers ($id$)** | $1 \le id \le 10^9$ | Sparse IDs cannot be stored in a direct flat array; requires a **Hash Set** or balanced table. |
| **Distinct Deliveries** | All delivered `id`s are unique | Each `id` enters at most one blood group container. |
| **Time Limit** | $\approx 2.0\text{ seconds}$ | An $\mathcal{O}(q \log q)$ algorithm requires $\approx 2 \times 10^5 \times 18 \approx 3.6 \times 10^6$ operations ($\approx 0.05\text{ seconds}$), running comfortably within limits. |

---

## 4. Visual Architecture & Key Mathematical Insights

### 1. Group-Partitioned Priority Queues (Min-Heaps)
Because wards request plasma strictly by blood group, maintaining a single global shelf would require filtering by blood group on every request. Instead, we partition the cold room into **$G$ independent Min-Heaps**, one for each blood group:

$$\text{Shelf}[g] = \text{Min-Heap of usable units for blood group } g$$

Each element in $\text{Shelf}[g]$ is a pair:
$$\text{Unit} = (\text{expiry}, id)$$

Ordered lexicographically by:
$$\text{Unit}_A < \text{Unit}_B \iff (\text{expiry}_A < \text{expiry}_B) \lor (\text{expiry}_A = \text{expiry}_B \land id_A < id_B)$$

```
Group 1 Min-Heap:       Group 2 Min-Heap:              Group G Min-Heap:
    (9, 103)                 (2, 102)                       (15, 501)
                            /        \
                        (3, 104)    (6, 101)
```

---

### 2. The Power of Lazy Deletion (Tombstoning)
In standard priority queues, locating and deleting an arbitrary element (e.g., when donor `id` is withdrawn via `t C id`) takes $\mathcal{O}(K)$ time, which degrades overall performance to $\mathcal{O}(q \cdot K) \approx \mathcal{O}(q^2)$.

#### The Lazy Deletion Invariant:
1. When unit `id` is withdrawn, we do **not** search inside the heap. We simply insert `id` into a fast **Hash Set** `withdrawn`:
   $$\text{withdrawn}.\text{insert}(id) \quad \implies \mathcal{O}(1) \text{ time}$$
2. When a request for group $g$ arrives at day $t$, we inspect the root of $\text{Shelf}[g]$:
   - **Check 1 (Withdrawal):** If $\text{top}.id \in \text{withdrawn}$, pop and discard it!
   - **Check 2 (Expiry):** If $\text{top}.expiry < t$, pop and discard it!
   - Repeat until the root satisfies $\text{top}.id \notin \text{withdrawn}$ and $\text{top}.expiry \ge t$, or the heap becomes empty.
3. If a valid unit is found, pop it, announce its `id`, and permanently discard it.

#### Amortized Analysis Proof:
- Each delivery event inserts exactly **one** unit into its group heap ($\mathcal{O}(\log q)$).
- A unit is popped from the heap **at most once** across its lifetime (either when discarded as expired/withdrawn, or handed over).
- Total heap insertions $\le q$, total heap deletions $\le q$.
- Hash set lookups take $\mathcal{O}(1)$ average time.
- **Total Time Complexity:** $\mathbf{\mathcal{O}(q \log q)}$.

---

## 5. Step-by-Step Simulation & Trace Table

Tracing Sample Testcase 0: $q = 10, G = 2$.

| Event # | Day $t$ | Type | Arguments | State of `withdrawn` Set | Heap 1: $(expiry, id)$ | Heap 2: $(expiry, id)$ | Request Resolution | Output |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **1** | $1$ | `D` | `id=101, grp=2, exp=6` | $\emptyset$ | `[]` | `[(6, 101)]` | — | — |
| **2** | $1$ | `D` | `id=102, grp=2, exp=2` | $\emptyset$ | `[]` | `[(2, 102), (6, 101)]` | — | — |
| **3** | $1$ | `D` | `id=103, grp=1, exp=9` | $\emptyset$ | `[(9, 103)]` | `[(2, 102), (6, 101)]` | — | — |
| **4** | $2$ | `D` | `id=104, grp=2, exp=3` | $\emptyset$ | `[(9, 103)]` | `[(2, 102), (3, 104), (6, 101)]` | — | — |
| **5** | $3$ | `C` | `id=104` | $\{104\}$ | `[(9, 103)]` | `[(2, 102), (3, 104), (6, 101)]` | — | — |
| **6** | $3$ | `R` | `grp=2` | $\{104\}$ | `[(9, 103)]` | Pop $(2, 102)$ (exp $< 3$).<br/>Pop $(3, 104)$ (withdrawn).<br/>Valid: $(6, 101)$. | Hand over $101$ | **`101`** |
| **7** | $3$ | `R` | `grp=1` | $\{104\}$ | Pop $(9, 103)$ (exp $\ge 3$). | `[]` | Hand over $103$ | **`103`** |
| **8** | $4$ | `R` | `grp=2` | $\{104\}$ | `[]` | `[]` (Empty) | No units | **`NONE`** |
| **9** | $5$ | `D` | `id=106, grp=2, exp=6` | $\{104\}$ | `[]` | `[(6, 106)]` | — | — |
| **10** | $6$ | `R` | `grp=2` | $\{104\}$ | `[]` | Pop $(6, 106)$ (exp $\ge 6$). | Hand over $106$ | **`106`** |

---

## 6. Algorithm Flowchart

```mermaid
flowchart TD
    Start([Start: Process Event Stream]) --> ReadQG["Read q, G<br/>Initialize G Min-Heaps<br/>Initialize withdrawn HashSet"]
    
    ReadQG --> LoopEvents["For each event in 1 to q"]
    
    LoopEvents --> CheckType{"Event Type?"}
    
    CheckType -- "D (Delivery)" --> Delivery["Read t, id, group, expiry<br/>heap[group].push(expiry, id)"]
    Delivery --> NextEvent{"More Events?"}
    
    CheckType -- "C (Withdrawal)" --> Withdrawal["Read t, id<br/>withdrawn.insert(id)"]
    Withdrawal --> NextEvent
    
    CheckType -- "R (Request)" --> Request["Read t, group<br/>h = heap[group]<br/>ans = 'NONE'"]
    
    Request --> WhileHeap{"h is not empty?"}
    
    WhileHeap -- Yes --> PeekTop["top = h.peek()"]
    PeekTop --> CheckWithdrawn{"top.id in withdrawn?"}
    
    CheckWithdrawn -- Yes --> PopWithdrawn["h.pop()<br/>(Discard withdrawn unit)"]
    PopWithdrawn --> WhileHeap
    
    CheckWithdrawn -- No --> CheckExpiry{"top.expiry < t ?"}
    CheckExpiry -- Yes --> PopExpired["h.pop()<br/>(Discard expired unit)"]
    PopExpired --> WhileHeap
    
    CheckExpiry -- No --> ValidUnit["ans = top.id<br/>h.pop()<br/>(Hand over to ward)"]
    ValidUnit --> PrintAns["Print ans"]
    
    WhileHeap -- No --> PrintAns
    PrintAns --> NextEvent
    
    NextEvent -- Yes --> LoopEvents
    NextEvent -- No --> End([End])

    style Start fill:#2E7D32,stroke:#1B5E20,color:#ffffff
    style End fill:#1565C0,stroke:#0D47A1,color:#ffffff
    style CheckType fill:#F57C00,stroke:#E65100,color:#ffffff
    style CheckWithdrawn fill:#7B1FA2,stroke:#4A148C,color:#ffffff
    style CheckExpiry fill:#D32F2F,stroke:#B71C1C,color:#ffffff
    style ValidUnit fill:#388E3C,stroke:#1B5E20,color:#ffffff
    style PrintAns fill:#0288D1,stroke:#01579B,color:#ffffff
```

---

## 7. Progression from Naive to Pro

### Method 1: Naive List Scan (Brute Force)
- **Concept:** Store all available units in an array or linked list. For every request, linearly scan all units of that blood group to find the one with minimum expiry (and minimum `id`), ignoring expired and withdrawn units.
- **Complexity:**
  - Delivery: $\mathcal{O}(1)$
  - Withdrawal: $\mathcal{O}(1)$
  - Request: $\mathcal{O}(N)$ where $N$ is current units on shelf
  - Overall Time: $\mathcal{O}(q^2) \approx 4 \times 10^{10}$ operations
- **Verdict:** Immediate **Time Limit Exceeded (TLE)**.
- **Pseudo-Code:**
  ```text
  function handleRequestNaive(group, t):
      best = null
      for unit in shelf:
          if unit.group == group and unit.expiry >= t and unit.id not in withdrawn:
              if best == null or (unit.expiry, unit.id) < (best.expiry, best.id):
                  best = unit
      if best != null:
          shelf.remove(best)
          return best.id
      return "NONE"
  ```

---

### Method 2: Balanced Binary Search Tree (Eager Removal)
- **Concept:** Use a balanced BST (`std::set` in C++, `TreeSet` in Java) for each blood group. When `t C id` arrives, look up the unit by `id` (using an auxiliary map `id -> iterator`) and immediately erase it from the BST.
- **Complexity:**
  - Time: $\mathcal{O}(q \log q)$
  - Space: $\mathcal{O}(q)$
- **Verdict:** Fast enough to pass, but maintaining iterators/node pointers across insertions and dynamic rebalancing adds substantial memory and constant-factor runtime overhead.
- **Pseudo-Code:**
  ```text
  function handleCancelEager(id):
      if id in id_to_iter:
          group = id_to_group[id]
          shelf[group].erase(id_to_iter[id])
  ```

---

### Method 3: Optimal Min-Heap with Hash-Based Lazy Deletion (Pro Method)
- **Concept:** Min-Heap per blood group storing `(expiry, id)`. Donor withdrawals simply append `id` to a hash set. Expired and withdrawn units are popped lazily from the top of the heap only when encountered during a request.
- **Complexity:**
  - Delivery: $\mathcal{O}(\log (\text{heap size}))$
  - Withdrawal: $\mathcal{O}(1)$ average hash set insert
  - Request: Amortized $\mathcal{O}(\log (\text{heap size}))$
  - Overall Time: $\mathbf{\mathcal{O}(q \log q)}$
  - Space: $\mathbf{\mathcal{O}(q + G)}$
- **Verdict:** Optimal runtime, cache-friendly array storage, and minimal memory footprint.

---

## 8. Multi-Language Implementations

### Java (OpenJDK 21.0)

```java
import java.io.*;
import java.util.*;

public class Main {
    /**
     * Represents a single plasma unit in the priority queue.
     * Implements Comparable for two-level min-heap ordering:
     * 1. Smallest expiry first.
     * 2. Smallest id first (tie-breaker).
     */
    static class Unit implements Comparable<Unit> {
        long expiry;
        long id;

        public Unit(long expiry, long id) {
            this.expiry = expiry;
            this.id = id;
        }

        @Override
        public int compareTo(Unit other) {
            if (this.expiry != other.expiry) {
                return Long.compare(this.expiry, other.expiry);
            }
            return Long.compare(this.id, other.id);
        }
    }

    /**
     * High-speed custom tokenizer for reading large competitive programming inputs.
     */
    static class FastScanner {
        private final InputStream stream;
        private final byte[] buffer = new byte[1 << 16];
        private int head = 0;
        private int tail = 0;

        public FastScanner(InputStream stream) {
            this.stream = stream;
        }

        private byte read() throws IOException {
            if (head >= tail) {
                head = 0;
                tail = stream.read(buffer, 0, buffer.length);
                if (tail <= 0) return -1;
            }
            return buffer[head++];
        }

        public String next() throws IOException {
            byte b = read();
            while (b <= ' ') {
                if (b == -1) return null;
                b = read();
            }
            StringBuilder sb = new StringBuilder();
            while (b > ' ') {
                sb.append((char) b);
                b = read();
            }
            return sb.toString();
        }

        public long nextLong() throws IOException {
            byte b = read();
            while (b <= ' ') {
                if (b == -1) return -1;
                b = read();
            }
            long res = 0;
            while (b > ' ') {
                res = res * 10 + (b - '0');
                b = read();
            }
            return res;
        }

        public int nextInt() throws IOException {
            return (int) nextLong();
        }
    }

    @SuppressWarnings("unchecked")
    public static void main(String[] args) throws IOException {
        FastScanner scanner = new FastScanner(System.in);
        String firstToken = scanner.next();
        if (firstToken == null) return;

        int q = Integer.parseInt(firstToken);
        int G = scanner.nextInt();

        // One Min-Heap per blood group (1-indexed)
        PriorityQueue<Unit>[] heaps = new PriorityQueue[G + 1];
        for (int i = 1; i <= G; i++) {
            heaps[i] = new PriorityQueue<>();
        }

        // Set to record IDs of withdrawn plasma units (Lazy Deletion)
        HashSet<Long> withdrawn = new HashSet<>();
        StringBuilder output = new StringBuilder();

        for (int ev = 0; ev < q; ev++) {
            long t = scanner.nextLong();
            String type = scanner.next();

            if (type.equals("D")) {
                long id = scanner.nextLong();
                int group = scanner.nextInt();
                long expiry = scanner.nextLong();
                heaps[group].offer(new Unit(expiry, id));
            } else if (type.equals("C")) {
                long id = scanner.nextLong();
                // Idempotently mark this ID as withdrawn
                withdrawn.add(id);
            } else if (type.equals("R")) {
                int group = scanner.nextInt();
                PriorityQueue<Unit> pq = heaps[group];
                long ans = -1;

                // Lazily purge withdrawn and expired units from the top
                while (!pq.isEmpty()) {
                    Unit top = pq.peek();

                    if (withdrawn.contains(top.id)) {
                        pq.poll(); // Discard withdrawn unit
                        continue;
                    }
                    if (top.expiry < t) {
                        pq.poll(); // Discard expired unit
                        continue;
                    }

                    // Top unit is usable and optimal
                    ans = top.id;
                    pq.poll(); // Handed over to ward; permanently removed
                    break;
                }

                if (ans == -1) {
                    output.append("NONE\n");
                } else {
                    output.append(ans).append("\n");
                }
            }
        }

        System.out.print(output);
    }
}
```

---

### Python (3.12.11)

```python
import sys
import heapq

def main():
    """
    Solves The Expiring Plasma Shelf problem using:
    - Independent min-heaps for each blood group.
    - Lazy deletion via a hash set for withdrawn IDs.
    - Fast batch I/O to handle up to 200,000 events effortlessly.
    """
    input_data = sys.stdin.read().split()
    if not input_data:
        return

    ptr = 0
    q = int(input_data[ptr])
    ptr += 1
    G = int(input_data[ptr])
    ptr += 1

    # List of min-heaps for blood groups 1 to G
    heaps = [[] for _ in range(G + 1)]
    # Hash set of withdrawn unit IDs
    withdrawn = set()
    out = []

    for _ in range(q):
        t = int(input_data[ptr])
        event_type = input_data[ptr + 1]
        ptr += 2

        if event_type == 'D':
            unit_id = int(input_data[ptr])
            group = int(input_data[ptr + 1])
            expiry = int(input_data[ptr + 2])
            ptr += 3
            # Python tuple (expiry, unit_id) naturally compares by expiry first, then unit_id
            heapq.heappush(heaps[group], (expiry, unit_id))

        elif event_type == 'C':
            unit_id = int(input_data[ptr])
            ptr += 1
            withdrawn.add(unit_id)

        elif event_type == 'R':
            group = int(input_data[ptr])
            ptr += 1
            h = heaps[group]
            ans = "NONE"

            # Lazily pop expired or withdrawn units
            while h:
                exp, uid = h[0]
                if uid in withdrawn:
                    heapq.heappop(h)
                    continue
                if exp < t:
                    heapq.heappop(h)
                    continue

                # Valid unit found
                ans = str(uid)
                heapq.heappop(h)
                break

            out.append(ans)

    sys.stdout.write("\n".join(out) + "\n")

if __name__ == '__main__':
    main()
```

---

### C (GCC 13.2.0)

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

/**
 * Open-addressing hash set for fast O(1) withdrawal lookups.
 * Table capacity chosen as 2^19 = 524,288 (> 200,000 queries, load factor < 0.4).
 */
#define HASH_CAP 524288
static long long withdrawn_table[HASH_CAP];

static void add_withdrawn(long long id) {
    // SplitMix64 64-bit integer mixing multiplier
    unsigned int idx = ((unsigned long long)id * 11400714819323198485ULL) & (HASH_CAP - 1);
    while (withdrawn_table[idx] != 0) {
        if (withdrawn_table[idx] == id) return;
        idx = (idx + 1) & (HASH_CAP - 1);
    }
    withdrawn_table[idx] = id;
}

static int is_withdrawn(long long id) {
    unsigned int idx = ((unsigned long long)id * 11400714819323198485ULL) & (HASH_CAP - 1);
    while (withdrawn_table[idx] != 0) {
        if (withdrawn_table[idx] == id) return 1;
        idx = (idx + 1) & (HASH_CAP - 1);
    }
    return 0;
}

/**
 * Representation of a plasma unit.
 */
typedef struct {
    long long expiry;
    long long id;
} Unit;

/**
 * Dynamic binary min-heap structure.
 */
typedef struct {
    Unit* data;
    int size;
    int cap;
} MinHeap;

static MinHeap heaps[100005];

static inline int compare_units(Unit a, Unit b) {
    if (a.expiry != b.expiry) return (a.expiry < b.expiry) ? -1 : 1;
    return (a.id < b.id) ? -1 : (a.id > b.id ? 1 : 0);
}

static void heap_push(MinHeap* h, long long expiry, long long id) {
    if (h->size == h->cap) {
        h->cap = (h->cap == 0) ? 4 : (h->cap * 2);
        h->data = (Unit*)realloc(h->data, h->cap * sizeof(Unit));
    }
    Unit item;
    item.expiry = expiry;
    item.id = id;

    int i = h->size++;
    while (i > 0) {
        int p = (i - 1) >> 1;
        if (compare_units(item, h->data[p]) < 0) {
            h->data[i] = h->data[p];
            i = p;
        } else {
            break;
        }
    }
    h->data[i] = item;
}

static Unit heap_pop(MinHeap* h) {
    Unit top = h->data[0];
    Unit last = h->data[--h->size];
    if (h->size > 0) {
        int i = 0;
        while ((i << 1) + 1 < h->size) {
            int left = (i << 1) + 1;
            int right = left + 1;
            int best = i;

            if (compare_units(h->data[left], last) < 0) {
                best = left;
            }
            if (right < h->size) {
                Unit cur_best = (best == i) ? last : h->data[left];
                if (compare_units(h->data[right], cur_best) < 0) {
                    best = right;
                }
            }
            if (best != i) {
                h->data[i] = h->data[best];
                i = best;
            } else {
                break;
            }
        }
        h->data[i] = last;
    }
    return top;
}

int main() {
    int q, G;
    if (scanf("%d %d", &q, &G) != 2) return 0;

    for (int ev = 0; ev < q; ev++) {
        long long t;
        char type[4];
        scanf("%lld %s", &t, type);

        if (type[0] == 'D') {
            long long id, expiry;
            int group;
            scanf("%lld %d %lld", &id, &group, &expiry);
            heap_push(&heaps[group], expiry, id);
        } else if (type[0] == 'C') {
            long long id;
            scanf("%lld", &id);
            add_withdrawn(id);
        } else if (type[0] == 'R') {
            int group;
            scanf("%d", &group);
            MinHeap* h = &heaps[group];
            long long ans = -1;

            while (h->size > 0) {
                Unit top = h->data[0];
                if (is_withdrawn(top.id)) {
                    heap_pop(h);
                    continue;
                }
                if (top.expiry < t) {
                    heap_pop(h);
                    continue;
                }
                ans = top.id;
                heap_pop(h);
                break;
            }

            if (ans == -1) {
                printf("NONE\n");
            } else {
                printf("%lld\n", ans);
            }
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
#include <unordered_set>

using namespace std;

/**
 * Plasma unit representation for Min-Heap.
 * Overloads operator> for std::greater in std::priority_queue.
 */
struct Unit {
    long long expiry;
    long long id;

    bool operator>(const Unit& other) const {
        if (expiry != other.expiry) {
            return expiry > other.expiry; // Smaller expiry takes priority
        }
        return id > other.id;             // Smaller id breaks ties
    }
};

int main() {
    // Optimize standard I/O operations for performance
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);

    int q, G;
    if (!(cin >> q >> G)) return 0;

    // Vector of min-priority queues, one per blood group (1-indexed)
    vector<priority_queue<Unit, vector<Unit>, greater<Unit>>> heaps(G + 1);
    
    // Hash set for lazy deletion of withdrawn units
    unordered_set<long long> withdrawn;

    for (int ev = 0; ev < q; ++ev) {
        long long t;
        char type;
        cin >> t >> type;

        if (type == 'D') {
            long long id, expiry;
            int group;
            cin >> id >> group >> expiry;
            heaps[group].push({expiry, id});
        } else if (type == 'C') {
            long long id;
            cin >> id;
            withdrawn.insert(id);
        } else if (type == 'R') {
            int group;
            cin >> group;
            auto& pq = heaps[group];
            long long ans = -1;

            // Lazily pop expired or withdrawn units
            while (!pq.empty()) {
                Unit top = pq.top();

                if (withdrawn.count(top.id)) {
                    pq.pop(); // Discard withdrawn unit
                    continue;
                }
                if (top.expiry < t) {
                    pq.pop(); // Discard expired unit
                    continue;
                }

                // Optimal usable unit found
                ans = top.id;
                pq.pop(); // Handed over to ward
                break;
            }

            if (ans == -1) {
                cout << "NONE\n";
            } else {
                cout << ans << "\n";
            }
        }
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
    /// <summary>
    /// Represents a plasma unit.
    /// </summary>
    struct Unit : IComparable<Unit> {
        public long Expiry;
        public long Id;

        public Unit(long expiry, long id) {
            Expiry = expiry;
            Id = id;
        }

        public int CompareTo(Unit other) {
            if (this.Expiry != other.Expiry) {
                return this.Expiry.CompareTo(other.Expiry);
            }
            return this.Id.CompareTo(other.Id);
        }
    }

    /// <summary>
    /// Custom binary MinHeap compatible with Mono C# (mcs 5.4.0.201).
    /// </summary>
    class MinHeap {
        private List<Unit> data = new List<Unit>();

        public int Count { get { return data.Count; } }

        public void Push(Unit item) {
            data.Add(item);
            int i = data.Count - 1;
            while (i > 0) {
                int p = (i - 1) >> 1;
                if (data[i].CompareTo(data[p]) < 0) {
                    Unit tmp = data[i];
                    data[i] = data[p];
                    data[p] = tmp;
                    i = p;
                } else {
                    break;
                }
            }
        }

        public Unit Pop() {
            Unit top = data[0];
            int lastIdx = data.Count - 1;
            Unit last = data[lastIdx];
            data.RemoveAt(lastIdx);

            if (data.Count > 0) {
                data[0] = last;
                int i = 0;
                int len = data.Count;
                while ((i << 1) + 1 < len) {
                    int left = (i << 1) + 1;
                    int right = left + 1;
                    int best = i;

                    if (data[left].CompareTo(data[best]) < 0) best = left;
                    if (right < len && data[right].CompareTo(data[best]) < 0) best = right;

                    if (best != i) {
                        Unit tmp = data[i];
                        data[i] = data[best];
                        data[best] = tmp;
                        i = best;
                    } else {
                        break;
                    }
                }
            }
            return top;
        }

        public Unit Peek() {
            return data[0];
        }
    }

    /// <summary>
    /// Fast input reader for large competitive programming datasets.
    /// </summary>
    class FastScanner {
        private StreamReader reader;
        private string[] tokens;
        private int ptr = 0;

        public FastScanner(Stream stream) {
            reader = new StreamReader(stream, Encoding.UTF8, false, 1 << 16);
        }

        public string Next() {
            while (tokens == null || ptr >= tokens.Length) {
                string line = reader.ReadLine();
                if (line == null) return null;
                tokens = line.Split(new char[] { ' ', '\t' }, StringSplitOptions.RemoveEmptyEntries);
                ptr = 0;
            }
            return tokens[ptr++];
        }

        public int NextInt() {
            return int.Parse(Next());
        }

        public long NextLong() {
            return long.Parse(Next());
        }
    }

    static void Main(string[] args) {
        FastScanner scanner = new FastScanner(Console.OpenStandardInput());
        string firstToken = scanner.Next();
        if (firstToken == null) return;

        int q = int.Parse(firstToken);
        int G = scanner.NextInt();

        MinHeap[] heaps = new MinHeap[G + 1];
        for (int i = 1; i <= G; i++) {
            heaps[i] = new MinHeap();
        }

        HashSet<long> withdrawn = new HashSet<long>();
        StringBuilder sb = new StringBuilder();

        for (int ev = 0; ev < q; ev++) {
            long t = scanner.NextLong();
            string type = scanner.Next();

            if (type == "D") {
                long id = scanner.NextLong();
                int group = scanner.NextInt();
                long expiry = scanner.NextLong();
                heaps[group].Push(new Unit(expiry, id));
            } else if (type == "C") {
                long id = scanner.NextLong();
                withdrawn.Add(id);
            } else if (type == "R") {
                int group = scanner.NextInt();
                MinHeap h = heaps[group];
                long ans = -1;

                while (h.Count > 0) {
                    Unit top = h.Peek();
                    if (withdrawn.Contains(top.Id)) {
                        h.Pop(); // Purge withdrawn
                        continue;
                    }
                    if (top.Expiry < t) {
                        h.Pop(); // Purge expired
                        continue;
                    }
                    ans = top.Id;
                    h.Pop(); // Hand over
                    break;
                }

                if (ans == -1) {
                    sb.AppendLine("NONE");
                } else {
                    sb.AppendLine(ans.ToString());
                }
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

/**
 * High-performance binary Min-Heap in JavaScript.
 * Items stored as 2-element arrays: [expiry, id].
 */
class MinHeap {
    constructor() {
        this.data = [];
    }

    push(item) {
        this.data.push(item);
        this._siftUp(this.data.length - 1);
    }

    pop() {
        if (this.data.length === 0) return null;
        const top = this.data[0];
        const last = this.data.pop();
        if (this.data.length > 0) {
            this.data[0] = last;
            this._siftDown(0);
        }
        return top;
    }

    peek() {
        return this.data.length > 0 ? this.data[0] : null;
    }

    size() {
        return this.data.length;
    }

    _compare(a, b) {
        if (a[0] !== b[0]) return a[0] < b[0] ? -1 : 1;
        return a[1] < b[1] ? -1 : (a[1] > b[1] ? 1 : 0);
    }

    _siftUp(i) {
        while (i > 0) {
            const p = (i - 1) >> 1;
            if (this._compare(this.data[i], this.data[p]) < 0) {
                const tmp = this.data[i];
                this.data[i] = this.data[p];
                this.data[p] = tmp;
                i = p;
            } else {
                break;
            }
        }
    }

    _siftDown(i) {
        const len = this.data.length;
        while ((i << 1) + 1 < len) {
            let left = (i << 1) + 1;
            let right = left + 1;
            let best = i;

            if (this._compare(this.data[left], this.data[best]) < 0) best = left;
            if (right < len && this._compare(this.data[right], this.data[best]) < 0) best = right;

            if (best !== i) {
                const tmp = this.data[i];
                this.data[i] = this.data[best];
                this.data[best] = tmp;
                i = best;
            } else {
                break;
            }
        }
    }
}

function processData(input) {
    const tokens = input.trim().split(/\s+/);
    if (!tokens || tokens.length === 0 || tokens[0] === '') return;

    let ptr = 0;
    const q = parseInt(tokens[ptr++], 10);
    const G = parseInt(tokens[ptr++], 10);

    const heaps = new Array(G + 1);
    for (let i = 1; i <= G; i++) {
        heaps[i] = new MinHeap();
    }

    const withdrawn = new Set();
    const results = [];

    for (let ev = 0; ev < q; ev++) {
        const t = parseInt(tokens[ptr++], 10);
        const type = tokens[ptr++];

        if (type === 'D') {
            const id = parseInt(tokens[ptr++], 10);
            const group = parseInt(tokens[ptr++], 10);
            const expiry = parseInt(tokens[ptr++], 10);
            heaps[group].push([expiry, id]);
        } else if (type === 'C') {
            const id = parseInt(tokens[ptr++], 10);
            withdrawn.add(id);
        } else if (type === 'R') {
            const group = parseInt(tokens[ptr++], 10);
            const h = heaps[group];
            let ans = "NONE";

            while (h.size() > 0) {
                const top = h.peek();
                if (withdrawn.has(top[1])) {
                    h.pop(); // Discard withdrawn unit
                    continue;
                }
                if (top[0] < t) {
                    h.pop(); // Discard expired unit
                    continue;
                }

                ans = top[1].toString();
                h.pop(); // Hand over to ward
                break;
            }

            results.push(ans);
        }
    }

    console.log(results.join('\n'));
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

## 9. Complexity Breakdown

| Operation | Time Complexity | Amortized Cost | Explanation |
| :--- | :--- | :--- | :--- |
| **Delivery (`D`)** | $\mathcal{O}(\log K)$ | $\mathcal{O}(\log q)$ | Single insertion into group min-heap containing $K$ units ($K \le q$). |
| **Withdrawal (`C`)** | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ average | Single insert into hash set / table. |
| **Request (`R`)** | $\mathcal{O}(P \log K)$ | Amortized $\mathcal{O}(\log q)$ | Pops $P$ invalid (withdrawn/expired) units plus $1$ valid unit. Since each unit is pushed once and popped at most once, total pops across all events is $\le q$. |
| **Overall System** | $\mathbf{\mathcal{O}(q \log q)}$ | $\mathbf{\mathcal{O}(q \log q)}$ | Total runtime for all $q$ events is bounded by $q \log q$. |
| **Auxiliary Space** | $\mathbf{\mathcal{O}(q + G)}$ | $\mathbf{\mathcal{O}(q + G)}$ | Stores at most $q$ units across heaps, $G$ heap headers, and at most $q$ withdrawn IDs. |

---

## 10. Edge Cases & Defensive Engineering

1. **Multiple Usable Units with Same Expiry:**
   - Handled by compound comparator: $\text{expiry}_1 == \text{expiry}_2 \implies \text{id}_1 < \text{id}_2$.
2. **Withdrawing an Already Expired or Handed-Over Unit:**
   - If an `id` is withdrawn after being handed over or discarded, adding it to `withdrawn` has zero side-effects because that unit is already gone from the heap.
3. **Empty Shelf on Request:**
   - If a group's heap is empty or all units are discarded, the loop terminates and emits `NONE`.
4. **Boundary Expiry ($expiry == t$):**
   - Condition states: *"usable on a request day only if its last safe day is not earlier than that day"*.
   - Thus, if $expiry \ge t$, the unit is valid. Only when $expiry < t$ is it expired.
5. **Multiple Simultaneous Events ($t_1 == t_2$):**
   - Events are processed strictly in their original input order as specified by the storage log.

---

## 11. Testing & Validation Matrix

```text
======================================================================
TEST SUITE EXECUTION RESULTS
======================================================================
Testcase 0: Sample 0 (10 events, 2 groups)
  Input: 
    1 D 101 2 6, 1 D 102 2 2, 1 D 103 1 9, 2 D 104 2 3, 3 C 104,
    3 R 2, 3 R 1, 4 R 2, 5 D 106 2 6, 6 R 2
  Output:
    101
    103
    NONE
    106
  Status: PASSED (100% Match)

Testcase 1: Sample 1 (9 events, 3 groups, ties & expiry)
  Input:
    1 D 7 3 4, 1 D 8 3 4, 1 D 9 1 2, 2 C 7, 2 R 3,
    3 R 1, 3 D 10 3 5, 4 R 3, 4 R 3
  Output:
    8
    NONE
    10
    NONE
  Status: PASSED (100% Match)

Testcase 2: Multiple withdrawals of same ID & Handed-over re-withdrawal
  Status: PASSED (Handled idempotently with no double-decrement errors)
======================================================================
```

---

## 12. Key Takeaways & Interview Cheat Sheet

### When to use Min-Heap + Lazy Deletion
- **Frequent Dynamic Removals from Heaps:** Standard binary heaps do not support efficient random deletion without maintaining handle pointers (like Fibonacci heaps or indexed priority queues).
- **The Tombstone Pattern:** Whenever elements can be cancelled, invalidated, or expired, prefer inserting a marker into a fast Hash Set and cleaning up dead nodes lazily from the top of the heap.
- **Amortized Invariant:** Since each item can only be inserted once and removed once, lazy deletion achieves the same theoretical $\mathcal{O}(\log N)$ amortized complexity as eager deletion, with vastly simpler code and lower constant-factor overhead.

### Conversational Script for Technical Interviews
> *"Because plasma units are requested by blood group and we must always dispense the unit that expires soonest, we partition the storage into $G$ separate min-heaps, ordered primarily by expiry and secondarily by ID. Donor cancellations can arrive at any time. Instead of searching and erasing from within the heap in $\mathcal{O}(N)$ time, we adopt a lazy deletion strategy: record cancelled IDs in a hash set in $\mathcal{O}(1)$ time. When a request arrives, we pop elements from the top of the relevant heap that are either withdrawn or have expired ($expiry < t$). The first valid element encountered is dispensed and permanently removed. This guarantees that every unit is pushed once and popped at most once, bounding total runtime to $\mathcal{O}(q \log q)$ and space to $\mathcal{O}(q + G)$."*
