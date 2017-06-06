---
id: 372
title: ActiveAdmin使用中遇到的问题总结
date: 2017-02-01T09:45:21+00:00
author: TY
layout: post
permalink: /p/372
categories:
  - WEB
tags:
  - ROR
  - rails
  - activeadmin
---

# ActiveAdmin和will_paginate冲突

如果同时使用了will_paginate和AA，那么会导致AA加载列表的时候报错。
解决办法：https://github.com/activeadmin/activeadmin/blob/master/docs/0-installation.md#gem-compatibility

增加一个初始化脚本```config/initializers/kaminari.rb```, 代码如下：

```
Kaminari.configure do |config|
  config.page_method_name = :per_page_kaminari
end
```

# ActiveAdmin样式冲突

安装AA之后，会吧静态文件放在app/assets下，这样会导致AA的样式将app的样式覆盖的问题。
解决办法是将AA相关的静态文件放入到```vender/assets```下。
