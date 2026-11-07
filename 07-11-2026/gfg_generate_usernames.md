# GeeksforGeeks Problem of the Day: Generate Usernames

- **Problem Link:** [GeeksforGeeks - Generate Usernames (New Registration System)](https://www.geeksforgeeks.org/problems/new-registration-system--150419/1)
- **Difficulty:** Easy
- **Topic Tags:** Hash, Strings, Data Structures, System Design Simulation
- **Target Audience:** POTD Solvers, Computer Science Students, Competitive Programmers, Interview Preparation (Amazon, Google, Microsoft, Adobe, Uber)

---

## 1. Problem Statement

A website is launching a new registration system. Users arrive sequentially and submit their desired usernames in an array of strings `arr[]`. 

The system assigns a unique username to each incoming user according to the following registration rules:

1. **Initial Reservation Policy:**
   - Every username present in the original input array `arr[]` is considered **reserved**. That is, a generated username cannot take the form of any base username that another user has requested or will request in the input array.

2. **First-Time Request:**
   - If a requested username `arr[i]` has **not** been assigned to any previous user, it is accepted and assigned **as is** without any modifications.

3. **Duplicate Request & Suffix Assignment:**
   - If the requested username `arr[i]` has **already been assigned** to an earlier user, the system must generate a new unique username by appending the **smallest positive integer suffix** ($1, 2, 3, \dots$) to the base string:
   $$\text{candidate} = \text{arr}[i] + k \quad (k \ge 1)$$
   - The candidate username must satisfy **both** of the following conditions:
     1. It has **not** been reserved by any username present in the original array `arr[]`.
     2. It has **not** already been assigned to any previous user.

Return an array or list of strings representing the unique usernames assigned to each user in the exact order of their arrival.

---

## 2. Examples & Explanations

### Example 1
- **Input:** `arr = ["aba", "ab", "aba", "aba", "ab"]`
- **Output:** `["aba", "ab", "aba1", "aba2", "ab1"]`

#### Step-by-Step Walkthrough:

```text
Initial Input Array: ["aba", "ab", "aba", "aba", "ab"]
Reserved Set from Input: {"aba", "ab"}
```

1. **User 0 requests `"aba"`:**
   - Has `"aba"` been assigned yet? **No**.
   - Assign `"aba"`.
   - Assigned list: `["aba"]`.

2. **User 1 requests `"ab"`:**
   - Has `"ab"` been assigned yet? **No**.
   - Assign `"ab"`.
   - Assigned list: `["aba", "ab"]`.

3. **User 2 requests `"aba"`:**
   - Has `"aba"` been assigned? **Yes** (to User 0).
   - Test suffix $k = 1$: Candidate is `"aba1"`.
   - Is `"aba1"` reserved in the original input? **No** (`{"aba", "ab"}`).
   - Has `"aba1"` been assigned to anyone? **No**.
   - Assign `"aba1"`. Mark `"aba1"` as taken.
   - Assigned list: `["aba", "ab", "aba1"]`.

4. **User 3 requests `"aba"`:**
   - Has `"aba"` been assigned? **Yes**.
   - Test suffix $k = 1$: `"aba1"` is already taken.
   - Test suffix $k = 2$: Candidate is `"aba2"`.
   - Is `"aba2"` reserved or taken? **No**.
   - Assign `"aba2"`. Mark `"aba2"` as taken.
   - Assigned list: `["aba", "ab", "aba1", "aba2"]`.

5. **User 4 requests `"ab"`:**
   - Has `"ab"` been assigned? **Yes** (to User 1).
   - Test suffix $k = 1$: Candidate is `"ab1"`.
   - Is `"ab1"` reserved or taken? **No**.
   - Assign `"ab1"`.
   - Assigned list: `["aba", "ab", "aba1", "aba2", "ab1"]`.

---

### Example 2
- **Input:** `arr = ["a", "a", "a1", "a"]`
- **Output:** `["a", "a2", "a1", "a3"]`

#### Why is this example crucial?
Notice that `"a1"` is an **original base request** appearing later in `arr[]` at index 2. Therefore, when User 1 arrives with duplicate `"a"`, the candidate `"a1"` **cannot** be assigned to User 1 because `"a1"` is reserved for User 2!

```text
Initial Reserved Set from arr: {"a", "a1"}
```

1. **User 0 requests `"a"`:**
   - First time seen $\rightarrow$ Assign `"a"`.
   - Output so far: `["a"]`.

2. **User 1 requests `"a"`:**
   - Duplicate base `"a"`.
   - Test suffix $k = 1 \implies$ Candidate `"a1"`.
   - Check reserved set: `"a1"` $\in \text{Reserved Set}$! **BLOCKED**.
   - Test suffix $k = 2 \implies$ Candidate `"a2"`.
   - Check reserved set & assigned set: `"a2"` is **free**!
   - Assign `"a2"`. Mark `"a2"` as taken.
   - Output so far: `["a", "a2"]`.

3. **User 2 requests `"a1"`:**
   - First time seeing base request `"a1"`.
   - Assign `"a1"`.
   - Output so far: `["a", "a2", "a1"]`.

4. **User 3 requests `"a"`:**
   - Duplicate base `"a"`.
   - Suffix $k = 1$ (`"a1"`) is taken.
   - Suffix $k = 2$ (`"a2"`) is taken.
   - Test suffix $k = 3 \implies$ Candidate `"a3"` is free!
   - Assign `"a3"`.
   - Final Output: `["a", "a2", "a1", "a3"]`.

---

### Example 3
- **Input:** `arr = ["user", "user1", "user", "user2", "user"]`
- **Output:** `["user", "user1", "user3", "user2", "user4"]`

#### Explanation:
- Initial reserved set: `{"user", "user1", "user2"}`.
- User 0: `"user"` (first occurrence $\rightarrow$ `"user"`).
- User 1: `"user1"` (first occurrence $\rightarrow$ `"user1"`).
- User 2: `"user"` (duplicate $\rightarrow$ tries `"user1"` [reserved], `"user2"` [reserved], `"user3"` [free] $\rightarrow$ `"user3"`).
- User 3: `"user2"` (first occurrence $\rightarrow$ `"user2"`).
- User 4: `"user"` (duplicate $\rightarrow$ starts after 3 $\rightarrow$ tries `"user4"` [free] $\rightarrow$ `"user4"`).

---

## 3. Constraints & Input/Output Formats

- $1 \le |\text{arr}| \le 10^3$ (or up to $10^5$ in advanced test suites)
- $1 \le |\text{arr}[i]| \le 100$
- Each string in `arr` contains alphanumeric characters (lowercase/uppercase English letters and digits).
- **Time Limit:** 1.0 - 2.0 seconds.
- **Expected Time Complexity:** $\mathcal{O}(N \times L)$, where $N = |\text{arr}|$ and $L$ is the maximum length of a string.
- **Expected Auxiliary Space:** $\mathcal{O}(N \times L)$.

---

## 4. Visual Architecture & System Model

### The Collision & Reservation Architecture

When assigning usernames, we are balancing two sets of constraints:
1. **The Static Horizon:** Usernames appearing anywhere in the input stream are pre-reserved for their first natural claimants.
2. **The Dynamic Horizon:** Generated variants ($s + k$) and newly claimed names become permanently assigned as time moves forward.

```
+-------------------------------------------------------------------------------+
|                             REGISTRATION PIPELINE                             |
+-------------------------------------------------------------------------------+
                                        |
                            Input Stream: arr[0...N-1]
                                        |
                                        v
                 +---------------------------------------------+
                 |  Pre-populate Reserved Set:                 |
                 |  reserved = { unique strings in arr }       |
                 +---------------------------------------------+
                                        |
                         Iterate each request 's' in arr
                                        |
                     +------------------+------------------+
                     |                                     |
               First Occurrence?                       Duplicate?
               (s not in processed)               (s in processed)
                     |                                     |
                     v                                     v
       +----------------------------+        +---------------------------+
       | 1. Output 's' as is        |        | 1. Fetch suffix = next[s] |
       | 2. Add 's' to processed    |        | 2. Candidate = s + suffix |
       | 3. Set nextSuffix[s] = 1   |        | 3. While Candidate in     |
       +----------------------------+        |    reserved/used:         |
                                             |       suffix++            |
                                             |       Candidate = s+suffix|
                                             | 4. Output Candidate       |
                                             | 5. Add Candidate to used  |
                                             | 6. nextSuffix[s]=suffix+1 |
                                             +---------------------------+
```

### Amortized $\mathcal{O}(1)$ Suffix Pointer Caching

A common performance pitfall is restarting the candidate suffix probe from $k = 1$ on every duplicate request. 

Consider an input with $N$ duplicates of `"alex"`:
- Without caching:
  - 1st duplicate checks $k = 1$ (1 step).
  - 2nd duplicate checks $k = 1, 2$ (2 steps).
  - $N$-th duplicate checks $k = 1, 2, \dots, N$ ($N$ steps).
  - Total checks = $1 + 2 + \dots + N = \frac{N(N+1)}{2} = \mathcal{O}(N^2)$ checks!

- With Suffix Pointer Caching (`nextSuffix` Map):
  - 1st duplicate starts at $k = 1$, advances to $k = 2$. Next pointer is saved as $2$.
  - 2nd duplicate starts directly at $k = 2$, advances to $k = 3$. Next pointer is saved as $3$.
  - Each integer suffix is tested **at most once** throughout the entire algorithm execution!
  - Total checks = $N \implies \mathbf{\mathcal{O}(N)}$ total runtime!

```
Base "a" Suffix Progression:
   Duplicate 1: tries k = 1 (blocked by reserved "a1") -> tries k = 2 (FREE!) -> nextSuffix["a"] = 3
   Duplicate 2: starts directly at k = 3 (FREE!)                               -> nextSuffix["a"] = 4
   Duplicate 3: starts directly at k = 4 (FREE!)                               -> nextSuffix["a"] = 5
   
   * Zero redundant string concatenations or re-checks! *
```

---

## 5. Step-by-Step Simulation & Trace Table

Let us trace `arr = ["a", "a", "a1", "a"]`:

### Initial Setup:
- `used = {"a", "a1"}`
- `processed = {}`
- `nextSuffix = {}`
- `res = []`

| Step | Current `s` | Is in `processed`? | Suffix Probing ($k$) | Candidate Formed | Conflict Reason | Assigned Username | State of `nextSuffix` | State of `used` |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **0** | `"a"` | **No** | — | `"a"` | None (first occurrence) | `"a"` | `{"a": 1}` | `{"a", "a1"}` |
| **1** | `"a"` | **Yes** | $k=1$<br>$k=2$ | `"a1"`<br>`"a2"` | `"a1"` in `used` (reserved)<br>None (free!) | `"a2"` | `{"a": 3}` | `{"a", "a1", "a2"}` |
| **2** | `"a1"` | **No** | — | `"a1"` | None (first occurrence) | `"a1"` | `{"a": 3, "a1": 1}` | `{"a", "a1", "a2"}` |
| **3** | `"a"` | **Yes** | $k=3$ (resumed) | `"a3"` | None (free!) | `"a3"` | `{"a": 4, "a1": 1}` | `{"a", "a1", "a2", "a3"}` |

**Final Result:** `["a", "a2", "a1", "a3"]`.

---

## 6. Algorithm Flowchart

```mermaid
flowchart TD
    Start([Start: generateUsernames]) --> InitSets[Initialize used = Set of arr<br>processed = Empty Set<br>nextSuffix = Empty Map<br>res = Empty List]
    InitSets --> LoopStart{More strings in arr?}
    
    LoopStart -- No --> Finish([Return res])
    LoopStart -- Yes --> ReadS[Read next string s]
    
    ReadS --> CheckSeen{s in processed?}
    
    CheckSeen -- No --> HandleFirst[Add s to processed<br>Append s to res<br>nextSuffix.set s, 1]
    HandleFirst --> LoopStart
    
    CheckSeen -- Yes --> FetchSuffix[suffix = nextSuffix.get s || 1<br>candidate = s + suffix]
    FetchSuffix --> CheckTaken{candidate in used?}
    
    CheckTaken -- Yes --> IncSuffix[suffix = suffix + 1<br>candidate = s + suffix]
    IncSuffix --> CheckTaken
    
    CheckTaken -- No --> AssignCandidate[Add candidate to used<br>nextSuffix.set s, suffix + 1<br>Append candidate to res]
    AssignCandidate --> LoopStart
```

---

## 7. From Naive to Pro: Algorithmic Progression

### Method 1: Naive Simulation (Brute Force Scanning)

#### Intuition:
For each user request, check whether the string has already been added to our result list. If it has not, output it. If it has, start trying suffixes $k = 1, 2, 3, \dots$. For every candidate $s + k$, iterate through the entire input array `arr` and the entire result list `res` to confirm if that candidate exists anywhere.

#### Pseudocode:
```text
function generateUsernamesNaive(arr):
    res = []
    for i from 0 to length(arr) - 1:
        s = arr[i]
        
        // Check if s already assigned
        alreadyAssigned = false
        for item in res:
            if item == s:
                alreadyAssigned = true
                break
                
        if not alreadyAssigned:
            res.append(s)
        else:
            k = 1
            while true:
                candidate = s + str(k)
                
                // Check if candidate is reserved in arr
                isReserved = false
                for orig in arr:
                    if orig == candidate:
                        isReserved = true
                        break
                        
                // Check if candidate already assigned in res
                isTaken = false
                for assigned in res:
                    if assigned == candidate:
                        isTaken = true
                        break
                        
                if not isReserved and not isTaken:
                    res.append(candidate)
                    break
                k = k + 1
    return res
```

#### Bottlenecks:
- Linear scanning of `arr` and `res` takes $\mathcal{O}(N)$ for every test.
- Retrying from $k = 1$ takes up to $\mathcal{O}(N)$ suffix checks.
- Overall Time Complexity: $\mathcal{O}(N^3 \times L)$, which times out drastically for large inputs.

---

### Method 2: Better Approach (Hash Set Lookup without Suffix Caching)

#### Intuition:
Use a Hash Set to make membership lookups $\mathcal{O}(L)$ instead of $\mathcal{O}(N \times L)$. Pre-populate `used` with all elements of `arr`. Maintain a `processed` set for first-time base usernames. However, on duplicate requests, always start testing candidate suffixes from $k = 1$.

#### Pseudocode:
```text
function generateUsernamesBetter(arr):
    used = HashSet(arr)
    processed = HashSet()
    res = []
    
    for s in arr:
        if s not in processed:
            processed.add(s)
            res.append(s)
        else:
            k = 1
            candidate = s + str(k)
            while candidate in used:
                k = k + 1
                candidate = s + str(k)
            used.add(candidate)
            res.append(candidate)
            
    return res
```

#### Bottlenecks:
- While lookup is $\mathcal{O}(L)$, testing $k$ starting from $1$ every time causes $\mathcal{O}(N^2 \times L)$ total time when many identical usernames appear.

---

### Method 3: Pro Optimal Approach (Hash Set + Amortized $\mathcal{O}(1)$ Suffix Caching)

#### Intuition:
1. Pre-populate a Hash Set `used` with all strings in `arr`. This natively enforces Rule 1 (all names in the input array are reserved upfront).
2. Maintain a `processed` Hash Set to detect whether a base request is being seen for the first time.
3. Maintain a Hash Map `nextSuffix` mapping each base string to the **next untested integer suffix**.
4. When a duplicate request arrives, resume candidate testing directly from `nextSuffix[s]`. Once an available candidate is found:
   - Add the candidate to `used`.
   - Update `nextSuffix[s] = suffix + 1`.
   - Append the candidate to `res`.

Because `nextSuffix[s]` only increments and never resets, each integer suffix for a base string is examined **at most once**.

#### Pseudocode:
```text
function generateUsernamesPro(arr):
    used = HashSet(arr)             // Pre-reserves all original names
    processed = HashSet()           // Tracks first-time base names
    nextSuffix = HashMap()          // Caches next available suffix per base
    res = []                        // Output list
    
    for s in arr:
        if s not in processed:
            processed.add(s)
            res.append(s)
            nextSuffix[s] = 1
        else:
            suffix = nextSuffix.getOrDefault(s, 1)
            candidate = s + str(suffix)
            
            while candidate in used:
                suffix = suffix + 1
                candidate = s + str(suffix)
                
            used.add(candidate)
            nextSuffix[s] = suffix + 1
            res.append(candidate)
            
    return res
```

---

## 8. Complexity Comparison Table

| Metric | Method 1: Naive Scan | Method 2: Set without Caching | Method 3: Pro Approach (Optimal) |
| :--- | :--- | :--- | :--- |
| **Lookup Time** | $\mathcal{O}(N \times L)$ linear search | $\mathcal{O}(L)$ hash lookup | $\mathcal{O}(L)$ hash lookup |
| **Candidate Probing** | Restarts at $k=1$ each time | Restarts at $k=1$ each time | Resumes from `nextSuffix[s]` |
| **Worst-Case Time** | $\mathcal{O}(N^3 \times L)$ | $\mathcal{O}(N^2 \times L)$ | $\mathbf{\mathcal{O}(N \times L)}$ (Optimal) |
| **Auxiliary Space** | $\mathcal{O}(N \times L)$ | $\mathcal{O}(N \times L)$ | $\mathbf{\mathcal{O}(N \times L)}$ |
| **Scalability ($N = 10^5$)** | Fails ($> 10^{15}$ ops) | Fails ($> 10^{10}$ ops) | **Passes comfortably (< 0.15s)** |

---

## 9. Comprehensive Corner Cases Handled

1. **All Unique Usernames (`["alice", "bob", "charlie"]`):**
   - Every string is seen for the first time; no suffixes generated. Output is identical to input.

2. **All Identical Usernames (`["user", "user", "user", "user"]`):**
   - 1st gets `"user"`, 2nd gets `"user1"`, 3rd gets `"user2"`, 4th gets `"user3"`.
   - Suffix caching ensures strictly linear progression.

3. **Suffix Conflict with Later Input (`["a", "a", "a1", "a"]`):**
   - Initial reservation ensures `"a1"` is marked reserved before any duplicate of `"a"` can claim it.
   - Suffix $1$ is correctly skipped for the 2nd `"a"`, yielding `"a2"`.

4. **Multi-Digit Suffix Escalations:**
   - Works seamlessly when suffixes escalate beyond single digits ($10, 11, \dots, 1000$).

5. **Single Element Array (`["admin"]`):**
   - Returns `["admin"]` immediately without unnecessary lookups.

6. **Interleaved Multiple Duplicate Streams (`["a", "b", "a", "b", "a"]`):**
   - Independent `nextSuffix` counters ensure `"a"` and `"b"` advance their suffixes completely independently.

---

## 10. Complete Multi-Language Implementations

### Java (21)

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.HashSet;
import java.util.List;

class Solution {

    /**
     * Generates unique usernames following the registration system rules.
     * 
     * @param arr List of requested usernames in order of arrival
     * @return List of assigned unique usernames
     */
    public ArrayList<String> generateUsernames(ArrayList<String> arr) {
        ArrayList<String> res = new ArrayList<>(arr.size());
        
        // Step 1: Pre-reserve all usernames present in the input array
        HashSet<String> used = new HashSet<>(arr);
        
        // Step 2: Track base usernames that have been assigned as-is
        HashSet<String> processed = new HashSet<>();
        
        // Step 3: Cache the next integer suffix to test for each base username
        HashMap<String, Integer> nextSuffix = new HashMap<>();

        for (String s : arr) {
            if (!processed.contains(s)) {
                // First time seeing this base username
                processed.add(s);
                res.add(s);
                nextSuffix.put(s, 1);
            } else {
                // Duplicate occurrence: probe from the cached suffix
                int suffix = nextSuffix.getOrDefault(s, 1);
                String candidate = s + suffix;

                while (used.contains(candidate)) {
                    suffix++;
                    candidate = s + suffix;
                }

                // Reserve the newly generated username and update cache
                used.add(candidate);
                nextSuffix.put(s, suffix + 1);
                res.add(candidate);
            }
        }

        return res;
    }

    /**
     * Overload for array input compatibility across various competitive platforms.
     */
    public ArrayList<String> generateUsernames(String[] arr) {
        ArrayList<String> list = new ArrayList<>(arr.length);
        for (String s : arr) {
            list.add(s);
        }
        return generateUsernames(list);
    }
}
```

---

### Python3

```python
class Solution:
    def generateUsernames(self, arr: list[str]) -> list[str]:
        """
        Generates unique usernames according to the system registration rules.
        
        :param arr: List of requested username strings
        :return: List of assigned unique username strings
        """
        res = []
        
        # Step 1: Pre-populate the used set with all original input strings
        # This reserves all naturally requested usernames upfront
        used = set(arr)
        
        # Step 2: Track base usernames that have already been assigned as-is
        processed = set()
        
        # Step 3: Map each base string to its next candidate integer suffix
        next_suffix = {}

        for s in arr:
            if s not in processed:
                # First time seeing this base username
                processed.add(s)
                res.append(s)
                next_suffix[s] = 1
            else:
                # Duplicate occurrence: resume suffix probe from cached index
                suffix = next_suffix.get(s, 1)
                candidate = f"{s}{suffix}"
                
                while candidate in used:
                    suffix += 1
                    candidate = f"{s}{suffix}"
                
                # Mark newly generated username as taken and cache next candidate
                used.add(candidate)
                next_suffix[s] = suffix + 1
                res.append(candidate)

        return res
```

---

### C++ (17)

```cpp
#include <vector>
#include <string>
#include <unordered_set>
#include <unordered_map>

class Solution {
public:
    /**
     * Generates unique usernames in sequential order of requests.
     * 
     * @param arr Vector of requested username strings
     * @return Vector of unique assigned usernames
     */
    std::vector<std::string> generateUsernames(std::vector<std::string>& arr) {
        std::vector<std::string> res;
        res.reserve(arr.size());

        // Step 1: Pre-reserve all usernames present in the input vector
        std::unordered_set<std::string> used(arr.begin(), arr.end());

        // Step 2: Track base usernames that have been assigned
        std::unordered_set<std::string> processed;

        // Step 3: Cache next available suffix integer for each base username
        std::unordered_map<std::string, int> nextSuffix;

        for (const std::string& s : arr) {
            if (processed.find(s) == processed.end()) {
                // First appearance of base username
                processed.insert(s);
                res.push_back(s);
                nextSuffix[s] = 1;
            } else {
                // Duplicate request: resume search from cached suffix
                int suffix = nextSuffix[s];
                std::string candidate = s + std::to_string(suffix);

                while (used.find(candidate) != used.end()) {
                    suffix++;
                    candidate = s + std::to_string(suffix);
                }

                // Reserve the newly created username and update cache
                used.insert(candidate);
                nextSuffix[s] = suffix + 1;
                res.push_back(std::move(candidate));
            }
        }

        return res;
    }
};
```

---

### C#

```csharp
using System;
using System.Collections.Generic;

public class Solution {

    /// <summary>
    /// Generates unique usernames according to the registration specifications.
    /// </summary>
    /// <param name="arr">List of requested username strings</param>
    /// <returns>List of unique assigned usernames</returns>
    public List<string> GenerateUsernames(List<string> arr) {
        List<string> res = new List<string>(arr.Count);

        // Step 1: Pre-reserve all usernames present in the input collection
        HashSet<string> used = new HashSet<string>(arr);

        // Step 2: Track base usernames that have already been assigned as-is
        HashSet<string> processed = new HashSet<string>();

        // Step 3: Cache the next integer suffix to test for each base username
        Dictionary<string, int> nextSuffix = new Dictionary<string, int>();

        foreach (string s in arr) {
            if (!processed.Contains(s)) {
                // First time seeing this base username
                processed.Add(s);
                res.Add(s);
                nextSuffix[s] = 1;
            } else {
                // Duplicate request: retrieve cached suffix
                int suffix = nextSuffix.ContainsKey(s) ? nextSuffix[s] : 1;
                string candidate = s + suffix;

                while (used.Contains(candidate)) {
                    suffix++;
                    candidate = s + suffix;
                }

                // Reserve the candidate and update the next suffix pointer
                used.Add(candidate);
                nextSuffix[s] = suffix + 1;
                res.Add(candidate);
            }
        }

        return res;
    }

    /// <summary>
    /// Overload for array input compatibility.
    /// </summary>
    public List<string> generateUsernames(string[] arr) {
        List<string> list = new List<string>(arr);
        List<string> resultList = GenerateUsernames(list);
        return resultList;
    }
}
```

---

### Javascript (Node v22)

```javascript
class Solution {
    /**
     * Generates unique usernames for sequential user registrations.
     * 
     * @param {string[]} arr - Array of requested username strings
     * @return {string[]} - Array of unique assigned username strings
     */
    generateUsernames(arr) {
        const res = [];

        // Step 1: Pre-reserve all usernames present in the input array
        const used = new Set(arr);

        // Step 2: Track base usernames that have already been assigned as-is
        const processed = new Set();

        // Step 3: Cache the next available integer suffix for each base username
        const nextSuffix = new Map();

        for (let i = 0; i < arr.length; i++) {
            const s = arr[i];

            if (!processed.has(s)) {
                // First time seeing this base username
                processed.add(s);
                res.push(s);
                nextSuffix.set(s, 1);
            } else {
                // Duplicate request: resume search from cached suffix
                let suffix = nextSuffix.get(s) || 1;
                let candidate = s + suffix;

                while (used.has(candidate)) {
                    suffix++;
                    candidate = s + suffix;
                }

                // Reserve candidate username and update next suffix pointer
                used.add(candidate);
                nextSuffix.set(s, suffix + 1);
                res.push(candidate);
            }
        }

        return res;
    }
}
```

