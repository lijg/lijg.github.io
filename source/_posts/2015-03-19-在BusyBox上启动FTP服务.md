---
title: 在BusyBox上启动FTP服务
date: 2015-03-19 18:10:06
tags: 
- busybox
categories: 
- Linux
---

BusyBox上自带ftp服务器——`ftpd`(Anonymous FTP Server)，没有身份验证功能，可以作为一个简易的传输文件的措施使用。

`ftpd`可以通过设定`inetd.conf`由`inetd`启动(在有链接时inetd程序会自动启用ftpd)，或者是搭配`tcpsvd`作为单独的守护进程使用。

使用方法:

#0 FTPD的参数
`ftpd`的使用方法如下
```
Usage: ftpd [-wvS] [-t N] [-T N] [DIR]

-w  允许上传
-v  打印错误信息
-S  错误信息写入SYSLOG
-t  多长时间无操作算作空闲(默认2分钟, 2 * 60)
-T  多长时间空闲后自动断开与客户端的连接(默认1小时，1 * 60 * 60)
DIR FTP根目录
```

#1. 使用tcpsvd

`tcpsvd`可以建立TCP Socket，并将其绑定到某个程序上，命令格式如下
```
Usage: tcpsvd [选项] IP PORT PROG [PROG ARGS]

IP: 要监听的IP地址
PORT:   要监听的端口
PROG:   要绑定的程序
PROG ARGS: 绑定应用的参数

选项:
    -l NAME,    本地主机名
    -u USER[:GRP],  绑定后切换到USER/GROUP
    -c N,   最大连接数
    -C N[:MSG]  同一个IP的最大连接数（MSG为超过时的响应信息）
    -v      打印详细输出
```

使用`tcpsvd`启动`ftpd`的命令如下:

```
tcpsvd 0 21 ftpd -w /root
```

#2. 使用inetd

`inetd`使用`/etc/inetd.conf`里管理服务

使用`inetd`启动`ftpd`,需要在`/etc/inetd.conf`中增加以下代码:
```
21 stream tcp nowait root ftpd ftpd -w /root
```
然后在`/etc/init.d/rc.local`中启动`inetd`服务。
```
/usr/sbin/inetd &
```