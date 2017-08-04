对于应用程序自行处理的信号来说，信号的生命周期要经过信号的安装登记、信号集操作、信号的发送和信号的处理四个阶段。
1. 信号的安装登记指的是在应用程序中，安装对此信号的处理方法。
2. 信号集操作的作用是用于对指定的一个或多个信号进行信号屏蔽，此阶段对有些应用程序来说并不需要。
3. 信号的发送指的是发送信号，可以通过硬件（如在终端上按下Ctrl-C）发送的信号和软件（如通过kill函数）发送的信号。
4. 信号的处理指的是操作系统对接收信号进程的处理，处理方法是先检查信号集操作函数是否对此信号进行屏蔽，如果没有屏蔽，操作系统将按信号安装函数中登记注册的处理函数完成对此进程的处理。


##简介
信号是与一定的进程相联系的。也就是说，一个进程可以决定在进程中对哪些信号进行什 么样的处理。
例如，一个进程可以忽略某些信号而只处理其他一些信号；另外，一个进程还可以选择如何处理信号。
总之，这些总与特定的进程相联系的。因此，首先要建立其信号和进程的对应关系，这就是信号的安装登记。

Linux 主要有两个函数实现信号的安装登记：signal和sigaction。
其中signal在系统调用的基础上实现，是库函数。它只有两个参数，不支持信号传递信息，主要是用于前32个非实时信号的安装。
而sigaction是较新的函数（由两个系统调用实现：sys_signal以及 sys_rt_sigaction），有三个参数，支持信号传递信息，主要用来与sigqueue系统调用配合使用。当然，sigaction同样支持非实时信号的安装，sigaction优于signal主要体现在支持信号带有参数。

##signal()函数
在signal函数中，有两个形参，分别代表需要处理的信号编号值和处理信号函数的指针。
它主要是用于前32种非实时信号的处理，不支持信号的传递信息。但是由于使用简单，易于理解，因此在许多场合被程序员使用。

**所需头文件:**
``` 
#include <signal.h>
```
**函数原型:**
```
typedef void (*sighandler_t)(int);
sighandler_t signal(int signum, sighandler_t handler);

//展开后如下
void (*signal(int signum,void(* handler)(int)))(int);
```
###函数说明:
**功能说明：**
设置信号处理方式。signal()会依参数signum指定的信号编号来设置该信号的处理函数。当指定的信号到达时就会跳转到参数handler指定的函数执行。

**参数说明：**
signum 指定信号编号,	handle 指定信号处理函数指针。

**返回值：**
成功返回先前的信号处理函数指针，失败返回SIG_ERR（-1)。

需要注意的是：
对于Unix系统来说，使用signal函数时，自定义处理信号函数执行一次后失效，对该信号的处理回到默认处理方式。
下面以一个例子进行说明，例如一程序 中使用signal(SIGQUIT, my_func)函数调用，其中my_func是自定义函数。应用进程收到SIGQUIT信号时，会跳转到自定义处理信号函数my_func处执行，执行后信号注册函数my_func失效，对SIGQUIT信号的处理回到操作系统的默认处理方式，当应用进程再次收到SIGQUIT信号时，会按操作系统默认的处理方式进行处理（即不再执行my_func处理函数）。而在Linux系统中，signal函数已被改写，由sigaction函数封装实现，则不存在上述问题。

###signal()范例
```
//signal.c
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>

void my_func(int sign_no)
{
    if(sign_no==SIGINT)
        printf("I have get SIGINT\n");
    else if(sign_no==SIGQUIT)
        printf("I have get SIGQUIT\n");
}
int main()
{
    printf("Waiting for signal SIGINT or SIGQUIT \n ");
    signal(SIGINT, my_func);
    signal(SIGQUIT, my_func);
    pause();
	pause();
    exit(0);
}
```
编译、运行与结果
```
gcc signal.c –o signal
./signal
Waiting for signal SIGINT or SIGQUIT
I have get SIGINT	//按下CTRL+C
I have get SIGQUIT	//按下CTRL+\
```

##sigaction()函数
sigaction函数用来查询和设置信号处理方式，它是用来替换早期的signal函数。



**所需头文件：**
```
#include <signal.h>
```

**函数原型：**
```
int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact);
```

###函数说明:
**功能说明：**
sigaction()会依参数signum指定的信号编号来设置该信号的处理函数。

**参数说明：**
1. signum：需要处理的信号编号，可以指定SIGKILL和SIGSTOP以外的所有信号。
2. act：信号处理结构，其详解如下：
结构如下：
```
	struct sigaction
{
	void (*sa_handler) (int);
	void  (*sa_sigaction)(int, siginfo_t *, void *);
	sigset_t sa_mask;
	int sa_flags;
	void (*sa_restorer) (void);
}
```
sa_handler：此参数和signal()的参数handler相同，此参数主要用来对信号旧的安装函数signal()处理形式的支持。
a_sigaction：新的信号安装机制，处理函数被调用的时候，不但可以得到信号编号，而且可以获悉被调用的原因以及产生问题的上下文的相关信息。
sa_mask：用来设置在处理该信号时暂时将sa_mask指定的信号搁置。
sa_flags：用来设置信号处理的其他相关操作，下列的数值可用。可用OR 运算（|）组合使用。
```
SA_NOCLDSTOP: 如果参数signum为SIGCHLD, 那么就不再接收子进程结束的消息，即当子进程结束时并不会通知给父进程。
SA_NOCLDWAIT：如果参数signum为SIGCHLD， 如果信号是SIGCHLD，当终止时，不要将孩子变成僵尸。
SA_NODEFER
SA_ONSTACK
SA_RESETHAND
SA_RESTART
SA_RESTORER
SA_SIGINFO
```
sa_restorer： 此参数没有使用。

3. oldact：如果参数oldact不是NULL指针，则原来的信号处理方式会由此结构sigaction返回
