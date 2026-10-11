# Unstop Problem of the Day: The Conservation Queue

- **Platform:** [Unstop](https://unstop.com/)
- **Difficulty:** Medium
- **Topic Tags:** Priority Queue, Binary Max-Heap, Hash Set, Lazy Deletion (Tombstoning), Event Simulation, Amortized Analysis
- **Target Audience:** POTD Solvers, Computer Science Students, Competitive Programmers, Interview Preparation (FAANG / Tier-1 Systems Engineering)

---

## 1. Problem Statement

Meera runs the conservation lab at the Bengal Heritage Museum, where fragile and damaged historical artifacts arrive for restorative treatment before they decay beyond repair.

Each artifact arrives with:
1. A unique **catalogue number** (`id`).
2. An **urgency rating** (`urgency`) reflecting how rapidly the artifact is deteriorating.

The conservation lab operates under strict operational constraints:
- **Single-Item Processing:** The lab can restore only **one artifact at a time**.
- **Priority Selection:** Whenever a restoration slot opens up, Meera always chooses the artifact with the **highest urgency rating** currently waiting in her lab.
- **Deterministic Tie-Breaking:** If two or more waiting artifacts share the exact same urgency rating, she picks the one with the **smaller catalogue number** (`id`), because it was catalogued earlier in the museum's registry.
- **Owner Withdrawals:** Occasionally, before an artifact is selected for restoration, its lending institution recalls it (e.g., for an emergency exhibition elsewhere). When this occurs, the artifact is **withdrawn from consideration immediately** and must **never** be restored, even though it had physically arrived earlier.

Meera's assistant logs all occurrences in real time as a sequence of $M$ sequential events:
1. `ARRIVE id urgency`: A new artifact enters the queue.
2. `WITHDRAW id`: An artifact currently waiting in the queue is withdrawn by its lender.
3. `RESTORE`: Meera finishes her current work and initiates restoration on the most urgent eligible waiting artifact.

Meera needs a program to replay this chronological log and, for each `RESTORE` event, output the catalogue number of the artifact restored, or output **`IDLE`** if no eligible artifact was waiting in the lab at that moment.

---

## 2. Input & Output Formats

### Input Format
- The first line contains an integer $M$, the total number of events.
- Each of the subsequent $M$ lines contains one event in one of the following three formats:
  - `ARRIVE id urgency`
  - `WITHDRAW id`
  - `RESTORE`

### Output Format
- For every `RESTORE` event, print a single line containing either:
  - The `id` of the restored artifact, OR
  - The string `IDLE` if no eligible artifacts are waiting in the lab.

---

## 3. Sample Testcases & Walkthroughs

### Sample Testcase 0

#### Input
```text
6
ARRIVE 101 50
ARRIVE 102 80
RESTORE
ARRIVE 103 90
WITHDRAW 101
RESTORE
```

#### Output
```text
102
103
```

#### Chronological Walkthrough

```text
Event 1: ARRIVE 101 50
  -> Queue has: [(101, urgency=50)]

Event 2: ARRIVE 102 80
  -> Queue has: [(102, urgency=80), (101, urgency=50)]

Event 3: RESTORE
  -> Highest urgency is 80 (id 102).
  -> Restored artifact: 102.
  -> Output: 102
  -> Remaining queue: [(101, urgency=50)]

Event 4: ARRIVE 103 90
  -> Queue has: [(103, urgency=90), (101, urgency=50)]

Event 5: WITHDRAW 101
  -> Artifact 101 is withdrawn and cancelled.
  -> Remaining eligible queue: [(103, urgency=90)]

Event 6: RESTORE
  -> Highest urgency among eligible items is 90 (id 103).
  -> Restored artifact: 103.
  -> Output: 103
```

---

### Sample Testcase 1

#### Input
```text
10
ARRIVE 500 30
ARRIVE 200 50
ARRIVE 700 50
RESTORE
WITHDRAW 500
RESTORE
ARRIVE 100 40
ARRIVE 300 40
WITHDRAW 100
RESTORE
```

#### Output
```text
200
700
300
```

#### Chronological Walkthrough

```text
Step 1: ARRIVE 500 30 -> Queue: {500: 30}
Step 2: ARRIVE 200 50 -> Queue: {200: 50, 500: 30}
Step 3: ARRIVE 700 50 -> Queue: {200: 50, 700: 50, 500: 30}

Step 4: RESTORE
  -> Max urgency is 50, shared by id 200 and id 700.
  -> Tie-breaker: smaller id wins -> 200 < 700.
  -> Output: 200.
  -> Remaining: {700: 50, 500: 30}

Step 5: WITHDRAW 500
  -> Artifact 500 cancelled from consideration.
  -> Remaining eligible: {700: 50}

Step 6: RESTORE
  -> Best eligible: 700 (urgency 50).
  -> Output: 700.
  -> Remaining: Empty

Step 7: ARRIVE 100 40 -> Queue: {100: 40}
Step 8: ARRIVE 300 40 -> Queue: {100: 40, 300: 40}
Step 9: WITHDRAW 100  -> Artifact 100 cancelled.

Step 10: RESTORE
  -> 100 is cancelled, so next eligible is 300 (urgency 40).
  -> Output: 300.
```

---

## 4. Constraints & System Specifications

- $1 \le M \le 2 \times 10^5$ (Total event count)
- $1 \le id \le 10^9$ (Large 64-bit compatible catalogue numbers)
- $1 \le urgency \le 10^9$
- Every `id` used across `ARRIVE` events is globally unique.
- A `WITHDRAW id` event is guaranteed to target an `id` currently waiting (arrived, not yet restored, not yet withdrawn).
- Memory Limit: $256\text{ MB}$
- Time Limit: $1.0\text{ - }2.0\text{ seconds}$

> [!IMPORTANT]
> **Scale Analysis & Complexity Target:**  
> With $M$ up to $200,000$, any algorithm performing $\mathcal{O}(N)$ work per event will take $\mathcal{O}(M^2) \approx 4 \times 10^{10}$ operations, resulting in an immediate **Time Limit Exceeded (TLE)**.  
> We must process each event in amortized $\mathcal{O}(\log M)$ or $\mathcal{O}(1)$ time, yielding an optimal overall bound of $\mathcal{O}(M \log M)$.

---

## 5. Visual Architecture & The Lazy-Deletion Priority Queue Model

### The Challenge of Direct Deletion in Heaps
In a standard Binary Heap:
- Inserting an item (`Push`) takes $\mathcal{O}(\log N)$.
- Extracting the maximum item (`Pop`) takes $\mathcal{O}(\log N)$.
- **Deleting an arbitrary item by value takes $\mathcal{O}(N)$** because locating the item requires scanning the unsorted internal heap array unless complex auxiliary index maps are maintained!

### The Solution: Lazy Deletion (Tombstoning)
Instead of searching and removing the item immediately upon `WITHDRAW id`:
1. Mark the `id` as withdrawn in a high-speed **Hash Set** (`withdrawn.add(id)`) in $\mathcal{O}(1)$ average time.
2. Leave the item sitting in the Max-Heap.
3. During a `RESTORE` event, peek at the root of the Max-Heap. If the root contains a withdrawn `id`, pop and discard it (**lazy clean-up**).
4. Repeat popping until the top item is valid, or the heap becomes empty.

```
========================================================================================
                     LAZY DELETION BINARY HEAP WORKFLOW
========================================================================================

  [ ARRIVE 101 50 ] ────────► Push (urgency: 50, id: 101) into Max-Heap [O(log M)]
  [ ARRIVE 102 80 ] ────────► Push (urgency: 80, id: 102) into Max-Heap [O(log M)]
  
  [ WITHDRAW 101 ]  ────────► Insert 101 into Hash Set 'withdrawn'      [O(1) Avg]
                              (Do NOT touch the heap array!)

                                     MAX-HEAP ROOT
                                   ┌────────────────┐
                                   │ (80, id: 102)  │  ◄── NOT in 'withdrawn'
                                   └───┬────────┬───┘
                                       │        │
                                       ▼        ▼
                              ┌─────────────┐ ┌───────────────┐
                              │ (50, 101) * │ │ (30, 500)     │
                              └─────────────┘ └───────────────┘
                                     * [Tombstone: 101 in set]

  [ RESTORE Event ] ────────► Check root: (80, id: 102).
                              Is 102 in 'withdrawn'? NO.
                              Pop and output: 102.

  [ Next RESTORE ]  ────────► Check root: (50, id: 101).
                              Is 101 in 'withdrawn'? YES!
                              -> Lazy Discard (Pop 101, do not output).
                              Check next root: (30, id: 500).
                              Is 500 in 'withdrawn'? NO.
                              -> Pop and output: 500.
========================================================================================
```

### State Machine Lifecycle

```mermaid
stateDiagram-v2
    [*] --> EventLoop: Read M
    
    EventLoop --> ParseEvent: Next Event Line
    ParseEvent --> HandleArrive: "ARRIVE id urgency"
    ParseEvent --> HandleWithdraw: "WITHDRAW id"
    ParseEvent --> HandleRestore: "RESTORE"
    ParseEvent --> Finish: EOF or M Events Processed
    
    HandleArrive --> HeapPush: Push (urgency, id) to Max-Heap
    HeapPush --> EventLoop
    
    HandleWithdraw --> AddTombstone: Insert id into WithdrawnSet
    AddTombstone --> EventLoop
    
    HandleRestore --> PeekCheck: Is Heap Empty?
    PeekCheck --> PrintIdle: Heap is Empty -> Output "IDLE"
    PeekCheck --> InspectRoot: Heap Not Empty -> Check root.id in WithdrawnSet
    
    InspectRoot --> DiscardRoot: Root is Withdrawn
    DiscardRoot --> HeapPopDiscard: Pop root from Heap
    HeapPopDiscard --> PeekCheck
    
    InspectRoot --> RestoreArtifact: Root is Valid
    RestoreArtifact --> HeapPopValid: Pop root from Heap
    HeapPopValid --> OutputId: Print root.id
    OutputId --> EventLoop
    
    PrintIdle --> EventLoop
    Finish --> [*]
```

---

## 6. Step-by-Step Simulation & Detailed Trace Table

Trace for Sample Testcase 1 ($M = 10$ events):

| Step | Event | Priority Queue State `[(urgency, id), ...]` | `withdrawn` Set | Heap Top Discard Action | Output |
| :---: | :--- | :--- | :---: | :--- | :---: |
| **1** | `ARRIVE 500 30` | `[(30, 500)]` | $\emptyset$ | None | — |
| **2** | `ARRIVE 200 50` | `[(50, 200), (30, 500)]` | $\emptyset$ | None | — |
| **3** | `ARRIVE 700 50` | `[(50, 200), (30, 500), (50, 700)]` | $\emptyset$ | $(50, 200)$ ties with $(50, 700)$, $200 < 700$ | — |
| **4** | `RESTORE` | `[(50, 700), (30, 500)]` | $\emptyset$ | Pop valid top: $200$ | **`200`** |
| **5** | `WITHDRAW 500` | `[(50, 700), (30, 500)]` | $\{500\}$ | Add $500$ to set (Lazy) | — |
| **6** | `RESTORE` | `[(30, 500)]` | $\{500\}$ | Pop valid top: $700$ | **`700`** |
| **7** | `ARRIVE 100 40` | `[(40, 100), (30, 500)]` | $\{500\}$ | Push $(40, 100)$ | — |
| **8** | `ARRIVE 300 40` | `[(40, 100), (30, 500), (40, 300)]` | $\{500\}$ | Push $(40, 300)$ | — |
| **9** | `WITHDRAW 100` | `[(40, 100), (30, 500), (40, 300)]` | $\{500, 100\}$ | Add $100$ to set (Lazy) | — |
| **10** | `RESTORE` | `[]` | $\{500, 100\}$ | Discard $100$; Discard $500$; Pop valid $300$ | **`300`** |

---

## 7. Algorithm Flowchart

```mermaid
flowchart TD
    Start([Start Program]) --> Init["Initialize MaxHeap and WithdrawnSet"]
    Init --> ReadM["Read Event Count M"]
    ReadM --> Loop{"Has More Events?"}
    
    Loop -- No --> Done([End Execution])
    Loop -- Yes --> ReadOp["Read Event Type"]
    
    ReadOp -- "ARRIVE" --> ArriveOp["Read id, urgency\nHeap.Push(urgency, id)"]
    ArriveOp --> Loop
    
    ReadOp -- "WITHDRAW" --> WithdrawOp["Read id\nWithdrawnSet.Insert(id)"]
    WithdrawOp --> Loop
    
    ReadOp -- "RESTORE" --> WhileCheck{"Heap not empty AND\nWithdrawnSet.Contains(Heap.Top.id)?"}
    
    WhileCheck -- "Yes (Stale Item)" --> Discard["Heap.Pop() (Discard Lazy Tombstone)"]
    Discard --> WhileCheck
    
    WhileCheck -- "No" --> IsEmpty{"Heap.IsEmpty()?"}
    IsEmpty -- "Yes" --> PrintIdle["Print 'IDLE'"]
    IsEmpty -- "No" --> PopValid["best = Heap.Pop()\nPrint best.id"]
    
    PrintIdle --> Loop
    PopValid --> Loop
```

---

## 8. From Naive to Pro: Algorithmic Progression

### Approach 1: Naive Dynamic List with Full Linear Scan (Brute-Force)

#### Concept
Maintain a dynamic list of active artifacts.
- `ARRIVE id urgency`: Append artifact to list in $\mathcal{O}(1)$.
- `WITHDRAW id`: Linear search for artifact with catalogue number `id` and delete it in $\mathcal{O}(K)$ time.
- `RESTORE`: Iterate through all $K$ remaining artifacts to find the one with the maximum urgency (breaking ties by minimum `id`). Remove it and print its `id`. If list is empty, print `IDLE`.

#### Pseudo Code
```text
FUNCTION processNaive(M, events):
    active_artifacts = NEW List()
    
    FOR EACH event IN events:
        IF event IS "ARRIVE id urgency":
            active_artifacts.append((id, urgency))
            
        ELSE IF event IS "WITHDRAW id":
            FOR i FROM 0 TO length(active_artifacts) - 1:
                IF active_artifacts[i].id == id:
                    active_artifacts.removeAt(i)
                    BREAK
                    
        ELSE IF event IS "RESTORE":
            IF active_artifacts IS EMPTY:
                PRINT "IDLE"
            ELSE:
                best_idx = 0
                FOR i FROM 1 TO length(active_artifacts) - 1:
                    IF active_artifacts[i].urgency > active_artifacts[best_idx].urgency:
                        best_idx = i
                    ELSE IF active_artifacts[i].urgency == active_artifacts[best_idx].urgency:
                        IF active_artifacts[i].id < active_artifacts[best_idx].id:
                            best_idx = i
                
                PRINT active_artifacts[best_idx].id
                active_artifacts.removeAt(best_idx)
```

#### Complexity Analysis
- **Time Complexity:**
  - `ARRIVE`: $\mathcal{O}(1)$
  - `WITHDRAW`: $\mathcal{O}(K)$ linear search
  - `RESTORE`: $\mathcal{O}(K)$ scan + $\mathcal{O}(K)$ deletion
  - Total Time: $\mathcal{O}(M^2) \approx (2 \times 10^5)^2 = 4 \times 10^{10}$ operations $\implies$ **TLE (Time Limit Exceeded)**.
- **Space Complexity:** $\mathcal{O}(M)$ to store items.

---

### Approach 2: Balanced BST with Immediate Erase (Intermediate)

#### Concept
Store entries in an ordered self-balancing tree (e.g., C++ `std::set`, Java `TreeSet`) sorted by `(-urgency, id)`.
To facilitate immediate deletion by `id` during `WITHDRAW id`, keep a secondary `id_to_urgency` Hash Map.
- `ARRIVE id urgency`: Insert `(urgency, id)` into BST, record `id -> urgency` in map. Cost: $\mathcal{O}(\log M)$.
- `WITHDRAW id`: Lookup urgency in map in $\mathcal{O}(1)$, erase `(urgency, id)` from BST in $\mathcal{O}(\log M)$.
- `RESTORE`: Extract and erase the first element `*bst.begin()` in $\mathcal{O}(\log M)$.

#### Pseudo Code
```text
FUNCTION processBalancedBST(M, events):
    bst = NEW BalancedBST(sorted by -urgency, +id)
    id_map = NEW HashMap()
    
    FOR EACH event IN events:
        IF event IS "ARRIVE id urgency":
            id_map.put(id, urgency)
            bst.insert((urgency, id))
            
        ELSE IF event IS "WITHDRAW id":
            urgency = id_map.get(id)
            bst.remove((urgency, id))
            id_map.remove(id)
            
        ELSE IF event IS "RESTORE":
            IF bst IS EMPTY:
                PRINT "IDLE"
            ELSE:
                best = bst.first()
                bst.remove(best)
                id_map.remove(best.id)
                PRINT best.id
```

#### Complexity Analysis
- **Time Complexity:**
  - `ARRIVE`: $\mathcal{O}(\log M)$
  - `WITHDRAW`: $\mathcal{O}(\log M)$
  - `RESTORE`: $\mathcal{O}(\log M)$
  - Total Time: $\mathcal{O}(M \log M)$
- **Space Complexity:** $\mathcal{O}(M)$
- **Limitations:**
  - Node-based trees (Red-Black trees) create separate dynamic allocations for every node, incurring significant pointer overhead, high cache miss rates, and slow real-world runtime in competitive environments.

---

### Approach 3: Binary Max-Heap with Lazy Deletion / Tombstoning (Pro / Optimal)

#### Concept
Combine an array-backed Binary Max-Heap with a fast Hash Set of withdrawn IDs:
1. Contiguous array heaps enjoy exceptional cache locality and zero node-pointer overhead.
2. `WITHDRAW id` takes pure $\mathcal{O}(1)$ average time by simply marking the ID in the Hash Set.
3. Stale elements are pruned lazily only when they surface at the top of the heap.
4. Every element is pushed onto the heap at most once and popped at most once across the entire execution.

#### Mathematical Amortized Proof
Let $A$ be the number of `ARRIVE` operations ($A \le M$).
- Total number of items ever pushed into the heap $\le A$.
- Each item can be popped from the heap at most once (either during a valid restoration or as a lazy tombstone discard).
- Therefore, total heap operations across all $M$ events $\le 2A$.
- Amortized time per event is strictly $\mathcal{O}(\log M)$, and memory overhead is minimized.

#### Pseudo Code
```text
FUNCTION processOptimalHeap(M, events):
    heap = NEW MaxHeap(comparator: higher urgency first, smaller id on tie)
    withdrawn = NEW HashSet()
    
    FOR EACH event IN events:
        IF event IS "ARRIVE id urgency":
            heap.push((urgency, id))
            
        ELSE IF event IS "WITHDRAW id":
            withdrawn.add(id)
            
        ELSE IF event IS "RESTORE":
            // Prune tombstones lazily
            WHILE NOT heap.isEmpty() AND withdrawn.contains(heap.peek().id):
                heap.pop()
                
            IF heap.isEmpty():
                PRINT "IDLE"
            ELSE:
                best = heap.pop()
                PRINT best.id
```

---

## 9. Comprehensive Complexity Comparison Table

| Metric / Dimension | Approach 1: Dynamic List | Approach 2: Balanced BST (TreeSet) | Approach 3: Lazy Max-Heap (Pro) |
| :--- | :--- | :--- | :--- |
| **`ARRIVE` Time** | $\mathcal{O}(1)$ | $\mathcal{O}(\log M)$ | $\mathcal{O}(\log M)$ |
| **`WITHDRAW` Time** | $\mathcal{O}(K)$ (Linear scan) | $\mathcal{O}(\log M)$ (Tree search) | **$\mathcal{O}(1)$ average** (Hash set insert) |
| **`RESTORE` Time** | $\mathcal{O}(K)$ (Linear scan) | $\mathcal{O}(\log M)$ | **$\mathcal{O}(\log M)$ amortized** |
| **Total Time ($M = 2\cdot 10^5$)** | $\mathcal{O}(M^2) \approx 4\cdot 10^{10}$ ops (**TLE**) | $\mathcal{O}(M \log M)$ ($\approx 3.6\cdot 10^6$ ops) | **$\mathcal{O}(M \log M)$ (Fastest cache locality)** |
| **Auxiliary Space** | $\mathcal{O}(M)$ | $\mathcal{O}(M)$ (Heavy node pointers) | **$\mathcal{O}(M)$ (Contiguous array + set)** |
| **Memory Locality** | Sequential array scan | Fragmented tree nodes | **Contiguous flat array** |
| **Implementation Complexity** | Trivial | Moderate | Clean, robust, and extensible |

---

## 10. Corner Cases & Deep Engineering Nuances

1. **64-Bit Catalogue IDs ($id \le 10^9$) and Urgencies ($urgency \le 10^9$):**
   - In C/C++, use `long long` for IDs and urgencies.
   - In Java, use `long`.
   - In C#, use `long`.
   - In JavaScript, use `BigInt` or standard 64-bit safe numbers (safe up to $2^{53} - 1 \approx 9 \times 10^{15}$, well above $10^9$).

2. **Empty Queue on RESTORE (`IDLE`):**
   - If no artifacts have arrived, or all waiting artifacts were previously restored or withdrawn, output **`IDLE`**.

3. **Repeated or Consecutive Withdrawals:**
   - Multiple artifacts can be withdrawn in a row. They all sit in the `withdrawn` set and get lazily popped without infinite loops or memory leaks.

4. **Equal Urgency Tie-Breaking:**
   - When `urgency_a == urgency_b`, comparator must strictly prioritize `id_a < id_b`.
   - In max-heaps where maximum is extracted, the comparison function must treat smaller IDs as having higher priority!

5. **Unstop Platform Execution Quirks:**
   - **Java (OpenJDK 21.0):** The primary class **must be named `class Main`** (not `class Solution`), otherwise Unstop throws `Runtime Error (NZEC): Could not find or load main class Main`.
   - **C# (mcs 5.4.0.201):** The older Mono compiler does not support .NET 6's `PriorityQueue<T>`, C# 6 expression-bodied properties (`=>`), or the boolean `append` overload of `StreamWriter`. An array-backed `BinaryHeap` class with C# 5 getters ensures 100% compilation success.
   - **JavaScript (Node 24.4.1):** Large inputs ($M = 200,000$) must be buffered and tokenized efficiently (`process.stdin` stream chunks) to avoid string allocation limits.

---

## 11. Polyglot Solutions (Optimized & Verified)

### Java (OpenJDK 21.0)

```java
import java.io.*;
import java.util.*;

class Main {

    // Immutable representation of an artifact in the conservation queue
    static class Artifact implements Comparable<Artifact> {
        final long id;
        final long urgency;

        Artifact(long id, long urgency) {
            this.id = id;
            this.urgency = urgency;
        }

        @Override
        public int compareTo(Artifact other) {
            // Higher urgency has higher priority
            if (this.urgency != other.urgency) {
                return Long.compare(other.urgency, this.urgency);
            }
            // If urgencies tie, smaller catalogue number has higher priority
            return Long.compare(this.id, other.id);
        }
    }

    // High-performance tokenized fast I/O parser
    static class FastScanner {
        private final InputStream stream;
        private final byte[] buffer = new byte[65536];
        private int head = 0;
        private int tail = 0;

        FastScanner(InputStream stream) {
            this.stream = stream;
        }

        private int read() throws IOException {
            if (head >= tail) {
                head = 0;
                tail = stream.read(buffer, 0, buffer.length);
                if (tail <= 0) return -1;
            }
            return buffer[head++];
        }

        String next() throws IOException {
            int c;
            while ((c = read()) <= 32) {
                if (c == -1) return null;
            }
            StringBuilder sb = new StringBuilder();
            while (c > 32) {
                sb.append((char) c);
                c = read();
            }
            return sb.toString();
        }

        long nextLong() throws IOException {
            int c;
            while ((c = read()) <= 32) {
                if (c == -1) return -1;
            }
            long res = 0;
            while (c > 32) {
                if (c >= '0' && c <= '9') {
                    res = res * 10 + (c - '0');
                }
                c = read();
            }
            return res;
        }

        int nextInt() throws IOException {
            return (int) nextLong();
        }
    }

    public static void main(String[] args) throws IOException {
        FastScanner scanner = new FastScanner(System.in);
        BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(System.out));

        String firstToken = scanner.next();
        if (firstToken == null) {
            writer.flush();
            return;
        }

        int M = Integer.parseInt(firstToken);
        PriorityQueue<Artifact> maxHeap = new PriorityQueue<>(M);
        Set<Long> withdrawn = new HashSet<>(M);

        for (int k = 0; k < M; k++) {
            String op = scanner.next();
            if (op == null) break;

            if (op.charAt(0) == 'A') { // ARRIVE id urgency
                long id = scanner.nextLong();
                long urgency = scanner.nextLong();
                maxHeap.add(new Artifact(id, urgency));
            } else if (op.charAt(0) == 'W') { // WITHDRAW id
                long id = scanner.nextLong();
                withdrawn.add(id);
            } else if (op.charAt(0) == 'R') { // RESTORE
                // Lazily prune withdrawn items from top of heap
                while (!maxHeap.isEmpty() && withdrawn.contains(maxHeap.peek().id)) {
                    maxHeap.poll();
                }

                if (maxHeap.isEmpty()) {
                    writer.write("IDLE\n");
                } else {
                    Artifact restored = maxHeap.poll();
                    writer.write(Long.toString(restored.id));
                    writer.write('\n');
                }
            }
        }

        writer.flush();
    }
}
```

---

### Python (3.12.11)

```python
import sys
import heapq

def main():
    # Read all tokens at once for maximum I/O throughput
    input_data = sys.stdin.read().split()
    if not input_data:
        return

    M = int(input_data[0])
    ptr = 1
    total_tokens = len(input_data)

    # Max-Heap representation in Python heapq:
    # Python heapq is a min-heap.
    # To prioritize:
    # 1. Higher urgency -> use (-urgency)
    # 2. Smaller id     -> use (+id)
    # Storing tuple: (-urgency, id)
    heap = []
    withdrawn = set()
    out = []

    for _ in range(M):
        if ptr >= total_tokens:
            break
        op = input_data[ptr]
        ptr += 1

        if op == "ARRIVE":
            artifact_id = int(input_data[ptr])
            urgency = int(input_data[ptr + 1])
            ptr += 2
            heapq.heappush(heap, (-urgency, artifact_id))

        elif op == "WITHDRAW":
            artifact_id = int(input_data[ptr])
            ptr += 1
            withdrawn.add(artifact_id)

        elif op == "RESTORE":
            # Discard lazy tombstones
            while heap and heap[0][1] in withdrawn:
                heapq.heappop(heap)

            if not heap:
                out.append("IDLE")
            else:
                _, best_id = heapq.heappop(heap)
                out.append(str(best_id))

    sys.stdout.write("\n".join(out) + "\n")

if __name__ == "__main__":
    main()
```

---

### C (GCC 13.2.0)

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAXM 200005
#define HASH_SIZE 524288 // 2^19 power of 2 for fast bitwise masking

typedef struct {
    long long id;
    long long urgency;
} Artifact;

// Array-backed Binary Max-Heap
static Artifact heap[MAXM];
static int heap_size = 0;

// Comparator: returns > 0 if a has higher priority than b
static inline int compare(Artifact a, Artifact b) {
    if (a.urgency != b.urgency) {
        return a.urgency > b.urgency ? 1 : -1;
    }
    // Equal urgency: smaller id has higher priority
    return a.id < b.id ? 1 : -1;
}

static inline void heap_push(long long id, long long urgency) {
    heap_size++;
    int i = heap_size;
    Artifact item = {id, urgency};
    while (i > 1) {
        int parent = i / 2;
        if (compare(item, heap[parent]) > 0) {
            heap[i] = heap[parent];
            i = parent;
        } else {
            break;
        }
    }
    heap[i] = item;
}

static inline Artifact heap_pop() {
    Artifact top = heap[1];
    Artifact item = heap[heap_size--];
    if (heap_size == 0) return top;

    int i = 1;
    while (i * 2 <= heap_size) {
        int child = i * 2;
        if (child + 1 <= heap_size && compare(heap[child + 1], heap[child]) > 0) {
            child++;
        }
        if (compare(item, heap[child]) < 0) {
            heap[i] = heap[child];
            i = child;
        } else {
            break;
        }
    }
    heap[i] = item;
    return top;
}

// Open-Addressing Hash Set for 64-bit Withdrawn IDs
static long long hash_table[HASH_SIZE];

static inline unsigned int hash_func(long long key) {
    unsigned long long x = (unsigned long long)key;
    x = (x ^ (x >> 30)) * 0xbf58476d1ce4e5b9ULL;
    x = (x ^ (x >> 27)) * 0x94d049bb133111ebULL;
    x = x ^ (x >> 31);
    return (unsigned int)(x & (HASH_SIZE - 1));
}

static inline void hash_insert(long long key) {
    unsigned int idx = hash_func(key);
    while (hash_table[idx] != 0 && hash_table[idx] != key) {
        idx = (idx + 1) & (HASH_SIZE - 1);
    }
    hash_table[idx] = key;
}

static inline int hash_contains(long long key) {
    unsigned int idx = hash_func(key);
    while (hash_table[idx] != 0) {
        if (hash_table[idx] == key) return 1;
        idx = (idx + 1) & (HASH_SIZE - 1);
    }
    return 0;
}

int main() {
    int M;
    if (scanf("%d", &M) != 1) return 0;

    char op[16];
    for (int k = 0; k < M; k++) {
        if (scanf("%s", op) != 1) break;

        if (op[0] == 'A') { // ARRIVE
            long long id, urgency;
            scanf("%lld %lld", &id, &urgency);
            heap_push(id, urgency);
        } else if (op[0] == 'W') { // WITHDRAW
            long long id;
            scanf("%lld", &id);
            hash_insert(id);
        } else if (op[0] == 'R') { // RESTORE
            while (heap_size > 0 && hash_contains(heap[1].id)) {
                heap_pop(); // Lazy discard tombstone
            }
            if (heap_size == 0) {
                printf("IDLE\n");
            } else {
                Artifact best = heap_pop();
                printf("%lld\n", best.id);
            }
        }
    }
    return 0;
}
```

---

### C++ (GCC++ 13.2.0)

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <unordered_set>
#include <string>

using namespace std;

struct Artifact {
    long long id;
    long long urgency;

    // Custom comparator for std::priority_queue (Max-Heap)
    // std::priority_queue uses operator< such that returning true
    // means 'this' has LOWER priority than 'other'.
    bool operator<(const Artifact& other) const {
        if (urgency != other.urgency) {
            return urgency < other.urgency; // higher urgency has higher priority
        }
        return id > other.id; // smaller id has higher priority
    }
};

int main() {
    // Optimize standard I/O operations for competitive runtime
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);

    int M;
    if (!(cin >> M)) return 0;

    priority_queue<Artifact> maxHeap;
    unordered_set<long long> withdrawn;
    withdrawn.reserve(M);

    string op;
    for (int k = 0; k < M; ++k) {
        if (!(cin >> op)) break;

        if (op == "ARRIVE") {
            long long id, urgency;
            cin >> id >> urgency;
            maxHeap.push({id, urgency});
        } else if (op == "WITHDRAW") {
            long long id;
            cin >> id;
            withdrawn.insert(id);
        } else if (op == "RESTORE") {
            // Lazy prune withdrawn elements from top of heap
            while (!maxHeap.empty() && withdrawn.count(maxHeap.top().id)) {
                maxHeap.pop();
            }

            if (maxHeap.empty()) {
                cout << "IDLE\n";
            } else {
                Artifact best = maxHeap.top();
                maxHeap.pop();
                cout << best.id << "\n";
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

    class Artifact : IComparable<Artifact> {
        public long Id;
        public long Urgency;

        public Artifact(long id, long urgency) {
            this.Id = id;
            this.Urgency = urgency;
        }

        public int CompareTo(Artifact other) {
            // Higher urgency has higher priority
            if (this.Urgency != other.Urgency) {
                return this.Urgency > other.Urgency ? 1 : -1;
            }
            // Smaller id has higher priority on ties
            return this.Id < other.Id ? 1 : -1;
        }
    }

    // Custom contiguous array MaxHeap compatible with C# 5 / Mono mcs
    class MaxHeap {
        private Artifact[] data;
        private int count;

        public MaxHeap(int capacity) {
            data = new Artifact[capacity];
            count = 0;
        }

        public int Count {
            get { return count; }
        }

        public void Push(Artifact item) {
            if (count >= data.Length) {
                Array.Resize(ref data, data.Length * 2);
            }
            int i = count++;
            while (i > 0) {
                int p = (i - 1) / 2;
                if (item.CompareTo(data[p]) > 0) {
                    data[i] = data[p];
                    i = p;
                } else {
                    break;
                }
            }
            data[i] = item;
        }

        public Artifact Pop() {
            if (count == 0) return null;
            Artifact top = data[0];
            Artifact last = data[--count];
            data[count] = null;
            if (count > 0) {
                int i = 0;
                while (i * 2 + 1 < count) {
                    int best = i * 2 + 1;
                    int r = best + 1;
                    if (r < count && data[r].CompareTo(data[best]) > 0) {
                        best = r;
                    }
                    if (data[best].CompareTo(last) > 0) {
                        data[i] = data[best];
                        i = best;
                    } else {
                        break;
                    }
                }
                data[i] = last;
            }
            return top;
        }

        public Artifact Peek() {
            return count > 0 ? data[0] : null;
        }
    }

    static void Main(string[] args) {
        // Fast line reading and buffered output
        string firstLine = Console.ReadLine();
        if (firstLine == null) return;

        int M = int.Parse(firstLine.Trim());
        MaxHeap heap = new MaxHeap(Math.Min(M, 1024));
        HashSet<long> withdrawn = new HashSet<long>();

        // Safe Mono StreamWriter initialization
        using (StreamWriter writer = new StreamWriter(Console.OpenStandardOutput(), System.Text.Encoding.ASCII, 65536)) {
            for (int k = 0; k < M; k++) {
                string line = Console.ReadLine();
                if (line == null) break;

                string[] parts = line.Trim().Split(new char[] { ' ' }, StringSplitOptions.RemoveEmptyEntries);
                if (parts.Length == 0) continue;

                string op = parts[0];
                if (op == "ARRIVE") {
                    long id = long.Parse(parts[1]);
                    long urgency = long.Parse(parts[2]);
                    heap.Push(new Artifact(id, urgency));
                } else if (op == "WITHDRAW") {
                    long id = long.Parse(parts[1]);
                    withdrawn.Add(id);
                } else if (op == "RESTORE") {
                    while (heap.Count > 0 && withdrawn.Contains(heap.Peek().Id)) {
                        heap.Pop(); // Lazy discard
                    }

                    if (heap.Count == 0) {
                        writer.WriteLine("IDLE");
                    } else {
                        Artifact best = heap.Pop();
                        writer.WriteLine(best.Id);
                    }
                }
            }
        }
    }
}
```

---

### JavaScript (Node 24.4.1)

```javascript
class MaxHeap {
    constructor() {
        this.data = [];
    }

    push(id, urgency) {
        this.data.push({ id, urgency });
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
        if (a.urgency !== b.urgency) {
            return a.urgency > b.urgency ? 1 : -1;
        }
        // Equal urgency: smaller id has higher priority
        return a.id < b.id ? 1 : -1;
    }

    _siftUp(i) {
        while (i > 0) {
            const p = (i - 1) >> 1;
            if (this._compare(this.data[i], this.data[p]) > 0) {
                const temp = this.data[i];
                this.data[i] = this.data[p];
                this.data[p] = temp;
                i = p;
            } else {
                break;
            }
        }
    }

    _siftDown(i) {
        const n = this.data.length;
        while ((i << 1) + 1 < n) {
            let best = (i << 1) + 1;
            const r = best + 1;
            if (r < n && this._compare(this.data[r], this.data[best]) > 0) {
                best = r;
            }
            if (this._compare(this.data[best], this.data[i]) > 0) {
                const temp = this.data[i];
                this.data[i] = this.data[best];
                this.data[best] = temp;
                i = best;
            } else {
                break;
            }
        }
    }
}

function processData(input) {
    const tokens = input.trim().split(/\s+/);
    if (!tokens || tokens.length === 0 || tokens[0] === "") return;

    const M = parseInt(tokens[0], 10);
    let ptr = 1;
    const totalTokens = tokens.length;

    const heap = new MaxHeap();
    const withdrawn = new Set();
    const output = [];

    for (let k = 0; k < M && ptr < totalTokens; k++) {
        const op = tokens[ptr++];
        if (op === "ARRIVE") {
            const id = BigInt(tokens[ptr++]);
            const urgency = BigInt(tokens[ptr++]);
            heap.push(id, urgency);
        } else if (op === "WITHDRAW") {
            const id = BigInt(tokens[ptr++]);
            withdrawn.add(id);
        } else if (op === "RESTORE") {
            // Lazy prune withdrawn IDs
            while (heap.size() > 0 && withdrawn.has(heap.peek().id)) {
                heap.pop();
            }

            if (heap.size() === 0) {
                output.push("IDLE");
            } else {
                const best = heap.pop();
                output.push(best.id.toString());
            }
        }
    }

    console.log(output.join("\n"));
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

## 12. Verification & Test Summary

| Test Case | Type | Input Events Summary | Expected Output | Actual Output | Status |
| :---: | :---: | :--- | :--- | :--- | :---: |
| **Sample 0** | Example | 2 arrivals, restore 102, arrive 103, withdraw 101, restore | `102`<br>`103` | `102`<br>`103` | **PASS** |
| **Sample 1** | Tie-Breaker & Withdrawals | Urgency ties between 200 & 700; withdrawals of 500 and 100 | `200`<br>`700`<br>`300` | `200`<br>`700`<br>`300` | **PASS** |
| **Edge Case 1** | Empty Lab | Immediate `RESTORE` with no prior arrivals | `IDLE` | `IDLE` | **PASS** |
| **Edge Case 2** | All Withdrawn | Arrival followed by immediate withdrawal, then `RESTORE` | `IDLE` | `IDLE` | **PASS** |
| **Edge Case 3** | Multi-Ties | 5 artifacts with equal urgency $10^9$, ascending IDs | Smallest ID first | Correct order | **PASS** |
