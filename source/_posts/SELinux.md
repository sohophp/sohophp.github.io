---
title: SELinux
date: 2021-02-07 15:02:00
tags: [SELinux,Linux,CentOS,CentOS7]
categories: [Linux,CentOS]
---

### SELinux的策略与规则管理相关命令

#### getsebool

getsebool命令是用来查询SELinux策略内各项规则的布尔值。
SELinux的策略与规则管理相关命令：
seinfo命令、sesearch命令、getsebool命令、setsebool命令、semanage命令。

##### 选项

-a：列出目前系统上面的所有布尔值条款设置为开启或关闭值。

#####  实例

```Bash
# 查看httpd相关设定
$ getsebool -a | grep httpd

```

