

# Floyd-Warshall Algorithm Implementation Guide

## Overview

The Floyd-Warshall algorithm finds the shortest paths between **all pairs** of vertices in a weighted graph. Unlike Dijkstra (which is like finding the best route from _your_ house to everywhere else), Floyd-Warshall is like generating a complete distance table for an entire country, showing the distance from _every_ city to _every other_ city.

<iframe width="560" height="315" src="https://www.youtube.com/embed/oNI0rf2P9gE?si=YI3nNRKvQw_zHL79" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## Step-by-Step Implementation

### Step 1: Include Necessary Headers

**Why?**

We need `vector` to create our 2D distance matrix (our "Distance Table"). We need `limits` to define "Infinity" because initially, we assume unconnected cities are unreachable.

- **Intuition:** Notice we **don't** need `<queue>` or `<stack>`. Floyd-Warshall doesn't explore the graph by "crawling" (like BFS/DFS). Instead, it uses **Dynamic Programming** to update a static table repeatedly. It’s less like a traveller exploring a map and more like a mathematician refining a spreadsheet.
    

C++

```cpp
#include <vector>
#include <limits>
#include <algorithm> // for std::min
#include <iostream>
using namespace std;
```

---

### Step 2: Initialize the Distance Matrix

**Why?**

Since we need the distance between _every_ pair $(i, j)$, an **Adjacency Matrix** ($N \times N$) is the only structure that makes sense.

- **The "Zero" Logic:** `dist[i][i] = 0`. The distance from you to yourself is always zero.
    
- **The "Infinity" Logic:** `dist[i][j] = INF`. If there is no direct road between city $i$ and city $j$, we assume (for now) that we can't get there.
    
- **The "Direct Road" Logic:** If the input says there is an edge from $A$ to $B$ with weight 5, we fill that in immediately. This represents our "Base Knowledge" before we start finding shortcuts.
    

C++

```cpp
int n; // number of vertices
long long INF = numeric_limits<long long>::max() / 2; // "Safe" Infinity to prevent overflow
vector<vector<long long>> dist(n, vector<long long>(n, INF));

// Set diagonal to 0 (distance to self is 0)
for (int i = 0; i < n; i++) {
    dist[i][i] = 0;
}

// Load initial edge weights (assuming 'edges' is your input list)
// for (auto e : edges) dist[e.u][e.v] = e.weight;
```

---

### Step 3: The Intermediate "Phase" Loop (The 'k' Loop)

**Deep Intuition (Crucial):**

This is the most important part of the algorithm and the hardest to understand. Why is `k` on the outside?

- **The "Permission" Analogy:** Imagine all nodes are locked doors. You cannot pass through them.
    
    - **Iteration `k=0`:** You "unlock" Node 0. Now, you check every path in the world. If a path from $A \to B$ is faster by going through Node 0 ($A \to 0 \to B$), you update your map.
        
    - **Iteration `k=1`:** You "unlock" Node 1. Now paths can go through Node 0, Node 1, or both!
        
    - **Iteration `k=5`:** You have unlocked nodes 0 through 5. You can form complex paths like $A \to 2 \to 5 \to 0 \to B$.
        
- **Summary:** We are building the solution layer-by-layer. We solve the problem for "paths using only node 0," then "paths using nodes 0 and 1," until eventually, we allow **all** nodes to be used as shortcuts.
    

C++

```cpp
// k represents the "intermediate node" we are currently 'unlocking'
for (int k = 0; k < n; k++) {
```

---

### Step 4: The Source and Destination Loops ('i' and 'j')

**Why?**


Once we have "unlocked" node `k` (Step 3), we need to check if this new node helps _anyone_.

- **The Broadcaster:** We iterate through every possible starting city `i` and every possible destination city `j`.
    
- **The Question:** We ask every pair $(i, j)$: "Hey, now that node `k` is open for business, is it faster for you to go through `k` instead of your current route?"
    

C++

```cpp
    // i represents the source vertex
    for (int i = 0; i < n; i++) {
        // j represents the destination vertex
        for (int j = 0; j < n; j++) {
```

---

### Step 5: Check for Reachability

**Why?**

You cannot use a shortcut if the road to the shortcut doesn't exist.

- **The Logic:** If the distance from $i \to k$ is Infinity, you can't get to the bridge. If the distance from $k \to j$ is Infinity, you can't get off the bridge.
    
- **Safety:** In C++, if you add `INF + 5`, it might overflow into a negative number, causing huge bugs. This check prevents that math error.
    

C++

```cpp
            // If the "bridge" k is unreachable from i, or j is unreachable from k...
            if (dist[i][k] == INF || dist[k][j] == INF)
                continue;
```

---

### Step 6: The Relaxation Logic (The "Triangle" Check)

**Why?**

This is the core decision. We are comparing two realities:

1. **Current Reality (`dist[i][j]`):** The best path we found so far (which might use nodes $0...k-1$ as intermediates).
    
