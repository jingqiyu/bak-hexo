---
title: php函数源再次码阅读学习
date: 2017-07-10 11:05:22
tags: php
---

```c
PHP_FUNCTION(strpos)
{
	zval *needle;
	char *haystack;
	char *found = NULL;
	char  needle_char[2];
	long  offset = 0;
	int   haystack_len;
    //zend_parse_parameters()是检查、初始化函数参数的一个过程、其中ZEND_NUM_ARGS是获取参数的个数的宏定义、TSRMLS 是线程安全资源管理器
    //sz|l 指定参数的类型 s 代表string、z zval “|" 后面的值为函数的可选参数 l long一般是代替php中的int 将参数复制给后面的几个变量。
	if (zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "sz|l", &haystack, &haystack_len, &needle, &offset) == FAILURE) {
		return;
	}
     
    //边界检查
	if (offset < 0 || offset > haystack_len) {
        //返回一个php Warningdoc 在 error_reporting 设置在E_WARNING 级别下的时候 报错  php_error_docref是类似格式化输出的语句 可以使用占位符的方式比如 %s 等.
		php_error_docref(NULL TSRMLS_CC, E_WARNING, "Offset not contained in string");
		RETURN_FALSE;
	}
    
    //判断类型是不是字符串
	if (Z_TYPE_P(needle) == IS_STRING) {
		if (!Z_STRLEN_P(needle)) {
			php_error_docref(NULL TSRMLS_CC, E_WARNING, "Empty needle");
			RETURN_FALSE;
		}
        //通过php_memnstr找到needle的位置
		found = php_memnstr(haystack + offset,
			                Z_STRVAL_P(needle),
			                Z_STRLEN_P(needle),
			                haystack + haystack_len);
	} else {
        //尝试将整形转换成字符 比如65 => 'A' 
		if (php_needle_char(needle, needle_char TSRMLS_CC) != SUCCESS) {
			RETURN_FALSE;
		}
        //需要注意的一点  neele_char 定义中长度是两个长度 C语言的字符串以\0结束。而php中却不支持，只是通过复制一个0来表示字符串结束
		needle_char[1] = 0;
		found = php_memnstr(haystack + offset,
							needle_char,
							1,
		                    haystack + haystack_len);
	}
	if (found) {
		RETURN_LONG(found - haystack);
	} else {
		RETURN_FALSE;
	}
}
```

