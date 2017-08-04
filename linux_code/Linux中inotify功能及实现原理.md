inotify 它是一个内核用于通知用户空间程序文件系统变化的机制。
##简介
Inotify 是一个Linux内核特性，它监控文件系统，并且及时向专门的应用程序发出相关的事件警告，比如删除、读、写和卸载操作等。

还可以跟踪活动的源头和目标等细节。在实际项目中，如果项目带有配置文件，那么怎么让配置文件的改变和项目程序同步而不需要重启程序呢？一个明显的应用是：在一个程序中，使用Inotify监视它的配置文件，如果该配置文件发生了更改（更新，修改）时，Inotify会产生修改的事件给程序，应用程序就可以实现重新加载配置文件，检测哪些参数发生了变化，并在应用程序内存的一些变量做相应的修改。当然另一种方法可以是通过cgi注册命令，并通过命令更新内存数据及更新配置文件

##inotify主要功能
它是一个内核用于通知用户空间程序文件系统变化的机制。

众所周知，Linux 桌面系统与 MAC 或 Windows 相比有许多不如人意的地方，为了改善这种状况，开源社区提出用户态需要内核提供一些机制，以便用户态能够及时地得知内核或底层硬件设备发生了什么，从而能够更好地管理设备，给用户提供更好的服务，如 hotplug、udev 和 inotify 就是这种需求催生的。Hotplug 是一种内核向用户态应用通报关于热插拔设备一些事件发生的机制，桌面系统能够利用它对设备进行有效的管理，udev 动态地维护 /dev 下的设备文件，inotify 是一种文件系统的变化通知机制，如文件增加、删除等事件可以立刻让用户态得知，该机制是著名的桌面搜索引擎项目 beagle 引入的，并在 Gamin 等项目中被应用。

##用户态接口
在用户态，inotify 通过三个系统调用和在返回的文件描述符上的文件 I/O 操作来使用，使用 inotify 的第一步是创建 inotify 实例：
```
int fd = inotify_init ();
```
每一个 inotify 实例对应一个独立的排序的队列。

文件系统的变化事件被称做 watches 的一个对象管理，每一个 watch 是一个二元组（目标，事件掩码），目标可以是文件或目录，事件掩码表示应用希望关注的 inotify 事件，每一个位对应一个 inotify 事件。Watch 对象通过 watch描述符引用，watches 通过文件或目录的路径名来添加。目录 watches 将返回在该目录下的所有文件上面发生的事件。
```
int wd = inotify_add_watch (fd, path, mask);
```
fd 是 inotify_init() 返回的文件描述符，path 是被监视的目标的路径名（即文件名或目录名），mask 是事件掩码, 在头文件 linux/inotify.h 中定义了每一位代表的事件。可以使用同样的方式来修改事件掩码，即改变希望被通知的inotify 事件。Wd 是 watch 描述符。

下面的函数用于删除一个 watch：
```
int ret = inotify_rm_watch (fd, wd);
```
fd 是 inotify_init() 返回的文件描述符，wd 是 inotify_add_watch() 返回的 watch 描述符。Ret 是函数的返回值。

文件事件用一个 inotify_event 结构表示，它通过由 inotify_init() 返回的文件描述符使用通常文件读取函数 read 来获得:
```
struct inotify_event {
        __s32           wd;             /* watch descriptor */
        __u32           mask;           /* watch mask */
        __u32           cookie;         /* cookie to synchronize two events */
        __u32           len;            /* length (including nulls) of name */
        char            name[0];        /* stub for possible name */
};
```
结构中的 wd 为被监视目标的 watch 描述符，mask 为事件掩码，len 为 name字符串的长度，name 为被监视目标的路径名，该结构的 name 字段为一个桩，它只是为了用户方面引用文件名，文件名是变长的，它实际紧跟在该结构的后面，文件名将被 0 填充以使下一个事件结构能够 4 字节对齐。注意，len 也把填充字节数统计在内。

通过 read 调用可以一次获得多个事件，只要提供的 buf 足够大。
```
size_t len = read (fd, buf, BUF_LEN);
```
buf 是一个 inotify_event 结构的数组指针，BUF_LEN 指定要读取的总长度，buf 大小至少要不小于 BUF_LEN，该调用返回的事件数取决于 BUF_LEN 以及事件中文件名的长度。Len 为实际读去的字节数，即获得的事件的总长度。

