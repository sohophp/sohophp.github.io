---
title: ubuntu下安装postfix+mysql+dovecot
date: 2021-05-21 15:51:54
tags:
---

## ubuntu
 

```bash
# 更新系统
$ apt update 
# 查看系统版本
$ uname -a 
# Linux 3fd33fb882db 5.8.0-53-generic #60~20.04.1-Ubuntu SMP Thu May 6 09:52:46 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux

# 因为我基于docker安装的ubuntu缺少一些工具

$ apt install iputils-ping vim net-tools telnet

```

## 新建mysql数据库 (已安装mysql-server，使用docker另一个mysql容器)

```bash
# 创建postfix用户，数据库和数据表,
mysql> CREATE DATABASE postfix;
# 选择postfix;
mysql> USE postfix;
# 创建postfix用户密码
mysql> CREATE USER postfix@'%' IDENTIFIED BY 'postfix';
# 指派权限
mysql> GRANT SELECT ,INSERT ,UPDATE ,DELETE ON postfix.* TO 'postfix'@'%';
# 刷新
mysql> FLUSH PRIVILEGES;
# 创建 domains表
mysql> CREATE TABLE domains (domain varchar(50) NOT NULL,PRIMARY KEY (domain));
# 创建 forwardings表
mysql> CREATE TABLE forwardings (source varchar(80) NOT NULL,destination TEXT NOT NULL,PRIMARY KEY(source));
# 创建 users表
mysql> CREATE TABLE users (email varchar(80) NOT NULL,`password` varchar(64) NOT NULL, quota int(10) default '104857600' , PRIMARY KEY (email));
# 创建 transport表
mysql> CREATE TABLE transport (domain varchar(128) NOT NULL default '',transport varchar(128) NOT NULL default '',UNIQUE KEY domain (domain));

```
测试数据
```bash 
# 域名
mysql> insert into domains values ('postfix.local');
# 邮箱用户
mysql> insert into users values ('user1@postfix.local',ENCRYPT('user1'),104857600);
# 创建邮件转发，多个邮箱地址使用,隔开
mysql> insert into forwardings values('user2@postfix.local','user1@postfix.local,user3@postfix.local');
# 转发所有邮件到另外的邮件服务器
#INSERT INTO transport VALUES ('domain.com', 'smtp:server2.domain.com');
```
## 安装postfix

```bash
#安装依赖
$ apt install openssl telnet libsasl2-2 libsasl2-modules libsasl2-modules-sql sasl2-bin libpam-mysql

# 安装postfix
$ apt install postfix postfix-mysql postfix-doc mailutils
 
# General type of mail configuration 输入数字 2 （Internet Site）
# System Mail Name 输入 postfix.local 
# Geographic area 时区输入数字 6 (Asia)
# Time Zone 输入数字 70 (Shanghai)


```


## 修改配置

虚拟domain配置文件

```bash 
$ vim /etc/postfix/mysql-virtual-domains.cf
```
内容：

```bash 
user = postfix
password = postfix
dbname = postfix
query = SELECT domain AS virtual FROM domains WHERE domain = '%s'
hosts = 172.18.0.136
```
虚拟forwarding配置文件

```bash
$ vim /etc/postfix/mysql-virtual-forwardings.cf

```
内容
```bash
user = postfix
password = postfix
dbname = postfix
query = SELECT destination FROM forwardings WHERE source='%s'
hosts = 172.18.0.136
```
虚拟mailbox配置文件

```bash
$ vim /etc/postfix/mysql-virtual-mailboxes.cf
```
内容
```bash
user = postfix
password = postfix
dbname = postfix
query = SELECT CONCAT(SUBSTRING_INDEX (email,'@',-1),'/',SUBSTRING_INDEX(email,'@',1),'/') FROM users WHERE email='%s'
hosts = 172.18.0.136
```

虚拟email2email配置文件
```bash
vim /etc/postfix/mysql-virtual-email2email.cf
```
内容
```bash
user = postfix
password = postfix
dbname = postfix
query = SELECT email FROM users WHERE email='%s'
hosts = 172.18.0.136
```

```bash
$ vim /etc/postfix/mysql-virtual-mydestination.cf
```
内容
```bash
user = postfix
password = postfix
dbname = postfix
table = transport
select_field = domain
where_field = domain
hosts = 172.18.0.136
```

修改配置文件权限
```bash
# 修改/etc/postfix/mysql-virtual-*.cf 用户组为postfix
$ chgrp postfix /etc/postfix/mysql-virtual-*.cf
# 禁止other用户访问
$ chmod o= /etc/postfix/mysql-virtual-*.cf

```

创建证书
```bash

$ mkdir /etc/postfix/certs
$ cd /etc/postfix/certs
# 创建证书
$ openssl req -new -outform PEM -out smtpd.cert -newkey rsa:2048 -nodes -keyout smtpd.key -keyform PEM -days 3650 -x509
# 修改权限
$ chmod o= smtpd.key

```

