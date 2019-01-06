## libev简介
一个 C 编写的功能全面的高性能事件循环库。
libev是一个event loop：事件驱动的库。通过向libev注册感兴趣的events，比如socket可读事件，libev会对所注册的事件的源进行管理，并在事件发生时触发相应的回调函数。

通过event watcher来注册事件。libev通过分配和注册watcher对不同类型的事件进行监听。
不同事件类型的watcher又对应不同的数据类型，watcher的定义模式是`struct ev_TYPE`或者`ev_TYPE`，

Libev 支持文件描述符事件的 select，poll，Linux 特有的 epoll，BSD 特有的 kqueue 以及 Solaris 特有的事件端口机制 (`ev_io`)，Linux 的 inotify 接口 (`ev_stat`)，Linux eventfd/signalfd（用于更快更干净的线程间唤醒 (`ev_async`)/信号处理 (`ev_signal`)），相对定时器 (`ev_timer`)，定制重新调度逻辑的绝对定时器 (`ev_periodic`)，同步的信号 (`ev_signal`)，进程状态变化事件 (`ev_child`)，以及处理事件循环机制自身的事件watcher (`ev_idle`，`ev_embed`，`ev_prepare` 和 `ev_check` watcher) 和文件watcher (`ev_stat`)，甚至是对 fork 事件的有限支持 (`ev_fork`)。

## glob functions
这些函数可以随时调用，甚至在以任何方式初始化库之前。

### ev_time
`ev_tstamp ev_time ()`
返回以 libev 所使用的格式的当前时间。注意 `ev_now`函数通常更快，且也常常返回你实际想知道的时间戳。
`ev_now_update` 和 `ev_now` 的结合也很有意思。

Libev 使用一个`ev_tstamp`数据类型来表示1970年以来的秒数，实际类型是 C 里面的double类型。

### ev_sleep
`void ev_sleep (ev_tstamp interval)`
休眠给定的时间：当前线程将阻塞，直到它被中断或经过了给定的时间间隔（大约 - 即使不中断，它可能也会早返回一点）。
如果 interval <= 0 就立即返回。基本上这是一个粒度比秒更高的 sleep()。
interval 的范围是有限的 - libev 只保证最长一天 (interval <= 86400) 的休眠时间是可以工作的。

### ev_version_major和ev_version_minor
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

### supported backends
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

`ev_embeddable_backends`返回其它事件循环中可嵌入的后端的集合。这个值是平台特有的，但可以包含当前系统不可用的后端。为了找出当前系统可能支持的可嵌入后端，你需要查看 `ev_embeddable_backends () & ev_supported_backends ()`，同样的建议采用的那些。参考 `ev_embed` watcher的描述来获得更多信息。

### ev_set_allocator
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

### ev_set_syserr_cb
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

### ev_feed_signal
`ev_feed_signal(int signum)`
这个函数可被用于 “模拟” 一个信号接收。在任何时候，任何上下文，包括信号处理器或随机线程，调用这个函数都是完全安全的。
它的主要用途是在你的进程中定制信号处理。比如，你可以默认在所有线程中阻塞信号（当创建任何 loops 时指定 `EVFLAG_NOSIGMASK`），然后在一个线程中，使用 `sigwait` 或其它的机制来等待信号，然后通过调用 `ev_feed_signal` 将它们“传送” 给 libev。

## ev_loop
Event loop 用一个结构体`struct ev_loop *`描述。
Libev 支持两类 loop，一是 default loop，支持子进程事件（child process event）；而动态创建的 event loops 就不支持这个功能。

### ev_default_loop和ev_loop_new
函数原型：
```
struct ev_loop *ev_default_loop (unsigned int flags);
struct ev_loop *ev_loop_new (unsigned int flags);
```
`ev_default_loop` 初始化 default loops。如果已经初始化了，那么直接返回并且忽略 `flags`。注意这个函数并不是线程安全的。只有这个 loop 可以处理`ev_child`事件。如果你不知道使用什么事件循环，则使用这个函数返回的loop或使用 `EV_DEFAULT` 宏。

`ev_loop_new`这个函数是线程安全的。一般而言，每个 thread 使用一个 loop。这将创建并初始化一个新的事件循环对象。如果循环无法初始化，则返回 `false`。

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
在ev_stat监听中不使用 *inotify* API,这个标记被指定时，则 libev 将不会试图为它的 `ev_stat` watcher使用 *inotify* API。
除了调试和测试之外，这个标志对于保全 inotify 文件描述符是非常有用的，否则使用 ev_stat watcher的每个循环消耗一个 *inotify* 句柄。

