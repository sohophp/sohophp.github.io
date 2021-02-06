---
title: CentOS7+Apache(Httpd)2.4+PHP7.2+MySQL(MariaDB)
date: 2021-02-05 16:34:30
tags: [CentOS,CentOS7,Apache,Httpd,PHP,PHP7,PHP7.2,MySQL,MariaDB,PHP-FPM]
categories: [Linux,CentOS]
toc: true
--- 


## 安装CentOS7

### 设定IP,HostName
   
   修改主机名

```Bash
$ hostnamectl set-hostname "主机名"
$ cat /etc/hostname
主机名
```


设定IP
nmtui 设定工具菜单,ssh client(Xshell,putty等)连接后设定connection就不能save了。

```Bash 
$ nmtui   
```

 ![](CentOS7-Apache-Httpd-2-4-PHP7-2-MySQL-MariaDB/20210205165530.png)
 
 查看网卡


 ```Bash 
   ip addr 
 ```
 <!--more-->
---

 进入/etc/sysconfig/network-scripts/编辑ifcfg-eth0

```Bash
$ cd /etc/sysconfig/network-scripts/
$ ll
total 232
-rw-r--r--. 1 root root   348 Feb  5 03:49 ifcfg-eth0
-rw-r--r--. 1 root root   254 Aug 19  2019 ifcfg-lo
...
$ vi ifcfg-eth0
```
![](CentOS7-Apache-Httpd-2-4-PHP7-2-MySQL-MariaDB/20210205170217.png)


如果不是使用root编辑会没有写权限 read only , 在vi中 ESC 输入 :w !sudo tee %


重启网卡

```Bash
$ ifup eth0

```

换Xshell连ssh截图和复制方便

修改时区

```Bash
# 删除原来的时区文件
$ sudo rm -rf /etc/localtime
使用上海时间 
$ sudo ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
$ sudo vim /etc/sysconfig/clock 
> ZONE="Asia/Shanghai"
> UTC=false
> ARC=false

```

安装 ntp
```Bash
# 安装 ntp,ntpdate
$ sudo yum install ntp ntpdate
# 指定ntp服务器同步时间
$ sudo ntpdate cn.pool.ntp.org
# 将系统时间写入硬件时间
$ sudo hwclock --systohc
# 查看系统时间
$ date
# 查看硬件时间
$ hwclock
# 查看时区
$ date -R
# 查看ntp配置文件，有些默认服务器
$ cat /etc/ntp.conf
# 设置开机启动
$ sudo systemctl enable ntpd
# 现在启动
$ sudo systemctl start ntpd
# 配置开机启动校验
$ sudo vim /etc/rc.d/rc.local
> /usr/sbin/ntpdate cn.pool.ntp.org > /dev/null 2>&1; /sbin/hwclock -w
# 配置定时任务
$ sudo crontab -e 
> 0 */1 * * * ntpdate cn.pool.ntp.org > /dev/null 2>&1; /sbin/hwclock -w
# 查看root定时任务
$ sudo crontab -l
> 0 */1 * * * ntpdate cn.pool.ntp.org > /dev/null 2>&1; /sbin/hwclock -w
# 查看现在时间
$ date -R
> Sat, 06 Feb 2021 12:44:09 +0800
```
 

安装 strace
在Linux系统中，strace命令是一个集诊断、调试、统计与一体的工具，可用来追踪调试程序，能够与其他命令搭配使用

```Bash
# 安装 strace
$ sudo yum install strace
```

查看selinux状态

```Bash
$ sestatus
SELinux status:                 enabled    (开启)
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing （强制执行）
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Max kernel policy version:      31 
```
查看firewalld状态

```Bash
$ systemctl status firewalld

firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running)(在运行) since Fri 2021-02-05 03:31:43 EST; 57min ago
     Docs: man:firewalld(1)
 Main PID: 719 (firewalld)
   CGroup: /system.slice/firewalld.service
           └─719 /usr/bin/python2 -Es /usr/sbin/firewalld --nofork --nopid

```
查看firewalld开放端口（--list-ports）和 服务(--list-services)

```Bash
$ sudo firewall-cmd --list-all 

public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0
  sources: 
  services: dhcpv6-client ssh
  ports: 
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
``` 
更新系统

```Bash
$ sudo yum update

```


## 安装Apache(Httpd)

```Bash
$ sudo yum install httpd  
```
开机启动httpd
```Bash
$ sudo systemctl enable httpd
```

