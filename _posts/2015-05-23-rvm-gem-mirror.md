---
id: 328
title: 使用rvm/gem的中国镜像加速
date: 2015-05-23T14:37:44+00:00
author: TY
layout: post
guid: http://t-y.me/?p=328
permalink: /p/328
views:
  - 420
duoshuo_thread_id:
  - 1163061200838197285
categories:
  - ROR
tags:
  - gem
  - mirrors
  - ruby
  - rvm
  - taobao
---
在国内使用淘宝的ruby镜像用来加速rvm/gem，淘宝镜像地址：http://ruby.taobao.org/

### rvm的设置方法

修改 RVM ，改用淘宝的镜像源作为下载源

    mac osx
    $ sed -i .bak 's!cache.ruby-lang.org/pub/ruby!ruby.taobao.org/mirrors/ruby!' $rvm_path/config/db
    linux
    $ sed -i 's!cache.ruby-lang.org/pub/ruby!ruby.taobao.org/mirrors/ruby!' $rvm_path/config/db
    

### gem的设置方法

    - $ gem sources --remove https://rubygems.org/
    - $ gem sources -a http://ruby.taobao.org/
    - $ gem sources -l
    - *** CURRENT SOURCES ***
    - http://ruby.taobao.org
    - # 请确保只有 ruby.taobao.org
    - $ gem install rails
    

_Enjoy it !_