- `EVFLAG_SIGNALFD`
在ev_signal监听中使用 *signalfd* API, 当设置这个标记时，则 libev 将试图为它的 `ev_signal` (和 `ev_child`) watcher使用 *signalfd* API。这个 API 同步地传递信号，这使它更快且可能使它能够获得入队的信号数据。只要你在对处理信号不感兴趣的线程中正确地阻塞信号，它也可以简化多线程中的信号处理。

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
值为 4，`epoll` 后端，linux特有和。
这是标准的`epoll(7)` 接口后端，这个后端可能比 `poll` 和 `select` 慢一点，但它的表现更好。尽管 `poll` 和 `select` 通常的表现大概为 O(total_fds)，其中 total_fds 是 fds 的总个数（或最高的 fd）， `epoll` 的表现为 O(1) 或 O(active_fds)。
这个后端映射 `EV_READ` 与 `EV_WRITE` 的方式与 `EVBACKEND_POLL` 的相同。

- `EVBACKEND_KQUEUE`
值为8，BSD系统特有。
这个后端把 `EV_READ` 映射成 `EVFILT_READ` 事件和 `NOTE_EOF`, 把 `EV_WRITE` 映射成 `EVFILT_WRITE` 事件和 `NOTE_EOF`.

- `EVBACKEND_DEVPOLL`
值为 16，Solaris 8特有。
根据报告，/dev/poll 只支持 sockets，且不是可嵌入的，这将大大限制这个后端的有用性。

- `EVBACKEND_PORT`
值为 32，Solaris 10特有。
这个后端映射 `EV_READ` 与 `EV_WRITE` 的方式与 `EVBACKEND_POLL` 的相同。

- `EVBACKEND_ALL`
尝试所有的后端。由于它是一个掩码，你可以做一些特殊的事情，比如 `EVBACKEND_ALL & ~EVBACKEND_KQUEUE`,但是不太推荐使用些设置，建议使用`ev_recommended_backends()`来获取和设置后端，或者更简单的干脆不指定后端，由ev库自己来选择使用。

- `EVBACKEND_MASK`
不是一个后端，一个用来从 `flags` 值中选择所有的后端位的掩码，在你想从标志值屏蔽任何后端的情况下（比如当修改 `LIBEV_FLAGS` 环境变量时）。
如果标记值中有一个或多个后端标记，则只会尝试这些后端（以这里列出的相反的顺序）。
如果没有指定，则会尝试 `ev_recommended_backends()` 中的所有后端。

### `ev_loop_destory`
函数原型：`void ev_loop_destroy (loop);`
销毁一个`ev_loop`事件循环对象（释放所有的内存和内核状态等等）。注意这里要将所有的 IO 清除光之后再调用，因为这个函数并不中止所有活跃（active）的 IO。部分 IO 不会被清除，比如 signal。这些需要手动清除。注意某些全局状态，比如信号状态（及安装的信号处理程序），将不会被这个函数释放，及相关的watcher（比如信号和 child watcher）将需要手动地停止。
这个函数一般和`ev_loop_new`一同使用。当然它也可以用于`ev_default_loop`返回的默认的 loop，只是在这种情况下不是线程安全的。
注意不建议对默认的 loop 调用这个函数，除了在极少的你真的需要释放它的资源的情况下。

### `ev_loop_fork`
函数原型：`void ev_loop_fork (struct ev_loop *loop);`
这个函数导致`ev_run`的子过程重设已有的 backend 的 kernel state。
重用父进程创建的 loop。可以和`pthread_atfork()`配合使用。
需要在每一个需要在 `fork` 之后重用的 loop 中调用这个函数。
必须在恢复之前或者调用`ev_run()`之前调用。如果是在fork之后创建的 loop，不需要调用。
此外，如果需要重用这个loop,必须要忽略信息`SIGPIPG`。
和`pthread_atfork()`代码结合如下：
```
post_fork_child (void)
{
  ev_loop_fork (EV_DEFAULT);
}
 
...
pthread_atfork (0, 0, post_fork_child);
```

### `ev_is_default_loop`
函数原型：`int ev_is_default_loop (struct ev_loop *loop);`
判断当前 loop 是不是 default loop。

### `ev_iteration`
函数原型：`unsigned int ev_iteration (struct ev_loop * loop);`
返回当前的 loop 的迭代数。等于 libev pool 新事件的数量（?）。这个值对应ev_prepare和ev_check调用，并在 prepare 和 check 之间增一。

### `ev_depth`
函数原型：`unsigned int ev_depth (struct ev_loop *loop);`
返回ev_run()进入减去退出次数的差值。
注意，导致ev_run异常退出的调用（setjmp / longjmp, pthread_cancel, 抛出异常等）均不会导致该值减一。

### `ev_backend`
函数原型：`unsigned int ev_backend (struct ev_loop *loop);`
返回 `EVBACKEND_*` 标记中的一个，以指明使用的事件后端。

