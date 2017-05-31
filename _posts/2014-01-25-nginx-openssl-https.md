---
id: 124
title: nginx配置https
date: 2014-01-25T13:05:59+00:00
author: TY
layout: post
guid: http://t-y.me/?p=124
permalink: /p/124
wp_github_commits_page_fields:
  - 'a:3:{s:15:"gc_widget_title";s:0:"";s:11:"github_user";s:0:"";s:11:"github_repo";s:0:"";}'
views:
  - 848
duoshuo_thread_id:
  - 1163061200838197258
categories:
  - WEB
tags:
  - https
  - nginx
  - openssl
---
前段时间在做一个我自己的私人项目（https://todoo.us)，考虑到安全性，需要用到https，于是了解了一下这方面的信息，这里写篇文章记录一下。

要服务器支持https，首先需要获取一个ssl证书，获取证书有两种方式：一是花钱购买，而是申请免费的证书。因为是私人项目，所以这里我用了全球唯一免费颁发ssl证书网站：http://startssl.com的免费ssl证书。这篇文章里不会讲到关于如何获取证书，稍微能看懂一点英文，按照网站的引导和提示照着做就可以了。不出意外的话，最后会得到两个文件：ssl.key,ssl.crt

1，下载startssl提供的两个pem文件备用：
  
    wget http://www.startssl.com/certs/ca.pem 
    wget http://www.startssl.com/certs/sub.class1.server.ca.pem

_2，通过得到的ssl.crt以及startssl提供的另外两个文件生成一个标准的crt文件：

  
    cat ssl.crt sub.class1.server.ca.pem ca.pem > /etc/nginx/conf/ssl-unified.crt

3，生成免输入密码的key文件
  
    openssl rsa -in ssl.key -out ssl.key.unsecure

4，上传这两个文件(ssl-unified.crt,ssl.key.unsecure)到nginx目录下
  
给nginx添加配置开启Https支持，因此特别注意，需要新建一个服务器监听443端口，而不是直接写在80端口的服务器里，如下：

    server {
  
		listen   443;
  
		server_name todoo.us,www.todoo.us;
		ssl on;
		ssl_certificate  /usr/local/nginx/conf/ssl/ssl-unified.crt;
		ssl_certificate\_key  /usr/local/nginx/conf/ssl/ssl.key.unsecure;


	    location / {

			proxy_read_timeout 120;
			proxy_connect_timeout 120;
			proxy_set_header X-Forwarded-For$proxy_add_x_forwarded_for;

			proxy\_set\_header Host $http_host;

			proxy_redirect off;
			proxy_pass http://localhost:8006;
  
  		}
  	}

重启nginx : 

    nginx -s reload

当然还需要将用户的Http请求重定向到https

    rewrite ^ https://todoo.us$request_uri? permanent;

最后的配置文件如下：
server {
  
     listen   80;
  
     server_name todoo.us www.todoo.us;
  
     access_log /var/log/nginx/wolverine_access.log ;
  
     error_log  /var/log/nginx/wolverine_error.log ;
  
     rewrite ^ https://todoo.us$request_uri?permanent;
     
}
     
server {
  
     listen   443;
  
     server_name todoo.us www.todoo.us   ;
     
     ssl on;
  
     ssl_certificate  /usr/local/nginx/conf/ssl/ssl.crt.uniffied;
  
     ssl_certificate_key  /usr/local/nginx/conf/ssl/ssl.key.unsecure;
     
     location / {
  
         proxy_read_timeout 120;
      
         proxy_connect_timeout 120;
      
         proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      
         proxy_set_header Host $http_host;
      
         proxy_redirect off;
      
         proxy_pass   http://localhost:8007  ;
  
     }
     
}

用gunicron将django启动并绑定到8007端口：
  
    /root/.virtualenvs/wolverine/bin/gunicorn -k gevent wsgi:application -b localhost:8007 -w 2
  
    stdout_logfile =  /tmp/wolverine.log

nginx -t 测试没问题就可以重启了，这个时候应该可以看到浏览器中的url变成https并且不会被浏览器认为是不安全的