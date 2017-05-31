---
id: 195
title: 在nginx中设置对指定的user agent返回403
date: 2014-09-04T15:28:13+00:00
author: TY
layout: post
guid: http://t-y.me/?p=195
permalink: /p/195
views:
  - 321
wp_github_commits_page_fields:
  - 'a:3:{s:15:"gc_widget_title";s:0:"";s:11:"github_user";s:0:"";s:11:"github_repo";s:0:"";}'
duoshuo_thread_id:
  - 1163061200838197275
categories:
  - SERVER
tags:
  - 403
  - nginx
  - user-agent
---
最近我的某个网站上经常会遭遇无良爬虫（AhrefsBot/5.0; +http://ahrefs.com/robot/）的频繁光顾，导致网站负载过高。这货完全不遵循robots协议，一怒之下只好在nginx中将此爬虫的请求拒绝掉。
  
设置很简单，如下（顺便将Wget也拒绝掉）： 



<pre class="prettyprint lang-bsh">if ($http_user_agent ~ (AhrefsBot|Wget)) {
    return 403;
 }</pre>