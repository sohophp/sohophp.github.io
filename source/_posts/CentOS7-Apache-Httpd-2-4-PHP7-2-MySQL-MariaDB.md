---
title: CentOS7+Apache(Httpd)2.4+PHP7.2+MySQL5.6
date: 2021-02-05 16:34:30
tags: [CentOS,CentOS7,Apache,Httpd,PHP,PHP7,PHP7.2,MySQL,MariaDB,PHP-FPM]
categories: [Linux,CentOS]
toc: true
--- 

## 安装CentOS7

### 虚拟机安裝 CentOS7

#### 选择虚拟机 Virtual Machine

### 最小安装CentOS7

以下是用 Hyper-V +　CentOS-7-x86_64-Minimal-2003.iso

一個root密碼，一個管理員用戶（用于VSCode|PHPStorm 使用ssh|sftp直接编辑项目），
（安装gitea加了个git用户，没有sudo权限）

其它全部 auto

### 修改主机名

```Bash
$ hostnamectl set-hostname "php72.vm"
$ cat /etc/hostname
php72.vm
```

### 配置网络

 查看网卡

 ```Bash
   ip addr 
 ```
 <!--more-->
---
修改网络配置

```Bash
$ cd /etc/sysconfig/network-scripts/
$ ll
total 232
-rw-r--r--. 1 root root   348 Feb  5 03:49 ifcfg-eth0
-rw-r--r--. 1 root root   254 Aug 19  2019 ifcfg-lo  
$ sudo vim /etc/sysconfig/network-scripts/ifcfg-eth0

TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=eth0
UUID=0296b603-2823-40a5-9f39-7743e87b527a
DEVICE=eth0
ONBOOT=yes
IPADDR=192.168.3.105
PREFIX=24
GATEWAY=192.168.3.1
DNS1=192.168.3.1 

# 如果不是使用root编辑会没有写权限 read only , 
# 在vi中 ESC 输入 :w !sudo tee % 
# 重启网卡
$ ifup eth0 
```

换Xshell连ssh截图和复制方便

### 修改时区

```Bash
# 删除原来的时区文件
$ sudo rm -rf /etc/localtime
# 使用上海时间 
$ sudo ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
$ sudo vim /etc/sysconfig/clock 
> ZONE="Asia/Shanghai"
> UTC=false
> ARC=false

```

### 安装 ntp 校验时间

```Bash
# 安装 ntp,ntpdate
$ sudo yum install ntp ntpdate
# 指定ntp服务器同步时间
$ sudo ntpdate cn.pool.ntp.org
# 将系统时间写入BIOS
$ sudo hwclock --systohc
# 查看系统时间
$ date
# 查看硬件时间
$ hwclock
# 查看时区
$ date -R
# 查看ntp配置文件，有些预设服务器
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
sudo yum update
```

## 安装Apache(Httpd)

```Bash
sudo yum install httpd  
```

开机启动httpd

```Bash
sudo systemctl enable httpd
```

现在启动httpd

```Bash
sudo systemctl start httpd

```

查看netstat命令属于哪个包

```Bash
yum whatprovides netstat
```

安装net-tools

```Bash
sudo yum install net-tools -y
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
sudo firewall-cmd --reload
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
sudo yum install vim -y
```

新建并编辑测试网页

```Bash
vim /var/www/html/index.html
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
curl http://ip
```

---

## 安装 MySQL5.6
  
### 安装MySQL5.6

```Bash
# CentOS7默认是mariadb,如果已经安装，卸载mariadb
$ yum remove mariadb mariadb-server 
```

安装MySQL5.6要用到的YUM源网址
<https://dev.mysql.com/downloads/repo/yum/>

```Bash
# 下载 Red Hat Enterprise Linux 7
$ wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
# 安装release
$ yum localinstall mysql80-community-release-el7-3.noarch.rpm
# 查看 MySQL 开启的 YUM 源
$ yum repolist all | grep mysql
# 关闭8 开启5.6 安装 mysql-server 
# 因为我安装过mariadb出现错误不能启动，卸载再次重新安装后正常
$ yum --disablerepo=mysql80-community --enablerepo=mysql56-community install mysql-server
$ systemctl start mysqld
$ systemctl enable mysqld
```

使用yum-config-manager 关闭mysql80-community 开启 mysql56-community
也可以直接编辑/etc/yum.repos.d/ 下的配置文件
(非必要) 怕以后忘了更新成mysql8

