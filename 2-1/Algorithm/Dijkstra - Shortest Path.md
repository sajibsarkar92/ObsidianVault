
# Dijkstra’s Algorithm Implementation Guide

## Overview

Dijkstra's algorithm finds the shortest path from a source vertex to all other vertices in a graph with **non-negative** edge weights. Unlike Bellman-Ford’s brute-force approach, Dijkstra uses a "Greedy" strategy—it always processes the closest unvisited vertex next, making it much faster for standard pathfinding.

## Step-by-Step Implementation

### Step 1: Include Necessary Headers

**Why?** We need `vector` for the graph, `limits` for infinity, and critically, `queue` for the **Priority Queue**. Dijkstra relies on efficiently finding the "minimum" distance node instantly. A standard queue (FIFO) won't work; we need a Min-Heap (Priority Queue) to always give us the smallest element in $O(\log n)$ time.

C++

```cpp
#include <vector>
#include <queue>   // Required for priority_queue
#include <limits>
#include <utility> // For pair
using namespace std;
```

### Step 2: Define the Graph Structure (Adjacency List)

**Why?** Bellman-Ford iterates through a simple list of _all_ edges because it processes everything blindly. Dijkstra, however, explores "radially" (like a ripple). When we are at a specific node, we only need to know _its_ direct neighbors. An **Adjacency List** is perfect for this—it lets us jump straight to the connected neighbors without scanning the whole graph.

C++

```cpp
// We use pairs to store {neighbor, weight}
using pii = pair<int, int>; 

// adj[u] contains a list of pairs representing edges from u
vector<vector<pii>> adj; 
```

### Step 3: Initialize Distance Array and Priority Queue

**Why?** We set all distances to Infinity because we haven't explored the map yet. We set the `source` distance to 0. Then, we push the source into the Priority Queue.

- **Crucial Detail:** The Priority Queue stores `{current_distance, node}`. We put distance _first_ so the queue automatically sorts by distance (smallest first).
    

C++

```cpp
int n; // number of vertices
vector<int> dist(n, numeric_limits<int>::max());
dist[source] = 0;

// Min-Priority Queue: stores {distance, node}
// 'greater' makes it a Min-Heap (smallest element on top)
priority_queue<pii, vector<pii>, greater<pii>> pq;

pq.push({0, source});
```

### Step 4: Main Loop (While Queue is Not Empty)

**Why?** This loop represents the "Frontier" of our exploration. As long as there are nodes in the queue, it means there are still paths we are actively exploring or improving. When the queue is empty, we have visited every reachable node.

C++

```cpp
while (!pq.empty()) {
```

### Step 5: The "Greedy" Selection

**Why?** We extract the node with the **smallest** distance from the queue. This is the core "Greedy" logic: we assume that because this node is the closest one we know of, we have found its optimal path (assuming non-negative weights).

C++

```cpp
    int d = pq.top().first;  // The distance to reach this node
    int u = pq.top().second; // The node index
    pq.pop();
```

### Step 6: The "Stale Entry" Check (Lazy Deletion)

Why? This is unique to C++. We cannot easily update a value inside a priority_queue. If we find a shorter path to a node that is already in the queue, we just push the new, better pair in. This means the queue might hold "old" (worse) versions of the same node.

We check: "Is the distance I just popped (d) worse than the best distance I already have stored (dist[u])?" If yes, it's garbage—throw it away.

C++

```cpp
    if (d > dist[u]) {
        continue; // Skip outdated/stale entries
    }
```

### Step 7: Explore and Relax Neighbors

**Why?** Now we look at neighbors of `u`. We calculate the cost to reach neighbor `v` _through_ `u`. If this new path is shorter than the old known path (`dist[v]`), we update it (Relaxation) and push the new path to the queue.

C++

```cpp
    for (auto& edge : adj[u]) {
        int v = edge.first;      // Neighbor vertex
        int weight = edge.second; // Edge weight

        // Relaxation Step
        if (dist[u] + weight < dist[v]) {
            dist[v] = dist[u] + weight;
            pq.push({dist[v], v}); // Add new improved path to queue
        }
    }
}
```

