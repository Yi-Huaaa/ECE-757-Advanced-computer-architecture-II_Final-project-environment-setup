# ECE 757: Advanced Computer Architecture II - Final Project Environment Setup

Welcome to the environment setup guide for the ECE 757 Final Project. This repository provides resources and instructions for setting up the simulation environments required for your project.

## 📌 Project Overview

For the final project, you have **two recommended options** for your simulation environment. Both environments have been prepared and tested to ensure they work with the PARSEC benchmark suite.

1. **gem5 + PARSEC** (via Docker)
2. **ChampSim + PARSEC** (Manual Build)

### ⚖️ Comparison & Recommendation (Read This First)

Before starting, please choose the path that best fits your hardware resources and project goals.

| Feature | Option 1: gem5 | Option 2: ChampSim (**Recommended**) |
| --- | --- | --- |
| **Complexity** | High (Full System Simulation) | Low (Trace-Driven Simulation) |
| **Setup Method** | Docker Image (Pre-built) | Manual Build (Script provided) |
| **Memory (RAM)** | **High** (Requires ~15GB System RAM) | **Low** (Very lightweight) |
| **Pros** | Simulates the entire system, including OS and actual code execution. Highly accurate. | Very fast setup, faster simulation speed, easier to modify hardware parameters. |
| **Cons** | Heavy resource usage; steeper learning curve. | **Trace-driven limitations:** It does not execute actual code logic or OS kernel code; it only replays pre-recorded memory/branch events. |
| **Supported OS** | **All** (Linux, Windows, macOS) via Docker | **All** (Linux, macOS, Windows via WSL) |

> **⭐ TA Recommendation:** If you have limited RAM or want a simpler startup process, choose **ChampSim**. If you need to observe OS interactions or full-system behavior, choose **gem5**.

---

## 🚀 Option 1: gem5 + PARSEC (Docker)

We have pre-built a Docker image that contains a full Linux environment with gem5 and PARSEC installed. This saves you hours of compilation time and dependency management.

### :bangbang: System Requirements

* **RAM:** You must dedicate **at least 12GB of RAM** to the Docker container. Your host machine should ideally have **16GB+ RAM**.
* **Disk Space:** **~20GB free space**.

### How to Use

1. **Download & Install Docker:** [Get Docker Desktop](https://www.docker.com/products/docker-desktop/)
  * Click the `Download Docker Desktop` and download the correct version based on your machine (Linux, Windows, and mac Intel, mac apple silicon (M1, M2, M3, M4)).
3. **Pull the Image:** Visit the Docker Hub link below for specific instructions on pulling and running the container on Linux, macOS (including Apple Silicon), and Windows.
* 🔗 **Docker Hub Link:** [yihuachung/gem5-parsec-lab](https://hub.docker.com/r/yihuachung/gem5-parsec-lab)


3. **Run the Container:** Follow the instructions in the Docker Hub description to launch the interactive shell.

> **Note:** Using Docker provides a consistent environment across different operating systems (Linux, Windows, and macOS), eliminating "it works on my machine" issues. This is also a great opportunity to learn how to use Docker!


### 📚 Advanced Notes & Resources
#### Manual gem5 Build & Docker Source (Optional)
This section is for students who:

* Want to **practice building a Docker image** from scratch.
* Prefer to install the gem5 environment **natively** on their local machine without using the TA's pre-built Docker image.
* Are curious about the specific commands and dependencies used to create the lab environment.

The TA has documented the complete build process, including all dependencies and compilation commands. You can follow the instructions in the file below to replicate the environment manually:

* 📄 **[`Gem5_Docker_Build_Guide.md`](https://github.com/Yi-Huaaa/ECE-757-Advanced-computer-architecture-II_Final-project-environment-setup/blob/main/Gem5_Docker_Build_Guide.md)**
* *Note: This is for **reference** only. We strongly suggest using the **Docker** image for the gem5 path to avoid environment conflict issues.* :sunglasses:


---

## 🚀 Option 2: ChampSim + PARSEC (Recommended)

ChampSim is a trace-driven simulator that is significantly easier to build and requires **far less memory** than gem5.

### How to Setup

We have provided a detailed step-by-step guide to downloading dependencies, building the simulator, and running traces.

1. **Open the Guide:** Click on **[`Champsim_build_guide.md`](https://github.com/Yi-Huaaa/ECE-757-Advanced-computer-architecture-II_Final-project-environment-setup/blob/main/Champsim_build_guide.md)** in this repository.
2. **Follow Instructions:** The guide includes scripts to automate the build process and links to download the required PARSEC trace files.

---

### :hammer_and_wrench: Useful Resources

* **Docker:** [you need to learn Docker RIGHT NOW!! // Docker Containers 101](https://www.youtube.com/watch?v=eGz9DS-aIeY)
* **What is Docker in 5 minutes**[What is Docker in 5 minutes](https://www.youtube.com/watch?v=_dfLOzuIg2o&t=18s)
* **gem5:** [Learning gem5 (Official Documentation)](https://www.gem5.org/documentation/gem5art/tutorials/parsec-tutorial)
* **gem5 programming**[Learning gem5 ASPLOS tutorial](https://www.youtube.com/watch?v=wo4b9FPEiHk&list=PL-J9GXT0E7ALwbi339_AsbGvKbdIw52Vf)
* **ChampSim:** [ChampSim GitHub Repository](https://github.com/ChampSim/ChampSim)
* **Introduction to ChampSim Microarchitectural Simulato** [Introduction to ChampSim Microarchitectural Simulato](https://www.youtube.com/watch?v=MuIUo-f31Z0&t=289s)
* **PARSEC trace files for ChampSim** [PARSEC trace files](https://mega.nz/folder/hp1wCRBI#TlHy4GKlEHW-Eyk4AfwBZA)
