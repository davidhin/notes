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

## Basic Phoenix commands

| Command               | Description                                                  |
| --------------------- | ------------------------------------------------------------ |
| `rcquota`             | Show how much quota you have left                            |
| `rcstat JOBID`        | Show information about a currently running job. JOBID can be found through squeue. |
| `sbatch FILENAME`     | Submit a job to the queue. FILENAME is a job script.         |
| `squeue -u STUDENTID` | Show what jobs you have currently submitted/are running      |
| `scancel JOBID`       | Cancel a submitted job                                       |

## Getting Started

> https://wiki.adelaide.edu.au/hpc/Anaconda

First, log in to phoenix. In my case,

```
ssh a1720858@phoenix.rc.adelaide.edu.au
```

Go to `$FASTDIR` partition.

```
cd $FASTDIR
```

Load Anaconda

```
module load Anaconda3/5.0.1
```

Create a Jupyter environment (only need to do this once)

```
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
module load Anaconda3/5.0.1
source activate main
export XDG_RUNTIME_DIR=""
jupyter notebook --no-browser --port=${port} --ip=${node}
source deactivate
```

When the job starts running, you look in the generated log file and follow the instructions. This will get the Jupyter Notebook on the Phoenix node to run in your browser.

## File storage
> https://wiki.adelaide.edu.au/hpc/Phoenix_data_management

Most work will be done in the FAST partition, which gives 4TB of storage space to every user. Additional information can be found above. 
