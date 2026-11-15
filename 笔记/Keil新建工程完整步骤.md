# Keil 新建标准库工程

芯片 `STM32F103C8T6`　环境 `Keil MDK5 + SPL + AC5`

## 1. 目录

```
工程根目录/
├── Start/      启动 + 内核
├── Library/    标准外设库
├── User/       main.c、conf、it
└── Project/    .uvprojx
```

## 2. 新建

1. `Project` → `New μVision Project` → 存到 `Project/`
2. 选器件：`STMicroelectronics` → `STM32F103` → `STM32F103C8`
3. 弹窗提示复制启动文件 → **否**

## 3. 加文件（Manage Project Items）

| 组 | 文件 |
|---|---|
| **Start** | `startup_stm32f10x_md.s`、`system_stm32f10x.c`、`core_cm3.c` |
| **Library** | `stm32f10x_rcc.c`、`stm32f10x_gpio.c`、`misc.c`（用哪个加哪个） |
| **User** | `main.c`、`stm32f10x_conf.h`、`stm32f10x_it.c` |

> ⚠️ Start 组**只加 `_md.s` 一个**。ld/hd/xl 全加会报 `multiply defined`。

## 4. 魔棒配置

| 页 | 设置 |
|---|---|
| **Target** | ✅ Use MicroLIB |
| **C/C++** → Define | `USE_STDPERIPH_DRIVER,STM32F10X_MD` |
| **C/C++** → Include Paths | `..\Start`　`..\Library`　`..\User` |
| **Debug** | 下拉选 `ST-Link Debugger` → Settings → Port `SW` |
| **Flash Download** | ✅ Reset and Run　✅ Erase Sectors |

> 这些设置**每个新工程都要重配**，Keil 不记忆。

## 5. 代码

```c
#include "stm32f10x.h"

void Delay(uint32_t t) { while (t--); }

int main(void)
{
    GPIO_InitTypeDef GPIO_InitStructure;

    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC, ENABLE);

    GPIO_InitStructure.GPIO_Pin   = GPIO_Pin_13;
    GPIO_InitStructure.GPIO_Mode  = GPIO_Mode_Out_PP;
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
    GPIO_Init(GPIOC, &GPIO_InitStructure);

    while (1)
    {
        GPIO_ResetBits(GPIOC, GPIO_Pin_13);
        Delay(5000000);
        GPIO_SetBits(GPIOC, GPIO_Pin_13);
        Delay(5000000);
    }
}
```

## 6. 编译下载

`F7` 编译 → `0 Error`
`F8` 下载 → `Programming Done` / `Verify OK` / `Application running`
PC13 LED 闪烁 = 成功

## 7. 报错对照

| 报错 | 原因 | 解决 |
|---|---|---|
| `undeclared identifier 'RCC_APB2Periph_GPIOC'` | 漏 Define 宏 | 填 `USE_STDPERIPH_DRIVER,STM32F10X_MD` |
| `cannot open source input file "stm32f10x.h"` | Include Paths 没配 | 加 `..\Start` `..\Library` `..\User` |
| `multiply defined` | Start 组加了多个 .s | 只留 `startup_stm32f10x_md.s` |
| 下载成功但灯不闪 | 没勾 Reset and Run / 延时太短 | 勾上；`Delay` 改 `5000000` |

## 8. 快捷键

| 键 | 功能 |
|---|---|
| F7 | 编译 |
| F8 | 下载 |
| F5 / Ctrl+F5 | 进调试 / 退调试 |
| F9 | 断点 |
