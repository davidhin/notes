# Phoenix HPC

## Important links

| Link                                                         | Desc                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| https://www.adelaide.edu.au/technology/research/high-performance-computing/phoenix-hpc/register-for-an-account | Register for a Phoenix account as an Adelaide uni student    |
| https://wiki.adelaide.edu.au/hpc/index.php/Main_Page         | Adelaide Uni's official wiki                                 |
| https://slurm.schedmd.com/documentation.html                 | Phoenix uses the slurm scheduler for tasks. Additional information can be found here if it is not present in Adelaide uni's wiki |
| https://docs.ycrc.yale.edu/clusters-at-yale/guides/jupyter/  | How to run Jupyter Notebook on a Phoenix job                 |
| https://wiki.adelaide.edu.au/hpc/Job_scripts                 | How to prepare a job script                                  |
| https://wiki.adelaide.edu.au/hpc/Phoenix_data_management     | Information about file system structure                                  |
| https://wiki.adelaide.edu.au/hpc/Partitions_and_QoSs         | Information about maximum resource allocations
| https://ipython.readthedocs.io/en/stable/install/kernel_install.html         | Ipykernel (use conda environments in jupyter notebook)




## Basic Phoenix commands

| Command               | Description                                                  |
| --------------------- | ------------------------------------------------------------ |
| `rcquota`             | Show how much quota you have left                            |
| `rcstat JOBID`        | Show information about a currently running job. JOBID can be found through squeue. |
| `sbatch FILENAME`     | Submit a job to the queue. FILENAME is a job script.         |
| `squeue -u STUDENTID` | Show what jobs you have currently submitted/are running      |
| `scancel JOBID`       | Cancel a submitted job                                       |
| `rcdu`                | Check disk usage                                             |

## Getting Started

**Before starting, an account must be registered with the [University of Adelaide](https://www.adelaide.edu.au/technology/research/high-performance-computing/phoenix-hpc/register-for-an-account)**

First, log in to phoenix. In my case,

```
ssh a1720858@phoenix.rc.adelaide.edu.au
```

Change the shell to use bash (if it is not already using bash). This only needs to be done once.
```
rcshell -s /bin/bash
```

Go to `$FASTDIR` partition.

```
cd $FASTDIR
```

Load Anaconda

```
module load arch/haswell
module load Anaconda3/5.0.1
```

Make sure the pkgs_dirs and envs_dirs are configured correctly, described [here](https://wiki.adelaide.edu.au/hpc/Anaconda#Configuring_your_conda_pkgs_dirs_and_envs_dirs)

Create a Jupyter environment (only need to do this once)

```
conda config --prepend pkgs_dirs /hpcfs/users/aXXXXXXX/myconda/pkgs
conda config --prepend envs_dirs /hpcfs/users/aXXXXXXX/myconda/envs
conda create -yn main anaconda python=3
```

Start the environment

```
source activate main
```

Install any necessary Python libraries. E.g.

```
pip install pandas
```

## Submitting a job script

> https://wiki.adelaide.edu.au/hpc/Job_scripts  

In phoenix, any time you want to use computing power, you have to queue for it. This is done through submitting a file called a Job Script. More details can be found in the phoenix wiki. For most of our scripts, they will look something like this:

In `jobscript.sh`:

```
#!/bin/bash
#SBATCH -p batch
#SBATCH -N 1
#SBATCH -n 4
#SBATCH --time=01:00:00
#SBATCH --mem=4GB

# Execute the program
module load arch/haswell
module load Anaconda3/5.0.1
source activate main
python3 somefile.py
source deactivate
```

You would then submit the job using

```
sbatch jobscript.sh
```

There is a maximum on the amount of CPU cores, RAM, etc you can request depending on the partition chosen. `Batch` automatically selects an appropriate partition. More information at https://wiki.adelaide.edu.au/hpc/Partitions_and_QoSs

## Starting a Jupyter Notebook on a job

> https://docs.ycrc.yale.edu/clusters-at-yale/guides/jupyter/  

Essentially, we use a special job script written like this to submit our job:

In a new file, `start_jupyter.sh`:

```bash
#!/bin/bash
#SBATCH --partition batch
#SBATCH --nodes 1
#SBATCH --ntasks-per-node 12
#SBATCH --mem-per-cpu 1G
#SBATCH --time 48:00:00
#SBATCH --job-name jupyter-lab
#SBATCH --output jupyter-%J.log

# get tunneling info
XDG_RUNTIME_DIR=""
port=$(shuf -i8000-9999 -n1)
node=$(hostname -s)
user=$(whoami)
cluster=$(hostname -f | awk -F"." '{print $2}')

# print tunneling instructions jupyter-log
echo -e "\
Use this command in your terminal:\
ssh -N -L ${port}:${node}:${port} ${user}@${cluster}phoenix.rc.adelaide.edu.au\

Use a Browser on your local machine to go to:\
localhost:${port}  (prefix w/ https:// if using password)
"

# Start notebook
module load arch/haswell
module load Anaconda3/5.0.1
source activate main
export XDG_RUNTIME_DIR=""
jupyter notebook --no-browser --port=${port} --ip=${node}
source deactivate
```

When the job starts running, you look in the generated log file and follow the instructions. This will get the Jupyter Notebook on the Phoenix node to run in your browser.

## Using Job Arrays
Job arrays are a useful feature to allow you to submit many (ideally) small tasks, allowing for true multiprocessing. Often this is done by splitting a file into many smaller files. Implementation is best explained with an example, found on phoenix at `/apps/examples/array_job/`. A more minimal example is also present at https://github.cs.adelaide.edu.au/a1720858/SecurityAnalytics/tree/master/examples/job_array

## File storage
> https://wiki.adelaide.edu.au/hpc/Phoenix_data_management

Most work will be done in the FAST partition, which gives 4TB of storage space to every user. Additional information can be found above. 

## Acknowledging_Phoenix_HPC_in_Publications
Refer to https://wiki.adelaide.edu.au/hpc/Phoenix/Acknowledging_Phoenix_HPC_in_Publications

## Deep Learning with GPU

### Example Job Script

To use GPU(s) in a job script, add --gres=gpu:N to any regular job script. For example, to use one GPU, a job script may look like this: (If using PyTorch, must load appropriate CUDA/CUDNN modules)

```bash
#SBATCH --partition batch
#SBATCH --nodes 1
#SBATCH -c 4
#SBATCH --time 00:10:00
#SBATCH --mem=4GB
#SBATCH --gres=gpu:1

module load arch/haswell
module load CUDA/9.0.176
module load CUDNN/5.1
source activate main
python3 somefile.py
source deactivate
```

### PyTorch Installation

An important note is that Phoenix's Nvidia driver version is **396.44** as of May 12 2020. The following installation has been tested and works as of this date. [source](https://pytorch.org/get-started/previous-versions/)

```
conda install pytorch==1.1.0 torchvision==0.3.0 cudatoolkit=9.0 -c pytorch
```

### Troubleshooting (PyTorch)

If manual troubleshooting is required, the following commands can be used in the terminal of a job on a GPU-enabled node:

- `nvidia-smi`  will show the Nvidia driver information.
- `nvcc --version` will show the currently loaded version of CUDA

In any Python environment of a job on a GPU-enabled node:

```python
import torch

## Check GPU
print("GPU Available: {}".format(torch.cuda.is_available()))
n_gpu = torch.cuda.device_count()
print("Number of GPU Available: {}".format(n_gpu))
print("GPU: {}".format(torch.cuda.get_device_name(0)))

## GPU Debugging
!nvcc --version
torch.backends.cudnn.enabled
print(torch.version.cuda)
```
