---
title: Android下配置以太网IP地址
date: 2015-05-19 18:55:35
tags:
categories: 
- Android
---

需要在命令行下配置，并取得root权限。

# 1 IP ADDR
## 1.1 DHCP

使用以下命令配置`eth0`使用`dhcp`：
```
ifconfig eth0 up
netcfg eth0 dhcp
```

## 1.2 STATIC IP

使用以下命令设置一个静态IP和一个子网掩码：
```
ifconfig eth0 [YOUR_IP_ADDRESS] netmask [NETMASK] up
```

然后使用以下命令配置网关：
```
ip route add default via [GATEWAY_IP] dev eth0
```

# 2 DNS

DNS使用通过Android环境属性配置的，而不是resolv.conf文件。配置命令是：
```
setprop net.dns1 [DNS_IP_ADDRESS1]
setprop net.dns2 [DNS_IP_ADDRESS2]
```