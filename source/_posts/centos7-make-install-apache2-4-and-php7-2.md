---
title: centos7 make install apache2.4 and php7.2
date: 2021-02-26 01:23:20
tags:
---

## 安装

```bash
# 安装开发环境
$ yum groupinstall "Development Tools"
# 安装工具
$ yum install wget

```

## install apache-2.4.46


### install  apr and apr-util

https://apr.apache.org/download.cgi

```bash

$ wget https://mirrors.tuna.tsinghua.edu.cn/apache/apr/apr-1.7.0.tar.gz 
$ tar -zxvf apr-1.7.0.tar.gz 

```
#### install  apr and apr-util

```bash 
$ wget https://mirrors.tuna.tsinghua.edu.cn/apache/apr/apr-util-1.6.1.tar.gz
$ tar -zxvf apr-util-1.6.1.tar.gz

```
### install apache

https://httpd.apache.org/download.cgi#apache24

```Bash
$ su
$ mkdir /var/src 
$ cd /var/src
$ wget https://downloads.apache.org/httpd/httpd-2.4.46.tar.gz
$ tar -zxvf httpd-2.4.46.tar.gz
$ cd httpd-2.4.46
$ ./configure --prefix=/usr/local/apache \
--sysconfdir=/etc/httpd24 \
--enable-so \
--enable-ssl \
--enable-cgi \
--enable-rewrite \
--with-zlib \
--with-pcre \
--with-apr=/usr/local/apr \
--with-apr-util=/usr/local/apr-util/ \
--enable-modules=most \
--enable-mpms-shared=all \
--with-mpm=prefork

$ make && make install

```


### install php

https://www.php.net/releases/

#### install php5.6

```bash
$ wget https://www.php.net/distributions/php-5.6.40.tar.gz
$ tar -zxvf php-5.6.40.tar.gz
$ cd php-5.6.40

```

#### install php5.5.6 

```bash
$ wget https://www.php.net/distributions/php-5.5.6.tar.gz
```

#### install PHP5.4.44

https://www.php.net/distributions/php-5.4.44.tar.gz

