# 使用buildroot构建工具链

构建arm cortex-A9 工具链

## buildroot下载

* 下载

```shell
git clone git@github.com:buildroot/buildroot.git
```

* 选择版本

```shell
git checkout -b xxxx.xx.xx origin/xxxx.xx.xx
```

## buildroot 配置

* 进入配置界面

```shell
make menuconfig
```

* 配置ARCH

```menuconfig
Target options  --->
    Target Architecture ()  --->
        (X) ARM (little endian)
    Target Architecture Variant ()  --->
        (X) cortex-A9 
    Target ABI ()  --->
        (X) EABI
    Floating point strategy ()  --->
        (X) VFPv3-D16 
    ARM instruction set ()  ---> 
        (X) ARM
    ...
```

* 配置toolchain

```menuconfig
Toolchain  --->
    Toolchain type ()  --->
        (X) Buildroot toolchain
    (buildroot) custom toolchain vendor name 
    C library ()  --->
        (X) glibc
    Kernel Headers ()  --->
        (X) Linux 5.13.x kernel headers
    [*] Install glibc utilities
    Binutils Version ()  --->
        (X) binutils 2.37 
    GCC compiler Version ()  ---> 
        (X) gcc 10.x
    [*] Enable C++ support
    [*] Build cross gdb for the host
        GDB debugger Version ()  --->
            (X) gdb 10.x
    [*] Enable MMU support 
    ...
```

## build toolchain

```shell
make toolchain
```

## toolchain使用

```shell
dev@linux:buildroot$ ./output/host/bin/arm-buildroot-linux-gnueabi-gcc -v

dev@linux:buildroot$ ./output/host/bin/arm-buildroot-linux-gnueabi-gcc test.c -o test

dev@linux:buildroot$ file test
test: ELF 32-bit LSB shared object, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.3, for GNU/Linux 5.13.0, not stripped
```