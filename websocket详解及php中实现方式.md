---
title: websocket详解及php中实现方式
date: 2019-01-13 21:29
keywords:
 - php
 - socket
 - websocket
description: websocket是什么?websocket的链接过程是怎样的?在php中websocket服务应该怎么创建?
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
	
	- socket函数族
		- socket_create( int $domain , int $type , int $protocol ): resource $return
			- 函数作用:创建一个socket资源并返回
			- 参数:
				- $domain    指定哪种网络协议,选项如下,[详细选项][2]
			 	- $type    参数用于选择套接字使用的类型,[详细选项][2]
			 	- $protocol 指定 domain 套接字下的具体协议,如tcp,[详细选项][2]
		   - 返回值: $return 返回一个创建的socket资源
		   - 说明:该函数创建一个新的可操作的socket资源并返回
		- socket_build(resource $socket , string $address [, int $port = 0 ]): bool $return
			- 函数作用: 给socket绑定地址
			- 参数: 
				- $socket 有socket_create返回的合法socket资源
				- $address 需要给套接字绑定的资源,如果套接字是 AF_INET 族，那么 address 必须是一个四点分法的 IP 地址（例如 127.0.0.1,如果套接字是 AF_UNIX 族，那么 address 是 Unix 套接字一部分（例如 /tmp/my.sock ）
				- $port 绑定的端口号,仅在AF_INET族生效
			- 返回值: $return 成功|失败 true|false
			- 说明:该函数作用是为socket绑定需要监听的地址
		- socket_listen( resource $socket [, int $backlog = 0 ]): bool $return
			- 函数作用:开始监听socket上绑定的链接
			- 参数:
				- $socket 需要监听的socket
				- $backlog 链接队列大小,如果连接请求到达时队列已满，则客户端可能会收到一个指示econnnrejected的错误，或者，如果基础协议支持重新传输，则可以忽略该请求，以便重试成功。
			- 返回值: $return 成功|失败 true|false
			- 说明:就是开始用绑定地址监听socket啦
		- socket_accept( resource $socket ): resource $return
			- 函数作用: 阻塞等待socket接受链接
			- 参数
				- $socket 等待接受的socket资源
			- 返回值: $return 新连接的socket资源(客户端)
			- 说明: 该函数一直阻塞等待,知道socket有新链接为止
		- socket_select( array &$read , array &$write , array &$except , int $tv_sec [, int $tv_usec = 0 ] ) :int|bool $return
			- 函数作用: 对给定的socket数组以制定时间运行select()系统调用来检查socket的活动状态
			- 参数:
				- $read 引用传递, 要检测的socket数组传入后将没有活动的socket删除,只剩下活动的socket链接
				- $write 引用传递, 检查$read数组中的socket是否可写
				- $except 引用传递 检查$read数组中的socket是否异常
				- $tv_sec 超时时间秒 (可与下一个参数配合使用),如果为null理解返回
				- $tv_usec超时时间微妙(可与上一个参数配合使用)
			- 返回值: $return 成功是返回检测到的活动的socket总数,超时返回0,出错返回false
			- 说明: 运用socket_select 可以实现多个socket同时监听,可以实现多socket一起链接
		- socket_set_option( resource $socket , int $level , int $optname , mixed $optval ): bool $return
				- 函数作用: 设置socket选项
				- 参数
					- $socket 合法的socket资源
					- $level 制定选项所在的协议级别
					- $optname 选型名称
					- $optva 选项值
				- 返回值: $return 成功与否 true|false
				- 说明: 就是设置socket选项啦,详情见[官网文档][3]
