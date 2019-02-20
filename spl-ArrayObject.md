---
title: spl-ArrayObject
date: '2019-02-19 23:00'
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
abbrlink: 7817fbb9
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
	- setFlags ( int $flags ) : void
		- 作用
			设置ArrayObject行为
		- $flags
			$flags值为```ArrayObject::STD_PROP_LIST```或```ArrayObject::ARRAY_AS_PROPS```
			 - 当为```ArrayObject::STD_PROP_LIST```时,博主暂时没有理解这个常熟的行为,文档中的解释是错误的,有哪位大佬清楚的话烦请评论告知,万分感谢
			 - 当为```ArrayObject::ARRAY_AS_PROPS```,表示以对象属性形式(->)设置的值将会更新到元素中,比如
				 ```php
				 $array = [
					'item1' => 'hello',
					'item2' => 'world'
				];
				$arrayObject = new ArrayObject($array);
				$arrayObject->setFlags(ArrayObject::ARRAY_AS_PROPS);
				$arrayObject->item3 = 'php';
				$arrayObject['itme4'] = 'spl';
				var_dump($arrayObject->getArrayCopy());

				$newArrayObject = new ArrayObject($array);
				$newArrayObject->item3 = 'php';
				$newArrayObject['itme4'] = 'spl';
				var_dump($newArrayObject->getArrayCopy());
				//output
				array(4) {
				  ["item1"]=>
				  string(5) "hello"
				  ["item2"]=>
				  string(5) "world"
				  ["item3"]=>
				  string(3) "php"
				  ["itme4"]=>
				  string(3) "spl"
				}
				array(3) {
				  ["item1"]=>
				  string(5) "hello"
				  ["item2"]=>
				  string(5) "world"
				  ["itme4"]=>
				  string(3) "spl"
				}

				 ```
		- void
			无返回
	- getFlags ( void ) : int
		- 作用
			得到设置的行为
		- void
			无输入
		- :int
			设置的行为(没有设置=>0或```ArrayObject::STD_PROP_LIST```=>1或```ArrayObject::ARRAY_AS_PROPS```=>2)
	- getIterator ( void ) : ArrayIterator
		- 作用
			从ArrayObject创建一个新的迭代器对象
		- void
			无输入
		- : ArrayIterator
			返回新创建的迭代器对象
	- getIteratorClass ( void ) : string
		- 作用
			返回ArrayObject所实现的迭代器类
		- void
			无输入
		- :string
			返回ArrayObject所实现的迭代器类名称
	- setIteratorClass ( string $iterator_class ) : void
		- 作用
			替换ArrayObject的迭代器类
		- $iterator_class
			合法的迭代器类名称(字符串)
		- :void
			无返回值
	- offsetExists ( mixed $index ) : bool
		- 作用
			判断元素键是否存在
		- $index
			传入的键
		- :bool
			结果true|false
	- offsetGet ( mixed $index ) : mixed
		- 作用
			得到制定键的值
		- $index
			指定的键
		- :mixed
			返回的值
	- offsetSet ( mixed $index , mixed $newval ) : void
		- 作用
			设置一个键,存在则覆盖
		- $index
			键
		- $newval
			值
		- :void
			无返回
	- offsetUnset ( mixed $index ) : void
		- 作用
			删除一个制定的键
		- $index
			删除的键
		- :void
			无返回
	- unserialize ( string $serialized ) : void
		- 这个函数吧,文档没说注释,我尝试了好久也是没找到它存在的意义.
- 应用场景
	ArrayObject一般用于框架的基础结构的数组工具集或Orm的实现中,我们常见的Laravel和Tp5框架,其model是基于ArrayAccess(ArrayObject也实现了它)实现的,而swoole扩展的框架新秀easyswoole其sql库中的SplArray工具就是继承了ArrayObject后实现的,其实我们也可以通过魔术方法```__call()```在ArrayObject的实现中直接去调用现有的所有数组方法都是没问题的,关键还是在于思路和基础知识的掌握程度,由于PHP官方文档的表达程度和一定的错误率导致多数高阶php功能都被雪藏确实挺遗憾的,所以博主建议大家多多研究下文档并动手实践,从而使你对这门语言的掌握程度更加明朗.
	
	----------

> 如有不妥之处，敬请留言回复以便更正，防止误导他人；漫漫码路，与君共勉！转载请注明

