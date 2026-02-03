Here is the comprehensive, self-contained guide for **Activity Scheduling & Interval Greedy Problems**, with C++ implementations for **every** major variant.

---

# 📅 Topic: Greedy Algorithms - Activity Scheduling & Intervals

## 1. The Base Problem: Activity Selection (Max Disjoint Intervals)

### **Problem Statement**

Given `N` activities, each with a `start` and `finish` time. You have **one resource**.

- **Goal:** Select the **maximum number** of non-overlapping activities.
    

### **Logic**

- **Strategy:** Always pick the activity that **finishes earliest**. This leaves the maximum remaining time for subsequent activities.
    
- **Sort:** Ascending order of **Finish Time**.
    

### **C++ Implementation**

C++

```
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

struct Activity {
    int start, finish;
};

bool compareFinish(Activity a, Activity b) {
    return a.finish < b.finish;
}

int maxActivities(vector<Activity>& acts) {
    if (acts.empty()) return 0;

    // 1. Sort by Finish Time
    sort(acts.begin(), acts.end(), compareFinish);
    
    int count = 1; 
    int last_finish = acts[0].finish;
    
    // 2. Iterate
    for (size_t i = 1; i < acts.size(); i++) {
        // If start time is after or equal to the previous finish time
        if (acts[i].start >= last_finish) {
            count++;
            last_finish = acts[i].finish;
        }
    }
    return count;
}
```

---

## 2. Variant: Weighted Job Scheduling (Max Profit)

### **Problem Statement**

Same as above, but each activity has a **Profit**.

- **Goal:** Maximize total profit of non-overlapping activities.
    
- **Why Greedy Fails:** A short job might finish early (good for greedy) but pay $1. A longer job might overlap it but pay $1000.
    

### **Logic**

- **Strategy:** **Dynamic Programming** + **Binary Search**.
    
- **Sort:** Ascending order of **Finish Time**.
    
- **State:** `dp[i]` = Max profit considering jobs `0` to `i`.
    
- **Transition:** `dp[i] = max(include, exclude)`.
    
    - `Exclude`: `dp[i-1]`.
        
    - `Include`: `profit[i] + dp[index_of_latest_non_conflicting]`.
        

### **C++ Implementation**

C++

```
struct Job {
    int start, finish, profit;
};

bool compareJob(Job a, Job b) {
    return a.finish < b.finish;
}

// Binary Search to find the latest job that finishes before jobs[i].start
int latestNonConflict(const vector<Job>& jobs, int i) {
    int low = 0, high = i - 1;
    int idx = -1;
    while (low <= high) {
        int mid = (low + high) / 2;
        if (jobs[mid].finish <= jobs[i].start) {
            idx = mid; // Potential candidate
            low = mid + 1; // Try to find a later one
        } else {
            high = mid - 1;
        }
    }
    return idx;
}

int maxWeightedProfit(vector<Job>& jobs) {
    sort(jobs.begin(), jobs.end(), compareJob);
    int n = jobs.size();
    
    vector<int> dp(n);
    dp[0] = jobs[0].profit;

    for (int i = 1; i < n; i++) {
        int include = jobs[i].profit;
        int prev = latestNonConflict(jobs, i);
        if (prev != -1) include += dp[prev];
        
        int exclude = dp[i-1];
        
        dp[i] = max(include, exclude);
    }
    return dp[n-1];
}
```

---

## 3. Variant: Meeting Rooms II (Min Resources)

### **Problem Statement**

Given intervals, find the **minimum number of rooms** required to schedule all meetings.

- (Equivalent to finding the maximum number of overlapping intervals at any point).
    

### **Logic**

- **Strategy:** Track the **end times** of meetings currently in rooms.
    
- **Sort:** Ascending order of **Start Time**.
    
- **Data Structure:** **Min-Heap** stores end times. Top of heap is the room that frees up earliest.
    
    - If `curr.start >= heap.top`: Reuse room (pop).
        
    - Always push `curr.end`.
        
    - Heap size is the answer.
        

### **C++ Implementation**

C++

```
#include <queue>

int minMeetingRooms(vector<vector<int>>& intervals) {
    if (intervals.empty()) return 0;
    
    // Sort by Start Time
    sort(intervals.begin(), intervals.end());
    
    // Min-Heap stores End Times of active meetings
    priority_queue<int, vector<int>, greater<int>> pq;
    
    pq.push(intervals[0][1]);
    
    for (size_t i = 1; i < intervals.size(); i++) {
        // If the earliest finishing meeting is done by the time this one starts
        if (intervals[i][0] >= pq.top()) {
            pq.pop(); // Room becomes free
        }
        // Add current meeting (either in the freed room or a new room)
        pq.push(intervals[i][1]);
    }
    
    return pq.size();
}
```

