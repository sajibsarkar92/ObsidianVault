Here are the enhanced, detailed notes for the **Rock Climbing / Grid Path** family of problems.

---

# 🧗 Topic: Rock Climbing & Grid Path Problems

## 1. The Base Problem: Minimum Path Cost (Rock Climbing)

### **Problem Statement**

You are given a 2D grid of size $R \times C$ representing a cliff. Each cell `grid[i][j]` has a value representing the **cost** (difficulty) to step on that rock.

- **Goal:** Find the minimum total cost to climb from the **Bottom Row** (any cell) to the **Top Row** (any cell).
    
- **Movement:** From a cell `(i, j)`, you can climb up to:
    
    1. Vertical Up: `(i-1, j)`
        
    2. Diagonal Up-Left: `(i-1, j-1)`
        
    3. Diagonal Up-Right: `(i-1, j+1)`
        

### **Intuition**

This is a classic "Shortest Path" problem on a DAG (Directed Acyclic Graph). A "Greedy" approach (always picking the cheapest next step) fails because a cheap step now might lead you into a patch of extremely expensive rocks later.

We must use DP. Since we want to reach the **Top**, we can build our solution from the **Bottom** up (or Top down).

- **Crucial Logic:** The cost to reach a cell `(i, j)` depends entirely on the minimum cost to reach the valid cells _below_ it.
    

### **Recurrence Relation**

Let `dp[i][j]` be the minimum cost to reach the top row starting from `(i, j)`.

$$dp[i][j] = \text{grid}[i][j] + \min(dp[i+1][j], dp[i+1][j-1], dp[i+1][j+1])$$

_(Note: Boundary checks needed for `j-1` and `j+1`)_.

### **C++ Implementation**

C++

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <climits>

using namespace std;

int rockClimbingMinCost(vector<vector<int>>& grid) {
    int R = grid.size();
    int C = grid[0].size();
    
    // We can modify the grid in-place to store DP values to save O(R*C) space.
    // Or use a separate DP table: vector<vector<int>> dp = grid;
    
    // Iterate from the SECOND to LAST row (R-2) up to the TOP (0)
    // The last row (R-1) is the base case: cost is just the cell value itself.
    for (int i = R - 2; i >= 0; i--) {
        for (int j = 0; j < C; j++) {
            // 1. Move straight down
            int min_prev = grid[i + 1][j];
            
            // 2. Move diagonal down-left (check bounds)
            if (j > 0) {
                min_prev = min(min_prev, grid[i + 1][j - 1]);
            }
            
            // 3. Move diagonal down-right (check bounds)
            if (j < C - 1) {
                min_prev = min(min_prev, grid[i + 1][j + 1]);
            }
            
            // Add current cell's cost to the best path from below
            grid[i][j] += min_prev;
        }
    }

    // The answer is the minimum value found anywhere in the Top Row (Row 0)
    int ans = INT_MAX;
    for (int j = 0; j < C; j++) {
        ans = min(ans, grid[0][j]);
    }
    return ans;
}
```

---

## 2. Variant: Maximum Gold (Gold Mine Problem)

### **The Variant**

Instead of minimizing effort, we want to **maximize profit**.

- **Movement:** Usually defined as moving **Right**, **Right-Up**, or **Right-Down** (starting from the Left column to the Right column).
    
- **Goal:** Collect max gold.
    

### **Logic Shift**

1. **Direction:** Logic flows from **Right to Left**. To arrive at col `j`, you came from `j-1`. But usually, implementation is easier **Right-to-Left** (calculating max gold _starting_ from `(i,j)` to the end).
    
2. **Objective:** Use `max()` instead of `min()`.
    
3. **Boundaries:** Invalid cells (outside grid) effectively yield `0` gold (or `-Infinity` if negative values exist).
    

### **C++ Implementation**

C++

```
int maxGold(vector<vector<int>>& mine) {
    int R = mine.size();
    int C = mine[0].size();

    // Iterate from second-to-last column backwards to the first column
    for (int j = C - 2; j >= 0; j--) {
        for (int i = 0; i < R; i++) {
            // Option 1: Move Right
            int right = mine[i][j + 1];

            // Option 2: Move Right-Up
            int right_up = (i > 0) ? mine[i - 1][j + 1] : 0;

            // Option 3: Move Right-Down
            int right_down = (i < R - 1) ? mine[i + 1][j + 1] : 0;

            // Store max gold collectable starting from here
            mine[i][j] += max({right, right_up, right_down});
        }
    }

    // The answer is the max value in the first column
    int max_gold = 0;
    for (int i = 0; i < R; i++) max_gold = max(max_gold, mine[i][0]);
    
    return max_gold;
}
```

---

## 3. Variant: Unique Paths (Counting)

### **The Variant**

A robot is at `(0, 0)` and needs to reach `(R-1, C-1)`.

- **Constraint:** Robot can only move **Down** or **Right**.
    
- **Goal:** Find the total number of unique paths to the destination.
    

### **Intuition**

To reach cell `(i, j)`, the robot _must_ have come from either:

1. The cell immediately **Above** `(i-1, j)`.
    
2. The cell immediately to the **Left** `(i, j-1)`.
    
    Therefore, `Paths(i, j) = Paths(Above) + Paths(Left)`.
    

### **C++ Implementation**

C++

```
int uniquePaths(int m, int n) {
    // dp[i][j] stores number of ways to reach cell (i, j)
    vector<vector<int>> dp(m, vector<int>(n, 0));

    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            // Base Case: Top-Left corner
            if (i == 0 && j == 0) {
                dp[i][j] = 1;
            } 
            else {
                int fromTop = (i > 0) ? dp[i - 1][j] : 0;
                int fromLeft = (j > 0) ? dp[i][j - 1] : 0;
                dp[i][j] = fromTop + fromLeft;
            }
        }
    }
    return dp[m - 1][n - 1];
}
```

---

## 4. Variant: Unique Paths with Obstacles

### **The Variant**

The grid contains obstacles (marked as `1`). Empty space is `0`.

- **Rule:** You cannot step on an obstacle.
    
- **Goal:** Count paths from `(0,0)` to `(R-1, C-1)`.
    

### **Logic Shift**

If a cell `(i, j)` contains an obstacle, it is impossible to be at that cell.

- `dp[i][j] = 0` (Dead end).
    
- This `0` will naturally propagate: any future cell relying on this obstacle cell will add `0` from this direction.
    

### **C++ Implementation**

C++

```cpp
#include <vector>
using namespace std;

