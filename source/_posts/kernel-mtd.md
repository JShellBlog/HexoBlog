---
title: kernel_mtd
date: 2019-06-25 11:43:30
tags:
    - Driver
    - kernel
categories:
    - Driver
---

<!--more-->

## 1. Flash 大致分类  
- Nor Flash (intel 开发)
- Nand Flash (Toshiba 开发)
- OneNand Flash(Samsung 开发)

NAND Flash在容量、功耗、使用寿命、写速度快、芯片面积小、单元密度高、擦除速度快、成本低等方面的优势使其成为高数据存储密度的理想解决方案。
NOR Flash的传输效率很高，但写入和擦除速度较低；

OneNAND结合了NAND存储密度高、写入速度快和NOR读取速度快的优点，整体性能完全超越常规的NAND和NOR。
__OneNAND采用NAND逻辑结构的存储内核和NOR的控制接口，并直接在系统内整合一定容量SRAM静态随即存储器作为高速缓冲区。__

当OneNAND执行程序时，代码必须从OneNAND存储核心载入到SRAM，然后在SRAM上执行。由于SRAM的速度优势，数据载入动作几乎可以在瞬间完成，用户感觉不到迟滞现象，加上SRAM被直接封装在OneNAND芯片内部，外界看起来就好像是OneNAND也具备程序的本地执行功能。

 Flash | 读 | 写 | 擦除 | 坏块 | XIP(eXecute In Place) | 容量 | 寿命 | 成本 
:-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: 
Nor Flash | 100MBps | 0.5MBps | 0.3MBps | - | YES | 常见<= 32M | 10几万 | 较高
Nand Flash | 15MBps | 7MBps | 64MBps |有 | - | 大容量 | 100万 | 低
OneNand Flash | 100MBps | 10MBps | 64MBps | 有 | YES | 较大容量 |  100万 | 较低



## 2. 延伸扩展  
![](https://gss2.bdstatic.com/-fo3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike92%2C5%2C5%2C92%2C30/sign=47c8a31259ee3d6d36cb8f99227f0647/e1fe9925bc315c60225ddb5a8db1cb13485477be.jpg)

### 2.1. SSD [SATA](https://baike.baidu.com/item/SATA) 接口 （串行口）  
![](https://gss3.bdstatic.com/-Po3dSag_xI4khGkpoWK1HF6hhy/baike/w%3D268/sign=226f2bbbb13533faf5b6942890d2fdca/d53f8794a4c27d1ee9ac99711bd5ad6edcc438f8.jpg)

### 2.2. SSD [NVME](https://baike.baidu.com/item/NVMe/20293531) 接口 （PCIe 口）
![](https://gss0.bdstatic.com/94o3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike150%2C5%2C5%2C150%2C50/sign=61b517595866d0166a14967af642bf62/cefc1e178a82b90112fe57a7798da9773812efd7.jpg)

[SSD技术扫盲之：什么是NVMe？ NVMe SSD有什么特点？](http://www.chinastor.com/baike/ssd/04103A942017.html)

参看资源：  
[OneNAND](https://blog.csdn.net/programxiao/article/details/6214607)

[三星OneNAND技术](https://www.chinaflashmarket.com/Instructor/102570)