现在启动httpd

```Bash
$ sudo systemctl start httpd

```

安装net命令行工具。
```Bash
$ sudo yum install net-tools -y
```
查看端口监听 （80/tcp 被 httpd 监听，说明httpd已经启动）
netstat 参数
-t (tcp) 仅显示tcp相关选项
-u (udp)仅显示udp相关选项
-n 拒绝显示别名，能显示数字的全部转化为数字
-l 仅列出在Listen(监听)的服务状态
-p 显示建立相关链接的程序名

```Bash
$ sudo netstat -tnlp | grep 80
tcp6       0      0 :::80      :::*     LISTEN      21835/httpd  
```


查看firewalld是否支持http服务

```Bash
$ sudo firewall-cmd --get-services | sed 's/ /\n/g' | grep http
http
https
wbem-http
wbem-https
```

firewalld开放http服务（80端口）,也可以用--add-port=80/tcp

```Bash
$ sudo firewall-cmd --zone=public --add-service=http --permanent
# 或者
$ sudo firewall-cmd --zone=public --add-port=80/tcp --permanent
```
重新加载firewalld配置使设定生效

```Bash
$ sudo firewall-cmd --reload
```

修改/var/www/html所属用户
```
$ sudo chown -R jason:jason /var/www/html
$ ll /var/www/

total 0
drwxr-xr-x. 2 root  root   6 Nov 16 11:19 cgi-bin
drwxr-xr-x. 2 jason jason 24 Feb  5 07:25 html
```
安装VIM
```Bash
$ sudo yum install vim -y
```

新建并编辑测试网页
```Bash
$ vim /var/www/html/index.html
```
内容
```Html
<html>
   <head>
   <meta chatset="utf-8" />
   <title>test title</title>
   </head>
   <body>
      <h1>test body </h1>
   <body>
</html>
```
测试静态页，或者浏览器打开IP网址
```Bash
$ curl http://localhost
```

---


## 安装 MariaDB (CentOS7默认用MariaDB而不是MySQL)

```Bash
$ sudo yum install mariadb mariadb-server
```
设定开机启动mariadb
```Bash
$ sudo systemctl enable mariadb 
```
现在启动mariadb
```Bash
$ sudo systemctl start mariadb
```

首次安装设定root密码,删除匿名用户，删除test数据表，
```Bash
$ sudo mysql_secure_installation
# 输入当前root密码，默认密码为空
Enter current password for root (enter for none): 
# 是否修改root密码 
Set root password? [Y/n] y
# 输入新密码
New password: 
# 再次输入新密码
Re-enter new password: 
# 移除匿名用户
Remove anonymous users? [Y/n] y
# 关闭root远程登录
Disallow root login remotely? [Y/n] y 
# 删除测试(test)数据库
Remove test database and access to it? [Y/n] y
# 重新加载特权表，使用修改生效
Reload privilege tables now? [Y/n] y

```

连接 mysql，添加用户。方便测试用%,正式使用localhost

```
# 登录mysql
$ mysql -u root -p
# 新建用户
MariaDB [(none)]> CREATE USER '用户名'@'%' IDENTIFIED BY '密码';
# 赋予权限
MariaDB [(none)]> GRANT PRIVILEGES ON *.* TO '用户名'@'%';
# 使生效 
MariaDB [(none)]> FLUSH PRIVILEGES;
# 退出
MariaDB [(none)]> quit;

```

## 安装PHP 7.2 

查看YUM安装PHP版本
```Bash
$ sudo yum info php
```

CentOS 7 YUM 默认安装php是5.4,所以安装YUM源用来安装PHP7.2
```Bash
$ sudo yum install epel-release
$ sudo rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
# 查看PHP版本
$ sudo yum search php | grep php72
$ sudo yum install php72w-cli php72w-common php72w-devel php72w-fpm php72w-gd php72w-mbstring php72w-mysqlnd php72w-pdo php72w-xml mod_php72w
$ php -v
PHP 7.2.34 (cli) (built: Oct  1 2020 13:37:37) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies

$ php-fpm -v
PHP 7.2.34 (fpm-fcgi) (built: Oct  1 2020 13:40:44)
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies

```
新建并编辑php测试文件
```Bash
$ rm -f /var/www/html/index.html
$ vim /var/www/html/index.php 
```
内容

```PHP
<?php phpinfo(); ?>
```