int uniquePathsWithObstacles(vector<vector<int>>& obstacleGrid, 
                             int srcRow, int srcCol, 
                             int dstRow, int dstCol) {
    int m = obstacleGrid.size();
    int n = obstacleGrid[0].size();
    
    // Validate coordinates are within bounds
    if (srcRow < 0 || srcRow >= m || srcCol < 0 || srcCol >= n ||
        dstRow < 0 || dstRow >= m || dstCol < 0 || dstCol >= n) {
        return 0;
    }
    
    // If source or destination is an obstacle, impossible
    if (obstacleGrid[srcRow][srcCol] == 1 || obstacleGrid[dstRow][dstCol] == 1) {
        return 0;
    }
    
    // If source equals destination, one trivial path (stay there)
    if (srcRow == dstRow && srcCol == dstCol) {
        return 1;
    }

    // Initialize DP grid with 0s
    vector<vector<int>> dp(m, vector<int>(n, 0));
    dp[srcRow][srcCol] = 1; // Start point

    // Iterate through grid, ensuring we process cells in order from source to destination
    // We need to handle the case where destination is "above" or "left" of source
    
    // Determine iteration direction
    int rowStart = min(srcRow, dstRow);
    int rowEnd = max(srcRow, dstRow);
    int colStart = min(srcCol, dstCol);
    int colEnd = max(srcCol, dstCol);
    
    // Only iterate within the bounding box from source to destination
    // This optimization avoids unnecessary computation
    for (int i = rowStart; i <= rowEnd; i++) {
        for (int j = colStart; j <= colEnd; j++) {
            // Skip source (already set) and obstacles
            if ((i == srcRow && j == srcCol) || obstacleGrid[i][j] == 1) {
                continue;
            }
            
            // Only consider valid incoming directions based on relative positions
            // If destination is below source, we can come from top
            if (i > srcRow && i > 0) {
                dp[i][j] += dp[i - 1][j];
            }
            // If destination is above source, we can come from bottom  
            if (i < srcRow && i < m - 1) {
                dp[i][j] += dp[i + 1][j];
            }
            // If destination is to the right of source, we can come from left
            if (j > srcCol && j > 0) {
                dp[i][j] += dp[i][j - 1];
            }
            // If destination is to the left of source, we can come from right
            if (j < srcCol && j < n - 1) {
                dp[i][j] += dp[i][j + 1];
            }
        }
    }
    
    return dp[dstRow][dstCol];
}
```

---

## 5. Variant: Minimum Path Sum (Triangle)

### **The Variant**

Given a triangle array, find the minimum path sum from top to bottom.

```
   2
  3 4
 6 5 7
