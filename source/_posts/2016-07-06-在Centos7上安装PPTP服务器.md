---
title: 在Centos7上安装PPTP服务器
date: 2016-07-06 16:25:50
tags:
- PPTP
- Centos
categories:
- Linux
---

PPTP是做什么用的就不介绍了，不懂的可以百毒一下，下面介绍在VPS上安装PPTP服务器的步骤，系统环境是Centos 7。

# 1 检查VPS是否支持

不是所有的VPS都支持PPTP服务，所以需要检查一下，命令如下：
```
cat /dev/ppp
```

如果返回`cat: /dev/ppp: No such file or directory` 或者 `cat: /dev/ppp: No such device or address`，说明系统支持。

# 2 安装组件
只需要安装一个`pptpd`，命令如下：
```
yum install -y pptpd
```

# 3 配置文件

## 3.1　pptpd.conf

编辑`/etc/pptpd.conf`，在文件的最后，去掉下面字段前面的#，
```
localip 192.168.0.1
remoteip 192.168.0.234-238,192.168.0.245
```
保存退出。

## 3.2 options.pptpd

编辑`/etc/ppp/options.pptpd`，搜索`ms-dns`，改为下面的字段：
```
ms-dns 8.8.8.8
ms-dns 8.8.4.4
```

## 3.3 chap-secrets

编辑`/etc/ppp/chap-secrets`，添加一行，格式是：
```
用户名   pptpd    密码    *
```

如，添加用户`ljgabc`，密码`123456`
```
ljgabc   pptpd    123456    *
```

# 4 内核参数

编辑`/etc/sysctl.conf`，在末尾添加一行，
```
net.ipv4.ip_forward = 1
```
打开IP转发功能。

之后运行以下命令使之立即生效：
```
sysctl -p
```

# 5 设置防火墙规则
打开命令行，执行以下命令：
```
firewall-cmd --permanent --new-service=pptp
cat >/etc/firewalld/services/pptp.xml<<EOF
<?xml version="1.0" encoding="utf-8"?>
<service>
  <port protocol="tcp" port="1723"/>
</service>
EOF
firewall-cmd --permanent --zone=public --add-service=pptp
firewall-cmd --permanent --zone=public --add-masquerade
firewall-cmd --permanent --direct --add-rule ipv4 filter INPUT 0 -p gre -j ACCEPT
firewall-cmd --reload
```

# 6 启动服务
现在可以启动PPTP服务，应该可以连上了
```
systemctl start pptpd
systemctl enable pptpd.service
```