### `ev_now`和`ev_noew_updata`
函数原型：
```
ev_tstamp ev_now (struct ev_loop *loop);
void ev_now_update (struct ev_loop *loop);
```
`ev_now`得到当前的 “事件循环时间(event loop time)”，它是事件循环接收事件并开始处理它们的时间。在 callback 调用期间，这个值是不变的。回调一被处理，这个时间戳就不会变了，它还是用于相对定时器的基时间。你可以把它当作事件发生（或更正确地说，libev 发生它）的时间。
`ev_now_updata`更新从`ev_now()`中返回的时间。不必要的话，不要使用，因为这个函数的开销相对是比较大的。它通常在`ev_run`中会自动完成更新。

### `ev_suspend`和`ev_resume`
函数原型：
```
void ev_suspend (struct ev_loop *loop);
void ev_resume (struct ev_loop *loop);
```
这两个函数挂起并恢复一个事件循环，当 loop 有一段事件不用，且超时不应该被处理时使用。同时其 timeout 也会暂停。如果恢复后，timer 会从上一次暂停状态继续及时——这一点对于实现一些要连同时间也一起冻结的功能时，非常有用。

注意已经 resume 的loop不能再 resume，反之已经 suspend 的 loop 不能再 suspend。

典型的使用场景是交互式的程序，比如游戏：当用户按下 `^Z` 挂起游戏，并在一小时后恢复，对于超时最好的处理是在程序挂起期间就像时间没有流逝一样。这可以通过在你的 `SIGTSTP` 处理程序中调用 `ev_suspend`，给你自己发送一个 `SIGSTOP` 并在之后直接调用 `ev_resume` 恢复定时器处理来实现。

### `ev_run`
函数原型：`bool ev_run (struct ev_loop *loop, int flags);`
初始化 loop 结束后，调用这个函数开始 loop。如果 flags == 0，直至 loop 没有活跃的时间或者是调用了 `ev_break` 之后停止。
Loop 可以是异常使能的，你可以在 callback 中调用`longjmp`来终端回调并且跳出 `ev_run`，或者通过抛出 C++ 异常。这些不会导致 `ev_depth` 值减少。
`event_run`函数告诉系统应用程序开始对事件进行处理，有事件发生时就调用watcher callbacks。
除非调用了`ev_break`或者不再有active的watcher，否则会一直重复这个过程。

`flags`标志值为`EVRUN_NOWAIT`时，`ev_run`会检查并且执行所有未解决的 events，但如果没有就绪的时间，ev_run 会立刻返回。当其值为`EVRUN_ONCE`时会检查所有的 events，在至少每一个 event 都执行了一次事件迭代之后才返回。但有时候，使用`ev_prepare/ev_check`会更好。

`ev_run`的大致工作流程：

- loop depth ++
- 重设ev_break状态
- 在首次迭代之前，调用所有 pending watchers

LOOP：
- 如果置了`EVFLAG_FORKCHECK`，则检查 fork，如果检测到 `fork`，则排队并调用所有的 fork watchers
- 排队并且调用所有 ready 的watchers
- 如果`ev_break`被调用了，则直接跳转至 FINISH
- 如果检测到了 fork，则分离并且重建 kernel state
- 使用所有未解决的变化更新 kernel state
- 更新`ev_now`的值
- 计算要 sleep 或 block 多久
- 如果指定了的话，sleep
- loop iteration ++
- 阻塞以等待事件
- 排队所有未处理的I/O事件
- 更新`ev_now`的值，执行 time jump 调整
- 排队所有超时事件
- 排队所有定期事件
- 排队所有优先级高于 pending 事件的 idle watchers
- 排队所有 check watchers
- 按照上述顺序的逆序，调用 watchers (check watchers -> idle watchers -> 定期事件 -> 计时器超时事件 -> fd事件)。信号和 child watchers 视为 fd watchers。
- 如果`ev_break`被调用了，或者使用了`EVRUN_ONCE`或者`EVRUN_NOWAIT`，则如果没有活跃的 watchers，则 FINISH，否则 continue

FINISH：
- 如果是`EVBREAK_ONE`，则重设 `ev_break` 状态
- loop depth --
- return

### `ev_break`
函数原型：`void ev_break (struct ev_loop *loop, int how);`
中断 loop。参数可以是 `EVBREAK_ONE`（执行完一个内部调用后返回）或`EVBREAK_ALL`（执行完所有）。
可被用于执行一个调用使`ev_run` 提前返回（但是只有在其处理完了所有outstanding 事件之后）。其中的 `how` 参数必须是 `EVBREAK_ONE`，它使最内层的 `ev_run` 返回，或者是 `EVBREAK_ALL`，它使所有嵌套的 `ev_run` 返回。
这个 "break 状态" 将在下次调用 `ev_run` 时被清除。
在任何 `ev_run` 调用之外调用 `ev_break` 也是安全的，只是在那种情况下不起作用。

### `ev_ref`和`ev_unref`
函数原型：
```
void ev_ref (struct ev_loop *loop);
void ev_unref (struct ev_loop *loop);
```
Ref/unref 可以被用于添加或移除一个事件循环的引用计数。
每个watcher持有一个引用计数，只要引用计数不为零，ev_run 就不会返回。
在做 start 之后要 unref；stop 之前要 ref。

