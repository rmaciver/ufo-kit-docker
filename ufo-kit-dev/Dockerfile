# Based on docker draft by Harinarayan Krishnan <hkrishnan@lbl.gov>

# RM: pull original dockerfile from ufo-kit gitrepo
# RM: change to nvidia/cuda:10.1-devel-ubuntu16.04
FROM nvidia/cuda:10.1-devel-ubuntu16.04


RUN apt-get update && apt-get install -y --no-install-recommends --force-yes \
  git \
  gcc \
  g++ \
  gobject-introspection \
  make \
  ca-certificates \
  cmake \
  liblapack-dev \
  libjpeg-dev \
  libglib2.0-dev \
  libjson-glib-dev \
  libopenmpi-dev \
  libhdf5-dev \
  libclfft-dev \
  libgsl-dev \
  libgirepository1.0-dev \
  python \
  python-dev \
  python-gobject \
  python-pip \
  pkg-config \
  ocl-icd-opencl-dev \
  fftw3-dev \
  clinfo \
  zlib1g-dev && \
  rm -rf /var/lib/apt/lists/*

RUN apt-get update && apt-get install -y --no-install-recommends --force-yes \
  libtiff5-dev

ENV LD_LIBRARY_PATH /usr/local/lib/:${LD_LIBRARY_PATH}

RUN mkdir -p /etc/OpenCL/vendors && \
    echo "libnvidia-opencl.so.1" > /etc/OpenCL/vendors/nvidia.icd

RUN rm /usr/local/cuda-10.1/targets/x86_64-linux/lib/libOpenCL.so*

RUN git clone https://github.com/ufo-kit/ufo-core.git
RUN git clone https://github.com/ufo-kit/ufo-filters.git
RUN git clone https://github.com/ufo-kit/tofu

RUN pip install --upgrade pip
RUN pip install setuptools

#RM: add pinned numpy install
RUN pip install numpy==1.15.4

# RM: replace with call to 0.14.0
#RUN cd /ufo-core-0.14.0 && mkdir build && cd build && cmake -DCMAKE_INSTALL_PREFIX=/usr .. && make install
RUN cd /ufo-core && mkdir build && cd build && cmake -DCMAKE_INSTALL_PREFIX=/usr .. && make install
RUN cd /ufo-filters && mkdir build && cd build && cmake ..  -DCMAKE_INSTALL_PREFIX=/usr && make install

# RM: bump down tofu install, put dep's first
# RM: add python QT bindings and python-qt4-gl
RUN apt-get update && apt-get install -y --no-install-recommends --force-yes \
  python-opengl \
  python-qt4 \
  python-qt4-gl

# RM: install pinned scipy before pyqtgraph, otherwise a python3 version is installed
RUN pip install scipy==0.14.0
RUN pip install pyqtgraph
RUN cd /tofu && python setup.py install

# RM: ez_ufo dep's and installed
RUN apt-get update && apt-get install -y --no-install-recommends --force-yes\
	python-tk

RUN git clone https://github.com/ufo-kit/concert.git
RUN cd /concert && python setup.py install

# RM: add X-server access
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y --no-install-recommends --force-yes\
	x11vnc \
	xvfb
RUN     mkdir ~/.vnc
RUN     x11vnc -storepasswd 1234 ~/.vnc/passwd

# RM:  access to host files, create group in host "1024" for two way access
ADD . /data
RUN addgroup --gid 1024 mygroup
RUN adduser --disabled-password --gecos "" --force-badname --ingroup mygroup myuser
USER myuser
