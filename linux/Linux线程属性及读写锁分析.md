<!--
author: Magelive
date: 2016-12-12
title: Linux线程分析
tags: Linux pthread
category: Linux
status: publish
summary:
-->

**目录**

- Linux线程属性(*pthread_attr_t*)分析
- Linux线程读写锁(*pthread_rwlock_t*)分析
- Linux线程私有数据(*pthread_key_t*)分析
- Linux线程屏障功能（*pthread_barrier_t*)分析
- Linux线程一次初始化(*pthread_once_t*)分析

## Linux线程属性(*pthread_attr_t*)分析
Posix线程中的线程属性*pthread_attr_t*主要包括scope属性、detach属性、堆栈地址、堆栈大小、优先级。在*pthread_create*中，把第二个参数设置为NULL的话，将采用默认的属性配置。

### pthread_attr_t的主要属性
#### *__detachstate*
表示新线程是否与进程中其他线程脱离同步。

如果设置为*PTHREAD_CREATE_DETACHED*则新线程不能用*pthread_join()*来同步，且在退出时自行释放所占用的资源。

缺省为*PTHREAD_CREATE_JOINABLE*状态。

这个属性也可以在线程创建并运行以后用*pthread_detach()*来设置，而一旦设置为*PTHREAD_CREATE_DETACH*状态（不论是创建时设置还是运行时设置）则不能再恢复到*PTHREAD_CREATE_JOINABLE*状态。

#### *__schedpolicy*
表示新线程的调度策略，主要包括*SCHED_OTHER*（正常、非实时）、*SCHED_RR*（实时、轮转法）和*SCHED_FIFO*（实时、先入先出）三种。

缺省为*SCHED_OTHER*，后两种调度策略仅对超级用户有效。

运行时可以用过*pthread_setschedparam()*来改变。

#### *__schedparam*
一个*struct sched_param*结构，目前仅有一个*sched_priority*整型变量表示线程的运行优先级。

这个参数仅当调度策略为实时（即*SCHED_RR*或*SCHED_FIFO*）时才有效，并可以在运行时通过*pthread_setschedparam()*函数来改变，缺省为0。

#### *__inheritsched*
有两种值可供选择：*PTHREAD_EXPLICIT_SCHED*和*PTHREAD_INHERIT_SCHED*，前者表示新线程使用显式指定调度策略和调度参数（即attr中的值），而后者表示继承调用者线程的值。

缺省为*PTHREAD_EXPLICIT_SCHED*。

#### *__scope*
表示线程间竞争CPU的范围，也就是说线程优先级的有效范围。

POSIX的标准中定义了两个值：*PTHREAD_SCOPE_SYSTEM*和*PTHREAD_SCOPE_PROCESS*，前者表示与系统中所有线程一起竞争CPU时间，后者表示仅与同进程中的线程竞争CPU。

目前LinuxThreads仅实现了*PTHREAD_SCOPE_SYSTEM*一值。

为了设置这些属性，POSIX定义了一系列属性设置函数，包括pthread_attr_init()、pthread_attr_destroy()和与各个属性相关的pthread_attr_getXXX/pthread_attr_setXXX函数。

