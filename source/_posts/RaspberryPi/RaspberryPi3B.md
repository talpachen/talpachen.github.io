---
title: Raspberry Pi 3 把玩
date: 2016-04-22 16:22:30
tags: [ARM, GIT, Linux]
---

Pi3 仍然是通过TF卡引导，仍然不支持SATA，这一点差评。
### 固件准备及SSH连接
首先要准备一张TF卡，使用Win32DiskImager烧录一份[镜像](https://www.raspberrypi.org/downloads/raspbian/)，烧录完毕后，将TF卡插入P3 3，然后连上电源及网线。
进入路由器管理界面，看下Pi 3的ip，利用SSH2连接即可，默认用户:pi；默认密码:raspberry
### 更新pi用户密码，及设置root密码
`sudo passwd`

`sudo passwd root`
### 扩展TF卡容量
`sudo raspi-config` 选择 1 ，回车

### 使用国内软件镜像服务器
`cd /etc/apt`

`sudo rm sources.list`

`sudo wget http://mirrors.cqu.edu.cn/distri/Raspbian/sources.list`

`sudo apt-get update`
### 散热性能测试
`sudo apt-get install sysbench`

`sysbench --num-threads=4 --test=cpu --cpu-max-prime=100000 run`

然后再连接一个ssh

`vi check.sh`

```
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_cur_freq
/opt/vc/bin/vcgencmd measure_temp
```

`chmod +x check.sh`

`watch ./check.sh`

这样可以查看到CPU实时频率及温度，一般不超过80度即可
### CPU 测试
`sysbench --num-threads=4 --test=cpu --cpu-max-prime=10000 run`

结果是：
```
Maximum prime number checked in CPU test: 10000


Test execution summary:
    total time:                          45.8467s
    total number of events:              10000
    total time taken by event execution: 183.3321
    per-request statistics:
         min:                                 18.21ms
         avg:                                 18.33ms
         max:                                 37.04ms
         approx.  95 percentile:              18.31ms

Threads fairness:
    events (avg/stddev):           2500.0000/3.54
    execution time (avg/stddev):   45.8330/0.01
```
### 超频
`vi /boot/config.txt`
```
arm_freq=1300
sdram_freq=450
```

`sysbench --num-threads=4 --test=cpu --cpu-max-prime=10000 run`

结果是：
```
Maximum prime number checked in CPU test: 10000


Test execution summary:
    total time:                          42.2620s
    total number of events:              10000
    total time taken by event execution: 168.9996
    per-request statistics:
         min:                                 16.81ms
         avg:                                 16.90ms
         max:                                 34.38ms
         approx.  95 percentile:              16.90ms

Threads fairness:
    events (avg/stddev):           2500.0000/2.92
    execution time (avg/stddev):   42.2499/0.00
```
我的板子是连在路由器USB口上的，供电不足，还是不超了。
### TF卡测试
```
pi@raspberrypi:~$ dd if=/dev/zero of=test bs=4k count=4k oflag=dsync
4096+0 records in
4096+0 records out
16777216 bytes (17 MB) copied, 62.1753 s, 270 kB/s
pi@raspberrypi:~$ dd if=/dev/zero of=test bs=2M count=20 oflag=dsync
20+0 records in
20+0 records out
41943040 bytes (42 MB) copied, 8.58916 s, 4.9 MB/s
```
c4的烂卡
