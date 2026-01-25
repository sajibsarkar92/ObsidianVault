# Maximum Flow & Ford-Fulkerson Algorithm

## The Problem

Imagine you have a network of pipes connecting a water source to a destination. Each pipe has a maximum capacity - it can only carry so much water per unit time. The fundamental question is: **What's the maximum amount of water we can push from the source to the destination?**

This is the **Maximum Flow Problem**, and it applies to many real-world scenarios:

- Shipping goods through a transportation network
- Data transmission through computer networks
- Traffic flow through road systems
- Supply chain optimization

# <span style="color:rgb(192, 0, 0)">Core Terminology</span>


![[19_maxflow_2-4.webp]]

### Network Components

**Network (Graph)** A directed graph G = (V, E) where:

- Vertices (V) represent junctions or nodes
- Edges (E) represent pipes/conduits that carry flow
- There's a **source node (s)** where flow originates
- There's a **sink node (t)** where flow terminates
- Every vertex lies on some path from s to t

### Capacity - c(u,v)

**Capacity** is the maximum amount of flow that can pass through an edge.

- Denoted as c(u,v) for edge from u to v
- Always non-negative: c(u,v) ≥ 0
- If edge (u,v) doesn't exist, we assume c(u,v) = 0
- Think of it as the "pipe size" - bigger pipes carry more flow

**Example:** c(s, v₁) = 16 means the edge from source s to vertex v₁ can carry at most 16 units of flow.

# <span style="color:rgb(192, 0, 0)">Flow - f(u,v)</span> 


**Flow** is the actual amount being sent through an edge.

A flow must satisfy three critical conditions:

1. **Capacity Constraint**: 0 ≤ f(u,v) ≤ c(u,v)
    
    - You can't send more than the pipe's capacity
    - You can't send negative flow
2. **Skew Symmetry**: f(u,v) = -f(v,u)
    
    - Flow from u to v is the negative of flow from v to u
    - This implies f(u,u) = 0 (no self-loops)
    - Sending 7 units from u to v and 3 units from v to u is equivalent to sending 4 units from u to v (net effect)
3. **Flow Conservation**: For every vertex v (except s and t):
    
    - ∑ f(u,v) = 0 (sum over all edges touching v)
    - "What comes in must go out"
    - No flow is created or destroyed at intermediate nodes
![[19_maxflow_2-10.webp]]

### Value of a Flow - |f|

The **value of a flow** is the total flow leaving the source (which equals the total flow entering the sink).

|f| = ∑ f(s,v) for all v in V

**Example:** If the source sends 11 units to v₁, 8 units to v₂, and 0 to others, then |f| = 19. This same 19 units must arrive at the sink.

### Residual Capacity - c_f(u,v)

This is the **additional capacity** available on an edge, given the current flow.

**Formula:** c_f(u,v) = c(u,v) - f(u,v)

**Why it matters:** The residual capacity tells us how much more flow we can push through an edge.

**Key insight:** Even if an edge is at full capacity in the forward direction, we can "push back" flow by using the reverse edge in the residual network.

**Example:**

- If c(u,v) = 10 and f(u,v) = 7, then c_f(u,v) = 3
- We can send 3 more units from u to v
- But also, c_f(v,u) = c(v,u) - f(v,u) = 0 - (-7) = 7
- We can "cancel" up to 7 units by reversing flow

**Why is residual capacity always non-negative?** Because c_f(u,v) = c(u,v) - f(u,v), and we know from the capacity constraint that f(u,v) ≤ c(u,v), therefore c_f(u,v) ≥ 0.

### Residual Network - G_f

![[19_maxflow_2-19.webp]]


The **residual network** is a transformed version of the original network that shows all possible ways to modify the current flow.

**Construction:**

- For each edge (u,v) in the original network:
    - If c_f(u,v) > 0, include edge (u,v) with capacity c_f(u,v)
    - If c_f(v,u) > 0, include edge (v,u) with capacity c_f(v,u)

**Purpose:** The residual network lets us:

1. Add more flow in the forward direction (if capacity remains)
2. **Reverse previous flow decisions** (this is crucial!)

The reverse edges allow the algorithm to "undo" bad choices.

### Augmenting Path

An **augmenting path** is a simple path from s to t in the residual network.

- It represents a way to increase the current flow
- The **residual capacity of the path** c_f(p) = min{c_f(u,v) : (u,v) ∈ p}
    - This is the bottleneck - the maximum additional flow we can push along this path
![[19_maxflow_2-22.webp]]

## Cuts and the Max-Flow Min-Cut Theorem

### What is a Cut?

A **cut (S,T)** is a partition of vertices into two sets:

- S contains the source s
- T = V - S contains the sink t

