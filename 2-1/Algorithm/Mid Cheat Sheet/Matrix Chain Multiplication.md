Here are the comprehensive notes for the **Matrix Chain Multiplication (MCM)** family, also known as **Interval DP**. This pattern is distinct because the sub-problems are defined by a range `[i, j]` (start and end indices), requiring iteration by the **length** of that range.

---

# 🔗 Topic: Matrix Chain Multiplication (MCM) & Interval DP

## 1. The Base Problem: Matrix Chain Multiplication

### **Problem Statement**

You are given a sequence of matrices with dimensions. You want to multiply them all together: $A_1 \times A_2 \times \dots \times A_N$.

- **Rule:** Matrix multiplication is associative: $(A \times B) \times C = A \times (B \times C)$. However, the **cost** (number of scalar multiplications) depends heavily on the parenthesization order.
    
- **Cost Formula:** Multiplying a matrix of size $(P \times Q)$ by a matrix of size $(Q \times R)$ requires **$P \times Q \times R$** scalar multiplications. The result is a $(P \times R)$ matrix.
    
- **Goal:** Find the parenthesization that results in the **minimum total cost** to multiply the entire chain.
    
- **Input:** An array `p[]` where matrix $A_i$ has dimensions `p[i-1] x p[i]`.
    
- _Example:_ `p = [10, 20, 30]`.
    
    - Matrix A (Index 1): $10 \times 20$.
        
    - Matrix B (Index 2): $20 \times 30$.
        
    - Cost = $10 \times 20 \times 30 = 6000$ operations.
        

### **Intuition (Why Interval DP?)**

You cannot use a simple greedy approach (e.g., "always multiply smallest matrices first") because a small multiplication now might create a massive matrix that is expensive to use later.

We must decide the **last** multiplication. If we multiply the chain from index $i$ to $j$, there must be some **split point** $k$ ($i \le k < j$) where the final operation happens between the left group $(A_i \dots A_k)$ and the right group $(A_{k+1} \dots A_j)$.

The cost for a specific split $k$ is:

1. Min cost to solve the **Left** part ($i$ to $k$).
    
2. Min cost to solve the **Right** part ($k+1$ to $j$).
    
3. Cost to multiply the two resulting matrices together:
    
    - Result of Left is $(p[i-1] \times p[k])$.
        
    - Result of Right is $(p[k] \times p[j])$.
        
    - Cost = **`p[i-1] * p[k] * p[j]`**.
        

### **State & Recurrence**

> **`dp[i][j]` = Min operations to multiply matrices from index `i` to `j`.**

$$dp[i][j] = \min_{i \le k < j} \{ dp[i][k] + dp[k+1][j] + (p[i-1] \times p[k] \times p[j]) \}$$

### **C++ Implementation**

C++

```
#include <iostream>
#include <vector>
#include <climits>
#include <algorithm>

using namespace std;

int matrixChainOrder(vector<int>& p) {
    int n = p.size(); 
    // Matrices are A1..A(n-1). 
    // dp[i][j] stores min scalar multiplications for chain A_i...A_j
    // Size is n x n. We use indices 1 to n-1.
    vector<vector<int>> dp(n, vector<int>(n, 0));

    // GAP STRATEGY: We must solve for small chains (Length 2) before large chains.
    // L is the chain length.
    for (int L = 2; L < n; L++) {
        
        // i is the starting index of the chain
        for (int i = 1; i < n - L + 1; i++) {
            
            // j is the ending index of the chain
            int j = i + L - 1;
            
            dp[i][j] = INT_MAX;
            
            // Try every possible split position k
            // k goes from i to j-1
            for (int k = i; k < j; k++) {
                int cost = dp[i][k] + dp[k + 1][j] + 
                           (p[i - 1] * p[k] * p[j]);
                
                if (cost < dp[i][j]) {
                    dp[i][j] = cost;
                }
            }
        }
    }
    // Result is the cost for the full chain from 1 to n-1
    return dp[1][n - 1];
}
```