```Bash
yum whatprovides yum-config-mananger
yum install yum-utils
yum repolist all | grep mysql
yum-config-manager --disable  mysql80-community
yum-config-manager --enable mysql56-community
yum repolist all | grep -E 'mysql.*enabled'
```

### 初始化

首次安装设定root密码,删除匿名用户，删除test数据库，

```Bash
$ sudo mysql_secure_installation
# 输入当前root密码，预设密码为空
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
# 重新加载特权表，使修改生效
Reload privilege tables now? [Y/n] y

```

### 连接 MySQL ,添加用户

方便测试用%,正式使用localhost

```
# 登录mysql
$ mysql -u root -p
# 新建用户
MySQL> CREATE USER '用户名'@'%' IDENTIFIED BY '密码';
# 赋予权限,简单方便用 ALL
MySQL> GRANT ALL PRIVILEGES ON *.* TO '用户名'@'%';
# 使生效 
MySQL> FLUSH PRIVILEGES;
# 退出
MySQL> quit;

```

---

## 安装PHP 7.2

查看YUM安装PHP版本

```Bash
sudo yum info php
```

CentOS 7 YUM 默认安装php是5.4,所以安装YUM源用来安装PHP7.2

```Bash
$ sudo yum install epel-release
$ sudo rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
# 查看PHP版本
$ sudo yum search php | grep php72
# 安装PHP72和用到的扩展
$ sudo yum install php72w-cli \
php72w-common \
php72w-devel \
php72w-fpm \
php72w-gd \
php72w-mbstring \
php72w-mysqlnd \
php72w-pdo \
php72w-xml \
mod_php72w
# 查看php版本
$ php -v
PHP 7.2.34 (cli) (built: Oct  1 2020 13:37:37) ( NTS )
# 查看php-fpm版本
$ php-fpm -v
PHP 7.2.34 (fpm-fcgi) (built: Oct  1 2020 13:40:44)
```

新建并编辑php测试文件

```Bash
rm -f /var/www/html/index.html
vim /var/www/html/index.php 
```

内容

```PHP
<?php phpinfo(); ?>
```

测试 <http://ip/index.php>
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

<https://cwiki.apache.org/confluence/display/HTTPD/PHP-FPM>

```Bash
# 开机启动php-fpm
$ sudo systemctl enable php-fpm
# 现在启动php-fpm
$ sudo systemctl start php-fpm
# 查看php-fpm状态
$ sudo systemctl status php-fpm
```

查看端口监听

```Bash
$ sudo netstat -nltp | grep 9000
tcp        0      0 127.0.0.1:9000          0.0.0.0:*               LISTEN      23148/php-fpm: mast 

```

所有.php使用php-fpm代理

```Bash
$ sudo vim /etc/httpd/conf.d/php-fpm.conf

> <FilesMatch \.php$>
>    SetHandler "proxy:fcgi://127.0.0.1:9000"
> </FilesMatch>
```

重启apache测试

```Bash
# 配置是否有錯
$ apachectl configtest
# 重启apache
$ sudo systemctl restart httpd
# 测试php-fpm
$ curl http://localhost/index.php | grep php-fpm

```

### 使用 .user.ini

```Bash
$ var /etc/php.ini
user_ini.filename = ".user.ini"
user_ini.cache_ttl = 300
$ systemctl restart httpd
# 在站点下新|修改.user.ini ,有缓存时间不会立即生效
$ vim .user.ini
> upload_max_filesize="20M"
```

### 创建VirtualHost，使用自定域名/主机名，开启rewrite,.htaccess

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
sudo systemctl restart httpd
```

测试<http://php72.vm/index.php> 或者：

```Bash
curl http://php72.vm | grep php-fpm
```

测试 PUT METHOD

```Bash
$ curl -X PUT -I  http://localhost
HTTP/1.1 200 OK
```

测试 DELETE METHOD

```Bash
$ curl -X DELETE -I  http://localhost
HTTP/1.1 200 OK
```

---

安装Git

```Bash
# 安装 Git
$ sudo yum install git -y
# 查看 Git版本
$ git --version
> git version 1.8.3.1
```

安装Composer

```Bash
$ curl https://install.phpcomposer.com/installer -o ~/composer-setup.php
$ php ~/composer-setup.php 
$ sudo mv ~/composer.phar /usr/bin/composer
$ sudo chmod +x /usr/bin/composer
$ composer --version
> Composer version 2.0.9 2021-01-27 16:09:27
```

---

## 安装 ModSecurity

### 安装 ModSecurity
<https://www.modsecurity.org/>

```Bash
# 安装mod_security
$ yum install mod_security
# 
# 重启httpd
$ sudo systemctl restart httpd
```

### 安装 CRS (CoreRuleSet) 规则集

<https://coreruleset.org/installation/>  

<!--more-->
```Bash

