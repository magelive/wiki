<!--
author: Magelive
date: 2016-12-12
title: libuv queue分析
tags: Linux,libuv,queue,list, libuv queue
category: libuv
status: publish
summary: libuv queue使用指针数组的特性，其定义了一个2维的指针数组，利用该指针数组的特性来构建与操作queue. 
-->
libuv queue使用指针数组的特性，其定义了一个2维的指针数组，利用该指针数组的特性来构建与操作queue. 
*libuv 中QUEUE的操作都在src/queue.h中。*

## libuv queue方法分析
```
	typedef void *QUEUE[2];

    # define QUEUE_NEXT(q)   (*(QUEUE **) &((*(q))[0]))
    # define QUEUE_PREV(q)   (*(QUEUE **) &((*(q))[1]))
    # define QUEUE_PREV_NEXT(q)  (QUEUE_NEXT(QUEUE_PREV(q)))
    # define QUEUE_NEXT_PREV(q)  (QUEUE_PREV(QUEUE_NEXT(q)))

	# define QUEUE_DATA(ptr, type, field) \
  		((type *) ((char *) (ptr) - offsetof(type, field)))

	# define QUEUE_INIT(q) \
 		do {  \
			QUEUE_NEXT(q) = (q); \
			QUEUE_PREV(q) = (q); \
		} \
  		while (0)
```
我们来一步一步分析根据其定义typedef void *QUEUE[2]的特性来分析,若有如下定义
```
	QUEUE wq;
	QUEUE_INIT（&wq);
	QUEUE_INIT(&wq1);
	QUEUE_HEAD（&wq);
	QUEUE_ADD（&wq, &wq1);
```
```
wq  	|-------|
		|wq[0]  |
		|-------|
		|wq[1]  |
		|-------|

wq1 	|-------|
		|wq1[0] |
		|-------|
		|wq1[1] |
		|-------|
```
那么QUEUE_NEXT和QUEUE_PREV展开则为：
```
	(*(QUEUE **) &((*(&wq))[0]))
	(*(QUEUE **) &((*(&wq))[1]))
```
其特点则是在\*（取值）运算, 以NEXT为例， （\*（&wq））[0]中存储的是下一个节点的QUEUE的地址，当对取&运算时，则取到了下一个节点地址的地址，再对其地址的地址取值（\*），则能得到下一个节点的真实地址。
 
## libuv中queue操作方法描述与说明
```
	QUEUE_DATA(ptr, type, field)
	//获取queue所在的结构的实际数据值
	//此用方法多用于libuv内部数据
	//例如：
	//已知uv_udp_send_t结构的req指针;
	//已知其内部QUEUE q的地址，求req的地址，则可使用 
	//uv_udp_send_t *req = QUEUE_DATA(q, uv_udp_send_t, queue)来获取 

	QUEUE_FOREACH(q, h)
	/*
	使用q来遍历head queue, q会不停的变换值，如：
	QUEUE_FOREACH(q, h)
	{
		uv_*_t data= QUEUE_DATA(q, uv_*_t, queue);
	}
	*/

	QUEUE_EMPTY(q)
	//判断queue q是否为空，即只有一个节点
	
	QUEUE_HEAD(q)
	//初始化HEAD，跟INIT的功能一致，不过INIT的设置了PREV值
	//QUEUE_EMPTY(QUEUE_HEAD(q)) 为True

	QUEUE_INIT(q)
	//初始化q的值，将q的next和prev均指向自己
	//QUEUE_EMPTY(QUEUE_INIT(q)) 为True

	QUEUE_ADD(h, n)
	//在head的前面插入节点node，即在队列尾加入node
	
	QUEUE_SPLIT(h, q, n)
	//

	QUEUE_MOVE(h, n)
	//从head中移除n至end中所有结点，同时n为新的queue中的head

	QUEUE_INSERT_HEAD(h, q)
	//在head后面插入节点q

	QUEUE_INSERT_TAIL(h, q)
	//在head前面，也就是队尾插入节点q

	QUEUE_REMOVE(q)
	//删除节点q
	
	// 
	
```