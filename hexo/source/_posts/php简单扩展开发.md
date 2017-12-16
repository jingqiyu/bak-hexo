---
title: php简单扩展开发
date: 2016-02-20 17:30:38
tags: php
---



1. 环境 ubuntu 15.04 php源码包  假设路径是 ~/php-5.6.14
2. cd   ~/php-5.6.14/ext 
3. vim hello_module.def  输入函数定义  如 void hello()
4. ./ext_skel --extname= hello_module--proto= hello_module.def  运行
5. cd hello_module
6. 将 
![image](https://wx4.sinaimg.cn/mw690/4d662fa1gy1flruuq2n2pj20jm02owem.jpg)
下的注释符号 DNL全部删除掉

7. 打开 hello_module.c 完成要达到的功能
8. 找到phpize 路径 使用命令
/usr/localhost/webserver/php/bin/phpize  
./configure --with-php-config=/usr/localhost/webserver/php/bin/php-config 和phpize同一路径

9. make && make install
10. 根据提示 找到生成的hello_module.so文件 

11. 建立一个文件夹 保存自己创建好的扩展文件
10. 打开php.ini  找到 extension_dir 定义为你创建好的文件夹
11. 重启php

![image](https://wx3.sinaimg.cn/mw690/4d662fa1gy1flruun6lvzj20rm0k2412.jpg)

