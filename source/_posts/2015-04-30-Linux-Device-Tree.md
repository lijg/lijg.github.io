---
title: Linux Device Tree
date: 2015-04-30 21:12:51
tags:
- Device Tree
- 设备树
categories:
- Linux
---

#0 什么是 Device Tree ?

* 从Power.org标准组织的嵌入式Power架构平台需求(ePART)引入
  * ePAPR 定一个了一个用来描述系统硬件信息的文件。启动时，启动程序将device tree加载到客户端的内存中，并将device tree在内存中地址作为指针传递给客户端
  * Device tree采用树形结构组织信息，包含一些用来表述系统中的物理设备的节点
  * 一个兼容ePAPR标准的device tree文件应该包含系统中所有无法被客户端动态检测到的设备信息

#1 介绍: 使用Device Tree启动

##1.1 之前

* 内核包含所有的硬件描述
* BootLoader加载整个内核镜像然后跳转到内核执行
  * 通常是uImage或zImage
* BootLoadr通过r2寄存器将一些额外的信息，称作ATAGS，传递给内核
  * 通常包含内存大小、内存地址、内核启动参数等等
* BootLoader通过r1寄存器将机器类型码传递给内核
* U-Boot中通过以下命令启动内核： bootm <内核加载到内存中的地址>
* 环境变量: bootm.image

![启动方式1](/images/2015-04-30/5f2dc406420db327698a1e3c8fff4.png)

##1.2 之后

* 内核中不再包含硬件描述信息, 硬件描述信息被包含在单独的镜像中： dtb
* BootLoader在启动时需要同时加载两个镜像: 内核 及 dtb文件
* 内核镜像同样是uImage 或 zImage
* dtb文件存放在 <kernel>/arch/arm/boot/dts, 每个板子一个文件
* BootLoader通过r2寄存器将dtb文件在内存中的地址传递给内核. dtb文件需要根据内存信息、内核命令行或其他信息作修改
* 不再需要机器码
* U-Boot命令行: bootm <内核被加载到内存中的地址> - <dtb被加载到内存中的地址>
* 环境变量: bootm.image, bootm.oftree

![启动方式2](/images/2015-04-30/a3a8187d30eb4e5085cf12cccacb6.png)

##1.3 兼容模式

* 某些BootLoader不支持Device Tree或者设备上使用的版本太老而不支持Device Tree
* 为了方便转换， 添加了一个兼容机制， 通过内核配置项： CONFIG_ARM_APPENDED_DTB.
  * 如果配置了该选项，内核会默认在紧接着内核镜像的地方寻找dtb文件镜像
  * Makefile中没有包含将内核和dtb打包成一个镜像的功能，需要手动实现
```
cat arch/arm/boot/zImage arch/arm/boot/dts/myboard.dtb > my-zImage
mkimage ... -d my-zImage my-uImage
```
* 此外, 如果配置了CONFIG_ARM_ATAG_DTB_COMPAT， 内核会使用BootLoader传递来的ATAGS信息, 覆盖Device Tree中的内容


#2 Device Tree的语法及编译

##2.1 基本语法

![dts语法](/images/2015-04-30/c57062f6ff3456416f7e1143822bb.png)

##2.2 编译

* 对于ARM平台，所有Device Tree的源文件(DTS)存放在arch/arm/boot/dts
    * .dts文件是板级描述
    * .dtsi是头文件，包含SoC级描述
* Device Tree Compiler(DTC)编译DTS，将其转换成二进制镜像
    * DTC的源文件放在scripts/dtc
    * 编译生成Device Tree Blob(DTB)文件。BootLoader在启动时加载DTB文件并将其传递给内核
* arch/arm/boot/dts/Makefile文件中指定了在编译时编译哪些DTB文件
```
dtb-$(CONFIG_ARCH_MVEBU) += armada-370-db.dtb \
armada-370-mirabox.dtb \
...
```

#3 Device Tree的示例片段

##3.1 Device Tree端
![dts示例](/images/2015-04-30/134be9b26c146e9e0252aaa971fdf.png)

##3.2 驱动端

![dts驱动](/images/2015-04-30/85380c03034a17323180d7757545b.png)


> 待续...