### `ev_set_io_collect_interval`和`ev_set_timeout_collect_interval`
函数原型：
```
void ev_set_io_collect_interval (struct ev_loop *loop, ev_tstamp interval);
void ev_set_timeout_collect_interval (struct ev_loop *loop, ev_tstamp interval);
```
两个值均默认为0，表示尽量以最小的延迟调用io和定时器callback。
但这是理想的情况，实际上，比如 select 这样低效的系统调用，由于可以一次性读取很多，所以可以适当地进行延时。
通过使用比较高的延迟，但是增加每次处理的数据量，以提高 CPU 效率。

### `ev_pending_count`、`ev_invoke_pending`和`ev_set_invoke_pending_cb`
函数原型：
```
int ev_pending_count (struct ev_loop *loop);
void ev_invoke_pending (struct ev_loop *loop);
void ev_set_invoke_pending_cb (struct ev_loop *loop, void (*invoke_pending_cb(EV_P)));
````
`ev_pending_count` 返回当前有多少个 pending 的 watchers。
`ev_invoke_pending` 调用所有的 pending 的 watchers。这个除了可以在 callback 中调用（少见）之外，更多的是在重载的函数中使用。通常，`ev_run`会在需要时自动执行此操作，但当覆盖invoke回调时，此调用非常方便。可以从watcher中调用此函数，例如，当您想进行一些长时间的计算并想将进一步的事件处理传递给另一个线程时（必须确保在`ev_invoke_pending`或`ev_run`中只执行一个线程）。
`ev_set_invoke_pending_cb`将覆盖loop中的调用挂起功能：在调用所有pending watchers时，`ev_run`将改用此回调函数来回调处理。例如，当您想在另一个上下文（另一个线程等）中调用实际的watchers时，这是很有用的。
如果要重置回调，使用`ev_invoke_pending`作为新callback。

### `ev_set_loop_release_cb`
函数原型：
```
void ev_set_loop_release_cb (struct ev_loop *loop,
                             void (*release)(EV_P)throw(),
                             void (*acquire)(EV_P)throw());
```

这是一个 lock 操作，你可以自定义 lock。其中 release 是 unlock，acquire 是 lock。release 是在 loop 挂起以等待events 之前调用，并且在开始回调之前调用 acquire。

### `ev_set_userdata`和`ev_userdata`
函数原型：
```
void ev_set_userdata (struct ev_loop *loop, void *data);
void *ev_userdata (struct ev_loop *loop);
```
设置/读取 loop 中的用户 data。

### `ev_verify`
函数原型：`void ev_verify (struct ev_loop *loop);`
验证当前 loop 的设置。如果发现问题，则打印 error msg 并 abort()。

## watcher

### watcher状态

#### initialised
在watcher被register到loop之前，它处于 initialised 状态。
可以通过调用 `ev_TYPE_init` 或 `ev_init` 和watcher特有的 `ev_TYPE_set` 函数来完成。
处于这种状态下，它仅仅是一些适合事件循环使用的内存块。它可以根据需要被移动、释放、复用等，只要你保持内存内容不变，或者再次调用 `ev_TYPE_init`。

#### started/running/active
调用`ev_TYPE_start`之后的状态，并且开始等待事件。在这个状态下，除了特别提及的少数情况之外，它不能存取、移动、释放，只能维持着对它的指针。
一旦watcher已经通过调用 `ev_TYPE_start` 启动了，则它就变成了loop的属性，并活跃地等待事件。
在这种状态下它不能被访问、移动、释放或其它操作，仅有的合法的事情是持有一个指向它的指针，并调用一些允许在活跃的watcher上调用的 libev 函数。

#### pending
当 watcher 是 active 并且一个让 watcher 感兴趣的事件到来，那么 watcher 进入 pending。
这个状态的 watcher 可以 access，但不能存取、移动、释放。

#### stopped
watcher可被 libev 隐式地停止（在这种情况中它可能依然处于挂起状态），或通过调用它的 `ev_TYPE_stop` 函数显式地停止。无论它是否处于活跃状态, `ev_TYPE_stop`将清除watcher中任何可能处于的挂起状态。因此在释放一个watcher时，常常需要显式地停止它。
停止的（不是挂起）watcher本质上是处于初始化状态的，可以以任何式来复用、移动和修改（当free了内存块时，需要重新 `ev_TYPE_init`）。
调用`ev_TYPE_stop`后的状态，此时状态与 initialized 相同。

### watcher 优先级模型

许多event loop都支持watcher的优先级，这些优先级通常以很小整数来表示，以整数的规律来影响watcher之间事件回调调用的顺序。

event loop中处理优先级的方式有两种：
- lock-out model
在此模型中，优先级较高watcher的“lock out”调用优先级较低的watcher，这意味着只要优先级较高的watche接收到事件，就不会调用优先级较低的watcher。
- only-for-ordering model
仅使用优先级在单个event loop中对回调调用进行排序：优先级较高的watcher在优先级较低的watcher之前被调用，但在对新事件进行轮询之前都被调用。

libev中除了idle watcher使用的是lock-out模式下，其它的所有watcher使用的都是only-for-ordering模式。
这是因为大多数内核接口不支持为watche实现lock-out model，并且大多数事件库只要它们的回调没有被执行就会一次又一次地对相同的事件进行轮询，在高优先级watcher锁定大量低优先级watcher的常见情况下，这是非常低效的。

在libev中，可以使用`ev_set_priority`设置watcher优先级。

静态（排序）优先级在两个或多个watcher处理同一资源时最有用：一个典型的使用场景是让`ev_io` watcher来接收数据，以及另一个`ev_timer` watcher来处理超时。在加载状态下，当程序处理其他作业时，也可以接收数据，但是在检查数据前，由于timer watcher会首先调用，因此会优先调用超时处理程序，导致数据无法处理。在这种情况下，给timer watcher一个低的优先级，给io watcher一个高的优先级，那么就可以确保优先处理IO watcher中的数据。

由于idle watcher使用的是lock-out model，这意味着idle watcher将仅在没有相同或更高优先级的watcher收到事件时执行。
例如，要模拟有多少其他事件库处理优先级，可以将`ev_idle` watcher与其它watcher关联，在正常watcher的callback中只需启动idle watcher。真正的处理是在idle watcher中的callback中完成。这会导致libev不断地轮询和处理kernel中关于watcher的事件和数据，在lock-out case很少的情况下，这样是可行的。但是，一般情况下，以这种方式来实现的lock-out model在其设计处理的负载类型下会表现得很糟糕。在这种情况下，在启动idle watcher之前最好停止实际的wathcer，这样内核就不必处理watcher事件，以防实际处理被延迟相当长的时间。

如下例子：一个I/O watcher stdin的例子，它以默认优先级更低的级别来运行，只在没有其他事件时处理数据：
```
ev_idle idle; // actual processing watcher
ev_io io;     // actual event watcher
 
