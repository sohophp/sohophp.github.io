---
title: Linux 常用命令备用
date: 2021-01-31 01:14:10
tags: [Linux,Centos]
comments: true
---

## netstat

```bash
netstat -nltp
```
查看当前监听端口，例如在安装了apache(httpd)和mod_ssl后，测试https不可用。
使用netstat -nltp 查看有apache在监听80,但没有443.(重新安装mod_ssl后正常).

---

***持续更新....***