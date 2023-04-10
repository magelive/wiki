# UBI

## 基础概念

* UBI文件系统：无排序区块镜像文件系统(Unsorted Block Image File System, UBIFS)是用于固态存储设备上，并与LogFS相互竞争，作为JFFS2的后继文件系统之一。

UBIFS涉及三个子系统

1. MTD子系统：flash驱动直接操作设备，而MTD在flash驱动之上，向上呈现统一的操作接口。所以MTD子系统的使命是：屏蔽不同flash的操作差异，向上提供统一的操作接口；对应drivers/mtd；

2. UBI子系统：UBI子系统是基于MTD子系统的，在MTD上实现nand特性的管理逻辑，向上屏蔽nand的特性；对应drivers/mtd/ubi；

3. UBIFS文件系统：是基于UBI子系统的文件系统，实现文件系统的所有基本功能。例如文件的实现，日志的实现；对应fs/ubifs；

ubi文件系统的结构:
[!avatar](images/ubi-arch.png)

## 参考（copy)
[UBI文件系统-----UBI文件系统概念、UBI文件系统开销、UBI文件系统使用方法](https://zhuanlan.zhihu.com/p/383367301)