static void
io_cb (EV_P_ ev_io *w, int revents)
{
  // stop the I/O watcher, we received the event, but
  // are not yet ready to handle it.
  ev_io_stop (EV_A_ w);
 
  // start the idle watcher to handle the actual event.
  // it will not be executed as long as other watchers
  // with the default priority are receiving events.
  ev_idle_start (EV_A_ &idle);
}
 
static void
idle_cb (EV_P_ ev_idle *w, int revents)
{
  // actual processing
  read (STDIN_FILENO, ...);
 
  // have to start the I/O watcher again, as
  // we have handled the event
  ev_io_start (EV_P_ &io);
}
 
// initialisation
ev_idle_init (&idle, idle_cb);
ev_io_init (&io, io_cb, STDIN_FILENO, EV_READ);
ev_io_start (EV_DEFAULT_ &io);
```


### watcher 通用函数

#### callback
`void (*)(struct ev_loop *loop, ev_TYPE *watcher, int revents);`

#### ev_init
`void ev_init (ev_TYPE *watcher, callback);`
初始化watcher的通用部分。watcher对象的内容可以是任意的，只有watcher的通用部分被初始化，在之后需要调用类型特有的 `ev_TYPE_set` 来初始化类型特有的部分。对于每一个类型，还有一个 `ev_TYPE_init` 可以把这两个调用合为一个。
你可以在任何时间重新初始化一个watcher，只要它已经停止（或从未启动），且没有挂起事件。

#### ev_TYPE_set
`void ev_TYPE_set (ev_TYPE *watcher, [args]);`
设置指定类型的 wetaher。init 函数必须在此之前被调用一次，此后可以设置任意次的 set 函数。不能对一个 active 的 watcher 调用此函数。

#### ev_TYPE_init
`void ev_TYPE_init(ev_TYPE *watch, callback, [args]);`
这个宏将 init 和 set 糅合在一起使用，相当于`ev_init`和`ev_TYPE_set`两条指令。

#### ev_TYPE_start
`void ev_TYPE_start (loop, ev_TYPE *watcher);`
启动（激活）给定的watcher。只有活跃的watcher可以接收事件。如果 watcher 已经是 active，则调用无效。。

#### ev_TYPE_stop
`void ev_TYPE_stop (loop, ev_TYPE *watcher);`
停止 watcher，并清空 pending 状态。如果要释放一个 Watcher，最好都显式地调用 stop。

#### ev_is_active 
`bool ev_is_active (ev_TYPE *watcher);`
判断watcher是不是active状态，如果 watcher 被执行了一次 start，并且未被 stop，则返回 true。

#### ev_is_pending
`bool ev_is_pending (ev_TYPE *watcher);`
当且仅当 watcher pending 时返回 true。（如：有未决的事件，但是 callback 未被调用）

####  ev_cb和ev_set_cb
```
callback ev_cb (ev_TYPE *watcher);
void ev_set_cb (ev_TYPE *watcher, callback);
```
返回或设置当前watcher callback。

#### ev_priority和ev_set_priority
```
int ev_priority (ev_TYPE *watcher);
void ev_set_priority (ev_TYPE *watcher, int priority);
```
Priority 是一个介于`EV_MAXPRI`（默认2）和`EV_MIN_PRI`（默认-2）之间的值。数值越高越优先被调用。但除了 `ev_idle`，每一个 watcher 都会被调用。当 watcher 是 active 或 pending 时并不能修改。实际上 priority 大于-2到2的范围也是没问题的。

#### ev_invoke
`void ev_invoke (struct ev_loop *loop, ev_TYPE *watcher, int revents);`
在指定的loop中以指定的revents参数唤醒wather来运行其设定的callback。不管loop和watcher是否是valid状态。

#### ev_clear_pending
`int ev_clear_pending (struct ev_loop *loop, ev_TYPE *watcher);`
清除watcher的pending状态，并返回revents状态。如果watcher不是pending状态，则返回0。

#### ev_feed_event
`void ev_feed_event (struct ev_loop *loop, ev_TYPE *watcher, int revents)`
给指定的loop中的watcher设置revents事件，相当于模拟发生特定的revents事件。

### watcher 类型

每一个watcher类型有一个附属的watcher结构体（一般是`struct ev_TYPE`或`ev_TYPE`）。

每一个watcher必须通过调用`ev_init (watcher *, callback)`来初始化，这个调用需要传入一个回调。每次在事件发生时，这个回调会被调到（或者在 I/O watcher的情况中，每次事件循环探测到给定的文件描述符可读和/或可写的时候）。

每一个watcher都有对应的`ev_TYPE_set`函数、`ev_TYPE_start`函数、`ev_TYPE_stop`函数。在`ev_run` 之前进行各个 watcher 的 `ev_start`。
可以采用`ev_TYPE_init`函数来替代`ev_init`和`ev_TYPE_set`。`ev_TYPE_start`激活相应的watcher，watcher只有被激活的时候才能接收事件。`ev_TYPE_stop`停止已经激活的watcher。

每一个watcher都还有它自己的 `ev_TYPE_set (watcher *, ...)`函数 ，参数列表依赖于watcher类型。还有一个调用中结合了初始化和设置：`ev_TYPE_init (watcher *, callback, ...)`。

只要 watcher 是 active，就不能再调用 init。
每个 callback 都有三个参数：loop, watcher, 事件的掩码值。可能的掩码值有：
- `EV_READ` 
`ev_io` watcher中的文件描述符已经变得可读。

- `EV_WRITE` 
`ev_io` watcher中的文件描述符已经变得可写。

- `EV_TIMER`
`ev_timer` watcher已经超时。

- `EV_PERIODIC`
`ev_periodic` watcher已经超时。

- `EV_SIGNAL`
`ev_signal` watcher中指定的信号已经由一个线程接收到。

- `EV_CHILD`
`ev_child` watcher中指定的 pid 已经接收到一个状态改变。

- `EV_STAT`
`ev_stat` watcher中指定的路径以某种方式改变了其属性。

- `EV_IDLE`
`ev_idle` watcher没有其它更好的事情要做。

- `EV_PREPARE/EV_CHECK`
所有的 `ev_prepare` watcher仅在 `ev_run` 开始收集新事件 *之前* 调用，而所有的 `ev_check` watcher仅在 `ev_run` 已经收集到了它们之后，但在任何接收到的事件的回调入队之前，被加入队列（而不是调用）。
这意味着 `ev_prepare` watcher是在事件循环休眠或为新事件而 poll 之前最后被调用的watcher，而 `ev_check` watcher将在一个事件循环迭代内任何其它相同或更低优先级的watcher之前被调用。
这两种watcher类型的回调可以启动或停止任何数量它们想要的watcher，所有这些都将被考虑在内（比如，`ev_prepare` watcher可能启动一个 idle watcher来保持`ev_run`不被阻塞）。

- `EV_EMBED`：
ev_embed watcher 中指定的嵌入式事件循环需要注意。

- `EV_FORK`
子线程中 fork 之后事件循环已经恢复（参考 ev_fork）。

- `EV_CLEANUP`：
事件循环将被销毁（参考 ev_cleanup）。
- 
- `EV_ASYNC`：
给定的 async watcher已经被异步地通知了（参考 ev_async）。

- `EV_CUSTOM`：
不是 libev 自身发送（或另外使用）的事件，但可以被 libev 的用户自由地用来通知watcher（比如，通过 ev_feed_event）。

- `EV_ERROR`：
发生未指定的错误，watcher已被停止。这可能发生在由于 libev 内存不足而watcher无法正常启动，发现一个文件描述符已经关闭，或其它问题。Libev 认为这些是应用程序的错误。在 libev 内存不够用时可能产生；fd 被外部关闭时也可能产生。


#### ev_io
ev_io用来监听io事件，当有标准输入或输出时，则会触发事件，执行回调函数。
支持 Linux 的select、poll、epoll；BSD 的kqueue；Solaris 的event port mechanisms
这个 watcher 负责检测文件描述符是否可写入数据或者是读出数据。fd最好设置为non-block。
注意有时候在调用read时是没有数据的（返回0），此时一个一个非阻塞的read会得到EAGAIN错误。

##### ev_io functions
```
void ev_io_init (ev_io *io, callback, int fd, int events);
void ev_io_set (ev_io *io, int fd, int events);
void ev_io_start(struct ev_loop *loop, ev_io *);
void ev_io_stop(ev_io *);
```
设置和启动`ev_io` watcher。
其中 events 可以是`EV_WRITE`和`EV_READ`的组合。

##### example
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

##### ev_io special problem
###### 文件描述符消失的特殊问题
部分后端(eg.kqueue, epoll)需要显示的调用close来关闭fd，原因在于当你在外部调用close来关闭fd时, 当fd小于0时，系统中相关的后端会悄悄的处理掉，而不会有相应的notify,如果此时有新的fd的值刚好跟之前watcher中的相同时，libev无法区别其是一个新的watcher还是之前的watcher,这样会导致回调的callback异常。
因此为了避免此种情况，libev在一般情况下都不会变动文件描述符，只会每次在调用`ev_io_set`时才更新其值。同样，在需要更新文件描述符时，必须调用`ev_io_set`或是`ev_io_init`来更新它，否则仅仅只是更改ev_io中的值是无效的。
这种情况一般出现在没有调用`ev_io_stop`就把fd关闭导致的。

###### 使用dup操作fd的特殊问题
部分后端（eg.epoll)无法为文件描述符注册事件,只能注册为基础的文件描述符（underlying file descriptions），这意味着使用`dup()`或其他奇怪操作的fd，只能由其中一个被接收到。

###### 关于文件的特殊问题
`ev_io`对于文件来说并没有什么用，只要文件存在，就可立即访问，并不需要事件来通知和等待。对于stdin和stdout，请谨慎使用，确保这两者没有被重定向至文件。

###### 关于 fork 的特殊问题
部分后端（eg.epoll,kqueue)根据不支持`fork()`,libev中使用`ev_loop_fork()`来完全支持`fork()`,并且使用EVFLAG_FORKCHECK。不过对于epoll和kqueue之外的无需担心。。

###### SIGPIPE的特殊问题
只是提醒一下：记得处理SIGPIPE事件。

###### 关于accept一个无法接受的连接
大多数 POSIX accpet 实现中在删除因为错误而导致的连接时（如 fd 到达上限）都回产生一个错误的操作，比如使 accept 失败但不拒绝连接，只产生ENFILE错误。但这个会导致 libev 还是将其标记为 ready 状态。
推荐方法是列出所有的错误并记录下来，或者是暂时关闭 watchers。

#### ev_timer
一个相对超时机制的定时器。所谓的“相对”，就是说这个定时器的参数是：指定以当前时间为基准，延迟多久出发事件。
这个定时器与基于万年历的日期/时间是无关的，只基于系统单调时间。

##### ev_timer functions
```
ev_timer_init (struct ev_timer *, callback, ev_tstamp at, ev_tstamp repeat);
ev_timer_set(struct ev_timer *, ev_tstamp at, ev_tstamp repeat);
ev_timer_start (struct ev_loop *, ev_timer *);
ev_timer_stop(struct ev_timer *);
void ev_timer_again(struct ev_loop *loop, ev_timer *w);
ev_tstamp ev_timer_remaining (loop, ev_timer *);
```
如果repeat为正，这个timer会重复触发，否则只触发一次。
`ev_timer_again`表示重新启动定时器，相当于调用`ev_timer_stop`同时并更新repeat，并重新使用`ev_timer_start`来启动 timer watcher。
`ev_timer_remaining` 获取定时器剩下的时间。

##### ev_timer 使用策略

- 使用标准的初始化和停止 API 来重设
```
//ev_init(timer, callback);
//ev_timer_set (timer, 60.0, 0.0);
ev_timer_init (timer, callback, 60.0, 6.0);
ev_timer_start (loop, timer)
```
- 使用ev_timer_again重设
使用ev_timer_again，可以忽略ev_timer_start
```
ev_init (timer, callback);
timer->repeat = 60.0;
ev_timer_again (loop, start);
```
初始化完全后，可在callback中改变 timeout 值，不管 timer 是否 active:
```
timer->repeat = 60.0;
ev_timer_again (loop, timer);
```

- 让 timer 超时，但视情况重新配置
这个方式的基本思路是因为许多 timeout 时间都比 interval 大很多，此时要记住上一次活跃的时间，然后再 callback 中检查真正的 timeout

```
ev_tstamp g_timeout = 60.0;
ev_tstamp g_last_activity;
ev_timer  g_timer;

