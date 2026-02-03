Here are the comprehensive notes for the **Job Selection (Activity Selection / Interval Scheduling)** family of problems. This is the "Hello World" of Greedy Algorithms regarding intervals.

---

# 📅 Topic: Greedy Algorithms - Job Selection (Interval Scheduling)

## 1. The Base Problem: Activity Selection (Max Events)

### **Problem Statement**

You are given `N` activities/jobs. Each job has a **Start Time** (`s`) and a **Finish Time** (`f`).

- **Constraint:** You can only work on one job at a time (non-overlapping).
    
- **Goal:** Select the **maximum number** of activities that can be performed.
    
- _Example:_
    
    - Job A: [1, 2]
        
    - Job B: [3, 4]
        
    - Job C: [0, 6]
        
    - Job D: [5, 7]
        
    - Job E: [8, 9]
        
    - Job F: [5, 9]
        
    - **Optimal:** A, B, D, E (Count 4). (Note: C and F are long duration and block others).
        

### **Intuition (The Greedy Choice)**

To maximize the _count_ of activities, we want to free up the resource (time) as early as possible for the next activity.

- **Bad Strategy:** Sort by Shortest Duration? (Fails: A short job might be in the middle of two other good jobs).
    
- **Bad Strategy:** Sort by Earliest Start? (Fails: A job starting at 0 might last until 100, blocking everything).
    
- **Correct Strategy:** Sort by **Earliest Finish Time**.
    
    - If you finish a job at time $t$, you maximize the remaining time $[t, \infty]$ for future jobs.
        

### **Algorithm Steps**

1. **Sort** all activities by **Finish Time** in ascending order.
    
2. Select the first activity (it finishes earliest).
    
3. Track the `last_finish_time`.
    
4. Iterate through the rest:
    
    - If `current_activity.start >= last_finish_time`, select it and update `last_finish_time`.
        

### **C++ Implementation**

C++

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

struct Job {
    int id;
    int start, finish;
};

// Sort by Finish Time Ascending
bool compareFinish(Job a, Job b) {
    return a.finish < b.finish;
}

int maxActivities(vector<Job>& jobs) {
    // 1. Sort
    sort(jobs.begin(), jobs.end(), compareFinish);
    
    int count = 1; // Always pick the first job
    int last_end = jobs[0].finish;
    
    // 2. Iterate
    for (size_t i = 1; i < jobs.size(); i++) {
        // If current job starts after (or at) the previous job finished
        if (jobs[i].start >= last_end) {
            count++;
            last_end = jobs[i].finish;
        }
    }
    return count;
}
```

---

## 2. Variant: Weighted Job Scheduling (Greedy Fails!)

### **Problem Statement**

Same as above, but each job has a **Profit/Weight**.

- **Goal:** Maximize **Total Profit** (not just the count of jobs).
    

### **Logic Shift**

- **Why Greedy Fails:** A job might finish very early (good for greedy) but have profit 1. A slightly longer job might overlap with it but have profit 100. Greedy would pick the cheap one and block the rich one.
    
- **Solution:** This becomes a **Dynamic Programming** problem ($O(N \log N)$).
    
- **Algorithm:**
    
    1. Sort by Finish Time.
        
    2. `dp[i] = max(include_job_i, exclude_job_i)`.
        
    3. `include_job_i = profit[i] + dp[index_of_latest_non_conflicting_job]`.
        
    4. Use **Binary Search** to find the non-conflicting job efficiently.
        

### **C++ Implementation (DP + Binary Search)**

C++

```
// Helper for Binary Search
// Find the latest job index < current_index that finishes before jobs[current_index].start
int binarySearch(const vector<Job>& jobs, int index) {
    int low = 0, high = index - 1;
    while (low <= high) {
        int mid = (low + high) / 2;
        if (jobs[mid].finish <= jobs[index].start) {
            if (jobs[mid + 1].finish <= jobs[index].start)
                low = mid + 1;
            else
                return mid;
        } else {
            high = mid - 1;
        }
    }
    return -1;
}

