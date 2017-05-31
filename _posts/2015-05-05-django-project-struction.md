---
id: 271
title: 更加实用的django项目目录结构
date: 2015-05-05T10:51:47+00:00
author: TY
layout: post
guid: http://t-y.me/?p=271
permalink: /p/271
views:
  - 813
duoshuo_thread_id:
  - 1163061200838197279
categories:
  - DJANGO
tags:
  - app
  - django
  - models
  - settings
  - urls
  - views
  - 目录
  - 结构
---
我个人不是很喜欢django的默认目录结构，用startproject demo1和startapp foo,startapp bar创建出来的目录结构大概长成这样： 

<pre class="prettyprint">.
|-- bar
|   |-- admin.py
|   |-- __init__.py
|   |-- migrations
|   |   `-- __init__.py
|   |-- models.py
|   |-- tests.py
|   `-- views.py
|-- demo1
|   |-- __init__.py
|   |-- __init__.pyc
|   |-- settings.py
|   |-- settings.pyc
|   |-- urls.py
|   `-- wsgi.py
|-- foo
|   |-- admin.py
|   |-- __init__.py
|   |-- migrations
|   |   `-- __init__.py
|   |-- models.py
|   |-- tests.py
|   `-- views.py
`-- manage.py</pre>

之所以说不喜欢这个结构，是有这么几点原因： 

1，django出于解耦合以及app复用的考虑，将app完全分开放在单独的目录下来管理，但是在实际的项目中其实并不需要做到这么松散，对于稍微大一点的项目，app之间的耦合有时候是必然的。而对于app复用这一点，我认为，如果你不是像外包公司那样，为了敷衍甲方需要分分分钟把前一个项目的某项功能(app)引入到新的项目中的话，是很难做到复用的，不同的项目之间哪怕是同一个功能，多少都会存在差异，打个比方，比如用户登录注册这个功能(app)，在项目a中可能是用手机号码注册、然后发短信验证，而在项目b中可能是email注册，发邮件验证，虽然他们都可一单独成一个app，但是因为这些差异存在，所以还是不能直接复用的。 

2，models.py文件被分开到了多个目录中，不方便管理。比如需要在foo的models中需要外链bar的models的某张表，就会很奇怪地从bar.models中import，这种情况恰恰又导致了耦合的产生。与其如此，那还不如把类似与models这种基础的组件单独放在一个文件（或者文件夹）下更好一些。 

3，从上面的目录结构中可以看见demo1（项目配置文件所在目录）与app的目录foo和bar是处于同级的，这对于非python/django的工程师来说，他们可能会认为demo1也是一个app，因为它跟foo和bar平行。 

由于以上的种种不爽，经过N多项目的检验，我觉得以下目录结构更适合于一个django项目： 



<pre class="prettyprint">|-- apps  
|   |-- bar
|   |   |-- __init__.py
|   |   |-- urls.py
|   |   `-- views.py
|   |-- foo
|   |   |-- __init__.py
|   |   |-- urls.py
|   |   `-- views.py
|   |-- __init__.py
|   `-- test.py
|-- base
|   |-- context.py
|   |-- http.py
|   |-- __init__.py
|   |-- json.py
|   |-- logger.py
|   |-- middlewares.py
|   |-- models.py
|   |-- forms.py
|   `-- pagination.py
|-- dev_settings.py
|-- env.sh
|-- fabfile.py
|-- __init__.py
|-- manage.py
|-- pro_settings.py
|-- scripts
|   |-- requirements.txt
|   |-- setupenv.sh
|   |-- update_sitemap.sh
|   `-- uwsgi.sh
|-- settings.py
|-- static
|   |-- admin
|   |-- fonts
|   |-- mobile
|   `-- web
|-- templates
|-- test_settings.py
|-- TODO.list
|-- urls.py
`-- wsgi.py</pre>



<table style="width:100%;" cellpadding="2" cellspacing="0" border="1" bordercolor="#CCCCCC">
  <tr>
    <td>
      <span>apps目录</span>
    </td>
    
    <td>
      <span>用来存放不同的功能模块。</span>
    </td>
  </tr>
  
  <tr>
    <td>
      <span>base目录</span>
    </td>
    
    <td>
      <span>用来存放一些基础组建相关的代码，比如表结构(models.py)，自定义的middlewares (middlewares.py)，修改过的分页方法(pagination.py)，表单相关(forms.py)等等。</span>
    </td>
  </tr>
  
  <tr>
    <td>
      <span>scripts目录</span>
    </td>
    
    <td>
      <span>存放一些shell文件相关的，比如setupenv.sh（初始化项目环境），uwsgi.sh(启动uwsgi)等等。</span>
    </td>
  </tr>
  
  <tr>
    <td>
      <span>templates</span>
    </td>
    
    <td>
      <span>模板文件。</span>
    </td>
  </tr>
  
  <tr>
    <td>
      <span>fabfile.py</span>
    </td>
    
    <td>
      <span>用fabrice实现自动化远程部署项目。</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
    </td>
  </tr>
  
  <tr>
    <td>
      <span>dev_settings.py</span>
    </td>
    
    <td>
      <span>开发环境下的配置文件</span>
    </td>
  </tr>
  
  <tr>
    <td>
      <span>pro_settings.py</span>
    </td>
    
    <td>
      <span>生产环境下的配置文件</span>
    </td>
  </tr>
  
  <tr>
    <td>
      <span>test_settings.py</span>
    </td>
    
    <td>
      <span>测试环境下的配置文件</span>
    </td>
  </tr>
  
  <tr>
    <td>
      <span>settings.py</span>
    </td>
    
    <td>
      <span>总配置项目</span>
    </td>
  </tr>
  
  <tr>
    <td>
      <span>urls.py</span>
    </td>
    
    <td>
      <span>总的url配置文件</span>
    </td>
  </tr>
</table>