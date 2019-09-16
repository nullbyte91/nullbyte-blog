---
layout: post
title:  "Chroot Jail From Docker Image"
categories: [ Embedded ]
image: assets/images/chroot_jail.jpg
---

I got a tast to port some of the pakages like ROS, Tensorflow on Embedded target which has busyBox as main root file system. Technicaly
the system doesn't have any package manager. 

So, I decide to create ARM Docker image on X86 system and Installing all the package inside docker and ship to embedded target.
Yes. It is possible to do that.

## Create a ARM Chroot jail from Docker Image

### Install Docker

```bash
# Uninstall old version
sudo apt-get remove docker docker-engine docker.io containerd runc

# Set up the repository
sudo apt-get update

sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo apt-key fingerprint 0EBFCD88

# on X86/amd64
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"

# Install docker engine
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io

# To verify docker able to pull the hello-world image
sudo docker run hello-world
```

### Emulating ARM on x86_64 system

```bash
sudo apt-get install gcc-arm-linux-gnueabihf libc6-dev-armhf-cross qemu-arm-static

# Create workspace folder
mkdir -p ~/arm_container/

# Copy Qemu binary to workspace folder
cp /usr/bin/qemu-aarch64-static ~/arm_container/

# Create Docker file
touch ~/arm_container/Dockerfile
```

### Docker file to install ROS and OpenCV

```bash
# Copy the below content to Dockerfile
FROM arm64v8/ubuntu:xenial
COPY qemu-aarch64-static /usr/bin/qemu-aarch64-static

# Forward system proxy setting
#ARG proxy
#ENV http_proxy $proxy
#ENV https_proxy $proxy

# Basic apt update
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends locales ca-certificates &&  rm -rf /var/lib/apt/lists/*

# Set the locale to en_US.UTF-8, because the Yocto build fails without any locale set.
RUN locale-gen en_US.UTF-8 && update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LC_ALL en_US.UTF-8

# Again, off the certificare
RUN echo "check_certificate = off" >> ~/.wgetrc
RUN echo "[global] \n\
trusted-host = pypi.python.org \n \
\t               pypi.org \n \
\t              files.pythonhosted.org" >> /etc/pip.conf

# Get basic packages
RUN apt-get update && apt-get install -y \
    apparmor \
    aufs-tools \
    automake \
    bash-completion \
    btrfs-tools \
    build-essential \
    cmake \
    createrepo \
    curl \
    dpkg-sig \
    g++ \
    gcc \
    git \
    iptables \
    jq \
    libapparmor-dev \
    libc6-dev \
    libcap-dev \
    libsystemd-dev \
    libyaml-dev \
    mercurial \
    net-tools \
    parallel \
    pkg-config \
    python-dev \
    python-mock \
    python-pip \
    python-setuptools \
    python-websocket \
    golang-go \
    iproute2 \
    iputils-ping \
    vim-common \
    vim \
    --no-install-recommends

# ROS Package installation
RUN apt-get update && apt-get install -y lsb-release sudo
RUN lsb_release=$(cat /etc/lsb-release |grep DISTRIB_CODENAME | cut -f2 -d"=")
RUN echo ${lsb_release}
RUN ls /etc/lsb-release
RUN cat /etc/lsb-release
RUN rm -rf /etc/apt/sources.list.d/ros-latest.list
RUN echo "deb http://packages.ros.org/ros/ubuntu xenial main" >> /etc/apt/sources.list.d/ros-latest.list
RUN cat /etc/apt/sources.list.d/ros-latest.list
RUN apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
RUN apt-get update && apt-get install -y ros-kinetic-ros-base
RUN rosdep init
RUN rosdep update

RUN echo "source /opt/ros/kinetic/setup.bash" >> ~/.bashrc
RUN source ~/.bashrc

# apt-key' to verify signature
RUN rm -rf /tmp
RUN mkdir /tmp
RUN chmod 1777 /tmp

# Opencv install
RUN apt-get update && apt-get upgrade
RUN apt-get install -y build-essential cmake unzip pkg-config \
    libjpeg-dev libpng-dev libtiff-dev \
    libavcodec-dev libavformat-dev libswscale-dev libv4l-dev \
    libxvidcore-dev libx264-dev \
    libgtk-3-dev libatlas-base-dev gfortran python3-dev wget


RUN mkdir -p /opt  && cd /opt && wget -O opencv.zip https://github.com/opencv/opencv/archive/4.0.0.zip &&  \
    unzip opencv.zip && mv opencv-4.0.0 opencv && \
    wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.0.0.zip && \
    unzip opencv_contrib.zip && mv opencv_contrib-4.0.0 opencv_contrib
RUN cd /opt/opencv && mkdir -p build && cd build && \
    cmake -D CMAKE_BUILD_TYPE=RELEASE \
        -D CMAKE_INSTALL_PREFIX=/usr/local \
        -D INSTALL_PYTHON_EXAMPLES=OFF \
        -D INSTALL_C_EXAMPLES=OFF \
        -D OPENCV_ENABLE_NONFREE=ON \
        -D OPENCV_EXTRA_MODULES_PATH=/opt/opencv_contrib/modules \
        -D OPENCV_GENERATE_PKGCONFIG=ON \
        -D OPENCV_PC_FILE_NAME=opencv.pc \
        -D BUILD_EXAMPLES=ON .. && \
    make -j8 && make install & ldconfig
```

### Build and Ship the docker image to Embedded Target

```bash
cd ~/arm_container/
# Build the ARM docker image
docker build --rm --build-arg proxy=$http_proxy --rm --tag arm_container .
# Note: Remove proxy if you are not using corporate proxy

# ZIP ARM Chroot from Docker Image
docker run -ti --rm arm_container:latest tar -cvpzf chroot.tar.gx --one-file-system /

# copy to the target system by using SCP
scp chroot.tar.gx username@ip_address:/home/

```

### Setup chroot jain on target side 

```bash
# Target side
cd /home/
mkdir chroot
mv chroot.tar.gx chroot
cd chroot
tar -xvf chroot.tar.gx

# Chroot we have to mount dynamic creation directory
mount -t proc proc /mnt/proc/
mount --rbind /sys/ /mnt/sys/
mount --rbind /dev/ /mnt/dev/

# Chroot to the new file systemc
chroot /home/chroot

#Execute bash to overcome overflow issue
/bin/bash

#Timezone issues - We have to configure tiemzone:
apt-get install -y tzdata
dpkg-reconfigure tzdata
```

<b>Note</b>:

This steps tested on Ubuntu 16.04 X86_64 System.