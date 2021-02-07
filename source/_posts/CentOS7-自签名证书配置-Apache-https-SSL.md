---
title: CentOS7 自签名证书配置 Apache https (SSL)
date: 2021-02-06 18:38:00
tags: [CentOS,CentOS7,SSL,Apache,Https]
categories: [Linux]
---

```Bash
$ su
$ yum install mod_ssl openssl openssl-devel
$ cp openssl.cnf openssl.cnf.bak

```