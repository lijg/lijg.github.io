---
title: Git Subtree
date: 2015-08-14 09:57:49
tags:
- Git
categories:
- Linux
---

### 目的

> 合并子仓库到项目子目录中

### 使用方法

* 添加远程分支
```
git remote add -f 分支名(仓库别名) 远程仓库地址
```

* 放到子目录
```
git subtree add --prefix=子目录名 分支名(仓库别名) 分支 --squash
```

* 更新仓库
```
git fetch 分支名(仓库别名) 分支
git pull --prefix=子目录名 分支名(仓库别名) 分支 --squash
```

* 推送更新
```
git subtree push --prefix=子目录名 分支名(仓库别名) 分支
```

完。