创建用户

```bash
$ groupadd -g 5000 vmail
$ useradd -c 'vmail' -g vmail -u 5000 vmail -d /home/vmail -m
```

使用postconf 配置main.cf

```bash
$ postconf -e 'myhostname = postfix.local'
$ postconf -e 'mydestination = localhost , proxy:mysql:/etc/postfix/mysql-virtual-mydestination.cf'
$ postconf -e 'mynetworks = 127.0.0.0/8, 172.18.0.125' 
$ postconf -e 'message_size_limit = 30720000'
$ postconf -e 'virtual_alias_domains ='
$ postconf -e 'virtual_alias_maps = proxy:mysql:/etc/postfix/mysql-virtual-forwardings.cf, mysql:/etc/postfix/mysql-virtual-email2email.cf'
$ postconf -e 'virtual_mailbox_domains = proxy:mysql:/etc/postfix/mysql-virtual-domains.cf'
$ postconf -e 'virtual_mailbox_maps = proxy:mysql:/etc/postfix/mysql-virtual-mailboxes.cf'

$ postconf -e 'virtual_mailbox_base = /home/vmail'
$ postconf -e 'virtual_uid_maps = static:5000'
$ postconf -e 'virtual_gid_maps = static:5000'
$ postconf -e 'smtpd_sasl_auth_enable = yes'
$ postconf -e 'broken_sasl_auth_clients = yes'
$ postconf -e 'smtpd_sasl_authenticated_header = yes'
$ postconf -e 'smtpd_recipient_restrictions = permit_mynetworks, permit_sasl_authenticated, reject_unauth_destination'
$ postconf -e 'smtpd_use_tls = yes'
$ postconf -e 'smtpd_tls_cert_file = /etc/postfix/certs/smtpd.cert'
$ postconf -e 'smtpd_tls_key_file = /etc/postfix/certs/smtpd.key'

# 以下为同一行
$ postconf -e 'proxy_read_maps = $local_recipient_maps $mydestination $virtual_alias_maps $virtual_alias_domains    $virtual_mailbox_maps $virtual_mailbox_domains $relay_recipient_maps $relay_domains $canonical_maps $sender_canonical_maps $recipient_canonical_maps $relocated_maps $transport_maps $mynetworks '
# 以上为同一行
 
#其它参考
#  queue_run_delay = 3s          //每3s扫描一次delay的邮件 
#  minimal_backoff_time = 3s         //在3s内不会重发delay的邮件
#  maximal_backoff_time = 6s         //在6s内则一定会重发邮件
#  maximal_queue_lifetime = 12s      //邮件超过12s没有发出，则退信
#  smtpd_sasl_auth_enable = yes      //启动SMTP认证
#  smtpd_sasl_security_options = noanonymous //禁止匿名使用SMTP服务
#  mynetworks = 127.0.0.1        //允许本服务器发送到外网地址
#  smtpd_recipient_restrictions = permit_mynetworks,permit_sasl_authenticated,reject_unauth_destination  //定义地址过滤规则

$ postconf -e 'virtual_transport = dovecot'
$ postconf -e 'local_transport = dovecot'

```

配置saslauthd

```bash
$ mkdir -p /var/spool/postfix/var/run/saslauthd
$ cp -a /etc/default/saslauthd /etc/default/saslauthd.bak
```

```bash
$ vim /etc/default/saslauthd
``` 
内容
```bash
START=yes
DESC="SASL Authentication Daemon"
NAME="saslauthd"
MECHANISMS="pam"
MECH_OPTIONS=""
THREADS=5
OPTIONS="-c -m /var/spool/postfix/var/run/saslauthd -r" 
```


```bash
$ vim /etc/pam.d/smtp
```
内容
```
auth    required   pam_mysql.so user=postfix passwd=postfix host=172.18.0.136 db=postfix table=users usercolumn=email passwdcolumn=password crypt=1
account sufficient pam_mysql.so user=postfix passwd=postfix host=172.168.0.136 db=postfix table=users usercolumn=email passwdcolumn=password crypt=1

```

```bash
$ vim /etc/postfix/sasl/smtpd.conf
```
内容
```
pwcheck_method: saslauthd
mech_list: plain login
allow_plaintext: true
auxprop_plugin: sql
sql_engine: mysql
sql_hostnames: 172.18.0.136
sql_user: postfix
sql_passwd: postfix
sql_database: postfix
sql_select: select password from users where email = '%u@%r'
```

修改权限

```bash 
$ chmod o= /etc/pam.d/smtp
$ chmod o= /etc/postfix/sasl/smtpd.conf
```
postfix加到sasl组

```bash
$ adduser postfix sasl

```

重启postfix ,saslauthd

```bash
$ /etc/init.d/postfix restart
$ /etc/init.d/saslauthd restart
```
