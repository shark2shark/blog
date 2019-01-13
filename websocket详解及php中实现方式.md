---
title: websocket详解及php中实现方式
date: 2019-01-13 21:29
keywords:
 - php
 - socket
 - websocket
description: websocket,php中实现websocket
categories:
 - php
 - socket
tags: 
 - php
 - socket
---

- **websocket是什么**
	随着web领域越来越复杂的交互,多数应用需要很强的数据交互逻辑和数据实时推送的需求,以往实现类似实时交互的方式一般用ajax轮训,即不停的向服务器请求,询问到底有没有消息或者http long poll,发起请求,没有消息一直阻塞着,有了再返回.这些方式对服务器和客户端的负载都是偏重的,尤其是ajax轮训,他们都要不停创建链接.所以开始有人思考能不能在客户端实现一种真正的长连接?websocket应用而生!简单的说websocket是一个全双工的基于tcp依赖于http的新的通讯协议,在websocket中服务器可以主动给客户端发送信息,客户端也可以主动给服务器发送信息,并且链接一旦创建是以长连接的方式存在,没有了轮训的资源耗费,对于websocket的解释知乎中有一篇点赞很高的回答,=[可以看看][1]
	- 为什么说websocket依赖于http?
		- websocket的链接创建使用了http的Upgrade来升级协议,创建成功后就跟http没有任何关系了,下面是我用curl模拟创建websocket的请求和响应:

			//curl --no-buffer -H 'Connection: Upgrade' -H 'Upgrade: websocket' -v -H 'Sec-WebSocket-Version: 13' -H 'Sec-WebSocket-Key: websocket'  http://127.0.0.1:8081

			//httpRequest
			 Connection: Upgrade
			 Upgrade: websocket
			 Sec-WebSocket-Version: 13
			 Sec-WebSocket-Key: websocket
			 换行

			//httpResponse
			 HTTP/1.1 101 Switching Protocols
			 Upgrade: websocket
			 Connection: Upgrade
			 Sec-WebSocket-Accept: qVby4apnn2tTYtB1nPPVYUn68gY=
			换行 

		- 解释一下
		  我向127.0.0.1:8081发送了一个http请求
		  请求头包含:Connection: Upgrade 告知服务器我这个连接是个特殊协议用来升级的
		  请求头包含:Upgrade: websocket 告知服务器我这个连接是用来转换成websocket协议的
		  请求头包含:Sec-WebSocket-Version: 13 告知服务器我需要的websocket协议版本是13
		  请求头包含:Sec-WebSocket-Key: websocket 告知服务器我用于升级websocket的加密可以是websocket,浏览器中这个key是随机字符串
		  
		  响应状态:101 Switching Protocols 告知客户端这是个交换协议
		  响应头包含: Upgrade: websocket 告知客户端将要升级的协议是websocket
		  响应头包含: Connection: Upgrade 告知客户端这个连接时特殊协议用来升级的
		  响应头包含: Sec-WebSocket-Accept: qVby4apnn2tTYtB1nPPVYUn68gY= 告知客户端即将升级的协议key加密后的值并要求客户端确认,如果确认成功,那么websocket协议就建立成功了,否则失败
		  
	- websocket是怎么依赖http的
		  上面分析了创建websocket的http请求和响应过程,这个过程就是websocket协议的握手环节,其中请求头Sec-WebSocket-Key和响应头Sec-WebSocket-Accept是一对,怎么说呢,在浏览器中请求头Sec-WebSocket-Key是一个随机字符串(KEY),发送到服务器后服务器用KEY+固定字符串"258EAFA5-E914-47DA-95CA-C5AB0DC85B11"用sha1加密后再用base64编码,在php中如下:
```php
		 
			 $acceptWsKey = base64_encode(sha1($wsKey . '258EAFA5-E914-47DA-95CA-C5AB0DC85B11', true));

```

- **php中websocket的实现方式**
	
	(待更新...)

	- socket函数族

	- 代码示例

  [1]: https://www.zhihu.com/question/20215561