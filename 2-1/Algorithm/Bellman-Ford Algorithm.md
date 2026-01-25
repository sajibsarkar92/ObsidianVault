# Bellman-Ford Algorithm Implementation Guide

## Overview

Bellman-Ford algorithm finds the shortest path from a source vertex to all other vertices in a weighted graph. Unlike Dijkstra's, it works with **negative edge weights** and can detect **negative cycles**.

<iframe width="560" height="315" src="https://www.youtube.com/embed/FtN3BYH2Zes?si=4QX6n5Hh_F8bY7NI" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>



---

## Step-by-Step Implementation

### Step 1: Include Necessary Headers

**Why?** We need vector to store our edges and distance values, and limits to represent infinity. Unlike Dijkstra's, we don't need a priority queue because Bellman-Ford processes edges in a simpler, more systematic way - it just goes through all edges repeatedly rather than picking the "best" vertex each time.

```cpp
#include <vector>
#include <limits>
using namespace std;
```

---

### Step 2: Define the Edge Structure

**Why?** Unlike Dijkstra's which uses an adjacency list, Bellman-Ford works better with an explicit list of all edges. This is because the algorithm needs to examine every single edge in the graph V-1 times (where V is the number of vertices). Having all edges in one simple list makes this repetitive checking straightforward - we can just iterate through the same edge list multiple times.

```cpp
struct Edge {
    int from;   // source vertex
    int to;     // destination vertex
    int weight; // edge weight (can be negative!)
};

vector<Edge> edges; // List of all edges in the graph
```

---

### Step 3: Initialize the Distance Array

**Why?** We start by assuming every vertex is infinitely far away from the source. Then we set the source's distance to 0 because obviously the distance from a vertex to itself is zero. This initialization gives us a starting point - we'll gradually improve these distance estimates by examining edges. Think of it like this: at first, we know nothing about how to reach other vertices, so we assume the worst (infinity), except for our starting point which we know we're already at.

```cpp
int n; // number of vertices
vector<int> dist(n, numeric_limits<int>::max());
dist[source] = 0;
```

---

### Step 4: Main Relaxation Loop - Run V-1 Times

**Why?** This is the heart of Bellman-Ford. Here's the deep intuition: In any graph, the longest possible shortest path (without cycles) can have at most V-1 edges, where V is the number of vertices. Why? Because a path visits vertices, and if you visit the same vertex twice, you've created a cycle. So the maximum number of edges in a simple path is V-1 (visiting all V vertices means using V-1 edges to connect them).

Each iteration of this loop finds paths that are "one edge longer" than before. After iteration 1, we've found all shortest paths that use 1 edge. After iteration 2, we've found all shortest paths that use up to 2 edges. After V-1 iterations, we've found all shortest paths that could possibly exist in the graph (those using up to V-1 edges).

```cpp
// Relax all edges V-1 times
for (int i = 0; i < n - 1; i++) {
```

---

### Step 5: Examine Every Edge for Relaxation

**Why?** In each iteration, we check every single edge to see if using that edge would give us a shorter path to its destination. We do this for ALL edges because we don't know which edges will be useful - we're not being greedy like Dijkstra's. This brute-force approach of checking all edges is slower but more thorough, and it's what allows us to handle negative weights correctly. The order doesn't matter because we're doing multiple passes anyway.

```cpp
    for (const Edge& e : edges) {
```

---

### Step 6: Check if Source Vertex is Reachable

**Why?** Before we can use an edge to improve our path, we need to make sure we've actually found a way to reach the starting vertex of that edge. If `dist[e.from]` is still infinity, it means we haven't discovered any path to `e.from` yet, so we can't use it as a stepping stone to reach `e.to`. This check prevents integer overflow too - if we tried to add a weight to infinity without this check, we'd get undefined behavior in C++.

```cpp
        if (dist[e.from] == numeric_limits<int>::max()) {
            continue; // Can't use this edge yet
        }
```

---

### Step 7: Perform Edge Relaxation

**Why?** "Relaxation" means asking: "Would it be shorter to reach vertex `to` by going through vertex `from` using this edge, compared to the best path we've found so far?" If going through this edge gives us a better (shorter) distance, we update our knowledge. This is called "relaxation" because we're "relaxing" our previous constraint - we had an upper bound on the distance, and we're lowering it (making it less strict). Over many iterations, these relaxations gradually converge to the true shortest distances.

```cpp
        // Try to improve the distance to e.to
        if (dist[e.from] + e.weight < dist[e.to]) {
            dist[e.to] = dist[e.from] + e.weight;
        }
    }
}
```

---

### Step 8: Check for Negative Cycles (Extra Iteration)

**Why?** After V-1 iterations, we should have found all shortest paths. But what if the graph has a negative cycle (a cycle whose total weight is negative)? If such a cycle exists and is reachable from the source, we can keep going around it to make distances smaller and smaller infinitely - there's no "shortest path" because we can always make it shorter by going around the cycle again!

