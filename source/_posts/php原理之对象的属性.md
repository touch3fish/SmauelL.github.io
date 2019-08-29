---
title: php原理之对象的属性
date: 2019-08-28 17:38:58
tags: 源码
category : php
---
{% cq %}之前用easyWechat时获取的用户信息是protected的对象,使用了`\0*\0`来获取到受保护的对象中的数据,最近在学习php底层的知识,本文讲述了为什么`\0*\0`能获取到受保护的数据{% endcq %}
## 对象的结构
在php中,一个对象,还是以一个zval做为载体的
```
typedef union _zvalue_value {
    long lval;
    double dval;
    struct {
        char *val;
        int len;
    } str;
    HashTable *ht;
    zend_object_value obj;
} zvalue_value;
```
<!--more-->
如果,一个zval是对象,那么zvalue_value中的对象,就指向一个zend_object_value的实例。一个zend_object_value包含两个成员,一个是标识符(整形序号),表明了当前对象存储在全局对象列表的位置,另外还有一个zend_object_handlers指针,指向当前对象所属类的handlers(标准操作集合).
真正的对象实体,zend_object中,保存了如下的关键信息入口:
```
1.ce,zend_class_entry 类入口
2.properties,hashTable普通属性集
```
## 对象的属性
如上所述,普通属性是一个hashTable,在PHP5以后,引入了访问权限控制,而访问权限属性,是通过属性名进行区分的(为此Zend引入了zend_mangle_property_name)
```
1.public 属性名
2.private \0类名\0属性名
3.protected \0*\0属性名
```
PHP通过这种方式,实现了对属性访问权限的标识,所以我们如果想要访问对象的私有/保护属性可以:
```
class Test{
    private $_name1 = "private";
    protected $_name2 = "protected";
}
$test = new Test();
$arr = (array)$test;
var_dump($arr["\0Test\0_name1"]);
var_dump($arr["\0*\0_name2"]);
//输出
string(7) "private"
string(9) "protected"
```
`参考自鸟哥博客`http://www.laruence.com