---
title: pdo GC
date: 2017-11-22 17:30:38
tags: php
---


```
$pdo = new PDO('mysql:host=127.0.0.1;port=3306;dbname=meizitu', 'root', '000000');
#$mysql = mysql_connect('127.0.0.1','root','000000');
sleep(5);
$stmt = $pdo->prepare('select * from FromBaseInfo');
$stmt = null;
$pdo = null; 
#mysql_query('select * from FromBaseInfo',$mysql); 
#mysql_close($mysql);
sleep(60); 
```


代码 | 计数变化
---|---
new pdo  | refcount = 1
pdo->prepare() | refcount = 2
stmt = null  | refcount = 1 
pdo = null  | refcount = 0 此时 mysql连接才被真正的释放。


