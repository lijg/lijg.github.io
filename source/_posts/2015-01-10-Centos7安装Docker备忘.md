---
title: Centos7安装Docker备忘
date: 2015-01-10 20:02:07
tags:
- Docker
categories:
- Linux
---


在Centos7上通过`yum install docker`后是不能直接使用docker的，还需要启动`docker`服务才可以。

通过以下两条命令启动`docker`服务并将其加入开机启动。

```
systemctl start docker.service
systemctl enable docker.service
```