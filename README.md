# Singularity on Abel

Singularity is a container platform built for HPC-environments. It is
similar to Docker, but doesn't require running a virtual machine, but
instead works as a layer on top of your host system, giving you
transparent access to the host filesystem.

## Usage

### Loading module

```
$ module load singularity
```

### Images

Pre-made images are found in `~eivinabe/nobackup/singularity-images/`:

```
ls ~eivinabe/nobackup/singularity-images/
ubuntu-16.04-cuda-8.0-cudnn-6.img
ubuntu-16.04-cuda-9.0-cudnn-7.img
ubuntu-17.04.img
```

They come with the most common NLP and machine learning packages
installed, as well as CUDA if indicated.

They come with gensim, sklearn, nltk, spacy, tensorflow, keras,
pandas, dynet, pytorch, torchvision and torchtext installed (tell me
if you want additional packages).

### Running shell in container

```
$ singularity shell <img>
```

### Running command in container

```
$ singularity exec <img> <cmd>
```

### Using the GPU

```
$ singularity exec --nv <img> <cmd>
```

### Bind to host directories

```
$ singularity exec -B <path> <img> <cmd>
```

## Building images

You need a linux-machine with root access in order to build a
singularity image. It is possible to use docker URLs directly without
having root, but this doesn't work for Abel because in order to access
folders on the host system, the folder has to be present in the
image. For instance, on Abel we need `/cluster`, `/work` and `/usit`.

### Recipe
To get an image that we can use on Abel we have to build it. Images
can be constructed from scratch, but it is much easier to base on an
existing Docker image. We define the image in a Singularity recipe file:

```
Bootstrap:docker
From:<docker-image-identifier>

%post
    mkdir /cluster /work /usit /projects # Create abel directories
	# Install additional packages
```

To use the GPU, we can use the [CUDA-images from
Nvidia](https://hub.docker.com/r/nvidia/cuda/):

```
Bootstrap:docker
From:nvidia/cuda:9.0-cudnn7-runtime-ubuntu16.04

%post
    mkdir /cluster /work /usit /projects
    touch /usr/bin/nvidia-smi
    apt-get update
    apt-get -y install python3-venv python3-pip git language-pack-en
    pip3 install --upgrade torch tensorflow-gpu
```

### Building

To build the image from recipe, run:

```
# singularity build <img-name> <recipe-file>
```
