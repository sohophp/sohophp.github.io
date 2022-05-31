---
title: WSL-Ubuntu20.04 安装 Apache2.4 + MySQL8.0 + PHP7.2 和 PHP8.1 ,composer,git 开发环境
date: 2022-04-30 11:55:58
tags: [WSL,Ubuntu,PHP,MySQL]
---

### 安装 Apache 2.4

```bash
$ sudo apt update
$ sudo apt upgrade
$ sudo apt install vim net-tools unzip curl wget
# 安装 Apache
$ sudo apt install apache2
# 修改 /etc/apache2/apache.conf

# <Directory /var/www/>
#        Options Indexes FollowSymLinks
#        AllowOverride None
#        Require all granted
#</Directory>
# 为:
# <Directory /var/www/>
#        Options FollowSymLinks
#        AllowOverride All
#        Require all granted
# </Directory>
$ sudo /etc/init.d/apache2 start

```

### 安装 MySQL-Server 8.0

```bash
# 安装 MySQL8.0
$ sudo apt install mysql-server
# 启动 MySQL-Server
$ sudo /etc/init.d/mysql start
# 初始化 MySQL 密码
$ sudo mysqld --initialize --user=mysql --console
# 查看密码
$ sudo cat /etc/mysql/debian.cnf
# 使用 [client]密码登录
$ mysql -udebian-sys-maint -p
# 修改root密码
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '新密码';
# 添加一个用户
mysql> CREATE USER '用户名'@'localhost' IDENTIFIED BY '密码';
# 设定密码
mysql> ALTER USER '用户名'@'localhost' IDENTIFIED WITH mysql_native_password BY '新密码';
# 授权
mysql> GRANT ALL PRIVILEGES ON *.* TO '用户名'@'localhost' WITH GRANT OPTION;
# 刷新权限
mysql> FLUSH PRIVILEGES;
# 查看用户表
mysql> select host,user,authentication_string from mysql.user;
# 退出
mysql> quit;
```

### 安装 PHP 7.2

```bash
# 安装 PHP 存储库
$ sudo apt install software-properties-common
# 1. If you are using php-gearman, you need to add ppa:ondrej/pkg-gearman
# 2. If you are using apache2, you are advised to add ppa:ondrej/apache2
# 3. If you are using nginx, you are advised to add ppa:ondrej/nginx-mainline   or ppa:ondrej/nginx
# 添加 PPA
$ sudo add-apt-repository ppa:ondrej/php
$ sudo apt update
$ sudo apt upgrade
$ sudo apt install \
libapache2-mod-php7.2 \
php7.2 \
php7.2-cli \
php7.2-common \
php7.2-curl \
php7.2-fpm \
php7.2-imagick \
php7.2-json \
php7.2-mbstring \
php7.2-mcrypt \
php7.2-sqlite3 \
php7.2-xml \
php7.2-zip \
php7.2-mysqli \
php7.2-gd

# 在Apache中启用php7.2-fpm
$ a2enmod proxy_fcgi setenvif
$ sudo a2enconf php7.2-fpm
$ sudo /etc/init.d/apache2 restart
```

### 安装 PHP8.1

```bash
# 安装 PHP8.1
$ sudo apt install libapache2-mod-php8.1 \
php8.1 \
php8.1-apcu \
php8.1-cli \
php8.1-common \
php8.1-curl \
php8.1-fpm \
php8.1-gd \
php8.1-imagick \
php8.1-mbstring \
php8.1-sqlite3 \
php8.1-xml \
php8.1-yaml \
php8.1-zip \
php8.1-mysql

# 启用 a2enconf php8.1-fpm,不是必要的
$ sudo a2enconf php8.1-fpm
# 修改 php8.1-fpm 的 Unix Domain Socket 方式为 TCP Socket方式,监听端口为 9001
# 编辑 /etc/php/8.1/fpm/pool.d/www.conf
# 注释掉 #listen = /run/php/php8.1-fpm.sock
# 添加一行 listen: 127.0.0.1:9001
$ sudo vim /etc/php/8.1/fpm/pool.d/www.conf

# 启动php8.1-fpm
$ sudo /etc/init.d/php8.1-fpm start
# 需要使用php8.1的.htaccess中加：
# <FilesMatch \.php$>
#     SetHandler "proxy:fcgi://127.0.0.1:9001"
# </FilesMatch>

# 切換 php-cli 版本
# 为了和CentOS的php81同名
# sudo ln -s /usr/bin/php8.1 /usr/bin/php81

# 切换到php7.2
$ sudo update-alternatives --set php /usr/bin/php7.2
# 或者
$ sudo ln -sf /usr/bin/php7.2 /etc/alternatives/php
# 切换到php8.1
$ sudo update-alternatives --set php /usr/bin/php8.1
# 或者
$ sudo ln -sf /usr/bin/php8.1 /etc/alternatives/php

```

