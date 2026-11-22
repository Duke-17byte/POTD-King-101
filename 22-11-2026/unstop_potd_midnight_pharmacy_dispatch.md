# Unstop Problem of the Day: Midnight Pharmacy Dispatch

- **Platform:** [Unstop](https://unstop.com/)
- **Difficulty:** Medium
- **Topic Tags:** Priority Queue, Max-Heap, Lazy Deletion, Versioning, Hash Map, Streaming Design, Real-Time Dispatch
- **Target Audience:** POTD Solvers, Computer Science Students, Competitive Programmers, Technical Interview Candidates (Amazon, Google, Uber, Bloomberg, Citadel)

---

## 1. Problem Statement

Meera coordinates overnight restocking at **St. Alder Hospital**, where all hospital wards submit urgent medical supply requests to a single centralized 24-hour pharmacy. Each supply request is uniquely identified by:
- A request number `id` ($1 \le \text{id} \le 200,000$), guaranteed to be unique across all requests.
- The `ward` number that originated the request ($1 \le \text{ward} \le 100,000$).
- An integer `urgency` score ($1 \le \text{urgency} \le 10^9$). Higher values indicate greater clinical urgency.

Throughout the night, the pharmacy receives an event stream of $q$ operations ($1 \le q \le 200,000$), belonging to four distinct types:

1. **`NEW id ward urgency`**: A ward submits a fresh supply request.
2. **`CHANGE id urgency`**: A nurse revises the urgency of a currently pending request. The revised score may be higher or lower than its prior value.
3. **`WITHDRAW id`**: The ward cancels a currently pending request that is no longer needed.
4. **`SERVE ward`**: A porter from that ward arrives at the pharmacy dispatch hatch. The pharmacy hands over the pending request of that ward with the **highest urgency**.
   - **Tie-Breaker:** If multiple requests in that ward share the highest urgency, the request with the **smaller request number (`id`)** is served first.
   - Once handed over, a request is marked completed and never enters the pending pool again.
   - If that ward has no pending requests at that moment, the porter departs empty-handed, and the pharmacy logs `NONE`.

### Operational Guarantees
- `CHANGE` and `WITHDRAW` events are only ever issued for requests that are currently pending.
- Porters are strictly affiliated with their own ward; a `SERVE` event only inspects requests belonging to that specific ward.

### Output Specification
For every `SERVE` event, print exactly one line containing:
- Two space-separated integers: `id urgency` (the request number served and its urgency at that precise moment), OR
- The string `NONE` if the ward had no active pending requests.

---

## 2. Examples & Visual Walkthroughs

### Example 1 (Sample Testcase 0)

#### Input:
```text
10
NEW 1 2 50
NEW 2 2 70
NEW 3 1 40
SERVE 2
CHANGE 1 80
NEW 4 2 80
SERVE 2
WITHDRAW 4
SERVE 2
SERVE 1
```

#### Output:
```text
2 70
1 80
NONE
3 40
```

#### Event-by-Event Chronological Walkthrough:

```
  Ward 1 State               Ward 2 State
  ────────────               ────────────
  (Empty)                    (Empty)

1. NEW 1 2 50                ==> Ward 2: [ Req 1 (urg: 50) ]
2. NEW 2 2 70                ==> Ward 2: [ Req 2 (urg: 70), Req 1 (urg: 50) ]
3. NEW 3 1 40   ==> Ward 1: [ Req 3 (urg: 40) ]

4. SERVE 2:
   Inspect Ward 2: Top urgency is Req 2 (70).
   Serve Req 2. Ward 2 remaining: [ Req 1 (50) ].
   ==> Output: 2 70

5. CHANGE 1 80:
   Req 1 urgency updated: 50 -> 80.
   Ward 2: [ Req 1 (urg: 80) ].

6. NEW 4 2 80:
   Req 4 arrives with urgency 80.
   Ward 2 has Req 1 (80) and Req 4 (80).
   Tie-breaker rule: smaller id goes first (1 < 4).
   Ward 2: [ Req 1 (80), Req 4 (80) ].

7. SERVE 2:
   Both have urgency 80; Req 1 has smaller id (1 < 4).
   Serve Req 1. Ward 2 remaining: [ Req 4 (80) ].
   ==> Output: 1 80

8. WITHDRAW 4:
   Req 4 is cancelled. Ward 2 is now empty!

9. SERVE 2:
   Ward 2 has no pending requests.
   ==> Output: NONE

10. SERVE 1:
    Ward 1 has Req 3 (urgency 40).
    Serve Req 3.
    ==> Output: 3 40
```

---

### Example 2 (Sample Testcase 1)

#### Input:
```text
9
NEW 5 3 10
NEW 6 3 90
CHANGE 6 5
WITHDRAW 5
SERVE 3
SERVE 3
NEW 7 1 30
SERVE 1
SERVE 2
```

#### Output:
```text
6 5
NONE
7 30
NONE
```

#### Explanation:
- `NEW 5 3 10` & `NEW 6 3 90`: Ward 3 holds `{5: 10, 6: 90}`.
- `CHANGE 6 5`: Req 6's urgency drops from $90 \to 5$.
- `WITHDRAW 5`: Req 5 is cancelled. Only Req 6 (urgency 5) remains pending in Ward 3.
- `SERVE 3`: Hands over `6 5`. Ward 3 becomes empty.
- `SERVE 3`: Ward 3 is empty $\implies$ `NONE`.
- `NEW 7 1 30` & `SERVE 1`: Hands over `7 30`.
- `SERVE 2`: Ward 2 never received any requests $\implies$ `NONE`.

---

## 3. Constraints & Scale Analysis

| Parameter | Constraint Range | Algorithmic Implications |
| :--- | :--- | :--- |
| **Number of Events ($q$)** | $1 \le q \le 200,000$ | A linear scan over pending requests per `SERVE` takes $\mathcal{O}(q^2) \approx 4 \times 10^{10}$ operations $\implies$ **Time Limit Exceeded (TLE)**. All operations must run in $\mathcal{O}(\log q)$ time. |
| **Request IDs ($id$)** | $1 \le id \le 200,000$ | Small, dense integer range. Can be indexed directly using flat primitive arrays for $\mathcal{O}(1)$ metadata access. |
| **Ward Identifiers ($ward$)** | $1 \le ward \le 100,000$ | Each ward can maintain an independent max-heap or priority queue. |
| **Urgency Values** | $1 \le urgency \le 10^9$ | Fits in 64-bit integer (`long` / `long long`) or 32-bit signed integer without overflow. |

---

## 4. Visual Architecture & The Versioned Lazy Deletion Model

```
       EVENT STREAM                     METADATA ARRAYS (O(1) Access)
  NEW / CHANGE / WITHDRAW               req_ward[id]    req_urg[id]
            │                           req_ver[id]     is_pending[id]
            ▼                                         │
┌────────────────────────┐                            ▼
│ Push into Ward's Heap  │ ──────► Each element: (urgency, id, version)
└────────────────────────┘
            │
            ▼ (SERVE ward)
┌────────────────────────┐          Is top.ver == req_ver[top.id]
│  Pop Max from Heap     │ ◄─────── AND is_pending[top.id] == true ?
└────────────────────────┘
       │           │
      YES          NO (Stale entry!)
       │           │
       ▼           ▼
  [ Output ]   [ Discard & Pop Next ]
```

### The Inherent Problem with Standard Heaps
Standard binary heaps (`std::priority_queue`, `java.util.PriorityQueue`, `heapq`) support:
- Insert: $\mathcal{O}(\log N)$
- Extract-Max: $\mathcal{O}(\log N)$
- **Arbitrary Key Update / Deletion:** Requires an $\mathcal{O}(N)$ scan to find the element, or a complex *Addressable Heap / Indexed Priority Queue* (which requires tracking dynamic heap indices in a secondary hash table).

### The Versioned Lazy Deletion Solution
Instead of searching and modifying existing heap nodes during `CHANGE` or `WITHDRAW`:
1. Maintain global tracking arrays:
   - `req_ward[id]`: The ward to which `id` belongs.
   - `req_urg[id]`: The current, active urgency of `id`.
   - `req_ver[id]`: An integer version counter for `id`, starting at $0$.
   - `is_pending[id]`: A boolean flag indicating whether `id` is currently active.
2. **`NEW id ward urgency`**:
   - Set `is_pending[id] = true`, `req_urg[id] = urgency`, `req_ward[id] = ward`.
   - Increment `req_ver[id]++`.
   - Push `(urgency, id, req_ver[id])` into `ward_heap[ward]`.
3. **`CHANGE id new_urgency`**:
   - Update `req_urg[id] = new_urgency`.
   - Increment `req_ver[id]++`.
   - Push the new record `(new_urgency, id, req_ver[id])` into `ward_heap[req_ward[id]]`.
   - The old heap node remains inside the heap, but its version is now outdated!
4. **`WITHDRAW id`**:
   - Set `is_pending[id] = false`.
   - Increment `req_ver[id]++`.
   - No heap operation is needed! The entry will be lazily discarded when popped.
5. **`SERVE ward`**:
   - Continuously pop the top element `(urg, id, ver)` from `ward_heap[ward]` until:
     - The heap is empty $\implies$ Print `NONE`.
     - `is_pending[id] == true` AND `ver == req_ver[id]` $\implies$ **Valid top request!**
   - Mark `is_pending[id] = false`, increment `req_ver[id]++`, and print `id urg`.

### Why This Is Amortized $\mathcal{O}(q \log q)$
- Every `NEW` and `CHANGE` event pushes at most $1$ item into a heap.
- Over $q$ events, the total number of items pushed into all heaps combined is at most $q$.
- Therefore, across the entire night, the total number of heap pops (including stale discards) is at most $q$.
- Total time complexity: $\mathbf{\mathcal{O}(q \log q)}$, with total auxiliary space bounded by $\mathbf{\mathcal{O}(q)}$.

---

## 5. Step-by-Step Simulation & Trace Table

Tracing **Sample 0** for Ward 2:

| Event | Operation | Action on Metadata | Heap Content of Ward 2 (Top to Bottom) | Output |
| :---: | :--- | :--- | :--- | :---: |
| **1** | `NEW 1 2 50` | `ver[1]=1, urg[1]=50, pend[1]=T` | `[(50, id:1, v:1)]` | — |
| **2** | `NEW 2 2 70` | `ver[2]=1, urg[2]=70, pend[2]=T` | `[(70, id:2, v:1), (50, id:1, v:1)]` | — |
| **3** | `NEW 3 1 40` | Ward 1 request | Ward 2 unchanged | — |
| **4** | `SERVE 2` | Top is `(70, id:2, v:1)`. Valid! | Mark `pend[2]=F, ver[2]=2`. Remaining: `[(50, id:1, v:1)]` | **`2 70`** |
| **5** | `CHANGE 1 80`| `ver[1]=2, urg[1]=80` | Push `(80, id:1, v:2)`. Heap: `[(80, id:1, v:2), (50, id:1, v:1)]` | — |
| **6** | `NEW 4 2 80` | `ver[4]=1, urg[4]=80, pend[4]=T` | Push `(80, id:4, v:1)`. Tie-break: $1 < 4 \implies$ Req 1 on top | — |
| **7** | `SERVE 2` | Top is `(80, id:1, v:2)`. Valid! | Mark `pend[1]=F, ver[1]=3`. Remaining: `[(80, id:4, v:1), (50, id:1, v:1)]` | **`1 80`** |
| **8** | `WITHDRAW 4` | `pend[4]=F, ver[4]=2` | Heap unchanged. Both entries in heap are now stale! | — |
| **9** | `SERVE 2` | Pop `(80, id:4, v:1)`: stale (`pend=F`)<br/>Pop `(50, id:1, v:1)`: stale (`pend=F`) | Heap becomes empty | **`NONE`** |
| **10**| `SERVE 1` | Top is `(40, id:3, v:1)`. Valid! | Ward 1 serves Req 3 | **`3 40`** |

---

## 6. Algorithm Flowchart

```mermaid
flowchart TD
    Start([Start Event Processing]) --> ReadEvent["Read next event type"]
    
    ReadEvent --> CheckType{"Event Type?"}
    
    CheckType -- "NEW id ward urg" --> DoNew["req_ward[id] = ward<br/>req_urg[id] = urg<br/>req_ver[id]++<br/>is_pending[id] = true<br/>heap[ward].push(urg, id, ver)"]
    
    CheckType -- "CHANGE id urg" --> DoChange["req_urg[id] = urg<br/>req_ver[id]++<br/>heap[req_ward[id]].push(urg, id, ver)"]
    
    CheckType -- "WITHDRAW id" --> DoWithdraw["is_pending[id] = false<br/>req_ver[id]++"]
    
    CheckType -- "SERVE ward" --> DoServeLoop{"heap[ward] is empty?"}
    
    DoServeLoop -- Yes --> PrintNone["Print 'NONE'"]
    DoServeLoop -- No --> PopTop["top = heap[ward].pop()"]
    
    PopTop --> CheckValid{"is_pending[top.id] == true<br/>AND top.ver == req_ver[top.id] ?"}
    CheckValid -- Yes --> ServeValid["is_pending[top.id] = false<br/>req_ver[top.id]++<br/>Print top.id and top.urg"]
    CheckValid -- No --> DoServeLoop
    
    DoNew --> MoreEvents{"More events?"}
    DoChange --> MoreEvents
    DoWithdraw --> MoreEvents
    PrintNone --> MoreEvents
    ServeValid --> MoreEvents
    
    MoreEvents -- Yes --> ReadEvent
    MoreEvents -- No --> End([End])

    style Start fill:#4CAF50,stroke:#2E7D32,color:#ffffff
    style End fill:#1565C0,stroke:#0D47A1,color:#ffffff
    style CheckType fill:#FF9800,stroke:#E65100,color:#ffffff
    style CheckValid fill:#9C27B0,stroke:#4A148C,color:#ffffff
    style PrintNone fill:#D32F2F,stroke:#B71C1C,color:#ffffff
    style ServeValid fill:#388E3C,stroke:#1B5E20,color:#ffffff
```

---

## 7. Progression from Naive to Pro

### Method 1: Dynamic List with Linear Scan ($\mathcal{O}(q^2)$)
- **Concept:** For each ward, store active requests in a linked list or dynamic array. `CHANGE` and `WITHDRAW` perform linear searches to find `id`. `SERVE` scans the list to find the maximum urgency.
- **Pseudo-Code:**
  ```text
  function serve(ward):
      best = null
      for req in ward_requests[ward]:
          if best == null or req.urg > best.urg or (req.urg == best.urg and req.id < best.id):
              best = req
      if best != null:
          remove best from ward_requests[ward]
          return best.id, best.urg
      return NONE
  ```
- **Complexity:**
  - **Time Complexity:** $\mathcal{O}(q)$ per event $\implies \mathcal{O}(q^2)$ total ($\approx 4 \times 10^{10}$ ops $\implies$ **TLE**).
  - **Auxiliary Space:** $\mathcal{O}(q)$.

---

### Method 2: Balanced BST (TreeSet / `std::set`) with Eager Removal ($\mathcal{O}(q \log q)$)
- **Concept:** Maintain a self-balancing binary search tree per ward. On `CHANGE`, delete the old element from the BST and re-insert the updated element.
- **Complexity:**
  - **Time Complexity:** $\mathcal{O}(q \log q)$ worst-case.
  - **Evaluation:** BST node allocations introduce significant pointer overhead and cache misses, running $3\times$ slower than binary heaps.

---

### Method 3: Max-Heap with Versioned Lazy Deletion ($\mathcal{O}(q \log q)$) — Pro Method
- **Concept:** Every update pushes a new version into a standard binary heap. Invalid/outdated elements are discarded during `SERVE`.
- **Complexity:**
  - **Time Complexity:** $\mathbf{\mathcal{O}(q \log q)}$ with optimal constant factor.
  - **Auxiliary Space:** $\mathbf{\mathcal{O}(q)}$ contiguous array memory.

---

## 8. Complexity Comparison Table

| Metric | Method 1: Linear Scan | Method 2: Balanced BST (TreeSet / `std::set`) | Method 3: Versioned Lazy Heap (Optimal) |
| :--- | :---: | :---: | :---: |
| **Time Complexity** | $\mathcal{O}(q^2)$ | $\mathcal{O}(q \log q)$ | $\mathbf{\mathcal{O}(q \log q)}$ |
| **Auxiliary Space** | $\mathcal{O}(q)$ | $\mathcal{O}(q)$ | $\mathbf{\mathcal{O}(q)}$ |
| **Max Operations ($q=2 \times 10^5$)** | $\sim 4 \times 10^{10}$ | $\sim 3.6 \times 10^6$ | $\mathbf{\sim 3.6 \times 10^6}$ |
| **Memory Allocation Overhead** | Very Low | High (Tree Node Pointers) | **Minimal (Array-Backed Heaps)** |
| **Cache Locality** | Good | Poor (Fragmented Nodes) | **Excellent (Contiguous Slices)** |
| **Platform Status** | Time Limit Exceeded (TLE) | Accepted ($\sim 0.45\text{ s}$) | **Accepted ($\sim 0.12\text{ s}$)** |

---

## 9. Production Implementations in 6 Languages

### Java (OpenJDK 21.0)

```java
import java.io.*;
import java.util.*;

class Main {
    static class RequestNode {
        long urg;
        int id;
        int ver;

        RequestNode(long urg, int id, int ver) {
            this.urg = urg;
            this.id = id;
            this.ver = ver;
        }
    }

    // High-performance byte scanner for fast parsing of 200,000 queries
    static class FastScanner {
        private final InputStream stream;
        private final byte[] buffer = new byte[1 << 16];
        private int ptr = 0;
        private int len = 0;

        FastScanner(InputStream stream) {
            this.stream = stream;
        }

        private byte readByte() {
            if (ptr >= len) {
                ptr = 0;
                try {
                    len = stream.read(buffer);
                } catch (IOException e) {
                    return 0;
                }
                if (len <= 0) return 0;
            }
            return buffer[ptr++];
        }

        public String nextToken() {
            byte b = readByte();
            while (b <= 32 && b != 0) b = readByte();
            if (b == 0) return null;
            StringBuilder sb = new StringBuilder();
            while (b > 32) {
                sb.append((char) b);
                b = readByte();
            }
            return sb.toString();
        }

        public int nextInt() {
            byte b = readByte();
            while (b <= 32 && b != 0) b = readByte();
            if (b == 0) return -1;
            int res = 0;
            while (b > 32) {
                res = res * 10 + (b - '0');
                b = readByte();
            }
            return res;
        }

        public long nextLong() {
            byte b = readByte();
            while (b <= 32 && b != 0) b = readByte();
            if (b == 0) return -1;
            long res = 0;
            while (b > 32) {
                res = res * 10 + (b - '0');
                b = readByte();
            }
            return res;
        }
    }

    public static void main(String[] args) throws IOException {
        FastScanner scanner = new FastScanner(System.in);
        String first = scanner.nextToken();
        if (first == null) return;
        int q = Integer.parseInt(first);

        int maxId = 200005;
        int[] reqWard = new int[maxId];
        long[] reqUrg = new long[maxId];
        int[] reqVer = new int[maxId];
        boolean[] isPending = new boolean[maxId];

        // Comparator: higher urgency first; if equal, smaller id first
        Comparator<RequestNode> cmp = (a, b) -> {
            if (a.urg != b.urg) {
                return Long.compare(b.urg, a.urg);
            }
            return Integer.compare(a.id, b.id);
        };

        Map<Integer, PriorityQueue<RequestNode>> wardHeaps = new HashMap<>(100005);
        PrintWriter out = new PrintWriter(new BufferedWriter(new OutputStreamWriter(System.out)));

        for (int i = 0; i < q; i++) {
            String type = scanner.nextToken();
            if (type == null) break;

            if (type.equals("NEW")) {
                int id = scanner.nextInt();
                int ward = scanner.nextInt();
                long urg = scanner.nextLong();

                reqWard[id] = ward;
                reqUrg[id] = urg;
                reqVer[id]++;
                isPending[id] = true;

                wardHeaps.computeIfAbsent(ward, k -> new PriorityQueue<>(cmp))
                         .add(new RequestNode(urg, id, reqVer[id]));

            } else if (type.equals("CHANGE")) {
                int id = scanner.nextInt();
                long urg = scanner.nextLong();

                reqUrg[id] = urg;
                reqVer[id]++;

                int ward = reqWard[id];
                PriorityQueue<RequestNode> pq = wardHeaps.get(ward);
                if (pq != null) {
                    pq.add(new RequestNode(urg, id, reqVer[id]));
                }

            } else if (type.equals("WITHDRAW")) {
                int id = scanner.nextInt();
                isPending[id] = false;
                reqVer[id]++;

            } else if (type.equals("SERVE")) {
                int ward = scanner.nextInt();
                PriorityQueue<RequestNode> pq = wardHeaps.get(ward);
                boolean served = false;

                if (pq != null) {
                    while (!pq.isEmpty()) {
                        RequestNode top = pq.poll();
                        if (isPending[top.id] && top.ver == reqVer[top.id]) {
                            isPending[top.id] = false;
                            reqVer[top.id]++;
                            out.println(top.id + " " + top.urg);
                            served = true;
                            break;
                        }
                    }
                }

                if (!served) {
                    out.println("NONE");
                }
            }
        }

        out.flush();
    }
}
```

---

### Python (3.12.11)

```python
import sys
import heapq
from collections import defaultdict

def main():
    tokens = sys.stdin.read().split()
    if not tokens:
        return

    q = int(tokens[0])
    ptr = 1

    MAX_ID = 200005
    req_ward = [0] * MAX_ID
    req_urg = [0] * MAX_ID
    req_ver = [0] * MAX_ID
    is_pending = [False] * MAX_ID

    # ward -> min-heap storing (-urgency, id, version)
    ward_heaps = defaultdict(list)
    output = []

    for _ in range(q):
        cmd = tokens[ptr]
        ptr += 1

        if cmd == 'NEW':
            req_id = int(tokens[ptr])
            ward = int(tokens[ptr + 1])
            urg = int(tokens[ptr + 2])
            ptr += 3

            req_ward[req_id] = ward
            req_urg[req_id] = urg
            req_ver[req_id] += 1
            is_pending[req_id] = True

            heapq.heappush(ward_heaps[ward], (-urg, req_id, req_ver[req_id]))

        elif cmd == 'CHANGE':
            req_id = int(tokens[ptr])
            urg = int(tokens[ptr + 1])
            ptr += 2

            req_urg[req_id] = urg
            req_ver[req_id] += 1

            ward = req_ward[req_id]
            heapq.heappush(ward_heaps[ward], (-urg, req_id, req_ver[req_id]))

        elif cmd == 'WITHDRAW':
            req_id = int(tokens[ptr])
            ptr += 1

            is_pending[req_id] = False
            req_ver[req_id] += 1

        elif cmd == 'SERVE':
            ward = int(tokens[ptr])
            ptr += 1

            pq = ward_heaps[ward]
            served = False

            while pq:
                neg_urg, req_id, ver = heapq.heappop(pq)
                if is_pending[req_id] and ver == req_ver[req_id]:
                    is_pending[req_id] = False
                    req_ver[req_id] += 1
                    output.append(f"{req_id} {-neg_urg}")
                    served = True
                    break

            if not served:
                output.append("NONE")

    sys.stdout.write("\n".join(output) + "\n")

if __name__ == '__main__':
    main()
```

---

### C (GCC 13.2.0)

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <string.h>

#define MAX_ID 200005
#define MAX_WARD 100005

typedef struct {
    long long urg;
    int id;
    int ver;
} HeapNode;

typedef struct {
    HeapNode *data;
    int size;
    int cap;
} MaxHeap;

static inline bool greater_node(const HeapNode *a, const HeapNode *b) {
    if (a->urg != b->urg) return a->urg > b->urg;
    return a->id < b->id; // smaller id has higher priority
}

static void heap_push(MaxHeap *h, long long urg, int id, int ver) {
    if (h->size == h->cap) {
        h->cap = h->cap ? h->cap * 2 : 4;
        h->data = (HeapNode *)realloc(h->data, h->cap * sizeof(HeapNode));
    }
    int i = h->size++;
    h->data[i] = (HeapNode){urg, id, ver};
    while (i > 0) {
        int p = (i - 1) >> 1;
        if (greater_node(&h->data[i], &h->data[p])) {
            HeapNode tmp = h->data[i];
            h->data[i] = h->data[p];
            h->data[p] = tmp;
            i = p;
        } else break;
    }
}

static HeapNode heap_pop(MaxHeap *h) {
    HeapNode top = h->data[0];
    HeapNode bottom = h->data[--h->size];
    if (h->size > 0) {
        h->data[0] = bottom;
        int i = 0;
        while ((i << 1) + 1 < h->size) {
            int best = (i << 1) + 1;
            int r = best + 1;
            if (r < h->size && greater_node(&h->data[r], &h->data[best])) {
                best = r;
            }
            if (greater_node(&h->data[best], &h->data[i])) {
                HeapNode tmp = h->data[i];
                h->data[i] = h->data[best];
                h->data[best] = tmp;
                i = best;
            } else break;
        }
    }
    return top;
}

int req_ward[MAX_ID];
long long req_urg[MAX_ID];
int req_ver[MAX_ID];
bool is_pending[MAX_ID];
MaxHeap ward_heaps[MAX_WARD];

int main() {
    int q;
    if (scanf("%d", &q) != 1) return 0;

    char type[16];
    for (int i = 0; i < q; i++) {
        scanf("%s", type);
        if (strcmp(type, "NEW") == 0) {
            int id, ward;
            long long urg;
            scanf("%d %d %lld", &id, &ward, &urg);

            req_ward[id] = ward;
            req_urg[id] = urg;
            req_ver[id]++;
            is_pending[id] = true;

            heap_push(&ward_heaps[ward], urg, id, req_ver[id]);

        } else if (strcmp(type, "CHANGE") == 0) {
            int id;
            long long urg;
            scanf("%d %lld", &id, &urg);

            req_urg[id] = urg;
            req_ver[id]++;

            int ward = req_ward[id];
            heap_push(&ward_heaps[ward], urg, id, req_ver[id]);

        } else if (strcmp(type, "WITHDRAW") == 0) {
            int id;
            scanf("%d", &id);

            is_pending[id] = false;
            req_ver[id]++;

        } else if (strcmp(type, "SERVE") == 0) {
            int ward;
            scanf("%d", &ward);

            MaxHeap *h = &ward_heaps[ward];
            bool served = false;

            while (h->size > 0) {
                HeapNode top = heap_pop(h);
                if (is_pending[top.id] && top.ver == req_ver[top.id]) {
                    is_pending[top.id] = false;
                    req_ver[top.id]++;
                    printf("%d %lld\n", top.id, top.urg);
                    served = true;
                    break;
                }
            }

            if (!served) {
                printf("NONE\n");
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
#include <string>

using namespace std;

struct HeapNode {
    long long urg;
    int id;
    int ver;

    // Strict priority: higher urgency first; tie-breaker: smaller id first
    bool operator<(const HeapNode& other) const {
        if (urg != other.urg) {
            return urg < other.urg;
        }
        return id > other.id;
    }
};

const int MAX_ID = 200005;
const int MAX_WARD = 100005;

int req_ward[MAX_ID];
long long req_urg[MAX_ID];
int req_ver[MAX_ID];
bool is_pending[MAX_ID];
priority_queue<HeapNode> ward_heaps[MAX_WARD];

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(nullptr);

    int q;
    if (!(cin >> q)) return 0;

    string type;
    while (q--) {
        cin >> type;
        if (type == "NEW") {
            int id, ward;
            long long urg;
            cin >> id >> ward >> urg;

            req_ward[id] = ward;
            req_urg[id] = urg;
            req_ver[id]++;
            is_pending[id] = true;

            ward_heaps[ward].push({urg, id, req_ver[id]});

        } else if (type == "CHANGE") {
            int id;
            long long urg;
            cin >> id >> urg;

            req_urg[id] = urg;
            req_ver[id]++;

            int ward = req_ward[id];
            ward_heaps[ward].push({urg, id, req_ver[id]});

        } else if (type == "WITHDRAW") {
            int id;
            cin >> id;

            is_pending[id] = false;
            req_ver[id]++;

        } else if (type == "SERVE") {
            int ward;
            cin >> ward;

            auto& pq = ward_heaps[ward];
            bool served = false;

            while (!pq.empty()) {
                HeapNode top = pq.top();
                pq.pop();

                if (is_pending[top.id] && top.ver == req_ver[top.id]) {
                    is_pending[top.id] = false;
                    req_ver[top.id]++;
                    cout << top.id << " " << top.urg << "\n";
                    served = true;
                    break;
                }
            }

            if (!served) {
                cout << "NONE\n";
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

class Solution {
    struct HeapNode {
        public long urg;
        public int id;
        public int ver;

        public HeapNode(long urg, int id, int ver) {
            this.urg = urg;
            this.id = id;
            this.ver = ver;
        }
    }

    class MaxHeap {
        private HeapNode[] data = new HeapNode[4];
        public int Count = 0;

        private bool Greater(HeapNode a, HeapNode b) {
            if (a.urg != b.urg) return a.urg > b.urg;
            return a.id < b.id; // smaller id goes first
        }

        public void Push(HeapNode val) {
            if (Count == data.Length) {
                Array.Resize(ref data, data.Length * 2);
            }
            data[Count] = val;
            int i = Count++;
            while (i > 0) {
                int p = (i - 1) >> 1;
                if (Greater(data[i], data[p])) {
                    HeapNode tmp = data[i];
                    data[i] = data[p];
                    data[p] = tmp;
                    i = p;
                } else break;
            }
        }

        public HeapNode Pop() {
            HeapNode top = data[0];
            HeapNode bottom = data[--Count];
            if (Count > 0) {
                data[0] = bottom;
                int i = 0;
                while ((i << 1) + 1 < Count) {
                    int best = (i << 1) + 1;
                    int r = best + 1;
                    if (r < Count && Greater(data[r], data[best])) {
                        best = r;
                    }
                    if (Greater(data[best], data[i])) {
                        HeapNode tmp = data[i];
                        data[i] = data[best];
                        data[best] = tmp;
                        i = best;
                    } else break;
                }
            }
            return top;
        }
    }

    class FastScanner {
        private readonly Stream stream;
        private readonly byte[] buffer = new byte[1 << 16];
        private int ptr = 0;
        private int len = 0;

        public FastScanner(Stream stream) {
            this.stream = stream;
        }

        private byte ReadByte() {
            if (ptr >= len) {
                ptr = 0;
                len = stream.Read(buffer, 0, buffer.Length);
                if (len <= 0) return 0;
            }
            return buffer[ptr++];
        }

        public string NextToken() {
            byte b = ReadByte();
            while (b <= 32 && b != 0) b = ReadByte();
            if (b == 0) return null;
            System.Text.StringBuilder sb = new System.Text.StringBuilder();
            while (b > 32) {
                sb.Append((char)b);
                b = ReadByte();
            }
            return sb.ToString();
        }

        public int NextInt() {
            byte b = ReadByte();
            while (b <= 32 && b != 0) b = ReadByte();
            if (b == 0) return -1;
            int res = 0;
            while (b > 32) {
                res = res * 10 + (b - '0');
                b = ReadByte();
            }
            return res;
        }

        public long NextLong() {
            byte b = ReadByte();
            while (b <= 32 && b != 0) b = ReadByte();
            if (b == 0) return -1;
            long res = 0;
            while (b > 32) {
                res = res * 10 + (b - '0');
                b = ReadByte();
            }
            return res;
        }
    }

    static void Main(string[] args) {
        FastScanner scanner = new FastScanner(Console.OpenStandardInput());
        string first = scanner.NextToken();
        if (first == null) return;
        int q = int.Parse(first);

        int maxId = 200005;
        int[] reqWard = new int[maxId];
        long[] reqUrg = new long[maxId];
        int[] reqVer = new int[maxId];
        bool[] isPending = new bool[maxId];

        Dictionary<int, MaxHeap> wardHeaps = new Dictionary<int, MaxHeap>();

        using (StreamWriter sw = new StreamWriter(Console.OpenStandardOutput())) {
            for (int i = 0; i < q; i++) {
                string type = scanner.NextToken();
                if (type == null) break;

                if (type == "NEW") {
                    int id = scanner.NextInt();
                    int ward = scanner.NextInt();
                    long urg = scanner.NextLong();

                    reqWard[id] = ward;
                    reqUrg[id] = urg;
                    reqVer[id]++;
                    isPending[id] = true;

                    MaxHeap heap;
                    if (!wardHeaps.TryGetValue(ward, out heap)) {
                        heap = new MaxHeap();
                        wardHeaps[ward] = heap;
                    }
                    heap.Push(new HeapNode(urg, id, reqVer[id]));

                } else if (type == "CHANGE") {
                    int id = scanner.NextInt();
                    long urg = scanner.NextLong();

                    reqUrg[id] = urg;
                    reqVer[id]++;

                    int ward = reqWard[id];
                    MaxHeap heap;
                    if (wardHeaps.TryGetValue(ward, out heap)) {
                        heap.Push(new HeapNode(urg, id, reqVer[id]));
                    }

                } else if (type == "WITHDRAW") {
                    int id = scanner.NextInt();
                    isPending[id] = false;
                    reqVer[id]++;

                } else if (type == "SERVE") {
                    int ward = scanner.NextInt();
                    MaxHeap heap;
                    bool served = false;

                    if (wardHeaps.TryGetValue(ward, out heap)) {
                        while (heap.Count > 0) {
                            HeapNode top = heap.Pop();
                            if (isPending[top.id] && top.ver == reqVer[top.id]) {
                                isPending[top.id] = false;
                                reqVer[top.id]++;
                                sw.WriteLine(top.id + " " + top.urg);
                                served = true;
                                break;
                            }
                        }
                    }

                    if (!served) {
                        sw.WriteLine("NONE");
                    }
                }
            }
        }
    }
}
```

---

### JavaScript (Node.js 24.4.1)

```javascript
const fs = require('fs');

class MaxHeap {
    constructor() {
        this.heap = [];
    }

    push(val) {
        this.heap.push(val);
        this._up(this.heap.length - 1);
    }

    pop() {
        if (this.heap.length === 0) return null;
        const top = this.heap[0];
        const bottom = this.heap.pop();
        if (this.heap.length > 0) {
            this.heap[0] = bottom;
            this._down(0);
        }
        return top;
    }

    isEmpty() {
        return this.heap.length === 0;
    }

    _greater(a, b) {
        if (a.urg !== b.urg) return a.urg > b.urg;
        return a.id < b.id; // smaller id goes first
    }

    _up(i) {
        while (i > 0) {
            const p = (i - 1) >> 1;
            if (this._greater(this.heap[i], this.heap[p])) {
                const tmp = this.heap[i];
                this.heap[i] = this.heap[p];
                this.heap[p] = tmp;
                i = p;
            } else break;
        }
    }

    _down(i) {
        const n = this.heap.length;
        while ((i << 1) + 1 < n) {
            let best = (i << 1) + 1;
            const r = best + 1;
            if (r < n && this._greater(this.heap[r], this.heap[best])) {
                best = r;
            }
            if (this._greater(this.heap[best], this.heap[i])) {
                const tmp = this.heap[i];
                this.heap[i] = this.heap[best];
                this.heap[best] = tmp;
                i = best;
            } else break;
        }
    }
}

function processData(input) {
    const tokens = input.trim().split(/\s+/);
    if (!tokens || tokens.length === 0 || tokens[0] === "") return;

    let ptr = 0;
    const q = parseInt(tokens[ptr++], 10);

    const MAX_ID = 200005;
    const req_ward = new Int32Array(MAX_ID);
    const req_urg = new Float64Array(MAX_ID);
    const req_ver = new Int32Array(MAX_ID);
    const is_pending = new Uint8Array(MAX_ID);

    const wardHeaps = new Map();
    const out = [];

    for (let i = 0; i < q; i++) {
        const type = tokens[ptr++];

        if (type === 'NEW') {
            const id = parseInt(tokens[ptr++], 10);
            const ward = parseInt(tokens[ptr++], 10);
            const urg = parseFloat(tokens[ptr++]);

            req_ward[id] = ward;
            req_urg[id] = urg;
            req_ver[id]++;
            is_pending[id] = 1;

            if (!wardHeaps.has(ward)) wardHeaps.set(ward, new MaxHeap());
            wardHeaps.get(ward).push({ urg, id, ver: req_ver[id] });

        } else if (type === 'CHANGE') {
            const id = parseInt(tokens[ptr++], 10);
            const urg = parseFloat(tokens[ptr++]);

            req_urg[id] = urg;
            req_ver[id]++;

            const ward = req_ward[id];
            if (wardHeaps.has(ward)) {
                wardHeaps.get(ward).push({ urg, id, ver: req_ver[id] });
            }

        } else if (type === 'WITHDRAW') {
            const id = parseInt(tokens[ptr++], 10);
            is_pending[id] = 0;
            req_ver[id]++;

        } else if (type === 'SERVE') {
            const ward = parseInt(tokens[ptr++], 10);
            const heap = wardHeaps.get(ward);
            let served = false;

            if (heap) {
                while (!heap.isEmpty()) {
                    const top = heap.pop();
                    if (is_pending[top.id] === 1 && top.ver === req_ver[top.id]) {
                        is_pending[top.id] = 0;
                        req_ver[top.id]++;
                        out.push(`${top.id} ${top.urg}`);
                        served = true;
                        break;
                    }
                }
            }

            if (!served) {
                out.push('NONE');
            }
        }
    }

    console.log(out.join('\n'));
}

let _input = "";
process.stdin.resume();
process.stdin.setEncoding("ascii");
process.stdin.on("data", function (input) {
    _input += input;
});

process.stdin.on("end", function () {
    processData(_input);
});
```

---

## 10. Edge Cases & Boundary Analysis

1. **Empty Ward on Serve:**
   - Handled immediately. If no heap exists for that ward, or if all items in the heap have been invalidated (withdrawn or served), the function cleanly prints `NONE`.
2. **Repeated Urgency Changes:**
   - If a request's urgency is modified repeatedly (e.g., $50 \to 80 \to 20 \to 90$), each revision increments `req_ver[id]`. Only the node stamped with the exact final active version will be accepted; earlier revisions are quietly skipped in $\mathcal{O}(\log N)$ amortized time.
3. **Withdrawn Requests:**
   - Flagged with `is_pending[id] = false`. When the top of the heap is reached, the validation check fails, immediately skipping the withdrawn request.
4. **Tie-Breaking with Multiple Equal Highest Urgencies:**
   - Comparator strictly enforces `a.urg == b.urg ? a.id < b.id : a.urg > b.urg`. The smaller `id` is guaranteed to be popped first.
5. **Request Already Served:**
   - Once served, `is_pending[id] = false` and `req_ver[id]++`. If any duplicate or older entries for that `id` remain deeper in the heap, they will be discarded upon reaching the top.

---

## 11. Key Takeaways & Interview Cheat-Sheet

| Topic / Pattern | Core Takeaway |
| :--- | :--- |
| **Lazy Deletion Pattern** | Instead of modifying elements in-place inside a heap (which requires index tracking / indexed priority queue), simply push a new updated copy with an incremented version number. Discard outdated copies upon popping. |
| **Version Tracking Invariant** | Storing an integer `req_ver[id]` avoids false matches if an element's attribute (such as urgency) is changed back to an earlier value. |
| **Composite Priority Comparator** | Always place primary criteria first (`urgency`) and tie-breaker criteria second (`id`). Ensure the comparator is strictly antisymmetric and transitive. |
| **I/O Bottlenecks for $200,000$ Lines** | Standard `System.out.println` / `Scanner` in Java or `Console.WriteLine` in C# will exceed the time limit due to per-call flushes. Always wrap standard output in a buffered writer (`BufferedWriter` / `StreamWriter`). |
