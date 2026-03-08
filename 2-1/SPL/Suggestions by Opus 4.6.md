# SPL-1 Codebase Review: Linux System Monitor

  

## What Does This Project Do?

  

This project is a **Linux System Monitor** — a tool that runs in the background, watches what every process on your computer is doing (how much CPU and memory each app uses), and lets you control those processes (pause, kill, change priority) through a simple terminal menu.

  

Think of it like building your own version of `htop` or the Windows Task Manager, but from scratch in C.

  

### How It Works (The Big Picture)

  

```

┌──────────────────────────────────────────────────────┐

│ main.c │

│ Starts the background thread + launches the menu │

└───────────┬──────────────────────────┬───────────────┘

│ │

┌────────▼────────┐ ┌─────────▼─────────┐

│ core.c │ │ display/menu.c │

│ Background Loop │ │ User Interface │

│ (runs every 4s) │ │ (waits for input) │

└────────┬─────────┘ └─────────┬──────────┘

│ │

┌────────▼──────────────┐ ┌────────▼──────────────┐

│ data_collector/ │ │ controller/ │

│ - data_extractor.c │ │ process_control.c │

│ - resource_calc.c │ │ (kill/pause/resume) │

│ - data_aggregator.c │ └───────────────────────┘

└────────┬──────────────┘

│

┌────────▼──────────────┐

│ cluster/ │

│ clustering.c │

│ (groups apps by │

│ resource usage) │

└───────────────────────┘

```

  

Every 4 seconds, the background thread:

1. Takes a **snapshot** of all running processes (reads from `/proc`)

2. Compares it to the **previous snapshot** to calculate CPU usage

3. Groups processes into **applications** (e.g., all Chrome tabs = one "Chrome" entry)

4. Runs a **clustering algorithm** to label apps as LOW / MEDIUM / HIGH impact

5. Writes everything to CSV files so you can see the data

  

---

  

## Review of Current Progress

  

### What's Working Well ✅

  

**1. Clean Modular Structure**

The project is nicely split into folders that each handle one job:

- `data_collector/` → reads process data from the system

- `controller/` → sends signals to processes

- `display/` → the menu the user sees

- `cluster/` → the machine learning part

  

This is good software design. Each piece can be understood and tested on its own.

  

**2. Solid Data Collection from `/proc`**

The `data_extractor.c` file reads from multiple `/proc` files for each process:

- `/proc/[pid]/stat` → CPU time (utime, stime), parent PID, session ID, start time

- `/proc/[pid]/status` → Memory (VmRSS), User ID

- `/proc/[pid]/smaps` → Proportional Set Size (PSS — a more accurate memory metric)

- `/proc/[pid]/cmdline` → The real application name

  

This is thorough. Most student projects only read one or two of these. PSS in particular is a smart choice because it properly handles shared memory between processes.

  

**3. Smart "Sliding Window" CPU Calculation**

CPU usage can't be measured from a single snapshot — you need to compare two snapshots taken at different times. The code does this correctly:

  

```

Snapshot A ──(wait 4 seconds)──> Snapshot B

CPU% = (Jiffies_B - Jiffies_A) / (System_Uptime_B - System_Uptime_A) × 100

```

  

This is the correct approach and matches how real tools like `top` do it.

  

**4. Application Grouping (Aggregation)**

The `data_aggregator.c` walks up the process tree to find each process's "root" application. It handles edge cases like:

- System processes (PID 1)

