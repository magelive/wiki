# liteos

## 源码下载

```shell
git clone git@github.com:LiteOS/LiteOS.git
```

## realview-pbx-a9架构范例

### 编译

#### 工具链获取

1. 自己编译工具链
使用crosstool-ng build `arm-none-eabi` 工具链

2. 从ARM官方下载
官网下载如下：
`
wget https://armkeil.blob.core.windows.net/developer/Files/downloads/gnu-rm/9-2019q4/gcc-arm-none-eabi-9-2019-q4-major-x86_64-linux.tar.bz2
`

### 配置LiteOS config

1. 拷贝默认配置

```shell
cp tools/build/config/realview-pbx-a9.config .config
```

2. 设置配置项

```shell
make menuconfig
```

### qemu模拟运行

```shell
qemu-system-arm -machine realview-pbx-a9 -smp 4 -m 512M -kernel out/realview-pbx-a9/Huawei_LiteOS.bin -nographic
```

## 参考
[Huawei LiteOS | 概览(官方文档)](https://support.huaweicloud.com/LiteOS/index.html)