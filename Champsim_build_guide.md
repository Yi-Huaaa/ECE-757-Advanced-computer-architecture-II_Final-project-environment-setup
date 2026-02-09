# ECE757: ChampSim + PARSEC Setup Guide

This guide details how to set up [ChampSim](https://github.com/ChampSim/ChampSim.git), download [PARSEC traces](https://mega.nz/folder/hp1wCRBI#TlHy4GKlEHW-Eyk4AfwBZA), and run simulations for the final project.

## System Requirements (Read First)

* **Linux Users (Recommended)**: Use the instructions below directly (Directly goto `ChampSim + PARSEC Installation Starts` section).
* **Windows Users**: You **MUST** use **WSL (Windows Subsystem for Linux)**.
    * Open PowerShell as Administrator and run `wsl --install`, then restart your computer.
    * Once inside the WSL Ubuntu terminal, follow the **Linux** instructions below (Goto `ChampSim + PARSEC Installation Starts` section).
* **macOS Users**: You need **Homebrew** installed. See the specific macOS commands in Section 1.


## 0. Prerequisites & Installation
Open your terminal and execute the following commands to set up the environment and install dependencies.

### For Linux / Windows (WSL)
```bash
# Install necessary packages
sudo apt update && sudo apt install -y build-essential git megatools
```
### For macOS
```bash
# Install Xcode Command Line Tools
xcode-select --install

# Install dependencies via Homebrew
brew install git megatools
```

--- 

## ChampSim + PARSEC Installation Starts (For: Linux / Windows (WSL) / macOS)

### 1. Setup ChampSim
Run these commands for all operating systems (linux/windows/macOS):

```bash
# Create workspace
mkdir -p ECE757_champsim_workspace && cd ECE757_champsim_workspace/

# Clone ChampSim repository
git clone https://github.com/ChampSim/ChampSim.git
cd ChampSim

# Install ChampSim dependencies (vcpkg)
git submodule update --init
./vcpkg/bootstrap-vcpkg.sh
./vcpkg/vcpkg install
```


## 2. Prepare Trace Files

First, create a directory to store the traces:

```bash
mkdir -p trace_files
```

Choose **one** of the following methods to get the trace files.

### Option A: Direct Download (Recommended)

Use `megadl` to download the files directly to the server.

```bash
sudo apt install megatools
megadl --path trace_files/ 'https://mega.nz/folder/hp1wCRBI#TlHy4GKlEHW-Eyk4AfwBZA'
```

### Option B: Manual Upload (If you don't have `sudo`)

If Option A fails, download the files to your local computer and upload them using `scp`.

1. **Download**: Get the files manually from this **[MEGA Link](https://mega.nz/folder/hp1wCRBI#TlHy4GKlEHW-Eyk4AfwBZA)**.
2. **Upload**: Open a terminal on your **local computer** (not the server) and run:
```bash
# Replace 'path/to/downloaded/files' and 'user@host' with your actual details; '~/ECE757_champsim_workspace/ChampSim/trace_files/' can be changed based on your path setting
scp path/to/downloaded/parsec_2.1.* user@your_server_ip:~/ECE757_champsim_workspace/ChampSim/trace_files/
```

### After a successful download, you should see six traces files:
```bash
parsec_2.1.blackscholes.simlarge.prebuilt.drop_1750M.length_250M.champsimtrace.xz
parsec_2.1.blackscholes.simlarge.prebuilt.drop_3000M.length_250M.champsimtrace.xz
parsec_2.1.bodytrack.simlarge.prebuilt.drop_2000M.length_250M.champsimtrace.xz
parsec_2.1.canneal.simlarge.prebuilt.drop_1250M.length_250M.champsimtrace.xz
parsec_2.1.raytrace.simlarge.prebuilt.drop_250M.length_250M.champsimtrace.xz
parsec_2.1.vips.simlarge.prebuilt.drop_27250M.length_250M.champsimtrace.xz
```


## 3. Compilation
Before running any simulation, you must configure and compile the simulator.

```bash
# 1. Generate configuration (using default settings)
./config.sh champsim_config.json

# 2. Compile the source code
make
```

## 4. Execution
To run a simulation, use the generated binary in the `bin/` directory.

**Example Command:**
Runs the *Blackscholes_1750M* benchmark with 1M warmup instructions and 10M simulation instructions.

```bash
# Run the execution 
./bin/champsim --warmup-instructions 1000000 --simulation-instructions 10000000 trace_files/parsec_2.1.blackscholes.simlarge.prebuilt.drop_1750M.length_250M.champsimtrace.xz
```
* About the above command: 
  * `--warmup-instructions`: Number of instructions to **warm up the cache**.
  * `--simulation-instructions`: Number of instructions to **record statistics**.
  * You have six benchmarks. You can change different benchmarks (e.g., change `trace_files/parsec_2.1.blackscholes.simlarge.prebuilt.drop_1750M.length_250M.champsimtrace.xz` to `trace_files/parsec_2.1.blackscholes.simlarge.prebuilt.drop_3000M.length_250M.champsimtrace.xz`). 

**Successful Outcomes of the Example:**

```bash
(base) ychung79@twhuang-desktop-03:~/ECE757_champsim_workspace/ChampSim$ ./bin/champsim --warmup-instructions 1000000 --simulation-instructions 10000000 trace_files/parsec_2.1.blackscholes.simlarge.prebuilt.drop_1750M.length_250M.champsimtrace.xz
[VMEM] WARNING: physical memory size is smaller than virtual memory size.

*** ChampSim Multicore Out-of-Order Simulator ***
Warmup Instructions: 1000000
Simulation Instructions: 10000000
Number of CPUs: 1
Page size: 4096

Off-chip DRAM Size: 16 GiB Channels: 1 Width: 64-bit Data Rate: 3205 MT/s
Warmup finished CPU 0 instructions: 1000003 cycles: 255364 cumulative IPC: 3.916 (Simulation time: 00 hr 00 min 01 sec)
Warmup complete CPU 0 instructions: 1000003 cycles: 255364 cumulative IPC: 3.916 (Simulation time: 00 hr 00 min 01 sec)
Heartbeat CPU 0 instructions: 10000004 cycles: 4855803 heartbeat IPC: 2.059 cumulative IPC: 1.956 (Simulation time: 00 hr 00 min 25 sec)
Simulation finished CPU 0 instructions: 10000000 cycles: 5098853 cumulative IPC: 1.961 (Simulation time: 00 hr 00 min 27 sec)
Simulation complete CPU 0 instructions: 10000000 cycles: 5098853 cumulative IPC: 1.961 (Simulation time: 00 hr 00 min 27 sec)

ChampSim completed all CPUs

=== Simulation ===
CPU 0 runs trace_files/parsec_2.1.blackscholes.simlarge.prebuilt.drop_1750M.length_250M.champsimtrace.xz

Region of Interest Statistics

CPU 0 cumulative IPC: 1.961 instructions: 10000000 cycles: 5098853
CPU 0 Branch Prediction Accuracy: 97.28% MPKI: 2.672 Average ROB Occupancy at Mispredict: 160.5
Branch type MPKI
BRANCH_DIRECT_JUMP: 0
BRANCH_INDIRECT: 0
BRANCH_CONDITIONAL: 2.672
BRANCH_DIRECT_CALL: 0
BRANCH_INDIRECT_CALL: 0
BRANCH_RETURN: 0

cpu0->cpu0_STLB TOTAL        ACCESS:      41333 HIT:      41228 MISS:        105 MSHR_MERGE:          0
cpu0->cpu0_STLB LOAD         ACCESS:      41333 HIT:      41228 MISS:        105 MSHR_MERGE:          0
cpu0->cpu0_STLB RFO          ACCESS:          0 HIT:          0 MISS:          0 MSHR_MERGE:          0
cpu0->cpu0_STLB PREFETCH     ACCESS:          0 HIT:          0 MISS:          0 MSHR_MERGE:          0
cpu0->cpu0_STLB WRITE        ACCESS:          0 HIT:          0 MISS:          0 MSHR_MERGE:          0
cpu0->cpu0_STLB TRANSLATION  ACCESS:          0 HIT:          0 MISS:          0 MSHR_MERGE:          0
cpu0->cpu0_STLB PREFETCH REQUESTED:          0 ISSUED:          0 USEFUL:          0 USELESS:          0
cpu0->cpu0_STLB AVERAGE MISS LATENCY: 704.1 cycles
cpu0->cpu0_L2C TOTAL        ACCESS:       8013 HIT:       1053 MISS:       6960 MSHR_MERGE:          0
cpu0->cpu0_L2C LOAD         ACCESS:       5907 HIT:         74 MISS:       5833 MSHR_MERGE:          0
cpu0->cpu0_L2C RFO          ACCESS:        971 HIT:          0 MISS:        971 MSHR_MERGE:          0
cpu0->cpu0_L2C PREFETCH     ACCESS:          0 HIT:          0 MISS:          0 MSHR_MERGE:          0
cpu0->cpu0_L2C WRITE        ACCESS:        962 HIT:        962 MISS:          0 MSHR_MERGE:          0
cpu0->cpu0_L2C TRANSLATION  ACCESS:        173 HIT:         17 MISS:        156 MSHR_MERGE:          0
cpu0->cpu0_L2C PREFETCH REQUESTED:          0 ISSUED:          0 USEFUL:          0 USELESS:          0
cpu0->cpu0_L2C AVERAGE MISS LATENCY: 214.5 cycles
cpu0->cpu0_L1I TOTAL        ACCESS:     810193 HIT:     810193 MISS:          0 MSHR_MERGE:          0
cpu0->cpu0_L1I LOAD         ACCESS:     810193 HIT:     810193 MISS:          0 MSHR_MERGE:          0
cpu0->cpu0_L1I RFO          ACCESS:          0 HIT:          0 MISS:          0 MSHR_MERGE:          0
cpu0->cpu0_L1I PREFETCH     ACCESS:          0 HIT:          0 MISS:          0 MSHR_MERGE:          0
cpu0->cpu0_L1I WRITE        ACCESS:          0 HIT:          0 MISS:          0 MSHR_MERGE:          0
cpu0->cpu0_L1I TRANSLATION  ACCESS:          0 HIT:          0 MISS:          0 MSHR_MERGE:          0
cpu0->cpu0_L1I PREFETCH REQUESTED:          0 ISSUED:          0 USEFUL:          0 USELESS:          0
cpu0->cpu0_L1I AVERAGE MISS LATENCY: - cycles
cpu0->cpu0_L1D TOTAL        ACCESS:    2135053 HIT:    2127939 MISS:       7114 MSHR_MERGE:         63
cpu0->cpu0_L1D LOAD         ACCESS:    1338929 HIT:    1333020 MISS:       5909 MSHR_MERGE:          2
cpu0->cpu0_L1D RFO          ACCESS:          0 HIT:          0 MISS:          0 MSHR_MERGE:          0
cpu0->cpu0_L1D PREFETCH     ACCESS:          0 HIT:          0 MISS:          0 MSHR_MERGE:          0
cpu0->cpu0_L1D WRITE        ACCESS:     795914 HIT:     794882 MISS:       1032 MSHR_MERGE:         61
cpu0->cpu0_L1D TRANSLATION  ACCESS:        210 HIT:         37 MISS:        173 MSHR_MERGE:          0
cpu0->cpu0_L1D PREFETCH REQUESTED:          0 ISSUED:          0 USEFUL:          0 USELESS:          0
cpu0->cpu0_L1D AVERAGE MISS LATENCY: 220.7 cycles
cpu0->cpu0_ITLB TOTAL        ACCESS:     722087 HIT:     722087 MISS:          0 MSHR_MERGE:          0
cpu0->cpu0_ITLB LOAD         ACCESS:     722087 HIT:     722087 MISS:          0 MSHR_MERGE:          0
cpu0->cpu0_ITLB RFO          ACCESS:          0 HIT:          0 MISS:          0 MSHR_MERGE:          0
cpu0->cpu0_ITLB PREFETCH     ACCESS:          0 HIT:          0 MISS:          0 MSHR_MERGE:          0
cpu0->cpu0_ITLB WRITE        ACCESS:          0 HIT:          0 MISS:          0 MSHR_MERGE:          0
cpu0->cpu0_ITLB TRANSLATION  ACCESS:          0 HIT:          0 MISS:          0 MSHR_MERGE:          0
cpu0->cpu0_ITLB PREFETCH REQUESTED:          0 ISSUED:          0 USEFUL:          0 USELESS:          0
cpu0->cpu0_ITLB AVERAGE MISS LATENCY: - cycles
cpu0->cpu0_DTLB TOTAL        ACCESS:    2039236 HIT:    1993370 MISS:      45866 MSHR_MERGE:       4533
cpu0->cpu0_DTLB LOAD         ACCESS:    2039236 HIT:    1993370 MISS:      45866 MSHR_MERGE:       4533
cpu0->cpu0_DTLB RFO          ACCESS:          0 HIT:          0 MISS:          0 MSHR_MERGE:          0
cpu0->cpu0_DTLB PREFETCH     ACCESS:          0 HIT:          0 MISS:          0 MSHR_MERGE:          0
cpu0->cpu0_DTLB WRITE        ACCESS:          0 HIT:          0 MISS:          0 MSHR_MERGE:          0
cpu0->cpu0_DTLB TRANSLATION  ACCESS:          0 HIT:          0 MISS:          0 MSHR_MERGE:          0
cpu0->cpu0_DTLB PREFETCH REQUESTED:          0 ISSUED:          0 USEFUL:          0 USELESS:          0
cpu0->cpu0_DTLB AVERAGE MISS LATENCY: 6.792 cycles
cpu0->LLC TOTAL        ACCESS:       7116 HIT:        157 MISS:       6959 MSHR_MERGE:          0
cpu0->LLC LOAD         ACCESS:       5833 HIT:          1 MISS:       5832 MSHR_MERGE:          0
cpu0->LLC RFO          ACCESS:        971 HIT:          0 MISS:        971 MSHR_MERGE:          0
cpu0->LLC PREFETCH     ACCESS:          0 HIT:          0 MISS:          0 MSHR_MERGE:          0
cpu0->LLC WRITE        ACCESS:        156 HIT:        156 MISS:          0 MSHR_MERGE:          0
cpu0->LLC TRANSLATION  ACCESS:        156 HIT:          0 MISS:        156 MSHR_MERGE:          0
cpu0->LLC PREFETCH REQUESTED:          0 ISSUED:          0 USEFUL:          0 USELESS:          0
cpu0->LLC AVERAGE MISS LATENCY: 198.5 cycles

DRAM Statistics

Channel 0 RQ ROW_BUFFER_HIT:          0
  ROW_BUFFER_MISS:       6959
  AVG DBUS CONGESTED CYCLE: 3.057
Channel 0 WQ ROW_BUFFER_HIT:          0
  ROW_BUFFER_MISS:          0
  FULL:          0
Channel 0 REFRESHES ISSUED:        425
(base) ychung79@twhuang-desktop-03:~/ECE757_champsim_workspace/ChampSim$ 
```

---

## 5. How to Modify Hardware Configuration for Your Final Project (Just an Example Here)

To change parameters (e.g., Cache size, Branch Predictor), follow this **Loop**:

1. **Edit Configuration**:
Open `champsim_config.json`. (Note: `champsim_config.json` file is under `ChampSim` folder)
* **L2 Cache Size**: Modify `"sets"` under `"L2C"` (e.g., change `1024` to `2048`).
* **Branch Predictor**: Change `"branch_predictor"` value (e.g., `"bimodal"`, `"gshare"`, `"perceptron"`).


```json
"L2C": {
    "sets": 2048,
    "ways": 8,
    ...
}
```

2. **Recompile (CRITICAL STEP)**:
You **must** run these commands every time you modify the JSON file:
```bash
./config.sh champsim_config.json
make
```

3. **Run Simulation**:
Execute the `./bin/champsim --warmup-instructions 1000000 --simulation-instructions 10000000 trace_files/parsec_2.1.blackscholes.simlarge.prebuilt.drop_1750M.length_250M.champsimtrace.xz
` command again to see the new results. Also, every paramerters (e.g., --warmup-instructions) are adjustable. 

