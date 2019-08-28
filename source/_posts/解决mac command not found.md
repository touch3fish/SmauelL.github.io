---
title: 解决mac command not found
date: 2019-08-22 12:08:44
tags: mac
category : mac
---
{% cq %}最近在配置mac的环境变量后,系统的所有命令都失效了,提示mac command not found,在此记录解决方法{% endcq %}

## 1.先让命令暂时可用  
命令如下
```
export PATH=/usr/bin:/usr/sbin:/bin:/sbin:/usr/X11R6/bin
```
<!--more-->
命令执行后不要关闭终端或者cd /usr/bin 下执行vi命令否则会再次失效  
## 2.检查bash_profile文件  
很有可能是你的PATH 环境变量设置错误，比如 `$PATH `漏了，我这里的错误是添加环境变量的时候没有使用`""`双引号引起来，还有`=`等号后面有空格，不知道等号后面有空格会不会影响，加上双引号，去掉空格就可以了` PATH=$PATH:$PATH1` 可以写成这样的格式：`export PATH=/usr/local/msyql/bin:$PATH`

## 3.保存bash_profile文件
```
source ~/.bash_profile
```


