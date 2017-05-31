---
id: 281
title: django forms三级联动下拉选框示例
date: 2015-05-05T16:22:39+00:00
author: TY
layout: post
guid: http://t-y.me/?p=281
permalink: /p/281
views:
  - 501
duoshuo_thread_id:
  - 1163061200838197280
categories:
  - DJANGO
tags:
  - ajax
  - django
  - forms
  - select
  - 下拉
  - 联动
---
在某个django的项目中需要实现省份-城市-地区的三级联动下拉选框功能，而django的forms不自带类似支持，所以写了这篇博客将方法分享出来，以便有需要的开发者可以少走弯路。 

这里主要是借助了jquery的插件cxselect，效果如下： 

![](http://tyblog.qiniudn.com/2015/05/c.gif)



具体的使用方法这里我就不用多写了，因为比较简单，唯一需要注意的是在forms中需要对字段添加自定义class，具体用法请参考cxselect的文档


示例代码下载：[download](http://tyblog.qiniudn.com/2015/05select_in_ajax_mode_demo.tar.gz) 



PS:上面的gif屏幕录像是在ubuntu下使用<a href="https://github.com/GNOME/byzanz" target="_blank">byzanz</a>生成的 

<pre class="prettyprint lang-bsh"># 6秒钟后开始录像，从左上角开始截取宽400px，高600px的区域，生成c.gif文件
byzanz-record -d 10 --delay 6  -c -x 0 -y 0 -w 400 -h 600 c.gif</pre>