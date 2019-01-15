---
title: pcntl-2
description: 什么是信号?php在多进程中信号是如何处理的?php中信号处理应该注意什么
keywords:
  - php
  - pcntl
categories:
  - php
  - pcntl
tags:
  - php
  - pcntl
abbrlink: 940bb56d
date: 2018-12-31 16:55:00
---

 - **信号**
		
	*说信号之前我们回想一下这样一个场景，相信不少人都经历过，由于在windows系统上面习惯了ctrl+c复制，所以有时候在服务器（linux）执行某个脚本的时候想复制这个正在执行的脚本打印出来的值的时候，顺其自然地鼠标选中然后ctrl+c。。然而复制没复制不说了，这脚本停了。但是脚本为什么停呢？这是因为我们在终端使用ctrl+c的时候系统会向当前进程发送一个叫SIGINT的信号（值是2），当进程接受到这个信号的时候如果代码中没有对这个信号做相应处理，那么程序会执行这个信号对应的默认行为，而SIGINT的默认行为就是终止进程。那么说到这里，相信大家对信号有点想法了；信号又叫软中断信号，用来通知进程发生了异步事件，要求进程对该事件进行处理，处理方式大致分三种：*
	
	1. 注册对应信号处理函数
	2. 忽律该信号
	3. 保留信号默认行为

< !--more-->

