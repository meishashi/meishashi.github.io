---
layout: post
title:  "多后台登录遇到的问题"
categories: laravel
tags:  laravel PHP
author: 燕南天
---

* content
{:toc}


#laravel 多后台登录遇到的问题

最近在学习laravel，想撸一个多后台登录的。

结果遇到了如下问题

```js
Argument 1 passed to Illuminate\Auth\EloquentUserProvider::validateCredentials() must be an instance of Illuminate\Contracts\Auth\Authenticatable, instance of App\Admin given, called in /Applications/MAMP/htdocs/blog/vendor/laravel/framework/src/Illuminate/Auth/SessionGuard.php on line 379
```

最后找出问题所在，特此记录下。

![](/assets/2018-11-20.png)

只需要把app/Admin.php 下的Model 改成Authenticatable就好了。