### **How It Works (The Diagonal Traversal)**

1. **Iterate Length (L):** We cannot just loop `i` and `j` normally. We must calculate the diagonal `dp[1][2], dp[2][3]...` first (chains of length 2). Then chains of length 3, etc.
    
2. **Split:** For `dp[1][4]` (Multiplying $A_1A_2A_3A_4$), we try:
    
    - $k=1$: $(A_1) \times (A_2A_3A_4)$
        
    - $k=2$: $(A_1A_2) \times (A_3A_4)$
        
    - $k=3$: $(A_1A_2A_3) \times (A_4)$
        
3. **Complexity:** Time $O(N^3)$, Space $O(N^2)$.
    

---

## 2. Variant: Palindrome Partitioning

### **The Variant**

Given a string `s`, partition it such that **every substring** of the partition is a palindrome.

- **Goal:** Return the **minimum cuts** needed.
    
- _Example:_ `s = "aab"`. Best cuts: `"aa" | "b"`. Answer: 1 cut.
    

### **Logic Shift**

This combines **Interval Pre-processing** with **1D DP**.

1. **Pre-computation:** We build a table `isPal[i][j]` that tells us if `s[i...j]` is a palindrome in $O(1)$. This takes $O(N^2)$.
    
2. **DP State:** `dp[i]` = Min cuts required for the suffix `s[i...N-1]`.
    
3. **Transition:** To find `dp[i]`, we try every possible _first cut_ at `j` (where `j` goes from `i` to `N-1`).
    
    - If `s[i...j]` is a palindrome, we make a cut there (Cost 1) and solve for the rest (`dp[j+1]`).
        

$$dp[i] = \min_{j=i \dots N-1, \text{ if isPal}[i][j]} (1 + dp[j+1])$$

_(Base case: if `isPal[i][N-1]` is true, `dp[i] = 0`)_.

### **C++ Implementation**

C++

```
int minCut(string s) {
    int n = s.length();
    // 1. Precompute Palindromes
    // pal[i][j] is true if s[i..j] is a palindrome
    vector<vector<bool>> pal(n, vector<bool>(n, false));

    for (int len = 1; len <= n; len++) {
        for (int i = 0; i <= n - len; i++) {
            int j = i + len - 1;
            // Check ends match AND inner part is palindrome
            if (s[i] == s[j]) {
                if (len <= 2 || pal[i + 1][j - 1]) {
                    pal[i][j] = true;
                }
            }
        }
    }

    // 2. 1D DP for Min Cuts
    // dp[i] stores min cuts for prefix s[0...i]
    vector<int> dp(n, 0);
    
    for (int i = 0; i < n; i++) {
        if (pal[0][i]) {
            dp[i] = 0; // Entire prefix is palindrome, 0 cuts
        } else {
            dp[i] = INT_MAX;
            for (int j = 0; j < i; j++) {
                // Try cutting after index j
                // If the right part s[j+1...i] is a palindrome...
                if (pal[j + 1][i]) {
                    dp[i] = min(dp[i], dp[j] + 1);
                }
            }
        }
    }
    return dp[n - 1];
}
```

---

## 3. Variant: Optimal Binary Search Tree (OBST)

### **The Variant**

Given a sorted array of `keys` and an array of `freq` (how often each key is searched), build a Binary Search Tree (BST) such that the **expected search cost** is minimized.

- **Cost Definition:** $\text{Cost of Tree} = \sum (\text{freq}[node] \times \text{level}[node])$.
    
- Root is at level 1, children at level 2, etc.
    

### **Logic Shift**

This maps directly to MCM. For a range of keys `i` to `j`, we must select a **Root** `r` ($i \le r \le j$).

- Once we pick `r` as root:
    
    1. Keys `i...r-1` become the **Left Subtree**.
        
    2. Keys `r+1...j` become the **Right Subtree**.
        
    3. **Crucial Insight:** By lifting `r` to be the root, _every single node_ in the subtrees is pushed down by 1 level. This increases the total cost by the sum of all frequencies in the range `[i, j]`.
        