### Linux线程属性函数
在设置线程属性 pthread_attr_t 之前，通常先调用pthread_attr_init来初始化，之后来调用相应的属性设置函数。
```c
int pthread_attr_init (pthread_attr_t* attr);
/*
功能： 对线程属性变量的初始化。
头文件： <pthread.h>
函数传入值： attr:线程属性。
函数返回值： 成功： 0 	失败： -1
*/

int pthread_attr_setscope (pthread_attr_t* attr, int scope);
/*
功能： 设置线程 __scope 属性。scope属性表示线程间竞争CPU的范围，也就是说线程优先级的有效范围。
	  POSIX的标准中定义了两个值：PTHREAD_SCOPE_SYSTEM和PTHREAD_SCOPE_PROCESS，
			前者表示与系统中所有线程一起竞争CPU时间，
			后者表示仅与同进程中的线程竞争CPU。
			默认为PTHREAD_SCOPE_PROCESS。
			目前LinuxThreads仅实现了PTHREAD_SCOPE_SYSTEM一值。
头文件： <pthread.h>
函数传入值：
	attr: 线程属性。
	scope: PTHREAD_SCOPE_SYSTEM，表示与系统中所有线程一起竞争CPU时间，
           PTHREAD_SCOPE_PROCESS，表示仅与同进程中的线程竞争CPU
函数返回值得：同init。
*/

int pthread_attr_setdetachstate (pthread_attr_t* attr, int detachstate);
/*
功能： 	设置线程detachstate属性。
			该表示新线程是否与进程中其他线程脱离同步，
			如果设置为PTHREAD_CREATE_DETACHED则新线程不能用pthread_join()来同步，
			且在退出时自行释放所占用的资源。
		缺省为PTHREAD_CREATE_JOINABLE状态。
		这个属性也可以在线程创建并运行以后用pthread_detach()来设置，
			而一旦设置为PTHREAD_CREATE_DETACH状态（不论是创建时设置还是运行时设置）
			则不能再恢复到PTHREAD_CREATE_JOINABLE状态。
头文件：      <phread.h>
函数传入值：
	attr:线程属性。
	detachstate:
		PTHREAD_CREATE_DETACHED，不能用pthread_join()来同步，
			且在退出时自行释放所占用的资源
		PTHREAD_CREATE_JOINABLE，能用pthread_join()来同步
函数返回值得：同init。
*/

int pthread_attr_setschedparam (pthread_attr_t* attr, 
								struct sched_param* param);
/*
功能： 设置线程schedparam属性，即调用的优先级。
头文件：     <pthread.h>
函数传入值：
	attr：线程属性。
    param：线程优先级。
		一个struct sched_param结构，目前仅有一个sched_priority整型变量
		表示线程的运行优先级。
		这个参数仅当调度策略为实时（即SCHED_RR或SCHED_FIFO）时才有效，
		并可以在运行时通过pthread_setschedparam()函数来改变，缺省为0
函数返回值：同init。
*/

 int pthread_attr_getschedparam (pthread_attr_t* attr,
								struct sched_param* param);
/*
功能： 得到线程优先级。
头文件： <pthread.h>
函数传入值：
	attr：线程属性；
	param：线程优先级；
函数返回值：同init。
*/
```
### pthread_attr_t使用范例
```c
void *thread_run1(void* arg)
{
	int i = 0;
	for （; i < 10; i++)
	{
		printf("this thread 1.\n");
		if (i == 5)
			pthread_exit(0);
	}
	return NULL;
}

void *thread_run2(void *arg)
{
	int i = 0;
	for (; i< 5; i++)
		printf("this thread 2.\n");
	return NULL;
}

int main(void)
{
	pthread_t t1, t2;
	pthread_attr_t attr;
	pthread_attr_init (&attr);   
	pthread_attr_setscope (&attr, PTHREAD_SCOPE_SYSTEM);   
	pthread_attr_setdetachstate (&attr, PTHREAD_CREATE_DETACHED);
	pthread_create(&t1, &attr, thread_run1, NULL);
	pthread_create(&t2, NULL, thread_run2, NULL);
	pthread_join(t2, NULL);
	return 0; 
}
```
从上面事例中，可以得到这么一个结果，就是线程一的线程函数一结束就自动释放资源，线程二就得等到pthread_join来释放系统资源。

