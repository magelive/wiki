<!--
author: Magelive
date: 2017-02-12
title: json-c中array_list介绍
tags:json libjson json-c array_list 
category: json-c
status: publish
summary: 
head: 
images: 
-->

<!-- # json-c源码分析——array_list分析 -->
json-c中array_list是用来存储json中的数组结构，其在json中的组织结构为：
```
["a", "b", "c"]
```
json-c中array_list数据操作的源文件为arraylist.c和arraylist.h

## array_list数据结构分析
```
struct array_list
{
	void **array;
	size_t length;
	size_t size;
	array_list_free_fn *free_fn;
};
```
### 详细说明
- array 数据存储地址
- length 数据长度，初始化为0
- size 数据长度，初始化为32,默认增长为其的2倍
- free_fn 数据空间释放函数指针

## array_list操作函数
### 1、New  free and expand
```
struct array_list* array_list_new(array_list_free_fn *free_fn)
{
	struct array_list *arr;
	arr = (struct array_list*)calloc(1, sizeof(struct array_list));
	if(!arr) return NULL;
	arr->size = ARRAY_LIST_DEFAULT_SIZE;
	arr->length = 0;
	arr->free_fn = free_fn;
	if(!(arr->array = (void**)calloc(sizeof(void*), arr->size))) {
  		free(arr);
 		return NULL;
	}
  	return arr;
}

extern void array_list_free(struct array_list *arr)
{
	size_t i;
	for(i = 0; i < arr->length; i++)
		if(arr->array[i]) arr->free_fn(arr->array[i]);
	free(arr->array);
	free(arr);
}

static int array_list_expand_internal(struct array_list *arr, size_t max)
{
	void *t;
	size_t new_size;

	if(max < arr->size) return 0;
	/* Avoid undefined behaviour on size_t overflow */
	if( arr->size >= SIZE_T_MAX / 2 )
		new_size = max;
	else
	{
 		new_size = arr->size << 1;
		if (new_size < max)
		new_size = max;
	}
	if (new_size > (~((size_t)0)) / sizeof(void*)) return -1;
	if (!(t = realloc(arr->array, new_size*sizeof(void*)))) return -1;
	arr->array = (void**)t;
	(void)memset(arr->array + arr->size, 0, (new_size-arr->size)*sizeof(void*));
	arr->size = new_size;
	return 0;
}

```
其重点在于\*\*array, array中存储的是\*array[]结构，其array[ARRAY_LIST_DEFAULT_SIZE]中存储指针地址，其指针指向具体的存储数据。
expand则是将array的空间扩展至其的两倍，如果不够，扩展至max参数。

### 2、	Put get add and del
```
void* array_list_get_idx(struct array_list *arr, size_t i)
{
	if(i >= arr->length) return NULL;
	return arr->array[i];
}

int array_list_put_idx(struct array_list *arr, size_t idx, void *data)
{
	if (idx > SIZE_T_MAX - 1 ) return -1;
	if(array_list_expand_internal(arr, idx+1)) return -1;
	if(arr->array[idx]) arr->free_fn(arr->array[idx]);
	arr->array[idx] = data;
	if(arr->length <= idx) arr->length = idx + 1;
	return 0;
}

int array_list_add(struct array_list *arr, void *data)
{
	return array_list_put_idx(arr, arr->length, data);
}

int array_list_del_idx( struct array_list *arr, size_t idx, size_t count )
{
	size_t i, stop;
	stop = idx + count;
	if ( idx >= arr->length || stop > arr->length ) return -1;
	for ( i = idx; i < stop; ++i ) {
		if ( arr->array[i] ) arr->free_fn( arr->array[i] );
	}
	memmove( arr->array + idx, arr->array + stop, (arr->length - stop) * sizeof(void*) );
	arr->length -= count;
	return 0;
}
```
- get函数可以根据idx获取array_list中idx位置的数据
- put函数则是将数据存储至array_list中的idx位置中
- add则是将数据添加至最后面
- del则是位于idx位置的数据移除
### 3、Sort length and bsearch
```
void array_list_sort(struct array_list *arr, int(*sort_fn)(const void *, const void *))
{
	qsort(arr->array, arr->length, sizeof(arr->array[0]), sort_fn);
}

void* array_list_bsearch(const void **key, struct array_list *arr,
		int (*sort_fn)(const void *, const void *))
{
	return bsearch(key, arr->array, arr->length, sizeof(arr->array[0]),
			sort_fn);
}

size_t array_list_length(struct array_list *arr)
{
	return arr->length;
}

```
- sort	
- bsearch
- length
