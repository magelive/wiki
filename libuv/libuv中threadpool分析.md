<!--
author: Magelive
date: 2016-12-12
title: libuv中threadpool分析
tags: Linux, libuv, threadpool, pool, thread
category: libuv
status: publish
summary: 
-->
libuv中的线程池是以
## libuv中的线程封装分析（基于Linux）
因为libuv是跨平台的网络基础库，它包含所有平台的差异性。所以其不能使用各操作系统的线程API接口，而是使用define及macro来进行编译时选择与封装。以下为Linux平台中的定义：
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
#  define UV_PLATFORM_SEM_T sem_t
# endif
```

```

void uv_mutex_lock(uv_mutex_t* mutex) {
  if (pthread_mutex_lock(mutex))
    abort();
}

```

## threadpool中数据结构分析
### threadpool全局变量
```
//libuv抽象
static uv_once_t once = UV_ONCE_INIT;
static uv_cond_t cond;
static uv_mutex_t mutex;
static unsigned int idle_threads;
static unsigned int nthreads;
static uv_thread_t* threads;
static uv_thread_t default_threads[4];
static QUEUE exit_message;
static QUEUE wq; 
static volatile int initialized;
void uv_mutex_lock(uv_mutex_t* mutex);
```
### struct uv__work
```
struct uv__work {
  void (*work)(struct uv__work *w);
  void (*done)(struct uv__work *w, int status);
  struct uv_loop_s* loop;
  void* wq[2];
};
```


## threadpool中Work函数分析
```
static void worker(void* arg) {
  struct uv__work* w;
  QUEUE* q;

  (void) arg;

  for (;;) {
    uv_mutex_lock(&mutex);

    while (QUEUE_EMPTY(&wq)) {
      idle_threads += 1;
      uv_cond_wait(&cond, &mutex);
      idle_threads -= 1;
    }

    q = QUEUE_HEAD(&wq);

    if (q == &exit_message)
      uv_cond_signal(&cond);
    else {
      QUEUE_REMOVE(q);
      QUEUE_INIT(q);  /* Signal uv_cancel() that the work req is
                             executing. */
    }

    uv_mutex_unlock(&mutex);

    if (q == &exit_message)
      break;

    w = QUEUE_DATA(q, struct uv__work, wq);
    w->work(w);

    uv_mutex_lock(&w->loop->wq_mutex);
    w->work = NULL;  /* Signal uv_cancel() that the work req is done
                        executing. */
    QUEUE_INSERT_TAIL(&w->loop->wq, &w->wq);
    uv_async_send(&w->loop->wq_async);
    uv_mutex_unlock(&w->loop->wq_mutex);
  }
}

```

