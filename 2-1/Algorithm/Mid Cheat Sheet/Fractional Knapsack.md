Here is the comprehensive guide for **Greedy Algorithms: Fractional Knapsack & Variants**, containing the complete logic and C++ implementation for every variant.

---

# 🎒 Topic: Greedy Algorithms - Fractional Knapsack Family

## 1. The Base Problem: Fractional Knapsack (Maximize Value)

### **Problem Statement**

Given `N` items with `weight` and `value`, and a knapsack capacity `W`. Maximize total value. You can break items (take fractions).

### **Logic**

- **Strategy:** "Best Bang for the Buck."
    
- **Sort Criteria:** Descending order of **Ratio** ($\frac{Value}{Weight}$).
    
- **Process:** Take whole items while they fit. If the next item doesn't fit, take exactly enough of it to fill the remaining capacity.
    

### **C++ Implementation**

C++

```
#include <iostream>
#include <vector>
#include <algorithm>
#include <iomanip>

using namespace std;

struct Item {
    int id;
    int value, weight;
};

// Comparator: Sort by Value/Weight Ratio Descending
bool compareRatio(Item a, Item b) {
    double r1 = (double)a.value / a.weight;
    double r2 = (double)b.value / b.weight;
    return r1 > r2;
}

double fractionalKnapsack(int W, vector<Item>& items) {
    sort(items.begin(), items.end(), compareRatio);

    int curWeight = 0;
    double finalValue = 0.0;

    for (const auto& item : items) {
        // Case 1: Take Whole Item
        if (curWeight + item.weight <= W) {
            curWeight += item.weight;
            finalValue += item.value;
        }
        // Case 2: Take Fraction
        else {
            int remain = W - curWeight;
            finalValue += item.value * ((double)remain / item.weight);
            break; // Knapsack is full
        }
    }
    return finalValue;
}
```

---

## 2. Variant: Minimize Weight for Target Value

### **Problem Statement**

You need to gather **at least** a total value `V`. Minimize the total weight carried. Fractional selection is allowed.

### **Logic**

- **Strategy:** To minimize weight, we want the most "efficient" value items.
    
- **Sort Criteria:** **Same as base problem** (Descending Ratio $\frac{Value}{Weight}$). High density means less weight is needed to accumulate value.
    
- **Process:** Take items until `currentValue >= TargetV`. If the last item pushes us over the target, we only take the fraction needed to hit the target exactly.
    

### **C++ Implementation**

C++

```
double minWeightForTargetVal(int TargetV, vector<Item>& items) {
    // Sort by Density Descending
    sort(items.begin(), items.end(), compareRatio);

    double currentVal = 0;
    double finalWeight = 0.0;

    for (const auto& item : items) {
        // If taking the whole item doesn't exceed target (or just reaches it)
        if (currentVal + item.value <= TargetV) {
            currentVal += item.value;
            finalWeight += item.weight;
        } 
        // We only need a fraction to reach the exact target
        else {
            double neededVal = TargetV - currentVal;
            double fraction = neededVal / item.value;
            
            finalWeight += item.weight * fraction;
            currentVal = TargetV;
            break; 
        }
    }
    
    // Edge Case: If we took everything and still didn't reach TargetV
    if (currentVal < TargetV) return -1.0; 
    
    return finalWeight;
}
```

---

## 3. Variant: Maximize Number of Items (Lightest First)

### **Problem Statement**

Maximize the **count** of items you can fit in capacity `W`. Value is irrelevant. (Assumes whole items usually, but fractional version logic is trivial).

### **Logic**

- **Strategy:** To fit more items, pick small items first.
    
- **Sort Criteria:** Ascending order of **Weight**.
    
- **Process:** Iterate sorted list, add item if it fits.
    

### **C++ Implementation**

C++

```
bool compareWeightAsc(Item a, Item b) {
    return a.weight < b.weight;
}

int maxNumItems(int W, vector<Item>& items) {
    sort(items.begin(), items.end(), compareWeightAsc);
    
    int count = 0;
    int curWeight = 0;
    
    for (const auto& item : items) {
        if (curWeight + item.weight <= W) {
            curWeight += item.weight;
            count++;
        } else {
            break; // Cannot fit any heavier items
        }
    }
    return count;
}
```

