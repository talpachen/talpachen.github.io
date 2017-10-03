---
title: Sigrok 编译环境
date: 2017-4-25 15:51:21
tags: [VSFLogic, Sigrok]
---

***记录Sigrok及Pulseview编译环境的配置及编译过程***
<!-- more -->
# 平台
- xubuntu-16.04.2-desktop-amd64.iso

# 准备
- `sudo passwd`
- `sudo apt-get update`
- `sudo apt-get upgrade -y`
- `sudo apt-get install openssh-server -y`
- `sudo apt-get install screen vim git-core gcc g++ make cmake libtool autoconf autoconf-archive automake libtool pkg-config libglib2.0-dev libglibmm-2.4-dev libzip-dev libusb-1.0-0-dev libftdi-dev libqt4-dev libboost-test-dev libboost-thread-dev libboost-filesystem-dev libboost-system-dev libqt5svg5-dev qt5-default qtbase5-dev check doxygen python-numpy python-dev python-gi-dev python-setuptools python3-dev swig default-jdk -y`
- `mkdir ~/vllogic`
- `cd ~/vllogic`
- `git clone git://sigrok.org/libsigrok`
- `git clone git://sigrok.org/libsigrokdecode`
- `git clone git://sigrok.org/sigrok-cli`
- `git clone git://sigrok.org/pulseview`

# 编译
## 平台： Ubuntu-17.04
### libsigrok
- `cd ~/vllogic/libsigrok`
- `./autogen.sh`
- `./configure`
- `make -j4`
- `sudo make install`
- `sudo ldconfig /usr/local/lib`

### libsigrokdecode
- `cd ~/vllogic/libsigrokdecode`
- `./autogen.sh`
- `./configure`
- `make -j4`
- `sudo make install`

### sigrok-cli
- `cd ~/vllogic/sigrok-cli`
- `./autogen.sh`
- `./configure`
- `make -j4`
- `sudo make install`

### pulseview
- `cd ~/vllogic/pulseview`
- `cmake .`
- `make -j4`
- `sudo make install`

## 平台： Windows
***[参考](http://sigrok.org/gitweb/?p=sigrok-util.git;a=blob;f=cross-compile/mingw/README;h=27a4aab4f6aa0a42215321b6764bd48876a4891b;hb=HEAD)***
### MXE环境
- `sudo apt-get install autoconf automake autopoint bash bison bzip2 flex gettext git g++ gperf intltool libffi-dev libgdk-pixbuf2.0-dev libtool libltdl-dev libssl-dev libxml-parser-perl make openssl p7zip-full patch perl pkg-config python ruby scons sed unzip wget xz-utils g++-multilib libc6-dev-i386 libtool-bin sdcc nsis -y`
- `cd ~`
- `git clone git://sigrok.org/sigrok-util`
- `git clone https://github.com/mxe/mxe.git mxe-git`
- `cd mxe-git`
- `git checkout fcbe7e30651b5c3be0cbd95541308cd5cbe393f7`
*注意，该版本使用QT5.7.1。截止2017-4-27，PulseView配合QT5.8编译会产生一些链接问题*
- `cp ../sigrok-util/cross-compile/mingw/* ./`
- `patch -p1 < ./mxe_fixes.patch`
- `make MXE_TARGETS=i686-w64-mingw32.static.posix gcc glib libzip libusb1 libftdi1 glibmm qt5 boost check`
- `./sigrok-cross-mingw`
