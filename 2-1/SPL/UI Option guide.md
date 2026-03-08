# UI & Visualization Ideas for the System Monitor

This document explores two big questions:

1. **How do we make the terminal version look better?** (Terminal-Based UI)
    
2. **How do we build a proper graphical app with charts and graphs?** (Visualization)
    

For the visualization part, we'll explore three paths — pure C, C++, and Python — and compare what each one can offer.

# Part 1: Terminal-Based UI

Right now, the menu uses plain `printf` and `scanf`. It works, but it doesn't _look_ like a real system monitor. The goal here is to make it look and feel more like `htop` — a live-updating, colorful, interactive terminal app.

## The Tool: ncurses

`ncurses` is a C library that gives you full control over what appears on the terminal screen. Instead of printing lines one after another (like `printf`), you can place text at _any position_ on the screen, use colors, draw borders, and update things in real time.

**Install it on Fedora:**

```
sudo dnf install ncurses-devel
```

**Compile with it:**

```
gcc -o monitor main.c core.c ... -lncurses -lpthread -lm
```

## What Would It Look Like?

Here's a rough idea of what the screen could look like:

```
┌─────────────────── System Monitor ────────────────────┐
│ CPU: [████████░░░░░░░░░░░░] 42%   Uptime: 3h 14m      │
│ MEM: [██████████████░░░░░░] 71%   Total: 16 GB        │
├───────────────────────────────────────────────────────┤
│ PID   │ Name           │ CPU %  │ MEM %  │ Status     │
│───────│────────────────│────────│────────│────────────│
│ 1842  │ firefox        │ 12.3%  │ 8.4%   │ HIGH       │
│ 2901  │ code           │ 5.1%   │ 4.2%   │ MEDIUM     │
│ 3055  │ spotify        │ 1.2%   │ 2.1%   │ MEDIUM     │
│ 1201  │ gnome-shell    │ 0.8%   │ 1.5%   │ LOW        │
│ 892   │ systemd        │ 0.0%   │ 0.3%   │ LOW        │
├───────────────────────────────────────────────────────┤
│ [S]uspend  [R]esume  [K]ill  [P]riority  [Q]uit       │
└───────────────────────────────────────────────────────┘
```

The whole screen refreshes every few seconds, colors change based on resource usage (red for high, green for low), and you can press single keys to take actions.

## How To Build This (Step by Step)

### Step 1: Basic ncurses Setup

Replace `printf` with ncurses functions:

```
#include <ncurses.h>

int main() {
    initscr();             // Take over the terminal
    cbreak();              // Don't wait for Enter key
    noecho();              // Don't show what user types
    curs_set(0);           // Hide the blinking cursor
    keypad(stdscr, TRUE);  // Enable arrow keys
    start_color();         // Enable colors

    // Define color pairs
    init_pair(1, COLOR_GREEN, COLOR_BLACK);  // Low usage
    init_pair(2, COLOR_YELLOW, COLOR_BLACK); // Medium usage
    init_pair(3, COLOR_RED, COLOR_BLACK);    // High usage

    // ... your display code here ...

    endwin();              // Give terminal back to normal
    return 0;
}
```

### Step 2: Drawing a Table

Use `mvprintw(row, col, "text")` to place text at exact positions:

```
void draw_header(int total_cpu, int total_mem) {
    mvprintw(1, 2, "CPU: [");
    
    // Draw a progress bar
    int bar_width = 20;
    int filled = (total_cpu * bar_width) / 100;
    for (int i = 0; i < bar_width; i++) {
        if (i < filled) addch(ACS_BLOCK); // Filled block
        else addch(' ');                  // Empty space
    }
    printw("] %d%%", total_cpu);
}
```

### Step 3: Real-Time Updates

Instead of using `scanf` (which waits forever), use `timeout()` to make `getch()` non-blocking:

```
timeout(100); // getch() will wait only 100ms, then return -1

while (1) {
    int key = getch();

    if (key == 'q') break;
    if (key == 'k') { /* kill selected process */ }

    // Redraw the screen with latest data
    clear();
    draw_header(cpu_percent, mem_percent);
    draw_process_table(app_list, app_count);
    refresh(); // Push changes to screen

    // The background thread is already updating data,
    // just read from the CSV or shared memory
}
```

### Step 4: Using Windows for Layout

