Here are the enhanced, comprehensive notes for the **Rod Cutting Problem** and its high-yield exam variants.

---

# 📝 Topic: Rod Cutting & Variants

## 1. The Base Problem: Standard Rod Cutting

### **Problem Statement**

Given a rod of length $N$ and a price array `price[]` where `price[i]` is the price of a piece of length $i+1$, determine the **maximum profit** obtainable by cutting up the rod and selling the pieces.

- **Input:** Rod Length $N$, `price[]` = $\{p_1, p_2, \dots, p_n\}$
    
- **Output:** Maximum Profit (Integer)
    

### **Intuition & Logic**

This is a classic **Unbounded Knapsack** problem. The core difficulty is that there are $2^{N-1}$ ways to cut the rod, which is too many to brute force.

However, the problem has **Optimal Substructure**:

If the optimal solution for a rod of length 8 involves cutting it into a piece of length 2 and a piece of length 6, then the piece of length 6 _must also be cut optimally_.

We define our state as:

> **`dp[i]` = The maximum profit obtainable for a rod of length `i`.**

To solve for length $i$, we consider every possible "first cut". We can cut off a piece of length $j$ (where $1 \le j \le i$), sell that piece immediately for `price[j-1]`, and then add the best profit we can get from the remaining rod length `dp[i-j]`.

$$dp[i] = \max_{1 \le j \le i} (price[j-1] + dp[i-j])$$

### **C++ Implementation**

C++

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <climits>

using namespace std;

