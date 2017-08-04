
##\_\_attribute\_\_((cleanup(...)))
用于修饰一个变量，在它的作用域结束时可以自动执行一个指定的方法。
###说明
1. 所谓作用域结束，包括大括号结束、return、goto、break、exception等各种情况。

2. 当然，可以修饰的变量不止基本类型，自定义结构（类）都是可以的。

3. 假如一个作用域内有若干个cleanup的变量，他们的调用顺序是先入后出的栈式顺序；而且，cleanup是先于这个对象的dealloc调用的。

###范例

``` 
#define local_type  __attribute__ ((cleanup(my_free)))
static void my_free(void* pmem)
{
	free(*pmem);
}


int foo(void)
{
	local_type int* p = (int*) malloc(sizeof(int));
	//
	// use *p
	// when return, the memory block pointed by p is freed automatically
	return 0;
}
```

##\_\_attribute\_\_((aligned(...)))
告诉编译器一个结构体或者类或者联合或者一个类型的变量(对象)分配地址空间时的地址对齐方式。

###与pragma pack(...)区别
1. pragma packet(n): 告诉编译器结构体或类内部的成员变量相对于第一个变量的地址的偏移量的对齐方式，缺省情况下，编译器按照自然边界对齐，当变量所需的自然对齐边界比n大 时，按照n对齐，否则按照自然边界对齐

2. \_\_attribute\_\_((aligned(n))): 告诉编译器一个结构体、类、联合或变量(对象)分配地址空间时的地址对齐方式。也就是说，如果将\_\_attribute\_\_((aligned(n)))作用于一个类型，那么该类型的变量在分配地址空间时，其存放的地址一定按照n字节对齐(n必须是2的幂次方)。并且其占用的空间，即大小，也是n的整数倍，以保证在申请连续存储空间的时候，每一个元素的地址也是按照m字节对齐。\_\_attribute\_\_((aligned(n)))也可以作用于一个单独的变量。

3. aligned 属性使被设置的对象占用更多的空间，相反的，使用packed 可以减小对象占用的空间。

###范例
```
#pragma pack(2)
struct test {
	char a;		//1 byte
	int c;		//4 byte
	short b;	//2 byte
}

sizeof(struct test) = 8
```

```
struct test {
	char a;		//1 byte
	int c;		//3 byte
	short b;	//2 byte
}__attribute__(aligned(32));
sizeof(sizeof test) = 32
```

##\_\_attribute__((section(...)))
其作用是将作用的函数或数据放入指定名为"section_name"输入段。
###说明
这里还要注意一下两个概念：输入段和输出段。

输入段和输出段是相对于要生成最终的elf或binary时的Link过程说的，Link过程的输入大都是由源代码编绎生成的目标文件.o，那么这些.o文件中包含的段相对link过程来说就是输入段，而Link的输出一般是可执行文件elf或库等，这些输出文件中也包含有段，这些输出文件中的段就叫做输出段。

输入段和输出段本来没有什么必然的联系，是互相独立，只是在Link过程中，Link程序会根据一定的规则（这些规则其实来源于Link Script），将不同的输入段重新组合到不同的输出段中，即使是段的名字，输入段和输出段可以完全不同。

###范例
```
#include <unistd.h>
#include <stdint.h>
#include <stdio.h>

typedef void (*myown_call)(void);

extern myown_call _myown_start;
extern myown_call _myown_end;

#define _init __attribute__((unused, section(".myown")))
#define func_init(func) myown_call _fn_##func _init = func

static void mspec1(void)
{
        write(1, "aha!\n", 5);
}

static void mspec2(void)
{
        write(1, "aloha!\n", 7);
}

static void mspec3(void)
{
        write(1, "hello!\n", 7);
}

func_init(mspec1);
func_init(mspec2);
func_init(mspec3);

/* exactly like below:
static myown_call mc1  __attribute__((unused, section(".myown"))) = mspec1;
static myown_call mc2  __attribute__((unused, section(".myown"))) = mspec2;
static myown_call mc3  __attribute__((unused, section(".myown"))) = mspec3;
*/

void do_initcalls(void)
{
        myown_call *call_ptr = &_myown_start;
        do {
                fprintf (stderr, "call_ptr: %p\n", call_ptr);
                (*call_ptr)();
                ++call_ptr;
        } while (call_ptr < &_myown_end);

}

int main(void)
{
        do_initcalls();
        return 0;
}
```
来自ld的链接脚本，可以使用：
```
ld --verbose > s.lds
```
获取内置lds脚本，并打开s.lds，同时在：
```
__bss_start = .;
```
之前添加以下内容：
```
_myown_start = .;
.myown           : { *(.myown) } = 0x90000000
_myown_end = .;
code_segment    : { *(code_segment) }
```
即定义了.myown段及_myown_start/_myown_end变量（0x90000000这个数值可能需要调整）。
保存并关闭lds文件，同时进行gcc编译。
```
gcc s.c -Wl,-Ts.lds
```
执行并查看结果
```
root@ubuntu:~$ ./a.out 
call_ptr: 0x601040
aha!
call_ptr: 0x601048
aloha!
call_ptr: 0x601050
hello!
```

