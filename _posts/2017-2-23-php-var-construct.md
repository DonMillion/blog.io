---
layout: post
title: php的变量
date: 2017-2-23
categories: blog
tags: [php原理]
description: 大概了解一下php变量结构。
---

php是一种弱类型语言，一个变量的类型可以随时改变，因为php内部自动进行了转化。

源码中的php变量结构大致为
```
typedef struct _zval_struct zval;
...
struct _zval_struct {
    /* 变量信息 */
    zvalue_value value;     /* 指针，指向变量具体的值的地址 */
    zend_uint refcount__gc; /* 引用计数 */
    zend_uchar type;    /* 变量具体的类型 */
    zend_uchar is_ref__gc; /* 表示是否为引用 */
};
```

