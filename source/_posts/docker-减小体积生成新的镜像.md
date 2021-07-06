---
title: docker 减小体积生成新的镜像
date: 2021-02-27 14:46:52
tags: docker
categories: [Docker]
---

Docker commit 每次都会加大镜像体积

解决办法:

使用Docker import 导入系统文件,丢弃镜像历史引用.

同样适用于全新安装系统生成docker镜像。

```bash
# 查看容器id, {container_id}
$ docker ps
# 开启一个{container_id}的交互式终端
$ docker exec -it {container_id} bash
$ cd /
# 清理,同时可以删除和清理其它
$ yum clean all && history -cw
# 查看/目录文件大小
$ du -h --exclude=/proc --exclude=/sys /
# 压缩成base.tar 排除/proc和/sys
# 在使用--exclude=/proc时会出错，所以使用 >/dev/null 可以看到错误消息
$ tar --exclude=/proc --exclude=/sys --exclude=/base.tar -cvf base.tar / >/dev/null
# 退出容器
$ exit

```

```bash
# 复制容器{ontainer_id}中的/base.tar文件到当前目录
$ docker cp {container_id}:/base.tar .
# 使用base.tar 生成{image_name} 镜像
$ cat base.tar | docker import - {image_name}
# 查看生成的镜像 (1.28G 转换后501Mb )
$ docker image ls

```
