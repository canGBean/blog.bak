---
title: Docker笔记-镜像查看、使用、获取、查找
date: 2018-09-18 17:25:17
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
- 镜像查看、使用、获取、查找

    
<!--more-->


## 1. 列出镜像
当运行容器时，使用的镜像如果在本地中不存在，docker 就会自动从 docker 镜像仓库中下载，默认是从 Docker Hub 公共镜像源下载。使用docker images 来列出本地主机上的镜像。
    
    [g@gd ~]$ sudo docker images
    [sudo] password for g: 
    REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
    docker.io/hello-world       latest              2cb0d9787c4d        3 weeks ago         1.85 kB
    docker.io/training/webapp   latest              6fae60ef3446        3 years ago         349 MB
各个选项说明:

|选项|说明|
|----|----|
|REPOSITORY|表示镜像的仓库源|
|TAG|镜像的标签|
|IMAGE ID|镜像ID|
|CREATED|镜像创建时间|
|SIZE|镜像大小|
## 2. 使用镜像
同一仓库源可以有多个 TAG，代表这个仓库源的不同个版本，如ubuntu仓库源里，有15.10、14.04等多个不同的版本，我们使用 REPOSITORY:TAG 来定义不同的镜像。所以，我们如果要使用版本为15.10的ubuntu系统镜像来运行容器时，命令如下

    [g@gd ~]$ sudo docker run -t -i ubuntu:15.10 /bin/bash
    Unable to find image 'ubuntu:15.10' locally
    Trying to pull repository docker.io/library/ubuntu ... 
    15.10: Pulling from docker.io/library/ubuntu
    7dcf5a444392: Pull complete 
    759aa75f3cee: Pull complete 
    3fa871dc8a2b: Pull complete 
    224c42ae46e7: Pull complete 
    Digest: sha256:cc767eb612212f9f5f06cd1f4e0821d781a5f83bc24d1182128a1088907d3825
    Status: Downloaded newer image for docker.io/ubuntu:15.10
使用：exit或者ctrl+d来退出整个系统(彻底退出容器，状态变为Exited)。
使用：先按，ctrl+p，再按，ctrl+q退出attach的容器（容器状态不改变）

如果你不指定一个镜像的版本标签，例如你只使用 ubuntu，docker 将默认使用 ubuntu:latest 镜像。如果要使用版本为14.04的ubuntu系统镜像来运行容器时，命令如下：

    [g@gd ~]$ sudo docker run -t -i ubuntu:14.04 /bin/bash
    Unable to find image 'ubuntu:14.04' locally
    Trying to pull repository docker.io/library/ubuntu ... 
    14.04: Pulling from docker.io/library/ubuntu
    8284e13a281d: Pull complete 
    26e1916a9297: Pull complete 
    4102fc66d4ab: Pull complete 
    1cf2b01777b2: Pull complete 
    7f7a2d5e04ed: Pull complete 
    Digest: sha256:4851d1986c90c60f3b19009824c417c4a0426e9cf38ecfeb28598457cefe3f56
    Status: Downloaded newer image for docker.io/ubuntu:14.04
    root@9db3cd759e82:/#    
## 3. 获取一个新的镜像
当我们在本地主机上使用一个不存在的镜像时Docker就会自动下载这个镜像。如果我们想预先下载这个镜像，我们可以使用 docker pull 命令来下载它。

    [g@gd ~]$ sudo docker pull ubuntu:13.10
    Trying to pull repository docker.io/library/ubuntu ... 
    13.10: Pulling from docker.io/library/ubuntu
    a3ed95caeb02: Pull complete 
    0d8710fc57fd: Pull complete 
    5037c5cd623d: Pull complete 
    83b53423b49f: Pull complete 
    e9e8bd3b94ab: Pull complete 
    7db00e6b6e5e: Pull complete 
    Digest: sha256:403105e61e2d540187da20d837b6a6e92efc3eb4337da9c04c191fb5e28c44dc
    Status: Downloaded newer image for docker.io/ubuntu:13.10
## 4.查找镜像
从 Docker Hub 网站来搜索镜像，Docker Hub 网址为：https://hub.docker.com/ 也可以使用docker search 命令来搜索镜像。比如我们需要一个httpd的镜像来作为我们的web服务。我们可以通过docker search 命令搜索 httpd 来寻找适合我们的镜像。

    [g@gd ~]$ sudo docker search python
    [g@gd ~]$ sudo docker search httpd
    INDEX       NAME                                         DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
    docker.io   docker.io/python                             Python is an interpreted, interactive, obj...   3104      [OK]       
    docker.io   docker.io/django                             Django is a free web application framework...   706       [OK]       
    docker.io   docker.io/pypy                               PyPy is a fast, compliant alternative impl...   144       [OK]       
    docker.io   docker.io/kaggle/python                      Docker image for Python scripts run on Kaggle   100                  [OK]

|字段|描述|
|----|----|
|NAME|镜像仓库源的名称|
|DESCRIPTION|镜像的描述|
|OFFICIAL|是否docker官方发布|

## 5.拖取镜像（见3）
下载完成后，我们就可以使用这个镜像了

    [g@gd ~]$ sudo docker run -t -i ubuntu:14.04  /bin/bash
    root@473252fbe898:/# 
    CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                       PORTS               NAMES
    473252fbe898        ubuntu:14.04        "/bin/bash"         6 minutes ago       Up 6 minutes                                     stoic_beaver
退出后再进入容器首先用docker ps -a 查找到该CONTAINERID对应编号（比如：0a3309a3b29e）进入该系统docker attach 0a3309a3b29e （此时没反应，ctrl+c就进入到ubuntu系统中去了）

