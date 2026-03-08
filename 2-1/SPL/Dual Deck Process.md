tags:

- "#SPL1"
    
- "#SystemMonitor"
    
- "#C"
    
- "#C++"
    
- "#Qt"
    
- "#Ncurses"
    
- "#Architecture"
    
    date: 2024-05-24
    
    aliases:
    
- Dual-Deck Architecture Master Plan
    

# 🌌 SPL-1 Master Plan: The Dual-Deck Architecture

> [!summary] The Vision
> 
> Combine a high-speed, hacker-style **Terminal UI (ncurses)** with a rich, professional **Graphical Dashboard (Qt)**.
> 
> **How it works (The Shared Data Bus):** We will keep the UI and the GUI as **two entirely separate processes**.
> 
> 1. **Process A (The Engine):** Your existing C code (`main.c`, `core.c`). It runs the ncurses terminal, collects data, runs K-Medoids, and writes to `clustered_report.csv`.
>     
> 2. **Process B (The Visualizer):** A lightweight C++ Qt app. It _only_ reads the CSV every few seconds and draws beautiful graphs using `QCustomPlot`.
>     

## 🏗️ Phase 1: Upgrading the C Core (Ncurses)

Your current `display/menu.c` uses `printf` and `scanf`, which blocks the terminal. We need to replace this with a real-time `ncurses` loop.

### 1. Prerequisites

Install ncurses on your Fedora system:

```
sudo dnf install ncurses-devel
```

### 2. Update `display/menu.c`

Rewrite your `display_menu()` function to use ncurses. It needs to be a non-blocking loop so the background threads initialized in `core.c` can keep running.

```c
// display/menu.c
#include <ncurses.h>
#include <unistd.h>
#include <signal.h>
#include <stdlib.h>
#include "menu.h"

pid_t gui_pid = -1; // Keeps track of the Qt GUI process ID

// Function to launch or close the Qt GUI
void toggle_gui() {
    if (gui_pid == -1) {
        gui_pid = fork();
        if (gui_pid == 0) {
            // Child process: launch the Qt app in the background
            // Redirecting output to /dev/null keeps ncurses clean
            execl("./gui/pulse-gui", "pulse-gui", (char *)NULL);
            exit(0);
        }
    } else {
        // GUI is already running, kill it safely
        kill(gui_pid, SIGTERM);
        gui_pid = -1;
    }
}

void start_ncurses_ui() {
    initscr();             // Start ncurses
    cbreak();              // Don't wait for Enter key
    noecho();              // Don't echo typed characters
    curs_set(0);           // Hide cursor
    timeout(1000);         // Non-blocking getch(): wait max 1 sec
    start_color();
    
    // Define color palettes
    init_pair(1, COLOR_GREEN, COLOR_BLACK);
    init_pair(2, COLOR_YELLOW, COLOR_BLACK);
    init_pair(3, COLOR_RED, COLOR_BLACK);

    while(1) {
        clear(); // Clear the screen for redraw

        // 1. Draw your UI (Read from your shared memory or structs)
        attron(COLOR_PAIR(1));
        mvprintw(1, 2, "=== SPL-1 SYSTEM MONITOR ===");
        attroff(COLOR_PAIR(1));
        
        mvprintw(3, 2, "Press [G] to toggle Visualizer | [Q] to Quit");
        mvprintw(4, 2, "[K] Kill Process | [S] Suspend");
        mvprintw(6, 2, "PID\tName\t\tCPU%%\tRAM%%\tCluster");
        mvprintw(7, 2, "--------------------------------------------------");

        // TODO: Loop through your app_list from data_aggregator.c and print the table
        // Example: mvprintw(row, col, "%d\t%s\t\t%.2f\t%.2f\t%s", pid, app_name, cpu, ram, cluster_label);

        refresh(); // Push memory buffer to terminal screen

        // 2. Handle User Input
        int ch = getch();
        if (ch == 'q' || ch == 'Q') break;
        if (ch == 'g' || ch == 'G') toggle_gui();
        // TODO: Add your kill/suspend logic here calling functions from process_control.c
    }

    // Cleanup before exiting
    if (gui_pid != -1) kill(gui_pid, SIGTERM); 
    endwin();
}
```

> [!warning] Silence the Engine!
> 
> Once ncurses is running, your background threads (`core.c`, `data_aggregator.c`, `clustering.c`) **cannot** use `printf`. If they print standard output, they will corrupt the ncurses screen. All background output must go silently into `clustered_report.csv` or `aggregated_data.csv`.

## 🎨 Phase 2: Building the Qt Visualizer (Deck 2)

This is a completely separate C++ project that sits in a subfolder (e.g., `gui/`).

### 1. Prerequisites

Install Qt development tools:

```
sudo dnf install qt5-qtbase-devel qt5-qtcharts-devel qt-creator
```

