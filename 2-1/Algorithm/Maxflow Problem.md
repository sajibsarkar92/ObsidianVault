 # Ford-Fulkerson Method (Edmonds-Karp) Implementation Guide

## Overview

The Ford-Fulkerson method computes the **Maximum Flow** in a flow network. We will implement the **Edmonds-Karp** variation, which is the standard, practical way to code this using **BFS**.

- **The Goal:** Push as much "stuff" (water, data, traffic) as possible from a **Source ($s$)** to a **Sink ($t$)** through a network of pipes with limited capacities.
    
- **The Analogy:** Think of a **network of water pipes**.
    
    - Each pipe has a width (Capacity).
        
    - You turn the tap on full blast at the source.
        
    - The "Max Flow" is the total volume of water reaching the sink per second.
        
    - **Bottleneck:** The flow is always limited by the narrowest pipe in the path.
        

<iframe width="560" height="315" src="[https://www.youtube.com/embed/HuWebp_-84w?si=example](https://www.google.com/search?q=https://www.youtube.com/embed/HuWebp_-84w%3Fsi%3Dexample)" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

## Step-by-Step Implementation

### Step 1: Include Necessary Headers

**What:** Standard libraries for input/output, queues (for BFS), and algorithms.

**Why?**

- `queue`: Essential for the Breadth-First Search (BFS) used to find the shortest augmenting path (the Edmonds-Karp optimization).
    
- `vector`: To store the `parent` array for path reconstruction.
    
- `algorithm`: For `min()` to find the bottleneck.
    
   
C++

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <algorithm> // For min
#include <cstring>   // For memset (optional, but good for clearing arrays)

using namespace std;

#define V 6 // Number of vertices in our example graph
```

---

### Step 2: Define The "Residual Graph"

**What:** We often use an Adjacency Matrix `rGraph[V][V]` for beginner implementations.

**Why?**

- **The Residual Graph Concept:** This is the core of the algorithm. It doesn't just store how much capacity is _left_. It stores two things:
    
    1. **Forward Edge:** How much _more_ flow we can push (Capacity - Current Flow).
        
    2. **Backward Edge:** How much flow we can _undo_ (push back).
        
- Using a matrix makes updating these forward/backward edges incredibly simple ($O(1)$ lookup).
    

C++

```cpp
// rGraph[i][j] holds the "residual capacity" from i to j.
// Initially, this is just the capacity of the original graph.
int rGraph[V][V];
```

---

### Step 3: The Path Finder (BFS Helper)

**What:** A function that asks, "Is there _any_ path from Source to Sink with capacity > 0?"

**Why?**

- If we can find a path where every edge has room for more flow, we call this an **Augmenting Path**.
    
- We use a `parent` array to remember the path we took, so we can walk backwards from the Sink to the Source later.


C++

```cpp
// Returns true if there is a path from s to t in residual graph
bool bfs(int rGraph[V][V], int s, int t, int parent[]) {
    // Create a visited array and mark all vertices as not visited
    bool visited[V];
    memset(visited, 0, sizeof(visited));

    queue<int> q;
    q.push(s);
    visited[s] = true;
    parent[s] = -1;

    while (!q.empty()) {
        int u = q.front();
        q.pop();

        for (int v = 0; v < V; v++) {
            // Find neighbors with POSITIVE residual capacity
            if (!visited[v] && rGraph[u][v] > 0) {
                // If we found the sink, we are done!
                if (v == t) {
                    parent[v] = u;
                    return true;
                }
                // Otherwise, keep searching
                q.push(v);
                parent[v] = u;
                visited[v] = true;
            }
        }
    }
    return false; // No path found
}
```

---

### Step 4: The Main Loop (Pushing Flow)

**What:** Keep finding paths and filling them up until no paths remain.

**Why?**

- We start with `max_flow = 0`.
    
- We run the `while(bfs(...))` loop. As long as BFS returns `true`, it means there is still room to push more water through the network.
    

C++

```cpp
int fordFulkerson(int graph[V][V], int s, int t) {
    int u, v;

    // 1. Initialize Residual Graph with original capacities
    int rGraph[V][V]; 
    for (u = 0; u < V; u++)
        for (v = 0; v < V; v++)
             rGraph[u][v] = graph[u][v];

    int parent[V];  // To store the path found by BFS
    int max_flow = 0; 
    
    // While there is an augmenting path from source to sink...
    while (bfs(rGraph, s, t, parent)) {
        // (Logic continues below...)
    }
    return max_flow;
}
```

---

### Step 5: Find the Bottleneck (Path Flow)

**What:** Look at the path we just found and find the _smallest_ pipe capacity.

**Why?**

- A chain is only as strong as its weakest link. A path's flow is limited by the edge with the minimum residual capacity.
    
- We backtrack from Sink ($t$) using the `parent` array to find this minimum value.
    

C++

```cpp
        // Find minimum residual capacity of the edges along the
        // path filled by BFS.
        int path_flow = INT_MAX;
        
        // Walk backwards from Sink (t) to Source (s)
        for (v = t; v != s; v = parent[v]) {
            u = parent[v];
            path_flow = min(path_flow, rGraph[u][v]);
        }
