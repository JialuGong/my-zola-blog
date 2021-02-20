+++
title = "linux 笔记"
weight = 1
order = 1
date = 2021-01-16
insert_anchor_links = "right"
[taxonomies]
tags=["linux"]
+++

#### 一些工具
- 传感器
```shell
$ sudo pacman -S lm_sensors //安装sensors
$ sensors-detect //一路yes enter
$ sensors //查看各个传感器消息
$ sudo pwmconfig //配置风扇转速等,注意：每次注销或者reboot之后配置的转速会失效
```
- ptree
```
$ ptree -p
```
可查看进程树

#### 日志等
- journalctl 
```shell
# 查看登录日志
$ journalctl SYS_LOG_FACILITY=10 
# (-b -1）记录系统上次启动所有信息　（－p）优先级,显示具有指定优先级的条目
# 此处用于检查上一次莫名的ｆｒｅｅｚｉｎｇ
$ sudo journalctl sudo journalctl -b -1 -p 4 
# 查看所有的pci设备
$lspci
```
some reference:
1. [journalctl for system logs](https://www.digitalocean.com/community/tutorials/how-to-use-journalctl-to-view-and-manipulate-systemd-logs)

#### 一些快捷键
- `ctrl+l`:清屏
- `ctrl+a`:回到命令行头部,`ctrl+e`:回到命令行尾部
- `ctrl+u`:删除命令行
- `ctrl+w`:删除命令行的单词
- `ctrl+x,ctrl+e`:编辑命令行

#### git 笔记

#### 问题及解决方案
1. >webcam 无法使用问题
    - `lsusb`查找相关设备信息
    ```shell
    Bus 001 Device 002: ID 1bcf:2c00 Sunplus Innovation Technology Inc. Integrated Webcam
    ```
    - 查看uvc　module
    ```shell
    sudo modprobe uvcvideo
    sudo dmesg | grep video
    ```
    - 查看video的文件
    ```shell
    ls -al /dev/video*
    ```
    - 运行`xawtv`

    报错
    ```
     Warning: Unable to load any usable ISO8859 font #!important
     Warning: Missing charsets in String to FontSet conversion
    ```
    解决办法
    ```shell
    sudo pacman -S xorg-fonts-misc #!important
    # export LC_ALL=C #!(会出现终端中文乱码现象)
    reboot
    ```


[references:]
- [archwiki:webcam setup](https://wiki.archlinux.org/index.php/webcam_setup)
- [arch forums](https://bbs.archlinux.org/viewtopic.php?id=240618)
- [gentoo forums](https://forums.gentoo.org/viewtopic-p-8444638.html?sid=634131addc5597966673b217c571b77e)