int rodCuttingBase(int n, vector<int>& prices) {
    // dp[i] stores max profit for rod of length i
    vector<int> dp(n + 1, 0);

    // Build the table bottom-up from length 1 to n
    for (int i = 1; i <= n; i++) {
        int max_val = INT_MIN;
        
        // Try every possible cut length j (1 to i)
        for (int j = 1; j <= i; j++) {
            // Profit = Price of piece j + Max profit of remaining length (i-j)
            max_val = max(max_val, prices[j - 1] + dp[i - j]);
        }
        dp[i] = max_val;
    }
    return dp[n];
}
```

### **Deep Dive: How It Works**

1. **Base Case:** `dp[0] = 0`. A rod of length 0 earns 0 profit.
    
2. **Iteration:** When we calculate `dp[5]`, the inner loop tries:
    
    - Cut length 1 + `dp[4]` (Best way to handle remaining 4)
        
    - Cut length 2 + `dp[3]` (Best way to handle remaining 3)
        
    - ...
        
    - Cut length 5 + `dp[0]` (Don't cut at all, sell as whole)
        
3. **Result:** `dp[n]` naturally accumulates the optimal strategy.
    

---

## 2. Variant: Rod Cutting with Cutting Cost

### **The Variant**

In the real world, cutting requires labor or machine time. Here, every time you make a physical cut, you incur a **fixed cost $C$**.

- **Key Distinction:** Selling the rod as a single raw piece (length $N$) costs **0**. Splitting it into two pieces (length $j$ and $N-j$) costs **$C$**.
    

### **Logic Shift**

We need to penalize the "splitting" action.

- **Choice A (No Cut):** Simply sell the rod of length $i$. Profit = `price[i-1]`.
    
- **Choice B (Cut):** Cut off a piece of size $j$. Profit = `price[j-1]` + `dp[i-j]` **$- C$**.
    

The recurrence becomes:

$$dp[i] = \max(price[i-1], \max_{1 \le j < i} \{ price[j-1] + dp[i-j] - C \})$$

### **C++ Implementation**

C++

```cpp
int rodCuttingWithCost(int n, vector<int>& prices, int cost) {
    vector<int> dp(n + 1, 0);

    for (int i = 1; i <= n; i++) {
        // Option 1: Don't cut at all. Sell the rod 'i' as is.
        // No cutting cost is applied here.
        int max_val = prices[i - 1];

        // Option 2: Make a cut at length j
        // Loop runs j < i because j=i implies no cut (handled above)
        for (int j = 1; j < i; j++) {
            // We subtract 'cost' because we performed a split operation
            max_val = max(max_val, prices[j - 1] + dp[i - j] - cost);
        }
        dp[i] = max_val;
    }
    return dp[n];
}
```

---

## 3. Variant: Rod Cutting with Max K Cuts

### **The Variant**

You are limited by your tools: you can make **at most K cuts**.

This breaks our original `dp[i]` state. Why? Because a rod of length 5 with _2 cuts remaining_ is very different from a rod of length 5 with _0 cuts remaining_.

### **Logic Shift**

We add a dimension to the DP state to track the constraint.

- **New State:** `dp[len][k]` = Max profit using a rod of length `len` having `k` cuts available.
    
- **Transition:**
    
    - If we sell the whole piece `len`, we use **0 cuts**.
        
    - If we cut off a piece of size `j`, we use **1 cut** and are left with `k-1` cuts for the remainder.
        

### **C++ Implementation**

C++

```cpp
int rodCuttingMaxKCuts(int n, vector<int>& prices, int k) {
    // dp[length][cuts_left]
    vector<vector<int>> dp(n + 1, vector<int>(k + 1, 0));

    // Iterate through all lengths
    for (int i = 1; i <= n; i++) {
        // Iterate through possible cuts remaining (0 to k)
        for (int cuts = 0; cuts <= k; cuts++) {
            
            // Base choice: Sell the rod 'i' as one piece (Uses 0 cuts)
            int max_val = prices[i - 1]; 

            // Only try splitting if we have cuts allowed (> 0)
            if (cuts > 0) {
                for (int j = 1; j < i; j++) {
                    // Cut off piece j. 
                    // Remainder is (i-j), and we have (cuts-1) left for it.
                    max_val = max(max_val, prices[j - 1] + dp[i - j][cuts - 1]);
                }
            }
            dp[i][cuts] = max_val;
        }
    }
    return dp[n][k];
}
```

---

## 4. Variant: Rod Cutting with Exact K Cuts

### **The Variant**

You must make **exactly K cuts**. If a rod cannot be split exactly $K$ times (e.g., trying to cut a length 2 rod 5 times), that configuration is invalid.

### **Logic Shift**

The tricky part here is **Initialization**.

- In standard DP, we initialize with 0 because profit is never negative.
    
- Here, we must initialize with **$-\infty$** (or a large negative number). This acts as a "flag" that a state is unreachable.
    
- **Base Case:** `dp[0][0] = 0`. Length 0 with 0 cuts is valid (and has 0 profit). All other `dp[0][k]` are invalid ($-\infty$).
    

### **C++ Implementation**

C++

```cpp
int rodCuttingExactKCuts(int n, vector<int>& prices, int k) {
    // 1. Initialize with -1e9 (Impossible state)
    // We use K+2 pieces logic: K cuts divides rod into K+1 pieces.
    // It's often easier to think: "Select K+1 pieces that sum to N".
    int pieces_needed = k + 1;
    
    // dp[pieces_count][current_length]
    vector<vector<int>> dp(pieces_needed + 1, vector<int>(n + 1, -1e9));

    // Base Case: 0 pieces have 0 length and 0 profit
    dp[0][0] = 0;

    // Iterate 1 to pieces_needed
    for (int p = 1; p <= pieces_needed; p++) {
        // Iterate through total length expected
        for (int len = 1; len <= n; len++) {
             // Try every possible size for the CURRENT piece (the p-th piece)
             for (int cut = 1; cut <= len; cut++) {
                 // Check if the previous state (p-1 pieces with length len-cut) was valid
                 if (dp[p-1][len-cut] != -1e9) {
                     dp[p][len] = max(dp[p][len], dp[p-1][len-cut] + prices[cut-1]);
                 }
             }
        }
    }

    // If result is still negative, it means exact K cuts wasn't possible
    return dp[pieces_needed][n] < 0 ? 0 : dp[pieces_needed][n];
}
```

---

## 5. Variant: Rod Cutting with Min Length Constraint

### **The Variant**

The machine has a mechanical limit: it cannot produce or cut any piece smaller than $L_{min}$.

- Every final piece sold must be $\ge L_{min}$.
    
- Every intermediate remainder must be $\ge L_{min}$ (or exactly 0).
    

### **Logic Shift**

We strictly control the **loops**.

1. If the current rod `i` is less than `minLen`, profit is 0 (can't sell).
    
2. The cut size `j` starts at `minLen`.
    
3. The remainder `i-j` must also be valid.
    

### **C++ Implementation**

C++

```cpp
int rodCuttingMinLen(int n, vector<int>& prices, int minLen) {
    vector<int> dp(n + 1, 0);

    for (int i = 1; i <= n; i++) {
        // If current rod is too short to be sold even as one piece
        if (i < minLen) {
            dp[i] = 0; 
            continue;
        }

        // Option 1: Sell whole (Valid since i >= minLen)
        int max_val = prices[i - 1];

        // Option 2: Cut piece of size j
        // Condition: The cut 'j' must be >= minLen
        // Condition: The remainder 'i-j' is handled by DP, but loop ensures we don't pick invalid j
        for (int j = minLen; j <= i - minLen; j++) {
            max_val = max(max_val, prices[j - 1] + dp[i - j]);
        }
        dp[i] = max_val;
    }
    return dp[n];
}
```

---

## 6. Variant: Maximize Product (Integer Break)

### **The Variant**

Instead of a price array, the "value" of a piece of length $L$ is just $L$.

You must break integer $N$ into $k \ge 2$ parts to **maximize the product** of the parts.

- _Example:_ $N=10$. Best break is $3+3+4$. Product $3 \times 3 \times 4 = 36$.
    

### **Logic Shift**

This is subtly different because the "price" is dynamic.

When splitting length $i$ into $j$ and $(i-j)$:

1. **Stop Recursion:** Use $(i-j)$ as a raw number. Product = $j \times (i-j)$.
    
2. **Continue Recursion:** Break $(i-j)$ further. Product = $j \times dp[i-j]$.
    
    We take the max of these two.
    

### **C++ Implementation**

C++

```cpp
int integerBreak(int n) {
    vector<int> dp(n + 1, 0);

    // Build up from length 2 to n
    for (int i = 2; i <= n; i++) {
        for (int j = 1; j < i; j++) {
            // Compare three things:
            // 1. Current best stored in dp[i]
            // 2. j * (i-j) -> Simply cut once and multiply parts. (e.g., 2 * 3)
            // 3. j * dp[i-j] -> Cut 'j', then recursively find best product for remainder.
            
            dp[i] = max(dp[i], max(j * (i - j), j * dp[i - j]));
        }
    }
    return dp[n];
}
```

---

## 7. Variant: Even/Odd Length Constraint

### **The Variant**

Market regulation: You can only sell pieces that have an **even** length.

### **Logic Shift**

This is a constraint on validity.

1. If the rod length $i$ is odd, you **cannot** sell it as a whole piece (Base option is invalid/0).
    
2. When making cuts, you are only allowed to cut off pieces $j$ where $j$ is even.
    

### **C++ Implementation**

C++

```cpp
int rodCuttingEvenOnly(int n, vector<int>& prices) {
    vector<int> dp(n + 1, 0);

    for (int i = 1; i <= n; i++) {
        // Base Option: Can only sell current whole piece if length i is even
        int max_val = (i % 2 == 0) ? prices[i - 1] : 0;

        // Loop for cuts
        for (int j = 1; j < i; j++) {
            // Constraint: The piece we cut off (j) MUST be even
            if (j % 2 == 0) {
                // We don't need to check if (i-j) is even; the DP recursion handles it.
                // If (i-j) ends up being impossible to split into evens, dp[i-j] will be low/0.
                max_val = max(max_val, prices[j - 1] + dp[i - j]);
            }
        }
        dp[i] = max_val;
    }
    return dp[n];
}
```