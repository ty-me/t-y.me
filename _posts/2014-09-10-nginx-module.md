---
id: 182
title: 2个nginx常用插件(module)推荐
date: 2014-09-09T20:46:05+00:00
author: TY
layout: post
guid: http://t-y.me/?p=182
permalink: /p/182
wp_github_commits_page_fields:
  - 'a:3:{s:15:"gc_widget_title";s:0:"";s:11:"github_user";s:0:"";s:11:"github_repo";s:0:"";}'
views:
  - 562
duoshuo_thread_id:
  - 1163061200838197274
categories:
  - SERVER
  - WEB
tags:
  - cocat
  - module
  - mod_strip
  - nginx
---
在生产环境的web项目中，通常需要对页面做一些优化来加快资源的加载速度，常见得方法有：合并多个js/css文件为一个、移除html代码中的空白字符、缓存静态资源、开启gzip等等。在本文中主要介绍两个个Nginx插件来实现合并js/css文件以及移除html中空白字符。
  
一，合并js/css
  
好在阿里巴巴的工程师们已经开发了concat 这个插件，专门用来解决这个问题。通过src=&#8221;??foo.js,bar.js&#8221;这种写法，可以将foo.js和bar.js的内容合并之后返回，concat已经应用在阿里巴巴很多web项目中，因此无论是性能还是稳定性，都是值得信奈的。查看淘宝网页源码可以看到这种用法如图：

![](http://tyblog.qiniudn.com/2014/08/A2C0CC91-04B6-45EB-8E6B-2B392CD4BD3E.jpg)  

**安装与配置concat**：
如果是新安装Nginx的话，配置concat的过程比较简单，只需编译时加上参数 &#8211;add-module=path\_to\_concat即可，本文中我假设是在为已经安装过的nginx配置该插件，因为实际工作中通常也是这样（记得将你需要操作的机器暂时从负载均衡节点中移除，安装测试成功后再加入进来）
  
1，先查看当前nginx的版本已经安装信息：命令：nginx -V输出如： 


    nginx version: nginx/1.2.3TLS SNI support enabledconfigure arguments: 
    --prefix=/usr/local/Cellar/nginx/1.2.3 
    --with-http_ssl_module 
    --with-pcre --with-ipv6 
    --with-cc-opt=-I/usr/local/include 
    --with-ld-opt=-L/usr/local/lib 
    --conf-path=/usr/local/etc/nginx/nginx.conf 
    --pid-path=/usr/local/var/run/nginx.pid 
    --lock-path=/usr/local/var/nginx/nginx.lock



显示当前nginx版本为1.2.3，配置参数中包含了一些module，再下面重新configure的时候需要带上
  
2，下载对应版本的nginx源码并解压：

    cd /src 
    wget http://nginx.org/download/nginx-1.2.3.tar.gz 
    tar -xvf nginx-1.2.3.tar
    
下载concat源码： 

    cd /srv/nginx-1.2.3/  
    git clone https://github.com/alibaba/nginx-http-concat.git


3，重新配置和编译nginx使得nginx支持concat模块:命令： 


    cd /src/nginx-1.2.3 ./configure  
    --prefix=/usr/local/Cellar/nginx/1.2.3 
    --with-http_ssl_module 
    --with-pcre --with-ipv6 
    --with-cc-opt=-I/usr/local/include 
    --with-ld-opt=-L/usr/local/lib 
    --conf-path=/usr/local/etc/nginx/nginx.conf 
    --pid-path=/usr/local/var/run/nginx.pid 
    --lock-path=/usr/local/var/nginx/nginx.lock 
    --with-module=http_concat_modulemakemake



**注意**：是make，不是make install
  
4，替换旧的nginx文件。如果前面的步骤没有出错的话，到这里新make出来的nginx二进制文件中已经包含了concat模块，现在需要替换掉久的nginx二进制文件，替换之前先将久的文件备份
    
    cp /usr/local/sbin/nginx /usr/local/sbin/nginx.bac
    
停止当前nginx服务：
    
    /usr/local/sbin/nginx -s stop

替换久的文件：

    cp ./objs/nginx /usr/local/sbin/nginx
    
重新启动

    nginx/usr/local/sbin/nginx5
    
到这里安装已经完成，接下来只需开启concat功能即可，比如： 

<pre class="prettyprint lang-bsh">server { 
    listen:80; 
    concat on; 
    root /data/html;
}</pre>



（注：我这里为了方便只只给出了基本的配置项）
  
完成重启nginx:

    nginx -t reload; 


二，移除html中的空白字符：**mod_strip** 1，安装过程跟前面类似，这里我不再重复。
  
2，配置也很简单： 


server {
    listen 80;
    concat on;
    root /data/html;
    strip on;
}


完成后reload一下nginx即可，生效后你讲看到这样的页面：
![](http://tyblog.qiniudn.com/2014/09/8742DCAD-62F8-412C-916D-5A59697EF8E3.jpg)