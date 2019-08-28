---
title: composer报错zlib_decode()
date: 2019-08-28 14:50:52
tags: laravel
category : composer
---
{% cq %}今天使用composer安装laravel遇到报错zlib_decode(),在此记录一下解决办法{% endcq %}

以前从来没有遇到的问题,报错的安装命令为
```
composer create-project  laravel/laravel bjyblog --prefer-dist
```
一直提示zlib_decode(),找了许多方法都没有效果。
<!--more-->
然后我就机智的将`--prefer-dist`移了一下位置
```
composer create-project --prefer-dist laravel/laravel bjyblog
```
这样就好了,网上有很多教程将`--prefer-dist`放到最后,希望大家不要踩坑。