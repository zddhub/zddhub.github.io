---
layout: post
title: Recognizer configuration on CentOS
category: Fun
tags: OpenSSE OpenCV boost CentOS
---

> I usually forget some things.

Today, I reinstall my centos system and configure recognizer on it (Recognizer is my online version demo, powered by OpenSSE v1.10. You can get an offline version on my [Github](https://github.com/zddhub/opensse)). This work actually took me two hours, it's unbearable thing! So, I decide to record.

<!-- more -->

Until now, OpenSSE only dependent on OpenCV and boost (Core engine has canceled Qt library in new version). We start from OpenCV.

## OpenCV install

* Install all the required packages using yum

```sh
yum groupinstall "Development Tools"
yum install gcc
yum install cmake
yum install git
yum install gtk2-devel
yum install pkgconfig
yum install numpy 
yum install ffmpeg
```

* Create working directory and check out the source code

```sh
mkdir /opt/working
cd /opt/working
git clone https://github.com/Itseez/opencv.git
cd opencv
git checkout tags/2.4.8.2
```

You can chooce any OpenCV version not less than 2.4.

* Create the Makefile

```sh
mkdir release
cd release
cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local ..
```

* Build and install

```sh
cd /opt/working/opencv/release
make
make install
```

* Add OpenCV to system

```sh
cd /etc/ld.so.conf.d/
echo "/usr/local/lib/" > opencv.conf
sudo ldconfig
```

## Boost install

```sh
wget http://sourceforge.net/projects/boost/files/boost/1.55.0/boost_1_55_0.tar.gz
tar -zxvf boost_1_55_0.tar.gz
./bootstrap.sh --prefix=/usr/local
./b2 install --with=all
``` 

## OpenSSE install

```sh
cd opensse/src
mkdir build
cd build
cmake ..
make
make install
```

## Recognizer (OpenSSE online version) build

Make sure database has a correct path before build recognizer(`params.json` and `recognize.cpp`), and then `mkdir /tmp/opensse-tmp/` to cache temp image.

Run web api, everything is going well, Do you want to [try](https://online.opensse.com/)?

