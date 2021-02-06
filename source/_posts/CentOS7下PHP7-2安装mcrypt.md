---
title: CentOS7下PHP7.2安装mcrypt
date: 2021-02-05 22:14:47
tags:
---

PHP7.2 的变更,MCrypt 扩展从内核移动到 PECL。 考虑到 mcrypt 库自 2007 年以来未见任何更新，不再建议使用。 代替品即可以用 OpenSSL 也可以用Sodium

以下是Centos7.5+PHP7.2安装mcrypt过程

```Bash
# 安裝依賴
$ yum install libmcrypt libmcrypt-devel mcrypt mhash
# 下載擴展 http://pecl.php.net/package/mcrypt
$ wget http://pecl.php.net/get/mcrypt-1.0.4.tgz
# 解壓
$tar zxfv mcrypt-1.0.4.tgz 
# 進目錄
$ cd mcrypt-1.0.4
# 看一下phpize在哪
$ whereis phpize
#執行phpize
$ /usr/bin/phpize
# 看一下php-config在哪
$ whereis php-config
#編譯安裝
$ ./configure --with-php-config=/usr/bin/php-config && make && make install
# 安裝成功會在這裏
$ ls /usr/lib64/php/modules/mcrypt.so
# 在php.ini裏加一行 extension=mcrypt.so 啟用
$ echo "extension=mcrypt.so" >> /etc/php.ini
# 查看是否安裝成功,如果成功會打印出 mcrypt
$ php -m | grep mcrypt
# 重啟httpd完成
$ systemctl restart httpd
```