**Net flow through cut:** f(S,T) = sum of all flows from vertices in S to vertices in T (including negative flows back from T to S)

**Capacity of cut:** c(S,T) = sum of all capacities from vertices in S to vertices in T (only positive capacities, no reverse)

**Minimum cut:** A cut with the smallest capacity among all possible cuts.

### Important Lemmas

**Lemma 26.5:** |f| = f(S,T)

- The value of any flow equals the net flow across any cut

**Lemma 26.6:** |f| ≤ c(S,T)

- The value of any flow is bounded from above by the capacity of any cut

![[19_maxflow_2-14.webp]]

- Example showing f(S,T) = 12 - 4 + 11 = 19

![[19_maxflow_2-15.webp]]

- Example showing c(S,T) = 12 + 14 = 26

### Max-Flow Min-Cut Theorem

This is one of the most beautiful results in algorithms:

**The maximum value of flow from s to t equals the minimum capacity of all cuts separating s from t.**

In other words: |f*| = c(S,T) for some minimum cut (S,T)

**Why this matters:**

- It gives us a way to verify if we've found the maximum flow
- When |f| = c(S,T) for some cut, we know f is maximum
- The bottleneck of the entire network is the minimum cut

**Three equivalent conditions:**

1. f is a maximum flow
2. The residual network G_f contains no augmenting paths
3. |f| = c(S,T) for some cut (S,T)

---

## Why Residual Networks Are Essential

**The Bad Path Problem:**

![[19_maxflow_2-21.webp]]

Consider this scenario:

```
Source --1--> a --1--> Destination
  |                        ^
  1                        1
  v                        |
  b ---------------------->
```

**Optimal solution:**

- Path 1: source → a → destination (1 unit)
- Path 2: source → b → destination (1 unit)
- Total flow = 2 units

**What if we choose badly first?** If we initially choose path source → a → b → destination, we send 1 unit and now:

- Edge (a,b) is at capacity
- No more augmenting paths exist using only forward edges
- We're stuck at flow = 1

**How residual networks save us:** The residual network adds a reverse edge (b,a) with capacity 1. Now we can:

- Find path source → b → a → destination
- This "pushes back" the flow on (a,b), effectively canceling it
- Reroutes to: 1 unit via source → a → destination, 1 unit via source → b → destination
- We recover to the optimal flow of 2!

**Key insight:** Path source → a → b → destination "overlaps with too many other paths" - it blocks both optimal paths. Residual networks detect this condition and allow us to reverse the bad decision.

---

## The Ford-Fulkerson Method

### Core Idea

![[19_maxflow_2-24.webp]]

Start with zero flow, then repeatedly find augmenting paths in the residual network and push flow along them until no more augmenting paths exist.

**Algorithm:**

```
Ford-Fulkerson(G, s, t)
1  for each edge (u,v) ∈ G.E do
2      f(u,v) = f(v,u) = 0
3  while ∃ path p from s to t in residual network G_f do
4      c_f(p) = min{c_f(u,v): (u,v) ∈ p}
5      for each edge (u,v) in p do
6          f(u,v) = f(u,v) + c_f(p)
7          f(v,u) = -f(u,v)
8  return f
```

**How it works:**

1. **Initialize:** Set all flows to 0
2. **Find augmenting path:** Look for any simple path from s to t in the residual network
3. **Find bottleneck:** The minimum residual capacity along this path
4. **Augment flow:** Increase flow along the path by the bottleneck amount
5. **Update residual network:** Adjust capacities for both forward and reverse edges
6. **Repeat:** Continue until no augmenting path exists

**Different implementations** of Ford-Fulkerson differ in how they choose the augmenting path in step 3.

---

## Visual Simulation: Step-by-Step Example from PDF

This example traces the exact simulation from your slides, showing how Ford-Fulkerson finds the maximum flow of 23.

### Initial State: Flow = 0



**Network structure:**

- Source s connects to vertices a, b, c, d
- Intermediate vertices a, b, c, d have edges between them
- All connect eventually to sink t
- All flows initially = 0

**Starting point:** No flow anywhere, |f| = 0

---

### Iteration 1: Flow = 4

![[19_maxflow_2-18.webp]]


![[19_maxflow_2-23.webp]]

**Augmenting path found:** (Shown in the diagram with highlighted path)

**What happened:**

- Found a path from s to t in the residual network
- Bottleneck capacity = 4 (the minimum capacity along the path)
- Pushed 4 units of flow along this path

**Result:** |f| = 4

**Residual network changes:**

- Forward edges along the path decreased by 4
- Reverse edges created with capacity 4 (allowing us to "undo" this flow if needed)
- All other edges remain unchanged

---

