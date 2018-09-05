---
title: Docker笔记-安装配置
date: 2018-09-05 14:31:13
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
    
<!--more-->

### 步骤

#### 1.根据要求安装相应内核的linux系统，本次测试试用了CentOS7,并将yum源更换为阿里源

    [root@gd ~]# uname -a
    Linux gd 3.10.0-123.el7.x86_64 #1 SMP Mon Jun 30 12:09:22 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux

#### 2.通过yum安装docker
    [g@runoob ~]# sudo yum -y install docker-io

#### 3.启动 Docker 后台服务
    [g@gd CentOS 7 x86_64]$ sudo service docker start
    Redirecting to /bin/systemctl start docker.service

#### 4.镜像加速
国内后续拉取 Docker 镜像十分缓慢，我们可以需要配置加速器来解决，我使用的是网易的镜像地址：
    
    http://hub-mirror.c.163.com。

新版的Docker使用etc/docker/daemon.json（Linux）
或者
%programdata%\docker\config\daemon.json（Windows）
来配置 Daemon。

请在该配置文件中加入（没有该文件的话，请先建一个）：
    
    {
      "registry-mirrors": ["http://hub-mirror.c.163.com"]
    }
    
Docker 官方提供的中国

    https://registry.docker-cn.com
    
七牛云加速器

    https://reg-mirror.qiniu.com/
    
重启docker服务

    $ sudo systemctl daemon-reload
    $ sudo systemctl restart docker
配置加速器之后，如果拉取镜像仍然十分缓慢，请手动检查加速器配置是否生效，在命令行执行docker info，如果从结果中看到了如下内容，说明配置成功。

    Registry Mirrors:
    http://hub-mirror.c.163.com

#### 5.运行一个web应用
前面我们运行的容器并没有一些什么特别的用处。接下来让我们尝试使用 docker 构建一个 web 应用程序。我们将在docker容器中运行一个 Python Flask 应用来运行一个web应用。

    [root@gd yum.repos.d]# docker pull training/webapp
    [root@gd yum.repos.d]# docker run -d -P training/webapp python app.py
    
-d:让容器在后台运行。
-P:将容器内部使用的网络端口映射到我们使用的主机上

使用 docker ps 来查看我们正在运行的容器

    [root@gd yum.repos.d]# docker ps
    CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS                     NAMES
    734cf64ec67a        training/webapp     "python app.py"     About a minute ago   Up About a minute   0.0.0.0:32769->5000/tcp   modest_brattain
    [root@gd yum.repos.d]# 
Docker 开放了 5000 端口（默认 Python Flask 端口）映射到主机端口 32769 上。我们也可以通过 -p 参数来设置不一样的端口：
    
    runoob@runoob:~$ docker run -d -p 5000:5000 training/webapp python app.py

#### 6.网络端口的快捷方式
通过docker ps 命令可以查看到容器的端口映射，docker还提供了另一个快捷方式：docker port,使用 docker port 可以查看指定（ID或者名字）容器的某个确定端口映射到宿主机的端口号。
上面我们创建的web应用容器ID为:734cf64ec67a modest_brattain
    
我可以使用docker port 734cf64ec67a 或docker port modest_brattain

        [root@gd yum.repos.d]# docker port 734cf64ec67a
        5000/tcp -> 0.0.0.0:32769
        [root@gd yum.repos.d]# docker port modest_brattain
        5000/tcp -> 0.0.0.0:32769

#### 7.查看WEB应用程序日志
docker logs [ID或者名字] 可以查看容器内部的标准输出。

        [root@gd yum.repos.d]# docker logs modest_brattain 
         * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
        10.23.2.3 - - [03/Aug/2018 10:44:41] "GET / HTTP/1.1" 200 -
        10.23.2.3 - - [03/Aug/2018 10:44:41] "GET /favicon.ico HTTP/1.1" 404 --
        
-f:让 docker logs 像使用 tail -f 一样来输出容器内部的标准输出。

        [root@gd yum.repos.d]# docker logs modest_brattain  -f
         * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
        10.23.2.3 - - [03/Aug/2018 10:44:41] "GET / HTTP/1.1" 200 -
        10.23.2.3 - - [03/Aug/2018 10:44:41] "GET /favicon.ico HTTP/1.1" 404 -

#### 8.查看WEB应用程序容器的进程
我们还可以使用 docker top 来查看容器内部运行的进程

        [root@gd yum.repos.d]# docker top modest_brattain 
        UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
        root                11111               11094               0                   18:42               ?                   00:00:00            python app.py
        [root@gd yum.repos.d]# 

#### 9.检查WEB应用程序
使用 docker inspect 来查看Docker的底层信息。它会返回一个 JSON 文件记录着 Docker 容器的配置和状态信息。

    [root@gd yum.repos.d]# docker inspect modest_brattain 

#### 10.停止WEB应用容器
    [root@gd yum.repos.d]# docker stop modest_brattain 

#### 11.重启WEB应用容器
已经停止的容器，我们可以使用命令 docker start 来启动。
        
    [root@gd yum.repos.d]# docker start modest_brattain

docker ps -l 查询最后一次创建的容器：

    [root@gd yum.repos.d]# docker ps -l
    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                     NAMES
    734cf64ec67a        training/webapp     "python app.py"     13 minutes ago      Up 41 seconds       0.0.0.0:32770->5000/tcp   modest_brattain

#### 12.移除WEB应用容器
我们可以使用 docker rm 命令来删除不需要的容器

    [root@gd yum.repos.d]# docker rm distracted_leavitt 
    distracted_leavitt
    
移除所有退出的容器

    [g@gd ~]$ sudo docker rm $(sudo docker container ls -f "status=exited" -q)