# 以下以root运行
$ su
# 安装 wget
$ sudo yum install wget 
# 进入/etc/httpd下
$ cd /etc/httpd
# 下载CRS最新版
$ wget https://github.com/coreruleset/coreruleset/archive/v3.3.0.tar.gz
# 解压缩
$ tar -zxvf v3.3.0.tar.gz
# 创建动态链接
$ ln -s coreruleset-3.3.0 /etc/httpd/crs
# 复制配置文件
$ cp crs/crs-setup.conf.example crs/crs-setup.conf
# 修改conf
$ sudo vim /etc/httpd/conf.d/mod_security.conf
# 加两行
> IncludeOptional crs/crs-setup.conf
> IncludeOptional crs/rules/*.conf
# 检查 httpd配置是否有错
$ apachectl configtest
# 重启 httpd
$ systemctl restart httpd
# 触发警报以进行测试
$ curl -I http://localhost?exec=/bin/bash
> HTTP/1.1 403 Forbidden
# 测试 PUT
curl -X PUT -I http://localhost
> HTTP/1.1 403 Forbidden

```

### 开放PUT,DELETE 方法

```Bash
$ cd /etc/httpd/crs/rules
$ cp REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf.example REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf
$ cp RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf.example RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf
# 从 REQUEST-901-INITIALIZATION.conf 中复制一段修改加到REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf 下
# id 要修改为唯一，在tx.allowed_methods里加PUT DELETE
$ vim REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf

SecRule &TX:allowed_methods "@eq 0" \
    "id:9011601,\
    phase:1,\
    pass,\
    nolog,\
    ver:'OWASP_CRS/3.3.0',\
    setvar:'tx.allowed_methods=GET PUT DELETE HEAD POST OPTIONS'"
$ apachectl configtest
$ systemctl restart httpd
$ curl -X PUT -I http://localhost
> HTTP/1.1 200 OK
```

### 添加白名单

```Bash

# 清空或者备份日志文件
# $ echo "" >/var/log/httpd/error_log
# 查看ModSecurity相关日志
$ tail -f /var/log/httpd/error_log | grep ModSecurity
# 找到 [id "941160"] 这类id,把规则id加到自己建的白名单文件
$ echo "SecRuleRemoveById 941160" >> /etc/httpd/crs/rules/white-list.conf
$ echo "SecRuleRemoveById 949110" >> /etc/httpd/crs/rules/white-list.conf
$ echo "SecRuleRemoveById 980130" >> /etc/httpd/crs/rules/white-list.conf
$ cat /etc/httpd/crs/rules/white-list.conf
> SecRuleRemoveById 941160
> SecRuleRemoveById 949110
> SecRuleRemoveById 980130
# 重启apache
$ systemctl restart httpd
$ echo "" >/var/log/httpd/error_log
$ tail -f /var/log/httpd/error_log
# 再测试还是有太多被匹配到。只是因为没有达到阈值能通过。
# 再测试找到误判id逐个写到white-list.conf
# 可能有重复的删除
$ sort white-list.conf | uniq > tmp.conf ;mv -f tmp.conf white-list.conf
$ cat white-list.conf
SecRuleRemoveById 11862
SecRuleRemoveById 932100
SecRuleRemoveById 932110
SecRuleRemoveById 932130
SecRuleRemoveById 941100
SecRuleRemoveById 941110
SecRuleRemoveById 941160
SecRuleRemoveById 941310
SecRuleRemoveById 942100
SecRuleRemoveById 949110
SecRuleRemoveById 980130
# 测试阶段可以修改 SecRuleEngine 为只检测
$ vim /etc/httpd/conf.d/mod_security.conf
#SecRuleEngine On
SecRuleEngine DetectionOnly
# 测试没问题再改为 On
# 重启 Apache 再接受蹂躏
$ systemctl restart httpd
```

### mod_evasive DDos 防火墙 for apache

一定要有DDOS防火墙
不然就等于光着身子被乱箭穿身，
总能被刺中一次！
最好是有硬件防火墙!
===

参考网址
<https://www.digitalocean.com/community/tutorials/how-to-protect-against-dos-and-ddos-with-mod_evasive-for-apache-on-centos-7>

epel for CentOS7 网址
<https://centos.pkgs.org/7/epel-x86_64/epel-release-7-13.noarch.rpm.html>

```Bash
# 查看是否安装了epel 
$ yum repolist all | grep epel
# 如果没有下载最新版
$ wget https://download-ib01.fedoraproject.org/pub/epel/7/x86_64/Packages/e/epel-release-7-13.noarch.rpm
$ yum localinstall epel-release-7-13.noarch.rpm
# 安装 mod_evasive
$ yum install yum-plugin-protectbase mod_evasive

```

```Bash
$ cat /etc/httpd/conf.d/mod_evasive.conf | grep -Ev '^$|#'

LoadModule evasive20_module modules/mod_evasive24.so 
<IfModule mod_evasive24.c>
    DOSHashTableSize    3097
    # 同一页每DOSPageInterval秒最大请求数，超过会把IP加入黑名单，预设 2          
    DOSPageCount        3       
    # 同一个网站每DOSSiteInterval秒最大请求数，超过会把IP加入黑名单  
    # 预设 50,考虑一个网页有多个css,js,图片等可以大些
    DOSSiteCount        50       
    DOSPageInterval     1        
    DOSSiteInterval     1
    # 加入黑名单秒数，预设10，可以大些        
    DOSBlockingPeriod   180 
    # 日志目录，要手动mkdir并修改权限 
    # DOSLogDir           "/var/log/mod_evasive"
    # 加黑名单时执行的命令，可以搭配firewalld封IP
    # DOSSystemCommand
</IfModule>

```

```Bash
# 添加 IP 白名单，用于纠错或者特殊需要
DOSWhitelist    111.111.111.111
DOSWhitelist    222.222.222.222
```

```Bash
# 重启apache
$ systemctl restart httpd
```

---

## 其它

### SELinux

#### SELinux 对指定目录开放写权限,或者web根目录全开

 ```Bash
 # 对指定目录开放写权限,或者web根目录全开
 $ sudo chcon -R -t httpd_sys_rw_content_t ./uploads
 ```

查看SElinux策略内httpd相关规则的布尔值
原则是先按默认,在需要时打开相关设定

```Bash
$ getsebool -a | grep httpd
httpd_anon_write --> off     匿名写关闭
httpd_builtin_scripting --> on
httpd_can_check_spam --> off
httpd_can_connect_ftp --> off     PHP连接ftp如果需要才打开
httpd_can_connect_ldap --> off
httpd_can_connect_mythtv --> off
httpd_can_connect_zabbix --> off
httpd_can_network_connect --> off    需要时打开,网络资源的访问如redis,读写网络资源
httpd_can_network_connect_cobbler --> off
httpd_can_network_connect_db --> off  连接外网数据库,如果需要才打开
httpd_can_network_memcache --> off    连接外网memcache
httpd_can_network_relay --> off
httpd_can_sendmail --> off          mail发信,要打开
httpd_dbus_avahi --> off 
httpd_dbus_sssd --> off
httpd_dontaudit_search_dirs --> off
httpd_enable_cgi --> on               PHP-FPM 是fast-cgi 要打开
httpd_graceful_shutdown --> on 
...下边其它都是off
```

### Sendmail

#### mail: command not found

```Bash
# 安装 sendmail mailx
$ sudo yum install sendmail mailx jwhois 
$ sudo systemctl enable sendmail
$ sudo systemctl start sendmail
# SELinux 开放httpd发信
$ sudo setsebool -P httpd_can_sendmail 1

```

```Bash
$ getsebool -a | grep -Ev off | grep httpd
httpd_builtin_scripting --> on
httpd_can_sendmail --> on
httpd_enable_cgi --> on
httpd_graceful_shutdown --> on


$ firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0
  sources: 
  services: dhcpv6-client http https ssh
  ports: 3000/tcp   (3000是因为我安装了gitea不包含在测试环境)
  protocols: 
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 

```

---

至此,
CentOS7

+ Apache 2.4
+ PHP-FPM 7.2
+ MySQL 5.6
+ ModSecurity
+ CRS
+ mod_evasive
+ Firewalld
+ SELinux
  
测试环境正常.

持续更新...

---
