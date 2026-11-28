# LeetCode POTD: Maximum Number of Items From Sale II

## Problem Statement

You are given a 2D integer array `items` where $\text{items}[i] = [\text{factor}_i, \text{price}_i]$, and an integer `budget`.

You are visiting a store that is hosting a promotional sale with specific rules:
1. You can buy any item $i$ as many times as you want by paying its price $\text{price}_i$.
2. For every copy of item $i$ that you purchase, you may receive **at most one free copy** of another item $j$ ($j \neq i$) such that $\text{factor}_j$ is a multiple of $\text{factor}_i$ (i.e., $\text{factor}_j \pmod{\text{factor}_i} == 0$).
3. For each ordered pair $(i, j)$, you can receive a free copy of item $j$ from purchases of item $i$ **at most once**.
4. In other words, purchasing item $i$ $k$ times allows you to claim at most $\min(k, C_i)$ free items, where $C_i$ is the count of distinct items $j$ ($j \neq i$) whose $\text{factor}_j$ is divisible by $\text{factor}_i$.
5. An item received for free cannot be used to trigger promotions for other items.

Return the **maximum total number of items** (purchased + promotional free items) that you can obtain without exceeding `budget`.

---

## 1. Executive Summary & Problem Classification

- **Platform:** LeetCode Daily Question (POTD)
- **Problem ID:** 3947
- **Difficulty:** Medium / Hard
- **Core Topics:** Greedy, Number Theory, Sieve of Eratosthenes, Frequency Counting, Prefix Processing
- **Key Insight:** 
  Unlike *Maximum Number of Items From Sale I* (where purchasing a single copy unlocked every divisible item all at once, leading to an unbounded knapsack formulation), *Sale II* restricts each purchased copy to granting **at most one bonus item**. 
  
  Therefore:
  - The first $C_i$ copies of item $i$ yield **2 items** for $\text{price}_i$ (effective cost: $\frac{\text{price}_i}{2}$).
  - Any subsequent copies of item $i$ yield only **1 item** for $\text{price}_i$ (effective cost: $\text{price}_i$).
  - Any copy purchased purely for 1 item should always be bought using the globally cheapest item, costing $P_{\min} = \min_{k} \text{price}_k$.
  - Consequently, buying a bonus-yielding copy of item $i$ is only economically viable if $\text{price}_i \le 2 \cdot P_{\min}$. If $\text{price}_i > 2 \cdot P_{\min}$, buying 2 copies of the cheapest item yields 2 items for strictly less cost!

---

## 2. Examples & Visual Walkthrough

### Example 1

**Input:**
```text
items = [[1, 6], [2, 4], [3, 5]]
budget = 19
```

**Step 1: Determine Divisibility Counts ($C_i$)**
- Item 0: $\text{factor} = 1, \text{price} = 6$.
  - Multiples of 1 among other items: Item 1 ($\text{factor} = 2$) and Item 2 ($\text{factor} = 3$).
  - $C_0 = 2$ eligible free items.
  - Up to 2 copies of Item 0 can each yield 2 items at cost 6 (effective cost: $3.00$ per item).
- Item 1: $\text{factor} = 2, \text{price} = 4$.
  - Multiples of 2 among other items: None.
  - $C_1 = 0$.
- Item 2: $\text{factor} = 3, \text{price} = 5$.
  - Multiples of 3 among other items: None.
  - $C_2 = 0$.

**Step 2: Identify Cheapest Base Item**
- $P_{\min} = \min(6, 4, 5) = 4$ (Item 1).
- 1 item without bonus costs 4.
- 2 items without bonus cost $2 \times 4 = 8$.

**Step 3: Evaluate Promotional Offers**
- Item 0 offers 2 items for price 6.
  - Since $6 \le 2 \times 4 = 8$, this offer is profitable!
  - Maximum copies available with bonus: $C_0 = 2$.
  - Total cost for 2 copies: $2 \times 6 = 12$. Total items: $2 \times 2 = 4$.
  - Remaining budget: $19 - 12 = 7$.

**Step 4: Spend Remaining Budget on Cheapest Item**
- Remaining budget: 7.
- Number of Item 1 copies we can buy: $\lfloor 7 / 4 \rfloor = 1$.
- Total items obtained: $4 + 1 = 5$.
- Remaining budget: $7 - 4 = 3$.