ncurses lets you create separate "windows" (like panels) for different sections:

```
// Create a window: height, width, start_y, start_x
WINDOW *header_win = newwin(4, 60, 0, 0);
WINDOW *table_win = newwin(15, 60, 4, 0);
WINDOW *footer_win = newwin(3, 60, 19, 0);

// Draw a border around each window
box(header_win, 0, 0);
box(table_win, 0, 0);

// Print inside a specific window
mvwprintw(header_win, 1, 2, "CPU: 42%%");

// Refresh each window separately
wrefresh(header_win);
wrefresh(table_win);
```

## ncurses Summary

|   |   |
|---|---|
|**Aspect**|**Details**|
|**Language**|C (same as your project)|
|**Difficulty**|Medium — new functions to learn, but same language|
|**Install**|`sudo dnf install ncurses-devel`|
|**Look**|Text-based but colorful and interactive (like `htop`)|
|**Graphs**|Only bar charts made from text characters|
|**Good for**|Making the current project look professional|
|**Limits**|No real graphs (no line charts, no pie charts)|

# Part 2: Visualization & Graphical App

This is where we talk about building something that looks like the **GNOME System Monitor** — a window with tabs, line graphs showing CPU/memory history over time, colored bars, and a process list you can click on.

We'll explore this in three languages/approaches:

## Option 1: Pure C with GTK + Cairo

### What Is It?

**GTK** (GIMP ToolKit) is the same library that GNOME (your Fedora desktop) uses. The GNOME System Monitor you mentioned? It's built with GTK. So you'd be using the exact same building blocks.

**Cairo** is a drawing library that comes bundled with GTK. It lets you draw lines, shapes, and curves on a blank canvas — which is how you'd make graphs.

### What Would It Look Like?

You'd get a real desktop window with:

- A process list (like a spreadsheet/table)
    
- Line graphs showing CPU and memory usage over time
    
- Color-coded bars for each process
    
- Buttons to kill/pause/resume processes
    
- Tabs to switch between views
    

It would look like a simpler version of what you see when you open "System Monitor" on Fedora.

### How Does It Work?

GTK gives you ready-made "widgets" (building blocks):

- `GtkWindow` → the main window
    
- `GtkTreeView` → a table with sortable columns (for the process list)
    
- `GtkNotebook` → tabs (one for processes, one for graphs)
    
- `GtkDrawingArea` → a blank canvas where you draw graphs using Cairo
    
- `GtkButton` → buttons for kill/pause actions
    

For graphs, you draw on a `GtkDrawingArea` using Cairo:

```
// This function gets called whenever the graph needs to be redrawn
gboolean draw_cpu_graph(GtkWidget *widget, cairo_t *cr, gpointer data) {
    double *history = (double *)data; // Array of past CPU values

    // Background
    cairo_set_source_rgb(cr, 0.1, 0.1, 0.1); // Dark background
    cairo_paint(cr);

    // Draw the line
    cairo_set_source_rgb(cr, 0.2, 0.8, 0.2); // Green line
    cairo_set_line_width(cr, 2.0);

    int width = gtk_widget_get_allocated_width(widget);
    int height = gtk_widget_get_allocated_height(widget);

    for (int i = 0; i < 60; i++) { // Last 60 data points
        double x = (double)i / 60.0 * width;
        double y = height - (history[i] / 100.0 * height);

        if (i == 0) cairo_move_to(cr, x, y);
        else cairo_line_to(cr, x, y);
    }
    cairo_stroke(cr);

    return FALSE;
}
```

### Install and Compile

```
# Install GTK 3 development files
sudo dnf install gtk3-devel

# Compile
gcc -o monitor_gui gui.c $(pkg-config --cflags --libs gtk+-3.0) -lm
```

### Pros and Cons

|   |   |
|---|---|
|**Pros ✅**|**Cons ❌**|
|Stays in C — same language as your project|GTK can be confusing at first (lots of boilerplate code)|
|Native look on Fedora — uses the same style as your desktop|You have to draw graphs yourself using Cairo (no ready-made chart widget)|
|Fast — compiled C code|Callback-based programming feels different from what you've been writing|
|Free and well-documented|Error messages can be cryptic|
|Your existing C code can be linked directly|Learning curve is real — GTK is a big library|

### Difficulty Rating: ⭐⭐⭐⭐ (Hard)

