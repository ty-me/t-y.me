---
id: 201
title: 在Mac上体验Docker
date: 2014-11-13T13:44:46+00:00
author: TY
layout: post
guid: http://t-y.me/?p=201
permalink: /p/201
views:
  - 1610
duoshuo_thread_id:
  - 1163061200838197276
categories:
  - SERVER
tags:
  - boot2docker
  - docker
  - mac
  - osx
  - ubuntu
---
![](http://tyblog.qiniudn.com/2014/09/osx-docker1.jpg)

Docker实在是风风光光的火了一把，我建议感兴趣得程序员们尤其是后端程序员们都来体验一下，能瞬间让人幸福感飙升，对码农的人生重新充满希望，套用一句老话来概括就是：人生苦短，我用Docker

本文简单记录一下我在mac os x上安装和体验docker的过程


# 一，软、硬件环境。 

我的机器是mbp是11年末的13 inch版，内存：10GB，CPU：Intel Core i5 2.4GHz，OS X ：10.9.3 ，已经安装好brew。

# 二，安装VirtualBox 

因为docker的daemon只支持在Linux系统下运行，因此需要在Mac安装一个Linux的虚拟机，并且在该虚拟机上运行docker的daemon，再在mac上通过docker的client连接docker主机。

而喜欢造轮子的朋友们已经帮我们做好了一个virtualbox的iso镜像，并提供一个mac上的工具来管理它，这就是传说中的boot2docker，因此需要先安装virtualbox。

从官网下载os x的安装包：（链接：http://download.virtualbox.org/virtualbox/4.3.16/VirtualBox-4.3.16-95972-OSX.dmg ）并安装即可。 

注意：如果你的虚拟机里已经有安装过其它发行版的Linux，也可以跳过安装boot2docker这几步，只需在虚拟机中安装docker并运行docker -d，再在mac上设置环境变量_DOCKER_HOST=tcp://xxx.xxx.xxx:xxx_ 即可 


# 三，安装boot2docker。 


执行初始化，这一步会去下载一个用于virtualbox的iso文件来创建一个虚拟机
网络状况不好的情况下时间会比较长，请耐心等待

    boot2docker init

启动

    boot2docker up
  
 
完成之后会提示：

    To connect the Docker client to the Docker daemon, please set:export DOCKER_HOST=tcp://192.168.59.103:2375

为了方便可以将<b>export DOCKER_HOST=tcp://192.168.59.103:2375</b>加入到~/.bash_profile中，如果用的是bash的话。boot2docker提供了很多命令来控制boot2docker-vm虚拟机，常用的命令可以通过boot2docker help查看，比如restart/init/up/down/info/status/ssh 等等。

    
 注意：以后需要先启动boot2docker的虚拟机，并运行docker的deamon，才能在mac上连接到docker，否则会提示：
     Cannot connect to the Docker daemon
  
# 四，开始体验

到此为止完整过程就算完成了，下面一起来体验一下吧。

1，列出本地镜像
    
    docker images

因为还没有pull任何镜像所以显示结果为空

     REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
&lt;none&gt;              &lt;none&gt;              511136ea3c5a        15 months ago       0 B    

2，搜索镜像
 
    
    docker search ubuntu

3, 启动一个容器

这里我以hello-world为例，
  
    foo@~: docker -t -i run ubuntu:12.04 /bin/bash
    
运行后会先检测本地是否有名字为ubuntu，tag为12.04的image，没有的话会自动pull一个到本地，下载的时间比较长，请忍一忍。如果本地以及有这个image的 话则会将它启动，并进入到终端：
 
     root@c19c68d262a2:/# 
     
现在，你可以在这个容器中针对具体的一个或者多个项目做一些环境配置，然后将它提交的私有仓库中，这样一来，企业中的其它同事就只需在docker中pull你的image，就可以很方便的将项目运行起来，真的就像docker官网首页那句话所说的一样：

   *Build, Ship, and Run Any App, Anywhere*
    

 4，关于docke的更多详细的用法和介绍，请参考本文后面给出的相关链接


# 五，参考文章：
 
 * http://yeasy.gitbooks.io/docker_practice/content/index.htmlDocker —— 从入门到实践 
    
 * http://docs.docker.com/installation/mac/ Installing Docker on Mac OS X
    