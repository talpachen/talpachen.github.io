---
title: Modelsim + Debussy 工具使用
date: 2016-10-07 13:18:51
tags: [FPGA]
---

# 参考
1. [怎样使用Debussy+ModelSim快速查看前仿真波形](http://www.cnblogs.com/yuphone/archive/2010/05/31/1747871.html)

# 软件准备
1. modelsim-win32-10.2c-se.exe
2. Debussy-54v9-NT.exe
3. 拷贝文件 .\Debussy\share\PLI\modelsim_pli\WINNT\novas.dll 至文件夹 .\modeltech_10.2c\win32
4. 编辑.\modeltech_10.2c\modelsim.ini
```
将
; Veriuser = veriuser.sl
替换为
Veriuser = novas.dll
```

# 测试用例
[例程1(点击下载)](test.zip)
[例程2(点击下载)](串转并测试.zip)
解压后，双击bat文件即可
