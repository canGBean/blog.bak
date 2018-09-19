---
title: Docker笔记-docker run命令及例子
date: 2018-09-19 12:54:17
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
- docker run命令
- ssh进入docker例子
    
<!--more-->

## docker run命令
创建一个新的容器并运行一个命令
    
    docker run
OPTIONS说明：

|命令|说明|
|----|----|
|-a|指定标准输入输出内容类型，可选 STDIN/STDOUT/STDERR 三项。-a stdion|
|-d|后台运行容器，并返回容器ID|
|-i|以交互模式运行容器，通常与-t同时使用|
|-p|端口映射，格式为：主机(宿主)端口:容器端口|
|-t|为容器重新分配一个伪输入终端，通常与 -i 同时使用|
|--name|为容器指定一个名称。--name="nginx-lb"|
|--dns|指定容器使用的DNS服务器，默认和宿主一致。--dns 8.8.8.8|
|--dns-search|指定容器DNS搜索域名，默认和宿主一致。--dns-search example.com|
|-h|指定容器的hostname|
|-e|设置环境变量。-e username="ritchie"|
|--env-file=[]|从指定文件读入环境变量|
|--cpuset="0-2" or --cpuset="0,1,2"|绑定容器到指定CPU运行|
|-m|设置容器使用内存最大值|
|--net="bridge"|指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型|
|--link=[]|添加链接到另一个容器|
|--expose=[]|开放一个端口或一组端口|

## 例子0
启动一个ubuntu容器，设置本地44端口映射容器的22端口，容器中安装SSH服务，本地通过SSH连接到容器中.
### 0.0创建并进入容器
    sudo docker run -itd -p 44:22 ubuntu:14.04 /bin/bash
    sudo docker attach [容器ID]
### 0.1在容器中安装ssh
    root@572bd8f35023:/# apt-get update
    root@572bd8f35023:/# apt-get install  openssh-client openssh-server 
    root@572bd8f35023:/# service ssh start
由于ubuntu中默认不允许root账号通过帐号密码登录，所以需修改配置文件，路径如下：

    root@572bd8f35023:/etc/ssh# vi sshd_config
修改以下内容：

    #Authentication:
    LoginGraceTime 120
    #PermitRootLogin without-password
    PermitRootLogin yes
    StrictModes yes
    UsePAM no  
之后重启服务

    root@572bd8f35023:/etc/ssh# service ssh restart
在系统重尝试telnet容器（使用容器的IP地址和22端口）

    [g@gd ~]$ sudo ssh -p 22 172.17.0.3
    [sudo] password for g: 
    root@172.17.0.3's password: 
    Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 3.10.0-123.el7.x86_64 x86_64)

     * Documentation:  https://help.ubuntu.com/
    Last login: Sat Aug  4 13:46:39 2018 from 172.17.0.1
    root@572bd8f35023:~#
使用映射的本地端口访问
    
    [g@gd ~]$ ssh root@127.0.0.1 -p 44
    root@127.0.0.1's password: 
    Last login: Sat Aug  4 13:49:14 2018 from 172.17.0.1
    root@572bd8f35023:~# 
## 例子1
基于例子0,将目前的容器保存为新的容器,并重新启动新容器。先使用exit彻底退出容器。

    root@572bd8f35023:/etc/ssh# exit
    exit
    [g@gd /]$ 
查询容器ID

    [g@gd ~]$ sudo docker ps -l
    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
    572bd8f35023        ubuntu:14.04        "/bin/bash"         50 minutes ago      Exited (0) 15 seconds ago                       dazzling_volhard
使用commit命令提交容器

    [g@gd ~]$ sudo docker commit 572bd8f35023 ubuntu14-ssh
    sha256:a4fb97b2bfaece99ca40bb4cf82f6be288fc2eddf89183623712ddc1bcaeef1e
    [g@gd ~]$ 
此时查看本地镜像，已经有了保存的新镜

    [g@gd ~]$ sudo docker images
    REPOSITORY                  TAG                 IMAGE ID            CREATED              SIZE
    ubuntu14-ssh                latest              a4fb97b2bfae        About a minute ago   251 MB
    docker.io/ubuntu            14.04               971bb384a50a        2 weeks ago          188 MB
    docker.io/ubuntu            13.10               7f020f7bf345        4 years ago          185 MB
此时SSH登录（按例子1中），发现无法登陆，原因为容器中的SSH服务没有启动。进入容器手工启动SSH发现报错：

    root@4dda3c9aa2ec:/usr/sbin# sshd
    sshd re-exec requires execution with an absolute path
此时需要手工执行,并重启sshd服务

    root@4dda3c9aa2ec:/etc/ssh# ssh-keygen -t dsa -f /etc/ssh/ssh_host_dsa_key
    root@4dda3c9aa2ec:/etc/ssh# ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key
    root@4dda3c9aa2ec:/etc/ssh# service ssh restart
    
删除建立容器，重新建立容器并启动SSH服务

    [g@gd ~]$  sudo  docker run -d -p 66:22 ubuntu14-ssh /usr/sbin/sshd -D
查询启动情况

    [g@gd ~]$ sudo docker ps -l
    CONTAINER ID        IMAGE               COMMAND               CREATED             STATUS              PORTS                NAMES
    e2366f3a995f        ubuntu14-ssh        "/usr/sbin/sshd -D"   4 seconds ago       Up 2 seconds        0.0.0.0:66->22/tcp   amazing_agnesi
此时SSH登录（按例子1中）,登录成功ss

