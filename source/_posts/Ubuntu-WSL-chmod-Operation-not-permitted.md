---
title: 'Ubuntu (WSL) chmod(): Operation not permitted'
date: 2022-04-29 14:43:33
tags: Ubuntu,WSL
---

Ubuntu20.04 (WSL) 在PHP中chmod时出现:
`chmod(): Operation not permitted `

或者中文

`chmod(): 此項操作並不被允許`

解决办法：
```
# 如果出现 umount: /mnt/c: target is busy.退出已打开的编辑器和目录浏览，并执行: cd 
$ cd
$ sudo umount /mnt/c
$ sudo mount -t drvfs C: /mnt/c -o metadata

```