---
- **PHP中的信号处理函数**
	1. pcntl_alarm(int $seconds) :int $return
		- **解释** 创建一个计时器，在指定的秒数后向进程发送一个SIGALRM信号
		- **参数** $seconds 秒数
		- **返回** $return 返回设置剩余秒数
		- **提示** 该函数发送的是SIGALRM信号，需要在程序中注册SIGALRM信号的处理，并且这个函数只执行一次，每次调用都会取消上一次的设置的SIGALRM信号
	2. pcntl_signal ( int $signo , callback $handler [, bool $restart_syscalls = true ] ): bool $return
		- **解释** 注册一个信号到来时的处理函数
		- **参数** $signo 注册的信号; $handler 处理的回调函数; $restart_syscall 忽略即可,[官方文档说此参数存在bug,详情点击了解][1]
		- **返回** $return 注册是否成功 true|false
		- **提示** 此方法是对制定信号的自定义处理，如果不注册，当对应信号到达时将执行系统默认行为（如开始例子中提到的SIGINT信号，如果不注册该信号处理函数，那么ctrl+c将会终止进程）
	3. pcntl_signal_dispatch ( void ): bool $return
		- **解释** 函数pcntl_signal_dispatch()调用每个等待信号通过pcntl_signal() 安装的处理器。
		- **参数** 无参数
		- **返回** $return 返回调用结果，成功如否，true|false
		- **提示** 其实这个函数就是对信号进程收到信号的手动检测，如果代码中没有自动检查信号的代码(ticks)，那么必须要手动检测注册的函数才会生效
	4. pcntl_async_signals ([ bool $on = NULL ] ): bool $return
		- **解释** 异步信号检测方法(用来替代ticks,后面会说);7.1以上可用
		- **参数** $on 默认false， 如果是false，将返回程序中是否开启了异步信号检测，用来判断；如果是true,则对程序进行异步信号检测，用来设置
		- **返回** $return 如果$on是false返回程序中是否开始了异步信号检测，如果是true，则返回在开启之前是否开启了一步信号检测;是或否，true|false
		- **说明** 7.1后新加的方法，用来替代旧的信号自动检测（ticks,效率很低下，后面会说）
	5. pcntl_signal_get_handler ( int $signo ): mixed $return
		- **解释** 获取指定信号的当前处理程序
		- **参数** $signo 信号
		- **返回** $return 可能返回常量SIG_DFL(系统默认行为)或SIG_IGN(忽略行为)，也有可能返回注册的处理方法的名字或者是闭包，视具体注册行为返回具体值
		- **说明** 这个方法就是用来获取信号的处理程序，$return中说的比较具体了
	6. pcntl_sigprocmask ( int $how , array $set [, array &$oldset ] ): bool $return
		- **解释** 函数pcntl_sigprocmask()用来增加，删除或设置阻塞信号，具体行为 依赖于参数how。
		- **参数** $how 行为; $set 设置的信号数组;$oldset 输出参数,返回之前的信号阻塞列表;
			 - $how参数的行为选项如下:
				1. SIG_BLOCK 信号加入阻塞列表
				2. SIG_UNBLOCK 信号移除阻塞列表
				3. SIG_SETMASK 用给定的列表替换当前列表
		- **返回** $return 返回设置结果，成功与否，true|false
		- **说明** 这个函数中所提到的阻塞，就是说让信号检测机制跳过这个信号的检测,注意，虽说信号检测机制跳过这个信号的检测，但是如果注册了对应的处理方法,pcntl_signal_get_handler函数获取到的还是注册的方法，这个阻塞可以理解为处理信号行为时被拦截
	7. pcntl_sigwaitinfo ( array $set [, array &$siginfo ] ): int $return
		- **解释** pcntl_sigwaitinfo()函数暂停调用脚本的执行直到接收到set 参数中列出的某个信号。只要其中的一个信号已经在等待状态(比如： 通过 pcntl_sigprocmask()函数阻塞)， 函数pcntl_sigwaitinfo()就回立刻返回。
		- **参数** $set 要等待的信号数组; $sigininfo 输出参数,返回信号的信息如下：
			- 以下元素会为所有信号设置：
				1. signo: 信号编号
				2. errno: 错误编号
				3. code: 信号代码
			- 下面元素可能会为SIGCHLD信号设置:
				1. status: 退出的值或信号
				2. utime: 用户消耗的时间
				3. stime: 系统（内核）消耗的时间
				4. pid: 发送进程ID
				5. uid: 发送进程的实际用户ID
			- 信号SIGILL, SIGFPE, SIGSEGV 和 SIGBUS 可能会被设置的元素:
				1. addr: 发生故障的内存位置
			- 可能会为SIGPOLL 信号设置的元素：
				1. band: Band event
				2. fd: 文件描述符
		- **返回** $return 返回等待到的信号
		- **说明** 这个函数就是在信号不阻塞的情况下阻塞当前代码等待信号
	8. pcntl_sigtimedwait ( array $set [, array &$siginfo [, int $seconds = 0 [, int $nanoseconds = 0 ]]] )
		- **解释** 函数pcntl_sigtimedwait()实际上与pcntl_sigwaitinfo() 的行为一致，不同在于它多了两个增强参数seconds和 nanoseconds，这使得脚本等待的事件有了一个时间的上限。
		- **参数** 前两参数跟pcntl_sigwaitinfo一致，$seconds设置等待秒数， $nanoseconds设置等待纳秒数
		- **返回** $return 成功时返回等待到的信号，超时返回-1
		- **说明** 带超时机制的pcntl_sigwaitinfo,可视为它的升级版
---
		
