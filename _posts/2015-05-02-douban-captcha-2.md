---
id: 247
title: 豆瓣验证码识别（二）
date: 2015-05-01T18:43:29+00:00
author: TY
layout: post
guid: http://t-y.me/?p=247
permalink: /p/247
views:
  - 1069
duoshuo_thread_id:
  - 1163061200838197278
categories:
  - PYTHON
  - WEB
tags:
  - ocr
  - opencv
  - pil
  - 识别
  - 验证码
---
分隔字符这一步在整个识别过程中也是非常关键的一步，能否将字符合理的分隔开来，将直接影响到能否准确识别。因为豆瓣的验证码字符间没有粘连、重叠的情况，所以这里用到的分隔方法相对简单：在竖直方向上扫描，根据像素点的变化情况可以很容易找到字符的边界，再根据这些边界将字符切割开即可，如图： 

![](http://tyblog.qiniudn.com/2015/03/s.jpg)


如果遇上像淘宝这种字符见有粘连重贴的情况就会麻烦一些： 

![](http://tyblog.qiniudn.com/2015/05/get_img.jpeg)

但还是有办法的，下图中褐色部分代表该位置的竖直方向上字符像素数变化情况，从波谷的地方切分即可，但是需要控制一些每个字符的间距，只有当两个波谷的间距大于一定的值才能分隔，否则会出现下图中u字母被分成两个l的情况。


![](http://tyblog.qiniudn.com/2015/05/tb2.jpg)


<pre>
def split(img):
    """  用竖线扫描分隔文字
    """

    frame = img.load()
    img_new = img.copy()
    frame_new = img_new.load()

    width, heigth = img.size
    line_status = None
    pos_x = []
    for x in xrange(width):
        pixs = []
        for y in xrange(heigth):
            pixs.append(frame[x, y])

        if len(set(pixs)) == 1:
            _line_status = 0
        else:
            _line_status = 1

        if _line_status &lt;&gt; line_status:
            if line_status &lt;&gt; None:
                if _line_status == 0:
                    _x = x
                elif _line_status == 1:
                    _x = x-1

                pos_x.append(_x)
                
                # 辅助线
                for _y in xrange(heigth):
                    frame_new[x, _y] = BLACK

        line_status = _line_status

    #img_new.show()
    # 开始切分
    i = 0
    divs = []
    boxs = []
    while True:
        try:
            xi = pos_x[i]
            xj = pos_x[i+1]
        except:
            break

        i += 2
        boxs.append([xi, xj])

    fixed_boxs = []
    i = 0
    while i &lt; len(boxs):
        box = boxs[i]
        if box[1] - box[0] &lt; 10:
            try:
                box_next = boxs[i+1]
                fixed_boxs.append([box[0], box_next[1]])
                i += 2
            except Exception, e:
                break
        else:
            fixed_boxs.append(box)
            i+=1


    for box in fixed_boxs:
        div = img.crop((box[0], 0, box[1], heigth))
        try:
            divs.append(format_div(div, size=(20, 40)))
        except:
            divs.append(div)

    # 过滤掉非字符的切片
    _divs = []
    for div in divs:
        width, heigth = div.size
        if width &lt; 5:
            continue

        frame = div.load()
        points = 0
        for i in xrange(width):
            for j in xrange(heigth):
                p = frame[i, j]
                if p == BLACK:
                    points += 1

        # 单张图片中至少有N个黑色点
        if points &lt;= 5:
            continue
        
        new_div = format_div(div) 
        _divs.append(new_div)
    
    return _divs
</pre>

切分开来之后，根据需要可以对每一个切片做一些简单处理，比如：去掉字符上下左右的空白、将图片旋转到字符宽度最小的的位置。



接下来就可以通过分隔之后的二值化图片，对a-z的每一个字符建立对应模板，识别过程只需要循环匹配这些模板，找到最相似的那个即可。我尝试过之后，发现识别率太低，于是换了一个思路，识别过程直接用<a href="https://code.google.com/p/tesseract-ocr/" target="_blank">tesseract</a>来做（对应的python客户端是pytesseract），识别率在25%左右，基本上能应付回帖的需要了。 

<pre>
def image_to_string(img, config='-psm 8'):
    """ 使用tesseract 识别图片中的文字
    """
    try:
        result = pytesseract.image_to_string(img, lang='eng', config=config)
        result = result.strip()
        return result.lower()
    except:
        return None</pre>