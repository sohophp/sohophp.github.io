---
title: CRM - Composer 源管理工具
date: 2021-02-05 12:57:42
tags: [Composer,PHP,CRM]
categories: [PHP,Composer]
---

> 以下引用github项目仓库READMD:
> https://github.com/slince/composer-registry-manager/edit/master/README-zh_CN.md
> 项目仓库网址：https://github.com/slince/composer-registry-manager

由于composer在国内下载速度非常慢，大家都习惯使用中国镜像，如果正在使用的镜像没有更新或者出现故障你可以使用Composer Registry Manager帮助你轻松地切换到另外一个镜像。
默认带了一些镜像,当然你也可以添加新的镜像。

## 安装

使用composer安装，执行下面命令

```bash
$ composer global require slince/composer-registry-manager
```

## 基本用法

### 列出所有可使用的镜像

```bash
$ composer repo:ls

 --- ------------- ------------------------------------------------ ------------------------------
      composer      https://packagist.org                            Europe, Canada and Singapore
      aliyun        https://mirrors.aliyun.com/composer              China
      tencent       https://mirrors.cloud.tencent.com/composer       China
  *   huawei        https://mirrors.huaweicloud.com/repository/php   China
      cnpkg         https://php.cnpkg.org                            China
      sjtug         https://packagist.mirrors.sjtug.sjtu.edu.cn      China
      phpcomposer   https://packagist.phpcomposer.com                China
      kkame         https://packagist.kr                             South Korea
      hiraku        https://packagist.jp                             Japan
      webysther     https://packagist.com.br                         Brazil
      solidworx     https://packagist.co.za                          South Africa
      indra         https://packagist.phpindonesia.id                Indonesia
      varun         https://packagist.in                             India
 --- ------------- ------------------------------------------------ ------------------------------
```
<!--more-->
标“*”表示当前正在使用的源;

你可以使用 `--location xx` 按地区过滤

```bash
$ composer repo:ls --location China
```

### 切换镜像

```bash
$ composer repo:use

Please select your favorite repository (defaults to composer) [composer]:
  [0 ] composer
  [1 ] aliyun
  [2 ] tencent
  [3 ] huawei
  [4 ] cnpkg
  [5 ] sjtug
  [6 ] phpcomposer
  [7 ] kkame
  [8 ] hiraku
  [9 ] webysther
  [10] solidworx
  [11] indra
  [12] varun
>
```
你也可以直接追加镜像名称来跳过选择

```bash
$ composer repo:use aliyun
```

添加选项 `--current/-c` 为当前项目切换源，默认是修改全局的源。

### 重置命令

如果你想丢弃所有自定义的镜像源，你可以使用下面命令：

```bash
$ composer repo:reset
```

### 所有命令

执行下面命令查看

```bash
$ composer repo
  _____   _____        ___  ___
 /  ___| |  _  \      /   |/   |
 | |     | |_| |     / /|   /| |
 | |     |  _  /    / / |__/ | |
 | |___  | | \ \   / /       | |
 \_____| |_|  \_\ /_/        |_|
 
 Composer Repository Manager version 2.0.0
 
 Usage:
   command [options] [arguments]
 
 Options:
   -h, --help            Display this help message
   -q, --quiet           Do not output any message
   -V, --version         Display this application version
       --ansi            Force ANSI output
       --no-ansi         Disable ANSI output
   -n, --no-interaction  Do not ask any interactive question
   -v|vv|vvv, --verbose  Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug
 
 Available commands for the "repo" namespace:
   repo:add     Creates a repository
   repo:ls      List all available repositories
   repo:remove  Remove a repository
   repo:use     Change current repository
 ```