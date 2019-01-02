## libevt简介
一个 C 编写的功能全面的高性能事件循环库。
libev是一个event loop：事件驱动的库。通过向libev注册感兴趣的events，比如socket可读事件，libev会对所注册的事件的源进行管理，并在事件发生时触发相应的回调函数。

通过event watcher来注册事件。libev通过分配和注册watcher对不同类型的事件进行监听。
不同事件类型的watcher又对应不同的数据类型，watcher的定义模式是struct ev_TYPE或者ev_TYPE，

Libev 支持文件描述符事件的 select，poll，Linux 特有的 epoll，BSD 特有的 kqueue 以及 Solaris 特有的事件端口机制 (`ev_io`)，Linux 的 inotify 接口 (`ev_stat`)，Linux eventfd/signalfd（用于更快更干净的线程间唤醒 (`ev_async`)/信号处理 (`ev_signal`)），相对定时器 (`ev_timer`)，定制重新调度逻辑的绝对定时器 (`ev_periodic`)，同步的信号 (`ev_signal`)，进程状态变化事件 (`ev_child`)，以及处理事件循环机制自身的事件观察者 (`ev_idle`，`ev_embed`，`ev_prepare` 和 `ev_check` 观察者) 和文件观察者 (`ev_stat`)，甚至是对 fork 事件的有限支持 (`ev_fork`)。


### watcher

- `ev_io`：支持 Linux 的select、poll、epoll；BSD 的kqueue；Solaris 的event port mechanisms
- `ev_signal`：支持各种信号处理、同步信号处理
- `ev_timer`：相对事件处理
- `ev_periodic`：排程时间表
- `ev_child`：进程状态变化事件
- `ev_stat`：监视文件状态
- `ev_fork`：有限的fork事件支持
- `ev_idle`：
- `ev_embed`：
- `ev_prepare`：
- `ev_check`：
- `ev_async`:
- `ev_cleanup`:
- `ev_prepare` and `ev_check`

`ev_init` 对一个watcher的与具体类型无关的部分进行初始化。
`ev_io_set` 对watcher的与io类型相关的部分进行初始化，如果是TYPE类型那么相应的函数就是`ev_TYPE_set`。
可以采用`ev_TYPE_init`函数来替代`ev_init`和`ev_TYPE_set`。`ev_io_start` 激活相应的watcher，watcher只有被激活的时候才能接收事件。`ev_io_stop`停止已经激活的watcher。


### functions

#### glob functions
这些函数可以随时调用，甚至在以任何方式初始化库之前。

##### ev_time
`ev_tstamp ev_time ()`
返回以 libev 所使用的格式的当前时间。注意 `ev_now`函数通常更快，且也常常返回你实际想知道的时间戳。
`ev_now_update` 和 `ev_now` 的结合也很有意思。

Libev 使用一个`ev_tstamp`数据类型来表示1970年以来的秒数，实际类型是 C 里面的double类型。

##### ev_sleep
`void ev_sleep (ev_tstamp interval)`
休眠给定的时间：当前线程将阻塞，直到它被中断或经过了给定的时间间隔（大约 - 即使不中断，它可能也会早返回一点）。
如果 interval <= 0 就立即返回。基本上这是一个粒度比秒更高的 sleep()。
interval 的范围是有限的 - libev 只保证最长一天 (interval <= 86400) 的休眠时间是可以工作的。

##### version
```
int ev_version_major ()
int ev_version_minor ()
```
获取当前链接库的版本信息，指的是运行时库的版本信息。
可以调用这两个函数，并且与系统定义的```EV_VERSION_MAJOR```和```EV_VERSION_MINOR```作对比，判断是否应该支持该库。
我们可以使用如下代码来确保我们没有被无意地链接到错误的版本。
```
assert (("libev version mismatch",
         ev_version_major () == EV_VERSION_MAJOR
         && ev_version_minor () >= EV_VERSION_MINOR));
```

##### supported backends
```
unsigned int ev_supported_backends ();
unsigned int ev_recommand_backends ();
unsigned int ev_embeddable_backends ();
```
`ev_supported_backends`返回编译进 libev 二进制（独立于你正在运行的系统上它们的可用性）中的所有后端的集合（比如，它们的对应 `EV_BACKEND_*` 值）。参考```ev_default_loop```获得这些值的描述。
例如：我们在代码中为了确认系统存在`epoll`方法，我们可以使用如下检测方法：
```
assert (("sorry, no epoll, no sex",
         ev_supported_backends () & EVBACKEND_EPOLL));
```
`ev_recommended_backends`返回编译进 libev 二进制文件且建议本平台使用的所有后端的集合，意味着它将可以用于大多数的文件描述符类型。这个集合通常比 ev_supported_backends 返回的要小，例如，大多数 BSD 上的 kqueue 都不会使用，除非你明确要求（假设你知道你在做什么），否则不会自动检测。如果你没有显式地指定，这是 libev 将探测的后端集合。

