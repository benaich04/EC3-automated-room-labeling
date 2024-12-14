# EC3-automated-room-labeling
This repository provides a complete workflow for running Stella VSLAM simulations, extracting data, and visualizing 3D and 2D trajectories. It includes detailed instructions, Python scripts for data processing and visualization, and solutions for common issues, making it a practical resource for VSLAM data analysis.


---

## Table of Contents

1. [Overview](#overview)
2. [Getting Started](#getting-started)
3. [Run the VSLAM Simulation](#run-the-vslam-simulation)
4. [Export Data for Analysis](#export-data-for-analysis)
5. [Copy Data to Host](#copy-data-to-host)
6. [Resolve Permission Issues](#resolve-permission-issues)
7. [Verify and Analyze Data](#verify-and-analyze-data)
8. [Visualize Data in 3D](#visualize-data-in-3d)
9. [Process and Visualize 2D Trajectory](#process-and-visualize-2d-trajectory)
10. [File Locations](#file-locations)
11. [Sample Outputs](#sample-outputs)

---

## Overview

This guide covers the entire process of running a Stella VSLAM simulation, from initial setup to data visualization. It provides:
- Step-by-step instructions for simulation and data extraction.
- Python scripts for processing and visualizing data.
- Solutions to common issues encountered during the workflow.

---

## Getting Started

### Prerequisites
- Docker installed and configured on your machine.
- Basic familiarity with command-line tools.
- Python installed, with Conda for virtual environment management.

### Required Files
- `orb_vocab.fbow`: Orb vocabulary file.
- `dense_hd.yaml`: Camera-specific settings for dense mapping.
- Video file for simulation (e.g., `F2_stern.mp4`).

---

## Run the VSLAM Simulation

### 1. Navigate to the Working Directory
```bash
cd ~/Desktop/stella_vslam_dense
```

### 2. Build the Docker Container
```bash
docker build -t stella_vslam_dense -f Dockerfile.socket . --build-arg NUM_THREADS=$(nproc)
```

### 3. Run the Docker Container
```bash
docker run -it --rm --gpus all --ipc=host --ulimit memlock=-1 --ulimit stack=67108864 -p 3001:3001 --name=stella_vslam_dense -v /home/mb9194/Desktop:/data stella_vslam_dense
```

### 4. Start the Simulation
```bash
./run_video_slam -v /data/orb_vocab.fbow -c /stella_vslam/example/dense/dense_hd.yaml -m /data/F2_stern.mp4 --frame-skip 3 -o /data/my_custom_output.db -p /data/my_custom_output.ply -k /data/my_custom_output_keyframes/
```
- Replace **`F2_stern.mp4`** with your video file.
- Adjust **`dense_hd.yaml`** based on camera settings.
- Customize output file names (e.g., `my_custom_output.db`, `my_custom_output.ply`).

---

## Export Data for Analysis

### 1. Create Output Directory
```bash
mkdir -p /data/my_custom_output
```

### 2. Verify Export Script
```bash
ls /stella_vslam/scripts
```

### 3. Run the Export Script
```bash
python3 /stella_vslam/scripts/export_sqlite3_to_nerfstudio.py /data/my_custom_output.db /data/my_custom_output/
```

---

## Copy Data to Host

### 1. Exit the Docker Container
```bash
exit
```

### 2. Find the Container ID
```bash
docker ps -a
```

### 3. Copy Data to Host Machine
```bash
docker cp <CONTAINER_ID>:/data/my_custom_output ~/Desktop/my_custom_output
```

---

## Resolve Permission Issues

### 1. Check Permissions
```bash
ls -ld ~/Desktop/my_custom_output
```

### 2. Change Ownership
```bash
sudo chown -R <your_username>:<your_username> ~/Desktop/my_custom_output
```

---

## Verify and Analyze Data

### 1. Check Output Directory
```bash
ls -lh ~/Desktop/my_custom_output
```
You should see:
- `dense.ply`
- `sparse.ply`
- `images/`
- `transforms.json`

### 2. Convert `transforms.json` to CSV
Run the provided `json_to_csv.py` script:
```bash
python3 json_to_csv.py
```
**Output**: `trajectory.csv` located in `~/Desktop/my_custom_output/`.

---

## Visualize Data in 3D

### 1. Prepare Conda Virtual Environment
Activate your Conda environment and install dependencies:
```bash
conda env list
conda activate csv_visualization
conda install numpy pandas matplotlib -y
```

### 2. Copy CSV File
```bash
cp ~/Desktop/my_custom_output/trajectory.csv ~/projects/csv_visualization/
```

### 3. Visualize Data
Run the `visualize_csv.py` script:
```bash
python visualize_csv.py
```

---

## Process and Visualize 2D Trajectory

### 1. Project 3D Trajectory to 2D
Run `projet_trajectory.py`:
```bash
python3 projet_trajectory.py
```

### 2. Visualize 2D Data
Execute the visualization script:
```bash
python visualize_csv.py
```

---

## File Locations

- **Input CSV**: `/home/mb9194/projects/csv_visualization/trajectory.csv`
- **Projected CSV**: `/home/mb9194/projects/csv_visualization/projected_trajectory.csv`
- **Python Scripts**:
  - Convert JSON to CSV: `json_to_csv.py`
  - Process Trajectory: `projet_trajectory.py`
  - Visualize Processed Data: `visualize_processed.py`
  - Visualize Raw Data: `visualize_raw.py`

---

## Sample Outputs

### Raw Data Visualization in 3D
*(Insert Image Placeholder)*

### Processed (Projected) Data Visualization
*(Insert Image Placeholder)*
```
