# 设备树

## 设备树概念

设备树(Device Tree)，将这个词分开就是“设备”和“树”，描述设备树的文件叫做DTS(Device Tree Source)，这个 DTS 文件采用树形结构描述板级设备，也就是开发板上的设备信息。

设备树的机制其实也是总线型的 BUS/Dev/Drv 模型，只是编写 Dev 的方式变了。即编写 设备树文件 .dts。dst 文件会被编译成 dtb 文件。dtb文件会传给内核, 内核会解析dtb文件, 构造出一系列的 device_node 结构体,device_node 结构体会转换为 platform_device 结构体。

设备树的一般操作方式是：开发人员根据开发需求编写dts文件，然后使用dtc将dts编译成dtb文件。



# 参考
[linux 设备树详解](https://blog.csdn.net/weixin_46640184/article/details/124271806)
[Linux 内核：设备树（1）dtb格式](http://www.manongjc.com/detail/24-fssksejhlfjkuqj.html)
[Linux驱动篇(六)——设备树之DTB结构（二)](https://zhuanlan.zhihu.com/p/144863497)
[linux内核设备树及编译](https://wenku.baidu.com/view/83622a577cd5360cba1aa8114431b90d6c8589bb.html)
[Device Tree 详解](https://kernel.meizu.com/device-tree.html)
