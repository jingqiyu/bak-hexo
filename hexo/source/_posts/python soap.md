---
title: python soap
date: 2016-11-20 17:30:38
tags: python
---

#### 简单的一个通过suds访问webservice的demo

适用： server端 （ 任意语言编写 soap2.0 协议 wsdl方式访问）

client端代码如下
```python
from suds.client import Client

soapUrl = 'http://data.webservice.jj.cn/safecenter/tkwsafecenterservice.asmx?WSDL'
soap_client = Client(soapUrl);
#d = {"uPID_IN":328944427};
d = dict(uPID_IN=32894427);
print d
exit()
rs = soap_client.service.GetMobileBindDetail(**d);
print rs.szMobile_OUT;
```
- suds模块是轻量级的，调用webservice非常方便 使用也很方便
- 函数参数准备 如果是多个参数可以使用字典的形式，也可以直接在函数中指定
- 接收webservice返回值