The hardest part isn't the C — it's learning how GTK works. GTK uses an "event-driven" model (callback functions) that's different from the top-down `main()` style you're used to. You'd need to understand signals, callbacks, and the GTK main loop.

**Realistic for your level?** Possible, but it would take significant learning time. The graph-drawing part (Cairo) is the most challenging piece.

## Option 2: C++ with Qt + QCustomPlot

### What Is It?

**Qt** (pronounced "cute") is a C++ framework for building desktop apps. It's used by apps like VLC, KDE, and Telegram Desktop. It comes with a built-in design tool (Qt Designer) where you can drag and drop buttons, tables, and labels to build your layout visually.

**QCustomPlot** is a free add-on for Qt that gives you beautiful, ready-made charts and graphs — line plots, bar charts, real-time scrolling graphs — without having to draw anything yourself.

### What Would It Look Like?

Similar to the GNOME System Monitor, but with Qt's style. You'd have:

- A table showing all processes with sortable columns
    
- Real-time scrolling line graphs for CPU and memory
    
- Color-coded bars
    
- Buttons and menus
    
- Professional-looking by default (Qt handles fonts, spacing, colors automatically)
    

### How Does It Work?

Qt uses a "signal and slot" system that's easier to understand than GTK's callbacks:

```
// When the "Kill" button is clicked, call the kill_process function
connect(killButton, &QPushButton::clicked, this, &MainWindow::kill_process);
```

For graphs, QCustomPlot makes it very simple:

```
// Create a graph
QCustomPlot *cpuPlot = new QCustomPlot(this);
cpuPlot->addGraph();
cpuPlot->graph(0)->setPen(QPen(Qt::green)); // Green line

// Set up the axes
cpuPlot->xAxis->setLabel("Time (seconds)");
cpuPlot->yAxis->setLabel("CPU %");
cpuPlot->yAxis->setRange(0, 100);

// Update with new data every second
void MainWindow::updateGraph() {
    double currentTime = QDateTime::currentDateTime().toMSecsSinceEpoch() / 1000.0;
    double cpuUsage = getCurrentCpuUsage(); // Your function

    cpuPlot->graph(0)->addData(currentTime, cpuUsage);
    cpuPlot->xAxis->setRange(currentTime - 60, currentTime); // Show last 60 seconds
    cpuPlot->replot();
}
```

### Connecting Your Existing C Code

Since C++ can call C functions directly, you can keep all your existing data collection code and just add a C++ layer on top:

```
// In your C++ file, include the C header
extern "C" {
    #include "data_collector/data_collector.h"
}

// Now you can call your existing functions
ProcessInfo *list = NULL;
SystemCpuInfo sys;
int count = extract_processes(&list, &sys);
```

### Install and Compile

```
# Install Qt development tools
sudo dnf install qt5-qtbase-devel qt5-qtcharts-devel

# Or use Qt Creator (a full IDE for Qt development)
sudo dnf install qt-creator
```

You'd typically use **Qt Creator** (an IDE) to build Qt projects, which handles compilation for you. But you can also use `qmake` or `cmake`.

### Pros and Cons

|   |   |
|---|---|
|**Pros ✅**|**Cons ❌**|
|QCustomPlot gives you beautiful graphs with almost no effort|You need to learn some C++ (but basic C++ is not far from C)|
|Qt Designer lets you build the layout visually (drag and drop)|Qt is a very large framework — can feel overwhelming|
|C++ can call your existing C code directly|Build system is more complex (qmake/cmake instead of simple gcc)|
|Looks professional on every operating system|Download size is large (~1 GB for Qt SDK)|
|Huge community and lots of tutorials|Might feel like "too much" for an SPL project|
|Real-time updating graphs are easy to set up|—|

### Difficulty Rating: ⭐⭐⭐ (Medium-Hard)

Easier than GTK because QCustomPlot does the heavy lifting for graphs, and Qt Creator gives you a visual layout tool. The main challenge is learning basic C++ (classes, some object-oriented concepts) and understanding how Qt organizes code.

**Realistic for your level?** Yes, especially if you've seen any C++ before. Basic Qt apps don't require advanced C++ knowledge. The learning curve is smoother than GTK.

## Option 3: Python with Tkinter + Matplotlib (or PyQt)

### What Is It?

This approach uses Python for the entire visualization layer:

- **Tkinter** → Python's built-in GUI library (comes with Python, nothing to install)
    
- **Matplotlib** → The most popular Python charting library (line graphs, bar charts, pie charts, etc.)
    
- **psutil** → A Python library that reads system info (like your C code does, but in one line of Python)
    

Alternatively, you could use **PyQt5** (Python version of Qt) for a more polished look.

### What Would It Look Like?

A desktop window with:

- Live-updating line graphs (CPU, memory over time)
    
- A process table
    
- Buttons to control processes
    
- The graphs would look like what you're used to seeing in Jupyter notebooks or data science projects
    

### How Does It Work?

#### Approach A: Python reads your CSV files

Your C program still runs in the background, writing to CSV files every 4 seconds. The Python GUI just reads those CSVs and displays the data. This is the **simplest approach** because you don't need to change your C code at all.

```
import tkinter as tk
from matplotlib.figure import Figure
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg
import csv
import os

class SystemMonitorApp:
    def __init__(self, root):
        self.root = root
        self.root.title("System Monitor")

        # Create a matplotlib figure
        self.fig, (self.cpu_ax, self.mem_ax) = plt.subplots(2, 1, figsize=(8, 6))
        self.canvas = FigureCanvasTkAgg(self.fig, master=root)
        self.canvas.get_tk_widget().pack()

        # Start updating
        self.update_graphs()

    def read_aggregated_csv(self):
        apps = []
        with open('aggregated_data.csv', 'r') as f:
            reader = csv.DictReader(f)
            for row in reader:
                apps.append(row)
        return apps

    def update_graphs(self):
        apps = self.read_aggregated_csv()

        # Bar chart of CPU usage per app
        names = [a['App_Name'] for a in apps[:10]] # Top 10
        cpus = [float(a['CPU_Percent']) for a in apps[:10]]

        self.cpu_ax.clear()
        self.cpu_ax.barh(names, cpus, color='steelblue')
        self.cpu_ax.set_xlabel('CPU %')
        self.cpu_ax.set_title('Top Apps by CPU Usage')

        self.canvas.draw()

        # Refresh every 4 seconds
        self.root.after(4000, self.update_graphs)

if __name__ == "__main__":
    root = tk.Tk()
    app = SystemMonitorApp(root)
    root.mainloop()
```

#### Approach B: Python collects data itself using psutil

Skip the C data collection entirely and use Python's `psutil` library:

```
import psutil

# One line to get CPU usage
cpu_percent = psutil.cpu_percent(interval=1)

# One line to get memory info
mem = psutil.virtual_memory()
print(f"Memory: {mem.percent}%")

# Get all running processes
for proc in psutil.process_iter(['pid', 'name', 'cpu_percent', 'memory_percent']):
    print(proc.info)
```

This is dramatically simpler than reading from `/proc` in C. But of course, the point of your SPL project is that you _built_ the data collection yourself in C, which is much more impressive.

### Install

```
# Tkinter comes with Python, but just in case:
sudo dnf install python3-tkinter

# Install matplotlib and psutil
pip install matplotlib psutil
```

### Pros and Cons

