## libevt简介
一个 C 编写的功能全面的高性能事件循环库。
libev是一个event loop：事件驱动的库。通过向libev注册感兴趣的events，比如socket可读事件，libev会对所注册的事件的源进行管理，并在事件发生时触发相应的回调函数。

通过event watcher来注册事件。libev通过分配和注册watcher对不同类型的事件进行监听。
不同事件类型的watcher又对应不同的数据类型，watcher的定义模式是struct ev_TYPE或者ev_TYPE，

Libev 支持文件描述符事件的 select，poll，Linux 特有的 epoll，BSD 特有的 kqueue 以及 Solaris 特有的事件端口机制 (ev_io)，Linux 的 inotify 接口 (ev_stat)，Linux eventfd/signalfd（用于更快更干净的线程间唤醒 (ev_async)/信号处理 (ev_signal)），相对定时器 (ev_timer)，定制重新调度逻辑的绝对定时器 (ev_periodic)，同步的信号 (ev_signal)，进程状态变化事件 (ev_child)，以及处理事件循环机制自身的事件观察者 (ev_idle，ev_embed，ev_prepare 和 ev_check 观察者) 和文件观察者 (ev_stat)，甚至是对 fork 事件的有限支持 (ev_fork)。


### watcher

- ev_io：支持 Linux 的select、poll、epoll；BSD 的kqueue；Solaris 的event port mechanisms
- ev_signal：支持各种信号处理、同步信号处理
- ev_timer：相对事件处理
- ev_periodic：排程时间表
- ev_child：进程状态变化事件
- ev_stat：监视文件状态
- ev_fork：有限的fork事件支持
- ev_idle：
- ev_embed：
- ev_prepare：
- ev_check：
- ev_async:
- ev_cleanup:
- ev_prepare and ev_check

ev_init对一个watcher的与具体类型无关的部分进行初始化。
ev_io_set对watcher的与io类型相关的部分进行初始化，如果是TYPE类型那么相应的函数就是ev_TYPE_set。
可以采用ev_TYPE_init函数来替代ev_init和ev_TYPE_set。ev_io_start激活相应的watcher，watcher只有被激活的时候才能接收事件。ev_io_stop停止已经激活的watcher。


### functions

#### glob functions
这些函数可以随时调用，甚至在以任何方式初始化库之前。

##### ev_time
`ev_tstamp ev_time ()`
返回以 libev 所使用的格式的当前时间。注意 ev_now 函数通常更快，且也常常返回你实际想知道的时间戳。ev_now_update 和 ev_now 的结合也很有意思。

##### ev_sleep
`void ev_sleep (ev_tstamp interval)`
休眠一段指定的时间。如果interval小于等于0，则立刻返回。最大支持一天，也就是86400秒


#### ev_loop

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
[Libev官网文档学习笔记01](https://segmentfault.com/a/1190000006173864)
 
[libev](https://metacpan.org/pod/distribution/EV/libev/ev.pod#NAME)

[libev 介绍](https://www.jianshu.com/p/2c78f7ec7c7f)

libev source ev.3