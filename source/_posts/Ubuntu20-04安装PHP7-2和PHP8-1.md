---
title: WSL-Ubuntu20.04 安装 Apache2.4 + MySQL8.0 + PHP7.2 和 PHP8.1 ,composer,git 开发环境
date: 2022-04-30 11:55:58
tags:
---

### 安装 Apache 2.4

```bash
$ sudo apt update
$ sudo apt upgrade
$ sudo apt install vim net-tools curl wget
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
php7.2-zip

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
php8.1-zip 

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
php8.1 composer-setup.php
chmod +x composer.phar
sudo mv composer.phar /usr/local/bin/composer81
php7.2 composer-setup.php
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