int maxWeightedProfit(vector<Job>& jobs) {
    sort(jobs.begin(), jobs.end(), compareFinish);
    int n = jobs.size();
    
    vector<int> dp(n);
    dp[0] = jobs[0].profit; // using struct Job {start, finish, profit}

    for (int i = 1; i < n; i++) {
        int include = jobs[i].profit;
        int l = binarySearch(jobs, i);
        if (l != -1) include += dp[l];
        
        int exclude = dp[i-1];
        
        dp[i] = max(include, exclude);
    }
    return dp[n-1];
}
```

---

## 3. Variant: Minimum Platforms (Railway Station)

### **Problem Statement**

Given the Arrival (`arr`) and Departure (`dep`) times of `N` trains.

- **Constraint:** A platform is occupied from arrival to departure.
    
- **Goal:** Find the **minimum number of platforms** required so that no train waits.
    

### **Logic Shift**

- We don't care about "which" train is on "which" platform. We only care about the **count of overlaps** at any instant.
    
- **Algorithm:**
    
    1. Sort `arr[]` and `dep[]` **separately**.
        
    2. Use two pointers (Sweep Line algorithm).
        
    3. If `arr[i] <= dep[j]`: A train arrived before the previous one left. We need +1 platform. Increment `i`.
        
    4. If `arr[i] > dep[j]`: A train left before the next one arrived. We free up -1 platform. Increment `j`.
        
    5. Track the maximum platforms needed at any moment.
        

### **C++ Implementation**

C++

```
int findPlatform(vector<int>& arr, vector<int>& dep) {
    sort(arr.begin(), arr.end());
    sort(dep.begin(), dep.end());
    
    int n = arr.size();
    int platforms_needed = 0;
    int max_platforms = 0;
    
    int i = 0, j = 0;
    
    while (i < n && j < n) {
        // If next event is an arrival
        if (arr[i] <= dep[j]) {
            platforms_needed++;
            i++;
        } 
        // If next event is a departure
        else {
            platforms_needed--;
            j++;
        }
        
        // Update global max
        if (platforms_needed > max_platforms) {
            max_platforms = platforms_needed;
        }
    }
    return max_platforms;
}
```

---

## 4. Variant: Meeting Rooms II (Min Heap Approach)

### **Problem Statement**

Given intervals of meetings `[start, end]`. Find minimum conference rooms required.

- This is conceptually identical to Minimum Platforms, but often solved using a **Min-Heap**.
    

### **Logic Shift**

- Sort meetings by **Start Time**.
    
- Use a Min-Heap to store the **End Times** of meetings currently running in rooms.
    
- For a new meeting:
    
    - Compare its `start` with the `min_end` (top of heap).
        
    - If `start >= min_end`: The earliest finishing meeting is over. We can reuse that room. **Pop** the heap.
        
    - **Push** the new meeting's `end` time into the heap.
        
- The size of the heap is the number of rooms required.
    

### **C++ Implementation**

C++

```
#include <queue>

int minMeetingRooms(vector<vector<int>>& intervals) {
    if (intervals.empty()) return 0;
    
    // Sort by Start Time
    sort(intervals.begin(), intervals.end(), [](const vector<int>& a, const vector<int>& b) {
        return a[0] < b[0];
    });
    
    // Min Heap stores End Times
    priority_queue<int, vector<int>, greater<int>> pq;
    
    // Add first meeting's end time
    pq.push(intervals[0][1]);
    
    for (size_t i = 1; i < intervals.size(); i++) {
        // If the room with earliest finish time is free
        if (intervals[i][0] >= pq.top()) {
            pq.pop(); // Reuse room (remove old end time)
        }
        // Add current meeting's end time (whether we reused or allocated new)
        pq.push(intervals[i][1]);
    }
    
    return pq.size();
}
```

---

## 5. Variant: Maximum Events You Can Attend (Daily Limit)

### **Problem Statement**

Given events `[start, end]`. You can attend a specific event on _any_ single day `d` where `start <= d <= end`.

- **Constraint:** You can only attend **one** event per day.
    
- **Goal:** Maximize events attended.
    

### **Logic Shift**

- **Greedy Strategy:** Sort events by **Start Time**.
    
- Use a **Min-Heap** to store the **End Times** of available events.
    
- Iterate through days `d` from 1 to MaxDay:
    
    1. Add all events that start on day `d` to the heap.
        
    2. Remove all events from heap that ended before day `d` (expired).
        
    3. Attend the event with the **earliest end time** (Top of Heap) because it's the most critical (will expire soonest). Pop it.
        
    4. Repeat.
        

### **C++ Implementation**

C++

```
int maxEvents(vector<vector<int>>& events) {
    // Sort by Start Time
    sort(events.begin(), events.end());
    
    priority_queue<int, vector<int>, greater<int>> pq; // Stores End Times
    int i = 0, n = events.size(), count = 0;
    
    // Iterate days. (Optimization: Jump to next event start if heap empty)
    for (int d = 1; d <= 100000; d++) {
        // 1. Add new events available today
        while (i < n && events[i][0] == d) {
            pq.push(events[i][1]);
            i++;
        }
        
        // 2. Remove expired events
        while (!pq.empty() && pq.top() < d) {
            pq.pop();
        }
        
        // 3. Attend event with earliest deadline
        if (!pq.empty()) {
            pq.pop();
            count++;
        }
        
        // Optimization: Break if no events left to process
        if (pq.empty() && i == n) break;
    }
    return count;
}
```