# Running SoftGym inside a docker

We provide both Dockerfile and pre-built Docker container for compiling SoftGym. Part of the docker solutions are borrowed from [PyFlex](https://github.com/YunzhuLi/PyFleX/blob/master/bindings/docs/docker.md) 

## Prerequisite

- Install [docker-ce](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
- Install [nvidia-docker](https://github.com/NVIDIA/nvidia-docker#quickstart)
- Install [Anaconda/Miniconda](https://www.anaconda.com/distribution/)
- Install Pybind11 using `conda install pybind11`

## Running pre-built Dockerfile

- Pull the pre-built docker file

```
sudo docker pull xingyu/softgym
```

- Assuming you are using conda, using the following command to run docker, 
which will mount the python environment and SoftGym into the docker container. 
Make sure you have replaced `PATH_TO_SoftGym` and `PATH_TO_CONDA` with the corresponding paths (make sure to use absolute path!).

```
nvidia-docker run \
  -v PATH_TO_SoftGym:/workspace/softgym \
  -v PATH_TO_CONDA:PATH_TO_CONDA \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  --gpus all \
  -e DISPLAY=$DISPLAY \
  -e QT_X11_NO_MITSHM=1 \
  -it xingyu/softgym:latest bash
```
As an example:
```
nvidia-docker run \
  -v ~/softgym:~/softgym \
  -v ~/software/miniconda3/:~/software/miniconda3/ \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  --gpus all \
  -e DISPLAY=$DISPLAY \
  -e QT_X11_NO_MITSHM=1 \
  -it xingyu/softgym:latest bash
```
This solution follows [this tutorial]( https://medium.com/@benjamin.botto/opengl-and-cuda-applications-in-docker-af0eece000f1) for running GL and CUDA application inside the docker. 

- Now you are in the Docker environment. Go to the softgym directory, create a conda env, set PATH and compile PyFlex

```
cd softgym
export PATH="PATH_TO_CONDA/bin:$PATH"
export PYFLEXROOT=${PWD}/PyFlex
export PYTHONPATH=${PYFLEXROOT}/bindings/build:$PYTHONPATH
export LD_LIBRARY_PATH=${PYFLEXROOT}/external/SDL2-2.0.4/lib/x64:$LD_LIBRARY_PATH

conda create -n softgym #you can add here packages, not needed
conda activate softgym

./compile_1.0.sh
```
- Now that PyFleX has properly compiled. You can move outside docker (`Ctrl+D`), export the environment variables and start playing with the examples.

```
cd repos/softgym

export PATH="PATH_TO_CONDA/bin:$PATH"
export PYFLEXROOT=${PWD}/PyFlex
export PYTHONPATH=${PYFLEXROOT}/bindings/build:$PYTHONPATH
export LD_LIBRARY_PATH=${PYFLEXROOT}/external/SDL2-2.0.4/lib/x64:$LD_LIBRARY_PATH

conda activate softgym
#Running an example

python examples/random_env.py --env_name ClothFlatten

#Probably missing a lot of packages, to install them:
conda install -c conda-forge pkgname
#For example
conda install -c conda-forge numpy

```
- If running the example fails to `import softgym.*` this is probably due to `PYTHONPATH` issues and you should make sure
the interpreter knows where to look for softgym package. More info on [PYTHONPATH]( https://docs.python.org/3/using/cmdline.html#envvar-PYTHONPATH).

## Running with Dockerfile

We also posted the [Dockerfile](Dockerfile). To generate the pre-built file, download the Dockerfile in this directory and run
```
docker build -t softgym .
```
in the directory that contains the Dockerfile.