gcc编译时错误排解
1. syntax error
可能报如下错误时
```
/usr/bin/ld:s.lds:1: syntax error
collect2: error: ld returned 1 exit status
```
解决方法:打开s.lds文件，将lds文件中的一些描述信息去除掉
2. 对XXX未定义的使用
可能的报错如下：
```
/tmp/cch1euXu.o：在函数‘do_initcalls’中：
test_session.c:(.text+0x90)：对‘_myown_end’未定义的引用
collect2: error: ld returned 1 exit status
```
解决方法：在lds中查看XXX是否书写正确。
##\_\_attribute__((weak))与\_\_attribute__((weakref(...)))
弱符号与弱引用

###说明
弱符号： 连接器发现同时存在弱符号和强符号，有限选择强符号，如果发现不存在强符号，只存在弱符号，则选择弱符号。如果都不存在：静态链接，恭喜，编译时报错，动态链接：对不起，系统无法启动。

弱引用： 默认的模块中对所有对外部符号的引用在链接时都是强引用，该符号必须能够被正确的“决议”（或者说“绑定”），否则连接器就会报错，GCC中科院通过 __attribute__((weakref))关键字显示的对一个外部符号的引用定义为弱引用，这样，即使连接的时候该符号没有正确的找到定义，链接 器也不会报错，而只是将该符号的值置为0。但这样得到的可执行程序在执行时会报错，因为当调用弱引用时，弱符号的地址为0，属于非法访问。因此，在程序中 调用一个外部符号时，应该先判断其值是否为0，若不为0再调用。

###范例
####弱符号
```
//main.c
int __attribute__((weak)) func(void)
{
	return 0;
}

int main()
{
	int a = func();
	if (a > 5){
		printf(" a > 5 \n");
	}else{
		printf(" a < 5 \n");
	｝
	return 0;
}
```
```
//func.c
int func(void)
{
	return 6;
}
```
编译、运行与查看结果
```
gcc main.c -o main
./main
a < 5

gcc main.c func.c -o main
./main
a > 5
```
####弱引用
```
//main.c

staic void func() __attribute__((weakref("bar")));
int main()
{
	if (func) {
		func();
	}else{
		printf("func not exist\n");
	}
}

```
```
//fun.c
void bar(void)
{
	print("this is bar.\n");
}
```
编译、运行与结果
```
gcc main.c -o main
./main
func not exist
gcc main.c func.c -o main
./main
this is bar
```
##__attribute__ ((visibility("default")))和__attribute__ ((visibility("hidden")))
visibility用于设置动态链接库中函数的可见性，将变量或函数设置为hidden，则该符号仅在本so中可见，在其他库中则不可见。

###说明
1. 在编译时，可用参数-fvisibility指定所有符号的可见性（不加此参数时默认外部可见，参考man g++中-fvisibility部分）；若需要对特定函数的可见性进行设置，需在代码中使用__attribute__设置visibility属性。

2. 编写大型程序时，可用-fvisibility=hidden设置符号默认隐藏，针对特定变量和函数，在代码中使用__attribute__ ((visibility("default")))另该符号外部可见，这种方法可用有效避免so之间的符号冲突。

