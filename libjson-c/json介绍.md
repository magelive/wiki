<!--
author: Magelive
date: 2017-02-10
title: 
tags: 
category: libjsonC
status: publish
summary: 
head: 
images: 
-->

## JSON数据格式
JSON(JavaScript Object Notation) 是一种轻量级的数据交换格式。JSON采用完全独立于语言的文本格式，这些特性使JSON成为理想的数据交换语言。易于人阅读和编写，同时也易于机器解析和生成。
### 基础结构
JSON建构于两种结构：
1. “名称/值”对的集合（A collection of name/value pairs）。不同的语言中，它被理解为对象（object），记录（record），结构（struct），字典（dictionary），哈希表（hash table），有键列表（keyed list），或者关联数组 （associative array）。

2. 值的有序列表（An ordered list of values）。在大部分语言中，它被理解为数组（array）。

### Json对象
* **数值**：可为整型或float型数据，请注意数值不需要加引号
* **字符串**:字符串（string）是由双引号包围的任意数量Unicode字符的集合，使用反斜线转义。一个字符（character）即一个单独的字符串（character string）。 
* **布尔**：true或false。请注意JSON格式中的值可以使用布尔类型，且不需要加引号，如果true或false被引号包裹，那么便解析为JSON字符串，请处理稍有不同。
* **字典（“名称/值”对的集合）**:即Key/value对的集合，以大括号（｛｝）括起来的数据集。以“{”（左括号）开始，“}”（右括号）结束。每个“名称”后跟一个“:”（冒号）；“‘名称/值’ 对”之间使用“,”（逗号）分隔。
* **数组**：即列表数据值，以中括号（[])括起来的数据集，在Json中，不同类型的数据可组织成同一数据，如：｛["aa", 12, true]｝。一个数组以“[”（左中括号）开始，“]”（右中括号）结束。值之间使用“,”（逗号）分隔。
* **null**:null对象，不同的程序不同的处理方式，在JavaScript及Java中，如果传进去的的null,即不带引号，则处理成空，若传进去的值带双绰号，则处理成null。如：｛key, null｝=>{}, {key, "null"}=>{key, null}。*特别注意的是，不建议使用此值。*
 
### 示例
简单地说，JSON 可以将 JavaScript 对象中表示的一组数据转换为字符串，然后就可以在函数之间轻松地传递这个字符串，或者在异步应用程序中将字符串从 Web 客户机传递给服务器端程序。这个字符串看起来有点儿古怪，但是 JavaScript 很容易解释它，而且 JSON 可以表示比"名称 / 值对"更复杂的结构。例如，可以表示数组和复杂的对象，而不仅仅是键和值的简单列表。

#### 表示名称 / 值对
按照最简单的形式，可以用下面这样的 JSON 表示 "名称 / 值对" ：{ "firstName": "Brett" }
这个示例非常基本，而且实际上比等效的纯文本 "名称 / 值对" 占用更多的空间：firstName=Brett
但是，当将多个"名称 / 值对"串在一起时，JSON 就会体现出它的价值了。首先，可以创建包含多个"名称 / 值对"的 记录，比如：
```json
{ "firstName": "Brett", "lastName":"McLaughlin", "email": "aaaa" }
```

从语法方面来看，这与"名称 / 值对"相比并没有很大的优势，但是在这种情况下 JSON 更容易使用，而且可读性更好。例如，它明确地表示以上三个值都是同一记录的一部分；花括号使这些值有了某种联系。

#### 表示数组
当需要表示一组值时，JSON 不但能够提高可读性，而且可以减少复杂性。例如，假设您希望表示一个人名列表。在 XML 中，需要许多开始标记和结束标记；如果使用典型的 名称 / 值 对（就像在本系列前面文章中看到的那种名称 / 值对），那么必须建立一种专有的数据格式，或者将键名称修改为 person1-firstName这样的形式。
如果使用 JSON，就只需将多个带花括号的记录分组在一起：
```
{
	"people":
			[
				{ "firstName": "Brett", "lastName":"McLaughlin", "email": "aaaa" },
				{ "firstName": "Jason", "lastName":"Hunter", "email": "bbbb"},
				{ "firstName": "Elliotte", "lastName":"Harold", "email": "cccc" }
			]
}
```
这不难理解。在这个示例中，只有一个名为 people的变量，值是包含三个条目的数组，每个条目是一个人的记录，其中包含名、姓和电子邮件地址。上面的示例演示如何用括号将记录组合成一个值。当然，可以使用相同的语法表示多个值（每个值包含多个记录）：
```
{ 
	"programmers": 
				[
					{ "firstName": "Brett", "lastName":"McLaughlin", "email": "aaaa" },
					{ "firstName": "Jason", "lastName":"Hunter", "email": "bbbb" },
					{ "firstName": "Elliotte", "lastName":"Harold", "email": "cccc" }
				],

	"authors": 
			[
				{ "firstName": "Isaac", "lastName": "Asimov", "genre": "science fiction" },
				{ "firstName": "Tad", "lastName": "Williams", "genre": "fantasy" },
				{ "firstName": "Frank", "lastName": "Peretti", "genre": "christian fiction" }
			],

	"musicians": 
			[
				{ "firstName": "Eric", "lastName": "Clapton", "instrument": "guitar" },	
				{ "firstName": "Sergei", "lastName": "Rachmaninoff", "instrument": "piano" }
			]
}
```
这里最值得注意的是，能够表示多个值，每个值进而包含多个值。但是还应该注意，在不同的主条目（programmers、authors 和 musicians）之间，记录中实际的名称 / 值对可以不一样。JSON 是完全动态的，允许在 JSON 结构的中间改变表示数据的方式。

