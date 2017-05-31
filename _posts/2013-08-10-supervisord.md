---
id: 15
title: 用supervisord管理你的进程
date: 2013-08-10T18:39:51+00:00
author: TY
layout: post
guid: http://t-y.me/?p=15
permalink: /p/15
duoshuo_thread_id:
  - 1163061200838197251
views:
  - 1725
wp_github_commits_page_fields:
  - 'a:3:{s:15:"gc_widget_title";s:0:"";s:11:"github_user";s:0:"";s:11:"github_repo";s:0:"";}'
categories:
  - PYTHON
tags:
  - linux
  - PYTHON
  - supervisor
---
重开博客，第一篇技术类的文章用来记录<a href="http://supervisord.org/" target="_blank">supervisord</a>的使用。

## supervisord 是什么？
  


它是一个由python实现的进程管理工具，借助supervisord，你可以更加容易得管理和监控服务器进程，它可以将一个普通进程转换成守护进程，当被Kill掉或者意外终止时，能自动重新启动。它还提供一个web管理界面，你可以通过浏览器、手机进行管理

<p style="text-align:center;">
  <a href="http://t-y.me/wp-content/uploads/2013/08/QQ20130810-8.png"><img class=" wp-image-22 aligncenter" alt="supervisord" src="http://t-y.me/wp-content/uploads/2013/08/QQ20130810-8-650x358.png" width="650" height="358" srcset="https://t-y.me/wp-content/uploads/2013/08/QQ20130810-8-650x358.png 650w, https://t-y.me/wp-content/uploads/2013/08/QQ20130810-8-624x344.png 624w, https://t-y.me/wp-content/uploads/2013/08/QQ20130810-8.png 917w" sizes="(max-width: 650px) 100vw, 650px" /></a>
</p>

<h2 style="text-align:left;">
  下载安装<br />
</h2>



<pre class="prettyprint lang-bsh">easy_install supervisor</pre>



<h2 style="text-align:left;">
  创建配置文件<br />
</h2>

vim /etc/supervisord.conf &nbsp;输入以下内容： 



<pre class="prettyprint lang-bsh">[unix_http_server]
file=/tmp/supervisor_monitor.sock

[supervisord]
pidfile=/tmp/supervisord_monitor.pid
logfile_backups=1

[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///tmp/supervisor_monitor.sock

[inet_http_server]
port=127.0.0.1:9001
username = admin
password = 123456</pre>



&nbsp;这些是supervisord的基本配置，没什么好说的，复制粘贴就ok。其中inet\_http\_server这一项是配置web管理界面的。
  
接下来是添加你需要管理的进程，同样在supervisord.conf后面追加内容如下： 



<pre class="prettyprint lang-bsh">[program:foo]
directory = /srv/foo/
environment = PYTHONPATH=/srv/foo/
command = /root/.virtualenvs/fooo/bin/python scheduler.py</pre>



其中foo是程序的名字，directory指定目录，enviroment设置执行程序所需的环境变量，command为启动程序所执行的命令。还有更多详细的配置选项和说明，请查阅这里的<a href="http://supervisord.org/configuration.html#program-x-section-settings" target="_blank">文档</a> 

## 启动 supervisord
  




<pre class="prettyprint lang-bsh">supervisord -c /etc/supervisord.conf -l /var/log/supervisord.log</pre>



如果一切顺利，ps -ef |grep foo应该能看到已经有一个进程在运行中了。你可以尝试kill -9 \`pgrep -f foo\`来杀死这个经常，稍等片刻supervisord会自动将它重启。 

## 其它命令
  


停止运行某个程序: supervisorctl stop foo 

重启某个程序： supervisorctl restart foo