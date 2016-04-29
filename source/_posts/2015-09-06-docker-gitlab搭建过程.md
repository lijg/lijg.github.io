---
title: docker-gitlab搭建过程
date: 2015-09-06 20:10:44
tags: 
- Docker
categories: 
- Linux
---

> 建议使用gogs替代gitlab

# 1 使用到的docker镜像

```
docker pull sameersbn/gitlab:7.14.1
ocker pull sameersbn/postgresql:9.4-3
ocker pull sameersbn/redis:latest
```

# 2 启动应用

## 2.1 运行postgresql数据库
```
docker run --name gitlab-postgresql -d \
    --env 'DB_NAME=gitlabhq_production' \
    --env 'DB_USER=gitlab' --env 'DB_PASS=password' \
    --volume /srv/docker/gitlab/postgresql:/var/lib/postgresql \
    sameersbn/postgresql:9.4-3
```

也支持使用外部的数据库，此步不需要。在第三步，启动gitlab时，需要指定环境变量
```
--env 'DB_TYPE=postgres' \
--env 'DB_HOST=192.168.1.100' \
--env 'DB_NAME=gitlabhq_production' \
--env 'DB_USER=gitlab' \
--env 'DB_PASS=password' \
```


## 2.2 运行redis环境
```
docker run --name gitlab-redis -d \
    --volume /srv/docker/gitlab/redis:/var/lib/redis \
    sameersbn/redis:latest
```

也支持使用外部的redis服务器，此步不需要。在第三步启动gitlab时，需要指定环境变量
```
--env 'REDIS_HOST=192.168.1.100' \
--env 'REDIS_PORT=6379' \
```

## 2.3 运行gitlab
```
docker run --name gitlab -d \
    --link gitlab-postgresql:postgresql --link gitlab-redis:redisio \
    --publish 10022:22 --publish 10080:80 \
    --env 'GITLAB_PORT=10080' --env 'GITLAB_SSH_PORT=10022' \
    --volume /srv/docker/gitlab/gitlab:/home/git/data \
sameersbn/gitlab:7.14.1
```

默认讲`gitlab`的`80`端口映射位主机的`10080`端口；
`22`端口映射为主机的`10022`端口。

稍等几分钟之后，gitlab就可以使用了。

# 3 使用

默认用户名、密码：
```
username: root
password: 5iveL!fe
```

# 4 邮件配置
`gitlab`默认使用`gmail`发送邮件，如果想要使用的话，需要在启动时指定`gmail`的用户名和密码
```
--env 'SMTP_USER=USER@gmail.com' \
--env 'SMTP_PASS=PASSWORD' \
```

可以通过更多的环境变量，修改smtp的配置
```
--env 'SMTP_HOST=mail.google.com' \
--env 'SMTP_PORT=587' \
--env 'SMTP_USER=USER@gmail.com' \
--env 'SMTP_PASS=PASSWORD' \
```

gitlab默认邮件地址是example@example.com，可以通过环境变量修改
```
--env 'GITLAB_EMAIL=example@example.com' \
--env 'GITLAB_EMAIL_DISPLAY_NAME=GitLab' \
--env 'GITLAB_EMAIL_REPLY_TO=noreply@example.com' \
```