**Output:** `5`

---

### Example 2

**Input:**
```text
items = [[2, 8], [1, 10], [6, 6], [4, 12], [5, 20], [5, 17]]
budget = 35
```

**Step 1: Divisibility Counts ($C_i$)**
- Item 0 ($\text{factor} = 2, \text{price} = 8$): Multiples are $\{6, 4\} \implies C_0 = 2$.
- Item 1 ($\text{factor} = 1, \text{price} = 10$): Multiples are $\{2, 6, 4, 5, 5\} \implies C_1 = 5$.
- Item 2 ($\text{factor} = 6, \text{price} = 6$): Multiples: None $\implies C_2 = 0$.
- Item 3 ($\text{factor} = 4, \text{price} = 12$): Multiples: None $\implies C_3 = 0$.
- Item 4 ($\text{factor} = 5, \text{price} = 20$): Multiples: $\{5\} \implies C_4 = 1$.
- Item 5 ($\text{factor} = 5, \text{price} = 17$): Multiples: $\{5\} \implies C_5 = 1$.

**Step 2: Base Price Analysis**
- $P_{\min} = \min(8, 10, 6, 12, 20, 17) = 6$ (Item 2).
- Threshold for a 2-for-price offer to be worthwhile: $\text{price} \le 2 \times P_{\min} = 12$.

**Step 3: Profitable Bonus Pool**
- Item 0: Price $8 \le 12$, count $C_0 = 2$. (Gives 2 items for 8).
- Item 1: Price $10 \le 12$, count $C_1 = 5$. (Gives 2 items for 10).
- Item 3: Price $12 \le 12$, but $C_3 = 0$.
- Items 4 & 5: Prices 20 and 17 $> 12$ (unprofitable compared to buying Item 2 twice).

**Step 4: Greedy Acquisition**
1. Buy up to 2 copies of Item 0 (Cost 8 each):
   - Budget allows $\lfloor 35 / 8 \rfloor = 4 \implies \text{take } \min(2, 4) = 2$ copies.
   - Cost: $2 \times 8 = 16$. Items: $2 \times 2 = 4$.
   - Remaining budget: $35 - 16 = 19$.
2. Buy copies of Item 1 (Cost 10 each):
   - Budget allows $\lfloor 19 / 10 \rfloor = 1 \implies \text{take } \min(5, 1) = 1$ copy.
   - Cost: $1 \times 10 = 10$. Items: $1 \times 2 = 2$.
   - Remaining budget: $19 - 10 = 9$.
3. Fill leftover budget with cheapest base item (Item 2, Cost 6):
   - Budget allows $\lfloor 9 / 6 \rfloor = 1$ copy.
   - Cost: 6. Items: 1.
   - Remaining budget: $9 - 6 = 3$.

**Total Items:** $4 + 2 + 1 = 7$.

**Output:** `7`

---

## 3. Constraints & Scale Analysis

- $1 \le \text{items.length} \le 10^5$
- $\text{items}[i] = [\text{factor}_i, \text{price}_i]$
- $1 \le \text{factor}_i \le 10^5$
- $1 \le \text{price}_i \le 10^9$
- $1 \le \text{budget} \le 10^9$

### Mathematical Scale Implications
1. **$N \le 10^5$ and $\text{factor} \le 10^5$**:
   Comparing every pair $(i, j)$ takes $\mathcal{O}(N^2)$ operations, which would result in $\sim 10^{10}$ steps and cause a **Time Limit Exceeded (TLE)**.
   Instead, we use a **Harmonic Sieve** over frequency arrays. Sum of harmonic series up to $M = 10^5$ is:
   $$\sum_{k=1}^{M} \frac{M}{k} = M \ln M \approx 10^5 \times 11.5 \approx 1.15 \times 10^6 \text{ operations}$$
   This runs in less than $20\text{ ms}$!

