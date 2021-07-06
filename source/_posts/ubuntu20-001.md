---
title: ubuntu20-001
date: 2021-02-26 17:47:23
tags:
---


## 取消鼠标中键粘贴功能

新建开机启动命令：
```bash
vim /etc/profile.d/xmodmap.sh
```
xmodmap.sh 内容如下：
```bash
#!/bin/sh
# 取消鼠标中键粘贴功能,
# 不输出错误消息 Warning: Only changing the first 8 of 10 buttons.
xmodmap -e 'pointer = 1 25 3 4 5 6 7 2' 2>/dev/null
```

## 优化开机启慢问题
```bash
# 列出开机启动占用时间长的服务
$ systemd-analyze blame 

# 46.780s plymouth-quit-wait.service   
# 关闭开机启动动画，反向操作用umask
sudo systemctl mask plymouth-quit-wait.service
# 关闭自动升级
sudo systemctl disable apt-daily.service
# 修改启动时选择系统的等待时间，默认10秒，修改为2秒
# GRUB_TIMEOUT=2
sudo vim /etc/default/grub
# 更新开机引导
sudo update-grub
# 设置开机分辨率，解决开机黑屏时间长，
sudo apt-get install v86d hwinfo
# 查看显示卡支持的分辨率
sudo hwinfo --framebuffer
# 修改grub加一行
# GRUB_GFXPAYLOAD_LINUX=1024×768x24
sudo vim /etc/default/grub
#启用framebuffer
echo FRAMEBUFFER=y | sudo tee /etc/initramfs-tools/conf.d/splash
#更新设置：
sudo update-grub
sudo update-grub2
sudo update-initramfs -u

# 安装经典桌面，解决卡顿
# 启动后登录画面选择 Gnome Flashback
sudo apt install gnome-session-flashback 



```

```bash
# 查看硬盘读写速度
$ hdparm -t /dev/sdb

# 使用gparted把目标硬盘分区 
# 查看现有硬盘分区情况
$ fdisk -l
# 目标硬盘分区为
# Device     Boot   Start       End   Sectors   Size Id Type
# /dev/sda1          2048   1050623   1048576   512M  b W95 FAT32
# /dev/sda2       1050624 250068991 249018368 118.8G 83 Linux
# 源硬盘分区为
# Device     Boot   Start       End   Sectors   Size Id Type
# /dev/sdb1  *       2048   1050623   1048576   512M  b W95 FAT32
# /dev/sdb2       1052670 625141759 624089090 297.6G  5 Extended
# /dev/sdb5       1052672 625141759 624089088 297.6G 83 Linux
# 把sdb5复制到sda2
$ dd if=/dev/sdb5 of=/dev/sda2
# 另外开个终端执行以下命令后。dd命令可以看到dd命令执行过程
$ watch -n 5 killall -USR1 dd

```
