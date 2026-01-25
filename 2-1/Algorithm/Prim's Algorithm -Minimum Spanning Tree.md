

# Prim's Algorithm Implementation Guide

## Overview

Prim's Algorithm finds the **Minimum Spanning Tree (MST)** of a connected, undirected graph.

- **The Goal:** Connect _all_ the nodes together using the minimum total edge weight, without forming any loops (cycles).
    
- **The Analogy:** Think of Prim's like **mold growing on a slice of bread**. It starts at one spot (source) and grows outward. It always expands to the closest (tastiest) crumb next, ensuring the colony stays connected as one big piece.
    
<iframe width="560" height="315" src="https://www.youtube.com/embed/4ZlRH0eK-qQ?si=9PKDc91WSwAQSKXZ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

## Step-by-Step Implementation

### Step 1: Include Necessary Headers

**What:** We import the standard libraries.

**Why?**

- `vector`: To store our graph (Adjacency List) and the `visited` array.
    
- `queue`: Specifically for the `priority_queue`. Prim's is a "Greedy" algorithm—it always wants the _cheapest_ edge right now. A Min-Priority Queue gives us the smallest number instantly ($O(1)$ access).
    
- `functional`: For the `greater` comparator (to make the queue a Min-Heap).
    

C++

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <functional> // For greater<pii>
using namespace std;
```

---

### Step 2: Define Graph Structure

**What:** We use an **Adjacency List**.

**Why?**

- Prim's algorithm crawls from node to neighbor. An adjacency matrix ($N \times N$) is wasteful because we only care about who is _connected_ to the current node.
    
- We use a `pair<int, int>` to store `{weight, neighbor}`.
    
    - **Crucial Detail:** In the Priority Queue, we put **weight first**. C++ sorts pairs by the first element. We want to sort by _cost_, not by the node index.
        

C++

```cpp
// {weight, node_index} - Weight comes first for sorting!
using pii = pair<int, int>; 

// adj[u] contains list of {weight, v}
vector<vector<pii>> adj; 
```

---

### Step 3: Initialization

**What:** Set up the "Frontier" (PQ) and the "Done" list (`visited`).

**Why?**

- **`priority_queue`**: Acts as the border of our growing tree. It holds all the potential bridges we _could_ build next.
    
- **`visited` array**: This is critical. Once a node joins the MST, we mark it `true`. If we try to add a visited node again, we would create a cycle (a loop), which is forbidden in a Tree.
    
- **Start Node:** We arbitrarily pick Node 0. It costs `0` to start there.
    

C++

```c++
int n = 5; // Example: 5 nodes
vector<bool> visited(n, false); // Tracks who is in the MST
int mst_cost = 0; // Total cost of the tree

// Min-Heap: smallest weight pops out first
priority_queue<pii, vector<pii>, greater<pii>> pq;