$$dp[i][j] = \sum_{k=i}^{j} \text{freq}[k] + \min_{i \le r \le j} (dp[i][r-1] + dp[r+1][j])$$

### **C++ Implementation**

C++

```
#include <iostream>
#include <vector>
#include <climits>
#include <numeric>

using namespace std;

// Helper to get sum of frequencies from i to j
int sum(const vector<int>& freq, int i, int j) {
    int s = 0;
    for (int k = i; k <= j; k++) s += freq[k];
    return s;
}

int optimalSearchTree(vector<int>& keys, vector<int>& freq, int n) {
    // dp[i][j] = Cost of optimal BST using keys[i] to keys[j]
    vector<vector<int>> dp(n, vector<int>(n, 0));

    // 1. Single keys (Length 1)
    for (int i = 0; i < n; i++) dp[i][i] = freq[i];

    // 2. Length 2 to n
    for (int L = 2; L <= n; L++) {
        for (int i = 0; i <= n - L; i++) {
            int j = i + L - 1;
            dp[i][j] = INT_MAX;
            int offset_sum = sum(freq, i, j);

            // Try making every key 'r' the root
            for (int r = i; r <= j; r++) {
                int leftCost = (r > i) ? dp[i][r - 1] : 0;
                int rightCost = (r < j) ? dp[r + 1][j] : 0;
                
                int val = leftCost + rightCost + offset_sum;
                if (val < dp[i][j]) dp[i][j] = val;
            }
        }
    }
    return dp[0][n - 1];
}
```

---

## 4. Variant: Burst Balloons (Reverse Thinking)

### **The Variant**

You are given `N` balloons. Bursting balloon `i` gives coins = `nums[i-1] * nums[i] * nums[i+1]`.

- After bursting, the neighbors become adjacent.
    
- Boundaries (`-1` and `N`) are treated as having value 1.
    
- **Goal:** Maximize coins collected.
    

### **Logic Shift (The "Last One" Trick)**

Standard Interval DP fails here because if you burst a balloon, the indices of neighbors change.

**Solution:** Instead of picking which balloon to burst _first_, we pick which balloon to burst **LAST**.

- Consider range `i` to `j`. Let's say we decide that balloon `k` ($i \le k \le j$) will be the **very last** balloon burst in this range.
    
- Since `k` is last, its immediate neighbors must be the boundaries of the range: `i-1` and `j+1`.
    
- Sub-problems: The range `i` to `k-1` and `k+1` to `j` are solved independently.
    

$$dp[i][j] = \max_{i \le k \le j} \{ dp[i][k-1] + dp[k+1][j] + (\text{nums}[i-1] \times \text{nums}[k] \times \text{nums}[j+1]) \}$$

### **C++ Implementation**

C++

```
int maxCoins(vector<int>& nums) {
    int n = nums.size();
    vector<int> arr(n + 2);
    arr[0] = 1; arr[n + 1] = 1;
    for(int i=0; i<n; i++) arr[i+1] = nums[i];
    
    vector<vector<int>> dp(n + 2, vector<int>(n + 2, 0));

    for (int len = 1; len <= n; len++) {
        for (int i = 1; i <= n - len + 1; i++) {
            int j = i + len - 1;
            for (int k = i; k <= j; k++) {
                int coins = dp[i][k - 1] + dp[k + 1][j] + 
                            (arr[i - 1] * arr[k] * arr[j + 1]);
                dp[i][j] = max(dp[i][j], coins);
            }
        }
    }
    return dp[1][n];
}
```

---

## 5. Variant: Boolean Parenthesization

### **The Variant**

Given a boolean expression string `T|F&T^F...` containing symbols `T` (True), `F` (False) and operators `|` (OR), `&` (AND), `^` (XOR).

