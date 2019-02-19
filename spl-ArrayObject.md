---
title: spl-ArrayObject
date: 2019-01-13 21:29
keywords:
 - php
 - spl
 - ArrayObject
description: php标准库SPL之ArrayObject
categories:
 - php
 - spl
tags: 
 - php
 - spl
---

- **SPL是什么?**
	SPL(Standard PHP Library)标准PHP类库,是用于解决典型问题(standard problems)的一组接口与类的集合。SPL属于PHP中较高层级的内容并且SPL在实际开发当中用的人较少,究其原因,应该跟PHP万能的数组有关系,从而导致多数PHPer温水煮青蛙,也有人说跟简白的文档有关系,仁者见仁,不过怎么说,SPL从PHP5开始加入到现在已经相当成熟,并且SPL相当之好用,对提升你代码水平和程序性能有显著的效果

- ArrayObject
	ArrayObject是Spl中的一个工具类,文档解释(This class allows objects to work as arrays.)意思是将对象以数组方式工作,乍一看似乎有点晕,这样理解:ArrayObject将一个数组或对象转化为了一个特定的对象(我们暂且叫他数组对象),这个对象实现了迭代丶数组访问丶序列化和Countable,可以使你非常简洁的以对象方式对传入的数组或对象进行统一化操作,比如平时对数组的常规操作可以用对象方式很统一且简洁的方式实现:
	```php
	$array = [
		8, 9, 6, 3, 4
	];
	//创建对象
	$arrayObject = new ArrayObject($array);
	//求count
	$arrayObject->count();
	//序列化数组
	$arrayObject->serialize();
	//追加值
	$arrayObject->append(200);
	//正排列
	$arrayObject->asort();
	```
	- 构造方法 ([ mixed $input = array() [, int $flags = 0 [, string $iterator_class = "ArrayIterator" ]]] )
		- $input
			待传入的数组或对象
		- $flags
			行为标记,比如当foreach对象时应该做什么等等,后面setFlag()会说到
		- $iterator_class
			迭代器类, 制定迭代当前对象时得类
	- append ( mixed $value ) : void
		- 作用
			追加新的值作为最后一个元素
		- $value
			所追加的值
		- :void
			无返回值
	- asort ( void ) : void
		- 作用
			对值进行正序排列并保持值键的关联关系不变
		- void
			无输入
		- :void
			无返回,直接操作元素
	- ksort ( void ) : void
		- 作用
			对键进行正序排列并保持值键的关联关系不变
		- void
			无输入
		- :void
			无返回
	- uasort ( callable $cmp_function ) : void
		- 作用
			基于用户自定义的排序函数对值进行正序排列并保持值键的关联关系不变
		- $cmp_function
			callable形式的自定义函数
		- :void
			无返回
	- uksort ( callable $cmp_function ) : void
		- 作用
			基于用户自定义的排序函数对键进行正序排列并保持值键的关联关系不变
		- $cmp_function
			callable形式的自定义函数
		- :void
			无返回
	- natsort ( void ) : void
		- 作用
			使用"自然顺序"算法对数组进行排序
		- void
			无输入
		- :void
			无返回	
	- natcasesort ( void ) : void
		- 作用
			使用"自然顺序"算法并不区分大小写对数组进行排序
		- void
			无输入
		- :void
			无返回	
	- count ( void ) : int
		- 作用
			统计 ArrayObject 内 public 属性的数量,如果不自己实现ArrayObject的话此方法统计的就是元素数量
		- void
			无输入
		- :int
			返回统计的数量	
	- serialize ( void ) : string
		- 作用
			序列化元素并输出
		- void
			无输入
		- :string
			返回序列化后的值
	- exchangeArray ( mixed $input ) : array
		- 作用
			以当前输入替换ArrayObject中存储的元素
		- $input
			新的数组或对象
		- :array
			以数组方式返回旧的元素
	- getArrayCopy ( void ) : array
		- 作用
			将传入的元素转化为数组
		- void
			无输入
		- :array
			返回转化的数组
	