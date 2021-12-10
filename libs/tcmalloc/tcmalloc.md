# tcmalloc

## what's tcmalloc

TCMalloc (google-perftools) 是用于优化C++写的多线程应用，比glibc 2.3的malloc快。这个模块可以用来让MySQL在高并发下内存占用更加稳定。

TCMalloc 是 Google 开发的内存分配器，在不少项目中都有使用，例如在 Golang 中就使用了类似的算法进行内存分配。它具有现代化内存分配器的基本特征：对抗内存碎片、在多核处理器能够 scale。据称，它的内存分配速度是 glibc2.3 中实现的 malloc的数倍。

## install tcmalloc 

1. download 

```shell
git clone git@github.com:gperftools/gperftools.git
```

2. build and install

```shell
./configure --prefix=/xxx/install --enable-frame-pointers

make

```


## 参考

[【性能】tcmalloc 使用和原理](https://blog.csdn.net/bandaoyu/_article/details/108630996)
[Tcmalloc](https://blog.csdn.net/zgaoq/article/details/87875287)
[图解 TCMalloc](https://zhuanlan.zhihu.com/p/29216091)
[tcmalloc原理](https://www.jianshu.com/p/7c55fbdef679)
[TCMalloc：线程缓存的Malloc](http://www.linuxeye.com/Linux/1912.html)