# Dhaka City Multi-Modal Routing System — Code Explanation

  

This document explains the complete codebase of the Dhaka City Routing System in plain English. It starts with the big picture — files, data structures, and how they relate — then walks through the entire program in the exact order the code runs, explaining every function in detail as we encounter it.

  

---

  

## Table of Contents

  

1. [Codebase Overview — File Structure](#1-codebase-overview--file-structure)

2. [The Building Blocks — Data Structures](#2-the-building-blocks--data-structures)

3. [How the Structs Work Together](#3-how-the-structs-work-together)

4. [Step 1: Parsing CSV Data](#4-step-1-parsing-csv-data)

5. [Step 2: Building the Road Layer](#5-step-2-building-the-road-layer)

6. [Step 3: Building Transit Layers](#6-step-3-building-transit-layers)

7. [Step 4: Linking Layers with Walk Edges](#7-step-4-linking-layers-with-walk-edges)

8. [Step 5: User Input and Configuration](#8-step-5-user-input-and-configuration)

9. [Step 6: Resolving GPS to Graph Nodes](#9-step-6-resolving-gps-to-graph-nodes)

10. [Step 7: Computing Edge Weights](#10-step-7-computing-edge-weights)

11. [Step 8: Running Dijkstra's Algorithm](#11-step-8-running-dijkstras-algorithm)

12. [Step 9: Output and KML Generation](#12-step-9-output-and-kml-generation)

13. [The Six Problems — Quick Reference](#13-the-six-problems--quick-reference)

  

---

  

## 1. Codebase Overview — File Structure

  

The project is split into 5 source files, each with a specific job:

  

```

├── types.h          → All data structures (structs) live here

├── parser.h/.cpp    → Reads CSV files + writes KML output

├── graph.h/.cpp     → Takes parsed data and builds the graph

├── solver.h/.cpp    → Core algorithm: GPS resolution, weighting, Dijkstra

├── main.cpp         → Entry point: ties everything together

├── input1-6.txt     → Pre-built inputs for each problem scenario

├── solution1-6.txt  → Output journey plans

└── solution1-6.kml  → Route files for Google Maps

```

  

### What each file does:

  

| File | Role | Key Functions |

|------|------|---------------|

| **types.h** | Defines all the data structures the whole codebase uses | `Point`, `Edge`, `Node`, `Route`, `Schedule`, `ProblemConfig`, `DijkState` |

| **parser.cpp** | Reads the raw CSV data files and converts them into structured C++ objects. Also writes the final map file. | `parseRoads()`, `parseTransit()`, `generateKML()` |

| **graph.cpp** | Takes the parsed data and constructs a graph — nodes connected by edges. This is where the road network, transit networks, and walking connections are built. | `getCreateNode()`, `buildRoadLayer()`, `buildTransitLayer()`, `linkLayersWithWalk()` |

| **solver.cpp** | The brains of the operation — resolves GPS coordinates, sets up problem rules, calculates edge weights, and runs Dijkstra's shortest path algorithm. | `formatTime()`, `resolvePointer()`, `buildConfig()`, `computeWeight()`, `solve()` |

| **main.cpp** | The orchestrator — calls everything in order, handles user input, and prints the results. | `main()` |

  

### Dependency chain:

  

```

types.h ← parser.h ← graph.h ← solver.h ← main.cpp

   ↑         ↑           ↑          ↑

   └─────────┴───────────┴──────────┘

         Everything depends on types.h

```

  

Every file includes `types.h` because all the shared data structures (`Node`, `Edge`, `Point`, etc.) are defined there. The chain flows left to right, and `main.cpp` ties them all together.

  

---

  

## 2. The Building Blocks — Data Structures

  

Before we look at any function, we need to understand the 7 structs in `types.h`. Every function in the codebase reads from or writes to these structures.

  

### 2.1 `Point` — A GPS Location

  

```cpp

struct Point {

    double lat;  // Latitude  (e.g., 23.834145)

    double lon;  // Longitude (e.g., 90.363833)

  

    double distanceTo(Point other) { ... }  // Haversine formula

};

```

  

A `Point` holds a GPS coordinate — a spot on the map of Dhaka. It also has one method:

  

**`distanceTo(Point other)`** calculates the **real-world distance in kilometers** between two GPS points using the **Haversine formula**. This formula accounts for the curvature of the Earth. You can't just subtract latitudes — the Earth is a sphere.

  

```cpp

double distanceTo(Point other){

    double R = 6371.0; // Earth's radius in km

  

    // Step 1: Convert degree differences to radians

    double dLat = (other.lat - lat) * pi / 180.0;

    double dLon = (other.lon - lon) * pi / 180.0;

  

    // Step 2: Haversine formula — measures separation on a sphere

    double a = sin(dLat/2)*sin(dLat/2) +

               cos(lat*pi/180.0) * cos(other.lat*pi/180.0) *

               sin(dLon/2) * sin(dLon/2);

  

    // Step 3: Convert to arc distance

    double c = 2 * atan2(sqrt(a), sqrt(1 - a));

  

    return R * c;  // Distance in km

}

```

  

**Example:** `mirpur10.distanceTo(uttara)` where Mirpur 10 is (23.8087, 90.3688) and Uttara is (23.8759, 90.3795) returns ~**7.6 km**.

  

---

  

### 2.2 `Edge` — A Connection Between Two Locations

  

```cpp

struct Edge {

    int to;           // ID of the destination node

    double distance;  // Distance in km

    int mode;         // 0=Car, 1=Metro, 2=Bikalpa Bus, 3=Uttara Bus, 4=Walk

    string stopName;  // Name of the stop (if applicable)

};

```

  

An `Edge` is one link on the map — a stretch of road, one Metro link between stations, one bus link, or a walking path. The `mode` tells us which transport we'd use. Think of it as: *"You can travel from here to node `to`, it's `distance` km, using `mode`."*

  

### Transport Mode Constants:

  

```cpp

const int MODE_CAR          = 0;  // Private car on roads

const int MODE_METRO        = 1;  // Dhaka Metro Rail (MRT Line-6)

const int MODE_BIKALPA_BUS  = 2;  // Bikalpa Paribahan bus service

const int MODE_UTTARA_BUS   = 3;  // Uttara bus service

const int MODE_WALK         = 4;  // Walking (connects transit stops to roads)

```

  

---

  

### 2.3 `Node` — A Location in the City

  

```cpp

struct Node {

    int id;            // Unique identifier (0, 1, 2, ...)

    Point pos;         // GPS coordinates of this location

    string name;       // Name (e.g., "Mirpur 10", "" for unnamed intersections)

    vector<Edge> Adj;  // Adjacency list — all connections FROM this node

};

```

  

Every intersection on a road, every Metro station, and every bus stop becomes a `Node`. The `Adj` (adjacency list) stores all the edges going out from this node.

  

**Key distinction:** Named nodes (e.g., `"Mirpur 10"`) are transit stops. Unnamed nodes (`name = ""`) are pure road intersections. This distinction is used later when building walking connections.

  

---

  

### 2.4 `Route` — One Step of the Final Journey

  

```cpp

struct Route {

    int mode;         // Transport mode used (Car, Metro, etc.)

    int sourceNode;   // Starting node ID of this step

    int destNode;     // Ending node ID of this step

    double distance;  // Distance in km for this step

    double cost;      // Cost in Taka for this step

    string startTime; // Clock time when this step begins (e.g., "7:30 AM")

    string endTime;   // Clock time when this step ends (e.g., "7:45 AM")

};

```

  

This is what the solver outputs. A complete journey is a `vector<Route>` — a sequence of steps. For example: `[Car 2km, Walk 0.05km, Metro 5km]`.

  

---

  

### 2.5 `Schedule` — Transit Operating Hours

  

```cpp

struct Schedule {

    int freqMinutes;  // How often the bus/train arrives (0 = always available)

    int startHour;    // Service start time (24h format)

    int endHour;      // Service end time (24h format)

};

```

  

Tells us when a transport mode runs and how often. For example: Metro runs from 1 AM to 11 PM, arriving every 5 minutes → `{5, 1, 23}`.

  

---

  

### 2.6 `ProblemConfig` — Settings for Each Problem

  

```cpp

struct ProblemConfig {

    int problemNumber;        // Which of the 6 problems (1–6)

    double costPerKm[5];      // Cost per km for each mode

    double speed[5];          // Speed (km/h) for each mode

    Schedule schedules[5];    // Operating hours and frequency for each mode

    double startTimeMinutes;  // Departure time (minutes from midnight)

    double deadlineMinutes;   // Deadline for Problem 6 (-1 if none)

};

```

  

This is the **control center**. By changing the values here, the same Dijkstra algorithm produces different results for each of the 6 problems. Think of it as a "settings dial" — twist it and the algorithm optimizes for something different.

  

---

  

### 2.7 `DijkState` — Priority Queue Entry

  

```cpp

struct DijkState {

    double weight;       // The value being minimized (distance, cost, or time)

    int node;            // Which node we've reached

    double currentTime;  // Clock time when we arrived here

  

    bool operator>(const DijkState& other) const {

        return weight > other.weight;

    }

};

```

  

This goes into the priority queue during Dijkstra's algorithm. The `operator>` overloading ensures the queue always pops the **smallest weight first** (min-heap behavior).

  

```cpp

// The '>' operator makes the priority queue a MIN-heap:

// If our weight is BIGGER than 'other', we rank LOWER

// So the state with the smallest weight always comes out first

bool operator>(const DijkState& other) const {

    return weight > other.weight;

}

```

  

**Example:** Queue has `[{weight:15.2}, {weight:3.7}, {weight:8.1}]` → pops `3.7` first, then `8.1`, then `15.2`.

  

---

  

## 3. How the Structs Work Together

  

Here's how all the data structures connect:

  

```

                    ┌─────────────────────────────────────────┐

                    │              ProblemConfig               │

                    │  (costs, speeds, schedules, problem #)   │

                    └──────────────────┬──────────────────────┘

                                       │ controls the rules

                                       ▼

 ┌─────────┐     ┌─────────┐     ┌──────────────┐     ┌──────────┐

 │  Point   │────▶│  Node   │────▶│    Edge      │────▶│DijkState │

 │ (GPS)    │     │ (id,pos,│     │ (to, dist,   │     │(weight,  │

 │          │     │  name,  │     │  mode)       │     │ node,    │

 │ lat, lon │     │  Adj[]) │     │              │     │ time)    │

 └─────────┘     └─────────┘     └──────────────┘     └──────────┘

                                                             │

                                                             ▼

                                                       ┌──────────┐

                                                       │  Route   │

                                                       │ (output) │

                                                       │ mode,    │

                                                       │ distance,│

                                                       │ cost,    │

                                                       │ times    │

                                                       └──────────┘

```

  

The flow is:

1. **Points** are GPS locations — they get wrapped into **Nodes**

2. **Nodes** are connected by **Edges** — together they form the graph

3. **ProblemConfig** tells the algorithm HOW to evaluate edges (by distance? cost? time?)

4. **DijkStates** are used during Dijkstra to track progress through the graph

5. **Routes** are the final output — one per leg of the journey

  

**Relationship between `Schedule` and `ProblemConfig`:** Each `ProblemConfig` contains 5 `Schedule` objects (one per transport mode). The schedule tells the algorithm whether a mode is running right now and how long you'd wait for it.

  

Now let's follow the code as it runs, from start to finish.

  

---

  

## 4. Step 1: Parsing CSV Data

  

The first thing `main()` does after printing a banner is load the data:

  

```cpp

// In main.cpp:

vector<Node> nodes;

vector<RoadData> roads = parseRoads("Roadmap-Dhaka.csv");

vector<TransitData> metro = parseTransit("Routemap-DhakaMetroRail.csv", MODE_METRO);

vector<TransitData> bikalpa = parseTransit("Routemap-BikolpoBus.csv", MODE_BIKALPA_BUS);

vector<TransitData> uttara = parseTransit("Routemap-UttaraBus.csv", MODE_UTTARA_BUS);

```

  

Two functions handle all the parsing:

  

---

  

### Function: `parseRoads()`

  

**File:** `parser.cpp`

**Signature:** `vector<RoadData> parseRoads(string filename)`

**Input:** Path to the road CSV file (e.g., `"Roadmap-Dhaka.csv"`, ~3 MB, thousands of rows)

**Output:** A list of `RoadData` objects — each one holds the GPS coordinates along a road and its total distance.

  

**What it does:** Opens the CSV file and reads it line by line. Each line represents one road segment in Dhaka. It splits each line by commas, extracts the GPS coordinates, and stores the total distance.

  

**CSV format:**

```

road_id, lon1, lat1, lon2, lat2, ..., lonN, latN, road_name, total_distance_km

```

  

**Step-by-step walkthrough:**

  

```cpp

vector <RoadData> parseRoads(string filename){

    vector <RoadData> segments;

    ifstream file(filename);       // Open the CSV file

    string line;

  

    while(getline(file, line)){    // Read one line at a time

        if (line.empty()) continue; // Skip blank lines

  

        // --- Tokenize: split the line by commas ---

        stringstream ss(line);

        string token;

        vector <string> tokens;

        while(getline(ss, token, ',')){

            tokens.push_back(token);

        }

  

        RoadData road;

        // Last token = total distance of this road in km

        road.totalDistance = stod(tokens.back());

  

        // All middle tokens = (longitude, latitude) pairs

        // Note: CSV stores longitude first, then latitude

        for (int i = 1; i + 1 < (int)tokens.size() - 2; i += 2) {

            Point p;

            p.lon = stod(tokens[i]);    // Longitude first

            p.lat = stod(tokens[i+1]);  // Latitude second

            road.points.push_back(p);

        }

        segments.push_back(road);

    }

    file.close();

    return segments;

}

```

  

**Example:** Given this CSV line:

```

101,90.3688,23.8087,90.3700,23.8100,90.3720,23.8120,Main Road,2.5

```

It creates a `RoadData` with 3 GPS points and `totalDistance = 2.5 km`.

  

---

  

### Function: `parseTransit()`

  

**File:** `parser.cpp`

**Signature:** `vector<TransitData> parseTransit(string filename, int mode)`

**Input:** Path to a transit CSV file + which mode it is (Metro, Bikalpa Bus, or Uttara Bus)

**Output:** A list of `TransitData` objects — each holds the GPS coordinates of stops and the station names.

  

**What it does:** Same idea as `parseRoads()`, but for transit. The key difference is that transit CSV files have **station names** as the last two tokens (start station and end station).

  

**CSV format:**

```

route_id, lon1, lat1, lon2, lat2, ..., start_stop_name, end_stop_name

```

  

**Step-by-step walkthrough:**

  

```cpp

vector <TransitData> parseTransit(string filename, int mode){

    vector <TransitData> routes;

    ifstream file(filename);

    string line;

  

    while(getline(file, line)){

        if(line.empty()) continue;

  

        // Split by commas (same tokenizer as parseRoads)

        stringstream ss(line);

        string token;

        vector <string> tokens;

        while(getline(ss, token, ',')){

            tokens.push_back(token);

        }

  

        TransitData data;

        data.mode = mode;  // MODE_METRO, MODE_BIKALPA_BUS, or MODE_UTTARA_BUS

  

        // Last two tokens are station names

        data.endName = tokens.back();    // e.g., "Uttara North"

        tokens.pop_back();

        data.startName = tokens.back();  // e.g., "Motijheel"

        tokens.pop_back();

  

        // Remaining middle tokens = (lon, lat) pairs for each stop

        for (int i = 1; i < (int)tokens.size()-1; i += 2) {

            Point p;

            p.lon = stod(tokens[i]);

            p.lat = stod(tokens[i+1]);

            data.stops.push_back(p);

        }

        routes.push_back(data);

    }

    file.close();

    return routes;

}

```

  

**Example:** For a Metro CSV line:

```

201,90.3950,23.7510,90.3940,23.7650,Motijheel,Kamalapur

```

Result: `TransitData` with `mode = MODE_METRO`, `startName = "Motijheel"`, `endName = "Kamalapur"`, and 2 GPS stop coordinates.

  

After parsing, we have flat lists of road segments and transit routes. They're not connected to each other yet — that's the graph builder's job.

  

---

  

## 5. Step 2: Building the Road Layer

  

```cpp

// In main.cpp:

buildRoadLayer(nodes, roads);

```

  

This is where the raw road data becomes a **connected graph**.

  

---

  

### Helper Function: `getCreateNode()` — The Deduplication Engine

  

Before we look at `buildRoadLayer()`, we need to understand its helper function. This is the **most important utility** in graph building:

  

**File:** `graph.cpp`

**Signature:** `int getCreateNode(vector<Node>& nodes, map<pair<double,double>, int>& coordMap, Point p)`

**Input:** The node list, a coordinate lookup map, and a GPS point

**Output:** The ID of the node at that GPS point (either found existing or created new)

  

**Problem it solves:** When building the graph, many roads share the same intersection point. Without deduplication, you'd create duplicate nodes at the same location and the roads wouldn't be connected at intersections.

  

**How it works:**

  

```cpp

int getCreateNode(vector<Node>& nodes,

                  map<pair<double, double>, int>& coordMap,

                  Point p) {

    // Use (lat, lon) as a unique key

    pair<double, double> key = {p.lat, p.lon};

  

    // Does a node already exist at these exact coordinates?

    if (coordMap.find(key) != coordMap.end()) {

        return coordMap[key];  // Yes! Return the existing node's ID

    }

  

    // No — create a brand new node

    Node newNode;

    newNode.id = nodes.size();  // ID = current count (0, 1, 2, ...)

    newNode.pos = p;

    nodes.push_back(newNode);

    coordMap[key] = newNode.id; // Remember it so we don't duplicate

    return newNode.id;

}

```

  

**Example:** Road A has points `[P1, P2, P3]` and Road B has points `[P4, P2, P5]`. Both pass through `P2`. First call for `P2` (from Road A) creates node #42. Second call for `P2` (from Road B) finds #42 already exists and returns it. Now both roads **share node #42** — that's an intersection!

  

---

  

### Function: `buildRoadLayer()`

  

**File:** `graph.cpp`

**Signature:** `void buildRoadLayer(vector<Node>& nodes, vector<RoadData>& roads)`

**Input:** An empty node list + the parsed road data

**Output:** Modifies `nodes` in-place — adds ~46,000 nodes connected by car edges

  

**What it does:** Takes every road from the CSV data and turns each consecutive pair of GPS points into two nodes connected by a bidirectional car edge.

  

**Step-by-step walkthrough:**

  

```cpp

void buildRoadLayer(vector<Node>& nodes, vector<RoadData>& roads) {

    map<pair<double, double>, int> coordMap;  // GPS → node ID lookup

  

    for (auto& road : roads) {

        // For each pair of consecutive points on this road:

        for (int i = 0; i < road.points.size()-1; i++) {

  

            // Get or create nodes for both endpoints

            int u = getCreateNode(nodes, coordMap, road.points[i]);

            int v = getCreateNode(nodes, coordMap, road.points[i+1]);

  

            // Calculate real-world distance between them

            double dist = road.points[i].distanceTo(road.points[i+1]);

  

            // Add bidirectional car edges (roads are two-way)

            Edge e1 = {v, dist, MODE_CAR, ""};  // u → v

            Edge e2 = {u, dist, MODE_CAR, ""};  // v → u

            nodes[u].Adj.push_back(e1);

            nodes[v].Adj.push_back(e2);

        }

    }

}

```

  

**After this function runs:** The `nodes` vector contains ~46,000 nodes, each representing a road intersection or curve point. They are connected by **car-only edges**. This forms the **foundation layer** — the road network of Dhaka. But there's no public transit yet.

  

---

  

## 6. Step 3: Building Transit Layers

  

```cpp

// In main.cpp:

buildTransitLayer(nodes, metro);

buildTransitLayer(nodes, bikalpa);

buildTransitLayer(nodes, uttara);

```

  

This function is called **three times** — once for Metro Rail, once for Bikalpa Bus, once for Uttara Bus. Each call overlays a new transit network on top of the road layer.

  

---

  

### Function: `buildTransitLayer()`

  

**File:** `graph.cpp`

**Signature:** `void buildTransitLayer(vector<Node>& nodes, vector<TransitData>& transit)`

**Input:** The existing graph (already has road nodes) + parsed transit data

**Output:** Adds transit nodes and edges on top of the existing road graph

  

**What it does:** Creates nodes for each transit stop and connects consecutive stops with transit edges (Metro, Bikalpa Bus, or Uttara Bus). It also labels the first and last stops with their human-readable station names.

  

**Step-by-step walkthrough:**

  

```cpp

void buildTransitLayer(vector<Node>& nodes, vector<TransitData>& transit) {

    map<pair<double, double>, int> coordMap;

  

    // IMPORTANT: Import all existing road nodes into the lookup map

    // This way, transit stops that happen to fall on a road intersection

    // will REUSE the existing road node instead of creating a duplicate

    for (auto& n : nodes) {

        coordMap[{n.pos.lat, n.pos.lon}] = n.id;

    }

  

    for (auto& route : transit) {

        for (size_t i = 0; i + 1 < route.stops.size(); i++) {

  

            // Get or create nodes for consecutive stops

            int u = getCreateNode(nodes, coordMap, route.stops[i]);

            int v = getCreateNode(nodes, coordMap, route.stops[i+1]);

  

            // Label first stop and last stop with their names

            if (i == 0) {

                nodes[u].name = route.startName;  // e.g., "Motijheel"

            }

            if (i + 2 == route.stops.size()) {

                nodes[v].name = route.endName;     // e.g., "Uttara North"

            }

  

            // Calculate distance and add bidirectional transit edges

            double dist = route.stops[i].distanceTo(route.stops[i+1]);

            Edge e1 = {v, dist, route.mode, ""};  // u → v

            Edge e2 = {u, dist, route.mode, ""};  // v → u

            nodes[u].Adj.push_back(e1);

            nodes[v].Adj.push_back(e2);

        }

    }

}

```

  

**Key insight:** By importing existing road nodes into `coordMap` first, any transit stop that falls on an existing road intersection **automatically reuses that road node**. This means you can drive to that intersection AND board the Metro/bus from the same node — a natural multi-modal connection point.

  

**After all three calls:** The graph now has road edges (Car), Metro edges, Bikalpa Bus edges, and Uttara Bus edges. But many transit stops don't fall exactly on road intersections — they're "islands" disconnected from the road network. That's what the next step fixes.

  

---

  

## 7. Step 4: Linking Layers with Walk Edges

  

```cpp

// In main.cpp:

linkLayersWithWalk(nodes);

cout << "[System] Graph Built: " << nodes.size() << " nodes." << endl;

```

  

---

  

### Function: `linkLayersWithWalk()`

  

**File:** `graph.cpp`

**Signature:** `void linkLayersWithWalk(vector<Node>& nodes)`

**Input:** The complete graph (road + transit nodes)

**Output:** Adds walk edges between transit stops and their nearest road intersections

  

**Why we need it:** Most transit stops don't land exactly on a road intersection. Without walk edges, these stops would be isolated — the algorithm couldn't route you from a road to a Metro station. This function bridges that gap.

  

**Strategy:** For every **named** node (transit stop), find the nearest **unnamed** node (road intersection) and connect them with a bidirectional walk edge.

  

**Step-by-step walkthrough:**

  

```cpp

void linkLayersWithWalk(vector<Node>& nodes) {

    for (int i = 0; i < (int)nodes.size(); i++) {

        // Only process NAMED nodes (transit stops)

        // Unnamed nodes (name == "") are road intersections — skip them

        if (nodes[i].name == "") continue;

  

        double bestDist = 1e18;   // Start with "infinity"

        int bestRoadNode = -1;

  

        // Brute-force search: find the nearest UNNAMED node

        for (int j = 0; j < (int)nodes.size(); j++) {

            // Skip self and skip other named nodes (we want a road node)

            if (i == j || nodes[j].name != "") continue;

  

            double d = nodes[i].pos.distanceTo(nodes[j].pos);

            if (d < bestDist) {

                bestDist = d;

                bestRoadNode = j;

            }

        }

  

        // Create bidirectional walk edges

        if (bestRoadNode != -1) {

            Edge toRoad = {bestRoadNode, bestDist, MODE_WALK, ""};

            Edge toStop = {i, bestDist, MODE_WALK, ""};

            nodes[i].Adj.push_back(toRoad);            // Transit stop → Road

            nodes[bestRoadNode].Adj.push_back(toStop);  // Road → Transit stop

        }

    }

}

```

  

**Example:** The Metro station "Mirpur 10" is at GPS coordinates that don't match any road intersection. This function finds the closest road node (maybe 50 meters away) and creates a walk edge. Now the algorithm can route: *"Drive to that road node → Walk 50m to Mirpur 10 → Board Metro."*

  

**Walk edges are free** (0 Tk/km cost) but they take time (walking speed is 2 km/h). So they don't increase monetary cost but do affect time in Problems 5 and 6.

  

**At this point the graph is fully built:** ~46,669 nodes, connected by Car, Metro, Bikalpa Bus, Uttara Bus, and Walk edges. The program prints the node count and moves on to user input.

  

---

  

## 8. Step 5: User Input and Configuration

  

```cpp

// In main.cpp:

int problemNo;

Point startPt, endPt;

// ... (prompts for problem number, GPS coordinates, departure time) ...

  

ProblemConfig cfg = buildConfig(problemNo);

cfg.startTimeMinutes = (startH * 60) + startM;

```

  

After the graph is built, `main()` asks the user for: which problem (1–6), source GPS, destination GPS, departure time, and optionally a deadline (Problem 6).

  

Then it calls `buildConfig()` to set up the rules for the selected problem:

  

---

  

### Function: `buildConfig()`

  

**File:** `solver.cpp`

**Signature:** `ProblemConfig buildConfig(int problemNo)`

**Input:** Problem number (1–6)

**Output:** A fully configured `ProblemConfig` struct

  

**What it does:** This is the **control panel** of the system. It fills in all the values that control how the algorithm behaves — costs, speeds, schedules, and problem-specific overrides. By changing these values, the same Dijkstra algorithm produces different results for each problem.

  

**Step-by-step walkthrough:**

  

```cpp

ProblemConfig buildConfig(int problemNo) {

    ProblemConfig cfg;

    cfg.problemNumber = problemNo;

  

    // --- COST PER KM (in Taka) ---

    cfg.costPerKm[MODE_CAR]          = 20.0;   // Most expensive

    cfg.costPerKm[MODE_METRO]        = 5.0;    // Cheapest

    cfg.costPerKm[MODE_BIKALPA_BUS]  = 7.0;

    cfg.costPerKm[MODE_UTTARA_BUS]   = 10.0;

    cfg.costPerKm[MODE_WALK]         = 0.0;    // Free!

  

    // --- SPEED (km/h) ---

    cfg.speed[MODE_CAR]          = 20.0;

    cfg.speed[MODE_METRO]        = 15.0;

    cfg.speed[MODE_BIKALPA_BUS]  = 10.0;

    cfg.speed[MODE_UTTARA_BUS]   = 12.0;

    cfg.speed[MODE_WALK]         = 2.0;

  

    // --- SCHEDULES: {frequency_minutes, start_hour, end_hour} ---

    cfg.schedules[MODE_CAR]          = {0, 0, 24};    // Always available

    cfg.schedules[MODE_METRO]        = {5, 1, 23};    // Every 5 min, 1 AM–11 PM

    cfg.schedules[MODE_BIKALPA_BUS]  = {20, 7, 22};   // Every 20 min, 7 AM–10 PM

    cfg.schedules[MODE_UTTARA_BUS]   = {10, 6, 23};   // Every 10 min, 6 AM–11 PM

    cfg.schedules[MODE_WALK]         = {0, 0, 24};     // Always available

  

    // --- PROBLEM-SPECIFIC SPEED OVERRIDES ---

    if (problemNo == 4) {

        // Light traffic → all vehicles speed up to 30 km/h

        for (int i = 0; i < 4; i++) cfg.speed[i] = 30.0;

    } else if (problemNo == 5) {

        // Heavy congestion → all vehicles crawl at 10 km/h

        for (int i = 0; i < 4; i++) cfg.speed[i] = 10.0;

    }

  

    return cfg;

}

```

  

**Configuration summary table:**

  

| Parameter | Car | Metro | Bikalpa Bus | Uttara Bus | Walk |

|-----------|-----|-------|-------------|------------|------|

| **Cost (Tk/km)** | 20 | 5 | 7 | 10 | 0 |

| **Default Speed (km/h)** | 20 | 15 | 10 | 12 | 2 |

| **Frequency (min)** | Always | Every 5 | Every 20 | Every 10 | Always |

| **Operating Hours** | 24/7 | 1 AM – 11 PM | 7 AM – 10 PM | 6 AM – 11 PM | 24/7 |

  

**Why Problem 4 and 5 override speed:** Problem 4 simulates light evening traffic (everything faster at 30 km/h), while Problem 5 simulates rush-hour congestion (everything slower at 10 km/h). Speed doesn't directly change the cost formula, but it changes **time**, which affects which transit services are still available and how long you wait.

  

---

  

## 9. Step 6: Resolving GPS to Graph Nodes

  

```cpp

// In main.cpp:

int src = resolvePointer(startPt, nodes);

int dst = resolvePointer(endPt, nodes);

```

  

The user typed raw GPS coordinates (e.g., `23.8087, 90.3688`), but Dijkstra needs **node IDs** (e.g., node #1234). This function bridges that gap.

  

---

  

### Function: `resolvePointer()`

  

**File:** `solver.cpp`

**Signature:** `int resolvePointer(Point p, vector<Node>& nodes)`

**Input:** A raw GPS point + the full graph (may be modified)

**Output:** The node ID representing that GPS location. Returns `-1` on failure.

  

**What it does:** Maps a raw GPS coordinate to a graph node. It tries three strategies in order, from best to worst:

  

---

  

**Case A — Exact Match:**

Loop through all nodes. If any node is within 0.00001 km (~1 centimeter) of the input, use it directly. This handles the case where the user picked a coordinate that's already in the graph.

  

```cpp

double epsilon = 0.00001;  // ~1 cm tolerance

for (int i = 0; i < (int)nodes.size(); i++) {

    if (p.distanceTo(nodes[i].pos) < epsilon) {

        return i;  // Found an exact match!

    }

}

```

  

---

  

**Case B — Point on a Road:**

If no exact match, check if the GPS point falls on an existing road segment. The mathematical trick: if point `P` is on the road between nodes `U` and `V`, then `distance(U→P) + distance(P→V) ≈ distance(U→V)`.

  

If this holds (within 5-meter tolerance), create a **new node** at `P` and splice it into the road:

  

```cpp

for (int u = 0; u < (int)nodes.size(); u++) {

    for (auto& edge : nodes[u].Adj) {

        if (edge.mode != MODE_CAR) continue;  // only check road edges

  

        int v = edge.to;

        double dUP = nodes[u].pos.distanceTo(p);  // U → P

        double dPV = nodes[v].pos.distanceTo(p);  // P → V

        double dUV = edge.distance;                 // U → V

  

        // Triangle inequality check: if P is on segment UV,

        // then dUP + dPV should be approximately equal to dUV

        if (abs(dUP + dPV - dUV) < 0.005) {

            int newId = nodes.size();

            Node newNode;

            newNode.id = newId;

            newNode.pos = p;

            newNode.name = "Source/Dest (On Road)";

  

            // Connect new node to both endpoints of the road segment

            newNode.Adj.push_back({u, dUP, MODE_CAR, ""});

            newNode.Adj.push_back({v, dPV, MODE_CAR, ""});

            nodes.push_back(newNode);

  

            // Also connect the endpoints TO the new node

            nodes[u].Adj.push_back({newId, dUP, MODE_CAR, ""});

            nodes[v].Adj.push_back({newId, dPV, MODE_CAR, ""});

  

            return newId;

        }

    }

}

```

  

---

  

**Case C — Random Point (Off-Road):**

If the point isn't on any road, find the **nearest node in the entire graph** and create a walk edge to it. The user is somewhere off-road and has to walk to the nearest known location.

  

```cpp

int bestNode = -1;

double minPath = 1e18;

  

for (int i = 0; i < (int)nodes.size(); i++) {

    double d = nodes[i].pos.distanceTo(p);

    if (d < minPath) {

        minPath = d;

        bestNode = i;

    }

}

  

if (bestNode != -1) {

    int newId = nodes.size();

    Node newNode;

    newNode.id = newId;

    newNode.pos = p;

    newNode.name = "Source/Dest (Off road)";

  

    // Bidirectional walk edge to nearest node

    newNode.Adj.push_back({bestNode, minPath, MODE_WALK, ""});

    nodes.push_back(newNode);

    nodes[bestNode].Adj.push_back({newId, minPath, MODE_WALK, ""});

  

    return newId;

}

return -1;

```

  

**Example walkthrough:** Say the user enters GPS `(23.810, 90.370)`:

1. **Case A:** No node within 1 cm → skip

2. **Case B:** Checks all car edges, finds the point lies between node #500 and #501 on a road → creates node #46670, connects it to both → returns #46670

3. If Case B also failed, **Case C** would find the closest node (say #200, 30m away), create a walk edge → return new node

  

---

  

## 10. Step 7: Computing Edge Weights

  

Before we see Dijkstra itself, we need to understand the function it calls **for every edge** it considers:

  

---

  

### Function: `computeWeight()`

  

**File:** `solver.cpp`

**Signature:** `double computeWeight(Edge& e, double curTimeMin, ProblemConfig cfg)`

**Input:** An edge, the current clock time (minutes since midnight), and the problem config

**Output:** A weight value (distance in km, cost in Tk, or time in hours). Returns `-1.0` if the service is unavailable.

  

**Why this is the "heart" of the system:** Dijkstra's algorithm adds up edge weights along a path and minimizes the total. By returning **different things** for different problems, the same Dijkstra algorithm solves completely different problems:

- Problem 1: returns raw distance → minimizes **km**

- Problems 2/3/4/6: returns `distance × cost` → minimizes **Taka**

- Problem 5: returns `waitTime + travelTime` → minimizes **hours**

  

**Step-by-step walkthrough:**

  

**Step 1 — Check service availability (Problems 4–6 only):**

  

```cpp

int m = e.mode;

  

if (cfg.problemNumber >= 4) {

    int hour = (int)(curTimeMin / 60) % 24;

    if (hour < cfg.schedules[m].startHour ||

        hour > cfg.schedules[m].endHour) {

        return -1.0;  // Service not running right now → block this edge

    }

}

```

  

For Problems 1–3, schedules are ignored (all modes are always available). For Problems 4–6, if you try to take the Metro at midnight (Metro closes at 11 PM), this returns -1 and the algorithm skips that edge.

  

**Step 2 — Calculate wait time:**

  

```cpp

double waitMin = 0;

if (cfg.schedules[m].freqMinutes > 0) {

    int minSinceMidnight = (int)curTimeMin;

    int rem = minSinceMidnight % cfg.schedules[m].freqMinutes;

    if (rem != 0) {

        waitMin = cfg.schedules[m].freqMinutes - rem;

    }

}

```

  

If a bus comes every 20 minutes and you arrive at 7:14 AM: `rem = 14 % 20 = 14`, so `waitMin = 20 - 14 = 6 minutes`. You wait 6 minutes for the next bus at 7:20.

  

**Step 3 — Calculate travel time:**

  

```cpp

double travelHours = e.distance / cfg.speed[m];

double waitHours = waitMin / 60.0;

```

  

**Step 4 — Return the weight based on problem number:**

  

```cpp

switch (cfg.problemNumber) {

    case 1:

        return e.distance;                     // Minimize DISTANCE (km)

    case 2: case 3: case 4: case 6:

        return e.distance * cfg.costPerKm[m];  // Minimize COST (Taka)

    case 5:

        return waitHours + travelHours;         // Minimize TIME (hours)

    default:

        return e.distance;

}

```

  

**Worked example:** A 3 km Metro edge, current time 7:14 AM, Problem 3:

- Service check: Metro runs 1 AM–11 PM → ✅ OK (but not checked for P3 anyway)

- Wait: Metro every 5 min → `14 % 5 = 4` → wait = `5 - 4 = 1 min`

- Travel: `3 km / 15 km/h = 0.2 hours`

- **Weight (P3 = cost):** `3 × 5 = 15 Tk`

- If this were **P5 (time):** `1/60 + 0.2 = 0.217 hours ≈ 13 min`

  

---

  

## 11. Step 8: Running Dijkstra's Algorithm

  

```cpp

// In main.cpp:

vector<Route> result = solve(nodes, src, dst, cfg);

```

  

This is the big one — the core algorithm.

  

---

  

### Helper Function: `formatTime()` — Minutes-to-Clock Converter

  

Before we look at `solve()`, there's a small helper it uses to format times:

  

**File:** `solver.cpp`

**Signature:** `string formatTime(double totalMinutes)`

**Input:** Time as total minutes since midnight (e.g., 450)

**Output:** Human-readable time string (e.g., `"7:30 AM"`)

  

```cpp

string formatTime(double totalMinutes) {

    int totalMin = (int)totalMinutes;

    int h = (totalMin / 60) % 24;   // Hours (0-23), wrapped for past midnight

    int m = totalMin % 60;           // Minutes (0-59)

    string suffix = " AM";

  

    if (h >= 12) {

        suffix = " PM";

        if (h > 12) h -= 12;  // 13→1, 14→2, ..., 23→11

    }

    if (h == 0) h = 12;  // Midnight = 12 AM, not 0 AM

  

    stringstream ss;

    ss << h << ":" << setfill('0') << setw(2) << m << suffix;

    return ss.str();  // e.g., "7:30 AM"

}

```

  

**Examples:** `450 → "7:30 AM"` | `780 → "1:00 PM"` | `0 → "12:00 AM"` | `1425 → "11:45 PM"`

  

---

  

### Function: `solve()` — Dijkstra's Shortest Path

  

**File:** `solver.cpp`

**Signature:** `vector<Route> solve(vector<Node>& nodes, int src, int dst, ProblemConfig& cfg)`

**Input:** The full graph, source node ID, destination node ID, problem config

**Output:** A `vector<Route>` — an ordered list of journey steps from source to destination. Returns an **empty vector** if no path exists.

  

**What it does:** This is a **modified Dijkstra's shortest path algorithm** adapted for multi-modal transport. It finds the optimal route according to the problem's criteria (shortest distance, cheapest cost, fastest time, etc.).

  

The algorithm works in **3 phases:**

  

---

  

#### Phase 1: Initialization

  

Set up tracking arrays and push the source node into the priority queue:

  

```cpp

int n = nodes.size();

  

// For each node, track:

vector<double> minWeight(n, 1e18);  // Best weight found so far (starts at ∞)

vector<int> parent(n, -1);          // Which node we came from (-1 = none)

vector<int> parentMode(n, -1);      // Which mode we used to get here

vector<double> arrivalTime(n, 0);   // When we arrived at this node

  

// Min-heap priority queue — smallest weight always comes out first

priority_queue<DijkState, vector<DijkState>, greater<DijkState>> pq;

  

// Source: weight = 0, arrival time = user's departure time

minWeight[src] = 0;

arrivalTime[src] = cfg.startTimeMinutes;

pq.push({0, src, cfg.startTimeMinutes});

```

  

---

  

#### Phase 2: Main Dijkstra Loop

  

This loop runs until the priority queue is empty or we reach the destination. Each iteration:

1. Pop the node with the **smallest accumulated weight**

2. For each outgoing edge, check if using it **improves** the best known path to the neighbor

3. If yes, update and push the neighbor into the queue

  

```cpp

while (!pq.empty()) {

    DijkState current = pq.top();

    pq.pop();

  

    // OPTIMIZATION: Skip if we already found a better way to reach this node

    if (current.weight > minWeight[current.node]) continue;

  

    // EARLY EXIT: If we've reached the destination, we're done

    if (current.node == dst) break;

  

    // Explore every edge leaving the current node

    for (auto& edge : nodes[current.node].Adj) {

  

        // ─── MODE FILTERING ───

        // Problem 1: ONLY allow Car + Walk (skip bus and metro edges)

        if (cfg.problemNumber == 1 &&

            (edge.mode != MODE_CAR && edge.mode != MODE_WALK))

            continue;

  

        // Problem 2: ONLY allow Car + Metro + Walk (skip bus edges)

        if (cfg.problemNumber == 2 &&

            (edge.mode != MODE_CAR && edge.mode != MODE_METRO

             && edge.mode != MODE_WALK))

            continue;

  

        // Problems 3-6: All modes allowed — no filtering

  

        // ─── COMPUTE WEIGHT ───

        double w = computeWeight(edge, current.currentTime, cfg);

        if (w < 0) continue;  // Service unavailable at this time

  

        // ─── CALCULATE ARRIVAL TIME ───

        double travelTimeMin = (edge.distance / cfg.speed[edge.mode]) * 60.0;

        double waitTimeMin = 0;

        if (cfg.schedules[edge.mode].freqMinutes > 0) {

            int rem = (int)current.currentTime % cfg.schedules[edge.mode].freqMinutes;

            if (rem != 0)

                waitTimeMin = cfg.schedules[edge.mode].freqMinutes - rem;

        }

        double nextTime = current.currentTime + waitTimeMin + travelTimeMin;

  

        // ─── DEADLINE CHECK (Problem 6 only) ───

        if (cfg.problemNumber == 6 &&

            cfg.deadlineMinutes > 0 &&

            nextTime > cfg.deadlineMinutes)

            continue;  // Would arrive too late — skip this edge entirely

  

        // ─── RELAXATION ───

        // If this path to the neighbor is BETTER than anything found before:

        if (current.weight + w < minWeight[edge.to]) {

            minWeight[edge.to] = current.weight + w;  // Update best weight

            parent[edge.to] = current.node;            // Remember we came from here

            parentMode[edge.to] = edge.mode;            // Remember the mode used

            arrivalTime[edge.to] = nextTime;            // Record arrival time

            pq.push({minWeight[edge.to], edge.to, nextTime});  // Add to queue

        }

    }

}

```

  

**Why this works for all 6 problems:** Dijkstra guarantees finding the path with the **minimum total weight**. By changing what "weight" means (via `computeWeight()`), the same algorithm solves:

- P1: shortest distance

- P2/3/4/6: cheapest cost

- P5: fastest time

  

The **mode filtering** restricts which transport modes are available, and the **deadline check** prunes paths that would arrive too late in Problem 6.

  

---

  

#### Phase 3: Path Reconstruction

  

After Dijkstra finishes, the optimal path exists as **parent pointers** — each node knows which node it came from. We trace backwards from destination to source, then reverse to get the forward path:

  

```cpp

vector<Route> finalPath;

int curr = dst;

  

// Walk backwards: destination → ... → source

while (parent[curr] != -1) {

    int pNode = parent[curr];   // The node we came from

    Route step;

    step.sourceNode = pNode;

    step.destNode = curr;

    step.mode = parentMode[curr];

    step.endTime = formatTime(arrivalTime[curr]);

  

    // Find the actual edge to get distance and calculate cost

    for (auto& e : nodes[pNode].Adj) {

        if (e.to == curr && e.mode == step.mode) {

            step.distance = e.distance;

            step.cost = e.distance * cfg.costPerKm[e.mode];

            step.startTime = formatTime(arrivalTime[pNode]);

            break;

        }

    }

    finalPath.push_back(step);

    curr = pNode;  // Move one step closer to source

}

  

// We built the path BACKWARDS (dst→src), so flip it

reverse(finalPath.begin(), finalPath.end());

return finalPath;

```

  

**Full example walkthrough (Problem 3 — cheapest, all modes):**

  

Suppose source = node #100, destination = node #500, departure at 7:30 AM (450 min):

  

1. **Init:** `minWeight[100] = 0`, push `{weight:0, node:100, time:450}`

2. **Pop #100:** Has a car edge to #101 (1 km). Weight = `1 × 20 = 20 Tk`. Since `0 + 20 = 20 < ∞`, update #101

3. **Pop #101:** Has a walk edge to Metro station #200 (0.05 km). Weight = `0.05 × 0 = 0 Tk`. Since `20 + 0 = 20 < ∞`, update #200

4. **Pop #200:** Has a Metro edge to #300 (5 km). Weight = `5 × 5 = 25 Tk`. Since `20 + 25 = 45 < ∞`, update #300

5. ... continues until #500 is reached ...

6. **Reconstruct:** `#500 ← #300 ← #200 ← #101 ← #100`

7. **Reverse:** `[Car #100→#101, Walk #101→#200, Metro #200→#300, ... → #500]`

  

---

  

## 12. Step 9: Output and KML Generation

  

Back in `main()`, after `solve()` returns, the results are printed and a KML file is generated:

  

```cpp

// In main.cpp:

if (result.empty()) {

    cout << "[Result] NO PATH FOUND." << endl;

    return 0;

}

```

  

If a path was found, `main()` loops through each `Route` step and prints a line like:

  

```

7:30 AM - 7:45 AM, Cost: Tk 40.00: Car from Coord(90.37, 23.81) to Mirpur 10

7:50 AM - 8:10 AM, Cost: Tk 25.00: Metro from Mirpur 10 to Farmgate

```

  

It handles station names vs. coordinates — if a node has a name (like "Mirpur 10"), it uses that; otherwise, it falls back to showing the GPS coordinates.

  

Then it prints a summary:

```

Total Distance: 12.5 km

Total Cost:     285.00 Tk

Arrival Time:   8:10 AM

```

  

---

  

### Function: `generateKML()` — KML Map File Writer

  

**File:** `parser.cpp`

**Signature:** `void generateKML(const string& filename, const vector<Route>& path, const vector<Node>& nodes)`

**Input:** Output filename, the solved route, and all graph nodes (for GPS lookups)

**Output:** Writes a `.kml` file to disk

  

**What it does:** Creates a KML file (Keyhole Markup Language) that draws a colored line along the solved route. Upload this to **Google My Maps** to see the route on a real map of Dhaka.

  

**Step-by-step walkthrough:**

  

```cpp

void generateKML(const string& filename,

                 const vector<Route>& path,

                 const vector<Node>& nodes) {

    ofstream file(filename);

  

    // Write KML/XML boilerplate header

    file << "<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n";

    file << "<kml xmlns=\"http://www.opengis.net/kml/2.2\">\n";

    file << "<Document>\n";

    file << "  <name>Dhaka Route Path</name>\n";

  

    // Define line style: red, 4 pixels wide

    file << "  <Style id=\"routeStyle\">\n";

    file << "    <LineStyle>\n <color>ff0000ff</color> <width>4</width> </LineStyle>\n";

    file << "  </Style>\n";

  

    // Start the line

    file << "  <Placemark>\n";

    file << "    <LineString>\n";

    file << "      <coordinates>\n";

  

    // Write the FIRST point (source)

    if (!path.empty()) {

        file << nodes[path[0].sourceNode].pos.lon << ","

             << nodes[path[0].sourceNode].pos.lat << ",0\n";

    }

  

    // Write every subsequent waypoint

    for (const auto& step : path) {

        file << nodes[step.destNode].pos.lon << ","

             << nodes[step.destNode].pos.lat << ",0\n";

    }

  

    file << "      </coordinates>\n";

    // Close all XML tags

    file << "    </LineString>\n";

    file << "  </Placemark>\n";

    file << "</Document>\n";

    file << "</kml>";

    file.close();

}

```

  

**KML coordinate format:** `longitude,latitude,altitude` (altitude is always 0 since we're on the ground).

  

**Example output inside the KML file:**

```xml

<coordinates>

90.3688,23.8087,0

90.3700,23.8100,0

90.3950,23.7650,0

</coordinates>

```

This tells Google Maps to draw a line connecting these 3 points.

  

---

  

## 13. The Six Problems — Quick Reference

  

All six problems run the **same algorithm**. The only differences are: which edges are allowed, what `computeWeight()` returns, whether schedules are checked, and whether a deadline applies.

  

| Problem | What it Minimizes | Allowed Modes | Speed | Schedules? | Deadline? |

|---------|-------------------|---------------|-------|------------|-----------|

| **1** | Distance (km) | Car, Walk | 20 km/h | No | No |

| **2** | Cost (Tk) | Car, Metro, Walk | Default | No | No |

| **3** | Cost (Tk) | All | Default | No | No |

| **4** | Cost (Tk) | All | **30 km/h** | **Yes** | No |

| **5** | Time (hours) | All | **10 km/h** | **Yes** | No |

| **6** | Cost (Tk) | All | Default | **Yes** | **Yes** |

  

### Problem 1: Shortest Distance (Car Only)

*"I only have a car. What's the shortest route?"*

Weight = raw distance. Only car + walk edges. No schedule checks.

  

### Problem 2: Cheapest Cost (Car + Metro)

*"I can use my car or the Metro. What's the cheapest way?"*

Weight = distance × cost/km. Metro is 4× cheaper than driving, so the algorithm routes you to the Metro for long stretches.

  

### Problem 3: Cheapest Cost (All Modes)

*"I'll take anything. What's the absolute cheapest?"*

Same as Problem 2 but buses are now available too. Bikalpa Bus (7 Tk/km) and Uttara Bus (10 Tk/km) might offer cheaper alternatives on certain routes.

  

### Problem 4: Cheapest Cost with Light Traffic

*"It's a low-traffic evening. What's cheapest considering transit hours?"*

Same cost formula but all speeds are boosted to 30 km/h. Schedule checking is ON — services outside operating hours are blocked. Faster speeds mean you can reach more stops before they close.

  

### Problem 5: Fastest Time (Heavy Traffic)

*"Rush hour is terrible. What's the fastest way?"*

Weight = wait time + travel time. All speeds drop to 10 km/h (simulating congestion). The algorithm weighs whether waiting for a bus (that might come in 20 minutes) is worth it vs. crawling through traffic in a car.

  

### Problem 6: Cheapest with Deadline

*"I need to be there by 8 PM. What's the cheapest way that makes it on time?"*

Weight = cost (same as P2/3). Extra constraint: any edge whose arrival time exceeds the deadline is **blocked**. Among all feasible paths that arrive on time, it picks the cheapest.