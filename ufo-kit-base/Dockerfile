# Docker file by M. Ryan MacIver, rmcmaciver@hotmail.com, 19 August 2019

FROM nvidia/cuda:10.1-base-ubuntu18.04
FROM nvidia/opencl:runtime-ubuntu18.04

RUN apt-get update && apt-get install -y --no-install-recommends --force-yes \
	python-pip \
	python-qt4 \
	python-opengl \
	python-qt4-gl \
	python-pip \
	libglib2.0-dev \
	python-gi \
	libjson-glib-dev

RUN pip install --upgrade pip
RUN pip install setuptools
RUN pip install pyqtgraph
RUN pip install scipy
RUN pip install numpy==1.15.4
RUN apt-get install git-core -y 

# UFO-KIT software install
RUN apt-get update && apt-get install -y --no-install-recommends --force-yes \
	libufo-bin \
	libufo-dev \
	ufo-filters 

RUN git clone https://github.com/ufo-kit/tofu.git
RUN cd /tofu && python setup.py install

# X-server access: enable GUI with a 'fake' screen
ENV DEBIAN_FRONTEND=noninteractive
RUN     apt-get install -y --no-install-recommends \
	x11vnc \
	xvfb
RUN     mkdir ~/.vnc
RUN     x11vnc -storepasswd 1234 ~/.vnc/passwd

# Data mount: access to host files, create group "1024" for wo way access
ADD . /data
RUN addgroup --gid 1024 mygroup
RUN adduser --disabled-password --gecos "" --force-badname --ingroup mygroup myuser 
USER myuser
