---
title: kernel_ubi
date: 2019-07-05 16:55:03
tags: 
    - mtd
    - ubi
categories: 
    - driver
---

## 1. 背景

Flash 设备存在如下缺点
- 存在坏块  
- 使用寿命较短  
- 存储介质不稳定(bitflip)  
- 读写速度慢  
- 只能通过擦除将0改成1  
- 最小读写单位为page or sub-page  

<!--more-->

Kernel 引入UBI (Unsorted Block Images)来解决这些问题。UBI 本身就是针对RAW Flash的一个卷管理系统，并且提供基于磨损均衡的逻辑到物理的映射。它类似于LVM（Logical Volume Manager)

![](https://raw.githubusercontent.com/JShell07/jshell07.github.io/master/images/kernel_ubi/mtd%20partition%20vs%20ubi%20volume.png)

因此UBI 具有如下特点：
- UBI provides volumes which may be dynamically created, removed, or re-sized
- UBI implements wear-leveling across whole flash device 
- UBI transparently handles bad physical eraseblocks;
- UBI minimizes chances to lose data by means of scrubbing. （针对Nand Flash 的bit flip现象，将有bit-flips的数据块移动到好的数据块上）

<!--more-->

## 2. 框架
在kernel MTD 模块看来， UBI 子系统的框架如下图：
![](https://raw.githubusercontent.com/JShell07/jshell07.github.io/master/images/kernel_ubi/ubi%20structure.png)

UBI 子模块的文件组织结构如下图：
![](https://raw.githubusercontent.com/JShell07/jshell07.github.io/master/images/kernel_ubi/ubi%20software%20structure.png)

文件名 | 说明
:-: | :-
cdev.c | 字符设备节点访问操作, attach/detach MTD, volume add/remove/rename/resize
build.c | /dev/ubictrl, /sys 等节点注册
attach.c | attach MTD sub-system
vmt.c | volume 逻辑操作，增删， 重命名， 重新设定size 
vtbl.c | vmt.c 的下层，实际操作读写
fastmap.c | 支持快速扫描MTD 设备
upd.c | update volume, 考虑到突然掉电等引起的更新错误
eba.c | Erase Block Association sub-system 子系统， 逻辑映射
wl.c | wear-leveling sub-system 磨损均衡子系统
io.c | <font color=red>与MTD 设备I/O 交互, R/W data, VID(volume ID)/EC(Erase Counter) header </font>

## 3. 代码分析
### 3.1. 数据结构
#### 3.1.1. UBI Headers
UBI 包含两个被CRC32 保护的64 bytes header在每一个非坏块的开始：
- erase counter(EC) header
- volume identifier(VID) header

因此，LEB < PEB 就是因为存储了UBI headers.

参见 ==drivers/mtd/ubi/ubi-media.h==
```c
struct ubi_ec_hdr {
	__be32  magic;
	__u8    version;
	__u8    padding1[3];
	__be64  ec; /* Warning: the current limit is 31-bit anyway! */
	__be32  vid_hdr_offset;
	__be32  data_offset;
	__be32  image_seq;
	__u8    padding2[32];
	__be32  hdr_crc;
} __packed;
```
每一次Erase 都会将EC 值增加。在unclean reboot 发生或者数据被corrupted， EC 将会被写入attach MTD 设备扫描的EC average。

```c
struct ubi_vid_hdr {
	__be32  magic;
	__u8    version;
	__u8    vol_type;
	__u8    copy_flag;
	__u8    compat;
	__be32  vol_id;
	__be32  lnum;
	__u8    padding1[4];
	__be32  data_size;
	__be32  used_ebs;
	__be32  data_pad;
	__be32  data_crc;
	__u8    padding2[4];
	__be64  sqnum;
	__u8    padding3[12];
	__be32  hdr_crc;
} __packed;
```

EC header 存储offset 为0， 而VID header 存储offset 为next min I/O Unit, sub-page or page.
- NOR Flash, min I/O 为 1 byte, VID header offset 为64
- Nand Flash, sub-page or page

![](https://raw.githubusercontent.com/JShell07/jshell07.github.io/master/images/kernel_ubi/UBI%20headers.png)

#### 3.1.2. volume table 
数据是存储在flash 设备上， 它包含了如下meta-data:  
- 卷名  
- 保留物理擦除快的数量  
- 类型(static or dynamica) 
- crc 校验  
- update marker（用于标记更新volume name or size）  

```c
struct ubi_vtbl_record {
	__be32  reserved_pebs;
	__be32  alignment;
	__be32  data_pad;
	__u8    vol_type;
	__u8    upd_marker;
	__be16  name_len;
	__u8    name[UBI_VOL_NAME_MAX+1];
	__u8    flags;
	__u8    padding[23];
	__be32  crc;
} __packed;
```

UBI 使用了两个逻辑擦除快(Logical EraseBlock)保存，LEB0 and LEB1. 他们两者相互拷贝，以此来保证突发事件例如掉电等异常情形。

UBI 需要维护三种table:
- volume table  
- eraseblock association(EBA) table  
- erase counter(EC) table  

volume table 是存储在Flash 上，它的修改只会发生在create, delete, re-size 时。

EBA table 是用于logical to phsical 映射关系。
EC table 包含每个PEB 的erase conter 值， UBI wear-leveling 将会使用此表格。

EBA, EC table 可以做到存储在flash 上，但是它需要journaling, journal replay, journal commit 等，在boot-loader 时保证简单，代码的size 是不太容易的。

因此， __EBA, EC table 默认在attach MTD 时，根据扫描的EC,VID header 信息在RAM 中构建.__



### 3.2. UBI Sub-system
#### 3.2.1. attach mtd

#### 3.2.2. eba, eraseblock association

__marking eraseblocks as bad__
判断依据：
- 写操作时失败，UBI 将数据搬移，并准备对该eraseblock 进行torturing(拷问，审查)。
- erase 操作失败EIO error， 直接将它标记为bad

torturing 主要是两方面
1. eraseblock, and read check all is 0xFF
2. write data, and read check

#### 3.2.3. wear-leveling

#### 3.2.4. fastmap
fastmap 利用存储在flash 上的fastmap volume 以此来加速attach。 
__fastmap 一般用于large flash， 例如4Gib Nand chips。__
是否启用fastmap, 需要考虑到fastmap 本身会占用一些PEB，并且fastmap pool full, volume layout change or detach时，fastmap都需要写入信息到保留的PEB 中。

fastmap volume中存储的信息有：
- erase value
- a list of all PEBs and their state  
- a list of all volumes and their EBA 
- a list of PEBs called fastmap pool   

fastmap 还有一个fastmap pool 概念， 它的size 大约为 5% of the total amount of PEBs。我们从fastmap pool 中取用PEBs，这样我们只需要关心fastmap pool 的PEBs， 而不用关心total PEBs, 减少了需要存储的信息量。

## 参看资料
[Linux UBI子系统设计初探](https://www.cnblogs.com/wahaha02/p/4814698.html)

[ubi 官方文档](http://www.linux-mtd.infradead.org/doc/ubi.html)

[ubi design pdf from MTD org](http://www.linux-mtd.infradead.org/doc/ubidesign/ubidesign.pdf)

[ubi ppt from MTD org](http://www.linux-mtd.infradead.org/doc/ubi.ppt)

[elinux/ubifs](https://elinux.org/UBIFS)