## Linux线程读写锁(*pthread_rwlock_t*)分析
### 读写锁原因与解决的问题
读写锁是用来解决读者写者问题的，读操作可以共享，写操作是排他的，读可以有多个在读，写只有唯一个在写，同时写的时候不允许读。
### 读写锁同步方式
具有强读者同步和强写者同步两种形式:
强读者同步：当写者没有进行写操作，读者就可以访问；
强写者同步：当所有写者都写完之后，才能进行读操作，读者需要最新的信息，一些事实性较高的系统可能会用到该所，比如定票之类的。
### 读写锁操作
```
pthread_rwlock_t  rw_lock = PTHREAD_RWLOCK_INITIALIZER;
int pthread_rwlock_init(pthread_rwlock_t *rwlock ,pthread_rwattr_t *attr);
/*
函数功能：读写锁的初始化
头文件：<pthread.h>
函数参数：
	rwlock 是一个指向读写锁的指针
	attr 是一个读写锁属性对象的指针,其作用与传递缺省读写锁属性对象的地址相同
		如果将NULL 传递给它,则使用默认属性来初始化一个读写锁。
返回值：0，表示成功，非0为一错误码
*/

int pthread_rwlock_destroy(pthread_rwlock_t* rwlock);
/*
函数功能：读写锁的销毁
头文件：<pthread.h>
函数参数：rwlock 是一个指向读写锁的指针
返回值：0，表示成功，非0为一错误码
*/

int pthread_rwlock_rdlock(pthread_rwlock_t* rwlock);
/*
函数功能：阻塞式获取读写锁的读锁操作
		如果读写锁由一个写者持有，则读线程会阻塞直至写入者释放读写锁。
头文件：<pthread.h>
函数参数：rwlock 是一个指向读写锁的指针
返回值：0，表示成功，非0为一错误码
*/

int pthread_rwlock_tryrdlock(pthread_rwlock_t* rwlock);
/*
函数功能：非阻塞式获取读写锁的读锁操作
		如果读写锁由一个写者持有，则读线程会阻塞直至写入者释放读写锁。
头文件：<pthread.h>
函数参数：rwlock 是一个指向读写锁的指针
返回值：0，表示成功;非0为一错误码, 会返回ebusy而不会让线程等待
*/

int pthread_rwlock_wrlock(pthread_rwlock_t* rwlock);
/*
函数功能：阻塞式获取读写锁的写锁操作
	如果对应的读写锁被其它写者持有，或者读写锁被读者持有，该线程都会阻塞等待。
头文件：<pthread.h>
函数参数：rwlock 是一个指向读写锁的指针
返回值：0，表示成功，非0为一错误码
*/

int pthread_rwlock_trywrlock(pthread_rwlock_t* rwlock);
/*
函数功能：非阻塞式获取读写锁的写锁操作
	如果对应的读写锁被其它写者持有，或者读写锁被读者持有，该线程都会阻塞等待。
头文件：<pthread.h>
函数参数：rwlock 是一个指向读写锁的指针
返回值：0，表示成功，非0为一错误码
*/

int pthread_rwlock_unlock(pthread_rwlock_t* rwlock);
/*
函数功能：释放读写锁
头文件：<pthread.h>
函数参数：rwlock 是一个指向读写锁的指针
返回值：0，表示成功，非0为一错误码
*/
```

### 互斥锁与读写锁的区别：
当访问临界区资源时（访问的含义包括所有的操作：读和写），需要上互斥锁；
当对数据（互斥锁中的临界区资源）进行读取时，需要上读取锁，当对数据进行写入时，需要上写入锁。

### 读写锁的优点：
对于读数据比修改数据频繁的应用，用读写锁代替互斥锁可以提高效率。因为使用互斥锁时，即使是读出数据（相当于操作临界区资源）都要上互斥锁，而采用读写锁，则可以在任一时刻允许多个读出者存在，提高了更高的并发度，同时在某个写入者修改数据期间保护该数据，以免任何其它读出者或写入者的干扰。

### 读写锁描述：
获取一个读写锁用于读称为共享锁，获取一个读写锁用于写称为独占锁，因此这种对于某个给定资源的共享访问也称为共享-独占上锁。