static void callback (EV_P_ev_timer *w, int revents)
{
    ev_tstamp after = g_last_activity - ev_now(EV_A) + g_timeout;
    
    // 如果小于零，表示时间已经发生了，已超时
    if (after < 0.0) {
        ......    // 执行 timeout 操作
    }
    else {
        // callback 被调用了，但是却有一些最近的活跃操作，说明未超时
        // 此时就按照需要设置的新超时事件来处理
        ev_timer_set (w, after, 0.0);
        ev_timer_start (loop, g_timer);
    }
}

```
启用这种模式，记得初始化时将`g_last_activity`设置为`ev_now`，并且调用一次`callback (loop, &g_timer, 0)`；当活跃时间到来时，只需修改全局的 timeout 变量即可，然后再调用一次 callback
```
g_timeout = new_value
ev_timer_stop (loop, &timer)
callback (loop, &g_timer, 0)
```
- 为 timer 使用双向链表
使用场景：有成千上万个请求，并且都需要 timeout。当 timeout 开始前，计算 timeout 的值，并且将 timeout 放在链表末尾。然后当链表前面的项需要触发时。使用`ev_timer`来将其触发掉。当有 activity 时，重算timeout,并将 timer 从 list中移至list 末尾，确保如果ev_timer已经被重新更新。
通过这种方式，可以在O（1）时间内管理无限数量的timer的启动、停止和更新，代价是出现严重的复杂情况，并且必须使用持续的超时来确保列表保持排序状态。

##### ev_timer special problem 

###### timeout太早的问题
假设在50.9秒的时候请求延时1秒，那么当51秒到来时，可能导致 timeout，这就是“太早”问题。Libev的策略是对于这种情况，在52秒时才执行 timeout。但是这又有“太晚”的问题，请程序员注意.

###### time更新的问题
libev只在`ev_run`收集新事件之前和之后更新其当前的时间，这导致在一次迭代中处理大量事件时，`ev_now`和`ev_time()`之间的差异不断增大。
如果怀疑事件处理被延迟，并且需要根据当前时间确定超时时间，请使用如下方法进行调整：
`ev_timer_set (&timer, after + (ev_time () - ev_now ()), 0.);`

###### 非同步时钟的特殊问题
Libev使用的时一个内部的单调时钟(Wall clock or monotonic clock)而不是系统时钟，而`ev_timer()`则是基于系统时钟的，所以在做比较的时候两者不同步。

###### 假死(suspended animation)问题
Suspenged animation，也称为休眠，指的是将机子置于休眠状态。
注意不同的机子不同的系统这个行为可能不一样。其中有一种休眠后会使得所有程序感觉只是经过了很小的一段时间一般（时间跳跃）。推荐在SIGTSTP处理中调用ev_suspend和ev_resume，但不能对SIGSTOP做任何事情。

##### example
- 创建一个60s后启动的timer

```
static void
one_minute_cb (struct ev_loop *loop, ev_timer *w, int revents)
{
  .. one minute over, w is actually stopped right here
}
 