可以在函数 inotify_init() 返回的文件描述符 fd 上使用 select() 或poll(), 也可以在 fd 上使用 ioctl 命令 FIONREAD 来得到当前队列的长度。close(fd)将删除所有添加到 fd 中的 watch 并做必要的清理。
```
int inotify_init (void);
int inotify_add_watch (int fd, const char *path, __u32 mask);
int inotify_rm_watch (int fd, __u32 mask);
```
###inotify 可以监视的文件系统事件包括：
1. IN_ACCESS，即文件被访问
2. IN_MODIFY，文件被 write
3. IN_ATTRIB，文件属性被修改，如 chmod、chown、touch 等
4. IN_CLOSE_WRITE，可写文件被 close
5. IN_CLOSE_NOWRITE，不可写文件被 close
6. IN_OPEN，文件被 open
7. IN_MOVED_FROM，文件被移走,如 mv
8. IN_MOVED_TO，文件被移来，如 mv、cp
9. IN_CREATE，创建新文件
10. IN_DELETE，文件被删除，如 rm
11. IN_DELETE_SELF，自删除，即一个可执行文件在执行时删除自己
12. IN_MOVE_SELF，自移动，即一个可执行文件在执行时移动自己
13. IN_UNMOUNT，宿主文件系统被 umount
14. IN_CLOSE，文件被关闭，等同于(IN_CLOSE_WRITE | IN_CLOSE_NOWRITE)
15. IN_MOVE，文件被移动，等同于(IN_MOVED_FROM | IN_MOVED_TO)
**注：上面所说的文件也包括目录。**

##内核实现原理
在内核中，每一个 inotify 实例对应一个 inotify_device 结构：
```
struct inotify_device {
		wait_queue_head_t       wq;             /* wait queue for i/o */
		struct idr              idr;            /* idr mapping wd -> watch */
		struct semaphore        sem;            /* protects this bad boy */
        struct list_head        events;         /* list of queued events */
        struct list_head        watches;        /* list of watches */
        atomic_t                count;          /* reference count */
        struct user_struct      *user;          /* user who opened this dev */
        unsigned int            queue_size;     /* size of the queue (bytes) */
        unsigned int            event_count;    /* number of pending events */
        unsigned int            max_events;     /* maximum number of events */
        u32                     last_wd;        /* the last wd allocated */
};
```
d_list 指向所有 inotify_device 组成的列表的，i_list 指向所有被监视 inode 组成的列表，count 是引用计数，dev 指向该 watch 所在的 inotify 实例对应的 inotify_device 结构，inode 指向该 watch 要监视的 inode，wd 是分配给该 watch 的描述符，mask 是该 watch 的事件掩码，表示它对哪些文件系统事件感兴趣。

结构 inotify_device 在用户态调用 inotify_init（） 时创建，当关闭 inotify_init（）返回的文件描述符时将被释放。结构 inotify_watch 在用户态调用 inotify_add_watch（）时创建，在用户态调用 inotify_rm_watch（） 或 close（fd） 时被释放。

无论是目录还是文件，在内核中都对应一个 inode 结构，inotify 系统在 inode 结构中增加了两个字段：
```
struct inotify_watch {
        struct list_head        d_list; /* entry in inotify_device's list */
        struct list_head        i_list; /* entry in inode's list */
        atomic_t                count;  /* reference count */
        struct inotify_device   *dev;   /* associated device */
        struct inode            *inode; /* associated inode */
        s32                     wd;     /* watch descriptor */
        u32                     mask;   /* event mask for this watch */
};
```

d_list 指向所有 inotify_device 组成的列表的，i_list 指向所有被监视 inode 组成的列表，count 是引用计数，dev 指向该 watch 所在的 inotify 实例对应的 inotify_device 结构，inode 指向该 watch 要监视的 inode，wd 是分配给该 watch 的描述符，mask 是该 watch 的事件掩码，表示它对哪些文件系统事件感兴趣。

结构 inotify_device 在用户态调用 inotify_init（） 时创建，当关闭 inotify_init（）返回的文件描述符时将被释放。结构 inotify_watch 在用户态调用 inotify_add_watch（）时创建，在用户态调用 inotify_rm_watch（） 或 close（fd） 时被释放。

无论是目录还是文件，在内核中都对应一个 inode 结构，inotify 系统在 inode 结构中增加了两个字段：

```
#ifdef CONFIG_INOTIFY
	struct list_head	inotify_watches; /* watches on this inode */
	struct semaphore	inotify_sem;	/* protects the watches list */
#endif
```

inotify_watches 是在被监视目标上的 watch 列表，每当用户调用 inotify_add_watch（）时，内核就为添加的 watch 创建一个 inotify_watch 结构，并把它插入到被监视目标对应的 inode 的 inotify_watches 列表。inotify_sem 用于同步对 inotify_watches 列表的访问。当文件系统发生第一部分提到的事件之一时，相应的文件系统代码将显示调用fsnotify_* 来把相应的事件报告给 inotify 系统，其中*号就是相应的事件名，目前实现包括：

fsnotify_move，文件从一个目录移动到另一个目录fsnotify_nameremove，文件从目录中删除fsnotify_inoderemove，自删除fsnotify_create，创建新文件fsnotify_mkdir，创建新目录fsnotify_access，文件被读fsnotify_modify，文件被写fsnotify_open，文件被打开fsnotify_close，文件被关闭fsnotify_xattr，文件的扩展属性被修改fsnotify_change，文件被修改或原数据被修改有一个例外情况，就是 inotify_unmount_inodes，它会在文件系统被 umount 时调用来通知 umount 事件给 inotify 系统。

