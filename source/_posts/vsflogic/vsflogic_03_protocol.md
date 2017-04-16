---
title: VSFLogic实现（三）：协议
date: 2016-07-28 20:00:00
tags: [VSFLogic]
---

*本文为思路笔记，并不能完整描述最终实现方案*

# USB类：VSFLOGIC
1. 控制通道
在USBD中增加一个自定义类，在端点0上提供GET/SET Interface接口，主机用这个接口查询设备信息，发送参数、命令等。
2. 数据通道
建立一组双向BULK端点，端点大小为512，以提供最佳数传速率。

# 缓冲
为追求最大速率，遵循以下规则确定缓冲特性
1. 双缓冲机制，但一个被SGPIO读写时，另一个被USB读写
2. 两个缓冲的总线占用不冲突
3. 尽量大
4. 考虑到LPC43xx全系列的兼容性，不使用M0SUB系统
5. 对于双缓冲的补充：考虑到200M双通道的数据传输压力，改用三缓冲

根据上述规则，确定使用如下缓冲：
1. [0x20000000, 0x20004fff] 20kB
2. [0x2000a000, 0x2000efff] 20kB
3. [0x20005000, 0x20009fff] 20kB (备用)

# 流程简述
1. 上位机发送采集命令
2. VSFLOGIC解析采集命令，并告知SGPIO控制程序（运行于M4或M0SUB）
3. SGPIO控制程序开始采集，填满缓冲后，将事件报告给VSFLOGIC，VSFLOGIC立即提供下一个缓冲（这是关键反应时间）
4. VSFLOGIC收到缓冲后，将其通过USB发送出去
5. USB发送完成后，VSFLOGIC回收缓冲，等待SGPIO控制程序报告采集完成事件
6. 采集命令执行完毕或上位机发送停止命令后，SGPIO控制程序停止，缓冲清洗

# 数据格式约定
数据通道是以流的形式发送数据。所以需要收发双方事先就明确知道数据的组包格式，据此，设立以下约定：
SGPIO控制程序上报缓冲中前16字节保留，按照以下结构提供信息
```
struct vllogic_in_pkt_info_t
{
	uint16_t pkt_size;
	uint16_t samples_count;
	uint32_t samples_start;
	uint8_t logic_unitsize; // bit = 2 ^ logic_unitsize
	uint8_t osc_unitsize;	// bit = 2 ^ osc_unitsize
	uint8_t logic_size;		
	uint8_t osc_size;		
	uint32_t dummy;
};
1. pkt_size：pkt总长度
2. samples_count：pkt中采样总点数
3. samples_start：pkt中起始采样点序号
4. logic_unitsize：每个逻辑通道连续占用数据位位数对2取对数
5. osc_unitsize：每个模拟通道连续占用数据位位数对2取对数
6. logic_size：一个逻辑及模拟共同采样周期内，逻辑数据占用字节数
7. osc_size：一个逻辑及模拟共同采样周期内，模拟数据占用字节数
```
