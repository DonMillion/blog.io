---
layout: post
title: php的变量
date: 2017-2-23
categories: blog
tags: [php原理, php]
description: 大概了解一下php变量结构。
---

#### php变量的机构 ####

php是一种弱类型语言，一个变量的类型可以随时改变，因为php内部自动进行了转化。

php5中的变量结构大致如下，内存占据24个字节：
```
struct _zval_struct {
    /* 变量信息 */
    union {
	    long lval;                  /* 长整形 */
	    double dval;                /* 双浮点数 */
	    struct {
	        char *val;
	        int len;
	    } str;                      /* 字符串类型 */
	    HashTable *ht;              /* 哈希表，即数组 */
	    zend_object_value obj;      /* 对象类型 */
	    zend_ast *ast;
	} value;
	/* 这个联合体是php变量的具体值得存放结构, 使用联合体，可以减少内存的占用 */

    zend_uint refcount__gc; /* 引用计数 */
    zend_uchar type;    /* 变量具体的类型 */
    zend_uchar is_ref__gc; /* 表示是否为引用 */
};
```


在php7中，变量的结构如下，内存占据16个字节：
```
struct _zval_struct {

	/* php7变量具体值，比php5类型多了很多，但只会占用一个内存。如果是不需要引用的类型，直接就是值，否则是指针 */
	union {
		long   lval;  /* 整形 */
		double dval;  /* 浮点型 */
		zend_refcount *counted;
		zend_string *str; /* 字符串类型 */
		zend_array *arr;  /* 数组类型 */
		zend_object *obj; /* 对象类型 */
		zend_resource *res; /* 资源来兴 */
		zend_reference *ref; /* 引用 */
		zend_ast_ref *ast; /* 这个目前我还不懂 */
		zval *zv;
		void *ptr;
		zend_class_entry *ce;
		zend_function *func; /* 这个应该是匿名函数用的，指向一个函数的地址 */
	} value;

	/* 这个应该是变量的类型信息 */
	union {
		struct {
			ZEND_ENDIAN_LOHI_4(
				zend_uchar type,
				zend_uchar type_flags,
				zend_uchar const_flags,
				zend_uchar reserved)
		} v;
		zend_unit type_info;
	} u1;

	/* 这个是一个标志位 */
	union {
		zend_unit var_flags;
		zend_unit next;
		zend_unit str_offset;
		zend_unit cache_slot;
	} u2;

}
```
由原来的4个成员变量变成3个联合体，由于联合体只会用其中一个的内存，所以总的内存消耗是下降了。
可以看出，如果变量是不需要引用类型（NULL、Boolean、Long、Double），结构体内的value就是具体的值，如果是需要引用类型(String、Array、Object、Resource、Reference)，结构体内的value就是一个指针，直接指向真实的存储地址。这样的方式比原来要直接明了。



#### 其他的一些小点 ####



