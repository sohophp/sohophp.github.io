---
title: docker
date: 2021-03-03 12:53:57
tags: Docker
categories: [Docker]
---

## mysql容器使用固定IP方法

```bash
# 创建自定网络
docker network create --subnet=172.18.0.0/16 mynetwork
# 指定使用的网络和ip运行mysql
docker run -p 33306:3306 --name mysql -v /home/jason/var/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=xxxxxx --net mynetwork --ip 172.18.0.136 -d mysql:5.6 
docker run --name php -v /home/jason/webroot:/var/www/html --net mynetwork --ip 172.18.0.80 -itd sohophp/php-multi-apache:latest start

```
