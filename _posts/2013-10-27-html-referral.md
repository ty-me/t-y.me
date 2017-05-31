---
id: 118
title: html中清除链接跳转时带referer的办法
date: 2013-10-27T14:30:47+00:00
author: TY
layout: post
guid: http://t-y.me/?p=118
permalink: /p/118
views:
  - 978
wp_github_commits_page_fields:
  - 'a:3:{s:15:"gc_widget_title";s:0:"";s:11:"github_user";s:0:"";s:11:"github_repo";s:0:"";}'
duoshuo_thread_id:
  - 1163061200838197257
categories:
  - WEB
tags:
  - referer
---
<p style="text-align: left;">
  最近在公司的项目中有一个需求是点击链接跳转到淘宝的某个产品页面，想想很简单，不就是放一个a便签嘛。但后来证明我还是图样了。当我把代码部署到我们在外网的测试服务器上时，点击链接，淘宝给出了这样一个错误页面：<a href="http://t-y.me/wp-content/uploads/2013/12/QQ20131227-1.png"><img class="size-full wp-image-119 aligncenter" alt="QQ20131227-1" src="http://t-y.me/wp-content/uploads/2013/12/QQ20131227-1.png" width="699" height="443" srcset="https://t-y.me/wp-content/uploads/2013/12/QQ20131227-1.png 699w, https://t-y.me/wp-content/uploads/2013/12/QQ20131227-1-650x411.png 650w, https://t-y.me/wp-content/uploads/2013/12/QQ20131227-1-624x395.png 624w" sizes="(max-width: 699px) 100vw, 699px" /></a>
</p>

<p style="text-align: left;">
  但是当我直接复制链接地址粘贴到地址栏时，不会出现这个错误。并且在我本地的开发环境下跳转过去也不会出现这个错误。
</p>

<p style="text-align: left;">
      简单分析一下，我认为是淘宝对referer做了限制，只允许它白名单里的网站跳转到淘宝。为了验证这个猜想，我打算做一个小实验：在外网环境下，分别做两次不同的跳转，一次带referer，一次不带。期望出现的结果是：带referer的那次跳转会出现上图所示错误；不带referer的那次跳转，能正确返回产品页面。
</p>

<p style="text-align: left;">
      为了完成这个试验，需要做一件事情，就是怎样让浏览器不向淘宝发送referer信息。经过在stackoverflow搜索，找到了<a href="http://referrer-killer.googlecode.com/git/example.html" target="_blank">这个</a>，专门用来解决阻止浏览器发送referer信息的问题。我通过ReferreKiller.js完成试验，结果符合预期，猜想成立。
</p>

<p style="text-align: left;">
      下面讲讲ReferreKiller.js的原理。首先作者它在跳转链接上添加rel=&#8221;noreferrer&#8221;属性，这个属性的作用就是告诉浏览器你不要发送referer信息给服务器，但是它在万恶的IE中是不起作用的，怎么解决这个问题呢？作者在a标签外面多套了一层iframe，因为某些版本的IE有一个BUG，就是点击在iframe中的链接不会发送referer信息，作者正好利用了这个BUG。
</p>