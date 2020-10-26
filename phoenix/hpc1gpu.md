# Notes about GPU in HPC1

### Set up Conda Environment (basics):

```bash
module load Anaconda3/2020.07

conda create -n "main" python=3.8 ipython

conda install -c conda-forge numpy pandas pygithub progressbar2 gensim fastparquet -y
conda install -c anaconda scikit-learn keras-gpu -y

conda activate main
ipython kernel install --name "main-venv" --user
conda deactivate
```

### Run 12 hour GPU job:

```bash
#!/bin/bash
#SBATCH --partition v100 
#SBATCH --nodes 1
#SBATCH -c 4
#SBATCH --time 12:00:00
#SBATCH --mem=32GB
#SBATCH --gres=gpu:1

# This is just to keep the job from automatically closing
module load arch/haswell
module load Anaconda3/5.1.0
module load CUDA/10.2.89
sleep 1d
```

### Use GPU in job array:

```bash
#!/bin/bash
#SBATCH -p v100
#SBATCH -N 1
#SBATCH -n 4
#SBATCH --time=02:00:00
#SBATCH --mem=20000MB
#SBATCH --gres=gpu:1
#SBATCH --array=1-26
#SBATCH --err="_logs/example_%a.err"
#SBATCH --output="_logs/example_%a.out"
#SBATCH --job-name="example"

# Setup Python Environment
module load arch/haswell
module load Anaconda3/2020.07
module load CUDA/10.2.89

source activate main

# Strange workaround - not sure why it works. Reference:
# https://stackoverflow.com/questions/36733179/
# Without it, the wrong python version is used (default instead of conda)
# You can try to remove and see if it still works
source deactivate main
source activate main

# Echo job id into results file
echo "array_job_index: $SLURM_ARRAY_TASK_ID"

# Read inputs
IFS=',' read -ra par <<< `sed -n ${SLURM_ARRAY_TASK_ID}p example.csv`
python -u example.py "${par[0]}" "${par[1]}"
```

### Check GPU in Python
For checking GPUs, use:

```python
import tf
print(tf.config.list_physical_devices("GPU"))
```



