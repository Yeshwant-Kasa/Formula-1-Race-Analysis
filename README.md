# F1 Race Replay 🏎️ 🏁

A Python application for visualizing Formula 1 race telemetry and replaying race events with interactive controls and a graphical interface.

> **Original Project by:** [Tom Shaw (IAmTomShaw)](https://github.com/IAmTomShaw/f1-race-replay)  
> **YouTube Tutorial:** [Watch on YouTube](https://youtu.be/TiQEElXyY2w?si=gUBvVcXAz7JRe1O1)
<img width="2553" height="1436" alt="Mexico GrandPrix 2025 RESULT Qualifying" src="https://github.com/user-attachments/assets/b15eae28-90b6-4f25-9d28-f234dd50f6f9" />



## 📋 Table of Contents

- [Features](#features)
- [Project Structure](#project-structure)
- [High-Level Architecture](#high-level-architecture)
- [Data Flow Pipeline](#data-flow-pipeline)
- [Installation](#installation)
- [Usage](#usage)
- [Controls](#controls)
- [Technical Stack](#technical-stack)
- [Performance Optimizations](#performance-optimizations)
- [Known Issues](#known-issues)
- [Contributing](#contributing)
- [License](#license)

---

## ✨ Features

* **Race Replay Visualization:** Watch the race unfold with real-time driver positions on a rendered track
* **Live Leaderboard:** See live driver positions, gaps, and current tyre compounds
* **Lap & Time Display:** Track the current lap and total race time
* **Driver Status:** Drivers who retire or go out are marked as "OUT" on the leaderboard
* **Interactive Controls:** Pause, rewind, fast forward, and adjust playback speed using on-screen buttons or keyboard shortcuts
* **Legend:** On-screen legend explains all controls
* **Driver Telemetry Insights:** View speed, gear, DRS status, and current lap for selected drivers when clicked on the leaderboard
* **Session Support:** Race, Sprint, Qualifying, and Sprint Qualifying sessions

---

## 📁 Project Structure
```
f1-race-replay/
├── main.py                          # Entry point - handles CLI arguments & session loading
├── requirements.txt                 # Python dependencies
├── README.md                        # Project documentation
├── roadmap.md                       # Planned features and project vision
│
├── resources/
│   └── preview.png                  # Race replay preview image
│
├── images/
│   └── tyres/                       # Tire compound images (soft, medium, hard, etc.)
│
├── src/
│   ├── f1_data.py                   # Telemetry loading, processing, and frame generation
│   ├── arcade_replay.py             # Core visualization engine and UI logic
│   ├── ui_components.py             # Reusable UI components (buttons, leaderboard, panels)
│   │
│   ├── interfaces/
│   │   ├── race_replay.py           # Race replay interface and telemetry visualization
│   │   └── qualifying.py            # Qualifying session interface and telemetry visualization
│   │
│   └── lib/
│       ├── tyres.py                 # Type definitions for telemetry data structures
│       └── time.py                  # Time formatting utilities
│
├── .fastf1-cache/                   # FastF1 cache (auto-generated on first run)
├── computed_data/                   # Computed telemetry data cache (auto-generated)
└── .gitignore                       # Git ignore file
```

### Key Files Explained

- **`main.py`**: Entry point that parses command-line arguments and initializes the appropriate session type
- **`src/f1_data.py`**: Core data processing module that fetches telemetry from FastF1 API, synchronizes data across all drivers, and generates frame-by-frame race snapshots
- **`src/arcade_replay.py`**: Main visualization engine using Python Arcade library for rendering track, drivers, and UI
- **`src/ui_components.py`**: Modular UI components (buttons, leaderboard, telemetry panel) for clean separation of concerns
- **`src/interfaces/`**: Session-specific interfaces for race and qualifying replays
- **`src/lib/`**: Utility modules for type definitions and time formatting

---

## 🏗️ High-Level Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                        USER INTERACTION                          │
│           (CLI: python main.py --year 2024 --round 12)          │
└────────────────────────┬────────────────────────────────────────┘
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                     MAIN CONTROLLER                              │
│                       (main.py)                                  │
│  • Parse CLI arguments (year, round, session type)              │
│  • Load F1 session via FastF1 API                               │
│  • Route to appropriate replay interface                        │
└────────────────────────┬────────────────────────────────────────┘
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                  DATA PROCESSING LAYER                           │
│                     (src/f1_data.py)                            │
│                                                                  │
│  ┌──────────────────────────────────────────────┐              │
│  │  1. Cache Check & Management                 │              │
│  │     • Check computed_data/ for cached data   │              │
│  │     • Load pickle if exists (fast path)      │              │
│  └──────────────────────────────────────────────┘              │
│                         ▼                                        │
│  ┌──────────────────────────────────────────────┐              │
│  │  2. FastF1 API Integration                   │              │
│  │     • Fetch session metadata                 │              │
│  │     • Download lap data for all drivers      │              │
│  │     • Retrieve telemetry streams             │              │
│  └──────────────────────────────────────────────┘              │
│                         ▼                                        │
│  ┌──────────────────────────────────────────────┐              │
│  │  3. Parallel Processing (Multiprocessing)    │              │
│  │     • Process each driver's telemetry        │              │
│  │     • Clean & normalize data                 │              │
│  │     • Interpolate missing values             │              │
│  └──────────────────────────────────────────────┘              │
│                         ▼                                        │
│  ┌──────────────────────────────────────────────┐              │
│  │  4. Frame Synchronization                    │              │
│  │     • Merge all driver data                  │              │
│  │     • Create unified timeline (10 FPS)       │              │
│  │     • Generate frame objects                 │              │
│  └──────────────────────────────────────────────┘              │
│                         ▼                                        │
│  ┌──────────────────────────────────────────────┐              │
│  │  5. Persistent Cache (Pickle)                │              │
│  │     • Serialize frames (~100-200MB)          │              │
│  │     • Save to computed_data/                 │              │
│  └──────────────────────────────────────────────┘              │
└────────────────────────┬────────────────────────────────────────┘
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│               VISUALIZATION ENGINE LAYER                         │
│                 (src/arcade_replay.py)                          │
│                                                                  │
│  ┌──────────────────────────────────────────────┐              │
│  │  Rendering Loop (60 FPS)                     │              │
│  │  • Calculate current frame index             │              │
│  │  • Apply playback speed multiplier           │              │
│  │  • Handle user input                         │              │
│  │  • Draw track, drivers, UI                   │              │
│  └──────────────────────────────────────────────┘              │
└────────────────────────┬────────────────────────────────────────┘
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                   UI COMPONENTS LAYER                            │
│                  (src/ui_components.py)                         │
│  • Leaderboard with gaps and tire info                          │
│  • Telemetry panel (speed, gear, DRS, RPM)                     │
│  • Control buttons (pause, rewind, speed)                       │
│  • Legend & time display                                        │
└─────────────────────────────────────────────────────────────────┘
```

### Architecture Principles

1. **Separation of Concerns**: Data processing, visualization, and UI are cleanly separated
2. **Modularity**: UI components are reusable across different session types
3. **Performance**: Multiprocessing for data loading, caching for instant replays
4. **Extensibility**: Easy to add new session types or telemetry features

---

## 🔄 Data Flow Pipeline
```
┌─────────────────────────────────────────────────────────────┐
│  Step 1: Raw Telemetry Data (FastF1 API)                   │
│  • 20 drivers × ~60 laps × ~200 points/lap                 │
│  • Position (X, Y), Speed, Gear, RPM, Throttle, Brake      │
│  • DRS status, Tire compound, Lap number                   │
│  • Total: ~500,000+ raw data points per race               │
└────────────────────────┬────────────────────────────────────┘
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  Step 2: Per-Driver Processing (Parallel - Multiprocessing)│
│                                                              │
│  Process 1 → Driver 1-5:   Clean → Interpolate → Normalize │
│  Process 2 → Driver 6-10:  Clean → Interpolate → Normalize │
│  Process 3 → Driver 11-15: Clean → Interpolate → Normalize │
│  Process 4 → Driver 16-20: Clean → Interpolate → Normalize │
│                                                              │
│  Each process handles:                                       │
│  • Remove invalid data points                               │
│  • Fill missing values (NumPy interpolation)               │
│  • Normalize timestamps to race start                       │
└────────────────────────┬────────────────────────────────────┘
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  Step 3: Frame Synchronization                              │
│  • Merge all driver timelines                               │
│  • Create common time axis (t=0 to t=race_end)             │
│  • Sample at 10 FPS (every 0.1 seconds)                    │
│  • Apply NumPy linear interpolation                         │
│                                                              │
│  Output: Synchronized frame objects                         │
│  Frame 0: {t: 0.0, lap: 1, drivers: {VER: {...}, HAM: {...}}}│
│  Frame 1: {t: 0.1, lap: 1, drivers: {VER: {...}, HAM: {...}}}│
│  ...                                                         │
│  Frame N: {t: 7200, lap: 58, drivers: {...}}               │
└────────────────────────┬────────────────────────────────────┘
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  Step 4: Serialization & Caching (Pickle)                  │
│  • Serialize frame list to binary format                    │
│  • Save to computed_data/{event}_{session}_telemetry.pkl   │
│  • File size: ~100-200MB per race                          │
│  • Reduces subsequent load time: 5 minutes → 5 seconds     │
└────────────────────────┬────────────────────────────────────┘
                         ▼
┌─────────────────────────────────────────────────────────────┐
│  Step 5: Visualization (Arcade Engine)                     │
│  • Load all frames into memory                              │
│  • Render at 60 FPS display rate                           │
│  • Navigate through frames at variable speed (0.5x-4x)     │
│  • Handle user input (pause, rewind, fast-forward)         │
│  • Update UI components (leaderboard, telemetry)           │
└─────────────────────────────────────────────────────────────┘
```

### Data Synchronization

The key challenge is synchronizing telemetry from 20 drivers sampled at different rates:
```python
# Pseudocode example
global_time = np.arange(race_start, race_end, 0.1)  # 10 FPS

for driver in all_drivers:
    # Original data at irregular intervals
    original_time = driver.telemetry['Time']
    original_x = driver.telemetry['X']
    
    # Interpolate to regular 10 FPS grid
    synchronized_x = np.interp(global_time, original_time, original_x)
```

---

## 🚀 Installation

### Prerequisites

- **Python 3.8+** (Check with `python --version`)
- **pip** (Python package manager)
- **Git** (for cloning the repository)

### Step 1: Clone the Repository
```bash
git clone https://github.com/IAmTomShaw/f1-race-replay.git
cd f1-race-replay
```

### Step 2: Create Virtual Environment (Recommended)
```bash
# Create virtual environment
python -m venv venv

# Activate it
# On Windows:
venv\Scripts\activate

# On macOS/Linux:
source venv/bin/activate
```

### Step 3: Install Dependencies
```bash
# Upgrade pip
python -m pip install --upgrade pip

# Install requirements
pip install -r requirements.txt
```

**Dependencies include:**
- `fastf1` - F1 telemetry API
- `arcade` - Graphics engine
- `numpy` - Numerical computing
- `pandas` - Data manipulation
- `matplotlib` - Plotting (optional)

### Step 4: Verify Installation
```bash
python main.py --help
```

You should see the usage instructions.

---

## 📖 Usage

### Basic Commands
```bash
# Replay a race (e.g., 2024 British GP - Round 12)
python main.py --year 2024 --round 12

# Replay a sprint race
python main.py --year 2024 --round 12 --sprint

# Replay qualifying session
python main.py --year 2024 --round 12 --qualifying

# Replay sprint qualifying
python main.py --year 2024 --round 12 --sprint-qualifying
```

### Utility Commands
```bash
# List all races in a season
python main.py --list-rounds --year 2024

# List all sprint races in a season
python main.py --list-sprints --year 2024

# Force refresh data (ignore cache)
python main.py --year 2024 --round 12 --refresh-data
```

### First Run Behavior

**First time running a race:**
- FastF1 will download telemetry data (2-5 minutes)
- `.fastf1-cache/` folder will be created automatically
- Telemetry will be processed and frames generated
- `computed_data/` folder will be created with cached frames
- Replay window will open

**Subsequent runs:**
- Loads from cache instantly (~5 seconds)
- No re-downloading or re-processing needed

### Command-Line Arguments

| Argument | Description | Example |
|----------|-------------|---------|
| `--year` | Race season year | `--year 2024` |
| `--round` | Race round number | `--round 12` |
| `--sprint` | Load sprint race session | `--sprint` |
| `--qualifying` | Load qualifying session | `--qualifying` |
| `--sprint-qualifying` | Load sprint qualifying | `--sprint-qualifying` |
| `--list-rounds` | List all rounds for a year | `--list-rounds --year 2024` |
| `--list-sprints` | List all sprint events | `--list-sprints --year 2024` |
| `--refresh-data` | Force re-download and re-process | `--refresh-data` |
| `--playback-speed` | Initial playback speed | `--playback-speed 2.0` |

---

## 🎮 Controls

### Keyboard Controls

| Key | Action |
|-----|--------|
| **SPACE** | Pause/Resume playback |
| **←** (Left Arrow) | Rewind 5 seconds |
| **→** (Right Arrow) | Fast forward 5 seconds |
| **↑** (Up Arrow) | Increase playback speed |
| **↓** (Down Arrow) | Decrease playback speed |
| **1** | Set speed to 0.5x |
| **2** | Set speed to 1x (normal) |
| **3** | Set speed to 2x |
| **4** | Set speed to 4x |

### Mouse Controls

- **Click on driver name** in leaderboard → View driver telemetry
- **Click Pause button** → Pause/Resume
- **Click Rewind button** → Rewind 5 seconds
- **Click Forward button** → Fast forward 5 seconds
- **Click Speed button** → Cycle through speeds

### On-Screen Display

- **Top-left**: Lap counter and race time
- **Left side**: Leaderboard with positions, gaps, and tire compounds
- **Right side**: Selected driver telemetry (speed, gear, DRS, RPM)
- **Bottom**: Control buttons and legend

---

## 🛠️ Technical Stack

### Core Technologies

- **Python 3.8+** - Primary programming language
- **FastF1 API** - Official F1 telemetry data source
- **Python Arcade** - 2D graphics and game engine
- **NumPy** - Numerical computing and array operations

### Data Processing

- **Pandas** - Data manipulation and analysis
- **NumPy Interpolation** - Synchronizing asynchronous telemetry streams
- **Python Multiprocessing** - Parallel processing for faster data loading
- **Pickle** - Serialization for persistent caching

### Visualization

- **Arcade Graphics Engine** - OpenGL-based rendering
- **Sprite System** - Efficient batch rendering
- **Custom UI Components** - Modular leaderboard, buttons, panels

### Development Tools

- **Git** - Version control
- **Virtual Environment (venv)** - Dependency isolation
- **pip** - Package management

---

## ⚡ Performance Optimizations

### 1. Intelligent Sampling
- Raw telemetry: 50-200 Hz
- Display rate: 10 FPS
- **Result:** 95% data reduction while maintaining visual smoothness

### 2. Pickle Caching
- First run: 2-5 minutes (download + process)
- Cached run: ~5 seconds (load from disk)
- **Result:** 95% load time reduction

### 3. Multiprocessing
- Parallel processing of 20 drivers across CPU cores
- 4-core CPU: ~4x speedup in data processing
- **Result:** Faster initial data preparation

### 4. NumPy Vectorization
- NumPy interpolation: 50x faster than Python loops
- Vectorized operations for position calculations
- **Result:** Sub-millisecond frame calculations

### 5. Sprite Batching
- Batch draw all 20 drivers in single OpenGL call
- Pre-computed leaderboard order per frame
- **Result:** Stable 60 FPS rendering

### 6. Memory Efficiency
- NumPy arrays instead of Python objects
- Efficient data structures (TypedDict)
- **Result:** ~40% memory reduction

---

## 🐛 Known Issues

### Leaderboard Accuracy

- **Issue**: Leaderboard may show inaccuracies in the first few corners
- **Cause**: GPS position data has limited precision at race start
- **Impact**: Temporary position swaps that don't reflect actual race order

### Pit Stop Effects

- **Issue**: Leaderboard temporarily affected when driver pits
- **Cause**: Driver position jumps due to pit lane route
- **Workaround**: Being addressed in future releases

### Final Lap Positions

- **Issue**: Sometimes incorrect at race finish
- **Cause**: Drivers' final X,Y positions may be ahead of slower finishers
- **Status**: Known issue, improvements planned

> These issues stem from telemetry data limitations and are being actively worked on. See [roadmap.md](roadmap.md) for planned improvements.

---

## 🤝 Contributing

Contributions are welcome! Here's how you can help:

### Ways to Contribute

1. **Report bugs** - Open an issue with detailed reproduction steps
2. **Suggest features** - Check [roadmap.md](roadmap.md) and propose new ideas
3. **Improve documentation** - Fix typos, add examples, clarify instructions
4. **Submit code** - Open a pull request with bug fixes or new features

### Contribution Guidelines
```bash
# Fork the repository
# Clone your fork
git clone https://github.com/YOUR_USERNAME/f1-race-replay.git

# Create a feature branch
git checkout -b feature/your-feature-name

# Make your changes
# Test thoroughly

# Commit with descriptive message
git commit -m "Add feature: detailed description"

# Push to your fork
git push origin feature/your-feature-name

# Open a Pull Request on GitHub
```

### Development Setup
```bash
# Install development dependencies (if any)
pip install -r requirements-dev.txt  # if exists

# Run tests (if available)
python -m pytest

# Check code style
flake8 src/
```

---

## 📝 License

This project is licensed under the MIT License. See [LICENSE](LICENSE) file for details.

---

## ⚠️ Disclaimer

No copyright infringement intended. Formula 1 and related trademarks are the property of their respective owners. All data used is sourced from publicly available APIs and is used for educational and non-commercial purposes only.

---

## 🙏 Acknowledgments

- **Original Creator**: [Tom Shaw (IAmTomShaw)](https://github.com/IAmTomShaw)
- **FastF1 Library**: [theOehrly/Fast-F1](https://github.com/theOehrly/Fast-F1)
- **Python Arcade**: [Arcade Library](https://api.arcade.academy/)
- **Formula 1**: For providing accessible telemetry data

---

## 🗺️ Roadmap

See [roadmap.md](roadmap.md) for planned features and project vision.

**Upcoming Features:**
- GUI menu for race selection
- Race control messages (VSC, Safety Car)
- Lap time and interval gaps
- Enhanced error handling
- Performance optimizations
- Mobile/web version

---

**Built with ❤️ by [Tom Shaw](https://tomshaw.dev)**

*Bringing F1 telemetry to life, one frame at a time* 🏎️💨
