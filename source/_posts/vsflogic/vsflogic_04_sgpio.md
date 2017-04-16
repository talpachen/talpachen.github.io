---
title: VSFLogic实现（四）：SGPIO
date: 2016-9-29 07:18:34
tags: [VSFLogic]
---

# 外设特点
1. 串行到并行与并行到串行的输入、输出功能
2. 内部双缓冲
3. 支持模式匹配及电平和边沿触发
4. 最大支持8路级联，构成256bit宽度的缓冲

# 驱动API
```
#define SGPIO_MODE_INPUT				0x00000000
#define SGPIO_MODE_OUTPUT				0x00000001
#define SGPIO_MODE_MATCH_RISING			0x00000000
#define SGPIO_MODE_MATCH_FALLING		0x00000010
#define SGPIO_MODE_MATCH_LOW			0x00000100
#define SGPIO_MODE_MATCH_HIGH			0x00000110

vsf_err_t lpc43xx_sgpio_init(void);
vsf_err_t lpc43xx_sgpio_fini(void);
vsf_err_t lpc43xx_sgpio_set_clk(uint32_t clk);
vsf_err_t lpc43xx_sgpio_slice_config(enum sgpio_slice_t io, uint8_t slices, uint32_t sgpio_mode);
vsf_err_t lpc43xx_sgpio_start();

```

# 逻辑信号采集流程
