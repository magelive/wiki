<!--
author: Magelive
date: 2016-12-12
title: libuv uv_loop_t 分析
tags: Linux, libuv
category: libuv
status: publish
summary: uv_loop_t是libuv中最基础的结构，它需要轮询I/O调度回调是运行基于不同来源的事件。
-->
 uv_loop_t是libuv中最基础的结构，它需要轮询I/O调度回调是运行基于不同来源的事件。
## uv_loop_t结构
```c
typedef struct uv_loop_s uv_loop_t;
struct uv_loop_s {
  /* User data - use this for whatever. */
  void* data;

  /* Loop reference counting. */
  unsigned int active_handles;
  void* handle_queue[2];
  void* active_reqs[2];

  /* Internal flag to signal loop stop. */
  unsigned int stop_flag;

  UV_LOOP_PRIVATE_FIELDS

};

/* Linux下结构 */
# define UV_LOOP_PRIVATE_FIELDS	\
	unsigned long flags;	\
	int backend_fd;	\
	void* pending_queue[2];	\
	void* watcher_queue[2];	\
	uv__io_t** watchers;	\
	unsigned int nwatchers;	\
	unsigned int nfds;	\
	void* wq[2];	\
	uv_mutex_t wq_mutex;	\
	uv_async_t wq_async;	\
	uv_rwlock_t cloexec_lock;	\
	uv_handle_t* closing_handles;	\
	void* process_handles[2];	\
	void* prepare_handles[2];	\
	void* check_handles[2];	\
	void* idle_handles[2];	\
	void* async_handles[2];	\
	struct uv__async async_watcher;	\
	struct{	\
	void* min;	\
    unsigned int nelts;	\
	} timer_heap;	\
	uint64_t timer_counter;	\
	uint64_t time;	\
	/*time : 从开机至此刻的精确秒数 */
	int signal_pipefd[2];	\
	uv__io_t signal_io_watcher;	\
	uv_signal_t child_watcher;	\
	int emfile_fd;	\
	UV_PLATFORM_LOOP_FIELDS	\

# ifndef UV_PLATFORM_LOOP_FIELDS
#  define UV_PLATFORM_LOOP_FIELDS /* empty */
# endif
	
```

```c
struct uv__async {
	uv__async_cb cb;
	uv__io_t io_watcher;
	int wfd;
};
```

```c
typedef struct uv__io_s uv__io_t;

struct uv__io_s {
	uv__io_cb cb;
	void* pending_queue[2];
	void* watcher_queue[2];
	unsigned int pevents; /* Pending event mask i.e. mask at next tick. */
	unsigned int events;  /* Current event mask. */
	int fd;
	UV_IO_PRIVATE_PLATFORM_FIELDS
};

# ifndef UV_IO_PRIVATE_PLATFORM_FIELDS
#  define UV_IO_PRIVATE_PLATFORM_FIELDS /* empty */
# endif

```

## 参考
[uv_loop_t — Event loop](http://docs.libuv.org/en/v1.x/loop.html# c.uv_run)
[uv_handle_t](http://docs.libuv.org/en/latest/handle.html)
