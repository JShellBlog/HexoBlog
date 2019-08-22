---
title: Linux_Flash
date: 2018-08-22 22:20:13
tags:
    - flash
    - kernel
categories:
    - drivers
---

## 1. Flash
闪存（Flash Memory）是非易失性长寿命存储器，是电子可擦除只读存储器（EEPROM）的变种，而闪存的大部分芯片需要块擦除。

![MOS管插图](https://raw.githubusercontent.com/JShell07/jshell07.github.io/master/images/Linux_Flash/MOS%E7%AE%A1.png)

<!--more-->

注：
>N型半导体即自由电子浓度远大于空穴浓度的杂质半导体。(对于锗、硅类半导体材料，掺杂Ⅴ族元素(磷、砷等)
P型半导体即空穴浓度远大于自由电子浓度的杂质半导体。在纯净的硅晶体中掺入三价元素（如硼）

常见的Flash有
- NOR Flash
- NAND Flash

| | NOR Flash | NAND Flash |
| :-: | :- | :- |
| 读速度 | 快 | 较快 |
| 单字节编程时间 | 快 | 慢 |
| 多字节编程时间 | 慢 | 快 |
| 寿命 | 十万次 | 百万次 |
| I/O 端口 | SRAM 接口，有足够地址引脚寻址 | 8个复用PIN 脚传送控制、地址、数据 |
| 功耗 | 高 | 低，需要额外RAM |
| 应用市场 | 1~16MB | 大容量 |
| 特点 | 程序可在芯片内运行 | 成本低，高存储密度，较快的写入和擦除速度 |

![Large Page Nand](https://raw.githubusercontent.com/JShell07/jshell07.github.io/master/images/Linux_Flash/Large_Page_Nand.png)

注：Spare Area 常常可以存储一些BBT信息。

Nand Flash 硬件特性：
- R/W 最小单位，Page
- 擦除最小单位，Block
- 擦除的含义是将整块都擦除为0xFF
- 写操作前，必须先擦除，然后再写

## 2. Linux MTD

![MTD Structure](https://raw.githubusercontent.com/JShell07/jshell07.github.io/master/images/Linux_Flash/Linux_MTD_Structure.png)

常见MTD 分区为：

![](https://raw.githubusercontent.com/JShell07/jshell07.github.io/master/images/Linux_Flash/MTD_Common_Partitions.png)

## 3. R/W Flow

`MTD Read Flow`
![MTD_Read_Flow](https://raw.githubusercontent.com/JShell07/jshell07.github.io/master/images/Linux_Flash/MTD_Read_Flow.png)

<br>

`MTD Read Flow`
![MTD_Write_Flow](https://raw.githubusercontent.com/JShell07/jshell07.github.io/master/images/Linux_Flash/MTD_Write_Flow.png)

## 4. Bad Block
Nand Flash 工艺不能保证Memory Cell在其生命周期中保持性能的可靠性，使用寿命一般100万次。

坏块一般分为：

- 固有坏块：生产中就有（芯片原厂将每个坏块第一个Page的spare area某个字节标记为非0xFF）
- 使用坏块：存储寿命（program/erase 错误，将block标记为坏块）

当然，也有伪坏块。芯片在才做过程中可能由于电压不稳定等因素导致Program、Erase错误，标记的坏块也可能是好的。解决办法是，retry 一次，然后查看状态。

`坏块的特性`

Page Program与Block Erase操作失败，会反映到Status Register。

`避免坏块策略`

**Wear-Leveling(损益均衡)技术**
![](https://raw.githubusercontent.com/JShell07/jshell07.github.io/master/images/Linux_Flash/Wear_Leveling.png)

**Buffer Cache**
类似于UBIFS 都会带有buffer cache，以及压缩，以此来尽量降低对I/O的操作。

`应对坏块`

为了保证数据正确性的R/W，我们使用BBT（Bad Block Table）。
![](https://raw.githubusercontent.com/JShell07/jshell07.github.io/master/images/Linux_Flash/BBT.png)