-  **PHP中的信号处理方式**
 1. 传统ticks方式
	 - 所需关键字
		 1. declare
		 2. ticks
	 - 关键字解释
		1. **declare解释** directive也是PHP代码结构的一种，directive 部分允许设定 declare 代码段的行为。目前只认识两个指令：ticks以及 encoding(忽略，不在本章讨论范围)；declare 代码段中的 statement 部分将被执行——怎样执行以及执行中有什么副作用出现取决于 directive 中设定的指令；declare 结构也可用于全局范围，影响到其后的所有代码(但如果有 declare 结构的文件被其它文件包含，则对包含它的父文件不起作用)，directive结构如下:

		``` php
			declare (directive)
				statement
		```
		2. **ticks解释** ticks是directive支持的两个指令中的一个，ticks表示在directive代码段中每执行N条**低级语句(不包含条件表达式和参数表达式)**，就会触发register_tick_function()注册的ticks事件（pcntl_sigmal()注册的函数也会被触发），可能有点懵，看代码吧

		3. 代码示例
			 ```php
				 //注册ticks函数
				register_tick_function(function () {
					var_dump('ticks函数触发');
				});

				---

				//ticks = 1 表示declare的大括号中每执行1条语句
				declare(ticks = 1) {
					$c = 0;
					//此处ticks计算一次
					while ($c) {
						$c --;
						//此处根据循环执行次数计算ticks（当前代码是0次）
					}
					//此处ticks计算一次
				}

				//执行输出如下
				php run.php 
				//string(17) "ticks函数触发"
				//string(17) "ticks函数触发"

				---

				//将$c改为2
				declare(ticks = 1) {
					$c = 2;
					//此处ticks计算1次
					while ($c) {
						$c --;
						//此处计算2次
					}
					//此处ticks计算1次
				}

				//执行输出如下
				php run.php 
				//string(17) "ticks函数触发"
				//string(17) "ticks函数触发"
				//string(17) "ticks函数触发"
				//string(17) "ticks函数触发"
			 ```
		4. 对declare如果还是有疑问，[可翻阅官方文档][2]
	- ticks形式的信号处理方式
		1. 基本原理：用pnctl_signal注册需要处理的信号和其对应的处理函数，进程运行期间每处理制定行数的代码，就去查看有没有注册的信号，有就执行处理函数，没有就继续跑，下面是一个ctrl+c杀不掉的死循环代码示例，信号检测就是通过ticks实现
		```php
			pcntl_signal(SIGINT, function () {
				var_dump('接收到了CTRL+C');
			});

			declare(ticks = 1)
				while (true) {
					sleep(2);
				}

			//终端ctrl+c输出如下
			//^Cstring(18) "接收到了CTRL+C"
		```
		2. 说明: 实际上用ticks是非常古老的处理方式，而且在程序运行期间大部分时间都是没有信号的，就好比ajax轮训一样，这样的处理方式效率是很低下的，不推荐这种方式

2. pcntl_signal_dispatch主动处理
	- 介绍: 由于ticks方式效率的低下, php5.3版本后加入了pcntl_signal_dispatch函数，目的是信号检测由开发者自己处理，这样可以显著的提升效率，但同时开发者需要更加细心去处理,用pcntl_signal_dispatch实现上个例子
	```php
		pcntl_signal(SIGINT, function () {
			var_dump('接收到了CTRL+C');
		});

		while (true) {
			sleep(2);
			//主动检测
			pcntl_signal_dispatch();
		}

	```
	- 说明: pcntl_signal_dispatch虽然提高了信号检测的效率，但是却增加了开发者开发的负担，如果php版本>=7.1,那么这种方式也不推荐
3. pcntl_async_signals异步信号处理
	- 介绍:不论是ticks还是pcntl_signal_dispatch，其处理方式都是阻塞的，他们都依赖信号检测的上一句代码稳妥的运行完毕，但是如果遇到这样一个情况，比如说，有时候网络突然卡死了，或者是写入或者读取一个很大的文件，那这种情况一般需要较长的时间，而这段时间里正好有信号产生了，由于ticks和pcntl_signal_dispatch都是语句执行后处理信号而无法在语句执行中处理信号，那这样就会对这个信号的处理造成很大程度的延时，介于这种情况php7.1后加入了pcntl_async_signals，用pcntl_async_signals实现上个例子

	```php
		pcntl_signal(SIGINT, function () {
			var_dump('接收到了CTRL+C');
		});

		pcntl_async_signals(true);

		while (true) {
			sleep(2);
		}

	```

	- 说明：如果php版本大于7.1，强烈推荐pcntl_async_signals这种方式，相比前两种来说这是目前最完美的信号检测方式



----------

> 如有不妥之处，敬请留言回复以便更正，防止误导他人；漫漫码路，与君共勉！转载请注明

  [1]: https://bugs.php.net/bug.php?id=52121
  [2]: http://www.php.net/manual/zh/control-structures.declare.php[root@izwz960p8wbkrjww8a18i6z 