- Orphaned processes (parent doesn't exist)

- Shell boundaries (stops at `bash`/`zsh` so terminal-launched apps aren't grouped under the shell)

- User boundaries (stops when the owner changes)

  

This is clever logic and shows good understanding of how Linux processes work.

  

**5. Hash Map for Fast Lookups**

Instead of comparing every process against every other one (which would be very slow), `resource_calculator.c` builds a hash map from the first snapshot. This lets you find an old process by its PID + start time in nearly constant time (O(1) average). The hash function used is solid — it's based on Murmur-like mixing which gives good distribution.

  

**6. K-Medoids Clustering**

The clustering module groups apps into 3 buckets (HIGH_IMPACT, MEDIUM_IMPACT, LOW_IMPACT) using K-Medoids, which is a real machine learning algorithm. K-Medoids is a better choice here than K-Means because it's more resistant to outliers (one process using 100% CPU won't ruin the averages).

  

**7. Thread Safety**

The code uses a `pthread_mutex_t` to make sure the background thread and the menu don't step on each other's toes when writing/reading CSV files.

  

---

  

### Things That Could Be Better 🔧

  

**1. The Background Thread Never Stops Cleanly**

  

Right now, the monitoring thread runs in an infinite `while(1)` loop and there's no way to tell it to stop. When you press "Exit" in the menu, the main function just returns and the OS kills the thread.

  

- **Why this matters:** This can leave CSV files half-written if the thread was in the middle of writing when the program exits.

- **How to fix it:**

Add a shared "flag" variable that both the thread and the menu can see:

  

```c

// In core.h

extern int should_stop;

  

// In the thread loop (core.c), change while(1) to:

while (!should_stop) {

// ... existing code ...

}

  

// In menu.c, when user picks Exit:

should_stop = 1;

pthread_join(monitor_thread, NULL); // Wait for thread to finish

```

  

**2. Memory Leak in the Aggregator's Lookup Tables**

  

`data_aggregator.c` creates two big arrays (`root_cache` and `pid_lookup`) using `malloc()` but never calls `free()` on them. Every time `aggregate_live_data()` runs (every 4 seconds), it checks if they're NULL and creates them if needed — but since they're never freed, the check always succeeds after the first run, so there's no growth. However, the memory is still leaked when the program exits.

  

- **How to fix it:**

Add a cleanup function:

  

```c

void cleanup_aggregator(void) {

free(root_cache); root_cache = NULL;

free(pid_lookup); pid_lookup = NULL;

}

```

  

Call it before the program exits.

  

**3. No Error Feedback in the Menu**

  

When you enter a PID and the process doesn't exist, the `kill()` call fails and `perror` prints the error. But there's no feedback about *what you can do* about it. Also, the menu doesn't show the user which PIDs are available.

  

- **How to fix it:**

Add a new menu option to display the current top processes with their PIDs (read from one of the CSV files):

  

```c

// Add as a new menu option, e.g., "0. Show Running Apps"

// Read aggregated_data.csv and print top entries

```

  

**4. Duplicate Function Declaration in the Header**

  

In `data_collector.h`, the function `aggregate_live_data()` is declared twice (lines 103-107 and 115-121). This technically works because the signatures match, but it's confusing.

  

- **How to fix it:** Simply delete one of the two declarations.

  

**5. CSV Files Are Written to the Current Directory**

  

The program writes `raw_data.csv`, `resource.csv`, `aggregated_data.csv`, and `clustered_report.csv` into whatever directory you happen to run the program from. This is fragile.

  

- **How to fix it:**

Define a fixed output path, or use a subfolder:

  

```c

#define OUTPUT_DIR "./monitor_output/"

// Then use: snprintf(path, sizeof(path), "%s%s", OUTPUT_DIR, "raw_data.csv");

```

  

**6. The `assignments` Array in Clustering Is Used Before Being Initialized**

  

In `clustering.c`, the `assignments` array is allocated with `malloc()` (which gives random garbage values), but then on line 40, it checks `if (assignments[i] != best_cluster)` — comparing against garbage on the first iteration.

  

- **How to fix it:**

Use `calloc()` instead of `malloc()` to initialize everything to zero:

  

```c

int *assignments = calloc(count, sizeof(int));

```

  

---

  

## Suggested Improvements

  

### Short-Term (Easy to Add)

  

#### 1. Add a Makefile

  

Right now there's no build system. Anyone who downloads this project has to guess how to compile it.

  

**Why it matters:** It's the first thing anyone (or a professor) will try to do.

  

**How to do it — create a file called `Makefile`:**

  

```makefile

CC = gcc

CFLAGS = -Wall -g

LDFLAGS = -lpthread -lm

  

SRCS = main.c core.c \

data_collector/data_extractor.c \

data_collector/data_aggregator.c \

data_collector/resource_calculator.c \

display/menu.c \

controller/process_control.c \

cluster/clustering.c

  

TARGET = system_monitor

  

$(TARGET): $(SRCS)

$(CC) $(CFLAGS) -o $(TARGET) $(SRCS) $(LDFLAGS)

  

clean:

rm -f $(TARGET) *.csv system_info.txt

```

  

Then you just type `make` to build.

  

---

  

#### 2. Add a "Show Top Apps" Option to the Menu

  

Let the user see what's running before they have to type a PID.

  

- Read `aggregated_data.csv` and print the top 10 entries sorted by CPU or memory

- This makes the whole tool much more usable

  

---

  

#### 3. Add Timestamps to CSV Files

  

Currently, each CSV is overwritten every 4 seconds with the latest data. If something interesting happens, the old data is gone forever.

  

- Add a timestamp column to each row, or

- Save snapshots to a running log file (append mode instead of overwrite)

  

---

  

### Medium-Term (Good Learning Opportunities)

  

#### 4. Add a Simple Sorting Algorithm

  

The aggregated data and resource data are currently unsorted. Adding a sort-by-CPU or sort-by-memory feature would be useful and is a great exercise.

  

**Hint:** You can use a simple bubble sort or the built-in `qsort()` function from `<stdlib.h>`:

  

```c

int compare_by_cpu(const void *a, const void *b) {

const AppSummary *x = (const AppSummary *)a;

const AppSummary *y = (const AppSummary *)b;

if (y->cpu_percentage > x->cpu_percentage) return 1;

if (y->cpu_percentage < x->cpu_percentage) return -1;

return 0;

}

  

// Then before writing to CSV:

qsort(list, count, sizeof(AppSummary), compare_by_cpu);

```

  

---

  

#### 5. Add a Config File Instead of Hardcoded Values

  

The monitoring interval (4 seconds), the number of clusters (3), and the impact thresholds (15%, 2%) are all hardcoded. Moving these to a simple config file makes the tool much more flexible.

  

**How to do it — a simple `config.txt` file:**

  

```

interval=4

clusters=3

high_threshold=15.0

medium_threshold=2.0

```

  

Then read it at startup with `fscanf()`.

  

---

  

#### 6. Better Process Tree Handling

  

The current aggregator walks up the tree from child to parent, but it could also show the tree visually in the output — like `pstree` does. This would be a great way to practice **recursive functions**.

  

---

  

### Long-Term / Advanced (Future Features)

  

#### 7. Historical Data and Trend Analysis

  

Right now, the tool only shows "what's happening right now." A natural next step is to keep data over time and show trends:

- Save each snapshot to a log file with timestamps

- Later, read the log and show average CPU over the last 5 minutes, or detect if memory usage is growing steadily (possible memory leak)

  

This would be a natural way to practice **file I/O** and basic **statistics** (averages, moving averages).

  

---

  

#### 8. Alerts / Notifications

  

Instead of the user manually checking for high-resource apps, the monitor could print a warning when something crosses a threshold:

  

```

⚠️ WARNING: "firefox" is using 85% of memory!

```

  

This is straightforward to add in the main loop — just check after clustering and print if any app is in the HIGH_IMPACT cluster.

  

---

  

#### 9. Network Monitoring

  

Linux exposes network stats in `/proc/net/` and per-process data in `/proc/[pid]/net/`. Adding network usage (bytes sent/received) would make this a more complete system monitor.

  

---

  

#### 10. Terminal UI (ncurses)

  

The current menu uses plain `printf` and `scanf`. A big upgrade would be to use the `ncurses` library to create a real-time updating display — like `htop`. This is a significant project on its own, but it would be a very impressive addition.

  

---

  

## Summary Table

  

| Area | Status | Priority |

|------|--------|----------|

| Modular structure | ✅ Well done | — |

| Data collection from `/proc` | ✅ Thorough | — |

| CPU calculation (sliding window) | ✅ Correct approach | — |

| App grouping (aggregation) | ✅ Smart logic | — |

| Hash map for fast lookups | ✅ Good implementation | — |

| K-Medoids clustering | ✅ Working | — |

| Thread safety with mutex | ✅ Present | — |

| Clean thread shutdown | ❌ Missing | High |

| Memory cleanup on exit | ❌ Missing | High |

| Build system (Makefile) | ❌ Missing | High |

| Sorting output | ❌ Not implemented | Medium |

| Show running apps in menu | ❌ Not implemented | Medium |

| Uninitialized array in clustering | ⚠️ Bug | High |

| Duplicate header declaration | ⚠️ Minor issue | Low |

| Config file | ❌ Not implemented | Low |

| Timestamps in CSV | ❌ Not implemented | Medium |

  

---

  

> **Overall:** This is a strong project for a second-year student. The core logic — reading `/proc`, calculating CPU with a sliding window, grouping processes by application, and clustering — is all correctly done and shows real understanding of operating systems concepts. The areas for improvement are mostly about "polish" (clean shutdown, build system, error handling) rather than fundamental problems. Good work! 👏