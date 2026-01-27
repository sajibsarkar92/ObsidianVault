# 0/1 Knapsack - Building the DP Table Intuitively

## The Problem Setup

You have a knapsack with maximum capacity **W**, and **n** items, each with:

- **weight[i]** - how heavy the item is
- **value[i]** - how valuable the item is

**Goal:** Maximize total value without exceeding weight capacity W.

**Constraint:** Each item can be taken 0 or 1 times (you can't take half an item or the same item twice).

---

## Example We'll Use Throughout

Let's work with a concrete example:

**Knapsack Capacity: 7 kg**

|Item|Weight|Value|
|---|---|---|
|1|1|1|
|2|3|4|
|3|4|5|
|4|5|7|

---

## Understanding the DP Table

### What Does dp[i][w] Mean?

**`dp[i][w]`** = The maximum value we can achieve by considering the **first i items** with a knapsack capacity of **w**.

Think of it as answering the question:

> "If I'm only allowed to choose from the first i items, and my bag can hold w kg, what's the best total value I can get?"

### Table Dimensions

- **Rows:** Represent items (0 to n)
    
    - Row 0 = "using 0 items" (base case)
    - Row i = "considering items 1 through i"
- **Columns:** Represent capacities (0 to W)
    
    - Column 0 = "capacity is 0 kg" (base case)
    - Column w = "capacity is w kg"

For our example: **5 rows × 8 columns** (items 0-4, capacity 0-7)

---

## Building the Table Step by Step

### Step 1: Initialize the Base Cases

**Base Case 1: Row 0 (No items)** If we have 0 items available, no matter what capacity we have, the maximum value is always 0.

**Base Case 2: Column 0 (No capacity)** If capacity is 0, no matter how many items we have, we can't take anything, so value is 0.

**Initial Table:**

```
        Capacity (w)
        0   1   2   3   4   5   6   7
    0   0   0   0   0   0   0   0   0
i=1     0   
i=2     0   
i=3     0   
i=4     0   
```

The first row and first column are all zeros - these are our foundation!

---

### Step 2: Fill Row 1 (Considering Item 1: w=1, v=1)

Now we consider **item 1** (weight=1, value=1) for each capacity from 1 to 7.

**For each capacity w, we ask:**

1. **Can I take item 1?** (Does weight[1] ≤ w?)
2. If yes: **Should I take it?** (Does taking it give more value than not taking it?)

Let's go column by column:

**dp[1][1]:** Capacity = 1 kg

- Can we take item 1 (weight=1)? YES, it fits exactly!
- Option 1 (don't take): dp[0][1] = 0
- Option 2 (take): value[1] + dp[0][1-1] = 1 + dp[0][0] = 1 + 0 = 1
- **Choose max(0, 1) = 1**

**dp[1][2]:** Capacity = 2 kg

- Can we take item 1? YES
- Option 1 (don't take): dp[0][2] = 0
- Option 2 (take): 1 + dp[0][1] = 1 + 0 = 1
- **Choose max(0, 1) = 1**

**Similarly for dp[1][3] through dp[1][7]:** All are 1, because we can always take item 1 (it only weighs 1 kg).

**Table after Row 1:**

```
        Capacity (w)
        0   1   2   3   4   5   6   7
    0   0   0   0   0   0   0   0   0
i=1     0   1   1   1   1   1   1   1
i=2     0   
i=3     0   
i=4     0   
```

**Intuition:** With only item 1 available, if we have any capacity ≥ 1, we take it and get value 1.

---

### Step 3: Fill Row 2 (Adding Item 2: w=3, v=4)

Now we can choose from **items 1 and 2**. Item 2 weighs 3 kg and is worth 4.

**dp[2][1]:** Capacity = 1 kg

- Can we take item 2 (weight=3)? NO, too heavy!
- So we can only use the result from previous items: dp[1][1] = 1
- **dp[2][1] = 1**

**dp[2][2]:** Capacity = 2 kg

- Can we take item 2? NO, still too heavy!
- **dp[2][2] = dp[1][2] = 1**

**dp[2][3]:** Capacity = 3 kg

- Can we take item 2? YES, it fits exactly!
- Option 1 (don't take): dp[1][3] = 1
- Option 2 (take): value[2] + dp[1][3-3] = 4 + dp[1][0] = 4 + 0 = 4
- **Choose max(1, 4) = 4**

**dp[2][4]:** Capacity = 4 kg

- Can we take item 2? YES
- Option 1 (don't take): dp[1][4] = 1
- Option 2 (take): 4 + dp[1][1] = 4 + 1 = 5
    - This means: take item 2 (value 4), and with remaining capacity 1, take item 1 (value 1)
- **Choose max(1, 5) = 5**

**dp[2][5] through dp[2][7]:** All become 5

- We can always take item 2 (value 4) + item 1 (value 1) = 5 total

**Table after Row 2:**

```
        Capacity (w)
        0   1   2   3   4   5   6   7
    0   0   0   0   0   0   0   0   0
i=1     0   1   1   1   1   1   1   1
i=2     0   1   1   4   5   5   5   5
i=3     0   
i=4     0   
```

**Intuition:** Now with 2 items, we can get value 5 by taking both items (needs 4 kg). If we only have 3 kg, we take just item 2 for value 4.

---

### Step 4: Fill Row 3 (Adding Item 3: w=4, v=5)

Now we can choose from **items 1, 2, and 3**. Item 3 weighs 4 kg and is worth 5.

**dp[3][1] to dp[3][3]:**

- Item 3 doesn't fit (needs 4 kg)
- Copy from row 2: dp[3][1]=1, dp[3][2]=1, dp[3][3]=4

**dp[3][4]:** Capacity = 4 kg

- Can we take item 3? YES
- Option 1 (don't take): dp[2][4] = 5 (take items 1+2)
- Option 2 (take): value[3] + dp[2][0] = 5 + 0 = 5 (take only item 3)
- **Choose max(5, 5) = 5** (both options are equally good!)

**dp[3][5]:** Capacity = 5 kg

- Option 1 (don't take): dp[2][5] = 5
- Option 2 (take): 5 + dp[2][1] = 5 + 1 = 6
    - Take item 3 (value 5), with remaining 1 kg take item 1 (value 1)
- **Choose max(5, 6) = 6**

**dp[3][6]:** Capacity = 6 kg

- Option 1: dp[2][6] = 5
- Option 2: 5 + dp[2][2] = 5 + 1 = 6
- **Choose max(5, 6) = 6**

**dp[3][7]:** Capacity = 7 kg

- Option 1: dp[2][7] = 5
- Option 2: 5 + dp[2][3] = 5 + 4 = 9
    - Take item 3 (value 5, weight 4), with remaining 3 kg take item 2 (value 4)
- **Choose max(5, 9) = 9**

**Table after Row 3:**

```
        Capacity (w)
        0   1   2   3   4   5   6   7
    0   0   0   0   0   0   0   0   0
i=1     0   1   1   1   1   1   1   1
i=2     0   1   1   4   5   5   5   5
i=3     0   1   1   4   5   6   6   9
i=4     0   
```

**Intuition:** With 7 kg capacity, we can now take item 2 + item 3 for total value 9!

---

### Step 5: Fill Row 4 (Adding Item 4: w=5, v=7)

Final item: weight=5, value=7.

**dp[4][1] to dp[4][4]:**

- Item 4 doesn't fit
- Copy from row 3: 1, 1, 4, 5

**dp[4][5]:** Capacity = 5 kg

- Can we take item 4? YES, fits exactly!
- Option 1 (don't take): dp[3][5] = 6
- Option 2 (take): 7 + dp[3][0] = 7 + 0 = 7
- **Choose max(6, 7) = 7**

**dp[4][6]:** Capacity = 6 kg

- Option 1: dp[3][6] = 6
- Option 2: 7 + dp[3][1] = 7 + 1 = 8
    - Take item 4 (value 7), with remaining 1 kg take item 1 (value 1)
- **Choose max(6, 8) = 8**

**dp[4][7]:** Capacity = 7 kg

- Option 1: dp[3][7] = 9 (items 2+3)
- Option 2: 7 + dp[3][2] = 7 + 1 = 8 (items 4+1)
- **Choose max(9, 8) = 9**

**Final Complete Table:**

```
        Capacity (w)
        0   1   2   3   4   5   6   7
    0   0   0   0   0   0   0   0   0
i=1     0   1   1   1   1   1   1   1
i=2     0   1   1   4   5   5   5   5
i=3     0   1   1   4   5   6   6   9
i=4     0   1   1   4   5   7   8   9
```

**Answer: dp[4][7] = 9**

The maximum value with 4 items and 7 kg capacity is **9**.

---

## The Recurrence Relation Explained

For each cell dp[i][w]:

```
if weight[i] > w:
    dp[i][w] = dp[i-1][w]  // Can't take item i, inherit from above
else:
    dp[i][w] = max(
        dp[i-1][w],                           // Don't take item i
        value[i] + dp[i-1][w - weight[i]]     // Take item i
    )
```

### Visual Understanding

When filling dp[i][w], we look at:

1. **Cell directly above** (dp[i-1][w]): "Best value without item i"
2. **Cell diagonally above-left** (dp[i-1][w-weight[i]]): "Best value with remaining capacity if we take item i"

```
        ... w-weight[i] ...  w
        
i-1     ... [look here] ... [or here]
                ↘            ↓
i       ...                 [current cell]
```

We take the maximum of these two options!

---

## Visualizing the Decision Path

Let's trace back **how we got to value 9** at dp[4][7]:

**At dp[4][7] = 9:**

- We chose option 1 (don't take item 4)
- Came from dp[3][7] = 9

**At dp[3][7] = 9:**

- We chose option 2 (take item 3, weight=4, value=5)
- Came from: 5 + dp[2][3] = 5 + 4 = 9

**At dp[2][3] = 4:**

- We chose option 2 (take item 2, weight=3, value=4)
- Came from: 4 + dp[1][0] = 4 + 0 = 4

**Items selected: Item 2 (w=3, v=4) + Item 3 (w=4, v=5)**

- Total weight: 3 + 4 = 7 kg ✓
- Total value: 4 + 5 = 9 ✓

---

## Key Insights from Table Building

### 1. **Row by Row = Item by Item**

Each row represents adding one more item to our "pool" of available items. As we go down rows, we have more choices.

### 2. **Columns Show Capacity Constraints**

Each column shows what happens with different weight limits. Higher capacities (right columns) generally allow higher values.

### 3. **Values Never Decrease**

As you move right (more capacity) or down (more items), values never decrease. You can always achieve at least as much as before because you have more options.

### 4. **The Magic of "Remaining Capacity"**

When we take an item, we look at dp[i-1][w - weight[i]]. This is asking: "What's the best I could do with the previous items using the leftover capacity?" This is the key to combining items optimally!

### 5. **Why Bottom-Up Works**

Each cell only depends on cells above it and to the left. Since we fill left-to-right, top-to-bottom, all dependencies are already computed when we need them.

---

## Common Patterns You'll Notice

### Pattern 1: Item Too Heavy

When an item doesn't fit, the entire row just copies from the row above for those capacities.

```
i-1     0   1   1   4   5   5   5   5
i       0   1   1   4   5   ...
        ↓   ↓   ↓   ↓   ↓
    (Item too heavy for capacities 1-4, so copy from above)
```

### Pattern 2: Item Fits Exactly

When capacity exactly equals item weight, we compare:

- Not taking it (value from above)
- Taking it (item's value + dp[i-1][0] = item's value + 0)

### Pattern 3: "Stair Step" Increases

Often you'll see values increase in a "stair step" pattern as capacity increases, because new items become usable at certain capacity thresholds.

---

## Practice Exercise

Try building the table for this example yourself:

**Capacity: 10 kg**

|Item|Weight|Value|
|---|---|---|
|1|2|6|
|2|2|10|
|3|3|12|

**Your table should be 4×11 (rows 0-3, columns 0-10)**

Start with the base cases, then fill row by row following the recurrence relation!

<details> <summary>Click to see the answer</summary>

```
        Capacity (w)
        0   1   2   3   4   5   6   7   8   9   10
    0   0   0   0   0   0   0   0   0   0   0   0
i=1     0   0   6   6   6   6   6   6   6   6   6
i=2     0   0  10  10  16  16  16  16  16  16  16
i=3     0   0  10  12  16  22  22  28  28  28  28
```

**Maximum value = 28** (take all three items: 6+10+12=28, weight: 2+2+3=7)

</details>

---

## Why This Approach Works: The Deep Intuition

Think of the DP table as a **map of all possible scenarios**:

- Each cell represents a specific scenario: "X items available, Y capacity"
- We systematically explore every scenario
- For each scenario, we make the locally optimal choice
- Because we've already solved all smaller scenarios optimally, our choice leads to global optimality

The beauty is: **we don't need to try all 2^n combinations** (which would be exponential). Instead, we solve n×W unique scenarios, reusing solutions to build bigger solutions.

This is the essence of dynamic programming: **optimal substructure + overlapping subproblems = efficient solution**.