以上提到的通知函数最后都调用 inotify_inode_queue_event（inotify_unmount_inodes直接调用 inotify_dev_queue_event ），该函数首先判断对应的inode是否被监视，这通过查看 inotify_watches 列表是否为空来实现，如果发现 inode 没有被监视，什么也不做，立刻返回，反之，遍历 inotify_watches 列表，看是否当前的文件操作事件被某个 watch 监视，如果是，调用 inotify_dev_queue_event，否则，返回。函数inotify_dev_queue_event 首先判断该事件是否是上一个事件的重复，如果是就丢弃该事件并返回，否则，它判断是否 inotify 实例即 inotify_device 的事件队列是否溢出，如果溢出，产生一个溢出事件，否则产生一个当前的文件操作事件，这些事件通过kernel_event 构建，kernel_event 将创建一个 inotify_kernel_event 结构，然后把该结构插入到对应的 inotify_device 的 events 事件列表，然后唤醒等待在inotify_device 结构中的 wq 指向的等待队列。想监视文件系统事件的用户态进程在inotify 实例（即 inotify_init（） 返回的文件描述符）上调用 read 时但没有事件时就挂在等待队列 wq 上。

##范例
```

#include <stdio.h>
#include <string.h>
#include <sys/inotify.h>
#include <sys/select.h>
#include <unistd.h>

#define EVENT_SIZE  ( sizeof (struct inotify_event) )
#define BUF_LEN     ( 1024 * ( EVENT_SIZE + 16 ) )
#define ERR_EXIT(msg,flag)	{perror(msg);goto flag;}

int main( int argc, char **argv ) 
{
	int length, i = 0;
	int fd;
	int wd;
	char buffer[BUF_LEN];

	if((fd = inotify_init()) < 0)
		ERR_EXIT("inotify_init",inotify_init_err);

	if( (wd = inotify_add_watch( fd, "/tmp",	IN_MODIFY | IN_CREATE | IN_DELETE ) ) < 0)
		ERR_EXIT("inofity_add_watch", inotify_add_watch_err);
	
	fd_set rfd;
	struct timeval tv;
	tv.tv_sec = 0;
	tv.tv_usec = 10000;//10millsecond
	while(true)
	{
		int retval;
		FD_ZERO(&rfd);
		FD_SET(fd, &rfd);
		retval = select(fd + 1, &rfd, NULL, NULL, &tv);
		if(retval == 0) 
			continue;
		else if(retval == -1)
			ERR_EXIT("select",select_err);

		// retval > 0
		length = read( fd, buffer, BUF_LEN );  
		if(length < 0)
			ERR_EXIT("read",read_err);

		//length >= 0
		int i = 0;
		while ( i < length ) 
		{
			struct inotify_event *event = ( struct inotify_event * ) &buffer[ i ];
			if ( event->len ) 
			{
				if ( event->mask & IN_CREATE ) 
				{
					if ( event->mask & IN_ISDIR ) 
						printf( "The directory %s was created.\n", event->name );       
					else
						printf( "The file %s was created.\n", event->name );
					if(strcmp(event->name,"kill") == 0)
						ERR_EXIT("success exit",success_exit);

				}
				else if ( event->mask & IN_DELETE ) 
				{
					if ( event->mask & IN_ISDIR ) 
						printf( "The directory %s was deleted.\n", event->name );       
					else
						printf( "The file %s was deleted.\n", event->name );
				}
				else if ( event->mask & IN_MODIFY ) 
				{
					if ( event->mask & IN_ISDIR )
						printf( "The directory %s was modified.\n", event->name );
					else
						printf( "The file %s was modified.\n", event->name );
				}
			}else
			{
				//TODO
				//when only a file(not directory) is specified by add watch function, 
				//event->len's value may be zero, we can handle it here
			}
			i += EVENT_SIZE + event->len;
		}
	}
success_exit:
	( void ) inotify_rm_watch( fd, wd );
	( void ) close( fd );
	return 0;

read_err:
select_err:
inotify_add_watch_err:
	( void ) inotify_rm_watch( fd, wd );
inotify_init_err:
	( void ) close( fd );

	return -1;
}
```
以上代码需要注意的地方：
1. 如果在/tmp目录下touch kill文件，程序则会退出
2. 如果只有一个add watch 一个file,那么这个file的更改产生的event事件中event->len是为0，需要额外的处理，此代码省略了具体的处理过程，以注释代替
3. 如果监测的是文件或目录的更改，使用 echo "xxx" >> file，会产生一个event事件，而使用echo "xxx" > file 会产生两个event事件，查了相关的资料，可能是因为后者需要先清空file文件内容，造成第一次event事件，再将xxx写入file保存，造成了第二次的event事件。

##参考
[Linux inotify功能及实现原理](http://blog.csdn.net/myarrow/article/details/7096460)
[ linux 监视文件系统inotify 测试](http://blog.csdn.net/hepeng597/article/details/7792565)