> [!tip] QCustomPlot
> 
> Download the `qcustomplot.h` and `qcustomplot.cpp` files from the [QCustomPlot website](https://www.qcustomplot.com/ "null") and drop them directly into your new `gui/` folder.

### 2. Qt Project Setup

Open Qt Creator, create a new "Qt Widgets Application" named `pulse-gui`. Place this entire project inside a `gui/` directory within your SPL-1 root directory.

### 3. The CSV Reader Timer (`mainwindow.cpp`)

The Qt app does zero system monitoring. It just parses what your C program wrote to the CSV.

```
#include "mainwindow.h"
#include "ui_mainwindow.h"
#include "qcustomplot.h"
#include <QTimer>
#include <QFile>
#include <QTextStream>

MainWindow::MainWindow(QWidget *parent) : QMainWindow(parent), ui(new Ui::MainWindow) {
    ui->setupUi(this);

    // Setup the plot (e.g., a Scatter plot to show your ML clusters)
    ui->customPlot->addGraph();
    ui->customPlot->graph(0)->setLineStyle(QCPGraph::lsNone);
    ui->customPlot->graph(0)->setScatterStyle(QCPScatterStyle(QCPScatterStyle::ssCircle, 8));
    
    ui->customPlot->xAxis->setLabel("CPU Usage (%)");
    ui->customPlot->yAxis->setLabel("Memory Usage (%)");
    
    // Read the CSV every 2 seconds (matching your C thread cycle)
    QTimer *timer = new QTimer(this);
    connect(timer, &QTimer::timeout, this, &MainWindow::readCSVAndUpdate);
    timer->start(2000); 
}

void MainWindow::readCSVAndUpdate() {
    // Look for the file generated by your C program one directory up
    QFile file("../clustered_report.csv"); 
    if (!file.open(QIODevice::ReadOnly | QIODevice::Text)) return;

    QTextStream in(&file);
    QVector<double> xData, yData;
    
    // Skip header line
    if (!in.atEnd()) in.readLine(); 

    while (!in.atEnd()) {
        QString line = in.readLine();
        QStringList fields = line.split(",");
        
        // Ensure we have enough columns (Adjust index based on your actual CSV structure)
        if (fields.size() >= 4) { 
            // Assume format: App_Name, CPU_Percent, Memory_Percent, Cluster_Label
            double cpu = fields[1].toDouble();
            double ram = fields[2].toDouble();
            xData.append(cpu);
            yData.append(ram);
            
            // Advanced: You can add multiple graphs and route data based on fields[3] 
            // to color code LOW (Green), MEDIUM (Yellow), HIGH (Red)
        }
    }
    file.close();

    // Update the plot with new data
    ui->customPlot->graph(0)->setData(xData, yData);
    ui->customPlot->rescaleAxes();
    ui->customPlot->replot();
}
```

> [!info] The ML Visualization Advantage
> 
> Because you implemented K-Medoids, the absolute best visualization is a **Scatter Plot**.
> 
> - X-axis = CPU
>     
> - Y-axis = Memory
>     
> 
> By coloring the dots based on the cluster label (Green=Low, Red=High), you visually _prove_ your algorithm works directly on the screen!

## ⚙️ Phase 3: Integration & Compilation

Because these are two different languages/ecosystems, you compile them separately but run them together.

### Step 1: Compile the GUI (Run once, or upon GUI changes)

Navigate into the `gui/` folder and build the Qt app.

```
cd gui
qmake pulse-gui.pro
make
cd ..
```

_(This produces the `gui/pulse-gui` executable)._

### Step 2: Compile the Engine

Update your root `Makefile` (or compilation script) to link the ncurses library.

```
gcc -o monitor main.c core.c cluster/clustering.c controller/process_control.c data_collector/*.c display/menu.c -lncurses -lpthread -lm
```

### Step 3: Run the Architecture

```
./monitor
```

1. Your terminal transforms into a dynamic ncurses UI.
    
2. The C program's threads silently write data to `clustered_report.csv`.
    
3. You press `G` in the terminal.
    
4. The Qt window launches, reads the CSV, and displays your K-Medoids clusters dancing on the screen in real-time.
    

## ✅ Implementation Checklist

- [ ] **1. Sanitize C Output.** Audit `core.c`, `data_aggregator.c`, and `clustering.c`. Comment out all `printf` statements in background threads to prevent ncurses corruption.
    
- [ ] **2. Ncurses Hello World.** Replace `display_menu()` in `display/menu.c` with a minimal ncurses loop to verify compilation with `-lncurses`.
    
- [ ] **3. Port Table UI.** Recreate your process list inside the ncurses loop using `mvprintw()`.
    
- [ ] **4. Fork Logic.** Add the `toggle_gui()` function with `fork()` and `execl()` to `menu.c` bound to the `G` key.
    
- [ ] **5. Qt Setup.** Create the `gui/` directory and initialize the Qt project via Qt Creator.
    
- [ ] **6. Integrate QCustomPlot.** Drop `qcustomplot.h/cpp` into the GUI project and configure a basic scatter graph.
    
- [ ] **7. CSV Parser.** Write the `QTimer` logic in C++ to read `../clustered_report.csv` and map the data to the graph axes.
    
- [ ] **8. Polish & Presentation.** Color-code the ncurses table rows and Qt scatter plot dots based on the K-Medoids cluster labels.