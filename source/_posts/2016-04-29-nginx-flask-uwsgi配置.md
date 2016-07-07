---
title: nginx+flask+uwsgi配置
date: 2016-04-29 18:03:11
tags:
- Nginx
- Flask
- uwsgi
categories:
- Linux
---

# 1 Nginx

```
yum install nginx
```

修改`/etc/nginx/flask.conf`

```
server {
    listion 80;
    server_name domain_name;
    localtion / {
        include uwsgi_params;
        uwsgi_pass 127.0.0.1:8889;
    }
}
```

启动`nginx`服务
```
systemctl enable nginx.service
systemctl start nginx.service
```

# 2 uWSGI

```
yum install uwsgi uwsgi-plugin-python
```

编辑`/etc/uwsgi.ini`，
```
[uwsgi]
uid = uwsgi
gid = uwsgi
pidfile = /run/uwsgi/uwsgi.pid
emperor = /etc/uwsgi.d
stats = /run/uwsgi/stats.sock
emperor-tyrant = true
cap = setgid,setuid
```

编辑`/etc/uwsgi.d/flask.ini`
```
[uwsgi]
uid = nginx
gid = nginx

disable-logging = true

socket = 127.0.0.1:8888

single-interpreter = true
master = true
plugin = python

app = hello
module = hello
callable = app

virtualenv = /var/www/flask-ve
pythonpath = /var/www/flask
chdir = /var/www/flask
```

启动`uWSGI`服务
```
systemctl enable uwsgi.service
systemctl start uwsgi.service
```

# 3 Flask

创建`virtualenv`
```
cd /var/www
virtualenv flask-ve
source flask-ve/bin/activate
pip install flask
```

创建`app`
```
cd /var/www
mkdir flask
cd flask
echo >> hello.py << EOF
#!/usr/bin/python
# -*- coding: utf-8 -*-

from flask import Flask

app = Flask(__name__)

@app.route('/'):
    return 'Hello World!'
```

完。