### 测试是否正常

```bash
# 查看网络监听，
#  localhost:9001 =>PHP8.1
#  localhost:mysql =>MySQL8.0
# [::]http => Apache2.4
# /run/php/php7.2-fpm.sock => PHP7.2
# 已全部正常启动
$ netstat -a 

```

### 安装 composer

```bash
curl  https://getcomposer.org/installer -o composer-setup.php
php composer-setup.php
chmod +x composer.phar
sudo mv composer.phar /usr/local/bin/composer
```

### 安装 GIT

```base
sudo apt install git
```

### 安装 npm

```bash
sudo apt install npm
```

### 开启 SSL

```bash
# 开启SSL
$ sudo a2enmod ssl
$ sudo a2ensite default-ssl
$ sudo /etc/init.d/apache2 restart
# 本地自签证书
# 使用 mkcert : https://github.com/FiloSottile/mkcert
PowerShell> mkcert -install locahost
# 会生成 localhost.pem和localhost-key.pem
# 
$ sudo vim /etc/apache2/sites-available/default-ssl.conf
# 修改内容，这里用的是windows用户名下文件位置，也可以复制到 wsl 下
#SSLCertificateFile     /etc/ssl/certs/ssl-cert-snakeoil.pem
#SSLCertificateKeyFile /etc/ssl/private/ssl-cert-snakeoil.key
SSLCertificateFile /mnt/c/Users/用户名/localhost.pem
SSLCertificateKeyFile /mnt/c/Users/用户名/localhost-key.pem

```

### 常见问题

1. PHP在使用chmod时出现 chmod(): Operation not permitte

```bash
#/etc/wsl.conf
#同时解决/mnt/c,d...等默认0777权限问题,在/etc/wsl.conf（如果没有就新建）编辑如下内容:
[automount]
enabled = true
root = /mnt/
options = "metadata,umask=22,fmask=111"
mountFsTab = true
[filesystem]
umask = 022
# 关闭WSL再重启
# Windows PowerShell> wsl --shutdown. 
# 不出意外就正常了。
# 如果不能解决 sudo chown www-data:www-data uploads 临时解决出错
```

2. 启动服务

```bash
$ sudo vim /usr/local/bin/startup
#----内容-----
/etc/init.d/mysql start
/etc/init.d/php8.1-fpm start
/etc/init.d/php7.2-fpm start
/etc/init.d/apache2 start
#---------  
$ sudo chmod a+x /usr/local/bin/startup
# 以后启动用 sudo startup启动
# 或者在windows下 wsl -d Ubuntu -u root 
```

3. SQLSTATE[HY000] [2054] The server requested authentication method unknown to the client

```bash
mysql> ALTER USER '用户名'@'localhost' IDENTIFIED WITH mysql_native_password BY '密码';
```

4. Could not instantiate mail function.

> 安装 sendmail或者postfix

```bash
$ sudo apt install sendmail
# 或者
$ sudo apt install postfix
```

5. ORDER BY clause is not in GROUP BY clause and contains nonaggregated

```
# 查看MySQL 的 sql_mode
mysql > select @@sql_mode;
# 默认值
# ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
# 原因是MySQL被Oracle收购后从MySQL5.7开始这个 ONLY_FULL_GROUP_BY 和 Oracle 一样，要求 SELECT 列一定要在 GROUP BY 中 ，或者本身是聚合列(SUM,AVG,MAX,MIN) 
# 去掉 ONLY_FULL_GROUP_BY 解除限制
# 如果出现 Invalid datetime format: 1292 Incorrect datetime value: '0000-00-00 00:00:00' 
# 去掉 NO_ZERO_IN_DATE,NO_ZERO_DATE ，或者把 0000-00-00 00:00:00 改成 NULL或者'1970-01-01 08:00:00'
$ sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
# 如果没有sql_mode加一行
sql_mode=STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
$ sudo /etc/init.d/mysql restart

```