`ev_embeddable_backends`返回其它事件循环中可嵌入的后端的集合。这个值是平台特有的，但可以包含当前系统不可用的后端。为了找出当前系统可能支持的可嵌入后端，你需要查看 `ev_embeddable_backends () & ev_supported_backends ()`，同样的建议采用的那些。参考 `ev_embed` 观察者的描述来获得更多信息。

##### ev_set_allocator
`ev_set_allocator (void (cb)(void *ptr, long size) throw ())`
重新设置realloc函数。对于一些系统（至少包括 BSD 和 Darwin）的 realloc 函数可能不正确，libev 已经给了替代方案。
你可以在高可用性程序中覆盖这个函数，比如，如果它无法分配内存就释放一些内存，使用一个特殊的分配器，或者甚至是休眠一会儿并重试直到有内存可用。
例如：用一个等待一会儿并重试的分配器替换 libev 分配器（例子需要一个与标准兼容的`realloc`）。
```
static void *persistent_realloc (void *ptr, size_t size)
{
  while (1) {
      void *newptr = realloc (ptr, size);
      if (newptr)
        return newptr;
   	  sleep (1);
    }
}

. . .
ev_set_allocator (persistent_realloc);
```

##### ev_set_syserr_cb
`ev_set_syserr_cb (void (*cb)(const char *msg) throw ())`
设置系统错误的callback。默认调用`perror()`并`abort()`。
设置在一个可重试系统调用错误（比如 `select`，`poll`，`epoll_wait` 失败）发生时调用的回调函数。消息是一个可打印的字符串，表示导致问题产生的系统调用或子系统。如果设置了这个回调，则 libev 将期待它补救这种状况，无论何时何地它返回。即 libev 通常将重试请求的操作，或者如果条件没有消失，执行 bad stuff（比如终止程序）。

例如：我们可以使用此函数将error信息输出到指定文件中，并终止程序
```
static void
fatal_error (const char *msg)
{
	FILE *fp = fopen("/tmp/error.msg", "w+");
	fprintf(fp, "%s\n", msg);
  	fclose(fp);
  	abort ();
}
 
. . .
ev_set_syserr_cb (fatal_error);
```

##### ev_feed_signal
`ev_feed_signal(int signum)`
这个函数可被用于 “模拟” 一个信号接收。在任何时候，任何上下文，包括信号处理器或随机线程，调用这个函数都是完全安全的。
它的主要用途是在你的进程中定制信号处理。比如，你可以默认在所有线程中阻塞信号（当创建任何 loops 时指定 `EVFLAG_NOSIGMASK`），然后在一个线程中，使用 `sigwait` 或其它的机制来等待信号，然后通过调用 `ev_feed_signal` 将它们“传送” 给 libev。

#### ev_loop
Event loop 用一个结构体`struct ev_loop *`描述。
Libev 支持两类 loop，一是 default loop，支持子进程事件（child process event）；而动态创建的 event loops 就不支持这个功能。

```
struct ev_loop *ev_default_loop (unsigned int flags);
struct ev_loop *ev_loop_new (unsigned int flags);
```
`ev_default_loop` 初始化 default loops。如果已经初始化了，那么直接返回并且忽略 flags。注意这个函数并不是线程安全的。只有这个 loop 可以处理`ev_child`事件。如果你不知道使用什么事件循环，则使用这个函数返回的那个（或通过 `EV_DEFAULT` 宏）。

`ev_loop_new`这个函数是线程安全的。一般而言，每个 thread 使用一个 loop。这将创建并初始化一个新的事件循环对象。如果循环无法初始化，则返回 fa

flags 参数可被用于指定特殊的行为或要使用的特定后端，且通常被指定为 0（或 `EVFLAG_AUTO`）。
flags支持如下值：
- `EVFLAG_AUTO` 
默认值，常用。如果你没有线索就是用它。

- `EVFLAG_NOENV`
指定 libev 不使用LIBEV_FLAGS环境变量。常用于调试和测试

- `EVFLAG_FORKCHECK`
除了在 fork 之后手动地调用 `ev_loop_fork`，你还可以通过启用这个标记让 libev 在每个迭代中检查 fork。
它通过在循环的每一次迭代中调用 `getpid()` 来工作，如果你执行大量的循环迭代但只做一点实际的工作，则这将可能会降低你的事件循环的速度，但它通常不明显（比如在 GNU/Linux 系统上，getpid 实际上是一个简单的 5-insn 序列而没有系统调用，因此非常块，但是在 GNU/Linux  还有 `pthread_atfork`，它可能会更快）。
当你使用这个标记时这个标记的巨大的好处是你可以忘记 `fork`（并忘记忘记告诉 libev 关于 fork，尽管你依然不得不忽略 SIGPIPE）。
这个标记不能被 LIBEV_FLAGS 环境变量的值覆盖或指定。

- `EVFLAG_NOINOTIFY`
在ev_stat监听中不使用 *inotify* API,这个标记被指定时，则 libev 将不会试图为它的 `ev_stat` 观察者使用 *inotify* API。
除了调试和测试之外，这个标志对于保全 inotify 文件描述符是非常有用的，否则使用 ev_stat 观察者的每个循环消耗一个 *inotify* 句柄。