测试 http://ip/index.php
或者:
```Bash
$ curl -I http://localhost/index.php
HTTP/1.1 200 OK
Date: Fri, 05 Feb 2021 14:11:07 GMT
Server: Apache/2.4.6 (CentOS) PHP/7.2.34
X-Powered-By: PHP/7.2.34
Content-Type: text/html; charset=UTF-8
```


[CentOS7下PHP7.2安装mcrypt](CentOS7下PHP7-2安装mcrypt.md)


### php-fpm

https://cwiki.apache.org/confluence/display/HTTPD/PHP-FPM

```Bash
# 开机启动php-fpm
$ sudo systemctl enable php-fpm
# 现在启动php-fpm
$ sudo systemctl start php-fpm
```
查看php-fpm状态

```Bash
$ sudo systemctl status php-fpm
 
 ● php-fpm.service - The PHP FastCGI Process Manager
   Loaded: loaded (/usr/lib/systemd/system/php-fpm.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2021-02-05 09:30:59 EST; 22min ago
 Main PID: 23148 (php-fpm)
   Status: "Processes active: 0, idle: 5, Requests: 8, slow: 0, Traffic: 0req/sec"
   CGroup: /system.slice/php-fpm.service
           ├─23148 php-fpm: master process (/etc/php-fpm.conf)
           ├─23149 php-fpm: pool www
```


查看端口监听

```Bash
$ sudo netstat -nltp | grep 9000
tcp        0      0 127.0.0.1:9000          0.0.0.0:*               LISTEN      23148/php-fpm: mast 

```

 ### 创建VirtualHost，使用自定域名/主机名，使用php-fpm,开启rewrite,.htaccess

修改/etc/hosts加入自定域名
```Bash 
$ sudo vim /etc/hosts
127.0.0.1 php72.vm  localhost localhost.localdomain localhost4 localhost4.localdomain4
::1       php72.vm  localhost localhost.localdomain localhost6 localhost6.localdomain
```

测试域名
```Bash
$ ping php72.vm
PING php72.vm (127.0.0.1) 56(84) bytes of data.
64 bytes from php72.vm (127.0.0.1): icmp_seq=1 ttl=64 time=0.046 ms
```

新建VirtualHost
```Bash
$ sudo vim /etc/httpd/conf.d/vhosts.conf

<VirtualHost *:80>
  DocumentRoot "/var/www/html"
  ServerName localhost
  ServerAlias php72.vm
# ProxyRequests Off
  ProxyPassMatch ^/(.*\.php(/.*)?)$ fcgi://127.0.0.1:9000/var/www/html/$1

</VirtualHost>
```
修改httpd.conf开启.htaccess

```Bash
$ sudo vim /etc/httpd/conf/httpd.conf

<Directory "/var/www/html"> 
    # 只要动态链接
    Options FollowSymLinks 
    # 使用.htaccess
    AllowOverride All 
    Require all granted
</Directory>
```

重启httpd
```Bash
$ sudo systemctl restart httpd
```
测试http://php72.vm/index.php 或者：

```Bash
$ curl http://php72.vm | grep php-fpm
```

测试 PUT METHOD
```Bash
$ curl -X PUT -I  http://localhost
HTTP/1.1 200 OK
Date: Fri, 05 Feb 2021 15:22:02 GMT
Server: Apache/2.4.6 (CentOS) PHP/7.2.34
X-Powered-By: PHP/7.2.34
Transfer-Encoding: chunked
Content-Type: text/html; charset=UTF-8
```

测试 DELETE METHOD

```Bash
$ curl -X DELETE -I  http://localhost
HTTP/1.1 200 OK
Date: Fri, 05 Feb 2021 15:22:12 GMT
Server: Apache/2.4.6 (CentOS) PHP/7.2.34
X-Powered-By: PHP/7.2.34
Transfer-Encoding: chunked
Content-Type: text/html; charset=UTF-8
```

安装Git

```Bash
# 安装 Git
$ sudo yum install git -y
# 查看 Git版本
$ git --version
git version 1.8.3.1
```

安装Composer

```Bash
$ curl https://install.phpcomposer.com/installer -o ~/composer-setup.php
$ php ~/composer-setup.php 
$ sudo mv ~/composer.phar /usr/bin/composer
$ sudo chmod +x /usr/bin/composer
$ composer --version
Composer version 2.0.9 2021-01-27 16:09:27
```
## 安装ModSecurity
 [安装 ModSecurity](ModSecurity.md)
 [安装 CRS 规则集](ModSecurity.md)