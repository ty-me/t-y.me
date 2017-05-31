---
id: 331
title: 在Rails项目中使用MongoDB
date: 2015-05-25T17:03:41+00:00
author: TY
layout: post
guid: http://t-y.me/?p=331
permalink: /p/331
views:
  - 632
duoshuo_thread_id:
  - 1163061200838197286
categories:
  - ROR
tags:
  - ActiveRecord
  - gem
  - mongodb
  - mongoid
  - rails
  - ruby
---
在ROR中，默认的ActiveRecord只支持关系型数据库，如果想在项目中使用mongodb的话，需要安装第三方的gem库，常用的有mongo_wrapper，mongoid，现在mongodi比较主流。
  
安装过程参考mongodi的[官方文档](http://mongoid.org/en/mongoid/docs/installation.html)，
  
>顺便提一下，查阅技术资料最好直接看官方文档，别人博客里写的tutorial绝对不是100%准确的，就像我现在写的这篇博客一样，或多或少都会存在个人理解上的偏差，因此照着别人的步骤做往往不一定能得到想要的结果，对于非官方的tutorial，作为参考就可以了。

## 安装

编辑Gemfile，添加一行

    gem "mongoid", "~&gt; 4.0.0"
    

使用bundle安装

    $ bundle isntall
    

## 配置

通过下面的命令会自动在config目录下生成mongoid.yml的配置文件

    rails g mongoid:config
    

在config/mongoid.yml里可以设置mongodb数据库的主机地址、端口、数据库名等信息，根据情况修改对应的项即可。
  
 修改config/application.rb文件，先删掉`require "rails/all"`和`config.active_record.raise_in_transactional_callbacks = true`这两行，再添加以下几行：
 <pre>
 require "action_controller/railtie" 
 require "action_mailer/railtie" 
 # require "active_resource/railtie" 
 # Comment this line for Rails 4.0+ 
 require "rails/test_unit/railtie" 
 require "sprockets/railtie" # Uncomment this line for Rails 3.1+
 </pre>
 
 修改`config/environments/development.rb`文件，将里面所有涉及到active_record的配置项都注释掉： 
 <pre>config.active_record.migration_error = :page_load</pre>
 
 在config/initializers/目录下新建一个`mongoid.rb`文件，添加一行： 
 ```Mongoid.load!("config/mongoid.yml", :production)```
 
## 开始使用 

经过上面的配置，就可以开始使用了，现在请忘记rake db:migrate命令吧，需要添加修改字段只要修改对应的model文件即可。 

<pre>
class Person include Mongoid::Document 
  field :first_name, type: String 
  field :last_name, type: String end
  </pre>

## 相关Gems 
* 如果觉得mongode生成的objectid在url中显示不够美观的话，那么可以安装`mongoid_auto_increment_id`来生成自增的整数形id。 
*  使用`mongoid-rails`可以防止query的注入 
*  因为在替换成mongoid之后，默认的migratetion就已经不能用来了，所以可以用`mongoid_rails_migrations`来实现针对mongodb的数据迁移功能。