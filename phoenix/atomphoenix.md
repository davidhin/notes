# How to use Atom + Hydrogen for Phoenix

This is a tutorial that will show you how to set up your Atom to work with Phoenix.

**Setting up the Jupyter Session on Phoenix**

> Note: Jupyter notebook/lab are used interchangeably below. 

1. Log into Phoenix
2. Submit a job script with `sbatch`. See example (you will want a long --time because otherwise your job will end and your session will get interrupted):

```
#!/bin/bash
#SBATCH --partition batch
#SBATCH --nodes 1
#SBATCH -c 4
#SBATCH --time 12:00:00
#SBATCH --gres=gpu:2
#SBATCH --mem=16GB

# This is just to keep the job from automatically closing
module load Anaconda3/5.0.1
jupyter notebook
```

5. Find the node name using `squeue -u ABC` where ABC is your username / id. 
6. Once your job has been allocated, under the `NODELIST` column, you will find your node ID. For example, `r2n45`. 
7. SSH into the node. For example, `ssh r2n45`. Now you are within your job node instead of the head node, so you can use your allocated resources freely. 
8. In your selected directory, create a script file (e.g. `start.sh`) which starts your Jupyter session. For example:

```
module load Anaconda3/5.0.1
module load Java
module load Singularity
module load git/2.21.0-foss-2016b
source activate main

# get tunneling info
XDG_RUNTIME_DIR=""
# port=$(shuf -i8000-9999 -n1)
port=8995
node=$(hostname -s)
user=$(whoami)
cluster=$(hostname -f | awk -F"." '{print $2}')

# print tunneling instructions jupyter-log
echo -e "
Use this command in your terminal:
ssh -N -L ${port}:${node}:${port} ${user}@${cluster}phoenix.rc.adelaide.edu.au
"

jupyter lab --no-browser --ip="$node" --port="$port"
```

9. You will see something like:

```
Use this command in your terminal:
ssh -N -L 8995:r2n43:8995 a1720858@phoenix.rc.adelaide.edu.au

[I 08:27:10.141 LabApp] JupyterLab alpha preview extension loaded from /apps/software/Anaconda3/5.0.1/lib/python3.6/site-packages/jupyterlab
JupyterLab v0.27.0
Known labextensions:
[I 08:27:10.143 LabApp] Running the core application with no additional extensions or settings
[I 08:27:10.148 LabApp] Serving notebooks from local directory: /fast/users/a1720858/honours/feature_extraction
[I 08:27:10.148 LabApp] 0 active kernels 
[I 08:27:10.148 LabApp] The Jupyter Notebook is running at: http://r2n43:8995/?token=effd5b02b5bf031abb6423bea696788cf9993a751a855350
[I 08:27:10.148 LabApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
[C 08:27:10.164 LabApp] 
    
    Copy/paste this URL into your browser when you connect for the first time,
    to login with a token:
        http://r2n43:8995/?token=effd5b02b5bf031abb6423bea696788cf9993a751a855350
```

10. In a new terminal, paste `ssh -N -L 8995:r2n43:8995 a1720858@phoenix.rc.adelaide.edu.au` to set up port forwarding. At this stage, you can freely go to `localhost:8995` to access your Jupyter session. The root directory of the session will be where you used `bash start.sh`. 

**Connecting the session to Atom**

If you want to use Atom/VS code instead of the Jupyter editor, it is also possible. The steps for VS code are similar and a tutorial can be found online [here](https://code.visualstudio.com/docs/python/jupyter-support#_connect-to-a-remote-jupyter-server). However, in my opinion, Atom works better for managing multiple kernels. 

1. Download and install [Atom](https://atom.io/)
2. Install the [hydrogen]("https://atom.io/packages/hydrogen") package
3. Go to `edit > preferences` and then under "Packages", click the settings button for Hydrogen. There will be a heading called Kernel Gateways. Here, you can specify the notebook (or multiple notebooks if you have submitted multiple jobs). Below is an example of two jobs. The relevant part is the baseUrl, where the port number has to be the same one as the one you have port forwarded above. The token is found in the URL returned by Jupyter. **You will have to manually copy this information into Atom every time you start a new remote Jupyter notebook, because the token changes.**

```
[{
  "name": "Remote server",
  "options": {
    "baseUrl": "http://localhost:8996",
    "token": "07f40ac91a3701efc5d3e8688ca03aa971d4591f4eba3862"
  }},{
    "name": "Remote server 2",
    "options": {
      "baseUrl": "http://localhost:8995",
      "token": "effd5b02b5bf031abb6423bea696788cf9993a751a855350"
    }
}]
```

4. Now, open any file and go to `Packages > Hydrogen > Connect to remote kernel`. You will see the gateway(s) that you specified in step 3. Click the relevant one.
5. Click "new session". This will start a session in your remote kernel. (note: if you clicked a gateway and cannot see "new session", just type some letters into the search bar and then backspace them, do this a few times to refresh it and you'll eventually see "new session"). 
6. Select your session (these are equivalent to the ipython python environments that you would normally set up for Jupyter notebook). Now, you are done.

**Extra: Use Remote-FTP**

You can use Remote-FTP to keep your local and remote code files synced easily. 

1. Install the [remote-ftp]("https://atom.io/packages/remote-ftp") package
2. Go to `packages > remote FTP > create SFTP file`. 
3. Input all the relevant information (host, user, pass, remote). Optional: local. Everything else is optional. 
4. Make sure the package settings is set to always upload on save. Now, your local files are synced to remote. 
5. To sync remote>local and see remote files, you can click `packages > remote FTP > toggle`. 