|   |   |
|---|---|
|**Pros ✅**|**Cons ❌**|
|Fastest to build — Python is much quicker to write than C/C++|It's a different language — you need to know (or learn) Python|
|Matplotlib gives you every type of chart imaginable|Performance is slower than C (but for a UI, this doesn't really matter)|
|Huge ecosystem — tons of tutorials and examples|Mixing C backend + Python frontend can feel disjointed|
|Can just read your existing CSV files — no changes to C code|Two separate programs running at the same time|
|psutil makes data collection trivial (but this misses the point of SPL)|Tkinter looks a bit "old-fashioned" by default|
|Easy to add new visualizations — changing a graph is 2-3 lines|—|

### Difficulty Rating: ⭐⭐ (Easiest)

If you already know some Python, this is by far the easiest path. Even if you don't, Python's syntax is very beginner-friendly.

**Realistic for your level?** Absolutely. This is the quickest way to get a working graphical monitor with real charts.

### Making Tkinter Look Modern

If you choose this option, use **ttkbootstrap** instead of plain Tkinter to get a modern look:

```
pip install ttkbootstrap
```

```
import ttkbootstrap as ttk
from ttkbootstrap.constants import *

app = ttk.Window(title="System Monitor", themename="darkly")
# Now all your widgets automatically look modern and dark-themed
```

# Comparison Table: All Options Side by Side

|   |   |   |   |   |
|---|---|---|---|---|
|**Feature**|**ncurses (Terminal)**|**GTK + Cairo (C)**|**Qt + QCustomPlot (C++)**|**Python + Matplotlib**|
|**Language**|C|C|C++|Python|
|**Runs in**|Terminal|Desktop window|Desktop window|Desktop window|
|**Graphs**|Text-based bars only|Draw your own (hard)|Ready-made (easy)|Ready-made (easy)|
|**Process table**|Text columns|GtkTreeView widget|QTableWidget|Tkinter Treeview|
|**Real-time updates**|Yes (easy)|Yes (timer callbacks)|Yes (QTimer)|Yes (root.after)|
|**Looks like**|htop|GNOME System Monitor|KDE System Monitor|Custom dashboard|
|**Install size**|Tiny (< 1 MB)|Medium (~50 MB)|Large (~1 GB)|Medium (~100 MB)|
|**Time to build**|1-2 weeks|3-4 weeks|2-3 weeks|1 week|
|**Difficulty**|⭐⭐⭐|⭐⭐⭐⭐|⭐⭐⭐|⭐⭐|
|**Reuses your C code**|Directly|Directly|Via `extern "C"`|Reads CSV files|
|**Best for**|Quick upgrade to current project|Full native Fedora app|Best-looking graphs|Fastest results|

# Graph Ideas (What To Visualize)

No matter which option you pick, here are the graphs and visuals that would make the most sense for your data:

### 1. CPU Usage Over Time (Line Graph)

- **X-axis:** Time (last 60 seconds or 5 minutes)
    
- **Y-axis:** CPU percentage (0-100%)
    
- **Lines:** One line for total CPU, or one line per top app
    
- **Why:** This is what every system monitor shows first. It answers "is my computer busy right now, and was it busy a minute ago?"
    

### 2. Memory Usage Over Time (Line Graph)

- Same format as CPU but for memory
    
- Show a horizontal line at 80% or 90% as a "danger zone"
    

### 3. Top Apps by CPU/Memory (Horizontal Bar Chart)

- **Y-axis:** App names (top 10)
    
- **X-axis:** CPU % or Memory %
    
- **Color:** Green/Yellow/Red based on your clustering labels
    
- **Why:** Quick answer to "what's eating my resources?"
    

### 4. Resource Distribution (Pie Chart or Donut Chart)

- Show how total memory is split among top apps + "everything else"
    
- Nice for a dashboard overview
    

### 5. Cluster Visualization (Scatter Plot)

- **X-axis:** CPU %
    
- **Y-axis:** Memory %
    
- **Color:** One color per cluster (LOW/MEDIUM/HIGH)
    
- **Each dot:** One application
    
- **Why:** This directly shows the output of your K-Medoids clustering algorithm — you can literally _see_ the clusters
    

### 6. Process Count Per App (Stacked Bar)

- Shows how many child processes each app has spawned
    
- Firefox might have 20, VS Code might have 10, etc.
    

# Recommended Path

Here's what I'd suggest depending on your goals:

### "I just want my SPL project to look better"

→ Go with **ncurses**. It stays in C, directly upgrades your current menu, and makes the project look much more polished. You can build it in a week or two.

### "I want to show impressive graphs for my presentation or report"

→ Go with **Python + Matplotlib** reading your CSV files. Build the C backend as your main project, then whip up a Python script that makes beautiful charts from your data. You could have this working in a day or two.

### "I want to build a full desktop app as the next semester project"

→ Go with **Qt + QCustomPlot in C++**. This gives you the best-looking result with ready-made graphing tools, and learning basic C++ alongside it is a natural progression from C. It's a more ambitious project but very rewarding.

### "I want to go all-in and build something truly native"

→ Go with **GTK + Cairo in C**. This is the hardest path, but you'd be building with the same tools the GNOME desktop is built with. Most impressive for anyone who understands what GTK is, but also the steepest learning curve.

> **Bottom line:** For your current SPL-1 project, the best bang for your buck is **ncurses for the terminal** + **Python/Matplotlib for pretty graphs**. For a future project, consider Qt + C++ as it hits the sweet spot between effort and results.