## Complete Implementation

C++

```cpp
#include <vector>
#include <queue>
#include <limits>
#include <utility>

using namespace std;

// Typedef for cleaner code: {distance, node} or {neighbor, weight}
using pii = pair<int, int>;

void dijkstra(int n, int source, const vector<vector<pii>>& adj, vector<int>& dist) {
    // Step 3: Initialize distances
    dist.assign(n, numeric_limits<int>::max());
    dist[source] = 0;
    
    // Min-Priority Queue
    priority_queue<pii, vector<pii>, greater<pii>> pq;
    pq.push({0, source});
    
    // Step 4: Main Loop
    while (!pq.empty()) {
        // Step 5: Greedy Selection
        int d = pq.top().first;
        int u = pq.top().second;
        pq.pop();
        
        // Step 6: Stale Entry Check
        if (d > dist[u]) 
            continue;
        
        // Step 7: Explore Neighbors
        for (const auto& edge : adj[u]) {
            int v = edge.first;
            int weight = edge.second;
            
            // Relaxation
            if (dist[u] + weight < dist[v]) {
                dist[v] = dist[u] + weight;
                pq.push({dist[v], v});
            }
        }
    }
}
```

## Example Usage

C++

```cpp
#include <iostream>

int main() {
    int n = 5; // 5 vertices (0 to 4)
    vector<vector<pii>> adj(n);
    
    // Add edges (using push_back to adjacency list)
    // Format: adj[u].push_back({v, weight});
    adj[0].push_back({1, 4});
    adj[0].push_back({2, 1});
    adj[2].push_back({1, 2}); // 0->2->1 = 3 (better than direct 0->1)
    adj[1].push_back({3, 1});
    adj[2].push_back({3, 5});
    adj[3].push_back({4, 3});
    
    vector<int> dist;
    dijkstra(n, 0, adj, dist);
    
    // Output results
    for (int i = 0; i < n; i++) {
        if (dist[i] == numeric_limits<int>::max()) {
            cout << "Node " << i << " is unreachable" << endl;
        } else {
            cout << "Shortest distance to Node " << i << ": " << dist[i] << endl;
        }
    }
    
    return 0;
}
```

## Time Complexity

- **Time:** $O(E \log V)$
    
    - We process every edge ($E$) at most once.
        
    - Pushing/Popping from the Priority Queue takes logarithmic time relative to vertices ($\log V$).
        
    - _Faster than Bellman-Ford's $O(V \cdot E)$._
        
- **Space:** $O(V + E)$
    
    - To store the Adjacency List and the Distance Array.
        

## Bellman-Ford vs Dijkstra's

|**Feature**|**Dijkstra's**|**Bellman-Ford**|
|---|---|---|
|**Negative Weights**|❌ Fails (Infinite Loops or Wrong Answer)|✅ Works perfectly|
|**Speed**|🚀 Fast: $O(E \log V)$|🐢 Slow: $O(V \cdot E)$|
|**Approach**|**Greedy**: "Pick the best option now."|**Dynamic Programming**: "Relax everything repeatedly."|
|**Structure**|Priority Queue + Adjacency List|Edge List|
|**Best For**|Maps, Networks (Standard routing)|Financial Graphs (Arbitrage), Negative Constraints|

## Key Intuitions to Remember

1. **Lazy Deletion (Step 6):** We don't delete old items from the Priority Queue; we just ignore them when they pop out. This avoids complex data structure operations.
    
2. **Why Adjacency List?** Since Dijkstra is like a fire spreading, we only care about what is _connected_ to the current node. An Edge List (like in Bellman-Ford) would force us to check irrelevant edges on the other side of the graph.
    
3. **Why Non-Negative?** Dijkstra assumes that once a node is "popped" from the queue, that is the _best_ possible way to reach it. Negative edges break this assumption because a path could get _longer_ (more steps) but _cheaper_ (negative weight), confusing the greedy logic.