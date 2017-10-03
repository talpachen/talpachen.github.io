---
title: BLE抓包全程分析——针对FRDM-KW41Z板官方hid_device例程
date: 2017-4-29 16:17:28
tags: [Bluetooth]
---
# 目的
通过对指定例程通讯数据包的分析，直观了解BLE部分运行机制。

# 平台
- 硬件：FRDM-KW41Z
- SDK：MKW41Z_ConnSw_1.0.2 [hid_device]
- 安卓手机：CM12.1，支持BLE
- Sniffer：nRF51822 + USB串口
<!-- more -->
# 流程
1. FRDM-KW41Z上调试运行hid_device例程（部分参数被修改），按一下SW4，开始广播
2. 启动ble-sniffer_win_1.0.1_1111_Sniffer.exe，选中FSL_HID，然后输入Passkey：999999
3. 启动wireshark，开始监听
4. 开启手机蓝牙，配对连接FSL_HID
5. 待鼠标箭头出现并移动一段时间后，断开蓝牙并移除设备
6. 停止wireshark监听

# 分析
## 广播阶段
### ADV_IND
### SCAN_REQ
### SCAN_RSP