```

---

### Step 6: Update Residual Capacities (The Magic)

**What:** Update the graph to reflect the flow we just pushed.

**Why?**

- **Forward Edge (`-=`):** We used up some capacity, so we subtract `path_flow`.
    
- **Backward Edge (`+=`):** This is the key to Ford-Fulkerson. We **add** capacity to the reverse edge ($v \to u$). This mathematically allows the algorithm to "undo" bad decisions later by pushing flow back the other way.
    

C++

```cpp
        // update residual capacities of the edges and reverse edges
        for (v = t; v != s; v = parent[v]) {
            u = parent[v];
            rGraph[u][v] -= path_flow; // Reduce capacity
            rGraph[v][u] += path_flow; // Add to reverse edge!
        }

        // Add path flow to overall flow
        max_flow += path_flow;
```

---

## Complete Implementation

C++

```cpp
#include <iostream>
#include <limits.h>
#include <queue>
#include <string.h>
#include <vector>
#include <algorithm>

using namespace std;

#define V 6 // Number of vertices in the graph

// BFS to find if there is a path from s to t in residual graph
// Also fills parent[] to store the path
bool bfs(int rGraph[V][V], int s, int t, int parent[]) {
    bool visited[V];
    memset(visited, 0, sizeof(visited));

    queue<int> q;
    q.push(s);
    visited[s] = true;
    parent[s] = -1;

    while (!q.empty()) {
        int u = q.front();
        q.pop();

        for (int v = 0; v < V; v++) {
            if (visited[v] == false && rGraph[u][v] > 0) {
                // If we find a connection to the sink node, 
                // there is no point in BFS anymore We just have 
                // to set its parent and can return true
                if (v == t) {
                    parent[v] = u;
                    return true;
                }
                q.push(v);
                parent[v] = u;
                visited[v] = true;
            }
        }
    }
    return false;
}

// Returns the maximum flow from s to t in the given graph
int fordFulkerson(int graph[V][V], int s, int t) {
    int u, v;

    // Create a residual graph and fill with initial capacities
    int rGraph[V][V]; 
    for (u = 0; u < V; u++)
        for (v = 0; v < V; v++)
             rGraph[u][v] = graph[u][v];

    int parent[V]; 
    int max_flow = 0; 

    // Augment the flow while there is a path from source to sink
    while (bfs(rGraph, s, t, parent)) {
        
        // Find minimum residual capacity of the edges along the path
        int path_flow = INT_MAX;
        for (v = t; v != s; v = parent[v]) {
            u = parent[v];
            path_flow = min(path_flow, rGraph[u][v]);
        }

        // update residual capacities of the edges and reverse edges
        for (v = t; v != s; v = parent[v]) {
            u = parent[v];
            rGraph[u][v] -= path_flow;
            rGraph[v][u] += path_flow;
        }

        max_flow += path_flow;
    }

    return max_flow;
}

int main() {
    // Graph creation: 6 nodes
    // adjacency matrix representation
    int graph[V][V] = { 
        {0, 16, 13, 0, 0, 0},
        {0, 0, 10, 12, 0, 0},
        {0, 4, 0, 0, 14, 0},
        {0, 0, 9, 0, 0, 20},
        {0, 0, 0, 7, 0, 4},
        {0, 0, 0, 0, 0, 0}
    };

    cout << "The maximum possible flow is " 
         << fordFulkerson(graph, 0, 5);

    return 0;
}
```

---

## Time & Space Complexity

### Time: $O(V E^2)$ (Edmonds-Karp)

- **Why?** Using BFS (Edmonds-Karp) guarantees that we find the **shortest** augmenting path (in terms of number of edges).
    
- Standard Ford-Fulkerson (using DFS) depends on the max flow value ($O(E \cdot f)$), which can be very slow if capacities are huge numbers.
    
- BFS limits the number of augmentations to $O(VE)$, and each BFS takes $O(E)$, resulting in $O(VE^2)$.
    

### Space: $O(V^2)$

- **Why?** We are using a $V \times V$ adjacency matrix for the residual graph.
    
- _Note:_ If you use an adjacency list, space drops to $O(V+E)$, but handling reverse edge pointers becomes slightly more complex code-wise.
    

---

## The Concept of "Residuals"

Understanding the Residual Graph is the hardest part. Here is the breakdown:

|**Graph Type**|**What the Edge Value Means**|**Visual**|
|---|---|---|
|**Original Graph**|"The pipe is this wide." (Total Capacity)|$C(u,v) = 10$|
|**Residual Graph**|"You can push X more units this way."|$rGraph[u][v] = 10$|
|**Residual (Reverse)**|"You can CANCEL X units of flow."|$rGraph[v][u] = 0$ (initially)|

**When we push 4 units of flow from $u \to v$:**

1. **Forward:** $10 - 4 = 6$ (Room for 6 more).
    
2. **Reverse:** $0 + 4 = 4$ (We can now push 4 units back from $v \to u$ to "undo" this flow if needed later).
    

Would you like a guide on **Dinic's Algorithm** next, which optimizes this process even further for massive graphs?