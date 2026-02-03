Here are the comprehensive notes for the **0/1 Knapsack Problem** and its essential exam variants. This is arguably the most fundamental problem in Dynamic Programming.

---

# 🎒 Topic: 0/1 Knapsack Problem & Variants

## 1. The Base Problem: Standard 0/1 Knapsack

### **Problem Statement**

You are given `N` items. Each item has a **Weight** (`wt[i]`) and a **Value** (`val[i]`).

You have a knapsack that can hold a maximum weight capacity `W`.

- **Goal:** Maximize the total value of items in the knapsack.
    
- **Constraint:** You can either **pick** an item or **leave** it (0 or 1). You cannot break items apart (No fractional knapsack).
    

### **Intuition**

For every item, we have a binary choice:

1. **Exclude:** We don't take the item. The value is the same as the solution for `N-1` items with capacity `W`.
    
2. **Include:** We take the item (only if `wt[i] <= W`). The value is `val[i]` + the solution for `N-1` items with capacity `W - wt[i]`.
    
    We take the **maximum** of these two choices.
    

### **State Definition***

> **`dp[i][w]` = Max value using a subset of the first `i` items with capacity `w`.**

$$dp[i][w] = \max(dp[i-1][w], \text{val}[i-1] + dp[i-1][w - \text{wt}[i-1]])$$

### **C++ Implementation (2D Array)**

C++

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

