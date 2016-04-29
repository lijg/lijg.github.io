---
title: 在ext4中保存uboot环境变量及uImage
date: 2015-05-08 22:19:57
tags: 
- uboot
categories: 
- Linux
---

在上一篇文章[《制作嵌入式Linux系统SD卡》](/post/zhi-zuo-qian-ru-shi-linuxxi-tong-sdqia)中，uImage、DTB及uboot的env都是通过dd烧录到sd卡上的，而不是在ext4分区中。这样每次升级uImage或dtb的时候，都需要拔下sd卡再烧录，稍显麻烦。如果uImage和dtb等都可以放在ext4分区中，就可以直接在启动后修改文件来升级内核或设备树了。

查看了一下uboot的源码，发现uboot是支持ext4文件系统的，而imx6的默认配置没有打开此选项，在mx6_sabresd.h中添加`CONFIG_FS_EXT4`选项，可以打开ext4文件系统支持。

之后在uboot命令行多了两个命令可以执行:
```
ext2ls - list files in a directory (default /)                                  
                                                                                
Usage:                                                                          
ext2ls <interface> <dev[:part]> [directory]                                     
    - list files from 'dev' on 'interface' in a 'directory'
```

```
ext2load - load binary file from a Ext2 filesystem                              
                                                                                
Usage:                                                                          
ext2load <interface> <dev[:part]> [addr] [filename] [bytes]                     
    - load binary file 'filename' from 'dev' on 'interface'                     
      to address 'addr' from ext2 filesystem. 
```

通过以下命令查看ext4分区的内容
```
=> ext2ls mmc 0:1 /                                                             
<DIR>       4096 .                                                              
<DIR>       4096 ..                                                             
<DIR>      16384 lost+found                                                     
<DIR>       4096 bin                                                            
<DIR>       4096 dev                                                            
<DIR>       4096 etc                                                            
<DIR>       4096 home                                                           
<SYM>         11 init                                                           
<DIR>       4096 lib                                                            
<DIR>       4096 mnt                                                            
<DIR>       4096 opt                                                            
<DIR>       4096 proc                                                           
<DIR>       4096 root                                                           
<DIR>       4096 sbin                                                           
<DIR>       4096 sys                                                            
<DIR>       4096 tmp                                                            
<DIR>       4096 usr                                                            
<DIR>       4096 var                                                            
<DIR>       4096 boot
```

通过以下命令将/boot/uImage和/boot/imx6q-sabresd-ldo.dtb加载到内存
```
ext2load mmc 0:1 ${loadaddr} /boot/uImage_4.4.3
ext2load mmc 0:1 ${dtbloadaddr} /boot/imx6q-sabresd-ldo.dtb
```
之后就跟正常启动一样了。

其实，不止是uImage和dtb，设置uboot的env都可以在ext4分区中设置。比如，在ext4中创建`/boot/uENV.txt`文件，添加以下内容：
```
loadaddr=0x12000000
dtbloadaddr=0x18000000
bootargs=console=ttymxc0,115200 init=/init rw root=/dev/mmcblk1p1
bootcmd=ext2load mmc 0:1 ${loadaddr} /boot/uImage_4.4.3;ext2load mmc 0:1 ${dtbloadaddr} /boot/imx6q-sabresd-ldo.dtb;bootm ${loadaddr} - ${dtbloadaddr}

```
保存后，在uboot中设置如下：
```
setenv bootcmd 'ext2load mmc 0:1 0x12000000 /boot/uENV.txt;env import -t 0x12000000;boot'
```
在启动时，uboot会自动使用uENV.txt中的内容覆盖启动变量，从ext4分区加载uImage和dtb，然后启动。