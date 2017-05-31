---
id: 233
title: 豆瓣验证码识别（一）
date: 2015-03-03T11:51:18+00:00
author: TY
layout: post
guid: http://t-y.me/?p=233
permalink: /p/233
views:
  - 1078
duoshuo_thread_id:
  - 1163061200838197277
categories:
  - PYTHON
tags:
  - ocr
  - 识别
  - 豆瓣
  - 验证码
---
<div>
  &nbsp; &nbsp; 在几年前写过一个豆瓣自动回帖的机器人程序，因为没有处理验证码，所以回帖速度和效率很差。最近正好春节放假在家比较闲，计划用给机器人开发一个验证码识别功能，来提高回帖的数量，同时用博客的形式记录下开发过程。
</div>

<div>
</div>

&nbsp; 在开始破解工作之前，需要先下载大量的验证码图片来观察，寻找规律，看准方向想好思路之后再开始撸起袖子写代码，比较不容易走弯路。 


![](http://tyblog.qiniudn.com/2015/02/20150214221836_33309.jpg) 

经过观察我发现，豆瓣的验证码初看上去比较高大上，总结起来这样有几个特点： 



  * 背景图案看上去比较复杂，但是大部分图片的背景颜色与字体色差较大 
  * 图案中有比较均匀的白色和黑色噪点 
  * 只有英文单词，没有数字、中文、符号等 
  * 字体为黑色，有稍微的旋转和扭曲 
  * 字体无遮盖、粘黏 
  * 每个字母周围都有白色描边 



&nbsp; &nbsp;先随便选一张验证码图片用photo&nbsp;shop打开后放大，用吸管工具吸取字体上的像素点可以看到，字体主体的黑色部分rgb值可大概在20以内。 

![](http://tyblog.qiniudn.com/2015/03/ps.png)


![](http://tyblog.qiniudn.com/2015/03/20150214221608_66526.png) 

因此这里可以简单得以20为阈值，将rgb值都小于该阈值的点设置为黑色，大于的设置为白色。 





<pre class="prettyprint lang-py">def pre(self):
        width, height = self.img.size
        threshold = 21
        for i in range(0, width):
            for j in range(0, height):
                p = self.frame[i, j]
                r, g, b = p
                if r &gt; threshold or g &gt; threshold or b &gt; threshold:
                    self.frame[i, j] = WHITE
                else:
                    self.frame[i, j ] = BLACK</pre>







二值化后得到的图片如下： 

![](http://tyblog.qiniudn.com/2015/03/20150214222038_48284.jpg) 

背景已经被去掉了，接下来是处理噪点。 

&nbsp; &nbsp; 常用的去噪方法有：均值滤波、中值滤波、高斯滤波以及BM3D等等。通过实验，我发现用中值滤波可以达到比较满意的效果，如图： 


![](http://tyblog.qiniudn.com/2015/02/1.jpg) 

选择不同的滤波窗口将会得到不同的去噪效果，“十”字窗口可以保留较多的细节，但是没法处理2pix\*2pix大小的噪点，因为用十字窗口对2\*2像素块上的每一个像素取中值，得到的都是黑色，因此这里我选择了3*3窗口，代码如下： 



<pre class="prettyprint lang-py">def remove_noise(self, window=1):
        """ 中值滤波移除噪点
        """
        if window == 1:
            # 十字窗口
            window_x = [1, 0, 0, -1, 0]
            window_y = [0, 1, 0, 0, -1]
        elif window == 2:
            # 3*3矩形窗口
            window_x = [-1,  0,  1, -1, 0, 1, 1, -1, 0]
            window_y = [-1, -1, -1,  1, 1, 1, 0,  0, 0]

        width, height = self.img.size
        for i in xrange(width):
            for j in xrange(height):
                box = []
                black_count, white_count =  0, 0
                for k in xrange(len(window_x)):
                    d_x = i + window_x[k]
                    d_y = j + window_y[k]
                    try:
                        d_point = self.frame[d_x, d_y]
                        if d_point == BLACK:
                            box.append(1)
                        else:
                            box.append(0)
                    except IndexError:
                        self.frame[i, j] =  WHITE
                        continue

                box.sort()
                if len(box) == len(window_x):
                    mid = box[len(box)/2]
                    if mid == 1:
                        self.frame[i, j] =  BLACK
                    else:
                        self.frame[i, j] =  WHITE</pre>

到这一步，噪点已经处理完成了，接下来就是切分单个字符，我将会在下一篇博客中详细介绍。





参考资料： 

* [常见验证码的弱点](http://drops.wooyun.org/tips/141) 

* [中值滤波](http://zh.wikipedia.org/zh-cn/%E4%B8%AD%E5%80%BC%E6%BB%A4%E6%B3%A2%E5%99%A8)