To detect this, we do one more pass through all edges. If we can STILL improve any distance after V-1 iterations, it means there's a negative cycle affecting that vertex. Why? Because legitimate shortest paths can't use more than V-1 edges, so if iteration V still finds improvements, those improvements must be coming from going around a cycle with negative total weight.

```cpp
// Check for negative weight cycles
for (const Edge& e : edges) {
    if (dist[e.from] != numeric_limits<int>::max() && 
        dist[e.from] + e.weight < dist[e.to]) {
        // Negative cycle detected!
        // dist[e.to] can be reduced further, which shouldn't happen
        return false; // or handle negative cycle as needed
    }
}
```

---

### Step 9: Return Success

**Why?** If we made it through the negative cycle check without finding any improvements, it means the distances we've computed are valid shortest paths. There are no negative cycles affecting reachable vertices, so the algorithm has successfully computed the shortest path tree from the source.

```cpp
return true; // No negative cycle, distances are valid
```

---

## Complete Implementation

```cpp
#include <vector>
#include <limits>
using namespace std;

struct Edge {
    int from;
    int to;
    int weight;
};

bool bellmanFord(vector<Edge>& edges, int n, int source, vector<int>& dist) {
    // Step 3: Initialize distances
    dist.assign(n, numeric_limits<int>::max());
    dist[source] = 0;
    
    // Step 4: Main loop - relax all edges V-1 times
    for (int i = 0; i < n - 1; i++) {
        // Step 5: Examine every edge
        for (const Edge& e : edges) {
            // Step 6: Check if source vertex is reachable
            if (dist[e.from] == numeric_limits<int>::max()) {
                continue;
            }
            
            // Step 7: Relax the edge
            if (dist[e.from] + e.weight < dist[e.to]) {
                dist[e.to] = dist[e.from] + e.weight;
            }
        }
    }
    
    // Step 8: Check for negative cycles
    for (const Edge& e : edges) {
        if (dist[e.from] != numeric_limits<int>::max() && 
            dist[e.from] + e.weight < dist[e.to]) {
            return false; // Negative cycle detected
        }
    }
    
    // Step 9: Success - no negative cycles
    return true;
}
```

---

## Example Usage

```cpp
int main() {
    int n = 5; // 5 vertices (0 to 4)
    vector<Edge> edges;
    
    // Add edges
    edges.push_back({0, 1, -1});
    edges.push_back({0, 2, 4});
    edges.push_back({1, 2, 3});
    edges.push_back({1, 3, 2});
    edges.push_back({1, 4, 2});
    edges.push_back({3, 2, 5});
    edges.push_back({3, 1, 1});
    edges.push_back({4, 3, -3});
    
    vector<int> dist;
    bool hasNoNegativeCycle = bellmanFord(edges, n, 0, dist);
    
    if (hasNoNegativeCycle) {
        // dist now contains shortest distances
        for (int i = 0; i < n; i++) {
            if (dist[i] == numeric_limits<int>::max()) {
                // Vertex i is not reachable from source
            } else {
                // dist[i] is shortest distance to vertex i
            }
        }
    } else {
        // Negative cycle exists
    }
    
    return 0;
}
```

---

## Time Complexity

- **O(V × E)** where V is vertices and E is edges
- We iterate V-1 times (outer loop)
- Each iteration examines all E edges
- Additional O(E) for negative cycle detection

## Space Complexity

- **O(V)** for the distance array
- O(E) for storing edges (input)

---

## Bellman-Ford vs Dijkstra's

|Feature|Bellman-Ford|Dijkstra's|
|---|---|---|
|**Negative Weights**|✅ Works with negative edges|❌ Only non-negative edges|
|**Negative Cycles**|✅ Can detect them|❌ Cannot handle them|
|**Time Complexity**|O(V × E) - slower|O((V + E) log V) - faster|
|**Approach**|Brute force - checks all edges|Greedy - picks best vertex|
|**Use Case**|Graphs with negative weights|General shortest path (positive weights)|

---

## Key Intuitions to Remember

1. **Why V-1 iterations?** Because the longest simple path in a graph has V-1 edges. Each iteration finds paths that are "one edge longer."
    
2. **Why check all edges every time?** Unlike Dijkstra's greedy approach, we don't know which edges will be useful when negative weights are involved, so we must check them all.
    
3. **Why can it handle negative weights?** Because it doesn't make greedy choices - it systematically considers all possible paths, no matter how "bad" they look initially.
    
4. **How does negative cycle detection work?** If we can still improve distances after V-1 iterations (when all legitimate shortest paths should be found), those improvements must be from going around a negative cycle.
    
5. **When to use Bellman-Ford?** When you have negative edge weights, need to detect negative cycles, or when the graph is dense and Dijkstra's wouldn't be much faster anyway.z