---
title: 在Raspberry Pi 3上搭建Git Server (补充Repo Mirrors)
date: 2016-04-25 14:11:55
tags: [ARM, GIT, Linux]
---

2016-10-31 补充：
1. 更换repo仓库地址:
下载 [附件](repo) 并放入~/bin
```
chmod a+x ~/bin/repo
export REPO_URL='https://gerrit-google.tuna.tsinghua.edu.cn/git-repo'
echo "export REPO_URL='https://gerrit-google.tuna.tsinghua.edu.cn/git-repo'" >> ~/.bashrc
```
2. 定期更新
```
crontab -e
在文件中添加以下内容:
0 3 * * * cd ~/aosp && repo sync -f -j4
```
3. 提供






------------------------------------------------------------------

# 准备
给SD卡刷好镜像后，在u盘/config.txt中加入下面三行：
```
arm_freq=1300
sdram_freq=500
over_voltage=2
```
将SD卡及USB硬盘都连上Pi3，上电

# 软件安装
先更新[国内软件源](https://mirrors.tuna.tsinghua.edu.cn/help/raspbian/)

`sudo apt-get update`

`sudo apt-get remove vim-tiny vim-common`

`sudo apt-get install nfs-kernel-server git vim sqlite3`

# USB硬盘处理
使用fdisk将硬盘分为两个区，P1为SWAP，2G，P2未ext4，将P2挂载到/home上

`sudo vi /etc/fstab`
```
proc            /proc           proc    defaults          0       0
/dev/mmcblk0p1  /boot           vfat    defaults          0       2
/dev/mmcblk0p2  /               ext4    defaults,noatime  0       1
/dev/sda1       swap            swap    defaults          0       0
/dev/sda2       /home           ext4    defaults          0       0
# a swapfile is not a swap partition, no line here
#   use  dphys-swapfile swap[on|off]  for that
```
重启

# NFS配置
暂时不启用

# REPO仓库
建立一个repo用户，专门用于建立大型repo的本地mirror
```
sudo useradd repo
sudo passwd repo
sudo mkdir /home/repo
sudo chown repo:repo /home/repo
```
准备一个repo脚本

`sudo vi /usr/local/bin/repo` 拷贝[附件](repo)内容

`sudo chmod a+x /usr/local/bin/repo`

`su repo && cd ~`

尝试建立AOSP的mirror：
```
mkdir aosp
cd aosp
repo init -u https://aosp.tuna.tsinghua.edu.cn/mirror/manifest --mirror
repo sync
```
拉取正常
```
cd ~
mkdir .ssh
vi vi authorized_keys #将需要访问repo用户的ssh公钥填进去
```
尝试拉取一个库`/home/repo/aosp/toolchain/mclinker.git`
```
git clone repo@<ip>:aosp/toolchain/mclinker.git
```
成功，REPO仓库可用。

# Gogs服务
建立一个gogs用户，用于建立一个类似Github的服务
```
sudo useradd gogs
sudo passwd gogs
sudo mkdir /home/gogs
sudo chown repo:repo /home/gogs
su gogs
cd ~
wget https://dl.gogs.io/gogs_v0.9.13_raspi2.zip
unzip gogs_v0.9.13_raspi2.zip
sqlite3 gogs.db #建立数据库
./gogs/gogs web #安装
```
服务自启动

`vi ~/gogs/scripts/systemd`
```
[Unit]
Description=Gogs (Go Git Service)
After=syslog.target
After=network.target
#After=mysqld.service
#After=postgresql.service
#After=memcached.service
#After=redis.service

[Service]
# Modify these two values and uncomment them if you have
# repos with lots of files and get an HTTP error 500 because
# of that
###
#LimitMEMLOCK=infinity
#LimitNOFILE=65535
Type=simple
User=gogs
Group=gogs
WorkingDirectory=/home/gogs/gogs
ExecStart=/home/gogs/gogs/gogs web
Restart=always
Environment=USER=gogs HOME=/home/gogs

[Install]
WantedBy=multi-user.target
```
```
su
cp /home/gogs/gogs/scripts/systemd/gogs.service /etc/systemd/system/
systemctl enable gogs
systemctl start gogs  
systemctl status gogs
```
至此，GOGS服务搭建完成

# 备份
需对整个TF卡进行备份，将TF卡拔出
使用读卡器连接linux
```
[llp@llp Desktop]$ sudo fdisk -l /sdc
[sudo] password for llp:
fdisk: cannot open /sdc: No such file or directory
[llp@llp Desktop]$ sudo fdisk -l /dev/sdc
Disk /dev/sdc: 3.7 GiB, 3965190144 bytes, 7744512 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x6f92008e

Device     Boot  Start     End Sectors  Size Id Type
/dev/sdc1         8192  131071  122880   60M  c W95 FAT32 (LBA)
/dev/sdc2       131072 2658303 2527232  1.2G 83 Linux


dd if=/dev/sdc of=./raspberry_git_20160425.img bs=512 count=2658304
```
这个img文件就与官方提供的img类似了
