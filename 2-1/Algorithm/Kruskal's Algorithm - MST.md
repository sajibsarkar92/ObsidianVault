

# Kruskal’s Algorithm Implementation Guide (Min-Heap Version)

## Overview

Kruskal's Algorithm builds a **Minimum Spanning Tree (MST)** by connecting the cheapest edges first.

- **The Analogy:** Think of the graph as a collection of separate islands. You have a budget to build bridges.
    
- **The Strategy:** You look at a catalog of all possible bridges, ranked from cheapest to most expensive (the Min-Heap). You build every bridge you can afford, **unless** it connects two islands that are already connected (because that would be redundant).
    
<iframe width="560" height="315" src="https://www.youtube.com/embed/4ZlRH0eK-qQ?si=D_KIHALG3-XzLHzc" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

## Step-by-Step Implementation

### Step 1: Include Necessary Headers

**Why?**

- `priority_queue`: This acts as our "Catalog" of bridges. It automatically keeps the cheapest bridge at the top.
    
- `vector`: To create the map of "Bosses" (DSU) to track who is connected to whom.
    

C++

```cpp
#include <iostream>
#include <vector>
#include <queue> 
using namespace std;
```

---

### Step 2: Define the Edge Structure

**Why?**

We need to store the `u`, `v`, and `weight` together.

- **Crucial for Heap:** We need to teach the Heap how to sort Edges. Since C++ Priority Queues are Max-Heaps by default, we use the `>` operator so that when we ask for `greater`, it puts the **smallest** weight at the top.
    

C++

```cpp
struct Edge {
    int u, v, weight;
    
    // Logic: "I am 'greater' than you if my weight is higher."
    // The Min-Heap uses this to push heavy edges to the bottom.
    bool operator>(const Edge& other) const {
        return weight > other.weight;
    }
};
```

---

### Step 3: The "Cycle Police" (DSU Functions)

**Why?**

Before building a bridge, we must check if it creates a loop. We do this by checking if the two nodes have the same "Boss."

- **`parent[i]`**: Stores the Boss of node `i`.
    
- **`find(i)`**: Climbs the chain of command to find the Ultimate Leader.
    
- **`unite(i, j)`**: Merges two groups by making one leader report to the other.
    

C++

```cpp
// The Array to store the "Boss" of every node
int parent[100]; 

// Function 1: Find the Ultimate Leader (with Path Compression)
int find(int i) {
    if (parent[i] == i) return i; // I am the boss
    return parent[i] = find(parent[i]); // Ask my boss for the real leader
}

// Function 2: Join two families
void unite(int i, int j) {
    int root_i = find(i);
    int root_j = find(j);
    if (root_i != root_j) {
        parent[root_i] = root_j; // Merge the groups
    }
}
```

---

### Step 4: The Min-Heap Setup

**Why?**

Instead of sorting a vector (which takes extra steps), we just dump all edges into this magic bucket. The bucket automatically hands us the cheapest edge whenever we ask.

C++

```cpp
// Defines a Priority Queue that stores Edges, using a Vector container,
// and orders them so the SMALLEST item is on top (Min-Heap).
priority_queue<Edge, vector<Edge>, greater<Edge>> pq;
```

---

### Step 5: The Main Loop (Build Bridges)

**Deep Intuition:**

This is the execution phase.

1. **Pop:** Take the cheapest bridge from the top of the heap.
    
2. **Check:** Ask the "Cycle Police" (`find`): "Are these two islands already connected?"
    
    - **If Yes (`find(u) == find(v)`)**: Throw the bridge away. It creates a cycle.
        
    - **If No**: Build it! Call `unite` to update the records.
        

C++

```cpp
    int mst_cost = 0;

    while (!pq.empty()) {
        // 1. Get cheapest edge
        Edge e = pq.top();
        pq.pop();

        // 2. Cycle Check (Are they in different groups?)
        if (find(e.u) != find(e.v)) {
            // 3. Safe to build!
            unite(e.u, e.v);
            mst_cost += e.weight;
            cout << "Bridge Built: " << e.u << " - " << e.v << endl;
        }
    }
```

---

## Complete Implementation (Single File)

C++

```cpp
#include <iostream>
#include <vector>
#include <queue> 

using namespace std;

// Step 2: Simple Edge Structure
struct Edge {
    int u, v, weight;
    
    // Comparator for Min-Heap (Smallest weight floats to top)
    bool operator>(const Edge& other) const {
        return weight > other.weight;
    }
};

// Step 3: DSU (Cycle Police) Variables
const int MAX_NODES = 100;
int parent[MAX_NODES];

// Recursive Find Function
int find(int i) {
    if (parent[i] == i) return i;
    return parent[i] = find(parent[i]);
}

// Unite Function
void unite(int i, int j) {
    int root_i = find(i);
    int root_j = find(j);
    if (root_i != root_j) {
        parent[root_i] = root_j;
    }
}

int main() {
    int n = 4; // Number of nodes (0, 1, 2, 3)

    // Initialization: Everyone is their own boss
    for(int i = 0; i < n; i++) parent[i] = i;

    // Step 4: Create Min-Heap
    priority_queue<Edge, vector<Edge>, greater<Edge>> pq;

    // Add edges to the heap {u, v, weight}
    pq.push({0, 1, 10});
    pq.push({0, 2, 6});
    pq.push({0, 3, 5});
    pq.push({1, 3, 15});
    pq.push({2, 3, 4});

    int mst_cost = 0;

    // Step 5: Main Logic Loop
    cout << "Construction Log:" << endl;
    while (!pq.empty()) {
        Edge e = pq.top(); // Peek at cheapest
        pq.pop();          // Remove it

        // Check for Cycle
        if (find(e.u) != find(e.v)) {
            // Not connected yet? Connect them.
            unite(e.u, e.v);
            mst_cost += e.weight;
            cout << "Connected Node " << e.u << " and " << e.v << " (Cost: " << e.weight << ")" << endl;
        } else {
            // Already connected? Discard.
            // (Implicitly ignored)
        }
    }

    cout << "Total Minimum Cost: " << mst_cost << endl;

    return 0;
}
```

---

## Time & Space Complexity

### Time: $O(E \log E)$

- **Heap Operations:** We push all $E$ edges into the heap, and pop them out. Each heap operation takes $\log E$.
    
- **DSU Operations:** The `find` and `unite` functions are extremely fast (nearly constant time, or $O(1)$) thanks to Path Compression.
    

### Space: $O(V + E)$

- We store all edges in the Heap ($E$) and the parent array ($V$).
    

---

## Key Intuitions to Remember

1. **The "Find" Check:** This is the only line that matters for correctness. `if (find(u) != find(v))` literally translates to: _"If there is no path between U and V, build this bridge."_
    
2. **The Min-Heap:** It does the sorting work for you. You don't need to write a `sort` function or manage array indices. You just throw edges in and pull them out one by one.
    
3. **Global vs Local:** Unlike Prim's (which grows from a specific start node), Kruskal's jumps all over the map, building isolated bridges (small trees) and eventually merging them all into one giant tree.