// Start at Node 0 with cost 0
pq.push({0, 0}); 
```

---

### Step 4: The Main Loop (The Growth Cycle)

**What:** Keep growing until we run out of options.

**Why?** As long as the `pq` is not empty, it means there are still edges connecting our tree to the "unknown" world. We must process them.

C++

```cpp
while (!pq.empty()) {
```

---

### Step 5: The Greedy Choice (Select and Inspect)

**What:** Pick the edge with the absolute lowest weight from the queue.

**Why?** This is the "Greedy" part. By picking the cheapest connection available _anywhere_ on the frontier, we guarantee the total cost remains minimal.

- **Extraction:** We split the pair back into `weight` and `u` (the node).
    

C++

```cpp
    int weight = pq.top().first;
    int u = pq.top().second;
    pq.pop();
```

---

### Step 6: The "Cycle" Check (Crucial!)

**What:** Check if node `u` is already part of the MST.

**Why?** This is the **most common bug** source.

- Since we push multiple edges to the same node into the PQ, we might pop a node that we _already visited_ via a cheaper path earlier.
    
- If `visited[u]` is true, adding this edge would connect the tree to itself, forming a loop. We **must** skip it.
    

C++

```c++
    // If we've already included this node, ignore this edge
    if (visited[u]) {
        continue;
    }

    // Otherwise, officially add it to the MST
    visited[u] = true;
    mst_cost += weight;
```

---

### Step 7: Explore and Expand (Push Neighbors)

**What:** Look at all neighbors of the newly added node `u`.

**Why?** Now that `u` is part of our territory, all of `u`'s connections are now valid candidates for future expansion.

- **Condition:** We only push neighbors that are **not** yet visited. If a neighbor is already in the tree, we don't need a bridge to them.
    
- **Note:** We push `{edge_weight, neighbor}`.
    

C++

```cpp
    for (auto& edge : adj[u]) {
        int w = edge.first; // weight
        int v = edge.second; // neighbor
        
        // Only consider neighbors not yet in the MST
        if (!visited[v]) {
            pq.push({w, v});
        }
    }
}
```

---

## Complete Implementation

C++

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <functional>

using namespace std;

// Typedef for {weight, node}
using pii = pair<int, int>;

int primMST(int n, const vector<vector<pii>>& adj) {
    // Step 3: Initialize Structures
    int mst_cost = 0;
    priority_queue<pii, vector<pii>, greater<pii>> pq;
    vector<bool> visited(n, false);

    // Start with Node 0 (Cost 0)
    pq.push({0, 0});
    
    // Step 4: Main Loop
    while (!pq.empty()) {
        // Step 5: Greedy Selection
        int weight = pq.top().first;
        int u = pq.top().second;
        pq.pop();

        // Step 6: Cycle Check
        if (visited[u]) {
            continue; // Node u is already in the MST
        }

        // Add node to MST
        visited[u] = true;
        mst_cost += weight;

        // Step 7: Expand to Neighbors
        for (auto& edge : adj[u]) {
            int w = edge.first;  // Weight of edge
            int v = edge.second; // Neighbor node
            
            // If neighbor v is not in MST, add edge to frontier
            if (!visited[v]) {
                pq.push({w, v});
            }
        }
    }
    
    return mst_cost;
}

int main() {
    int n = 4;
    vector<vector<pii>> adj(n);

    // Graph Construction (Undirected)
    // 0 --(1)--> 1
    // 0 --(3)--> 2
    // 1 --(1)--> 2
    // 2 --(4)--> 3
    
    // Add edges: {weight, neighbor}
    // Node 0
    adj[0].push_back({1, 1});
    adj[0].push_back({3, 2});

    // Node 1
    adj[1].push_back({1, 0});
    adj[1].push_back({1, 2}); 

    // Node 2
    adj[2].push_back({3, 0});
    adj[2].push_back({1, 1});
    adj[2].push_back({4, 3});

    // Node 3
    adj[3].push_back({4, 2});

    cout << "Minimum Cost of MST: " << primMST(n, adj) << endl;
    return 0;
}
```

---

## Time & Space Complexity

### Time: $O(E \log V)$

- **Why?** In the worst case, we might push every edge into the priority queue ($E$). Each push/pop operation on a heap takes logarithmic time ($\log$).
    
- So, for $E$ edges, it takes $E \times \log(\text{size of queue}) \approx E \log V$.
    

### Space: $O(V + E)$

- **Why?** We store the graph ($V+E$) and the priority queue/visited array ($V$).
    

---

## Prim's vs. Dijkstra (The Confusion)

They look almost identical code-wise. What is the difference?

| **Feature**  | **Prim's Algorithm (MST)**           | **Dijkstra's Algorithm (Shortest Path)**            |
| ------------ | ------------------------------------ | --------------------------------------------------- |
| **Goal**     | Connect everyone cheaply.            | Find fastest route from A to B.                     |
| **PQ Value** | Cost of **just the new edge** ($w$). | Cost of **total path from source** ($dist[u] + w$). |
| **Update**   | `pq.push({w, v})`                    | `pq.push({dist[u] + w, v})`                         |
| **Concept**  | "How much to bridge the gap?"        | "How far is it total?"                              |