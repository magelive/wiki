# qemu模拟arm环境

## qemu安装

1. 查看系统中qemu支持的arm版本
```shell
qemu-system-arm --version
qemu-system-arm -M ?
```

上[https://download.qemu.org](https://download.qemu.org) 查看最新的qemu源码版本

获取当前最新版本，6.1版本

```shell
wget https://download.qemu.org/qemu-6.1.0.tar.xz
```

下载完后，解压缩包：

```shell
tar -Jxf ./qemu-6.1.0.tar.xz
```

编译安装：

```shell
cd qemu-6.1.0
./configure
make && make install
```

查看版本：

```shell
qemu-system-arm --version
QEMU emulator version 6.1.0
Copyright (c) 2003-2021 Fabrice Bellard and the QEMU Project developers
```

## qemu-system-arm 命令使用

1. 查看支持的 machines

```shell
qemu-system-arm -M ?
```

2. 查看支持的cpu类型

```shell
qemu-system-arm -cpu ?
```
