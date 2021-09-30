# uboot 开发指南

## 代码下载
```shell
git clone https://source.denx.de/u-boot/u-boot.git

git check -b v2021.10-rc2
```

## 开发环境配置

### toolchain 下载
因为使用arm环境来调试，因此直接下载arm工具链，当然牛X的可以自己编译对应的工具链，我懒得搞就直接下载：

linaro toolchain 官网地址如下：
[https://releases.linaro.org/components/toolchain/binaries](
https://releases.linaro.org/components/toolchain/binaries)

选一个合适的版本下载即可。

```shell
wget https://releases.linaro.org/components/toolchain/binaries/7.5-2019.12/arm-linux-gnueabi/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabi.tar.xz
```

下载完后，解压并将其bin路径添加至`PATH`中
```shell
export PATH=$PATH:${PWD}/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabi/bin/
```

## 调试环境配置

使用qemu来模拟arm板来调试uboot，因此首先要安装qemu-system-arm环境。
```shell
sudo apt install qemu qemu-system-arm
```

安装完后，可以使用`qemu-system-arm -M ?`来查看其支持的平台。

我们这里直接使用qemu-arm platfrom。

后续编译及运行参数都选的`QEMU 2.11 ARM Virtual Machine`。

## 编译 uboot

```shell
cd u-boot
make qemu_arm_defconfig ARCH=arm CROSS_COMPILE=arm-linux-gnueabi

make -j4 ARCH=arm CROSS_COMPILE=arm-linux-gnueabi-
```

## 运行

```shell
qemu-system-arm -M virt -m 512M -kernel u-boot -nographic -append "console=ttyAMA0"
```
