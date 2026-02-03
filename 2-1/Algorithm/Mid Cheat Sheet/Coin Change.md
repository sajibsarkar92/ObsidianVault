Here are the expanded and detailed notes for the **Coin Change Problem** and its **Finite Supply (0/1)** variant.

---

# 💰 Topic: Coin Change & Variants

## 1. The Base Problem: Minimum Coins (Unbounded)

### **Problem Statement**

You are given an integer array `coins` representing coins of different denominations and an integer `amount` representing a total amount of money.

- **Goal:** Return the **fewest number of coins** that you need to make up that amount.
    
- **Constraint:** You have an **infinite number** of each kind of coin.
    
- **Failure Case:** If that amount of money cannot be made up by any combination of the coins, return `-1`.
    
- _Example:_ `coins = [1, 2, 5]`, `amount = 11`.
    
    - Possible ways: `1+1+...+1` (11 coins), `2+2+2+2+2+1` (6 coins), `5+5+1` (3 coins).
        
    - Optimal Answer: 3.
        

### **Intuition**

This is a minimization problem. We want to reach the target `amount` (let's call it $V$) using the smallest number of steps (coins).

Think of this "bottom-up".

- To solve for `amount = 11`, we can reach it from:
    
    - `amount = 10` (by adding a $1 coin). Cost: `dp[10] + 1`.
        
    - `amount = 9` (by adding a $2 coin). Cost: `dp[9] + 1`.
        
    - `amount = 6` (by adding a $5 coin). Cost: `dp[6] + 1`.
        
- We take the **minimum** of these three previous states.
    

### **Logic & Recurrence**

Let `dp[i]` be the minimum coins needed to make amount `i`.

- **Initialization:** `dp[0] = 0` (0 coins for 0 amount). All other `dp` values set to a large number (`Infinity` or `amount + 1`) to represent "unreachable".
    
- **Recurrence:**
    
    $$dp[i] = \min_{c \in coins} (dp[i - c]) + 1$$
    

### **C++ Implementation**

C++

```
#include <iostream>
#include <vector>
#include <algorithm>
#include <climits>

using namespace std;

int minCoins(vector<int>& coins, int amount) {
    // 1. Initialize DP array
    // We use 'amount + 1' as our "Infinity" because it's impossible 
    // to use more coins than the amount itself (assuming min coin is 1).
    vector<int> dp(amount + 1, amount + 1);

    // 2. Base Case
    dp[0] = 0;

    // 3. Iteration
    // Order of loops (Coins vs Amount) DOES NOT MATTER for MIN problems.
    // Usually, iterating amount 1 to N is intuitive.
    for (int i = 1; i <= amount; i++) {
        for (int coin : coins) {
            // Can we use this coin? (Is the current amount >= coin value?)
            if (i >= coin) {
                // dp[i] = min(current_best, best_for_remainder + 1 coin)
                dp[i] = min(dp[i], dp[i - coin] + 1);
            }
        }
    }

    // 4. Result Check
    // If dp[amount] is still "Infinity", it means we never found a path.
    return (dp[amount] > amount) ? -1 : dp[amount];
}
```

---

## 2. Variant: Total Ways (Combinations vs Permutations)

This is the most confusing part for students. "Count the number of ways" can mean two things.

### **Case A: Combinations (Order does NOT matter)**

- **Scenario:** You have coins `{1, 2}` and target `3`.
    
    - `1 + 1 + 1`
        
    - `1 + 2`
        
    - `2 + 1` (This is the SAME set of coins as `1 + 2`. Don't count it twice!)
        
- **Target Answer:** 2 ways (`{1,1,1}` and `{1,2}`).
    

### **Logic Shift: Loop Order**

To prevent duplicates like `1+2` and `2+1`, we must enforce an implicit order. We say: "Use all the 1s first. Then use all the 2s."

- **Outer Loop:** Coins
    
- **Inner Loop:** Amount
    
    This ensures that once we move to the next coin, we never go back to the previous coin.
    

### **C++ Implementation (Combinations)**

C++

```
int changeCombinations(int amount, vector<int>& coins) {
    vector<int> dp(amount + 1, 0);
    
    // Base Case: There is 1 way to make amount 0 (by choosing nothing)
    dp[0] = 1;

    // CRITICAL: Iterate Coins FIRST
    // This forces us to process coins in specific order (e.g., all 1s, then all 2s...)
    for (int coin : coins) {
        for (int i = coin; i <= amount; i++) {
            // We can make amount 'i' by adding 'coin' to existing ways of making 'i-coin'
            dp[i] += dp[i - coin];
        }
    }
    
    return dp[amount];
}
```

### **Case B: Permutations (Order DOES matter)**

- **Scenario:** Climbing Stairs. You can take 1 step or 2 steps. Target 3.
    
    - `1 + 1 + 1`
        
    - `1 + 2`
        
    - `2 + 1` (This is a valid, distinct sequence of steps).
        
- **Target Answer:** 3 ways.
    

### **Logic Shift: Loop Order**

To allow `2+1` and `1+2` to both count, at every step `i`, we must be allowed to try _any_ coin as the last step.

- **Outer Loop:** Amount
    
- **Inner Loop:** Coins
    

### **C++ Implementation (Permutations)**

C++

```
int changePermutations(int amount, vector<int>& coins) {
    vector<unsigned int> dp(amount + 1, 0); // Unsigned to prevent overflow
    dp[0] = 1;

    // CRITICAL: Iterate Amount FIRST
    // For every amount 'i', we check ALL coins.
    // This means for amount 3, we try ending with 1 (from dp[2]) AND ending with 2 (from dp[1]).
    for (int i = 1; i <= amount; i++) {
        for (int coin : coins) {
            if (i >= coin) {
                dp[i] += dp[i - coin];
            }
        }
    }
    
    return dp[amount];
}
```

---

## 3. Variant: Finite Supply (Bounded / 0-1 Knapsack)

### **The Variant**

In the standard Coin Change problem, you can use `coin[0]` five million times if you want.

Here, you are given a specific set of coins, e.g., `coins = {1, 2, 5}`. You effectively have **one of each**.

- **Goal:** Can you make amount 3?
    
    - Unbounded answer: Yes (`1+1+1` or `1+2`).
        
    - Finite Supply answer: Yes (`1+2`). You cannot do `1+1+1` because you only have one `1`.
        

### **Logic Shift: The "Reverse" Loop**

This is the single most important concept in 1D Dynamic Programming optimization.

- If we iterate **forward** (`i` from `coin` to `amount`), `dp[i]` uses `dp[i-coin]`. But `dp[i-coin]` might have _already_ used the current coin in this same loop! This leads to multiple usages (Unbounded).
    
- If we iterate **backward** (`i` from `amount` down to `coin`), `dp[i]` uses `dp[i-coin]`. Since we are moving downwards, `dp[i-coin]` represents the state from the **previous** coin's iteration. It has _not_ seen the current coin yet. This ensures we use each coin at most once.
    

### **C++ Implementation**

C++

```
// Returns true if we can make exactly 'amount' using each coin at most once
bool canMakeAmountFinite(vector<int>& coins, int amount) {
    vector<bool> dp(amount + 1, false);
    
    // Base Case: We can always make 0 amount
    dp[0] = true;

    // Iterate through each specific coin we have
    for (int coin : coins) {
        
        // CRITICAL: Iterate BACKWARDS from amount down to coin
        // Example: Coin = 5.
        // If we go forward: dp[5] becomes true (using coin). Later dp[10] sees dp[5] is true,
        // so dp[10] becomes true. We just used 5 twice!
        // If we go backward: We calculate dp[10] first. It looks at dp[5] (which is false from previous turns).
        // Then we calculate dp[5] and set it to true.
        for (int i = amount; i >= coin; i--) {
            if (dp[i - coin] == true) {
                dp[i] = true;
            }
        }
    }
    
    return dp[amount];
}
```

---

## 4. Variant: Count Subsets with Given Sum (Finite Supply)

### **The Variant**

Given an array `arr = {1, 2, 3, 3}`, find how many **subsets** sum up to `target = 6`.

- This is the "Ways" version of the Finite Supply problem.
    
- Note: The two `3`s are distinct items.
    

### **Logic Shift**

1. Use the **Combinations** loop structure (Coins first).
    
2. Use the **Finite Supply** loop direction (Backwards).
    
3. Use `+` instead of `OR`.
    

### **C++ Implementation**

C++

```
int countSubsets(vector<int>& nums, int target) {
    vector<int> dp(target + 1, 0);
    dp[0] = 1; // 1 way to make sum 0 (empty subset)

    for (int num : nums) {
        // Backward loop ensures each number in 'nums' is used only once per subset
        for (int i = target; i >= num; i--) {
            dp[i] += dp[i - num];
        }
    }
    
    return dp[target];
}
```

---

## 5. Variant: Minimum Coins with Finite Supply

### **The Variant**

Find the minimum number of coins to make `amount`, but you only have the specific list `coins`.

- _Example:_ `coins = {1, 2, 5}`, `amount = 11`.
    
- Result: `-1` (Impossible). You only have sum $1+2+5 = 8$.
    

### **Logic Shift**

We initialize with `Infinity` and minimize, but we strictly use the **Backward Loop**.

### **C++ Implementation**

C++

```
int minCoinsFinite(vector<int>& coins, int amount) {
    // Initialize with a value larger than max possible coins (which is coins.size())
    int INF = amount + 999;
    vector<int> dp(amount + 1, INF);
    
    dp[0] = 0;

    for (int coin : coins) {
        // Backward loop for finite supply
        for (int i = amount; i >= coin; i--) {
            if (dp[i - coin] != INF) {
                dp[i] = min(dp[i], dp[i - coin] + 1);
            }
        }
    }

    return dp[amount] >= INF ? -1 : dp[amount];
}
```