# Using Singularity

#### 1. Research Workflow

**Local Workflow**

1. Develop code locally in a repository linked to Singularity-Hub
2. Upload data to Google Drive (or other)
3. Push code to GitHub

**Running jobs on HPC**

1. Pull and build image from singularity-hub
2. Run HPC Job (Todo: Expand on this)

#### 2. Singularity Installation (local) (needs sudo)

The latest version on Phoenix is currently 2.6.1. To install locally, run:

```bash
VERSION=2.6.1
wget https://github.com/singularityware/singularity/releases/download/$VERSION/singularity-$VERSION.tar.gz
tar xvf singularity-$VERSION.tar.gz
```

Now, there should be a folder called singularity-2.6.1. We have to apply a bug fix here now ([Link to source](https://stackoverflow.com/questions/63061424/)). Then open the file `mount.c` placed in `singularity-2.6.1/src/lib/image/squashfs/mount.c`. In this file find the line of code

```c
if ( singularity_mount(loop_dev, mount_point, "squashfs", MS_NOSUID|MS_RDONLY|MS_NODEV, "errors=remount-ro") < 0 )
```

Change this to

```c
if ( singularity_mount(loop_dev, mount_point, "squashfs", MS_NOSUID|MS_RDONLY|MS_NODEV, "") < 0 )
```

Don't forget to save! Then install. 

```bash
cd singularity-$VERSION
./configure --prefix=/usr/local
make
sudo make install
```

#### 3. Singularity-Hub

To make your builds available through singularity-hub, go to https://singularity-hub.org/collections/my. This way, you can pull singularity image from shub. 