在处理 JSON 格式的数据时，没有需要遵守的预定义的约束。所以，在同样的数据结构中，可以改变表示数据的方式，甚至可以以不同方式表示同一事物。

### JSON和XML的比较

* 可读性
	JSON和XML的可读性可谓不相上下，一边是简易的语法，一边是规范的标签形式，很难分出胜负。

* 可扩展性
	XML天生有很好的扩展性，JSON当然也有，没有什么是XML能扩展，而JSON却不能。不过JSON在Javascript主场作战，可以存储Javascript复合对象，有着xml不可比拟的优势。

* 编码难度
	XML有丰富的编码工具，比如Dom4j、JDom等，JSON也有提供的工具。无工具的情况下，相信熟练的开发人员一样能很快的写出想要的xml文档和JSON字符串，不过，xml文档要多很多结构上的字符。

* 解码难度
	XML的解析方式有两种：
	一是通过文档模型解析，也就是通过父标签索引出一组标记。例如：xmlData.getElementsByTagName("tagName")，但是这样是要在预先知道文档结构的情况下使用，无法进行通用的封装。
	另外一种方法是遍历节点（document 以及 childNodes）。这个可以通过递归来实现，不过解析出来的数据仍旧是形式各异，往往也不能满足预先的要求。

	凡是这样可扩展的结构数据解析起来一定都很困难。

	JSON也同样如此。如果预先知道JSON结构的情况下，使用JSON进行数据传递简直是太美妙了，可以写出很实用美观可读性强的代码。如果你是纯粹的前台开发人员，一定会非常喜欢JSON。但是如果你是一个应用开发人员，就不是那么喜欢了，毕竟xml才是真正的结构化标记语言，用于进行数据传递。

	而如果不知道JSON的结构而去解析JSON的话，那简直是噩梦。费时费力不说，代码也会变得冗余拖沓，得到的结果也不尽人意。但是这样也不影响众多前台开发人员选择JSON。因为json.js中的toJSONString()就可以看到JSON的字符串结构。当然不是使用这个字符串，这样仍旧是噩梦。常用JSON的人看到这个字符串之后，就对JSON的结构很明了了，就更容易的操作JSON。

	以上是在Javascript中仅对于数据传递的xml与JSON的解析。在Javascript地盘内，JSON毕竟是主场作战，其优势当然要远远优越于xml。如果JSON中存储Javascript复合对象，而且不知道其结构的话，我相信很多程序员也一样是哭着解析JSON的。

* 实例比较

	XML和JSON都使用结构化方法来标记数据，下面来做一个简单的比较。

	用XML表示中国部分省市数据如下：
	```
	<?xml version="1.0" encoding="utf-8"?>	
	<country>	
	    <name>中国</name>	
	    <province>	
	        <name>黑龙江</name>	
	     <cities>	
	            <city>哈尔滨</city>	
	            <city>大庆</city>	
	        </cities>	
	    </province>	
	    <province>	
	        <name>广东</name>	
	        <cities>
	            <city>广州</city>	
	            <city>深圳</city>	
	            <city>珠海</city>	
	        </cities>	
	    </province>	
	</country>
	```
	用JSON表示如下：
	```
	{	
		name:"中国", 
		province:
				[ 
					{
						name:"黑龙江",
						cities:	{city:["哈尔滨","大庆"]}
					},	
					{
						name:"广东",
						cities:{ city:["广州","深圳","珠海"]}
					}
				]
	}
	```

编码的可读性，xml有明显的优势，毕竟人类的语言更贴近这样的说明结构。json读起来更像一个数据块，读起来就比较费解了。不过，我们读起来费解的语言，恰恰是适合机器阅读，所以通过json的索引.province[0].name就能够读取“黑龙江”这个值。

编码的手写难度来说，xml还是舒服一些，好读当然就好写。不过写出来的字符JSON就明显少很多。去掉空白制表以及换行的话，JSON就是密密麻麻的有用数据，而xml却包含很多重复的标记字符。

### 参考与引用
>[JSON 数据格式](http://www.cnblogs.com/SkySoot/archive/2012/04/17/2453010.html)
>[前端学习——JSON格式详解](http://blog.csdn.net/xukai871105/article/details/32346797)