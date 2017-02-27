---
layout: post
comments: false
title:  "ClickHouse on Centos 7"
date:   2017-02-23 00:00:00 +0000
categories: ClickHouse
---

Instructions to build ClickHouse on Centos 7

The folks over at ClickHouse have some instructions to build binaries for Debian [here](https://github.com/yandex/ClickHouse/blob/master/doc/build.md). If you need to build a RPM, you can follow the steps mentioned [here](https://github.com/redsoftbiz/clickhouse-rpm).

This note is meant for those who simply need to get ClickHouse running on Centos 7 without the explicit need to get RPMs in place. I am going to try and map these instructions to those mentioned by the ClickHouse build instructions (wherever possible).

**Test for SSE 4.2**

    grep -q sse4_2 /proc/cpuinfo && echo "SSE 4.2 supported" || echo "SSE 4.2 not supported"

**Install GCC**

(The default GCC on Centos is at version 4.8.x. You'll need GCC 5+ to build ClickHouse. That step comes up in a bit.)

    sudo yum install gcc gcc-c++

**Install Git, CMake and wget**

    sudo yum install git cmake wget

**Detect number of threads**

    export THREADS=$(grep -c ^processor /proc/cpuinfo) 

**Install GCC 6 from sources**

    wget ftp://ftp.fu-berlin.de/unix/languages/gcc/releases/gcc-6.3.0/gcc-6.3.0.tar.bz2
    tar xf gcc-6.3.0.tar.bz2
    cd gcc-6.3.0
    ./contrib/download_prerequisites
    cd ..
    mkdir gcc-build
    cd gcc-build
    ../gcc-6.3.0/configure --prefix=/home/vagrant/install/gcc-6.3.0 --enable-languages=c,c++ --disable-multilib
    make -j $THREADS
    sudo make install
    /home/vagrant/install/gcc-6.3.0/bin/gcc --version
    sudo ln -s /home/vagrant/install/gcc-6.3.0/bin/gcc /usr/local/bin/gcc-6
    sudo ln -s /home/vagrant/install/gcc-6.3.0/bin/g++ /usr/local/bin/g++-6
    sudo ln -s /home/vagrant/install/gcc-6.3.0/bin/gcc /usr/local/bin/cc
    sudo ln -s /home/vagrant/install/gcc-6.3.0/bin/g++ /usr/local/bin/c++

**Use GCC 6 for builds**

    export CC=gcc-6
    export CXX=g++-6
    
**Install required libraries from packages**

    sudo yum install libicu-devel readline-devel openssl-devel unixODBC-devel libtool-ltdl-devel
    
**Install MySQL dependencies**

    wget http://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm
    sudo yum -y --nogpgcheck install mysql57-community-release-el7-9.noarch.rpm
    sudo yum -y install mysql-community-devel
    sudo ln -s /usr/lib64/mysql/libmysqlclient.a /usr/lib64/libmysqlclient.a
    
    
    

    
....
