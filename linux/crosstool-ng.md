# crosstool-ng 构建工具链

## crosstool-ng 安装

```shell
git clone https://github.com/crosstool-ng/crosstool-ng

git checkout crosstool-ng-1.24.0

./configure --prefix=${PREFIX}

make && make install

```

## crosstool-ng 编译工具链

1. 查看默认配置

```shell
ct-ng list-samples
```

2. 默认配置

```shell
ct-ng arm-unknown-eabi
```

3. 修改配置

```shell
ct-ng menuconfig
```

4. 编译

```shell
ct-ng build
```

### 查找build后的目录

根据`.config`中的CT_PREFIX_DIR来决定去哪找build后的工具链。

```shell
ls #{CT_PREFIX_DIR}/* -hal
```

## 工具链介绍

交叉编译工具链命名规则：`arch [-vendor] [-os] [-(gnu)eabi]`

* arch - 体系架构，如ARM，MIPS
* verdor - 工具链提供商
* os - 目标操作系统
* eabi - 嵌入式应用二进制接口

不同交叉编译工具链简单说明：

* arm-none-eabi：没有操作系统的，不支持那些跟操作系统关系密切的函数（Glibc），使用嵌入式newlib专用库，常用于编译ARM7、Contex-M和contex-R系统的arm裸机系统（boot，kernel），不能编译应用软件。
* arm-none-linux-eabi：指定linux系统，使用Glibc
* arm-none-linux-gnueabi： 主要用于基于ARM架构的Linux系统，可用于编译 ARM 架构的 u-boot、Linux内核、linux应用等。使用Glibc，经过Codesourcery公司优化。
* arm-eabi-gcc：android ARM编译器
* arm-none-uclinuxeabi： 用于uCLinux，使用Glibc。
* arm-none-symbianelf： 用于symbian。

ABI 和 EABI的区别：

* ABI：二进制应用程序接口(Application Binary Interface (ABI) for the ARM Architecture)。在计算机中，应用二进制接口描述了应用程序（或者其他类型）和操作系统之间或其他应用程序的低级接口。

* EABI：嵌入式ABI。嵌入式应用二进制接口指定了文件格式、数据类型、寄存器使用、堆积组织优化和在一个嵌入式软件中的参数的标准约定。

两者主要区别，ABI是计算机上的，EABI是嵌入式平台上（如ARM，MIPS等）。

arm-linux-gnueabi-gcc 和 arm-linux-gnueabihf-gcc 区别:
两个交叉编译器分别适用于 armel 和 armhf 两个不同的架构，只是armel 和 armhf 这两种架构在对待浮点运算采取不同的策略。他们其实只是在gcc 的选项 -mfloat-abi 的默认值不同。gcc 的选项 -mfloat-abi 有三种值 soft、softfp、hard（其中后两者都要求arm 里有 fpu 浮点运算单元，soft 与后两者是兼容的，但 softfp 和 hard 两种模式互不兼容）：

* soft： 不用fpu进行浮点计算，即使有fpu浮点运算单元也不用，而是使用软件模式。

* softfp： armel架构（对应的编译器为 arm-linux-gnueabi-gcc ），用fpu计算，但是传参数用普通寄存器传，这样中断的时候，只需要保存普通寄存器，中断负荷小，但是参数需要转换成浮点的再计算。

* hard： armhf架构（对应的编译器 arm-linux-gnueabihf-gcc ），用fpu计算，传参数也用fpu中的浮点寄存器传，省去了转换，性能最好，但是中断负荷高。