- `EVFLAG_SIGNALFD`
在ev_signal监听中使用 *signalfd* API, 当设置这个标记时，则 libev 将试图为它的 `ev_signal` (和 `ev_child`) 观察者使用 *signalfd* API。这个 API 同步地传递信号，这使它更快且可能使它能够获得入队的信号数据。只要你在对处理信号不感兴趣的线程中正确地阻塞信号，它也可以简化多线程中的信号处理。

- `EVFLAG_NOSIGMASK`
使 libev 避免修改 signal mask。这样的话，你要使 signal 是非阻塞的。在未来的 libev 中，这个 mask 将会是默认值。当指定这个标记时，则 libev 将避免修改信号掩码。特别地，这意味着当你想接收信号时你不得不确保它们是未阻塞的。
当你想要执行你自己的信号处理，或想要仅在特定的线程中处理信号并想要避免 libev 不阻塞信号时，这个行为很有用。
在一个线程的程序中它也是 POSIX 要求的，由于 libev 调用 `sigprocmask`，其行为是未正式定义的。
这个标记的行为将在未来的 libev 版本中变为默认的行为。

- `EVBACKEND_SELECT`
值为 1，可移植的 `select` 通用后端。
这是不完全的标准 `select(2)` 后端。因为 libev 尝试滚动自己的 `fd_set` 而不限制 fds 的数量，但是如果失败，则期望在使用此后端时 fds 的数量相当低的限制。它不能太好地缩放（O(highest_fd)），但对于少量的（low-numbered :）fds 它通常是最快的后端。
为了从这个后端获得良好的性能你需要大量的并发（大多数文件描述符应该处于忙碌状态）。如果你在编写一个服务器，你应该在循环的
`accept()` 的一个迭代中接受尽可能多的连接。可以使用 `ev_set_io_collect_interval()` 来增加每个迭代中通知的数量。
这个后端把 `EV_READ` 映射到 `readfds` 集合，并把 `EV_WRITE` 映射到 `writefds` 集合（为了绕过 Microsoft Windows bugs，还可以在该平台上设置的 `exceptfds`）。

- `EVBACKEND_POLL`
值为 2，`poll` 后端，除了 windows 外的其它地方都可用。
这是标准的 `poll(2)` 后端。它比 `select` 更复杂，但对稀疏 fds 的处理更好，且对你可以使用的 fds 的个数没有人为限制（除了在非活跃 fds 比较多时，它将大大减慢）。参考上面的 `EVBACKEND_SELECT` 的条目，获得性能提示。
这个后端把 `EV_READ` 映射为 `POLLIN | POLLERR | POLLHUP`，把 `EV_WRITE` 映射为 `POLLOUT | POLLERR | POLLHUP`。

- `EVBACKEND_EPOLL`
- `EVBACKEND_KQUEUE`
- `EVBACKEND_DEVPOLL`
- `EVBACKEND_PORT`


ev_run、ev_break以及ev_loop_default都是event loop控制函数。
event loop定义为struct ev_loop。
有两种类型的event loop，分别是default类型和dynamically created类型，区别是前者支持子进程事件。
ev_default_loop和ev_loop_new函数分别用于创建default类型或者dynamically created类型的event loop。

event_run函数告诉系统应用程序开始对事件进行处理，有事件发生时就调用watcher callbacks。
除非调用了ev_break或者不再有active的watcher，否则会一直重复这个过程。


## example

## ev_io

```c
#include <stdio.h>
#include <ev.h>

ev_io stdin_watcher;
ev_io stdout_watcher;

static void
stdin_cb (EV_P_ ev_io *w, int revents)
{
	printf("w->fd = %d\n", w->fd);
	puts ("stdin ready");
	ev_io_stop (EV_A_ w); 
	ev_break (EV_A_ EVBREAK_ALL);
}

int
main (void)
{
    struct ev_loop *loop = EV_DEFAULT;
    ev_io_init (&stdin_watcher, stdin_cb, 0, EV_READ);
    ev_io_init (&stdout_watcher, stdout_cb, 1, EV_WRITE);
    ev_io_start (loop, &stdin_watcher);
    ev_io_start (loop, &stdout_watcher);
    ev_run (loop, 0);
    return 0;
}

```

## 参考
[libev](https://metacpan.org/pod/distribution/EV/libev/ev.pod#NAME)
[libev 介绍](https://www.jianshu.com/p/2c78f7ec7c7f)
[libev的使用——结合Socket编程](https://blog.csdn.net/cxy450019566/article/details/52606512)
[Libev 官方文档学习笔记 - 01：概述和 ev_loop](https://segmentfault.com/a/1190000006173864)
[Libev 官方文档学习笔记 - 02：watcher 基础](https://segmentfault.com/a/1190000006200077)
[Libev 官方文档学习笔记 - 03：常用 watcher 接口](https://segmentfault.com/a/1190000006679929)
[使用 libev 构建 TCP 响应服务器（echo server）的简单流程](https://segmentfault.com/a/1190000006691243)


libev source ev.3