---

## 4. Variant: Egyptian Fraction

### **Problem Statement**

Represent a fraction $N/D$ as a sum of unique unit fractions ($1/x$).

Example: $6/14 \rightarrow 1/3 + 1/11 + 1/231$.

### **Logic**

- **Strategy:** Always subtract the **largest possible unit fraction**.
    
- **Math:** The largest unit fraction $\le \frac{n}{d}$ is $\frac{1}{\lceil d/n \rceil}$.
    
- **Process:** Find ceiling of $d/n$, print it, compute remainder, recurse.
    

### **C++ Implementation**

C++

```
void printEgyptian(int n, int d) {
    // Base Case 1: Done
    if (d == 0 || n == 0) return;
    
    // Base Case 2: n divides d (e.g. 3/9 -> 1/3)
    if (d % n == 0) {
        cout << "1/" << d / n;
        return;
    }
    
    // Base Case 3: Improper fraction (e.g. 5/2 -> 2 + ...)
    if (n > d) {
        cout << n / d << " + ";
        printEgyptian(n % d, d);
        return;
    }
    
    // Greedy Step: 
    // We want 1/x <= n/d. The smallest x is ceiling(d/n).
    int x = (d / n) + 1;
    cout << "1/" << x << " + ";
    
    // Recurse: New fraction is (n/d) - (1/x)
    // = (nx - d) / (dx)
    printEgyptian(n * x - d, d * x);
}
```

---

## 5. Variant: Job Sequencing with Deadlines

### **Problem Statement**

Given jobs with `(Deadline, Profit)`. Each job takes 1 unit of time. Maximize total profit.

### **Logic**

- **Strategy:** Delay every job as much as possible to keep early time slots free for other jobs.
    
- **Sort Criteria:** Descending order of **Profit**.
    
- **Process:** For a job with deadline `d`, try to schedule it at time `d`. If `d` is occupied, try `d-1`, `d-2`, etc.
    

### **C++ Implementation**

C++

```
struct Job { char id; int dead; int profit; };

bool compareJobProfit(Job a, Job b) { return a.profit > b.profit; }

void jobSequencing(vector<Job>& jobs) {
    sort(jobs.begin(), jobs.end(), compareJobProfit);
    
    int n = jobs.size();
    // Find max deadline to determine schedule array size
    int maxDead = 0;
    for(auto j : jobs) maxDead = max(maxDead, j.dead);
    
    // slot[i] stores the Job ID scheduled at time i (1-based index)
    // Initialize with -1 (empty)
    vector<int> slot(maxDead + 1, -1);
    
    int totalProfit = 0;
    int jobsCount = 0;

    for (int i = 0; i < n; i++) {
        // Try to find a free slot starting from the deadline backwards
        for (int j = jobs[i].dead; j > 0; j--) {
            if (slot[j] == -1) {
                slot[j] = i; // Assign job index i to slot j
                totalProfit += jobs[i].profit;
                jobsCount++;
                break; 
            }
        }
    }
    
    cout << "Jobs Done: " << jobsCount << ", Profit: " << totalProfit << endl;
}
```

---

## 6. Exam Theory: Why Greedy Fails for 0/1 Knapsack

**Question:** Why can't we just use the Fractional Knapsack greedy strategy for 0/1 Knapsack?

**Answer:**

In **Fractional Knapsack**, the greedy choice property holds because we can fill any remaining "gap" with a fraction of the next best item. The resource (capacity) is never wasted.

In **0/1 Knapsack**, picking the highest density item might leave a gap that is too small for any other item, wasting capacity that could have been used by a combination of lower-density items to achieve a higher total value.

**Counter-Example:**

- **Capacity:** 50
    
- **Item A:** Val 60, Wt 10 (Ratio 6)
    
- **Item B:** Val 100, Wt 20 (Ratio 5)
    
- **Item C:** Val 120, Wt 30 (Ratio 4)
    

1. **Greedy:** Picks A (10kg). Remaining 40kg. Picks B (20kg). Remaining 20kg. C (30kg) **doesn't fit**.
    
    - Total Value: $60 + 100 = 160$.
        
2. **Optimal:** Skip A. Pick B (20kg) + C (30kg). Total Wt 50.
    
    - Total Value: $100 + 120 = 220$.