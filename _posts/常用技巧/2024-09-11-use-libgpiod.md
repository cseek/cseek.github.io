---
title: 在 jetson orin nx 上 使用 libgpiod 库操作 gpio
categories: [常用技巧]
tags: [libgpiod]
---

> Ubuntu 22.04 使用较新的内核（5.15+），默认已弃用旧的 sysfs GPIO接口（`CONFIG_GPIO_SYSFS`），改用新的字符设备接口（`CONFIG_GPIO_CDEV`）。所以 `/sys/class/gpio` 并不存在，新内核推荐使用 libgpiod 工具操作GPIO，有命令行工具，也有 c 语言操作接口。下面就以 `jetson orin nx` 为例讲述一下如何使用。
{: .prompt-info }

## 安装 libgpiod 命令行工具
一般情况我只会用到 gpioinfo，用来查找 chip 和 line 的编号。
```bash
# 安装
sudo apt install gpiod
# 查看可用 GPIO 芯片
gpiodetect
# 查看 GPIO 引脚信息（例如芯片0）
gpioinfo 0
# 控制 GPIO
sudo gpioset 0 17=1
# 设置io电平，并保持，直到ctrl + c或者 回车
sudo gpioset -m wait gpiochip0 43=0
# 查看 gpio 的电平状态
cat /sys/kernel/debug/gpio
```
## 通过引脚对照表找到引脚对应的 GPIO 编号

![](/assets/img/post/常用技巧/orin-pin.png)

由此可得:

| 引脚 | gpio 编号 |
|------|----------|
|   7  |  GPIO09 |
|   15 |  GPIO12 |
|   29 |  GPIO01 |
|   31 |  GPIO11 |
|   32 |  GPIO07 |
|   33 |  GPIO13 |

## 通过 GPIO 编号找到对应的 PORT

