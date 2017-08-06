## data[0]介绍
经常在阅读Linux下的代码时，会发现如下结构：
```
struct XXXX{
	int param1;
	float param2;
	...
	char data[0];
};
```
在此结构中，data是一个数组名；但该数组没有元素；该数组的真实地址紧随结构体XXXX之后，而这个地址就是结构体后面数据的地址（如果给这个结构体分配的内容大于这个结构体实际大小，后面多余的部分就是这个data的内容）；这种声明方法可以巧妙的实现C语言里的数组扩展。
注意上面结构体的最后一个字段，数组长度居然为0。只有GNU C允许使用这种用法，目的是为了访问不定长结构体时节省空间和便利性。

实际用时采取这样：
```
struct XXXX *p = (struct XXXX *)malloc(sizeof(struct XXXX )+LEN)
```
这样就可以通过p->data 来操作后面LEN长度的空间数据。
同样，我们也可以使用指针来达到这样的目的：
```
struct XXXX {
	int param1;
	float param2;
	...
	char *data;
};
```
两种使用方法：
```
struct XXXX *p = (struct XXXX *）malloc(sizeof(struct XXXX)+LEN);
p->data = (char *)p+sizeof(struct XXXX);
```
```
struct XXXX *p = (struct XXXX *)malloc(sizeof(struct XXXX));
p->data = (char *)malloc(LEN);
```

一个指针。一个可变长度数组，一般作为struct的最后一个元素（gcc扩展）。
编译器不会给char data[0]分配空间 能达到节省4或是8个字节的效果。
采用char *data，需要进行二次分配，操作比较麻烦，很容易造成内存泄漏。而直接采用变长的数组，只需要分配一次，然后进行取值即可以。

## char data[0], char *data, char data[]程序对比
```
//test_data0.c
#include <stdio.h>

typedef struct s1{
	int i;
	char data[0];
}s1;

typedef struct s2{
	int i;
	char *data;
}s2;

typedef struct s3{
  	int i;
 	char data[];
}s3;

int main()
{
 	printf("sizeof(s1) = %lu\n", sizeof(s1));
  	printf("sizeof(s2) = %lu\n", sizeof(s2));
 	printf("sizeof(s3) = %lu\n", sizeof(s3));

 	s1 bs1;
  	s2 bs2;
 	s3 bs3;

	printf("bs1 address : %p, i: %p, data : %p\n", &bs1, &bs1.i, &bs1.data);
 	printf("bs2 address : %p, i: %p, data : %p\n", &bs2, &bs2.i, &bs2.data);
 	printf("bs3 address : %p, i: %p, data : %p\n", &bs3, &bs3.i, &bs3.data);

 	return 0;
}
```
编译、运行与结果
```
gcc test_data0.c -o td
./td
sizeof(s1) = 4
sizeof(s2) = 16
sizeof(s3) = 4
bs1 address : 0x7ffc8cd040b0, i: 0x7ffc8cd040b0, data : 0x7ffc8cd040b4
bs2 address : 0x7ffc8cd040a0, i: 0x7ffc8cd040a0, data : 0x7ffc8cd040a8
bs3 address : 0x7ffc8cd040c0, i: 0x7ffc8cd040c0, data : 0x7ffc8cd040c4

```
结果可以看出data[0]和data[]不占用空间，且地址紧跟在结构后面，而char *data作为指针，占用8个字节，地址不在结构之后。
这里需要注意的是在x64（64位）的机器上sizeof(char \*)占的是8位字符宽，而x86（32位）的机器上sizeof(char \*)占4位字符宽。

## 参考
[char data[0]用法总结](http://blog.csdn.net/maopig/article/details/7243646)

[C语言变长数组data[0]【总结】](http://www.cnblogs.com/Anker/p/3744127.html)

[char *data 与 char data[0]有什么区别？](http://bbs.csdn.net/topics/350153324)

[GNU C中的零长度数组](http://blog.csdn.net/liuaigui/article/details/3680404)

[结构体中最后一个成员为[0]长度数组的用法](http://blog.chinaunix.net/uid-26750459-id-3191136.html)
