---
title: VSFLogic实现（二）：USB Device
date: 2016-07-11 17:13:03
tags: [VSFLogic, USB]
---

# 应用层功能
### 高速数据通道
分配端点：BULK IN 1、BULK OUT 1
功能：JTAG数据通道、逻辑分析仪/虚拟示波器的控制及数据通道

### CDC 物理串口
分配端点：INT IN 2、BULK IN 3、BULK OUT 3
功能：USB 物理串口

### CDC 调试串口
分配端点：INT IN 4、BULK IN 5、BULK OUT 5
功能：内部调试使用

# 驱动层设计
### 初始化
1. 时钟配置
2. 寄存器配置
3. 全局中断配置

### 端点配置，以端点0为例
首先，LPC43XX的这个usb ip需要在内存中建立如下结构：
```
struct td_t
{
	// next dtd pointer
	uint32_t NextTD;

	// dtd token
	uint32_t : 3;
	__IO uint32_t TransactionErr : 1;
	uint32_t : 1;
	__IO uint32_t BufferErr : 1;
	__IO uint32_t Halted : 1;
	__IO uint32_t Active : 1;
	uint32_t : 2;
	uint32_t MultiplierOverride : 2;
	uint32_t : 3;
	__IO uint32_t IntOnComplete : 1;
	__IO uint32_t TotalBytes : 15;
	uint32_t : 1;

	uint32_t BufferPage[5];

	uint32_t reserved;
};

struct ed_t
{
	uint32_t : 15;
	__IO uint32_t IntOnSetup : 1;
	uint32_t MaxPacketSize : 11;
	uint32_t : 2;
	__IO uint32_t ZeroLengthTermination : 1;
	uint32_t Mult : 2;

	uint32_t currentTD;

	__IO struct td_t overlay;

	__IO uint8_t SetupPackage[8];

	uint16_t TransferCount;
	__IO uint16_t IsOutReceived;

	// reserverd
	struct td_t* td_head;
	uint32_t td_num;
	uint32_t reserved;
};
```
每个端点都需要指定一个et_t，每个ed_t都需要连接至少一个td_t。
对于大部分对传输速度要求不是很高的端点，可以简单的使用一个ed_t配一个td_t的方案。

### 高速端口，以bulkio为例
此端口独立拥有一串TD，对于in端点，直接使用高层buffer；对于out端点，独立准备双缓冲buffer空间。