---

## 4. Variant: Interval Covering (Video Stitching)

### **Problem Statement**

Given a range `[0, T]` and a list of clips `[start, end]`.

- **Goal:** Select the **minimum number of clips** to cover the entire range.
    

### **Logic**

- **Strategy:** From the current covered point `curEnd`, select the clip that starts before `curEnd` and reaches the **farthest right**.
    
- **Sort:** Ascending order of **Start Time**.
    

### **C++ Implementation**

C++

```
int minClipsToCover(vector<vector<int>>& clips, int T) {
    sort(clips.begin(), clips.end()); // Sort by Start Time
    
    int count = 0;
    int i = 0;
    int curEnd = 0;
    int maxReach = 0;
    
    while (curEnd < T) {
        // Find all clips that can extend the current coverage
        // (i.e., they start before or at the point where we are currently stuck)
        while (i < clips.size() && clips[i][0] <= curEnd) {
            maxReach = max(maxReach, clips[i][1]);
            i++;
        }
        
        // If we cannot extend further but haven't reached T
        if (maxReach <= curEnd) return -1;
        
        curEnd = maxReach;
        count++;
    }
    return count;
}
```

---

## 5. Variant: Non-Overlapping Intervals (Min Removal)

### **Problem Statement**

Given a collection of intervals, find the **minimum number of intervals you need to remove** to make the rest non-overlapping.

- _Note:_ This is the exact dual of the Base Problem (Max Activity Selection). Maximize kept = Minimize removed.
    

### **Logic**

- **Strategy:** Keep the intervals that finish earliest (to leave room for others).
    
- **Sort:** Ascending order of **Finish Time**.
    
- **Count:** If overlap occurs, **remove** the current one (increment removal count). Else, update `last_end`.
    

### **C++ Implementation**

C++

```
int eraseOverlapIntervals(vector<vector<int>>& intervals) {
    if (intervals.empty()) return 0;

    // Sort by Finish Time
    sort(intervals.begin(), intervals.end(), [](const vector<int>& a, const vector<int>& b) {
        return a[1] < b[1];
    });

    int removed = 0;
    int last_end = intervals[0][1];

    for (size_t i = 1; i < intervals.size(); i++) {
        // If overlap (Start is strictly less than previous End)
        if (intervals[i][0] < last_end) {
            removed++; // We remove the current interval because it conflicts
            // We keep the previous one because it finished earlier (greedy choice)
        } else {
            last_end = intervals[i][1]; // No conflict, update end
        }
    }
    return removed;
}
```

---

## 6. Variant: Minimizing Lateness

### **Problem Statement**

Jobs have `duration` and `deadline`. Minimize the maximum lateness ($L_{max} = \max(0, \text{finish} - \text{deadline})$).

### **Logic**

- **Strategy:** **Earliest Deadline First (EDF)**.
    
- **Sort:** Ascending order of **Deadline**.
    

### **C++ Implementation**

C++

```
struct Task {
    int duration, deadline;
};

bool compareDeadline(Task a, Task b) {
    return a.deadline < b.deadline;
}

int minLateness(vector<Task>& tasks) {
    sort(tasks.begin(), tasks.end(), compareDeadline);
    
    int time_now = 0;
    int max_lateness = 0;
    
    for (auto& t : tasks) {
        time_now += t.duration;
        int lateness = max(0, time_now - t.deadline);
        max_lateness = max(max_lateness, lateness);
    }
    return max_lateness;
}
```

---

## 7. Variant: Meeting Rooms I (Can Attend All?)

### **Problem Statement**

Determine if a person can attend **all** meetings (i.e., check if there are **zero** overlaps).

### **Logic**

- **Strategy:** If even one pair overlaps, return false.
    
- **Sort:** Ascending order of **Start Time**.
    
- Check if `current.start < prev.end`.
    

### **C++ Implementation**

C++

```
bool canAttendMeetings(vector<vector<int>>& intervals) {
    // Sort by Start Time
    sort(intervals.begin(), intervals.end(), [](const vector<int>& a, const vector<int>& b) {
        return a[0] < b[0];
    });

    for (size_t i = 1; i < intervals.size(); i++) {
        // If current starts before previous ends -> Overlap
        if (intervals[i][0] < intervals[i - 1][1]) {
            return false;
        }
    }
    return true;
}
```