---
title: 在Centos7上安装ShaodowSocks服务
date: 2016-05-13 20:48:01
tags:
- ShadowSocks
- Centos
categories:
- Linux
---

工作中经常用到Google，用时ShadowSocks加ProxySwitch就再方便不过了，以下记录在VPS上安装ShadowSocks服务的过程。

# 1 安装ShadowSocks

软件只需要安装ShadowSocks，这是一个Python软件，可以用easy_install或pip进行安装
```
easy_install shadowsocks
或者
pip install shadowsocks
```

此时会在`/usr/bin`下安装`ssserver`，这就就是ShadowSocks的服务器端软件。

# 2 配置

ShadowSocks配置使用Json文件，以下是我的电性配置文件`/etc/shadowsocks.json`
```
{
        "server":"0.0.0.0",     # 本机IP
        "server_port":5689,     # 大于2048小于65536
        "local_address":"127.0.0.1",
        "local_port":1080,
        "password":"密码",      # 设置一个密码
        "timeout":300,
        "method":"rc4-md5",     # 加密方法
        "fast_open":true,
        "workers":1
}

```
# 3 自动启动

创建`/etc/systemd/system/shadowsocks-server.service`，内容如下：

```
[Unit]
Description=Shadowsocks Server
After=network.target

[Service]
Type=forking
PIDFile=/run/shadowsocks/server.pid
PermissionsStartOnly=true
ExecStartPre=/bin/mkdir -p /run/shadowsocks
ExecStartPre=/bin/chown root:root /run/shadowsocks
ExecStart=/usr/bin/ssserver --pid-file /var/run/shadowsocks/server.pid -c /etc/shadowsocks.json -d start
Restart=on-abort
User=root
Group=root
UMask=0027

[Install]
WantedBy=multi-user.target
```

启动服务并添加到自动启动：
```
systemctl start shadowsocks-server.service
systemctl enable shadowsocks-server.service
```

