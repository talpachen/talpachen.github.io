---
title: CloudLoader（一）：一个开源单片机OTA通用方案
date: 2017-4-11 21:04:44
tags: [MCU, OTA, CloudLoader]
---
# 由头
前几天，EEWorld社区内有童靴提出要搞些物联网方面的事情，但缺乏点子，不知道搞捣鼓些什么东西。看到他们的讨论后，我突然想到近些日子常要对一些设备做远程升级，由于设备的Bootloader比较传统，还需要我远程实时配合操作人员进行，过程很麻烦。在这个事情上，倘若把设备的Bootloader改成OTA方式，我只要把固件放置到服务器上，通知操作人员，操作人员对设备重新上一次电即可。我想，这应该也是一个物联网应用，对于有OTA需求的设备来讲也是有意义的，当然开源且易部署也是必要条件。
<!-- more -->
# 设计目标
1. CloudLoader与APP相互独立，代码上互不干涉
2. 传输安全性保障
3. 提供一个硬件通讯通道，APP可以利用此通道与云端通讯（比如串口）

# 软件平台
基于[VSF](https://github.com/versaloon/vsf) （[历史版本](https://github.com/talpachen/vsf_2014-2016)）