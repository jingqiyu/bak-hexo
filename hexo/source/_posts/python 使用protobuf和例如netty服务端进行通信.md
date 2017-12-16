---
title: python 使用protobuf和例如netty服务端进行通信
date: 2016-11-05 12:31:38
tags: python
---

首先按照python protobuf配置 将protobuf配置好

当我们想使用protobuf做为协议进行通信时，好多网络程序，要求在报文前加上报文的varint值。 我们现在看看如何使用。

准备好一个message之后

```python 
import google.protobuf.internal import encoder
serializeStr = clubsys.SerializePartialToString() #得到序列化后的对象二进制流
varint = encoder._VarintBytes( len( serializeStr ) ) #计算报文长度的varint值
sendMsg = varint+serializeStr #拼装消息
```



响应报文格式和发送格式类似，也是varint+实际报文。由于3个字节的可以解析出我们在这个业务下想要的最大报文长度。这里面先接受3个字节 解析出varint所占长度和message所占长度


通过google.protobuf.internal import decoder 的 _DecodeVarint  解析 
然后继续接受剩下的字符串 。组合成完整的字符串后，

切片后，然后再通过protbobuf的parseFromString 进行解码

```python
BUFFSIZE = 3
returnStr = mysock.recv( BUFFSIZE )
msgLength,jumpLength = decoder._DecodeVarint( returnStr, 0 )
message = mysock.recv( msgLength+jumpLength-3 )
returnStr= returnStr+message
clubsys.ParseFromString( returnStr[jumpLength::] )
```

解码部分最好写成动态尝试从1个字节、2个字节 逐一对其进行解码。
