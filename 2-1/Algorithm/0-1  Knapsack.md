# 0/1 Knapsack Problem - Complete Guide

## The Problem

You're a thief who broke into a store with a knapsack that can carry a maximum weight of `W` kilograms. The store has `n` items, each with:

- A **weight** (how heavy it is)
- A **value** (how much money you can get by selling it)

Your goal: Fill your knapsack to maximize the total value of items you steal, without exceeding the weight capacity.

The **"0/1"** means: for each item, you either take it (1) or leave it (0). You **cannot** take fractional amounts (like half an item) or take the same item multiple times.

---

## A Concrete Scenario

Imagine you're at a jewelry store with a knapsack that can carry **maximum 7 kg**. You see these items:

|Item|Weight (kg)|Value ($)|
|---|---|---|
|Diamond Ring|1|1000|
|Gold Necklace|3|2000|
|Silver Watch|4|3000|
|Emerald Bracelet|5|4000|

**Question:** Which items should you take to maximize value while staying within 7 kg?

Let's think through some options:

- Take everything? (1+3+4+5 = 13 kg) ❌ Too heavy!
- Take Diamond + Gold + Silver? (1+3+4 = 8 kg) ❌ Still too heavy!
- Take Diamond + Gold? (1+3 = 4 kg, value = $3000) ✅ Fits, but is it optimal?
- Take Gold + Silver? (3+4 = 7 kg, value = $5000) ✅ Fits and better!

The answer is **Gold Necklace + Silver Watch** for $5000 total value.

But how do we systematically find this answer for any input?

---

## The Recursive Intuition

### The Core Decision

At each item, you face a simple choice:

1. **Take it** (if it fits in the remaining capacity)
2. **Leave it** (skip to the next item)

Let's define our problem recursively:

**`knapsack(i, w)`** = Maximum value we can get considering items from index `i` to the end, with `w` weight capacity remaining.

### The Recursive Cases

When we're at item `i` with capacity `w`, we have two options:

**Option 1: Skip item i**

- We don't take this item
- Move to the next item with the same capacity
- Value = `knapsack(i+1, w)`

**Option 2: Take item i** (only if `weight[i] <= w`)

- We take this item's value
- Reduce our remaining capacity by this item's weight
- Move to the next item
- Value = `value[i] + knapsack(i+1, w - weight[i])`

We pick whichever gives us **more value**!

### Why This Works - Deep Intuition

Think of it like this: you're standing in front of item `i`, and you're asking yourself "What's the maximum value I can possibly get from this point forward?"

To answer this, you consider both possibilities:

1. A future where you didn't take item `i`
2. A future where you did take item `i` (and now have less capacity)

You peek into both futures, see which one gives you more total value, and make the optimal choice!

The beauty is: **you don't need to know the entire future right now**. You just need to know that your future self will also make optimal decisions. That's the essence of recursion with memoization - trust that subproblems will be solved optimally.

### Base Cases

**When do we stop recursing?**

