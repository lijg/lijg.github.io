---
title: Centos架设SVN服务器
date: 2016-01-04 09:43:34
tags: 
- Centos 
- SVN
categories: 
- Linux
---

## 1. 安装工具
```
yum install -y httpd svnversion mod_dav_svn php php-mysql mariadb-server

systemctl start httpd mariadb
systemctl enable httpd mariadb
```

## 2. 下载usvn
```
# 假设svn文件要放在 /home/svn下
mkdir /home/svn
cd /home/svn
wget https://codeload.github.com/usvn/usvn/tar.gz/1.0.7

tar -xvf usvn-1.0.7.tar.gz

chcon -R -t httpd_sys_content_t /home/svn
chcon -R -t httpd_sys_rw_content_t /home/svn

vi /etc/httpd/conf.d/usvn.conf
```
输入以下内容
```
Alias /usvn /home/svn/usvn-1.0.7/public
<Directory "/home/svn/usvn-1.0.7/public">
    Options +SymLinksIfOwnerMatch
    AllowOverride All
    Order allow,deny
    Allow from all
</Directory>
```

之后，重启httpd服务
```
systemctl restart httpd.service
```

## 3. 创建数据库
```
# 设置mysql密码，按提示操作
mysql_secure_install

mysql -u root -p

> create database usvn;
> grant all on usvn.* to usvn@localhost identified by 'usvn';
> flush privileges;
```

## 4. 安装usvn
```
访问http://ip/usvn/install.php
```
