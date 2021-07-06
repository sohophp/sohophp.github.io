---
title: ubuntu20 install php7.2
date: 2021-05-03 11:15:00
tags: [php,ubuntu,linux]
---


```bash
# update and upgrade

$ sudo apt update && apt upgrade
# install software-properties-common

$ sudo apt-get install software-properties-common

# add apt repository ppa:ondrej/php

$ sudo add-apt-repository ppa:ondrej/php
# install php7.2 and extensions
# optional: libapache2-mod-php7.2 
# optional: php7.2-xdebug  
$ sudo apt install php7.2 \
php7.2-cli \
php7.2-common \
php7.2-curl \
php7.2-fpm \
php7.2-gd \
php7.2-json \
php7.2-mbstring \
php7.2-mcrypt \
php7.2-mysql  \
php7.2-opcache \
php7.2-sqlite3 \
php7.2-yaml \
php7.2-zip \
php7.2-dom
# enable startup php7.2-fpm
$ sudo systemctl enable php7.2-fpm
# start php7.2-fpm
$ sudo systemctl start php7.2-fpm
$ sudo systemctl status php7.2-fpm
$ sudo a2enmod proxy_fcgi setenvif
# enable rewrite
$ sudo a2emod enable rewrite
# enable php7.2-fpm
$ sudo a2enconf php7.2-fpm
$ sudo systemctl restart apache2
# 切換默認php命令 php7.2,php7.4 ...
update-alternatives --set php /usr/bin/php7.2

```