## Linux线程私有（线程存储）数据(*pthread_key_t*)分析
大家都知道，在多线程程序中，所有线程共享程序中的变量。现在有一全局变量，所有线程都可以使用它，改变它的值。而如果每个线程希望能单独拥有它，那么就需要使用线程存储了。表面上看起来这是一个全局变量，所有线程都可以使用它，而它的值在每一个线程中又是单独存储的。这就是线程存储的意义。
### Linux线程私有数据pthread_key_t相关函数与结构
```
pthread_key_t key;
int pthread_key_create(pthread_key_t *key, void (*destructor)(void*));
/*
函数功能：创建一个类型为pthread_key_t 类型的变量
头文件：<pthread.h>
函数参数：
	key 为指向一个键值的指针
	destructor 如果这个参数不为空，
	那么当每个线程结束时，系统将调用这个函数来释放绑定在这个键上的内存块。
返回值：0，表示成功，非0为一错误码
*/

int pthread_key_delete(pthread_key_t key);
/*
函数功能：注销一个私有key,
	这个函数并不检查当前是否有线程正使用该key，
	也不会调用清理函数（destr_function）
	而只是将key释放以供下一次调用pthread_key_create()使用。
头文件：<pthread.h>
函数参数：key 为一个键值
返回值：0，表示成功，非0为一错误码
*/

int pthread_setspecific(pthread_key_t key, const void *pointer));
/*
函数功能：当线程中需要存储特殊值的时候，可以调用此函数
头文件：<pthread.h>
函数参数：key 为一个键值
		pointer 为 void* 变量，可以存储任何类型的值
返回值：0，表示成功，非0为一错误码
*/

void *pthread_getspecific(pthread_key_t key);
/*
函数功能：需要取出所存储的值
头文件：<pthread.h>
函数参数：key 为要取私有数据的键值
返回值：返回取得数据的地址
*/
```

### pthread_key_t范例
```c

#include <malloc.h>
#include <pthread.h>
#include <stdio.h>

/* The key used to associate a log file pointer with each thread. */
static pthread_key_t thread_log_key;


/* Write MESSAGE to the log file for the current thread. */
void write_to_thread_log (const char* message)
{
	FILE* thread_log = (FILE*) pthread_getspecific (thread_log_key);
	fprintf (thread_log, “%s\n”, message);
}

/* Close the log file pointer THREAD_LOG. */
void close_thread_log (void* thread_log)
{
	fclose ((FILE*) thread_log);	
}

void* thread_function (void* args)
{
	char thread_log_filename[20];
	FILE* thread_log;

	/* Generate the filename for this thread’s log file. */
	sprintf (thread_log_filename, “thread%d.log”, (int) pthread_self ());

	/* Open the log file. */
	thread_log = fopen (thread_log_filename, “w”);

	/* Store the file pointer in thread-specific data under thread_log_key. */
	pthread_setspecific (thread_log_key, thread_log);
	write_to_thread_log (“Thread starting.”);
	/* Do work here... */
	return NULL;
}

int main ()
{
	int i;
	pthread_t threads[5];

	/* Create a key to associate thread log file pointers in
	thread-specific data. Use close_thread_log to clean up the file
	pointers. */
	pthread_key_create (&thread_log_key, close_thread_log);

	/* Create threads to do the work. */
	for (i = 0; i < 5; ++i)
		pthread_create (&(threads[i]), NULL, thread_function, NULL);

	/* Wait for all threads to finish. */
	for (i = 0; i < 5; ++i)
		pthread_join (threads[i], NULL);

	return 0;
}  
```

