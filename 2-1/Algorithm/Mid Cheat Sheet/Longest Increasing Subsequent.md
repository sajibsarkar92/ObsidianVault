Here are the comprehensive notes for the **Longest Increasing Subsequence (LIS)** family of problems.

_Note: I have combined **LIS** (Problem #5) and **MSIS** (Problem #4) into one note because MSIS is mathematically just a tiny variation of LIS. Learning LIS first makes MSIS trivial._

---

# 📈 Topic: Longest Increasing Subsequence (LIS) & Variants

## 1. The Base Problem: Standard LIS (O(N²) Version)

### **Problem Statement**

Given an integer array `nums`, return the length of the longest strictly increasing subsequence.

- **Subsequence:** Delete some elements, keep the rest in order.
    
- **Strictly Increasing:** `[2, 3, 5]` is valid. `[2, 2, 5]` is not.
    
- _Example:_ `nums = [10, 9, 2, 5, 3, 7, 101, 18]`
    
    - LIS is `[2, 3, 7, 18]`. Length = 4.
        

### **Intuition & Logic**

This problem differs from LCS because we only have **one** array.

The core difficulty is that we don't know _where_ the subsequence ends. It could end at index 5, or index 20.

We define our state based on the **ending index**:

> **`dp[i]` = The length of the longest increasing subsequence that ENDS at index `i`.**

To calculate `dp[i]`:

1. We look back at all previous indices `j` (where `0 <= j < i`).
    
2. If `nums[j] < nums[i]`, it means we can technically extend the sequence ending at `j` by appending `nums[i]`.
    
3. We take the maximum of all such extensions.
    

$$dp[i] = 1 + \max(\{dp[j] \mid 0 \le j < i \text{ and } nums[j] < nums[i]\} \cup \{0\})$$

### **C++ Implementation**

C++

```
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

int lengthOfLIS(vector<int>& nums) {
    int n = nums.size();
    if (n == 0) return 0;

    // dp[i] initializes to 1 because every element is an LIS of length 1 by itself
    vector<int> dp(n, 1);
    
    int overallMax = 1;

    for (int i = 1; i < n; i++) {
        // Check all previous elements
        for (int j = 0; j < i; j++) {
            // If valid extension found
            if (nums[i] > nums[j]) {
                dp[i] = max(dp[i], 1 + dp[j]);
            }
        }
        overallMax = max(overallMax, dp[i]);
    }
    
    return overallMax;
}
```

### **How It Works**

1. **State:** `dp` array is `[1, 1, 1, 1, ...]`.
    
2. **Inner Loop:** For index `i=3` (value 5), it looks back:
    
    - Is 2 < 5? Yes. `dp[3]` becomes `dp[index_of_2] + 1` = 2.
        
    - Is 9 < 5? No.
        
    - Is 10 < 5? No.
        
3. **Result:** The answer is the max value in the _entire_ `dp` array, not just the last element (because the longest sequence might end earlier).
    

---

## 2. Variant: Maximum Sum Increasing Subsequence (MSIS)

### **The Variant**

Instead of maximizing the _length_ (number of elements), maximize the **sum** of the elements in the increasing subsequence.

- _Example:_ `[1, 101, 2, 3, 100, 4, 5]`
    
    - LIS Length: 5 (`1, 2, 3, 4, 5`) $\to$ Sum: 15
        
    - MSIS Sum: 106 (`1, 5, 100`)? No.
        
    - Best MSIS: `1, 2, 3, 100` $\to$ Sum: 106. Actually `1, 101` is 102. `101` alone is 101.
        
    - Real Max: `1, 2, 3, 100` = 106. (Wait, `1, 101` is valid. `101` is valid).
        
    - Best is `[1, 101]`. Sum 102. (Wait, `[1, 2, 3, 100]` is sum 106).
        
    - Correct Logic: Sum is 106.
        

### **Logic Shift**

The structure is identical to LIS. We just change the "score" function.

- LIS adds **+1** (count) for extending a sequence.
    
- MSIS adds **+nums[i]** (value) for extending a sequence.
    

### **C++ Implementation**

C++

```
int maxSumIS(vector<int>& nums) {
    int n = nums.size();
    // Initialize dp[i] with nums[i] (base sum is the element itself)
    vector<int> dp = nums; 
    
    int overallMax = dp[0];

    for (int i = 1; i < n; i++) {
        for (int j = 0; j < i; j++) {
            // Same increasing condition
            if (nums[i] > nums[j]) {
                // Update sum instead of length
                dp[i] = max(dp[i], dp[j] + nums[i]);
            }
        }
        overallMax = max(overallMax, dp[i]);
    }
    return overallMax;
}
```

---

## 3. Variant: LIS in O(N log N)

### **The Variant**

The $O(N^2)$ solution is too slow for $N = 10^5$. We need a faster way.

- _Note:_ This is technically a "Patience Sorting" approach, not pure DP, but it's the standard optimal solution required in exams.
    

### **Logic Shift**

We maintain a `tails` array. `tails[i]` stores the **smallest ending element** of all increasing subsequences of length `i+1`.

- **Greedy Strategy:** If we want to extend a subsequence of length 2, we prefer one that ends with 3 over one that ends with 5, because 3 makes it easier to add more numbers later.
    
- The `tails` array will always be sorted. We can use **Binary Search** to find update positions.
    

### **C++ Implementation**

C++

```
int lengthOfLISOptimized(vector<int>& nums) {
    if (nums.empty()) return 0;
    
    vector<int> tails;
    
    for (int num : nums) {
        // Binary search for the first element in 'tails' >= num
        // lower_bound returns iterator to first element >= num
        auto it = lower_bound(tails.begin(), tails.end(), num);
        
        if (it == tails.end()) {
            // If num is greater than all tails, it extends the longest LIS found so far
            tails.push_back(num);
        } else {
            // If num is smaller, it replaces an existing tail
            // This allows us to form a sequence of the SAME length but with a SMALLER ending
            *it = num;
        }
    }
    return tails.size();
}
```

---

## 4. Variant: Longest Bitonic Subsequence

### **The Variant**

Find the longest subsequence that first **increases** and then **decreases**.

- _Example:_ `[1, 11, 2, 10, 4, 5, 2, 1]`
    
- Valid Bitonic: `1, 2, 10, 5, 2, 1` (Length 6).
    

### **Logic Shift**

A bitonic sequence has a "peak" element.

1. Compute `LIS[i]`: Longest Increasing Subsequence ending at `i` (Left to Right).
    
2. Compute `LDS[i]`: Longest Decreasing Subsequence starting at `i` (Right to Left).
    
    - _Note:_ LDS from Right to Left is the same logic as LIS.
        
3. The max bitonic length utilizing element `i` as the peak is `LIS[i] + LDS[i] - 1`.
    
    - We subtract 1 because the peak element `i` is counted in both sequences.
        

### **C++ Implementation**

C++

```
int longestBitonicSubsequence(vector<int>& nums) {
    int n = nums.size();
    if (n == 0) return 0;

    vector<int> lis(n, 1);
    vector<int> lds(n, 1);

    // 1. Compute LIS (Left to Right)
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (nums[i] > nums[j]) lis[i] = max(lis[i], lis[j] + 1);
        }
    }

    // 2. Compute LDS (Right to Left)
    // Conceptually LIS on the reversed array
    for (int i = n - 1; i >= 0; i--) {
        for (int j = n - 1; j > i; j--) {
            if (nums[i] > nums[j]) lds[i] = max(lds[i], lds[j] + 1);
        }
    }

    // 3. Find Max Peak
    int maxLen = 0;
    for (int i = 0; i < n; i++) {
        // Constraint: A strict bitonic sequence usually requires both increasing and decreasing parts
        // but often the problem allows just increasing or just decreasing.
        // Assuming general definition:
        maxLen = max(maxLen, lis[i] + lds[i] - 1);
    }
    return maxLen;
}
```

---

## 5. Variant: Number of LIS

### **The Variant**

Find the **number of different** longest increasing subsequences.

- _Example:_ `[1, 3, 5, 4, 7]`
    
    - LIS Length is 4.
        
    - Subsequences: `1, 3, 5, 7` and `1, 3, 4, 7`.
        
    - Answer: 2.
        

### **Logic Shift**

We need a second array `count[i]`.

- `dp[i]`: Length of LIS ending at `i`.
    
- `count[i]`: Number of ways to achieve that length.
    
- **Update Rule:**
    
    - If `dp[j] + 1 > dp[i]`: We found a _new_ max length. `dp[i]` updates, and `count[i]` becomes `count[j]`.
        
    - If `dp[j] + 1 == dp[i]`: We found _another way_ to reach the current max length. `count[i] += count[j]`.
        

### **C++ Implementation**

C++

```
int findNumberOfLIS(vector<int>& nums) {
    int n = nums.size();
    vector<int> dp(n, 1);
    vector<int> count(n, 1);

    int maxLen = 1;

    for (int i = 0; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (nums[i] > nums[j]) {
                if (dp[j] + 1 > dp[i]) {
                    // Found a longer path
                    dp[i] = dp[j] + 1;
                    count[i] = count[j];
                } else if (dp[j] + 1 == dp[i]) {
                    // Found another path of same length
                    count[i] += count[j];
                }
            }
        }
        maxLen = max(maxLen, dp[i]);
    }

    // Sum up counts of all indices that achieved the maxLen
    int totalOps = 0;
    for (int i = 0; i < n; i++) {
        if (dp[i] == maxLen) totalOps += count[i];
    }
    return totalOps;
}
```

---

## 6. Variant: Building Bridges (LIS on Pairs)

### **The Variant**

You have pairs of cities $(North_i, South_i)$ on two parallel river banks. You want to build bridges connecting corresponding cities $(N_i \to S_i)$.

- **Constraint:** Bridges cannot cross each other.
    
- **Goal:** Maximize number of bridges.
    

### **Logic Shift**

This reduces to LIS.

1. **Sort** the pairs based on the North bank coordinates. (If North coordinates are equal, sort by South).
    
2. Once sorted by North, any valid non-crossing selection must have **increasing South coordinates**.
    
3. Problem becomes: Find **LIS of the South coordinates**.
    

### **C++ Implementation**

C++

```
struct Bridge { int n, s; };

// Comparator: Sort by North ascending. 
// If North is same, sort South ascending to allow multiple bridges from same coordinate (if allowed)
// or logic depends on exact problem statement.
bool compareBridges(Bridge a, Bridge b) {
    if (a.n != b.n) return a.n < b.n;
    return a.s < b.s;
}

int maxBridges(vector<Bridge>& bridges) {
    sort(bridges.begin(), bridges.end(), compareBridges);
    
    // Extract South coordinates
    vector<int> southCoords;
    for(auto b : bridges) southCoords.push_back(b.s);
    
    // Run LIS on South coordinates
    // (Can use O(N^2) or O(N log N) version)
    return lengthOfLIS(southCoords); 
}
```

---

## 7. Variant: Box Stacking Problem (3D LIS)

### **The Variant**

You are given a set of 3D boxes with dimensions $(L, W, H)$. You want to stack them to maximize total height.

- **Constraint:** Box A can go on Box B only if `A.L < B.L` AND `A.W < B.W`. (Strictly smaller base).
    
- **Rotation:** You can rotate boxes (use any dimension as height).
    

### **Logic Shift**

1. **Generate Rotations:** For every box $(l, w, h)$, generate all 3 valid orientations: $(l, w, h), (w, h, l), (h, l, w)$.
    
    - Ensure usually $length \ge width$ for consistency.
        
2. **Sort:** Sort all rotations by Base Area ($L \times W$) in descending order. (Big bases at bottom).
    
3. **DP (MSIS variation):** `dp[i]` = Max Height ending with box `i` on top.
    
    - Check all boxes `j` below `i` (indices $0 \dots i-1$).
        
    - If `box[i]` fits on `box[j]` ($L_i < L_j$ and $W_i < W_j$), then `dp[i] = max(dp[i], dp[j] + height[i])`.
        

### **C++ Snippet**

C++

```
struct Box { int l, w, h; };

// 1. Generate all 3 rotations for every input box
// 2. Sort boxes by Area (l * w) Descending
// 3. Apply logic similar to Maximum Sum Increasing Subsequence
//    But instead of summing array values, we sum 'h'
//    And condition is (b1.l > b2.l && b1.w > b2.w)

int maxStackHeight(vector<Box>& boxes) {
    // ... Pre-processing sort ...
    int n = boxes.size();
    vector<int> dp(n);
    
    int maxHeight = 0;
    for(int i = 0; i < n; i++) {
        dp[i] = boxes[i].h; // Base height
        for(int j = 0; j < i; j++) {
            // Check if i can sit on top of j
            if (boxes[i].l < boxes[j].l && boxes[i].w < boxes[j].w) {
                dp[i] = max(dp[i], dp[j] + boxes[i].h);
            }
        }
        maxHeight = max(maxHeight, dp[i]);
    }
    return maxHeight;
}
```