###范例
```
//test.c
#include<stdio.h>  
#include<stdlib.h>  
 
__attribute ((visibility("default"))) void not_hidden ()  
{  
	printf("exported symbol\n");  
}  
 
__attribute ((visibility("hidden"))) void is_hidden ()  
{  
	printf("hidden one\n");  
}  
```
将test.c编译成so
```
gcc -shared -fPIC -o libtest.so test.c 
```
查看hiddlen
```
root@ubuntu:~$ readelf -s libtest.so |grep hidden
    11: 00000000000006a0    19 FUNC    GLOBAL DEFAULT   12 not_hidden
    40: 00000000000006b3    19 FUNC    LOCAL  DEFAULT   12 is_hidden
    55: 00000000000006a0    19 FUNC    GLOBAL DEFAULT   12 not_hidden
```
引用测试
```
//main.c
extern void not_hidden();
extern void is_hidden();

int main()
{
	not_hidden();
	is_hidden();
	return 0;
}
```
编译引用
```
root@ubuntu:~$gcc main.c -o main -L. -ltest
/tmp/ccXsI2E0.o：在函数‘main’中：
main.c:(.text+0x14)：对‘is_hidden’未定义的引用
collect2: error: ld returned 1 exit status
```
##\_\_attribute__((alias(...)))
alias属性用于设置一个函数的别名。
```
//main.c
int funca()
{
	printf("this is funca.\n");
}

int funcb() __attribute__((alias("funca)));

int main()
{
	funcb();
	return 0;
}
```
编译、运行与结果
```
gcc main.c -o main
./main
this is funca.
```
	
##attribute引用方法
```
#define _printf_attr_(a,b) __attribute__ ((format (printf, a, b)))
#define _alloc_(...) __attribute__ ((alloc_size(__VA_ARGS__)))
#define _sentinel_ __attribute__ ((sentinel))
#define _noreturn_ __attribute__((noreturn))
#define _unused_ __attribute__ ((unused))
#define _destructor_ __attribute__ ((destructor))
#define _pure_ __attribute__ ((pure))
#define _const_ __attribute__ ((const))
#define _deprecated_ __attribute__ ((deprecated))
#define _packed_ __attribute__ ((packed))
#define _malloc_ __attribute__ ((malloc))
#define _weak_ __attribute__ ((weak))
#define _likely_(x) (__builtin_expect(!!(x),1))
#define _unlikely_(x) (__builtin_expect(!!(x),0))
#define _public_ __attribute__ ((visibility("default")))
#define _hidden_ __attribute__ ((visibility("hidden")))
#define _weakref_(x) __attribute__((weakref(#x)))
#define _introspect_(x) __attribute__((section("introspect." x)))
#define _alignas_(x) __attribute__((aligned(__alignof(x))))
#define _cleanup_(x) __attribute__((cleanup(x)))
#define _weak_alisa_(x) __attribute__((weak, alis(x)))  
```


##参考
[\_\_attribute__](http://nshipster.com/__attribute__/)
[\_\_ATTRIBUTE__ 你知多少？](http://www.cnblogs.com/astwish/p/3460618.html)
[\_\_attribute__详解](http://blog.csdn.net/fcryuuhou/article/details/6667653)
[C语言字节对齐 \_\_align(),\_\_attribute((aligned (n))),#pragma pack(n)](http://www.cnblogs.com/ransn/p/5081198.html)
[#pragma pack(n)和\_\_attribute__((aligned(m)))的区别](http://blog.163.com/tyw_andy/blog/static/11679021200910635047652/)
[关于内存对齐的那些事](http://blog.csdn.net/markl22222/article/details/38051483)
[gcc的\_\_attribute__编译属性](http://blog.sina.com.cn/s/blog_5e11a56a0100c8h5.html###)
[利用 GCC 的 \_\_attribute__ 属性的 section 选项来控制数据区的基地址](http://blog.sina.com.cn/s/blog_679f93560102uzek.html)
[利用gcc的\_\_attribute__编译属性section子项构建初始化函数表](https://my.oschina.net/u/180497/blog/177206)
[\_\_attribute__((weak)) 博大精深的gcc ------ 关于弱符号的用法](http://blog.chinaunix.net/uid-7828352-id-4477460.html)
[__attribute__ 之weak,alias属性 .](http://blog.sina.com.cn/s/blog_a9303fd90101d5su.html)
[浅谈C语言中的强符号、弱符号、强引用和弱引用](http://www.jb51.net/article/56924.htm)
[gcc \_\_attribute__关键字举例之visibility](http://blog.csdn.net/starstarstone/article/details/7493144?utm_source=tuicool)
[GCC扩展 \_\_attribute__ ((visibility("hidden")))](http://liulixiaoyao.blog.51cto.com/1361095/814329)
[gcc __attribute__关键字举例之alias](http://blog.csdn.net/starstarstone/article/details/7490404)