- 代码示例
	```php
		//监听地址
		$address = '0.0.0.0';

		//监听端口
		$port = '8081';

		//定义一个sockets的数据，用来存放所有socket，包括服务端，用来给socket_select使用
		$sockets = [];

		//创建socket(第一个参数表示ipv4网络协议,第二个参数表示提供一个顺序化的、可靠的、全双工的、基于连接的字节流,第三个参数表示创建的具体协议是tcp)
		$server = socket_create(AF_INET, SOCK_STREAM, SOL_TCP);

		//端口复用，防止服务重启前绑定的端口还未释放导致的无法重新启动服务
		socket_set_option($server, SOL_SOCKET, SO_REUSEADDR, 1);

		//绑定地址和端口
		socket_bind($server, $address, $port);

		//开始监听
		socket_listen($server, 10);

		//将主服务socket压入sockets数组
		array_push($sockets, $server);

		//socket_select 第二个参数  空数组即可，详情看文档
		$selectWrite = [];

		//定义一个数组用于存放所有握手成功的socket
		$handShake = [];

		//socket_select 第三个参数   为Null即可，详情看文档
		$except = null;

		//轮训检测
		while (true) {

			//socket_select第一个参数，引用一个sockets数组并将不活动的socket删除，因为引用的原因，此处应该每轮循环都重新赋值
			$tempSelectRead = $sockets;

			//同上
			$tempSelectWrite = $selectWrite;

			$selectResult = socket_select($tempSelectRead, $tempSelectWrite, $except, null);

			//socket_select如果兼听不到活跃的socket或出错会返回0|false，因此到这直接跳过本轮循环进入下一轮
			if (!$selectResult) continue;

			//此时tempSelectRead中的socket就是当前活动的socket，将其循环处理
			foreach ($tempSelectRead as $socket) {

				//首先判断当前socket是不是主服务，如果是主服务那就处理链接请求
				if ($socket == $server) {
					//等待接受一个链接
					$client = socket_accept($socket);

					if ($client) {
						//此时得到的client就是一个新的客户端的socket，将其也放入全部sockets数组中
						array_push($sockets, $client);

						var_dump('新链接' . array_search($client, $sockets) . '已创建');

					}

				} else {

					//不是主服务的话就是客户端了,二话不说先把数据接受了，此处注意，只有没有握手时（也就是http部分传递的不是二进制数据，可以直接使用，一旦握手成功后续接受的到所有数据都是二进制需要解码处理）
					$buff = socket_read($socket, 1024);

					//如果没有收到消息,说明客户端主动关闭了,用socket_read函数，客户端关闭后收到的是一个空字符串(实际上socket关闭后服务端收到的是一个7字节长度的数据)
					if ($buff === '') {

						var_dump('客户端' . array_search($socket, $sockets) . '主动关闭');

						//将关闭的客户端从sockets数组中删除
						unset($sockets[array_search($socket, $sockets)]);

						//如果握手成功的话 那把他从握手数组中也删了
						if (isset($socket, $handShake))
							unset($handShake[array_search($socket, $handShake)]);

						//关闭这个客户端
						socket_close($socket);

					} else {

						//如果客户端没有关闭，首先应该判断的是这个链接是否握手成功,如果没有握过手,先握手
						if (!in_array($socket, $handShake)) {

							//得到http请求头中升级ws协议的key
							if (!preg_match("/Sec\-WebSocket\-Key:\s(.*)\r\n/", $buff, $result)) throw new Exception('未获取到协议升级KEY');

							$wsKey = $result[1];

							//编码key 编码方式: 请求头Sec-WebSocket-Key的值 + 固定字符串"258EAFA5-E914-47DA-95CA-C5AB0DC85B11" sha1进行hash后base64编码
							$acceptWsKey = base64_encode(sha1($wsKey . '258EAFA5-E914-47DA-95CA-C5AB0DC85B11', true));

							//组合响应头, 组合时\r\n用PHP_EOL替代或者\r\n写在双引号中,单引号\r\n就是一个字符串，在夸系统传递中会造成握手不成功的问题
							$responseHttpHeader = 'HTTP/1.1 101 Switching Protocols' . PHP_EOL;

							$responseHttpHeader .= 'Upgrade: websocket' . PHP_EOL;

							$responseHttpHeader .= 'Connection: Upgrade' . PHP_EOL;

							//此处注意，ws握手响应要求最后一行多拼写一个/r/n
							$responseHttpHeader .= 'Sec-WebSocket-Accept: ' . $acceptWsKey . PHP_EOL . PHP_EOL;

							//输出相应供客户端完成握手
							socket_write($socket, $responseHttpHeader, strlen($responseHttpHeader));

							//标记该客户端是握手状态
							array_push($handShake, $socket);

							var_dump('新连接' . array_search($socket, $sockets) . '握手成功');

						} else {
							//如果握手成功，则在此处处理数据逻辑,此处所有收到的数据要进行二进制解码输出的数据要用二进制编码

							//解码数据
							$message = getMsg($buff);

							var_dump($message);

							//编码输出
							$returnMessage = buildMsg($message);

							socket_write($socket, $returnMessage, strlen($returnMessage));
						}
					}
				}
			}
		}
	```

----------

> 如有不妥之处，敬请留言回复以便更正，防止误导他人；漫漫码路，与君共勉！转载请注明


  [1]: https://www.zhihu.com/question/20215561
  [2]: http://www.php.net/manual/zh/function.socket-create.php
  [3]: http://php.net/manual/zh/function.socket-set-option.php