2. **$\text{budget} \le 10^9$ and $\text{price}_i \le 10^9$**:
   We cannot do dynamic programming indexed by budget (e.g. `dp[budget]`).
   The decision process must be purely analytical: **Greedy Selection** with 64-bit integer arithmetic (`long` in Java/C++, `int64` in C#).

---

## 4. Visual Architecture & Algorithmic Proof

### Economic Equivalence Table

| Action | Cost | Items Acquired | Effective Cost per Item | Availability Limit |
| :--- | :---: | :---: | :---: | :---: |
| **Bonus Copy of Item $i$** | $\text{price}_i$ | 2 | $\frac{\text{price}_i}{2}$ | At most $C_i$ copies |
| **Plain Copy of Item $i$** | $\text{price}_i$ | 1 | $\text{price}_i$ | $\infty$ |
| **Cheapest Item $k$** | $P_{\min}$ | 1 | $P_{\min}$ | $\infty$ |

### Lemma 1: The Cutoff Boundary
> **Claim:** Buying a promotional bonus copy of item $i$ (costing $\text{price}_i$ for 2 items) is optimal if and only if $\text{price}_i \le 2 \cdot P_{\min}$.

**Proof:**
Suppose we spend $\text{price}_i$ on one bonus copy of item $i$, yielding 2 items.
Alternatively, we could purchase 2 plain copies of the cheapest item $k$, which yields 2 items at a total cost of $2 \cdot P_{\min}$.
- If $\text{price}_i > 2 \cdot P_{\min}$, the plain purchase of 2 cheapest items gives 2 items for strictly less money, leaving more budget for further purchases.
- If $\text{price}_i \le 2 \cdot P_{\min}$, the promotional copy yields 2 items for less than or equal money compared to any plain purchases.
- Therefore, any promotional offer with $\text{price}_i > 2 \cdot P_{\min}$ can be safely ignored. $\blacksquare$

### Lemma 2: Greedy Ordering of Promotional Bundles
> **Claim:** Among all promotional offers satisfying $\text{price}_i \le 2 \cdot P_{\min}$, it is always optimal to purchase bundles in ascending order of $\text{price}_i$.

**Proof:**
Every promotional bundle yields identical utility (exactly 2 items). When two bundles yield identical utility, the one with strictly lower cost dominates because it leaves a larger residual budget. Hence, sorting by $\text{price}_i$ ascending is strictly optimal. $\blacksquare$

```
                   BUDGET ALLOCATION PIPELINE
                   
  Total Budget ──┐
                 ▼
       ┌──────────────────┐
       │ Sort Bonus Deals │  (Cheapest price first)
       │ price_i <= 2*Pmin│
       └─────────┬────────┘
                 │
                 ▼
       ┌──────────────────┐
       │ Greedily Consume │  Take = min(C_i, floor(budget / price_i))
       │   Bonus Packs    │  Items += 2 * Take
       └─────────┬────────┘  Budget -= Take * price_i
                 │
                 ▼
       ┌──────────────────┐
       │   Exhaust Rest   │  Items += floor(budget / Pmin)
       │  on Cheapest Pmin│
       └─────────┬────────┘
                 │
                 ▼
            Max Items
```

---

## 5. Flowchart

```mermaid
flowchart TD
    A[Start: items, budget] --> B[Find P_min = min price_i]
    B --> C[Compute Frequency Array of factors: freq[f]]
    C --> D[Sieve: For each factor f, count multiples in freq]
    D --> E[For each item i: C_i = multiples of factor_i - 1]
    E --> F[Filter profitable items: price_i <= 2 * P_min AND C_i > 0]
    F --> G[Aggregate total available bonus copies by price]
    G --> H[Sort distinct prices in ascending order]
    H --> I{For each price p with count k}
    I -->|budget >= p| J[take = min k, budget // p]
    J --> K[total_items += 2 * take; budget -= take * p]
    K --> I
    I -->|All bonus prices visited or budget exhausted| L[total_items += budget // P_min]
    L --> M[Return total_items]
```

---

## 6. Naive vs. Optimal Comparison

| Metric | Naive Brute Force / Pairwise | Optimal Sieve + Greedy |
| :--- | :--- | :--- |
| **Divisibility Counting** | Compare every pair: $\mathcal{O}(N^2)$ | Sieve over frequency array: $\mathcal{O}(M \log M + N)$ |
| **Optimization Strategy** | Dynamic Programming over budget $\mathcal{O}(\text{budget})$ | Sort & Greedy Allocation: $\mathcal{O}(N \log N)$ |
| **Time Complexity** | $\mathcal{O}(N^2 + N \cdot \text{budget})$ (TLE) | $\mathcal{O}(M \log M + N \log N)$ ($\approx 35\text{ ms}$) |
| **Space Complexity** | $\mathcal{O}(N^2)$ or $\mathcal{O}(\text{budget})$ (MLE) | $\mathcal{O}(M + N)$ ($\approx 15\text{ MB}$) |
| **Max $N$ Handled** | $\approx 10^3$ | $\ge 10^5$ |

---

## 7. Pseudo-Code: Progression from Naive to Pro

### Method 1: Pairwise Counting (Naive - TLE)
```text
function countItemsNaive(items, budget):
    P_min = min(item.price for item in items)
    bonus_items = []
    
    for i from 0 to len(items) - 1:
        C_i = 0
        for j from 0 to len(items) - 1:
            if i != j and items[j].factor % items[i].factor == 0:
                C_i += 1
        if C_i > 0 and items[i].price <= 2 * P_min:
            bonus_items.append((items[i].price, C_i))
            
    sort bonus_items by price ascending
    total_items = 0
    for (price, count) in bonus_items:
        take = min(count, budget // price)
        total_items += 2 * take
        budget -= take * price
        
    total_items += budget // P_min
    return total_items
```

### Method 2: Sieve Counting + Greedy Allocation (Optimal - Pro)
```text
function maxItems(items, budget):
    P_min = min(item.price for item in items)
    max_f = max(item.factor for item in items)
    
    freq = array of size (max_f + 1) filled with 0
    for item in items:
        freq[item.factor] += 1
        
    multiples = array of size (max_f + 1) filled with 0
    for f from 1 to max_f:
        if freq[f] > 0:
            for m from f to max_f step f:
                multiples[f] += freq[m]
                
    price_to_count = map()
    cutoff = 2 * P_min
    
    for item in items:
        f = item.factor
        p = item.price
        C_i = multiples[f] - 1  // exclude self
        if C_i > 0 and p <= cutoff:
            price_to_count[p] += C_i
            
    sorted_prices = sort keys of price_to_count ascending
    total_items = 0
    
    for p in sorted_prices:
        available = price_to_count[p]
        take = min(available, budget // p)
        total_items += 2 * take
        budget -= take * p
        if budget < p:
            break
            
    total_items += budget // P_min
    return total_items
```

---

## 8. Multi-Language Implementations

### Java (21)

```java
import java.util.*;

class Solution {
    /**
     * Computes the maximum total number of items obtained within budget.
     * 
     * @param items  2D array where items[i] = [factor_i, price_i]
     * @param budget maximum total cost allowed
     * @return maximum items (purchased + promotional free)
     */
    public int maximumSaleItems(int[][] items, int budget) {
        int n = items.length;
        long pMin = Long.MAX_VALUE;
        int maxFactor = 0;

        for (int[] item : items) {
            pMin = Math.min(pMin, item[1]);
            maxFactor = Math.max(maxFactor, item[0]);
        }

        // Frequency array of factors
        int[] freq = new int[maxFactor + 1];
        for (int[] item : items) {
            freq[item[0]]++;
        }

        // Sieve to count multiples of each factor present in the dataset
        int[] multiplesCount = new int[maxFactor + 1];
        for (int f = 1; f <= maxFactor; f++) {
            if (freq[f] > 0) {
                int sum = 0;
                for (int m = f; m <= maxFactor; m += f) {
                    sum += freq[m];
                }
                multiplesCount[f] = sum;
            }
        }

        // Group bonus counts by price for profitable items (price <= 2 * pMin)
        long cutoffPrice = 2 * pMin;
        Map<Long, Long> priceToBonusCopies = new HashMap<>();

        for (int[] item : items) {
            int factor = item[0];
            long price = item[1];
            long c_i = multiplesCount[factor] - 1; // Exclude itself

            if (c_i > 0 && price <= cutoffPrice) {
                priceToBonusCopies.put(price, priceToBonusCopies.getOrDefault(price, 0L) + c_i);
            }
        }

        // Sort promotional prices in ascending order
        List<Long> prices = new ArrayList<>(priceToBonusCopies.keySet());
        Collections.sort(prices);

        long totalItems = 0;
        long remainingBudget = budget;

        // Greedily consume the cheapest promotional bundles (2 items per bundle)
        for (long p : prices) {
            long available = priceToBonusCopies.get(p);
            long canBuy = remainingBudget / p;
            long take = Math.min(available, canBuy);

            totalItems += 2 * take;
            remainingBudget -= take * p;
        }

        // Spend all remaining budget on the cheapest plain item
        totalItems += (remainingBudget / pMin);

        return (int) totalItems;
    }
}
```

---

### Python3

```python
from collections import defaultdict
from typing import List

class Solution:
    def maximumSaleItems(self, items: List[List[int]], budget: int) -> int:
        """
        Computes maximum items obtainable within budget using Sieve + Greedy selection.
        """
        p_min = min(price for _, price in items)
        max_factor = max(factor for factor, _ in items)

        # Build frequency array of factors
        freq = [0] * (max_factor + 1)
        for factor, _ in items:
            freq[factor] += 1

        # Sieve: count multiples for each factor
        multiples = [0] * (max_factor + 1)
        for f in range(1, max_factor + 1):
            if freq[f] > 0:
                multiples[f] = sum(freq[m] for m in range(f, max_factor + 1, f))

        # Filter and group profitable promotional offers
        cutoff = 2 * p_min
        price_to_bonuses = defaultdict(int)

        for factor, price in items:
            c_i = multiples[factor] - 1  # Exclude self
            if c_i > 0 and price <= cutoff:
                price_to_bonuses[price] += c_i

        total_items = 0
        sorted_prices = sorted(price_to_bonuses.keys())

        # Greedily purchase 2-for-price bundles starting with lowest cost
        for p in sorted_prices:
            available = price_to_bonuses[p]
            take = min(available, budget // p)
            total_items += 2 * take
            budget -= take * p

        # Exhaust residual budget on the globally cheapest single item
        total_items += budget // p_min

        return total_items
```

---

### C++ (17)

```cpp
#include <vector>
#include <algorithm>
#include <climits>
#include <map>

using namespace std;

class Solution {
public:
    /**
     * @brief Computes maximum items obtainable via Harmonic Sieve + Greedy optimization.
     * @param items 2D vector where items[i] = [factor_i, price_i]
     * @param budget Total maximum expenditure allowed
     * @return Maximum count of items
     */
    int maximumSaleItems(vector<vector<int>>& items, int budget) {
        long long pMin = LLONG_MAX;
        int maxFactor = 0;

        for (const auto& it : items) {
            pMin = min(pMin, (long long)it[1]);
            maxFactor = max(maxFactor, it[0]);
        }

        // Count occurrences of each factor
        vector<int> freq(maxFactor + 1, 0);
        for (const auto& it : items) {
            freq[it[0]]++;
        }

        // Harmonic Sieve to find count of multiples for each factor
        vector<int> multiples(maxFactor + 1, 0);
        for (int f = 1; f <= maxFactor; ++f) {
            if (freq[f] > 0) {
                int count = 0;
                for (int m = f; m <= maxFactor; m += f) {
                    count += freq[m];
                }
                multiples[f] = count;
            }
        }

        // Use std::map to automatically maintain sorted price order
        long long cutoff = 2 * pMin;
        map<long long, long long> bonusDeals;

        for (const auto& it : items) {
            int factor = it[0];
            long long price = it[1];
            long long c_i = multiples[factor] - 1; // Exclude the item itself

            if (c_i > 0 && price <= cutoff) {
                bonusDeals[price] += c_i;
            }
        }

        long long totalItems = 0;
        long long remainingBudget = budget;

        // Greedily pick bonus deals from cheapest to dearest
        for (auto const& [price, count] : bonusDeals) {
            long long take = min(count, remainingBudget / price);
            totalItems += 2 * take;
            remainingBudget -= take * price;
        }

        // Buy plain copies of the cheapest item with remaining budget
        totalItems += (remainingBudget / pMin);

        return (int)totalItems;
    }
};
```

---

### C

```c
#include <stdio.h>
#include <stdlib.h>
#include <limits.h>

typedef struct {
    long long price;
    long long count;
} BonusDeal;

static int compareDeals(const void* a, const void* b) {
    long long diff = ((const BonusDeal*)a)->price - ((const BonusDeal*)b)->price;
    if (diff < 0) return -1;
    if (diff > 0) return 1;
    return 0;
}

/**
 * Computes maximum total items obtainable using C standard library.
 */
int maximumSaleItems(int** items, int itemsSize, int* itemsColSize, int budget) {
    long long pMin = LLONG_MAX;
    int maxFactor = 0;

    for (int i = 0; i < itemsSize; ++i) {
        if (items[i][1] < pMin) {
            pMin = items[i][1];
        }
        if (items[i][0] > maxFactor) {
            maxFactor = items[i][0];
        }
    }

    // Allocate frequency array
    int* freq = (int*)calloc(maxFactor + 1, sizeof(int));
    for (int i = 0; i < itemsSize; ++i) {
        freq[items[i][0]]++;
    }

    // Allocate multiples count array
    int* multiples = (int*)calloc(maxFactor + 1, sizeof(int));
    for (int f = 1; f <= maxFactor; ++f) {
        if (freq[f] > 0) {
            int sum = 0;
            for (int m = f; m <= maxFactor; m += f) {
                sum += freq[m];
            }
            multiples[f] = sum;
        }
    }

    // Collect qualifying bonus deals
    long long cutoff = 2 * pMin;
    BonusDeal* deals = (BonusDeal*)malloc(itemsSize * sizeof(BonusDeal));
    int dealsCount = 0;

    for (int i = 0; i < itemsSize; ++i) {
        int factor = items[i][0];
        long long price = items[i][1];
        long long c_i = multiples[factor] - 1;

        if (c_i > 0 && price <= cutoff) {
            deals[dealsCount].price = price;
            deals[dealsCount].count = c_i;
            dealsCount++;
        }
    }

    // Free sieve buffers early
    free(freq);
    free(multiples);

    // Sort deals by price ascending
    qsort(deals, dealsCount, sizeof(BonusDeal), compareDeals);

    // Compress identical prices into contiguous entries
    long long totalItems = 0;
    long long remainingBudget = budget;
    int idx = 0;
    while (idx < dealsCount) {
        long long currentPrice = deals[idx].price;
        long long totalAvailable = 0;
        while (idx < dealsCount && deals[idx].price == currentPrice) {
            totalAvailable += deals[idx].count;
            idx++;
        }

        long long take = remainingBudget / currentPrice;
        if (take > totalAvailable) {
            take = totalAvailable;
        }

        totalItems += 2 * take;
        remainingBudget -= take * currentPrice;
    }

    free(deals);

    // Fill remaining budget with cheapest item
    totalItems += (remainingBudget / pMin);

    return (int)totalItems;
}
```

---

### C#

```csharp
using System;
using System.Collections.Generic;

public class Solution {
    /// <summary>
    /// Computes maximum items obtainable within budget using Sieve + Sorted Greedy.
    /// </summary>
    public int MaximumSaleItems(int[][] items, int budget) {
        long pMin = long.MaxValue;
        int maxFactor = 0;

        foreach (var item in items) {
            if (item[1] < pMin) pMin = item[1];
            if (item[0] > maxFactor) maxFactor = item[0];
        }

        // Build factor frequencies
        int[] freq = new int[maxFactor + 1];
        foreach (var item in items) {
            freq[item[0]]++;
        }

        // Sieve multiples calculation
        int[] multiples = new int[maxFactor + 1];
        for (int f = 1; f <= maxFactor; f++) {
            if (freq[f] > 0) {
                int sum = 0;
                for (int m = f; m <= maxFactor; m += f) {
                    sum += freq[m];
                }
                multiples[f] = sum;
            }
        }

        // Map price to bonus count
        long cutoff = 2 * pMin;
        var bonusDeals = new SortedDictionary<long, long>();

        foreach (var item in items) {
            int factor = item[0];
            long price = item[1];
            long c_i = multiples[factor] - 1;

            if (c_i > 0 && price <= cutoff) {
                if (!bonusDeals.ContainsKey(price)) {
                    bonusDeals[price] = 0;
                }
                bonusDeals[price] += c_i;
            }
        }

        long totalItems = 0;
        long remainingBudget = budget;

        // Greedily consume promotions in ascending price order
        foreach (var kvp in bonusDeals) {
            long price = kvp.Key;
            long available = kvp.Value;
            long take = Math.Min(available, remainingBudget / price);

            totalItems += 2 * take;
            remainingBudget -= take * price;
        }

        // Residual budget to buy plain copies of the cheapest item
        totalItems += (remainingBudget / pMin);

        return (int)totalItems;
    }
}
```

---

### Javascript (Node.js v22)

```javascript
/**
 * @param {number[][]} items
 * @param {number} budget
 * @return {number}
 */
var maximumSaleItems = function(items, budget) {
    let pMin = Infinity;
    let maxFactor = 0;

    for (let i = 0; i < items.length; i++) {
        const [factor, price] = items[i];
        if (price < pMin) pMin = price;
        if (factor > maxFactor) maxFactor = factor;
    }

    // Frequency array of factors
    const freq = new Int32Array(maxFactor + 1);
    for (let i = 0; i < items.length; i++) {
        freq[items[i][0]]++;
    }

    // Harmonic Sieve for multiples count
    const multiples = new Int32Array(maxFactor + 1);
    for (let f = 1; f <= maxFactor; f++) {
        if (freq[f] > 0) {
            let sum = 0;
            for (let m = f; m <= maxFactor; m += f) {
                sum += freq[m];
            }
            multiples[f] = sum;
        }
    }

    // Group available bonus counts by price
    const cutoff = 2 * pMin;
    const priceToBonus = new Map();

    for (let i = 0; i < items.length; i++) {
        const [factor, price] = items[i];
        const c_i = multiples[factor] - 1;

        if (c_i > 0 && price <= cutoff) {
            priceToBonus.set(price, (priceToBonus.get(price) || 0) + c_i);
        }
    }

    // Sort distinct prices ascending
    const sortedPrices = Array.from(priceToBonus.keys()).sort((a, b) => a - b);
    let totalItems = 0n;
    let bigBudget = BigInt(budget);
    const bigPMin = BigInt(pMin);

    for (const p of sortedPrices) {
        const available = BigInt(priceToBonus.get(p));
        const bigP = BigInt(p);
        const canBuy = bigBudget / bigP;
        const take = available < canBuy ? available : canBuy;

        totalItems += 2n * take;
        bigBudget -= take * bigP;
    }

    // Plain cheapest purchases
    totalItems += bigBudget / bigPMin;

    return Number(totalItems);
};
```

---

### Typescript

```typescript
/**
 * Computes maximum items obtainable with budget.
 * Uses typed arrays for performance and BigInt for large budget precision.
 */
function maximumSaleItems(items: number[][], budget: number): number {
    let pMin = Infinity;
    let maxFactor = 0;

    for (const [factor, price] of items) {
        if (price < pMin) pMin = price;
        if (factor > maxFactor) maxFactor = factor;
    }

    // Build factor frequency array
    const freq = new Int32Array(maxFactor + 1);
    for (const [factor] of items) {
        freq[factor]++;
    }

    // Harmonic sieve to count multiples
    const multiples = new Int32Array(maxFactor + 1);
    for (let f = 1; f <= maxFactor; f++) {
        if (freq[f] > 0) {
            let count = 0;
            for (let m = f; m <= maxFactor; m += f) {
                count += freq[m];
            }
            multiples[f] = count;
        }
    }

    // Collect eligible promotional packages
    const cutoff = 2 * pMin;
    const priceMap = new Map<number, number>();

    for (const [factor, price] of items) {
        const c_i = multiples[factor] - 1;
        if (c_i > 0 && price <= cutoff) {
            priceMap.set(price, (priceMap.get(price) ?? 0) + c_i);
        }
    }

    const sortedPrices = Array.from(priceMap.keys()).sort((a, b) => a - b);
    let totalItems = 0n;
    let remainingBudget = BigInt(budget);
    const bigPMin = BigInt(pMin);

    for (const p of sortedPrices) {
        const available = BigInt(priceMap.get(p)!);
        const bigP = BigInt(p);
        const canAfford = remainingBudget / bigP;
        const take = available < canAfford ? available : canAfford;

        totalItems += 2n * take;
        remainingBudget -= take * bigP;
    }

    totalItems += remainingBudget / bigPMin;

    return Number(totalItems);
}
```

---

## 9. Complexity Breakdown

### Time Complexity
1. **Finding Extremes ($P_{\min}, M$):** $\mathcal{O}(N)$ passes over the input array.
2. **Frequency Array Construction:** $\mathcal{O}(N)$.
3. **Harmonic Sieve:**
   $$\sum_{f=1}^{M} \frac{M}{f} = \mathcal{O}(M \ln M)$$
   For $M = 10^5$, $M \ln M \approx 1.15 \times 10^6$ operations, which takes $< 15\text{ ms}$.
4. **Filtering and Sorting Prices:**
   There are at most $N$ qualifying items, yielding at most $N$ unique prices.
   Sorting distinct prices takes $\mathcal{O}(K \log K)$ where $K \le \min(N, \text{unique prices})$.
5. **Greedy Traversal:** $\mathcal{O}(K)$ iterations.
- **Total Time Complexity:** $\mathcal{O}(M \ln M + N \log N)$, completely independent of `budget`.

### Space Complexity
1. **Frequency & Sieve Arrays:** Two arrays of size $M + 1 \implies 2 \times 10^5 \times 4\text{ bytes} \approx 800\text{ KB}$.
2. **Price Hash Map / Tree:** At most $N$ keys $\implies \mathcal{O}(N)$ memory.
- **Total Auxiliary Space:** $\mathcal{O}(M + N)$, requiring $< 20\text{ MB}$ across all runtimes.

---

## 10. Edge Cases & Defensive Engineering

1. **`budget < pMin`**:
   The user cannot afford even a single item. The greedy loop will set $\text{take} = 0$, and $\lfloor \text{budget} / P_{\min} \rfloor = 0$. Return value is correctly `0`.
2. **All $C_i = 0$ (No item divides any other item)**:
   The map of promotional deals remains empty. The entire budget is spent on buying $\lfloor \text{budget} / P_{\min} \rfloor$ copies of the cheapest item.
3. **Identical Factors and Prices**:
   Items can share the exact same factor (e.g. $[2, 5]$ and $[2, 5]$). The sieve counts both, meaning $C_i$ will correctly include identical factors from other elements in the list because $2 \mid 2$.
4. **Large Budget and Item Counts ($10^9$)**:
   Since values exceed $2^{31}-1$, 64-bit integer arithmetic (`long` in Java/C++, `BigInt` in JS) prevents integer overflow during multiplications like $2 \times \text{take}$.
5. **Factor $= 1$**:
   A factor of 1 divides all items in the array. Thus, $C_i = N - 1$. The sieve handles $f=1$ natively without special-case branches.

---

## 11. Testing & Validation Matrix

| Test Case | `items` | `budget` | Expected | Key Verification Angle |
| :---: | :---: | :---: | :---: | :--- |
| **Ex 1** | `[[1,6],[2,4],[3,5]]` | `19` | `5` | Single promotional item, mixed residual budget |
| **Ex 2** | `[[2,8],[1,10],[6,6],[4,12],[5,20],[5,17]]` | `35` | `7` | Unprofitable promotions filtered out ($20 > 12$) |
| **Edge 1** | `[[1,100]]` | `50` | `0` | Budget strictly less than minimum item price |
| **Edge 2** | `[[3,10],[5,10],[7,10]]` | `55` | `5` | Coprime factors: zero bonus promotions possible |
| **Edge 3** | `[[1,1],[1,1],[1,1]]` | `10` | `10` | Multiple identical factors and prices |
| **Scale** | $10^5$ items, max factor $10^5$ | $10^9$ | Large | Confirms $\mathcal{O}(M \log M)$ runs well within $1\text{ s}$ limit |

---

## 12. Key Takeaways & Interview Cheat Sheet

- **Look for Equivalence Ratios:** When a promotion gives 1 free copy per purchase up to a limit, express it as a limited bundle: $(\text{price}, 2 \text{ items})$.
- **Benchmark Against the Default Action:** The default cheapest way to buy 1 item is $P_{\min}$. Therefore, buying 2 items without a deal costs $2 P_{\min}$. Any promotional pack costing $> 2 P_{\min}$ is inferior and should be ignored.
- **Harmonic Sieve for Multiples:** Never do pairwise modulo $\mathcal{O}(N^2)$ when coordinates/factors are bounded by $10^5$. Count frequencies, then iterate through multiples with a step size of $f$ in $\mathcal{O}(M \log M)$ time.
- **Greedy Dominance:** When all remaining candidate bundles yield the exact same utility (2 items), sorting strictly by price ascending gives the mathematically optimal solution.
