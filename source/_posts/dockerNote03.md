---
title: Docker笔记-使用Dockerfile构建镜像
date: 2018-09-22 17:54:43
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
- 使用Dockerfile构建镜像


<!--more-->


## 0.构建镜像
使用命令 docker build，从零开始来创建一个新的镜像。先创建一个Dockerfile 文件，其中包含一组指令来告诉Docker 如何构建我们的镜像。

    vim Dockerfile  
    
```SHELL
FROM centos:6.7
MAINTAINER Fisher "123test321"

RUN /bin/echo 'root:123456' | chpasswd
RUN useradd test
RUN /bin/echo 'test:123456' | chpasswd
RUN /bin/echo -e "LANG=\"en_US.UTF-8\"" >/etc/default/local
EXPOSE 22
EXPOSE 80
CMD /usr/sbin/sshd -D
```
每一个指令都会在镜像上创建一个新的层，每一个指令的前缀都必须是大写的。
第一条FROM，指定使用哪个镜像源
RUN 指令告诉docker 在镜像内执行命令，安装了什么。
然后，我们使用 Dockerfile 文件，通过 docker build 命令来构建一个镜像。

### 先拉去一个centos镜像

    [g@gd dockernote]$ sudo docker pull centos:6.7
    Trying to pull repository docker.io/library/centos ... 
    6.7: Pulling from docker.io/library/centos
    cbddbc0189a0: Pull complete 
    Digest: sha256:9587f36d043e0ab9ad0f4c17e3022c360d57b3b970299b3bf7ddd81f0248a60e
    Status: Downloaded newer image for docker.io/centos:6.7
    
    [g@gd dockernote]$ sudo docker images
    REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
    docker.io/centos            6.7                 f2e2f7b8308b        6 weeks ago         191 MB
    

### 现在根据Dockerfile文件进行构建

    [g@gd dockernote]$ sudo docker build -t test/centos:6.7 .
    Sending build context to Docker daemon 2.048 kB
    Step 1/9 : FROM centos:6.7
     ---> f2e2f7b8308b
    Step 2/9 : MAINTAINER Fisher "123test321"
     ---> Running in 653b4e8dd6fe
     ---> e0f5b9d7da93
    Removing intermediate container 653b4e8dd6fe
    Step 3/9 : RUN /bin/echo 'root:123456' | chpasswd
     ---> Running in 741d8f79bbe2

     ---> 711778d5ef16
    Removing intermediate container 741d8f79bbe2
    Step 4/9 : RUN useradd test
     ---> Running in f7c6041daad8

     ---> ccc50ae9098f
    Removing intermediate container f7c6041daad8
    Step 5/9 : RUN /bin/echo 'test:123456' | chpasswd
     ---> Running in 78728cacc451

     ---> d64fba43a5bd
    Removing intermediate container 78728cacc451
    Step 6/9 : RUN /bin/echo -e "LANG=\"en_US.UTF-8\"" >/etc/default/local
     ---> Running in 8684334383c5

     ---> 2fa584ad84a6
    Removing intermediate container 8684334383c5
    Step 7/9 : EXPOSE 22
     ---> Running in 6af3694a78a6
     ---> 783787fe5e2f
    Removing intermediate container 6af3694a78a6
    Step 8/9 : EXPOSE 80
     ---> Running in dfe88a5b0002
     ---> ce1afb89d5f4
    Removing intermediate container dfe88a5b0002
    Step 9/9 : CMD /usr/sbin/sshd -D
     ---> Running in 0f4f3ad0a2db
     ---> c792523d8886
    Removing intermediate container 0f4f3ad0a2db
    Successfully built c792523d8886

参数说明
- -t ：指定要创建的目标镜像名
- .  ：Dockerfile 文件所在目录，可以指定Dockerfile 的绝对路径

### 查看构建完成的镜像

    [g@gd dockernote]$ sudo docker images
    REPOSITORY                  TAG                 IMAGE ID            CREATED              SIZE
    test/centos                 6.7                 c792523d8886        About a minute ago   191 MB
    docker.io/centos            6.7                 f2e2f7b8308b        6 weeks ago          191 MB


### 使用新的镜像来创建容器
    
    [g@gd dockernote]$ sudo docker run -it  test/centos:6.7  /bin/bash
    [root@391854bafeae /]# id test
    uid=500(test) gid=500(test) groups=500(test)
    [root@391854bafeae /]# 

## 1.设置镜像标签
使用 docker tag 命令，为镜像添加一个新的标签。此时镜像多一个标签。

    [g@gd ~]$ sudo docker tag c792523d8886 test/centos:test
    [g@gd ~]$ sudo docker images
    REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
    test/centos                 6.7                 c792523d8886        About an hour ago   191 MB
    test/centos                 test                c792523d8886        About an hour ago   191 MB
    docker.io/centos            6.7                 f2e2f7b8308b        6 weeks ago         191 MB
