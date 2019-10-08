# Dockerfiles for UFO framework
Dockerfiles for x86_64 GNU/Linux that include components of the UFO framework for tomographic data processing. The following Dockerfile is included: ufo-kit-base/Dockerfile

## Features
* Installation of UFO-core and UFO-filters Debian packages
* X-server access from container to run GUI components

## Requirements
* NVIDIA capable GPU
* NVIDIA driver
* Docker 19.03
* nvidia-container-toolkit

Read the updated information about [nvidia-docker](https://github.com/NVIDIA/nvidia-docker). Docker 19.03 supports nvidia without the need for the nvidia-docker wrapper, but nvidia-container-toolkit must be installed to support this. It is suggested to be familiar with basic Docker commands before proceeding.

## Setup
#### GPU driver
The NVIDIA driver for the GPU must be installed on the host.It may be installed alone or along with the CUDA toolkit:

(1) [NVIDIA driver](https://www.nvidia.com/Download/index.aspx); or \
(2) [CUDA toolkit](https://developer.nvidia.com/cuda-toolkit).

The CUDA toolkit version and driver version must be [matched](https://github.com/NVIDIA/nvidia-docker/wiki/CUDA) with the GPU architecture of your device. For example, if you intend to use CUDA 10.1 in your container, you must use Linux x86_64 Driver Version >= 418.39.

#### Build Docker image
Dowload the Dockerfile and save it to a local folder called "ufo". Rename it "Dockerfile." Build the image:

``` shell
$ docker make ufo ufo-kit:local
```
If sucessful, the container can be launched:
``` shell
$ docker run --gpus all -it ufo-kit:local
```
Check the functionality of the container:
``` shell
$ nvidia-smi && clinfo
```

### Run interactive container with GUI and data mounted
A GUI-capable container with a folder mapped to the host can be setup as follows. Please consider the security implications of the steps that modify file permisssions and allow access to the x-server of the host.

1. Permit x-server access from the container. \
Use:
``` shell
$ xhost +local:root
```
If desired, this can be reversed later with:  
``` shell
$ xhost -local:root
```

2. Set file permissions. \
The directory that will be accessible in the container can permits two-way file sharing. A group 1024 was added to the container during the build, so it is necessary that the host is also a member of this group:
``` shell
$ dir=/media/user/test
$ chown :1024 $dir
$ chmod 775 $dir
$ chmod g+s $dir
```

The full command to launch the container with x-server access and data directory mounted:

```shell
$ docker run \
	--gpus all \
	-v /media/user/test:/data \
	-v /tmp/.X11-unix:/tmp/.X11-unix \
	-e DISPLAY=unix$DISPLAY \
	-it ufo-kit:local
```

Consider putting the above command in an executable bash script to launch the container:

``` shell
$ echo 'docker run --gpus all -v /media/user/test:/data \
	-v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY=unix$DISPLAY \
	-it ufo-kit:local' >> ufo-kit-base_launch.sh
$ sudo chmod +x ufo-kit-base_launch.sh
$ ./ufo-kit-base_launch.sh
```

Inside the container, test a ufo-launch command on a grayscale version of lena:
``` shell
$ ufo-launch read path=lena_gs.tif ! blur size=20 sigma=5 ! write filename=blur.tif
$ ufo-launch read path=lena_gs.tif ! calculate expression="'v+1000'" ! write filename=plus1000.tif
```
