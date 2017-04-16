---
title: Wireshark之路由器监听工具
date: 2016-7-20 10:25:49
tags: [net]
---

# 现成可执行文件
在mips或arm架构路由器下面可以直接使用下面的可执行文件
## [Yrpcapd_mips](rpcapd)
## [Yrpcapd_arm](rpcapd.arm)

```
wget  // rpcapt
chmod +x ./rpcapt
rpcapd -n –d
```
回到客户端，打开wireshark，捕获远程接口中填入服务端IP及默认的2002端口即可


# 自行编译参考
```
PATH=$PATH:/home/llp/openwrt/openwrt/staging_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2/bin
export PATH
STAGING_DIR=/home/llp/openwrt/openwrt/staging_dir/toolchain-mipsel_24kec+dsp_gcc-4.8-linaro_uClibc-0.9.33.2
export STAGING_DIR
CC=mipsel-openwrt-linux-gcc CXX=mipsel-openwrt-linux-g++ AR=mipsel-openwrt-linux-ar RANLIB=mipsel-openwrt-linux-ranlib ac_cv_linux_vers=2 ./configure --host=mipsel-openwrt-linux --with-pcap=linux


ac_cv_linux_vers=2 ./configure --build=x86_64-unknown-linux-gnu --host=mipsel-openwrt-linux --with-pcap=linux

# winpcap/wpcap/libpcap/pcap-int.h 里加上一行 #include <string.h>

cd rpcapd

#vi Makefile
#CC=mipsel-openwrt-linux-gcc

make
```

# 其他参考
http://rick008.blog.51cto.com/3349394/1585279