ev_timer mytimer;
ev_timer_init (&mytimer, one_minute_cb, 60., 0.);
ev_timer_start (loop, &mytimer);
```
- 创建一个timer，在10秒不活动后超时。

```
static void
timeout_cb (struct ev_loop *loop, ev_timer *w, int revents)
{
  .. ten seconds without any activity
}
 
ev_timer mytimer;
ev_timer_init (&mytimer, timeout_cb, 0., 10.); /* note, only repeat used */
ev_timer_again (&mytimer); /* start timer */
ev_run (loop, 0);
 
// and in some piece of code that gets executed on any "activity":
// reset the timeout to start ticking again at 10 seconds
ev_timer_again (&mytimer);
```



#### ev_signal 
`ev_signal`：支持各种信号处理、同步信号处理
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
- `ev_prepare` 
- `ev_check`



## example

## ev_io



```

## 参考
[libev](https://metacpan.org/pod/distribution/EV/libev/ev.pod#NAME)
[libev 介绍](https://www.jianshu.com/p/2c78f7ec7c7f)
[libev的使用——结合Socket编程](https://blog.csdn.net/cxy450019566/article/details/52606512)
[Libev 官方文档学习笔记 - 01：概述和 ev_loop](https://segmentfault.com/a/1190000006173864)
[Libev 官方文档学习笔记 - 02：watcher 基础](https://segmentfault.com/a/1190000006200077)
[Libev 官方文档学习笔记 - 03：常用 watcher 接口](https://segmentfault.com/a/1190000006679929)
[使用 libev 构建 TCP 响应服务器（echo server）的简单流程](https://segmentfault.com/a/1190000006691243)
[Libev库学习（详细）](https://blog.csdn.net/guankeliang/article/details/82911856)
[c10K problem](http://www.kegel.com/c10k.html)
[libev 事件库](https://jin-yang.github.io/post/linux-libev.html)
libev source ev.3
