---
id: 168
title: 使用logrotate切割日志文件
date: 2014-05-07T14:03:27+00:00
author: TY
layout: post
guid: http://t-y.me/?p=168
permalink: /p/168
wp_github_commits_page_fields:
  - 'a:3:{s:15:"gc_widget_title";s:0:"";s:11:"github_user";s:0:"";s:11:"github_repo";s:0:"";}'
views:
  - 417
duoshuo_thread_id:
  - 1163061200838197272
categories:
  - SERVER
tags:
  - nginx log logrotate
---
最近了解到可以用logrotate来统一切割系统的各种日志文件，这里以nginx的日志为例简单记录一下配置方法。

1，首先需要安装logrotate

    apt-get install -y 

2，配置nginx的日志切割规则

    vi /etc/logrotate.d/nginx

  
    /var/log/nginx/*.log {
	  
    daily
	  
    missingok
	  
    rotate 52
	  
    compress
	  
    delaycompress
	  
    notifempty
	  
    create 0640 www-data adm
	  
    sharedscripts
	  
    prerotate
		  
    if [ -d /etc/logrotate.d/httpd-prerotate ]; then \
			  
    run-parts /etc/logrotate.d/httpd-prerotate; \
		  
    fi; \
	  
    endscript
	  
    postrotate
		  
    [ ! -f /var/run/nginx.pid ] || kill -USR1 \`cat /var/run/nginx.pid\`
	  
    endscript
  
    }
  