1. **If `i >= n`** (no more items to consider)
    
    - Return 0 (can't add any more value)
2. **If `w == 0`** (no capacity left)
    
    - Return 0 (can't carry anything)

### Example Walkthrough

Let's trace through our jewelry store example with just 2 items for simplicity:

- Item 0: Diamond Ring (weight=1, value=1000)
- Item 1: Gold Necklace (weight=3, value=2000)
- Capacity = 4 kg

```
knapsack(0, 4):  // At Diamond, capacity 4
  Option 1 (skip): knapsack(1, 4) = ?
  Option 2 (take): 1000 + knapsack(1, 3) = ?
  
  Let's explore Option 1:
    knapsack(1, 4):  // At Gold, capacity 4
      Option 1 (skip): knapsack(2, 4) = 0  (no more items)
      Option 2 (take): 2000 + knapsack(2, 1) = 2000 + 0 = 2000
      Return max(0, 2000) = 2000
  
  Let's explore Option 2:
    knapsack(1, 3):  // At Gold, capacity 3
      Option 1 (skip): knapsack(2, 3) = 0
      Option 2 (take): 2000 + knapsack(2, 0) = 2000 + 0 = 2000
      Return max(0, 2000) = 2000
    
    So: 1000 + 2000 = 3000

knapsack(0, 4) = max(2000, 3000) = 3000
```

Answer: Take both items for $3000!

---

## Why Do We Need Dynamic Programming?

The recursive solution has **overlapping subproblems**.

For example, `knapsack(2, 5)` might be called from:

- `knapsack(1, 5)` when we skip item 1
- `knapsack(1, 8)` when we take item 1 (if it weighs 3)

Without memoization, we'd recalculate `knapsack(2, 5)` multiple times! The time complexity would be **O(2^n)** - exponential!

With memoization, we calculate each unique `(i, w)` pair only once and store the result. This reduces time complexity to **O(n × W)**.

---

## Step-by-Step Implementation

### Step 1: Include Headers

**Why?** We need vector for storing items and the DP table, and algorithm for the max function. These are the basic building blocks for our solution.

```cpp
#include <vector>
#include <algorithm>
using namespace std;
```

---

### Step 2: Define the Recursive Function Signature

**Why?** We need to pass the items (weights and values), the current index we're considering, the remaining capacity, and a DP table to store already-computed results. The DP table prevents us from solving the same subproblem multiple times - it's our "memory" of past decisions.

```cpp
int knapsackHelper(vector<int>& weights, vector<int>& values, 
                   int i, int capacity, vector<vector<int>>& dp) {
```

---

### Step 3: Check Base Cases

**Why?** Before doing any recursive work, we need to check if we've reached a stopping condition. If there are no more items to consider (i >= n) or no capacity left (capacity <= 0), we can't add any more value, so we return 0. These base cases prevent infinite recursion and give our recursion something concrete to return to.

```cpp
    int n = weights.size();
    
    // Base case: no more items or no capacity
    if (i >= n || capacity <= 0) {
        return 0;
    }
```

---

### Step 4: Check Memoization Table

**Why?** Before doing expensive recursive calculations, we check if we've already solved this exact subproblem before. The DP table stores results for each (item index, capacity) pair. If we've already computed knapsack(i, capacity), we just return that stored answer instead of recalculating. This is what makes our algorithm efficient - we never solve the same subproblem twice! We use -1 to indicate "not yet calculated" because 0 is a valid answer (zero value).

```cpp
    // Check if already computed
    if (dp[i][capacity] != -1) {
        return dp[i][capacity];
    }
```

---

### Step 5: Option 1 - Skip Current Item

**Why?** The first choice we always have is to simply not take the current item. When we skip item i, nothing changes except we move to consider the next item (i+1). Our capacity stays the same because we didn't add any weight. This represents the "what if I leave this behind?" branch of our decision tree. Even if an item looks valuable, sometimes skipping it allows us to take a better combination of other items later.

```cpp
    // Option 1: Don't take item i
    int skipItem = knapsackHelper(weights, values, i + 1, capacity, dp);
```

---

### Step 6: Option 2 - Take Current Item (If It Fits)

**Why?** The second option is to take the current item, but we can only do this if the item's weight doesn't exceed our remaining capacity. When we take item i, three things happen: (1) we gain its value, (2) we reduce our capacity by its weight, and (3) we move to the next item. This represents the "what if I take this?" branch. The capacity check is crucial - it prevents us from overfilling our knapsack.

```cpp
    int takeItem = 0;
    if (weights[i] <= capacity) {
        // Take item i: add its value and reduce capacity
        takeItem = values[i] + knapsackHelper(weights, values, i + 1, 
                                               capacity - weights[i], dp);
    }
```

---

### Step 7: Choose the Better Option

**Why?** We've explored both possible futures - one where we took the item and one where we didn't. Now we need to decide which decision leads to more total value. The max function picks the better of the two options. This is the "optimal" part of dynamic programming - at each decision point, we make the locally optimal choice (take whichever path gives more value), and because all our subproblems also make optimal choices, our final answer is globally optimal.

```cpp
    // Take maximum of both options
    int result = max(skipItem, takeItem);
```

---

### Step 8: Store Result in DP Table

**Why?** Before returning our answer, we store it in the DP table at position [i][capacity]. This is memoization in action! The next time we encounter this exact same subproblem (same item index, same remaining capacity), we'll find our answer already waiting in step 4, and we won't need to do all this recursive work again. This single step is what transforms our algorithm from exponential O(2^n) to polynomial O(n×W) time complexity.

```cpp
    // Memoize and return
    dp[i][capacity] = result;
    return result;
}
```

---

### Step 9: Create the Main Function

**Why?** This is the public interface to our solution. Users don't need to know about the recursive helper or the DP table - they just want to pass in weights, values, and capacity, and get back the maximum value. This function handles all the setup work: creating and initializing the DP table with -1 (meaning "not computed yet"), and starting the recursion from item 0 with full capacity. It's a clean wrapper that hides the implementation complexity.

```cpp
int knapsack(vector<int>& weights, vector<int>& values, int capacity) {
    int n = weights.size();
    
    // Create DP table: dp[i][w] = max value from items i..n-1 with capacity w
    // Initialize with -1 (not computed)
    vector<vector<int>> dp(n, vector<int>(capacity + 1, -1));
    
    return knapsackHelper(weights, values, 0, capacity, dp);
}
```

---

## Complete Recursive DP Implementation

```cpp
#include <vector>
#include <algorithm>
using namespace std;

// Helper function for recursion with memoization
int knapsackHelper(vector<int>& weights, vector<int>& values, 
                   int i, int capacity, vector<vector<int>>& dp) {
    int n = weights.size();
    
    // Base case
    if (i >= n || capacity <= 0) {
        return 0;
    }
    
    // Check memo table
    if (dp[i][capacity] != -1) {
        return dp[i][capacity];
    }
    
    // Option 1: Skip item i
    int skipItem = knapsackHelper(weights, values, i + 1, capacity, dp);
    
    // Option 2: Take item i (if it fits)
    int takeItem = 0;
    if (weights[i] <= capacity) {
        takeItem = values[i] + knapsackHelper(weights, values, i + 1, 
                                               capacity - weights[i], dp);
    }
    
    // Choose best option and memoize
    dp[i][capacity] = max(skipItem, takeItem);
    return dp[i][capacity];
}

// Main function
int knapsack(vector<int>& weights, vector<int>& values, int capacity) {
    int n = weights.size();
    vector<vector<int>> dp(n, vector<int>(capacity + 1, -1));
    return knapsackHelper(weights, values, 0, capacity, dp);
}
```

---

## Bottom-Up DP Approach (Iterative)

The recursive approach is top-down (starts from the problem, breaks down to base cases). We can also solve it bottom-up (starts from base cases, builds up to the answer).

### The DP Table Meaning

`dp[i][w]` = maximum value achievable using items 0 to i-1 with capacity w

### Step-by-Step Bottom-Up Implementation

#### Step 1: Create and Initialize DP Table

**Why?** In bottom-up DP, we build our solution from the ground up, starting with the simplest cases. We create a 2D table where rows represent items (0 to n) and columns represent capacities (0 to W). We make it (n+1)×(W+1) sized so that dp[0][w] can represent "using 0 items" (base case). When we use 0 items, we get 0 value regardless of capacity - that's why the first row is initialized to 0. Similarly, with 0 capacity, we get 0 value regardless of items - that's why the first column is 0. These zeros are our "foundation" that we build upon.

```cpp
int knapsackBottomUp(vector<int>& weights, vector<int>& values, int W) {
    int n = weights.size();
    
    // dp[i][w] = max value using first i items with capacity w
    vector<vector<int>> dp(n + 1, vector<int>(W + 1, 0));
    
    // Base case: dp[0][w] = 0 (no items = no value)
    // Already initialized to 0
```

---

#### Step 2: Fill the DP Table Row by Row

**Why?** We process items one by one (outer loop) and for each item, we consider all possible capacities (inner loop). This systematic filling ensures that when we're calculating dp[i][w], all the subproblems we depend on (dp[i-1][...]) have already been solved. It's like building a skyscraper floor by floor - each floor depends on the one below it being complete. By the time we reach dp[n][W], we've considered all items with full capacity, which is our final answer.

```cpp
    for (int i = 1; i <= n; i++) {
        for (int w = 0; w <= W; w++) {
```

---

#### Step 3: Skip Current Item (Always an Option)

**Why?** For item i (which is actually weights[i-1] in our 0-indexed array - note the i-1!), we can always choose not to take it. If we don't take item i, the best we can do is the same as the best we could do with the previous i-1 items and the same capacity w. This is dp[i-1][w]. Think of it as saying: "If I don't add this new item to the mix, my maximum value is whatever it was before this item was available."

```cpp
            // Option 1: Don't take item i (0-indexed: items[i-1])
            int skipItem = dp[i - 1][w];
```

---

#### Step 4: Take Current Item (If It Fits)

**Why?** If the current item's weight is small enough to fit (weights[i-1] <= w), we have the option to take it. When we take it, we gain its value (values[i-1]), but we also use up some capacity (weights[i-1]). So we need to look up: what's the best value we could get with the remaining capacity using the previous items? That's dp[i-1][w - weights[i-1]]. Adding the current item's value to that gives us the total value of the "take item" option. If the item doesn't fit (weight > w), we can't take it, so we just use skipItem.

```cpp
            int takeItem = 0;
            if (weights[i - 1] <= w) {
                // Take item i: add its value + best with remaining capacity
                takeItem = values[i - 1] + dp[i - 1][w - weights[i - 1]];
            }
```

---

#### Step 5: Store the Maximum

**Why?** Just like in the recursive version, we choose whichever option gives us more value. The key insight is that dp[i][w] depends only on values from the previous row (dp[i-1][...]). This means we're building our solution incrementally - each cell represents the optimal solution for a subproblem, and we use those optimal subproblems to solve bigger problems. This is the principle of optimality in dynamic programming.

```cpp
            // Store the better choice
            dp[i][w] = max(skipItem, takeItem);
        }
    }
```

---

#### Step 6: Return the Final Answer

**Why?** After filling the entire table, dp[n][W] contains the answer to our original problem: the maximum value we can achieve using all n items with capacity W. We've systematically built up from "using 0 items" to "using all items", and from "capacity 0" to "capacity W". The bottom-right cell of our table holds the final answer.

```cpp
    // Answer is using all n items with full capacity W
    return dp[n][W];
}
```

---

## Complete Bottom-Up Implementation

```cpp
#include <vector>
#include <algorithm>
using namespace std;

int knapsackBottomUp(vector<int>& weights, vector<int>& values, int W) {
    int n = weights.size();
    
    // dp[i][w] = max value using first i items with capacity w
    vector<vector<int>> dp(n + 1, vector<int>(W + 1, 0));
    
    // Fill the table
    for (int i = 1; i <= n; i++) {
        for (int w = 0; w <= W; w++) {
            // Option 1: Skip item i-1
            int skipItem = dp[i - 1][w];
            
            // Option 2: Take item i-1 (if it fits)
            int takeItem = 0;
            if (weights[i - 1] <= w) {
                takeItem = values[i - 1] + dp[i - 1][w - weights[i - 1]];
            }
            
            dp[i][w] = max(skipItem, takeItem);
        }
    }
    
    return dp[n][W];
}
```

---

## Space-Optimized Version

**Key insight:** We only ever need the previous row of the DP table!

**Why this works:** When calculating dp[i][w], we only look at dp[i-1][...]. We never need dp[i-2], dp[i-3], etc. So instead of keeping an entire 2D table, we can use just two 1D arrays: one for the "previous row" and one for the "current row" we're building. This reduces space from O(n × W) to O(W). We can even optimize further to use just one array by carefully updating it backwards!

```cpp
int knapsackSpaceOptimized(vector<int>& weights, vector<int>& values, int W) {
    int n = weights.size();
    vector<int> dp(W + 1, 0);
    
    for (int i = 0; i < n; i++) {
        // Iterate backwards to avoid using updated values
        for (int w = W; w >= weights[i]; w--) {
            dp[w] = max(dp[w], values[i] + dp[w - weights[i]]);
        }
    }
    
    return dp[W];
}
```

**Why backwards?** If we went forward, we'd overwrite dp[w - weights[i]] before using it for the next w. Going backwards ensures we use the "old" values from the previous iteration, which is exactly what we need!

---

## Example Usage

```cpp
int main() {
    // Jewelry store example
    vector<int> weights = {1, 3, 4, 5};
    vector<int> values = {1000, 2000, 3000, 4000};
    int capacity = 7;
    
    // Method 1: Recursive with memoization
    int maxValue1 = knapsack(weights, values, capacity);
    
    // Method 2: Bottom-up DP
    int maxValue2 = knapsackBottomUp(weights, values, capacity);
    
    // Method 3: Space-optimized
    int maxValue3 = knapsackSpaceOptimized(weights, values, capacity);
    
    // All three should give the same answer: 5000
    // (Take Gold Necklace + Silver Watch)
    
    return 0;
}
```

---

## Complexity Analysis

|Approach|Time Complexity|Space Complexity|
|---|---|---|
|Naive Recursion|O(2^n)|O(n) stack space|
|Memoization (Top-Down)|O(n × W)|O(n × W)|
|Bottom-Up DP|O(n × W)|O(n × W)|
|Space-Optimized|O(n × W)|O(W)|

Where:

- n = number of items
- W = capacity of knapsack

---

## Key Intuitions Summary

1. **The Binary Choice:** At each item, you have exactly two options - take it or leave it. This binary nature is why we call it "0/1" knapsack.
    
2. **Optimal Substructure:** The optimal solution contains optimal solutions to subproblems. If you know the best way to pack items 2-n, you can figure out the best way to pack items 1-n by just deciding about item 1.
    
3. **Overlapping Subproblems:** The same subproblem (item i, capacity w) appears multiple times in the recursion tree. Memoization prevents redundant calculations.
    
4. **Building Up vs Breaking Down:** Top-down (recursive) starts with the full problem and breaks it down. Bottom-up starts with base cases and builds to the full problem. Both work!
    
5. **Space Optimization:** Since we only need the previous row, we can reduce space. Going backwards in the array prevents us from overwriting values we still need.
    
6. **The Greedy Approach Fails:** You might think "just take the most valuable items first" or "take the lightest items first", but these greedy approaches don't always give optimal solutions. DP is necessary because we need to consider all possible combinations.
    

---

## When to Use 0/1 Knapsack Pattern

This pattern appears in many problems:

- Subset sum (can we pick items that sum to target?)
- Partition equal subset sum
- Minimum subset sum difference
- Count of subset sum
- Target sum
- Any problem where you select/don't select items to optimize something

The key identifier: **binary choice per item + optimization + constraint**