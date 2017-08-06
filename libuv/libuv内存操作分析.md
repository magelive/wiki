<!--
author: Magelive
date: 2016-12-12
title: libUV 内存管理策略分析
tags: Linux,libuv, memery,string,malloc,free
category: libuv
status: publish
summary: libuv使用第三方的内存管理方法来进行内存管理，其自身并没有内存管理方法，但其提供接口，可供用户调用第三方的方法来操作与管理内存。libuv内存操作的代码在uv-common.h和uv-common.c中。
-->

libuv使用第三方的内存管理方法来进行内存管理，其自身并没有内存管理方法，但其提供接口，可供用户调用第三方的方法来操作与管理内存。libuv内存操作的代码在uv-common.h和uv-common.c中。

## libuv内存操作方法介绍
    void* uv__malloc(size_t size)
    void uv__free(void* ptr)
    void* uv__calloc(size_t count, size_t size)
	void* uv__realloc(void* ptr, size_t size)

	int uv_replace_allocator(uv_malloc_func malloc_func,
								uv_realloc_func realloc_func,
								uv_calloc_func calloc_func,
								uv_free_func free_func)
libuv中的内存操作与Linux系统的内存操作方法大概一致，提供malloc、calloc、和realloc三个内存分配方法和free操作方法，而uv_replace_allocator则是为了更换以上四种方法的接口。

## libuv内存接口的实现说明
    typedef void* (*uv_malloc_func)(size_t size);
    typedef void* (*uv_realloc_func)(void* ptr, size_t size);
    typedef void* (*uv_calloc_func)(size_t count, size_t size);
    typedef void (*uv_free_func)(void* ptr);

	typedef struct {
  		uv_malloc_func local_malloc;
  		uv_realloc_func local_realloc;
  		uv_calloc_func local_calloc;
  		uv_free_func local_free;
	} uv__allocator_t;

	static uv__allocator_t uv__allocator = { 
  		malloc,
  		realloc,
  		calloc,
  		free,
	};

	void* uv__malloc(size_t size) {
  		return uv__allocator.local_malloc(size);
	}
	
	void uv__free(void* ptr) {
  		int saved_errno; 
  		saved_errno = errno;
  		uv__allocator.local_free(ptr);
		errno = saved_errno;
	}

	void* uv__calloc(size_t count, size_t size) {
		return uv__allocator.local_calloc(count, size);
	}

	void* uv__realloc(void* ptr, size_t size) {
		return uv__allocator.local_realloc(ptr, size);
	}

	int uv_replace_allocator(uv_malloc_func malloc_func,
	                         uv_realloc_func realloc_func,
	                         uv_calloc_func calloc_func,
	                         uv_free_func free_func)
	{
		if (malloc_func == NULL || realloc_func == NULL ||
			calloc_func == NULL || free_func == NULL) {
			return UV_EINVAL;
		}
	
		uv__allocator.local_malloc = malloc_func;
		uv__allocator.local_realloc = realloc_func;
		uv__allocator.local_calloc = calloc_func;
		uv__allocator.local_free = free_func;
		return 0;
	}

## libuv内存操作范例

假设我们已经实现一个内存操作池，内存池中已经实现了malloc、realloc、calloc及free的功能，全他们的函数名分别为, mempool_malloc、mempool_realloc、mempool_calloc、mempool_free。
那么我们想在libuv使用此内存池来管理libuv中的内存部分，则可如下使用：

    uv_replace_allocator（mempool_malloc, mempool_realloc, 
    						mempool_calloc, mempool_free);

在使用时，则直接使用uv__malloc、uv__realloc、uv__calloc和uv__free即可。