int knapsack01(int W, vector<int>& wt, vector<int>& val, int n) {
    // dp[i][w]
    vector<vector<int>> dp(n + 1, vector<int>(W + 1, 0));

    for (int i = 1; i <= n; i++) {
        for (int w = 0; w <= W; w++) {
            // Option 1: Don't include item i (Value is same as using i-1 items)
            // Note: Current item is index i-1 in 0-indexed vectors
            int notTaken = dp[i - 1][w];
            
            // Option 2: Include item i (if weight allows)
            int taken = 0;
            if (wt[i - 1] <= w) {
                taken = val[i - 1] + dp[i - 1][w - wt[i - 1]];
            }
            
            dp[i][w] = max(notTaken, taken);
        }
    }
    
    return dp[n][W];
}
```

---

## 2. Variant: Space Optimization (1D Array)

### **The Variant**

The 2D solution uses $O(N \times W)$ space. If $W$ is large, this crashes memory. Can we reduce it?

### **Logic Shift**

Notice that to calculate row `i`, we _only_ need row `i-1`. We don't need `i-2`, `i-3`, etc.

However, if we use a single array `dp[w]`, and we iterate forward, we run into the **Unbounded Knapsack** problem (reusing the same item multiple times).

- **Fix:** Iterate **Backwards** ($W \to 0$). This ensures that when calculating `dp[w]`, the value `dp[w - weight]` comes from the _previous_ iteration (i.e., it doesn't include the current item yet).
    

### **C++ Implementation**

C++

```cpp
int knapsackOptimized(int W, vector<int>& wt, vector<int>& val, int n) {
    // dp[w] stores max value for capacity w
    vector<int> dp(W + 1, 0);

    for (int i = 0; i < n; i++) {
        // Iterate BACKWARDS to prevent using the same item twice
        for (int w = W; w >= wt[i]; w--) {
            dp[w] = max(dp[w], val[i] + dp[w - wt[i]]);
        }
    }
    return dp[W];
}
```

---

## 3. Variant: Subset Sum Problem

### **The Variant**

Given an array `nums` and a target `sum`, determine if **any** subset of the array sums up to the target.

- This is Knapsack where `weight = value = nums[i]`. We just care about "filling the capacity".
    

### **Logic Shift**

- State: `dp[w]` is **boolean** (True/False).
    
- Logic: `dp[w] = dp[w] OR dp[w - nums[i]]`.
    
- Order: Iterate backwards (finite supply of numbers).
    

### **C++ Implementation**

C++

```cpp
bool subsetSum(vector<int>& nums, int target) {
    vector<bool> dp(target + 1, false);
    dp[0] = true; // Base case: Sum 0 is always possible (empty set)

    for (int num : nums) {
        for (int s = target; s >= num; s--) {
            // If we could make (s - num), we can now make s
            if (dp[s - num]) {
                dp[s] = true;
            }
        }
    }
    return dp[target];
}
```

---

## 4. Variant: Partition Equal Subset Sum

### **The Variant**

Can you partition an array into two subsets with **equal sum**?

- _Example:_ `[1, 5, 11, 5]` $\to$ `{1, 5, 5}` and `{11}`. Both sum to 11. True.
    

### **Logic Shift**

This is exactly the **Subset Sum** problem.

1. Calculate total sum of array.
    
2. If sum is **odd**, return `false` (cannot split odd number into two equal integers).
    
3. If sum is **even**, we need to find a subset with sum equal to `TotalSum / 2`.
    
    - Run Subset Sum algorithm with `target = TotalSum / 2`.
        

---

## 5. Variant: Count Subsets with Sum S

### **The Variant**

Instead of checking _if_ a subset exists, count **how many** subsets sum to `S`.

### **Logic Shift**

- Change `dp` from `bool` to `int`.
    
- Change `OR` logic to `SUM` logic: `dp[s] += dp[s - num]`.
    
- Still iterate backwards.
    

### **C++ Snippet**

C++

```cpp
int countSubsets(vector<int>& nums, int target) {
    vector<int> dp(target + 1, 0);
    dp[0] = 1;

    for (int num : nums) {
        for (int s = target; s >= num; s--) {
            dp[s] += dp[s - num];
        }
    }
    return dp[target];
}
```

---

## 6. Variant: Minimum Subset Sum Difference

### **The Variant**

Partition the array into two subsets $S1, S2$ such that $|Sum(S1) - Sum(S2)|$ is **minimized**.

### **Logic Shift**

1. Calculate `TotalSum`.
    
2. We want to find a subset sum $S1$ that is as close to `TotalSum / 2` as possible.
    
3. Run the **Subset Sum** boolean DP for range `TotalSum`.
    
4. Check the `dp` array. Find the largest index `i` (where `i <= TotalSum/2`) such that `dp[i]` is true.
    
5. Answer = `TotalSum - 2*i`. (Since $S2 = Total - S1$, diff = $|Total - 2*S1|$).
    

### **C++ Snippet**

C++

```
int minSubsetDiff(vector<int>& nums) {
    int total = 0;
    for(int x : nums) total += x;
    
    // Run subset sum logic
    vector<bool> dp(total + 1, false);
    dp[0] = true;
    
    for(int num : nums) {
        for(int s = total; s >= num; s--) {
            if(dp[s-num]) dp[s] = true;
        }
    }
    
    // Find closest valid subset sum to total/2
    int s1 = 0;
    for(int i = total/2; i >= 0; i--) {
        if(dp[i]) {
            s1 = i;
            break;
        }
    }
    
    return total - 2 * s1;
}
```

---

## 7. Variant: Target Sum (Assign Signs)

### **The Variant**

You are given an array `nums`. You must assign a `+` or `-` sign to each element so the total equals `Target`.

- _Example:_ `[1, 1, 1, 1, 1]`, `Target = 3`.
    
- Solution: `+1+1+1+1-1` = 3.
    

### **Logic Shift (Mathematical Reduction)**

Let $P$ be the set of positive numbers, $N$ be the set of negative numbers.

$Sum(P) - Sum(N) = Target$

$Sum(P) + Sum(N) = TotalSum$

Adding the equations:

$2 \times Sum(P) = Target + TotalSum$

$Sum(P) = (Target + TotalSum) / 2$

Problem reduces to: **Count Subsets with Sum = (Target + TotalSum) / 2**.

- Corner Case: If `(Target + TotalSum)` is odd, return 0.
    

---

## 8. Variant: Dual Knapsack (Min Weight for Fixed Value)

### **The Variant**

Standard Knapsack maximizes value for fixed capacity.

What if capacity $W$ is enormous ($10^9$) but max value $V$ is small ($10^4$)?

The standard $O(N \cdot W)$ array will overflow memory.

### **Logic Shift**

We flip the DP state.

- **Old State:** `dp[w] = max_value`
    
- **New State:** `dp[v] = min_weight_to_get_value_v`
    
- We iterate through possible **Values** (from MaxPossibleValue down to 0).
    
- Initialize `dp` with `Infinity`, `dp[0] = 0`.
    
- Transition: `dp[v] = min(dp[v], wt[i] + dp[v - val[i]])`.
    
- Answer: The largest `v` such that `dp[v] <= W`.
    

### **C++ Implementation**

C++

```
int knapsackHighCapacity(int W, vector<int>& wt, vector<int>& val, int n) {
    int maxValSum = 0;
    for(int v : val) maxValSum += v;
    
    // dp[v] = min weight to achieve value v
    vector<int> dp(maxValSum + 1, 1e9 + 7);
    dp[0] = 0;

    for (int i = 0; i < n; i++) {
        for (int v = maxValSum; v >= val[i]; v--) {
            dp[v] = min(dp[v], wt[i] + dp[v - val[i]]);
        }
    }

    // Find max value where weight is within capacity
    for (int v = maxValSum; v >= 0; v--) {
        if (dp[v] <= W) return v;
    }
    return 0;
}
```