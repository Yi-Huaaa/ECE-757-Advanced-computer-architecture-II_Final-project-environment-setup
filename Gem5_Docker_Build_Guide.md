# 🛠️ Building the Gem5 + PARSEC Docker Environment

This guide records the step-by-step process of creating the `gem5-parsec-lab` Docker image. It is intended for advanced students or TAs who want to understand how the environment was built from scratch.

--

## 1. Install Docker on Linux (Ubuntu)

If you are using Linux, follow these commands to install Docker. For macOS or Windows, simply install **Docker Desktop**. 
> Docker Desktop [Link](https://www.docker.com/products/docker-desktop/)


```bash
# 1. Remove old versions (if any)
sudo apt-get remove docker docker-engine docker.io containerd runc

# 2. Update package list and install utilities
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg lsb-release

# 3. Create a directory for encryption keys
sudo mkdir -p /etc/apt/keyrings

# 4. Download and save the official Docker GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 5. Add the Docker repository to Apt sources
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 6. Update the list again to include the new repo
sudo apt-get update

# 7. Install Docker Engine and components
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# 8. Add your user to the docker group (avoids using 'sudo' every time)
sudo usermod -aG docker $USER

# 9. Apply group changes immediately
newgrp docker

# 10. Verify installation
docker run hello-world
# "Hello from Docker!" This message shows that your installation appears to be working correctly. ...
```

### For step 1, you should get output looks like this:
```
(base) ychung79@twhuang-desktop-03:~$ sudo usermod -aG docker $USER
(base) ychung79@twhuang-desktop-03:~$ newgrp docker
(base) ychung79@twhuang-desktop-03:~$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
17eec7bbc9d7: Pull complete 
ea52d2000f90: Download complete 
Digest: sha256:05813aedc15fb7b4d732e1be879d3252c1c9c25d885824f6295cab4538cb85cd
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

(base) ychung79@twhuang-desktop-03:~$ 
```

## 2. Setup the Development Workspace
We will verify that we can compile gem5 on the host machine (or inside a temporary container) before packaging it.

```bash
# 1. Create a workspace on your host machine
mkdir -p ~/gem5_workspace
cd ~/gem5_workspace

# 2. Launch a clean Ubuntu 22.04 container
# --rm: Automatically remove the container after exit (clean up)
# -v $(pwd):/workspace: Mount current directory to /workspace inside the container
# --privileged: Grant full permissions (avoids compilation issues)
docker run --rm -it --privileged -v $(pwd):/workspace -w /workspace ubuntu:22.04 bash
```

### For step 2, you should get output looks like this:
```bash
(base) ychung79@twhuang-desktop-03:~/ECE757_gem5_workspace$ docker run --rm -it --privileged -v $(pwd):/workspace -w /workspace ubuntu:22.04 bash
Unable to find image 'ubuntu:22.04' locally
22.04: Pulling from library/ubuntu
6f4ebca3e823: Pull complete 
e4eb0721af32: Download complete 
Digest: xxxxxxxx...
Status: Downloaded newer image for ubuntu:22.04
root@da3d74fb724e:/workspace# 
```

## 3. Install and Compile Gem5
Once inside the container (root@...:/workspace#), install the dependencies and compile gem5.
```bash
# 1. Update software repositories
apt-get update

# 2. Install all tools required to build Gem5 (Takes 1-3 mins)
apt-get install -y build-essential git m4 scons zlib1g zlib1g-dev \
    libprotobuf-dev protobuf-compiler libprotoc-dev libgoogle-perftools-dev \
    python3-dev python3-six python-is-python3 libboost-all-dev \
    pkg-config wget vim nano

# 3. Download Gem5 source code
git clone https://github.com/gem5/gem5.git
```

## 4. Compile Gem5

```bash
# Enter the gem5 directory (Crucial step!)
cd gem5

# Compile gem5 into an executable
# This builds an X86 simulator using all available CPU cores (-j)
# Note: This process may take 20-40 minutes.
scons build/X86/gem5.opt -j $(nproc)
```

### For steps 3 and 4, you should get output looks like this:
```
scons: Reading SConscript files ...

You're missing the pre-commit/commit-msg hooks. These hook help to ensure your
code follows gem5's style rules on git commit and your commit messages follow
our commit message requirements. This script will now install these hooks in
your .git/hooks/ directory.
Press enter to continue, or ctrl-c to abort:
 Cannot find 'pre-commit'. Please ensure all Python requirements are 
installed. This can be done via 'pip install -r requirements.txt'.
It is strongly recommended you install the pre-commit hooks before working with gem5. Do you want to continue compilation (y/n)?
y
Mkdir("/workspace/gem5/build/X86/gem5.build")
Checking for linker -Wl,--as-needed support... yes
Checking for compiler -gz support... yes
Checking for linker -gz support... yes
(...)
 [     CXX] src/base/date.cc -> X86/base/date.o
 [    LINK]  -> X86/gem5.opt
scons: done building targets.
*** Summary of Warnings ***
Warning: Header file <capstone/capstone.h> not found.
         This host has no capstone library installed.
Warning: Header file <png.h> not found.
         This host has no libpng library.
         Disabling support for PNG framebuffers.
Warning: Couldn't find HDF5 C++ libraries. Disabling HDF5 support.
root@d99eed69eff7:/workspace/gem5# 
```

### Notes:
1. scons：Gem5 不用 make，而是用這個叫做 SCons (基於 Python) 的建置工具。
2. build/X86：我們告訴它：「請幫我造一個 X86 架構 的模擬器」。
3. 如果你要跑 ARM 的 Benchmark，這裡就要改成 build/ARM。但為了教學通用性，我們先做 X86。
4. it is possibly to take 20-40 mins. you can use `htop` to check whether it is working or not. 

### Quick Verification through (Hello World)
Test if the compiled binary works correctly.
```bash
build/X86/gem5.opt configs/deprecated/example/se.py --cmd=tests/test-progs/hello/bin/x86/linux/hello
# Expected Result: You should see Hello world! in the output.
```

* Correct output:
```bash
root@d99eed69eff7:/workspace/gem5# build/X86/gem5.opt configs/deprecated/example/se.py --cmd=tests/test-progs/hello/bin/x86/linux/hello
gem5 Simulator System.  https://www.gem5.org
gem5 is copyrighted software; use the --copyright option for details.
gem5 version 25.1.0.0
gem5 compiled Feb  4 2026 20:04:58
gem5 started Feb  4 2026 20:09:37
gem5 executing on d99eed69eff7, pid 25596
command line: build/X86/gem5.opt configs/deprecated/example/se.py --cmd=tests/test-progs/hello/bin/x86/linux/hello
warn: The se.py script is deprecated. It will be removed in future releases of  gem5.
Global frequency set at 1000000000000 ticks per second
warn: No dot file generated. Please install pydot to generate the dot file and pdf.
src/mem/dram_interface.cc:690: warn: DRAM device capacity (8192 Mbytes) does not match the address range assigned (512 Mbytes)
src/base/statistics.hh:279: warn: One of the stats is a legacy stat. Legacy stat is a stat that does not belong to any statistics::Group. Legacy stat is deprecated.
system.remote_gdb: Listening for connections on port 7000
**** REAL SIMULATION ****
Hello world!
Exiting @ tick 5943000 because exiting with last active thread context
root@d99eed69eff7:/workspace/gem5# 
```

## 5. Prepare PARSEC Benchmarks (The "Resource Trick")
We want to pre-download the PARSEC disk images and kernels so students don't have to wait for 3GB downloads.

### Step A: Configure the Script
Create a python script to trigger the download.
```bash
# Set environment variable path
export GEM5_RESOURCE_PATH=/workspace/gem5_resources

# Create the run script
cd /workspace/gem5
vim run_parsec.py
```

Paste the following code into run_parsec.py. This script sets up a basic X86 simulation.

```python
import os
os.environ["GEM5_RESOURCE_PATH"] = "/workspace/gem5_resources"

import m5
from m5.objects import *
from gem5.utils.requires import requires
from gem5.components.boards.x86_board import X86Board
from gem5.components.memory.single_channel import SingleChannelDDR3_1600
from gem5.components.cachehierarchies.classic.private_l1_private_l2_walk_cache_hierarchy import PrivateL1PrivateL2WalkCacheHierarchy
from gem5.components.processors.simple_switchable_processor import SimpleSwitchableProcessor
from gem5.components.processors.cpu_types import CPUTypes
from gem5.isas import ISA
from gem5.resources.resource import obtain_resource
from gem5.simulate.simulator import Simulator
from gem5.simulate.exit_event import ExitEvent

# 1. Setup Hardware Parameters
# Using a classic L1+L2 cache hierarchy
cache_hierarchy = PrivateL1PrivateL2WalkCacheHierarchy(
    l1d_size="32kB",
    l1i_size="32kB",
    l2_size="256kB"
)

# Setup Memory (3GB)
memory = SingleChannelDDR3_1600(size="3GB")

# Setup CPU
# We use "SwitchableProcessor" for faster booting.
# Boot Linux with "ATOMIC" (fast, simple) or KVM.
# Switch to "TIMING" (detailed) for the actual benchmark.
processor = SimpleSwitchableProcessor(
    starting_core_type=CPUTypes.ATOMIC,
    switch_core_type=CPUTypes.TIMING,
    isa=ISA.X86,
    num_cores=1
)

# Setup the Motherboard (X86Board)
board = X86Board(
    clk_freq="3GHz",
    processor=processor,
    memory=memory,
    cache_hierarchy=cache_hierarchy
)

# 2. Setup Software (PARSEC Benchmark)
# This will automatically download the x86-parsec Disk Image
# The command string defines what runs inside the simulated Linux
command = "cd /home/gem5/parsec-benchmark;" \
          "source env.sh;" \
          "parsecmgmt -a run -p blackscholes -c gcc-hooks -i simsmall -n 1;" \
          "m5 exit;"

board.set_kernel_disk_workload(
    kernel=obtain_resource("x86-linux-kernel-5.4.49"),
    disk_image=obtain_resource("x86-parsec"),
    readfile_contents=command
)

# Define a handler to manage what happens when Linux calls "m5 exit"
def handle_exit():
    # First m5 exit: Linux has finished booting
    print("Done booting Linux! Switching CPU to Timing mode...")
    processor.switch()
    yield False # yield False means: "Continue simulation, do not exit!"
    
    # Second m5 exit: Benchmark is finished
    print("Benchmark completed! Exiting simulation.")
    yield True # yield True means: "Simulation is effectively over, stop now."

# 3. Start Simulation
simulator = Simulator(
    board=board,
    on_exit_event={
        # Pass the Exit event to the handle_exit function defined above
        ExitEvent.EXIT : handle_exit()
    }
)

print("Starting the simulation! Note: First run will download ~3GB files.")
simulator.run()
```

### Step B: The Symlink Trick (Crucial!)
Gem5 downloads files to ~/.cache/gem5 by default. We need to redirect this to our mounted /workspace so the files are saved on the host machine (Mac/Linux), not just inside this temporary container.

```bash
# 1. Create our target directory (mapped to host)
mkdir -p /workspace/gem5_resources

# 2. Remove any partial downloads
rm -rf /root/.cache/gem5

# 3. Create the "Portal" (Symbolic Link)
# This fools gem5 into saving cached files directly to our host folder
mkdir -p /root/.cache
ln -s /workspace/gem5_resources /root/.cache/gem5
```

### Step C: Trigger Download
Run the script. This will take time as it downloads ~3GB of data.
```bash
build/X86/gem5.opt run_parsec.py
```

You should see output looks like: 
```bash
root@d99eed69eff7:/workspace/gem5# build/X86/gem5.opt run_parsec.py
gem5 Simulator System.  https://www.gem5.org
gem5 is copyrighted software; use the --copyright option for details.
gem5 version 25.1.0.0
gem5 compiled Feb  4 2026 20:04:58
gem5 started Feb  4 2026 21:05:04
gem5 executing on d99eed69eff7, pid 25610
command line: build/X86/gem5.opt run_parsec.py
warn: Base 10 memory/cache size 3GB will be cast to base 2 size 3GiB.
info: Using default config
warn: Resource x86-linux-kernel-5.4.49 is not compatible with gem5 version 25.1.0.0.
warn: Resource x86-parsec is not compatible with gem5 version 25.1.0.0.
warn: Resource x86-linux-kernel-5.4.49 is not compatible with gem5 version 25.1.0.0.
warn: Resource x86-parsec is not compatible with gem5 version 25.1.0.0.
Starting the simulation! Note: First run will download ~3GB files.
warn: Base 10 memory/cache size 256kB will be cast to base 2 size 256KiB.
warn: Base 10 memory/cache size 32kB will be cast to base 2 size 32KiB.
warn: Base 10 memory/cache size 32kB will be cast to base 2 size 32KiB.
Global frequency set at 1000000000000 ticks per second
warn: board.workload.acpi_description_table_pointer.rsdt adopting orphan SimObject param 'entries'
warn: No dot file generated. Please install pydot to generate the dot file and pdf.
src/mem/dram_interface.cc:690: warn: DRAM device capacity (8192 Mbytes) does not match the address range assigned (4096 Mbytes)
src/sim/kernel_workload.cc:46: info: kernel located at: /root/.cache/gem5/x86-linux-kernel-5.4.49-1.0.0
      0: board.pc.south_bridge.cmos.rtc: Real-time clock set to Sun Jan  1 00:00:00 2012
      board.pc.com_1.device: Listening for connections on port 3456
src/base/statistics.hh:279: warn: One of the stats is a legacy stat. Legacy stat is a stat that does not belong to any statistics::Group. Legacy stat is deprecated.
src/dev/intel_8254_timer.cc:128: warn: Reading current count from inactive timer.
board.remote_gdb: Listening for connections on port 7000
build/X86/arch/x86/generated/exec-ns.cc.inc:27: warn: instruction 'fninit' unimplemented
build/X86/arch/x86/generated/exec-ns.cc.inc:27: warn: instruction 'sgdt_Ms' unimplemented
build/X86/arch/x86/generated/exec-ns.cc.inc:27: warn: instruction 'fwait' unimplemented
src/dev/x86/pc.cc:117: warn: Don't know what interrupt to clear for console.
src/dev/x86/i8042.cc:290: warn: Write to unknown i8042 (keyboard controller) command port.
src/arch/generic/debugfaults.hh:145: warn: MOVNTDQ: Ignoring non-temporal hint, modeling as cacheable!
Done booting Linux! Switching CPU to Timing mode...
switching cpus
src/sim/power_state.cc:105: warn: PowerState: Already in the requested power state, request ignored
build/X86/arch/x86/generated/exec-ns.cc.inc:27: warn: instruction 'fwait' unimplemented
Benchmark completed! Exiting simulation.
root@d99eed69eff7:/workspace/gem5# 
```

#### See results
```
ls -l m5out
head -n 20 m5out/stats.txt
cat m5out/board.pc.com_1.device
```

Correct output, the last line of your `stats.txt` file should be:
```
---------- End Simulation Statistics   ----------
```

### Step D: Clean Up and Exit
Once the download is finished, check the results and exit the temporary container.

```bash
# Verify files exist
ls -F /workspace/gem5_resources

# Exit the container
exit
```


### 6. Construct the Dockerfile
Now that we have the compiled `gem5` and the downloaded `gem5_resources` on our host machine (`~/gem5_workspace`), we can package them into a final Docker image.

Create a file named `Dockerfile` in `~/gem5_workspace`:

```docker
# 1. Use Ubuntu 22.04 as the base image
FROM ubuntu:22.04

# 2. Set environment variables to avoid interactive prompts during installation
ENV DEBIAN_FRONTEND=noninteractive

# 3. Install necessary tools (Same dependencies we installed manually)
# This includes build tools (gcc, scons) and Python environment
RUN apt-get update && apt-get install -y \
    build-essential \
    git \
    m4 \
    scons \
    zlib1g \
    zlib1g-dev \
    libprotobuf-dev \
    protobuf-compiler \
    libprotoc-dev \
    libgoogle-perftools-dev \
    python3-dev \
    python3-pip \
    libboost-all-dev \
    pkg-config \
    vim \
    nano \
    && rm -rf /var/lib/apt/lists/*

# 4. Create the working directory
WORKDIR /workspace

# 5. [CRITICAL] Copy our pre-compiled Gem5 into the image
# This saves students from compiling it themselves (~40 mins saved)
COPY gem5 /workspace/gem5

# 6. [CRITICAL] Copy the downloaded PARSEC resources
COPY gem5_resources /workspace/gem5_resources

# 7. Set environment variable so Gem5 knows where resources are located
ENV GEM5_RESOURCE_PATH=/workspace/gem5_resources

# 8. Set Python path to ensure scripts can be executed directly
ENV PYTHONPATH=/workspace/gem5

# 9. Default entry point
CMD ["/bin/bash"]
```
### 7. Build and Publish the Image
```bash
cd ~/gem5_workspace

# Clean up temporary output files to keep the image small
sudo rm -rf gem5/m5out
sudo find gem5 -type d -name "__pycache__" -exec rm -rf {} +
sudo find gem5/build/X86 -name "*.o" -delete

# Build the Docker Image (Tag it as gem5-parsec-lab)
docker build -t gem5-parsec-lab .
# Note: `-t gem5-parsec-lab`, `t` means tag, just give your image a name (tag)

```

#### Outputs:
```
(base) ychung79@twhuang-desktop-03:~/ECE757_gem5_workspace$ docker build -t gem5-parsec-lab .
[+] Building 477.4s (11/11) FINISHED                                               docker:default
=> [internal] load build definition from Dockerfile                                            0.0s
=> => transferring dockerfile: 1.35kB                                                   0.0s
=> [internal] load metadata for docker.io/library/ubuntu:22.04                                      0.0s
=> [internal] load .dockerignore                                                     0.0s
=> => transferring context: 2B                                                      0.0s
=> [1/6] FROM docker.io/library/ubuntu:22.04@sha256:c7eb020043d8fc2ae0793fb35a37bff1cf33f156d4d4b12ccc7f3ef8706c38b1           0.0s
=> => resolve docker.io/library/ubuntu:22.04@sha256:c7eb020043d8fc2ae0793fb35a37bff1cf33f156d4d4b12ccc7f3ef8706c38b1           0.0s
=> [internal] load build context                                                     58.2s
=> => transferring context: 27.93GB                                                   58.2s
=> CACHED [2/6] RUN apt-get update && apt-get install -y  build-essential  git  m4  scons  zlib1g  zlib1g-dev  libpr 0.0s
=> CACHED [3/6] WORKDIR /workspace                                                    0.0s
=> CACHED [4/6] COPY gem5 /workspace/gem5                                                 0.0s
=> CACHED [5/6] RUN mkdir -p /root/.cache/gem5                                              0.0s
=> CACHED [6/6] COPY gem5_resources /root/.cache/gem5                                           0.0s
=> exporting to image                                                          419.1s
=> => exporting layers                                                         371.0s
=> => exporting manifest sha256:9430863f54a6d9a328e590fdd8eff29cd28852fd3ad111ac0aed2c7d02aca2bf                     0.0s
=> => exporting config sha256:641c5b53fc6c2422ed324f6bfac017d4bf4636a6c00a13b373cf4cfea0ef1df9                      0.0s
=> => exporting attestation manifest sha256:88e0dadfba9a6e5072cffc135681f4315cc84bf678bd1faacd2aab155c782414               0.0s
=> => exporting manifest list sha256:34d92f9b18bd61360f824bc736d10e9fad43e48cfd01f45bfab8be39471f5827                   0.0s
=> => naming to docker.io/library/gem5-parsec-lab:latest                                         0.0s
=> => unpacking to docker.io/library/gem5-parsec-lab:latest                                       48.1s
(base) ychung79@twhuang-desktop-03:~/ECE757_gem5_workspace$
```



#### Verify the Image
Run the new image to ensure everything works.
```bash
docker run --rm -it gem5-parsec-lab

# Inside the container:
cd gem5
build/X86/gem5.opt run_parsec.py
```

#### Outputs:
```bash
root@6b4b1c5335e2:/workspace# cd gem5/
root@6b4b1c5335e2:/workspace/gem5# build/X86/gem5.opt run_parsec.py
gem5 Simulator System.  https://www.gem5.org
gem5 is copyrighted software; use the --copyright option for details.
gem5 version 25.1.0.0
gem5 compiled Feb  4 2026 20:04:58
gem5 started Feb  4 2026 23:47:55
gem5 executing on 6b4b1c5335e2, pid 16
command line: build/X86/gem5.opt run_parsec.py
warn: Base 10 memory/cache size 3GB will be cast to base 2 size 3GiB.
info: Using default config
warn: Resource x86-linux-kernel-5.4.49 is not compatible with gem5 version 25.1.0.0.
warn: Resource x86-parsec is not compatible with gem5 version 25.1.0.0.
warn: Resource x86-linux-kernel-5.4.49 is not compatible with gem5 version 25.1.0.0.
warn: Resource x86-parsec is not compatible with gem5 version 25.1.0.0.
Starting the simulation! Note: First run will download ~3GB files.
warn: Base 10 memory/cache size 256kB will be cast to base 2 size 256KiB.
warn: Base 10 memory/cache size 32kB will be cast to base 2 size 32KiB.
warn: Base 10 memory/cache size 32kB will be cast to base 2 size 32KiB.
Global frequency set at 1000000000000 ticks per second
warn: board.workload.acpi_description_table_pointer.rsdt adopting orphan SimObject param 'entries'
warn: No dot file generated. Please install pydot to generate the dot file and pdf.
src/mem/dram_interface.cc:690: warn: DRAM device capacity (8192 Mbytes) does not match the address range assigned (4096 Mbytes)
src/sim/kernel_workload.cc:46: info: kernel located at: /root/.cache/gem5/x86-linux-kernel-5.4.49-1.0.0
      0: board.pc.south_bridge.cmos.rtc: Real-time clock set to Sun Jan  1 00:00:00 2012
      board.pc.com_1.device: Listening for connections on port 3456
(...)
```

#### Push to Docker Hub
Share the image with others.

```bash
docker login

# Tag and Push 
docker tag gem5-parsec-lab (your_docker_id_here)/gem5-parsec-lab:v1.1

# push to docker hub!
docker push yihuachung/gem5-parsec-lab:v1.1
```

#### Outputs:
```bash
(base) ychung79@twhuang-desktop-03:~/ECE757_gem5_workspace$ docker tag gem5-parsec-lab yihuachung/gem5-parsec-lab:v1.1
(base) ychung79@twhuang-desktop-03:~/ECE757_gem5_workspace$ docker push yihuachung/gem5-parsec-lab:v1.1
The push refers to repository [docker.io/yihuachung/gem5-parsec-lab]
e14f9e897b91: Pushed 
da0ce5594790: Pushed 
1e1c6591b61e: Pushed 
6f4ebca3e823: Mounted from library/ubuntu 
2d89bb662b2a: Pushed 
9d7d67b56f83: Pushed 
09adb88bda8c: Pushed 
v1.1: digest: sha256:34d92f9b18bd61360f824bc736d10e9fad43e48cfd01f45bfab8be39471f5827 size: 856
(base) ychung79@twhuang-desktop-03:~/ECE757_gem5_workspace$ 
```


### Success! The image is now live and ready for students to use. Congrats! 