### Iteration 2: Flow = 11

![[19_maxflow_2-26.webp]]

**Augmenting path found:** (Next path shown in subsequent diagram)

![[19_maxflow_2-27.webp]]

**What happened:**

- Found another augmenting path from s to t
- Bottleneck capacity = 7 (which is 11 - 4 = 7 additional units)
- Pushed 7 more units along this new path

**Result:** |f| = 4 + 7 = 11

**Why this path works:**

- The residual network still has available capacity on certain edges
- This path might use different edges than the first path
- Some edges may now be at reduced capacity, forcing us to find alternative routes

**Residual network changes:**

- More forward capacities reduced
- More reverse edges added
- Network becomes more complex with multiple routing options

---

### Iteration 3: Flow = 19

![[19_maxflow_2-28.webp]]

**Augmenting path found:** (Next path shown in subsequent diagram)

![[19_maxflow_2-29.webp]]

**What happened:**

- Found yet another augmenting path
- Bottleneck capacity = 8 (which is 19 - 11 = 8 additional units)
- Pushed 8 more units along this path

**Result:** |f| = 11 + 8 = 19

**Observation:**

- We're getting closer to the maximum
- Available paths are becoming more constrained
- The residual network is getting more saturated

---

### Iteration 4: Flow = 23 (Maximum!)

![[19_maxflow_2-30.webp]]

**Augmenting path found:** (Final path shown in subsequent diagram)

![[19_maxflow_2-31.webp]]

**What happened:**

- Found one final augmenting path
- Bottleneck capacity = 4 (which is 23 - 19 = 4 additional units)
- Pushed 4 more units along this path

**Result:** |f| = 19 + 4 = 23

---

### Final State: No More Augmenting Paths

![[19_maxflow_2-32.webp]]

**Critical observation:** The residual network G_f contains no augmented paths.

**What this means:**

- There is NO simple path from s to t in the current residual network
- Either all paths are blocked by edges with zero residual capacity
- Or the sink t is unreachable from source s in G_f

**Therefore, by the Max-Flow Min-Cut Theorem:**

- Since the residual network contains no augmenting paths (condition 2)
- The flow f is a maximum flow in G (condition 1)
- And |f| = 23 = c(S,T) for some minimum cut (condition 3)

**Maximum flow = 23 units** ✓

---

## Why Ford-Fulkerson Works: The Deep Intuition

### Insight 1: Augmenting Paths Always Increase Flow

Every time we find an augmenting path:

- We identify a bottleneck (minimum residual capacity along the path)
- We push that amount of flow through
- The total flow |f| increases by at least the bottleneck amount

