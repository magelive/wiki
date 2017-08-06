<!--
author: Magelive
date: 2016-12-12
title: libuv中线程实现分析（基于Linux平台）
tags: Linux,libuv, thread
category: libuv
status: publish
summary: libuv中的线程通常在libuv 内部使用, 用于模拟系统调用的异步特性。libuv 同样可以利用线程让你异步完成一项本可能阻塞的任务, 通常是创建子线程, 在子线程完成任务后获取其结果。
-->
libuv中的线程通常在libuv 内部使用, 用于模拟系统调用的异步特性。libuv 同样可以利用线程让你异步完成一项本可能阻塞的任务, 通常是创建子线程, 在子线程完成任务后获取其结果。

目前存在两个主流的线程库, Windows 线程库实现和 pthreads. libuv 的线程库 API 与 pthread API 相似, 因此两者具有相同的语义。

libuv的线程API也非常有限, 因为不同平台上线程库的语义和语法也不同, API功能的完善程度也不尽相同。

## libuv线程接口介绍
因为libuv是跨平台的网络基础库，它包含所有平台的差异性。所以其不能使用各操作系统的线程API接口，而是使用define及macro来进行编译时选择与封装。以下为Linux平台中的定义：
### libuv基本数据介绍
```
//Linux 平台上的具体实现
typedef pthread_once_t uv_once_t;
typedef pthread_t uv_thread_t;
typedef pthread_mutex_t uv_mutex_t;
typedef pthread_rwlock_t uv_rwlock_t;
typedef UV_PLATFORM_SEM_T uv_sem_t;
typedef pthread_cond_t uv_cond_t;
typedef pthread_key_t uv_key_t;
typedef pthread_barrier_t uv_barrier_t;
# ifndef UV_PLATFORM_SEM_T
	# define UV_PLATFORM_SEM_T sem_t
# endif
```

### libuv线程API实现分析
libuv中的线程，
```c
/*
	创建线程
*/
int uv_thread_create(uv_thread_t *tid, void (*entry)(void *arg), void *arg)｛

｝ 

uv_thread_t uv_thread_self(void);
/*
	获取线程ID
*/

int uv_thread_join(uv_thread_t *tid);
/*
	线程等待
*/

int uv_thread_equal(const uv_thread_t* t1, const uv_thread_t* t2);
/*
	判断是否为同一线程
*/

int uv_mutex_init(uv_mutex_t* mutex);
/*
	线程互斥锁初始化
*/

void uv_mutex_destroy(uv_mutex_t* mutex);
/*
	互斥锁销毁
*/

void uv_mutex_lock(uv_mutex_t* mutex);
/*
	互斥锁上锁
*/

int uv_mutex_trylock(uv_mutex_t* mutex);
/*
	互斥锁try lock
*/

void uv_mutex_unlock(uv_mutex_t* mutex);
/*
*/

int uv_rwlock_init(uv_rwlock_t* rwlock);

void uv_rwlock_destroy(uv_rwlock_t* rwlock);

void uv_rwlock_rdlock(uv_rwlock_t* rwlock);

int uv_rwlock_tryrdlock(uv_rwlock_t* rwlock);

void uv_rwlock_rdunlock(uv_rwlock_t* rwlock);

void uv_rwlock_wrlock(uv_rwlock_t* rwlock);

int uv_rwlock_trywrlock(uv_rwlock_t* rwlock);

void uv_rwlock_wrunlock(uv_rwlock_t* rwlock);

void uv_once(uv_once_t* guard, void (*callback)(void));

int uv_sem_init(uv_sem_t* sem, unsigned int value);


void uv_sem_destroy(uv_sem_t* sem);


void uv_sem_post(uv_sem_t* sem);

void uv_sem_wait(uv_sem_t* sem);

int uv_sem_trywait(uv_sem_t* sem); 

int uv_cond_init(uv_cond_t* cond);

void uv_cond_destroy(uv_cond_t* cond);

void uv_cond_signal(uv_cond_t* cond);

void uv_cond_broadcast(uv_cond_t* cond);

void uv_cond_wait(uv_cond_t* cond, uv_mutex_t* mutex);


int uv_cond_timedwait(uv_cond_t* cond, uv_mutex_t* mutex, uint64_t timeout);

void uv_barrier_destroy(uv_barrier_t* barrier);

int uv_barrier_wait(uv_barrier_t* barrier);

int uv_key_create(uv_key_t* key);

void uv_key_delete(uv_key_t* key);

void* uv_key_get(uv_key_t* key);

void uv_key_set(uv_key_t* key, void* value);

```