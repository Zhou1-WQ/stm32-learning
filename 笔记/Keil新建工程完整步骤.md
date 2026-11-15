# 新建 Keil 标准库工程 · 完整步骤

> 芯片：STM32F103C8T6　|　环境：Keil MDK 5 + 标准外设库（SPL）+ AC5
> 这份是给"重装系统 / 换电脑 / 建新工程"时照着走用的。

## 零、先准备三个文件夹

从标准库模板（江科大资料包）复制一份，结构保持这样：

```
你的工程根目录/
├── Start/      # 启动文件 + 内核文件
├── Library/    # 标准外设库
├── User/       # main.c、conf、it
└── Project/    # Keil 工程文件放这里
```

---

## 一、新建工程

1. Keil → `Project` → `New μVision Project`
2. 保存到 `Project/` 目录，命名如 `Template`
3. 弹出的器件选择：`STMicroelectronics` → `STM32F103` → **`STM32F103C8`** → OK
4. 弹窗 "Copy Startup file..." 选 **否**（我们用模板里的，不复制）

---

## 二、加文件分组（最重要的一步）

点工具栏的 **Manage Project Items**（三个彩色方块图标），建三个组：

| 组名 | 加什么文件 |
|---|---|
| **Start** | `startup_stm32f10x_md.s`、`stm32f10x.h`、`system_stm32f10x.c`、`core_cm3.c`、`core_cm3.h`、`stm32f10x.h` |
| **Library** | `misc.c`、`stm32f10x_gpio.c`、`stm32f10x_rcc.c`、`stm32f10x_usart.c`、`stm32f10x_tim.c`、`stm32f10x_adc.c`、`stm32f10x_can.c` 等（用哪个加哪个，也可以全加） |
| **User** | `main.c`、`stm32f10x_conf.h`、`stm32f10x_it.c`、`stm32f10x_it.h` |

⚠️ **Start 组里只加 `startup_stm32f10x_md.s` 这一个 .s 文件**。文件夹里还躺着 ld/hd/xl/_vl 几个，全加进去会报"重复定义"——每个启动文件里都有同名的中断向量表。

---

## 三、魔棒配置（Options for Target）

点工具栏的**魔术棒**图标，逐页设置：

### 1. Target 页
- ✅ **Use MicroLIB**（不勾，后面 printf 重定向会卡死）

### 2. C/C++ 页
- **Define** 框填：
  ```
  USE_STDPERIPH_DRIVER,STM32F10X_MD
  ```
  - 逗号必须英文
  - 漏了 `USE_STDPERIPH_DRIVER` → 所有 `RCC_xxx`/`GPIO_xxx` 报 "use of undeclared identifier"
  - 漏了 `STM32F10X_MD` → 芯片容量档位不对

- **Include Paths** 点右边 `...` 添加三个：
  ```
  ..\Start
  ..\Library
  ..\User
  ```

### 3. Debug 页
- 右上下拉框选 **`ST-Link Debugger`**
- 点旁边 **Settings** → 确认 `Port: SW`、`SW Device` 里能读到 `IDCODE 0x1BA01477`

### 4. Debug → Settings → Flash Download 页
- ✅ **Reset and Run**（烧完自动复位运行）
- ✅ Erase Sectors
- Programming Algorithm 里要有 **`STM32F10x Med-density Flash 128K`**，没有就点 Add 添加

> 这里的设置**每个新工程都要重配一次**，Keil 不记忆。

---

## 四、写点灯代码

`User/main.c`：

```c
#include "stm32f10x.h"

void Delay(uint32_t t) { while (t--); }          // 空循环延时

int main(void)
{
    GPIO_InitTypeDef GPIO_InitStructure;

    RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC, ENABLE);   // 开 GPIOC 时钟

    GPIO_InitStructure.GPIO_Pin   = GPIO_Pin_13;          // PC13
    GPIO_InitStructure.GPIO_Mode  = GPIO_Mode_Out_PP;     // 推挽输出
    GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;     // 速度档位
    GPIO_Init(GPIOC, &GPIO_InitStructure);                // 写入寄存器

    while (1)
    {
        GPIO_ResetBits(GPIOC, GPIO_Pin_13);   // 低电平 → 亮
        Delay(5000000);
        GPIO_SetBits(GPIOC, GPIO_Pin_13);     // 高电平 → 灭
        Delay(5000000);
    }
}
```

---

## 五、编译与下载

- **F7** 编译 → `0 Error(s)`
- **F8** 下载 → `Programming Done.` / `Verify OK.` / `Application running...`
- 板载 LED（PC13）开始闪烁 = 成功

---

## 六、这张表里有四个必错点

| 现象 | 原因 | 解决 |
|---|---|---|
| `use of undeclared identifier 'RCC_APB2Periph_GPIOC'` | 漏了 Define 宏 | 填 `USE_STDPERIPH_DRIVER,STM32F10X_MD` |
| `cannot open source input file "stm32f10x.h"` | Include Paths 没配 | 加 `..\Start` `..\Library` `..\User` |
| `multiply defined` 一堆 | Start 组加了多个 .s | 只留 `startup_stm32f10x_md.s` |
| 下载成功但灯不闪 | Reset and Run 没勾 / 或延时太短 | 勾上 Reset and Run；延时放大 10 倍 |

---

## 七、常用快捷键

| 键 | 功能 |
|---|---|
| F7 | 编译 |
| F8 | 下载（LOAD） |
| F5 | 进入调试（在线仿真） |
| Ctrl+F5 | 退出调试 |
| F9 | 加/删断点 |

---

*本文档随实践更新。新踩的坑都往第六节的表里加。*
