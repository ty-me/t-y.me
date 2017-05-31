---
id: 370
title: 怎样在git中创建一个自定义的hook
date: 2015-09-08T13:30:09+00:00
author: TY
layout: post
guid: http://t-y.me/?p=370
permalink: /p/370
views:
  - 411
categories:
  - PYTHON
  - SERVER
tags:
  - git
  - git-hook
  - gitlab
  - hooks
---
当我们把代码push到远端之后，通常需要有一些后续的动作，比如当我向`dev`分支push代码之后，我会去测试环境下面同步最新的`dev`分支的代码，来验证一下新的代码有没有什么问题。再比如，像java/c/c++等等这种静态语言，在push之后，通常都会重新build出一个包，用来测试或者发布。这些push之后的后续的操作，都可以借助git提供的`hook`来完成。

## git hooks(钩子)简介

`git hooks`是一些小的脚本程序，当git产生某些事件的时候，会去触发这些事件所对应的程序，因此我们可以根据自己的需要，来编写自己的`hook`。我们常用的事件无非这么几种：
  
* `pre-receive` 它是在push的时候第一个被触发的事件。可以在这里做一些对push权限的管理，比如不允许某些用户向特定的分支push代码。
  
* `update` 它跟上面的`pre-receive`很像，区别是它会被每一个有变更的branch所触发，加入开发者一次push上来了多个branch，那么它会被触发n次。
  
* `post-receive` 它会在最后一步被触发。可以在这里作一些后续的操作，比如启动一个服务、发送一份通知邮件、触发CI系统等等。
  
更多其它的事件类型，可以点击[这里](https://www.kernel.org/pub/software/scm/git/docs/githooks.html)查阅

## 举个栗子

![](http://tyblog.qiniudn.com/15-9-8/90323361.jpg)
  
接下来我用hook实现这样一个功能：**当开发者push代码之后，在屏幕上打印`v2exz`某个栏目下的最新帖子**

### 新建custom_hooks目录

我这里用了gitlab，所以目录会有些不太一样

    mkdir -p /var/opt/gitlab/git-data/repositories/www/YOUR_PROJECT.git/custom_hooks
    

修改一下目录权限：

    chmod -R 775 custom_hooks
    

### 用python实现一个update的hook

可以用任何你喜欢的语言来实现一个hook，这里我用了python，需要注意的是，一定要在文件的第一行加指明脚本运行环境。否则脚本不能被正常允许
  
在上一步的customer_hooks目录下新建一个文件:

    touch update
    

源代码如下（需要安装requests和feedparser）：

    #!/usr/bin/python
    #encoding:utf-8
    import sys
    import requests
    import feedparser
    def get_params():
        """
        获取脚本执行时的参数
        """
        args = sys.argv
        assert len(args) == 4
        hook, refs, old, new  = args
        branch = refs.split('/')[-1]
        params = {'hook':hook, 'refs':refs, 'old':old, 'new':new, 'branch':branch}
        return params
    def print_v2ex_items():
        max_items = 5
        rss_url = 'https://www.v2ex.com/feed/tab/creative.xml'
        xml_content= requests.get(rss_url).content
        data = feedparser.parse(xml_content)
        items = data.get('entries', [])
        for item in items[:max_items]:
            title = item['title']
            url = item['link']
        print(title.encode('utf-8'))
        print url
            print '\n' * 2
    def main():
        params = get_params()
        if params['branch'] &lt;&gt; 'master':
            print_v2ex_items()
    

为了不影响代码的部署工作，只在非`master`的分支才调用输出方法。
  
修改一下文件权限和所有者

    chmod 775 update
    chwon git:git update
    

### 优化

到此，已经算完成了。push了几次之后，不难发现响应速度会比较慢，那是因为每次运行都会去v2ex重新拉一下rss，其实没有必要，因此，可以将拉去rss的动作丢在crontab中执行，hook中直接读取和解析就可以了。

#### 新增一条crontab任务，每分钟执行一次

    */1 * * * *  cd /tmp/ &amp;&amp; /usr/bin/curl https://www.v2ex.com/feed/tab/creative.xml -O
    

执行之后会在/tmp/目录下创建creative.xml文件。
  
修改一下print\_v2ex\_items方法，直接从creative.xml文件中解析rss即可：

    def print_v2ex_items():
        max_items = 5
        data = feedparser.parse('/tmp/creative.xml')
        items = data.get('entries', [])
        for item in items[:max_items]:
            title = item['title']
            url = item['link']
        print(title.encode('utf-8'))
        print url
            print '\n' * 2
    

修改之后的代码如下：

    #!/usr/bin/python
    #encoding:utf-8
    # author:http://t-y.me/
    import sys
    import feedparser
    def get_params():
        """
        [
          '/var/opt/gitlab/git-data/repositories/www/nile.git/custom_hooks/update', 
          'refs/heads/test', 
          '435f94d411bde154f12efcf6b086bd3127d323d1', 
          '7457a08857d879a1c159d3c24926a010a10dba8e'
        ]
        """
        args = sys.argv
        assert len(args) == 4
        hook, refs, old, new  = args
        branch = refs.split('/')[-1]
        params = {'hook':hook, 'refs':refs, 'old':old, 'new':new, 'branch':branch}
        return params
    def print_v2ex_items():
        max_items = 5
        data = feedparser.parse('/tmp/creative.xml')
        items = data.get('entries', [])
        for item in items[:max_items]:
            title = item['title']
            url = item['link']
        print(title.encode('utf-8'))
        print url
            print '\n' * 2
    def main():
        params = get_params()
        if params['branch'] &lt;&gt; 'master':
            print_v2ex_items()
    if __name__ == '__main__':
        main()
    

## 参考资料

http://blog.hgomez.net/2015/03/02/Gitlab-custom-hooks-Bash-Way.html
  
http://stackoverflow.com/questions/3762084/git-empty-arguments-in-post-receive-hook
  
http://stackoverflow.com/questions/7331519/find-git-branch-name-in-post-update-hook
  
http://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks#Server-Side-Hooks

\*\* 升级WP后，MD编辑器竟然吞掉了引用块中的回车符&#8230; \*\*