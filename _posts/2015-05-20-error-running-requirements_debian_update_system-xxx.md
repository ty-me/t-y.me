---
id: 322
title: Error running requirements_debian_update_system xxx
date: 2015-05-20T12:18:22+00:00
author: TY
layout: post
guid: http://t-y.me/?p=322
permalink: /p/322
views:
  - 447
duoshuo_thread_id:
  - 1163061200838197284
categories:
  - SERVER
  - WEB
tags:
  - ruby
  - rvm
  - ubuntu
---
在_Ubuntu_ 12.04上用rvm安装_ruby_ 2.2.2的时候报了这个错误：
  
![](http://tyblog.qiniudn.com/2015/05/Screenshot-from-2015-05-20-115731.png)

    Searching for binary rubies, this might take some time.
    No binary rubies available for: ubuntu/12.04/x86_64/ruby-2.2.2.
    Continuing with compilation. Please read 'rvm help mount' to get more information on binary rubies.
    Checking requirements for ubuntu.
    Installing requirements for ubuntu.
    Updating system
    ........................
    Error running 'requirements_debian_update_system ruby-2.2.2',
    showing last 15 lines of /home/tianyu/.rvm/log/1432093936_ruby-2.2.2/update_system.log
    ++ case "${TERM:-dumb}" in
    ++ case "$1" in
    ++ [[ -t 2 ]]
    ++ return 1
    ++ printf %b 'There has been error while updating '\''apt-get'\'', please give it some time and try again later.
    404 errors should be fixed for rvm to proceed. Check your sources configured in:
        /etc/apt/sources.list
        /etc/apt/sources.list.d/*.list
    \n'
    There has been error while updating 'apt-get', please give it some time and try again later.
    404 errors should be fixed for rvm to proceed. Check your sources configured in:
        /etc/apt/sources.list
        /etc/apt/sources.list.d/*.list
    ++ return 100
    Requirements installation failed with status: 100.
    

根据提示不难发现是因为在`sudo apt-get update`的时候几个资源404了，所以rvm install失败。
  
解决办法是根据`sudo apt-get update`的warning提示，将`/etc/apt/sources.list.d/`下404的资源链接移除即可。
  
记录一下，给遇到过这个问题的同学们参考。