## Linux线程屏障功能（*pthread_barrier_t*)分析
### pthread_barrier的相关结构：
```
pthread_barrier_t
```
### pthread_barrier的相关函数
#### 函数原型
pthread_barrier 系列函数在<pthread.h>中定义，用于多线程的同步，它包含三个函数：
```
#include <pthread.h>
int pthread_barrier_init(pthread_barrier_t *restrict barrier, 
						const pthread_barrierattr_t *restrict attr, 
						unsigned count);
int pthread_barrier_wait(pthread_barrier_t *barrier);
int pthread_barrier_destroy(pthread_barrier_t *barrier);
```
函数执行成功返回 0，执行失败则返回一个错误号，我们可以通过该错误号获取相关的错误信息。
#### 参数解释
pthread_barrier_t，是一个计数锁，对该锁的操作都包含在三个函数内部，我们不用关心也无法直接操作。只需要实例化一个对象丢给它就好。
pthread_barrierattr_t，锁的属性设置，设为NULL让函数使用默认属性即可。
count，你要指定的等待个数。
### pthread_barrier的相关使用场景及解决方案
#### 使用场景
这种“屏障”机制最大的特点就是最后一个执行wait的动作最为重要，就像赛跑时的起跑枪一样，它来之前所有人都必须等着。所以实际使用中，pthread_barrier_*常常用来让所有线程等待“起跑枪”响起后再一起行动。比如我们可以用pthread_create（） 生成100 个线程，每个子线程在被create出的瞬间就会自顾自的立刻进入回调函数运行。但我们可能不希望它们这样做，因为这时主进程还没准备好，和它们一起配合的其它线程还没准备好，我们希望它们在回调函数中申请完线程空间、初始化后停下来，一起等待主进程释放一个“开始”信号，然后所有线程再开始执行业务逻辑代码。
#### 解决方案
为了解决上述场景问题，我们可以在init时指定n+1个等待，其中n是线程数。而在每个线程执行函数的首部调用wait()。这样100个pthread_create()结束后所有线程都停下来等待最后一个wait()函数被调用。这个wait()由主进程在它觉得合适的时候调用就好。最后这个wait()就是鸣响的起跑枪。
### pthread_barrier使用范例
```
void* thread_run(void *arg)
{
	pthread_barrier_t *barrier = (thread_barrier_t *）arg;
	pthread_barrier_wait(barrier);
	//do somethings...
}

void init(int thread_num)
{
	pthread_barrier_t barrier;
	pthread_t* thread = (pthread_t *)malloc(sizeof(pthread_t)*thread_num);
	pthread_barrier_init(&barrier,NULL, thread_num + 1);
	int i;
	for (i = 0; i < thread_num; i ++
	{
		pthread_create(&thread[i], NULL, thread_run, &barrier); 
	}
	//do something...
	thread_barrier_wait(&barrier);
	//do something...
	return;
}
```

**在这里我们需要注意的是使用barrier这个屏障我们无法获取线程的结束状态，若想要获取相关线程结束状态我们仍然需要调用pthread_join函数。当然我们一般也不会把pthread_barrier_wait 放在某个线程结束时候，这显然是很无聊的，这个函数调用往往出现在线程之间的某个位置，接下来线程等待其它协同线程到达屏障后再处理一些其它事务。**

## Linux线程一次初始化(*pthread_once_t*)分析
在多线程环境中，有些事仅需要执行一次。通常当初始化应用程序时，可以比较容易地将其放在main函数中。但当你写一个库时，就不能在main里面初始化了，你可以用静态初始化，但使用一次初始化（pthread_once）会比较容易些。

在LinuxThreads中，实际"一次性函数"的执行状态有三种：NEVER（0）、IN_PROGRESS（1）、DONE （2），如果once初值设为1，则由于所有pthread_once()都必须等待其中一个激发"已执行一次"信号，因此所有pthread_once ()都会陷入永久的等待中；如果设为2，则表示该函数已执行过一次，从而所有pthread_once()都会立即返回0。

