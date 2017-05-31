---
id: 177
title: 防止暴力破解root密码的一些安全策略
date: 2014-07-09T22:09:54+00:00
author: TY
layout: post
guid: http://t-y.me/?p=177
permalink: /p/177
views:
  - 477
wp_github_commits_page_fields:
  - 'a:3:{s:15:"gc_widget_title";s:0:"";s:11:"github_user";s:0:"";s:11:"github_repo";s:0:"";}'
duoshuo_thread_id:
  - 1163061200838197273
categories:
  - SERVER
tags:
  - DenyHosts
  - linux
  - ssh
  - ubuntu
---
当服务器处在外网环境中时，永远都会面领各种各样安全风险，很多人为了方便给root账号设置一个简单地好记密码，对于这种弱密码其实黑客很容易破解，这样一来大大降低了服务器安全系数。所以千万不要图一时省事，让黑客有机可乘，这里我记录和分享一下在防止暴力破解root密码的一些基本安全设置。

1，修改ssh的默认端口。

2，禁止root账号登陆。

3，新建一个账号(不要用admin之类容易猜到的账号，并设置一个复杂的密码)并让其可以sudo su成root，如果可以，最好也禁止密码登录，只允许通过ssh key方式认证登录。

4，使用DenyHosts防止暴力破解将对方尝试破解的ip拒绝掉。

以上4点，是在初始化一台新的服务器时候起码要做到的。

贴一个日志：


    tail -n 1000 auth.log |grep &#8216;Failed password for root &#8216;
  
    sshd[30294]: Failed password for root from 61.128.110.39 port 45014 ssh2
  
    sshd[30311]: Failed password for root from 61.128.110.39 port 46366 ssh2
  
    sshd[30459]: Failed password for root from 61.128.110.39 port 47481 ssh2

从上面的日志可以看到对方在用root账号尝试不同的ssh端口号，由于我禁止了root账号登陆，所以就算端口号和密码都猜对了也会失败，在失败3次之后 61.128.110.39就自动被防火墙拒绝掉（/etc/hosts.deny ）。