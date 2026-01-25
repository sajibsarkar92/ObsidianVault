

# Matrix Chain Multiplication: The "Parentheses Game"

## The Core Concept

Forget matrices for a second. Imagine you have a long chain of numbers to multiply: $A \times B \times C \times D$.

You can't multiply them all at once. You have to pick **two** neighbors, multiply them to get a result, and then multiply that result by the next one.

- You could do: $(A \times B)$ first... then multiply by $C$... then $D$.
    
- Or: $(B \times C)$ first... then multiply $A$ by that result... then $D$.
    

In Matrix multiplication, the **order doesn't change the answer, but it changes the COST.**

- **Bad Order:** The computer does 1,000,000 calculations.
    
- **Good Order:** The computer does 5,000 calculations.
    

**The Goal:** We want to find the perfect places to put the parentheses `( )` to make the cost as small as possible.

<iframe width="560" height="315" src="https://www.youtube.com/embed/prx1psByp7U?si=JF2THD6SB8ShV7c_" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>


---

## Step-by-Step Logic & Implementation

### Step 1: The Input (The "Dimensions Array")

We don't need the actual numbers inside the matrices. We only care about their **shape** (rows and columns).

We store these in an array `p[]`.

- If `p = {10, 30, 5}`, it means:
    
    - Matrix A is **10x30**.
        
    - Matrix B is **30x5**.
        

C++

```cpp
#include <vector>
#include <climits>
#include <iostream>
using namespace std;

// p[] stores dimensions. A[i] is p[i-1] x p[i]
vector<int> p = {10, 30, 5, 60}; 
int n = p.size() - 1; // Number of matrices
```

### Step 2: The Scoreboard (The DP Table)

We need a place to write down the answers to small problems so we don't have to re-calculate them.

- `m[i][j]` answers the question: **"What is the cheapest cost to multiply everything from Matrix i to Matrix j?"**
    
- **The Diagonal (`m[i][i]`):** The cost to multiply **one** matrix is 0. (It's already done).
    

C++

```cpp
// m[i][j] = Best cost for range i to j
vector<vector<int>> m(n + 1, vector<int>(n + 1));

// Initialize diagonal to 0
for (int i = 1; i <= n; i++) m[i][i] = 0;
```

### Step 3: The "Length" Loop (Building Up)

**The Logic:** You cannot figure out the best way to multiply 5 matrices until you know the best way to multiply 2, 3, and 4 matrices.

So, we iterate by **Chain Length (`L`)**.

1. Solve all pairs (Length 2).
    
2. Solve all triplets (Length 3).
    
3. ...Up to the full chain.
    

C++

```cpp
// L is the length of the sub-chain we are solving right now
for (int L = 2; L <= n; L++) {
```

### Step 4: The "Sliding Window" Loop (Start and End)

**The Logic:** If we are looking for chains of Length 2, we need to check **every** pair: `(A,B)`, `(B,C)`, `(C,D)`.

We slide a "window" of size `L` across our list of matrices.

- `i` is the start of the window.
    
- `j` is the end of the window.
    

C++

```cpp
    // i is the start index
    for (int i = 1; i <= n - L + 1; i++) {
        int j = i + L - 1; // j is the end index
        m[i][j] = INT_MAX; // Assume it's infinite cost at first
```

### Step 5: The "Split" Loop (The Decision Maker `k`)

**The Logic:** This is the most important part.

Imagine you have a chain from $i$ to $j$: `A x B x C x D`.

At the _very end_, this long chain must have been formed by multiplying **two pieces** together. Where was the cut?

- Was it `(A) * (B C D)`? (Cut after A)
    
- Was it `(A B) * (C D)`? (Cut after B)
    
- Was it `(A B C) * (D)`? (Cut after C)
    

We define `k` as the **split point**. We try cutting at _every_ possible `k`.

C++

```cpp
        // k is the split point. Try cutting anywhere between i and j-1
        for (int k = i; k <= j - 1; k++) {
```

### Step 6: Calculating the Cost

**The Logic:** The cost for a specific cut `k` is the sum of three things:

1. Cost to solve the **Left Side** (we already calculated this in an earlier loop!).
    
2. Cost to solve the **Right Side** (we already calculated this too!).
    
3. Cost to **Weld** the two sides together (Left Rows $\times$ Common $\times$ Right Cols).
    

C++

```cpp
            // Cost = Left + Right + Welding Cost
            int q = m[i][k] + m[k+1][j] + (p[i-1] * p[k] * p[j]);

            // If this cut is cheaper than previous best, save it!
            if (q < m[i][j]) {
                m[i][j] = q;
            }
        }
    }
}
```

### Step 7: The Result

The answer to "What is the best cost for the whole chain (1 to n)?" is sitting in the top-right corner of our table.

C++

```cpp
cout << "Cheapest Cost: " << m[1][n] << endl;
```

---

## Why does this work? (Summary)

It works because of **Dynamic Programming**.

When the loops get to the big chains (Length 5), the inner math `m[i][k]` and `m[k+1][j]` is just "looking up" the answers you already wrote down when the loop was at Length 2 and 3.

**Complexity Check:**

- **Time:** $O(n^3)$ — Because we have three loops nested inside each other ($L$, $i$, $k$).
    
- **Space:** $O(n^2)$ — Because we need a table of size $n \times n$.