### once函数
```c
pthread_once_t once_control = PTHREAD_ONCE_INIT;
int pthread_once(pthread_once_t* once_control, void (*init_routine)(void));
/*
函数功能：pthread_once函数首先检查控制变量，判断是否已经完成初始化，如果完成就简单地返回；
		否则，pthread_once调用初始化函数，并且记录下初始化被完成。
		如果在一个线程初始时，另外的线程调用pthread_once，
		则调用线程等待，直到那个现成完成初始话返回。
头文件：<pthread.h>
函数参数：
	once_control 是一个控制变量。控制变量必须使用PTHREAD_ONCE_INIT宏静态地初始化。
	init_routine 函数仅执行一次，究竟在哪个线程中执行是不定的，是由内核调度来决定。
返回值：若成功返回0，若失败返回错误编号。
*/
```

### once范例
```
int i = 0;
pthread_once_t once_control = PTHREAD_ONCE_INIT;
void _once_run()
{
	i++;
}
void* thread_run(void *)
{
	pthread_once(&once_control, _once_run);
	printf(" i = %d\n", i);
}

int main()
{
	……
	pthread_create(&thread_id, NULL, thread_run, NULL);
}

```

## Linux线程取消点
### 线程取消的定义
一般情况下，线程在其主体函数退出的时候会自动终止，但同时也可以因为接收到另一个线程发来的终止（取消）请求而强制终止。

### 线程取消的语义
线程取消的方法是向目标线程发Cancel信号，但如何处理Cancel信号则由目标线程自己决定，或者忽略、或者立即终止、或者继续运行至Cancelation-point（取消点），由不同的Cancelation状态决定。
线程接收到CANCEL信号的缺省处理（即pthread_create()创建线程的缺省状态）是继续运行至取消点，也就是说设置一个CANCELED状态，线程继续运行，只有运行至Cancelation-point的时候才会退出。

### 取消点
根据POSIX标准，pthread_join()、pthread_testcancel()、pthread_cond_wait()、pthread_cond_timedwait()、sem_wait()、sigwait()等函数以及read()、write()等会引起阻塞的系统调用都是Cancelation-point，而其他pthread函数都不会引起Cancelation动作。但是pthread_cancel的手册页声称，由于LinuxThread库与C库结合得不好，因而目前C库函数都不是Cancelation-point；但CANCEL信号会使线程从阻塞的系统调用中退出，并置EINTR错误码，因此可以在需要作为Cancelation-point的系统调用前后调用pthread_testcancel()，从而达到POSIX标准所要求的目标，即如下代码段：
```
pthread_testcancel();
    retcode = read(fd, buffer, length);
    pthread_testcancel();
```

### 程序设计方面的考虑
如果线程处于无限循环中，且循环体内没有执行至取消点的必然路径，则线程无法由外部其他线程的取消请求而终止。因此在这样的循环体的必经路径上应该加入pthread_testcancel()调用。

### 与线程取消相关的pthread函数
```
int pthread_cancel(pthread_t thread) 
```
发送终止信号给thread线程，如果成功则返回0，否则为非0值。发送成功并不意味着thread会终止。
```
int pthread_setcancelstate(int state, int *oldstate) 
```
设置本线程对Cancel信号的反应，state有两种值：PTHREAD_CANCEL_ENABLE（缺省）和PTHREAD_CANCEL_DISABLE，分别表示收到信号后设为CANCLED状态和忽略CANCEL信号继续运行；old_state如果不为NULL则存入原来的Cancel状态以便恢复。
```
int pthread_setcanceltype(int type, int *oldtype) 
```
设置本线程取消动作的执行时机，type由两种取值：PTHREAD_CANCEL_DEFFERED和PTHREAD_CANCEL_ASYCHRONOUS，仅当Cancel状态为Enable时有效，分别表示收到信号后继续运行至下一个取消点再退出和立即执行取消动作（退出）；oldtype如果不为NULL则存入运来的取消动作类型值。
```
void pthread_testcancel(void) 
```
检查本线程是否处于Canceld状态，如果是，则进行取消动作，否则直接返回。