下载 [Jetson_Orin_NX_and_Orin_Nano_series_Pinmux_Config_Template.xlsm](https://developer.nvidia.com/downloads/jetson-orin-nx-and-orin-nano-series-pinmux-config-template)表格，查找 PORT。
![](/assets/img/post/常用技巧/pinmux.png)

由此可得:

| 引脚 | gpio 编号 | PORT  |
|------|----------|-------|
|   7  |  GPIO09  | PAC.06|
|   15 |  GPIO12  | PN.01 |
|   29 |  GPIO01  | PQ.05 |
|   31 |  GPIO11  | PQ.06 |
|   32 |  GPIO07  | PG.06 |
|   33 |  GPIO13  | PH.00 |

## 通过 PORT 找到 Line

执行 gpioinfo 命令

```bash
gpioinfo 

gpiochip0 - 164 lines:
        line   0:      "PA.00" "regulator-vdd-3v3-sd" output active-high [used]
        line   1:      "PA.01"       unused   input  active-high
        line   2:      "PA.02"       unused   input  active-high
        line   3:      "PA.03"       unused   input  active-high
        line   4:      "PA.04"       unused   input  active-high
        line   5:      "PA.05"       unused   input  active-high
        line   6:      "PA.06"       unused   input  active-high
        line   7:      "PA.07"       unused   input  active-high
        line   8:      "PB.00"       unused   input  active-high
        line   9:      "PC.00"       unused   input  active-high
        line  10:      "PC.01"       unused   input  active-high
        line  11:      "PC.02"       unused   input  active-high
        line  12:      "PC.03"       unused   input  active-high
        line  13:      "PC.04"       unused   input  active-high
        line  14:      "PC.05"       unused   input  active-high
        line  15:      "PC.06"       unused   input  active-high
        line  16:      "PC.07"       unused   input  active-high
        line  17:      "PD.00"       unused   input  active-high
        line  18:      "PD.01"       unused   input  active-high
        line  19:      "PD.02"       unused   input  active-high
        line  20:      "PD.03"       unused   input  active-high
        line  21:      "PE.00"       unused   input  active-high
        line  22:      "PE.01"       unused   input  active-high
        line  23:      "PE.02"       unused   input  active-high
        line  24:      "PE.03"       unused   input  active-high
        line  25:      "PE.04"       unused   input  active-high
        line  26:      "PE.05"       unused   input  active-high
        line  27:      "PE.06"       unused   input  active-high
        line  28:      "PE.07"       unused   input  active-high
        line  29:      "PF.00"       unused   input  active-high
        line  30:      "PF.01"       unused   input  active-high
        line  31:      "PF.02"       unused   input  active-high
        line  32:      "PF.03"       unused   input  active-high
        line  33:      "PF.04"       unused   input  active-high
        line  34:      "PF.05"       unused   input  active-high
        line  35:      "PG.00" "Force Recovery" input active-low [used]
        line  36:      "PG.01"       unused   input  active-high
        line  37:      "PG.02"       unused   input  active-high
        line  38:      "PG.03"       unused   input  active-high
        line  39:      "PG.04"       unused   input  active-high
        line  40:      "PG.05"       unused   input  active-high
        line  41:      "PG.06"       unused   input  active-high
        line  42:      "PG.07"       unused   input  active-high
        line  43:      "PH.00"       unused   input  active-high
        line  44:      "PH.01"       unused   input  active-high
        line  45:      "PH.02"       unused   input  active-high
        line  46:      "PH.03" "camera-control-output-low" output active-high [used]
        line  47:      "PH.04"       unused   input  active-high
        line  48:      "PH.05"       unused   input  active-high
        line  49:      "PH.06"       unused  output  active-high
        line  50:      "PH.07"       unused   input  active-high
        line  51:      "PI.00"       unused   input  active-high
        line  52:      "PI.01"       unused   input  active-high
        line  53:      "PI.02"       unused   input  active-high
        line  54:      "PI.03"       unused   input  active-high
        line  55:      "PI.04"       unused   input  active-high
        line  56:      "PI.05"       kernel   input  active-high [used]
        line  57:      "PI.06"       unused   input  active-high
        line  58:      "PJ.00"       unused   input  active-high
        line  59:      "PJ.01"       unused   input  active-high
        line  60:      "PJ.02"       unused   input  active-high
        line  61:      "PJ.03"       unused   input  active-high
        line  62:      "PJ.04"       unused   input  active-high
        line  63:      "PJ.05"       unused   input  active-high
        line  64:      "PK.00"       unused   input  active-high
        line  65:      "PK.01"       unused   input  active-high
        line  66:      "PK.02"       unused   input  active-high
        line  67:      "PK.03"       unused   input  active-high
        line  68:      "PK.04"       unused  output  active-high
        line  69:      "PK.05"       unused  output  active-high
        line  70:      "PK.06"       unused   input  active-high
        line  71:      "PK.07"       unused   input  active-high
        line  72:      "PL.00"       unused   input  active-high
        line  73:      "PL.01"       unused   input  active-high
        line  74:      "PL.02"       unused   input  active-high
        line  75:      "PL.03"       unused   input  active-high
        line  76:      "PM.00"       kernel   input  active-high [used]
        line  77:      "PM.01"       unused   input  active-high
        line  78:      "PM.02"       unused   input  active-high
        line  79:      "PM.03"       unused   input  active-high
        line  80:      "PM.04"       unused   input  active-high
        line  81:      "PM.05"       unused   input  active-high
        line  82:      "PM.06"       unused   input  active-high
        line  83:      "PM.07"       unused   input  active-high
        line  84:      "PN.00"       unused   input  active-high
        line  85:      "PN.01"       unused   input  active-high
        line  86:      "PN.02"       unused   input  active-high
        line  87:      "PN.03"       unused   input  active-high
        line  88:      "PN.04"       unused   input  active-high
        line  89:      "PN.05"       unused   input  active-high
        line  90:      "PN.06"       unused   input  active-high
        line  91:      "PN.07"       unused   input  active-high
        line  92:      "PP.00"       unused   input  active-high
        line  93:      "PP.01"       unused   input  active-high
        line  94:      "PP.02"       unused   input  active-high
        line  95:      "PP.03"       unused   input  active-high
        line  96:      "PP.04"       unused   input  active-high
        line  97:      "PP.05"       unused   input  active-high
        line  98:      "PP.06"       unused   input  active-high
        line  99:      "PP.07"       unused   input  active-high
        line 100:      "PQ.00"       unused   input  active-high
        line 101:      "PQ.01"       unused   input  active-high
        line 102:      "PQ.02"       unused   input  active-high
        line 103:      "PQ.03"       unused  output  active-high
        line 104:      "PQ.04"       unused   input  active-high
        line 105:      "PQ.05"       unused   input  active-high
        line 106:      "PQ.06"       unused   input  active-high
        line 107:      "PQ.07"       unused   input  active-high
        line 108:      "PR.00"       unused   input  active-high
        line 109:      "PR.01"       unused   input  active-high
        line 110:      "PR.02"       unused   input  active-high
        line 111:      "PR.03"       unused   input  active-high
        line 112:      "PR.04"       unused   input  active-high
        line 113:      "PR.05"       unused   input  active-high
        line 114:      "PX.00"       kernel   input  active-high [used]
        line 115:      "PX.01"       kernel   input  active-high [used]
        line 116:      "PX.02"       unused   input  active-high
        line 117:      "PX.03"       unused   input  active-high
        line 118:      "PX.04"       unused   input  active-high
        line 119:      "PX.05"       unused   input  active-high
        line 120:      "PX.06"       unused   input  active-high
        line 121:      "PX.07"       unused   input  active-high
        line 122:      "PY.00"       unused   input  active-high
        line 123:      "PY.01"       unused   input  active-high
        line 124:      "PY.02"       unused   input  active-high
        line 125:      "PY.03"       unused   input  active-high
        line 126:      "PY.04"       unused   input  active-high
        line 127:      "PY.05"       unused   input  active-high
        line 128:      "PY.06"       unused   input  active-high
        line 129:      "PY.07"       unused   input  active-high
        line 130:      "PZ.00"       unused   input  active-high
        line 131:      "PZ.01"  "interrupt"   input  active-high [used]
        line 132:      "PZ.02"       unused   input  active-high
        line 133:      "PZ.03"       unused   input  active-high
        line 134:      "PZ.04"       unused   input  active-high
        line 135:      "PZ.05"       unused   input  active-high
        line 136:      "PZ.06"       unused   input  active-high
        line 137:      "PZ.07"       unused   input  active-high
        line 138:     "PAC.00"       unused  output  active-high
        line 139:     "PAC.01"       unused   input  active-high
        line 140:     "PAC.02"       unused   input  active-high
        line 141:     "PAC.03"       unused   input  active-high
        line 142:     "PAC.04"       unused   input  active-high
        line 143:     "PAC.05"       unused   input  active-high
        line 144:     "PAC.06"       unused   input  active-high
        line 145:     "PAC.07"       unused   input  active-high
        line 146:     "PAD.00"       unused   input  active-high
        line 147:     "PAD.01"       unused   input  active-high
        line 148:     "PAD.02"       unused   input  active-high
        line 149:     "PAD.03"       unused   input  active-high
        line 150:     "PAE.00"       unused   input  active-high
        line 151:     "PAE.01"       unused   input  active-high
        line 152:     "PAF.00"       unused   input  active-high
        line 153:     "PAF.01"       unused   input  active-high
        line 154:     "PAF.02"       unused   input  active-high
        line 155:     "PAF.03"       unused   input  active-high
        line 156:     "PAG.00"       unused   input  active-high
        line 157:     "PAG.01"       unused   input  active-high
        line 158:     "PAG.02"       unused   input  active-high
        line 159:     "PAG.03"       unused   input  active-high
        line 160:     "PAG.04"       unused   input  active-high
        line 161:     "PAG.05"       unused   input  active-high
        line 162:     "PAG.06"       unused   input  active-high
        line 163:     "PAG.07"       unused   input  active-high
gpiochip1 - 32 lines:
        line   0:     "PAA.00"       unused   input  active-high
        line   1:     "PAA.01"       unused   input  active-high
        line   2:     "PAA.02"       unused   input  active-high
        line   3:     "PAA.03"       unused   input  active-high
        line   4:     "PAA.04"       unused  output  active-high
        line   5:     "PAA.05" "regulator-vdd-3v3-pcie" output active-high [used]
        line   6:     "PAA.06"       unused   input  active-high
        line   7:     "PAA.07"       unused   input  active-high
        line   8:     "PBB.00"       unused   input  active-high
        line   9:     "PBB.01"       unused   input  active-high
        line  10:     "PBB.02"       unused   input  active-high
        line  11:     "PBB.03"       unused  output  active-high
        line  12:     "PCC.00"       unused  output  active-high
        line  13:     "PCC.01"       unused  output  active-high
        line  14:     "PCC.02"       unused  output  active-high
        line  15:     "PCC.03"        "mux"  output  active-high [used]
        line  16:     "PCC.04"       unused   input  active-high
        line  17:     "PCC.05"       unused   input  active-high
        line  18:     "PCC.06"       unused   input  active-high
        line  19:     "PCC.07"       unused   input  active-high
        line  20:     "PDD.00"       unused   input  active-high
        line  21:     "PDD.01"       unused   input  active-high
        line  22:     "PDD.02"       unused   input  active-high
        line  23:     "PEE.00"       unused   input  active-high
        line  24:     "PEE.01"       unused   input  active-high
        line  25:     "PEE.02"       unused   input  active-high
        line  26:     "PEE.03"       unused   input  active-high
        line  27:     "PEE.04"      "Power"   input   active-low [used]
        line  28:     "PEE.05"       unused   input  active-high
        line  29:     "PEE.06"       unused   input  active-high
        line  30:     "PEE.07"       unused   input  active-high
        line  31:     "PGG.00"       unused   input  active-high
```

由此可得:

| 引脚 | gpio 编号 | PORT  |    CHIP   | LINE |
|------|----------|-------|-----------|------|
|   7  |  GPIO09  | PAC.06| gpiochip0 |  144 |
|   15 |  GPIO12  | PN.01 | gpiochip0 |  85  |
|   29 |  GPIO01  | PQ.05 | gpiochip0 |  105 |
|   31 |  GPIO11  | PQ.06 | gpiochip0 |  106 |
|   32 |  GPIO07  | PG.06 | gpiochip0 |  41  |
|   33 |  GPIO13  | PH.00 | gpiochip0 |  43  |

## 编写 c++ 代码操作 GPIO

+ 安装 lbgpdio 库

```bash
sudo apt install libgpiod-dev
```
+ 参考文档
[传送门](https://libgpiod.readthedocs.io/en/latest/index.html)

+ 参考代码

```c++
#include <gpiod.h> // sudo apt install libgpiod-dev
#include <cstdint>
#include <string>
#include <tuple>
#include <string>
#include <map>

namespace GpioHelper {
using Pin = uint32_t;
using Chip = std::string;
using Line = uint32_t;
const uint32_t DirectionOut = 0;
const uint32_t DirectionIn = 1;
auto get_pin_map() -> std::map<Pin, std::tuple<Chip, Line>> {
    return {
        {7,  {"gpiochip0", 144}}, // GPIO09 PAC.06
        {15, {"gpiochip0", 85}},  // GPIO12 PN.01
        {29, {"gpiochip0", 105}}, // GPIO01 PQ.05
        {31, {"gpiochip0", 106}}, // GPIO11 PQ.06
        {32, {"gpiochip0", 41}},  // GPIO07 PG.06
        {33, {"gpiochip0", 43}}   // GPIO13 PH.00
    };
}
} // namespace GpioHelper

int main() {
    // 以开发板引脚 7 为例：
    GpioHelper::Pin pin(7);
    struct gpiod_chip *pchip;
    struct gpiod_line *pline;
    // 打开 chip
    auto map = get_pin_map();
    auto chip = std::get<0>(map[pin]);
    auto line = std::get<1>(map[pin]);
    pchip = gpiod_chip_open_by_name(chip.c_str());
    if (!pchip) {
        std::cout << "Failed to open GPIO chip\n";
        return -1;
    }
    // 获取 GPIO Line
    pline = gpiod_chip_get_line(pchip, line_num);
    if (!pline) {
        gpiod_chip_close(pchip);
        std::cout << "Failed to get GPIO line\n";
        return -1;
    }
    // 设置 GPIO 方向
    auto ret = gpiod_line_request_output(pline, "gpio", GpioHelper::DirectionOut);
    if (ret < 0) {
        gpiod_line_release(pline);
        gpiod_chip_close(pchip);
        std::cout << "Failed to set GPIO direction\n";
        return -1;
    }
    // 设置高电平
    gpiod_line_set_value(pline, 1);
    // 读取电平
    int val = gpiod_line_get_value(pline);
    std::cout << val << "\n";
    // 设置低电平
    gpiod_line_set_value(pline, 0);
    // 读取电平
    val = gpiod_line_get_value(pline);
    std::cout << val << "\n";
    // 释放资源
    if (pline) {
        gpiod_line_release(pline);
    }
    if (pchip) {
        gpiod_chip_close(pchip);
    }
    return 0;
}

```