```

From row `i` index `j`, you can move to `(i+1, j)` or `(i+1, j+1)`.

### **Intuition**

This is easier than rectangular grids because we don't need boundary checks (every element has exactly 2 children below it).

We work **Bottom-Up**.

- For the `3` in row 1, we look at `6` and `5` below it. The best path through `3` is `3 + min(6, 5) = 8`.
    
- For the `4` in row 1, we look at `5` and `7` below it. The best path through `4` is `4 + min(5, 7) = 9`.
    
- Finally, for the top `2`, we take `2 + min(8, 9) = 10`.
    

### **C++ Implementation**

C++

```
int minimumTotal(vector<vector<int>>& triangle) {
    int n = triangle.size();
    
    // Start from the second to last row (n-2)
    // Move upwards to the tip (0)
    for (int i = n - 2; i >= 0; i--) {
        for (int j = 0; j < triangle[i].size(); j++) {
            // Update current cell with min of its two children
            triangle[i][j] += min(triangle[i + 1][j], triangle[i + 1][j + 1]);
        }
    }
    // The tip of the triangle now holds the total min path sum
    return triangle[0][0];
}
```

---

## 6. Variant: Dungeon Game (Minimum HP)

### **The Variant**

This is a "hard" variant.

- **Scenario:** You start at `(0,0)` and move to `(M,N)`. Grid cells add HP (positive) or remove HP (negative).
    
- **Constraint:** Your HP must **never drop to 0 or less** at any moment.
    
- **Goal:** Find the minimum initial HP you need to start with.
    

### **Logic Shift (Forward vs Backward)**

- **Why Forward DP Fails:** If you maximize HP going forward, you might choose a path that gives you 1000 HP but then hits a -2000 demon. If you minimize HP loss, you might die early. The "best" state at `(i,j)` isn't just a number; it depends on the _future_.
    
- **Solution: Backward DP.** We start from the Princess (End). We ask: "If I am at the end, what HP do I need?" (Answer: 1). Then we step back.
    
- **Equation:** `Need = min(Next_Right_Need, Next_Down_Need) - current_orb_or_demon`.
    
    - If `Need` becomes $\le 0$, it means the current cell provided enough healing. We cap it at `1` (minimum alive state).
        

### **C++ Implementation**

C++

```
int calculateMinimumHP(vector<vector<int>>& dungeon) {
    int m = dungeon.size();
    int n = dungeon[0].size();
    
    // dp[i][j] = Min HP needed AT start of cell (i,j)
    vector<vector<int>> dp(m + 1, vector<int>(n + 1, INT_MAX));

    // Base Case: To survive AFTER the princess cell, we need 1 HP
    dp[m][n - 1] = 1;
    dp[m - 1][n] = 1;

    // Iterate backwards
    for (int i = m - 1; i >= 0; i--) {
        for (int j = n - 1; j >= 0; j--) {
            // Look at right and down neighbors
            int minNeedNext = min(dp[i + 1][j], dp[i][j + 1]);
            
            // Calculate what we need HERE
            // If next needs 5 HP, and current cell is -3 (demon), we need 5 - (-3) = 8 HP.
            // If next needs 5 HP, and current cell is +10 (orb), we need 5 - 10 = -5 HP.
            int need = minNeedNext - dungeon[i][j];
            
            // Constraint: HP cannot be 0 or negative. Min is 1.
            dp[i][j] = max(1, need);
        }
    }
    return dp[0][0];
}
```

---

## 7. Variant: Cherry Pickup (Two Simultaneous Paths)

### **The Variant**

- **Scenario:** Go from `(0,0)` to `(N,N)` collecting cherries. Then go **back** from `(N,N)` to `(0,0)` collecting more.
    
- **Twist:** Once you pick a cherry, the cell becomes empty (0). You cannot pick it twice.
    
- **Goal:** Maximize total cherries.
    

### **Intuition**

Going down then back up is equivalent to **two people** starting at `(0,0)` and going to `(N,N)` simultaneously.

- **State:** `dp[r1][c1][r2]`.
    
    - Person 1 is at `(r1, c1)`.
        
    - Person 2 is at `(r2, c2)`.
        
    - Note: Since both take `k` steps, `r1 + c1 = r2 + c2`. We can calculate `c2` as `r1 + c1 - r2`. We don't need a 4th dimension.
        
- **Transition:** We try 4 combinations of moves: (Down, Down), (Down, Right), (Right, Down), (Right, Right).
    
- **Double Counting:** If `r1 == r2` (and thus `c1 == c2`), they are on the **same cell**. We add the cherry count only **once**.
    

### **C++ Implementation (Core Logic)**

C++

```
// 3D Memoization Table
int memo[50][50][50];
int N;

int solve(int r1, int c1, int r2, vector<vector<int>>& grid) {
    int c2 = r1 + c1 - r2; // Derived coordinate
    
    // 1. Boundary / Obstacle Checks
    if (r1 >= N || r2 >= N || c1 >= N || c2 >= N || 
        grid[r1][c1] == -1 || grid[r2][c2] == -1) 
        return -1e9; // Impossible path
    
    // 2. Base Case: Reached Destination
    if (r1 == N - 1 && c1 == N - 1) return grid[r1][c1];

    if (memo[r1][c1][r2] != -1) return memo[r1][c1][r2];

    // 3. Collect Cherries
    int cherries = grid[r1][c1];
    // If they are NOT on the same cell, add second person's cherries
    if (r1 != r2) cherries += grid[r2][c2];

    // 4. Explore 4 simultaneous moves
    // DD (Down, Down), DR, RD, RR
    int maxNext = max({
        solve(r1 + 1, c1, r2 + 1, grid), // Both Down
        solve(r1 + 1, c1, r2, grid),     // Down, Right
        solve(r1, c1 + 1, r2 + 1, grid), // Right, Down
        solve(r1, c1 + 1, r2, grid)      // Both Right
    });

    memo[r1][c1][r2] = cherries + maxNext;
    return memo[r1][c1][r2];
}
```