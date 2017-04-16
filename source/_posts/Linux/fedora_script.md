---
title: Fedora自动脚本
date: 2016-08-05 00:00:00
tags: [Linux]
---

# Fedora24
## 脚本下载

## 脚本内容
```
sudo wget -P /etc/yum.repos.d/
sudo wget -P /etc/yum.repos.d/
sudo wget -P /etc/yum.repos.d/ https://copr.fedorainfracloud.org/coprs/jenslody/codeblocks-release/repo/fedora-24/jenslody-codeblocks-release-fedora-24.repo
sudo dnf makecache

sudo systemctl start sshd.service
sudo systemctl enable sshd.service

sudo dnf install codeblocks -y

```