Since flow is bounded (it can't exceed the minimum cut capacity), and each iteration increases flow, we must eventually reach a state where no more augmenting paths exist.

### Insight 2: Residual Networks Enable Course Correction

The genius of residual networks:

- **Forward edges** represent "I can send more flow this way"
- **Reverse edges** represent "I can undo/reduce flow I previously sent"

When we send flow along an edge (u,v):

- The forward residual capacity decreases: c_f(u,v) = c(u,v) - f(u,v)
- The reverse residual capacity increases: c_f(v,u) = c(v,u) - f(v,u) = 0 - (-f(u,v)) = f(u,v)

This means if we later find that flow along (u,v) was a mistake, we can:

- Use the reverse edge (v,u) in a new augmenting path
- Effectively "push back" some flow
- Reroute it through a better path

### Insight 3: Termination Guarantees Optimality

**When the algorithm terminates:**

- No augmenting path exists from s to t in G_f
- This means there's a "cut" in the residual network
- All edges crossing from S (reachable from s) to T (unreachable) have zero residual capacity

**This implies:**

- All forward edges from S to T are at full capacity in the original network
- All reverse edges from T to S have zero flow in the original network
- Therefore: |f| = c(S,T) for this cut

**By the Max-Flow Min-Cut Theorem:** Since we've found a flow f where |f| = c(S,T) for some cut, this flow must be maximum!

### The Three Equivalent Conditions in Action

At termination:

1. ✓ **f is a maximum flow** (what we wanted to prove)
2. ✓ **G_f contains no augmenting paths** (why the algorithm stopped)
3. ✓ **|f| = c(S,T) for some cut** (the mathematical guarantee)

These three statements are logically equivalent - if any one is true, all three must be true.

---

## Time Complexity Analysis

![[19_maxflow_2-33.webp]]
![[19_maxflow_2-34.webp]]
### For Integer Capacities

**Key observations:**

- Each augmenting path takes O(E) time to find (using DFS or BFS)
- Each augmenting path takes O(E) time to augment the flow
- With integer capacities, each augmentation increases |f| by at least 1
- Maximum possible flow is |f*| (the optimal flow value)

**Therefore:**

- Number of iterations ≤ |f*|
- Time per iteration = O(E)
- __Total time: O(E × |f_|)_*

### The Problem with This Analysis

**This is NOT polynomial in input size!**

The time depends on |f*|, which is not a function of |V| or |E|. The value |f*| can be exponentially large compared to the number of vertices and edges.

**Special cases:**

- If capacities are rational, we can scale them to integers
- If capacities are irrational, Ford-Fulkerson might never terminate!

### The Pathological Case

![[19_maxflow_2-37.webp]]

![[19_maxflow_2-38.webp]]
![[19_maxflow_2-39.webp]]
![[19_maxflow_2-40.webp]]
Consider a network where |f*| = 2,000,000:

```
         1,000,000
    s --------------- a
    |                 |
    | 1               | 1
    |                 |
    b --------------- t
         1,000,000
```

With edge (a,b) having capacity 1.

**If we're unlucky:**

- Iteration 1: Path s → a → b → t, flow +1
- Iteration 2: Path s → b → a → t, flow +1 (using reverse of a→b)
- Iteration 3: Path s → a → b → t, flow +1
- Iteration 4: Path s → b → a → t, flow +1
- **Repeat 999,999 more times...**

We alternate back and forth 2,000,000 times, each time only increasing flow by 1!

---

## The Edmonds-Karp Algorithm: A Polynomial Solution

**[INSERT: "The Edmonds-Karp Algorithm" slides from PDF]**

### The Simple Fix

**Key modification:** Instead of choosing ANY augmenting path, choose the **shortest augmenting path** (in terms of number of edges).

**How:** Use Breadth-First Search (BFS) on the residual network to find the shortest path from s to t.

### Why This Works

**Theorem:** The total number of flow augmentations in Edmonds-Karp is O(VE).

**Time complexity:**

- Each BFS takes O(E) time
- Number of augmentations = O(VE)
- **Total time: O(VE²)** - This IS polynomial!

### Example: The Pathological Case Solved

**[INSERT: "The Edmonds-Karp Algorithm - example" diagram from PDF]**

On the same pathological network that took 2,000,000 iterations:

- **Edmonds-Karp completes in just 2 iterations!**

**Why:**

- Iteration 1: Shortest path s → a → t (flow = 1,000,000)
- Iteration 2: Shortest path s → b → t (flow = 1,000,000)
- Done!

By always choosing the shortest path, we avoid the back-and-forth oscillation.

---

## Summary

**Maximum Flow Problem:** Find the maximum amount that can flow from source to sink in a capacity-constrained network.

**Ford-Fulkerson Method:** Iteratively find augmenting paths and push flow until no more paths exist.

**Why it works:**

- Residual networks allow us to correct routing mistakes via reverse edges
- Termination occurs exactly when flow equals minimum cut capacity
- Max-Flow Min-Cut theorem guarantees optimality

**Key concepts to remember:**

- **Capacity c(u,v):** Maximum an edge can carry
- **Flow f(u,v):** Actual amount being sent (satisfies capacity constraint, skew symmetry, flow conservation)
- **Residual capacity c_f(u,v):** How much more flow we can push (or cancel via reverse)
- **Residual network G_f:** Shows all ways to modify current flow (forward + reverse edges)
- **Augmenting path:** A path in G_f from s to t that increases flow
- **Cut (S,T):** Partition of vertices; capacity c(S,T) bounds flow
- **Max-Flow Min-Cut:** Maximum flow value = Minimum cut capacity

**Complexity:**

- Basic Ford-Fulkerson: O(E × |f*|) - pseudo-polynomial
- Edmonds-Karp (using BFS): O(VE²) - polynomial

**The beauty:** The algorithm is conceptually simple, yet the Max-Flow Min-Cut theorem provides a profound guarantee of correctness. The ability to reverse flow decisions through residual networks is what makes the greedy approach work!




![[19_maxflow_2-3.webp]]




![[19_maxflow_2-8.webp]]
![[19_maxflow_2-9.webp]]

![[19_maxflow_2-11.webp]]
![[19_maxflow_2-12.webp]]
![[19_maxflow_2-13.webp]]


![[19_maxflow_2-16.webp]]
![[19_maxflow_2-17.webp]]
















![[19_maxflow_2-34.webp]]
![[19_maxflow_2-35.webp]]
![[19_maxflow_2-36.webp]]


![[19_maxflow_2-41.webp]]
![[19_maxflow_2-42.webp]]
![[19_maxflow_2-43.webp]]
![[19_maxflow_2-44.webp]]
![[19_maxflow_2-45.webp]]
![[19_maxflow_2-46.webp]]