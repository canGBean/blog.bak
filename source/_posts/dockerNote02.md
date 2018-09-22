---
title: Docker笔记-学习过程中遇到的问题00
date: 2018-09-22 14:04:05
categories: 
- PaaS
- Docker
tags: 
- Docker
- 容器
---


**关于Docker笔记**

记录下个人学习Docker的使用和配置，均为入门级别，如有疏漏和错误的地方请指正。

笔记整理自:

http://www.runoob.com/docker/docker-tutorial.html
http://www.docker.org.cn/book/docker/what-is-docker-16.html

**Docker笔记内容**
- 学习至今的一些问题
- 1.为什么创建ubuntu容器，配置SSH后，启动SSH要用-D
- 2.删除问题1中的容器
- 3.删除Create状态容器
- 4.删除DEAD状态容器

<!--more-->

## 0.环境情况
运行了几个测试容器后，目前的所有容器情况如下

    [root@gd /]# docker ps -a
    CONTAINER ID        IMAGE               COMMAND               CREATED             STATUS              PORTS                NAMES
    4efda020260f        ubuntu:15.10        "/bin/bash"           About an hour ago   Up About an hour                         friendly_visvesvaraya
    4f8672a1b42d        ubuntu14-ssh        "/usr/sbin/sshd -D"   2 hours ago         Up 11 minutes       0.0.0.0:66->22/tcp   clever_curran
    e2366f3a995f        ubuntu14-ssh        "/usr/sbin/sshd -D"   6 weeks ago         Dead                                     amazing_agnesi

## 1.为什么创建ubuntu容器，配置SSH后，启动SSH要用-D
在之前的笔记中有，测试配置SSH容器，但是发现新启动一个容器必须要用-D参数。为什么需要用守护进程方式启动呢？

    [g@gd ~]$  sudo  docker run -d -p 66:22 ubuntu14-ssh /usr/sbin/sshd -D


## 2.删除-D的容器
删除环境中的clever_curran容器。需要先stop，然后在rm。如果rm执行报错，可以尝试rm -f。如下所示

首先停止容器    

    [root@gd /]# docker stop clever_curran 
    clever_curran
    
查看容器状态
    
    [root@gd /]# docker ps -a
    CONTAINER ID        IMAGE               COMMAND               CREATED             STATUS                     PORTS               NAMES
    4efda020260f        ubuntu:15.10        "/bin/bash"           About an hour ago   Up About an hour                               friendly_visvesvaraya
    4f8672a1b42d        ubuntu14-ssh        "/usr/sbin/sshd -D"   2 hours ago         Exited (0) 4 seconds ago                       clever_curran
    e2366f3a995f        ubuntu14-ssh        "/usr/sbin/sshd -D"   6 weeks ago         Dead                                           amazing_agnesi

移除容器，也可以强制移除 docker rm -f clever_curran

    [root@gd /]# docker rm clever_curran 
    clever_curran

再次查看容器状态
    
    [root@gd /]# docker ps -a
    CONTAINER ID        IMAGE               COMMAND               CREATED             STATUS              PORTS               NAMES
    4efda020260f        ubuntu:15.10        "/bin/bash"           About an hour ago   Up About an hour                        friendly_visvesvaraya
    e2366f3a995f        ubuntu14-ssh        "/usr/sbin/sshd -D"   6 weeks ago         Dead                                    amazing_agnesi


## 3.删除Create状态容器
同问题2中强制移除，可以使用rm -f来执行

## 4.删除DEAD状态容器
在删除DEAD状态容器时（容器name：amazing_agnesi），报如下错误：

    Error response from daemon: Driver devicemapper failed to remove root filesystem e2366f3a995f282768df1c2a6dac6f21d8c80f7fd9d84b2d14c7a8451392d124: remove /var/lib/docker/devicemapper/mnt/f44d19834a0b458f2b5f840ee2e370b9a4981a696b9dc2bdad7aa7c18365e056: device or resource busy

按提示umount了路径的文件还是报错。执行

    cat /proc/mounts | grep "mapper/docker" | awk '{print $2}'

后按提示umount了路径的文件还是报错

根据提示，可能是有某些地方还挂载着e2366f3a995f282768df1c2a6dac6f21d8c80f7fd9d84b2d14c7a8451392d124这个文件。于是

    [root@gd 22751]# find /proc/*/mounts | xargs grep -E "e2366f"
    grep: /proc/21578/mounts: 没有那个文件或目录
    /proc/22751/mounts:shm /var/lib/docker/containers/e2366f3a995f282768df1c2a6dac6f21d8c80f7fd9d84b2d14c7a8451392d124/shm tmpfs rw,context="system_u:object_r:container_file_t:s0:c259,c596",nosuid,nodev,noexec,relatime,size=65536k 0 0
根据返回值，显示进程22751占用了该文件，查看下22751是什么

    [root@gd ~]# ps aux|grep 22751
    root     21663  0.0  0.0 112656   992 pts/3    S+   15:54   0:00 grep --color=auto 22751
    root     22751  0.0  0.0 190780  4192 ?        Ss   8月22   0:02 /usr/sbin/cupsd -f

kill掉该进程

    [root@gd ~]# kill -9 22751
    [root@gd ~]# ps aux|grep 22751
    root     21673  0.0  0.0 112656   992 pts/3    S+   15:54   0:00 grep --color=auto 22751
    [root@gd ~]# 

再次删除容器，此时返回正确

    [root@gd mnt]# docker rm -f amazing_agnesi 
    amazing_agnesi
    [root@gd mnt]# docker ps -a
    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
    4efda020260f        ubuntu:15.10        "/bin/bash"         4 hours ago         Exited (0) 3 minutes ago                       friendly_visvesvaraya