2. **New Potential Reality (`dist[i][k] + dist[k][j]`):** The path that specifically detours through our newly unlocked node `k`.
    

If the Detour is cheaper, we overwrite the old reality. This is effectively checking the **Triangle Inequality**: is the sum of two sides ($i \to k$ and $k \to j$) smaller than the third side ($i \to j$)?

C++

```cpp
            // Update dist[i][j] if the path through k is shorter
            if (dist[i][k] + dist[k][j] < dist[i][j]) {
                dist[i][j] = dist[i][k] + dist[k][j];
            }
        }
    }
}
```

---

### Step 7: Negative Cycle Detection (Optional)

**Why?**

- **Normal World:** The distance from your house to your house is 0. You are already there.
    
- **Negative Cycle World:** Imagine a path $A \to B \to A$ where the total cost is $-5$. If you take this path, you end up back at $A$ with a "cost" of $-5$. Do it again, and you have $-10$.
    
- **Detection:** If, after running the algorithm, `dist[i][i] < 0`, it means there is a "Time Machine" loop (negative cycle) involving node `i` that allows you to reduce costs infinitely.
    

C++

```cpp
for (int i = 0; i < n; i++) {
    if (dist[i][i] < 0) {
        // Negative cycle detected!
        return; 
    }
}
```

---

## Complete Implementation

C++

```cpp
#include <vector>
#include <iostream>
#include <algorithm>

using namespace std;

// We use a large number for Infinity, but not so large that adding two of them overflows
const long long INF = 1e18; 

void floydWarshall(int n, const vector<vector<int>>& edges) {
    // Step 2: Initialize Matrix
    vector<vector<long long>> dist(n, vector<long long>(n, INF));

    // Set self-distance to 0
    for (int i = 0; i < n; i++) dist[i][i] = 0;

    // Add initial known edges (Direct paths)
    for (const auto& edge : edges) {
        int u = edge[0]; // From
        int v = edge[1]; // To
        int w = edge[2]; // Weight
        dist[u][v] = w; 
        
        // Note: If undirected graph, also add: dist[v][u] = w;
    }

    // Step 3: k Loop (The "Phase" - unlocking intermediate nodes)
    for (int k = 0; k < n; k++) {
        // Step 4: i Loop (The Source)
        for (int i = 0; i < n; i++) {
            // Step 4: j Loop (The Destination)
            for (int j = 0; j < n; j++) {
                
                // Step 5: Reachability Check (Don't bridge across infinity)
                if (dist[i][k] == INF || dist[k][j] == INF) 
                    continue;

                // Step 6: Relaxation (Is the detour through k cheaper?)
                if (dist[i][k] + dist[k][j] < dist[i][j]) {
                    dist[i][j] = dist[i][k] + dist[k][j];
                }
            }
        }
    }

    // Output Result
    cout << "Shortest Distance Matrix:" << endl;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            if (dist[i][j] == INF) cout << "INF\t";
            else cout << dist[i][j] << "\t";
        }
        cout << endl;
    }
}
```

---

## Time & Space Complexity

### Time: $O(V^3)$

- **Intuition:** We have three nested loops (`k`, `i`, `j`). Each runs `V` times (where V is the number of vertices).
    
- **Implication:** If $V=10$, operations = 1,000. If $V=100$, operations = 1,000,000. If $V=1000$, operations = 1,000,000,000 (too slow for 1 second limit!).
    
- **Rule of Thumb:** Only use this if $V \le 400$.
    

### Space: $O(V^2)$

- We store a 2D matrix.
    

---

## Algorithm Comparison

|**Feature**|**Floyd-Warshall**|**Bellman-Ford**|**Dijkstra**|
|---|---|---|---|
|**Goal**|**All** Pairs (Whole Map)|**Single** Source (One Origin)|**Single** Source (One Origin)|
|**Concept**|Dynamic Programming (Matrix)|Brute Force Edge Relaxation|Greedy Priority Queue|
|**Negative Edges**|✅ Works|✅ Works|❌ Fails|
|**Key Loop**|`for(k) for(i) for(j)`|`for(V-1) for(all edges)`|`while(!pq.empty())`|
|**Best For**|Small, dense graphs ($V < 400$)|Negative weights exist|Standard navigation (Google Maps)|

---

## Key Intuitions to Remember

1. **The "k" Loop Order:** If you put the `k` loop inside `i` or `j`, the algorithm breaks. You **must** finish "Phase k" (unlocking node k) before you can move to "Phase k+1". You cannot use a bridge before it is built!
    
2. **Transitive Property:** The algorithm essentially says: _"If I can get from A to B, and B to C, I can definitely get from A to C."_ It propagates this truth until the shortest paths stabilize.
    
3. **Dense Graphs:** This algorithm is actually _faster_ than Dijkstra if the graph is "Dense" (meaning almost every city is connected to every other city), because it doesn't have the overhead of a Priority Queue.