- **Goal:** Count the number of ways to parenthesize the expression such that it evaluates to **True**.
    

### **Logic Shift**

We need two DP tables:

1. `T[i][j]`: Ways to make range `i..j` evaluate to True.
    
2. `F[i][j]`: Ways to make range `i..j` evaluate to False.
    

When splitting at operator `k` (say `&`):

- `TotalTrue += T[i][k] * T[k+1][j]` (Since True & True = True).
    
- `TotalFalse` accounts for other combinations (True & False, False & True, False & False).
    

### **C++ Implementation**

C++

```
int countWaysToTrue(string S) {
    int n = S.length();
    int numOperands = (n + 1) / 2;
    vector<vector<int>> T(numOperands, vector<int>(numOperands, 0));
    vector<vector<int>> F(numOperands, vector<int>(numOperands, 0));

    for (int i = 0; i < numOperands; i++) {
        if (S[i * 2] == 'T') T[i][i] = 1;
        else F[i][i] = 1;
    }

    for (int L = 2; L <= numOperands; L++) {
        for (int i = 0; i <= numOperands - L; i++) {
            int j = i + L - 1;
            for (int k = i; k < j; k++) {
                int total_ik = T[i][k] + F[i][k];
                int total_kj = T[k + 1][j] + F[k + 1][j];
                char op = S[k * 2 + 1];

                if (op == '&') {
                    T[i][j] += T[i][k] * T[k + 1][j];
                    F[i][j] += (total_ik * total_kj) - (T[i][k] * T[k + 1][j]);
                } else if (op == '|') {
                    F[i][j] += F[i][k] * F[k + 1][j];
                    T[i][j] += (total_ik * total_kj) - (F[i][k] * F[k + 1][j]);
                } else if (op == '^') {
                    T[i][j] += F[i][k] * T[k + 1][j] + T[i][k] * F[k + 1][j];
                    F[i][j] += T[i][k] * T[k + 1][j] + F[i][k] * F[k + 1][j];
                }
            }
        }
    }
    return T[0][numOperands - 1];
}
```

---

## 6. Variant: Min/Max Value of Expression

### **The Variant**

Given an arithmetic expression (e.g., `1+2*3-4`). Add parentheses to maximize/minimize the result.

### **Logic Shift**

Similar to Boolean Parenthesization, but we track **Min** and **Max** values because multiplying two negative numbers can create a maximum.

- `maxVal[i][j]` and `minVal[i][j]`.
    
- At split `k`, calculate 4 products: `min*min`, `min*max`, `max*min`, `max*max`.
    

### **C++ Implementation**

C++

```
long long calculate(long long a, long long b, char op) {
    if (op == '+') return a + b;
    if (op == '-') return a - b;
    if (op == '*') return a * b;
    return 0;
}

void minMaxExpression(vector<int>& nums, vector<char>& ops) {
    int n = nums.size();
    vector<vector<long long>> minVal(n, vector<long long>(n));
    vector<vector<long long>> maxVal(n, vector<long long>(n));

    for (int i = 0; i < n; i++) minVal[i][i] = maxVal[i][i] = nums[i];

    for (int L = 2; L <= n; L++) {
        for (int i = 0; i <= n - L; i++) {
            int j = i + L - 1;
            minVal[i][j] = LLONG_MAX; maxVal[i][j] = LLONG_MIN;

            for (int k = i; k < j; k++) {
                long long a = calculate(minVal[i][k], minVal[k + 1][j], ops[k]);
                long long b = calculate(minVal[i][k], maxVal[k + 1][j], ops[k]);
                long long c = calculate(maxVal[i][k], minVal[k + 1][j], ops[k]);
                long long d = calculate(maxVal[i][k], maxVal[k + 1][j], ops[k]);

                minVal[i][j] = min({minVal[i][j], a, b, c, d});
                maxVal[i][j] = max({maxVal[i][j], a, b, c, d});
            }
        }
    }
}
```