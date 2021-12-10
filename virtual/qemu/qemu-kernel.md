# qemu vexpress-a9 linux kernel

## kernel download

```shell
git clone git@github.com:torvalds/linux.git
```

## kernel build

### toolchain set

```shell
#!/bin/bash
export PATH=$PATH:${toolchain}/gcc-arm-none-eabi-9-2019-q4-major/bin
```

### env

```shell
export ARCH=arm
export CROSS_COMPILE=arm-none-eabi-
```

### config

```shell
make vexpress_defconfig
```

### make

```shell
make -j8
```

### test

```shell
qemu-system-arm -M vexpress-a9 -m 512M -kernel arch/arm/boot/zImage -dtb arch/arm/boot/dts/vexpress-v2p-ca9.dtb -nographic -append "console=ttyAMA0"
```

## rootfs

### busybox

* download

```shell
git clone https://git.busybox.net/busybox
git checkout -b 1.34 origin/1_34_stable
```

* config

```shell
make menuconfig
```

```txt
Settings  --->
    [*] Build static binary (no shared libs) 
    (arm-unknown-linux-gnueabi-) Cross compiler prefix //设置编译工具链前缀
    (./_install) Destination path for 'make install'
```

* make and install

```shell
make && make install
```

*注意：busybox编译时，需要带有libc库的工具链，否则会报错各种头文件找不到*


### rootfs

#### 准备文件

```shell
mkdir rootfs
cp _install/* rootfs
mkdir -p proc sys tmp root var mnt etc lib dev etc/init.d
touch etc/fstab etc/inittab etc/profile etc/init.d/rcS
echo "vexpress" > etc/hostname

cp ~/${TOOLCHAINS_DIR}/sysroot/lib/* rootfs/lib -rvf
rm rootfs/lib/*.a

arm-unknown-linux-gnueabi-strip rootfs/lib/*.so

fakeroot mknod tty1 c 4 1
fakeroot mknod tty2 c 4 2
fakeroot mknod tty3 c 4 3
fakeroot mknod tty4 c 4 4
fakeroot mknod console c 5 1
fakeroot mknod null c 1 3
```

#### etc 文件夹下各文件内容

在创建完etc下各文件目录后，需要将如下各文件的内容拷贝至etc各文件中。

* etc/fstab

```txt
#/etc/fstab
#device      mount-point        type    options         dump    fsck order
proc    /proc      proc defaults                0       0
tmpfs   /tmp    tmpfs   defaults                0       0
sysfs   /sys    sysfs   defaults                0       0
tmpfs   /dev    tmpfs   defaults                0       0
var     /dev    tmpfs   defaults                0       0
ramfs   /dev    ramfs   defaults                0       0
debugfs      /sys/kernel/debug  debugfs         defaults        0       0
```

_注意：debubfs需要内核打开CONFIG_DEBUG_FS支持，否则会提示debugfs mount 失败。_

* etc/inittab

```txt
# /etc/inittab
::sysinit:/etc/init.d/rcS
console::respawn:-/bin/sh
::ctrlaltdel:/sbin/reboot
::shutdown:/bin/umount -a -r
::restart:/sbin/init

```

* etc/profile

```txt
# etc/profile
USER="root"
LOGNAME=$USER
export HOSTNAME=`/bin/hostname`
export USER=root
export HOME=/root
export PS1="[$USER@$HOSTNAME \W]\# "
PATH=/bin:/sbin:/usr/bin:/usr/sbin
LD_LIBRARY_PATH=/lib:/usr/lib:$LD_LIBRARY_PATH
export PATH LD_LIBRARY_PATH
```

* /etc/init.d/rcS

```txt
#!/bin/sh
# /etc/init.d/rcs
PATH=/sbin:/bin:/usr/sbin:/usr/bin
runlevel=S
prevlevel=N
umask 022
export PATH runlevel prevlevel

mount -a
mkdir -p /dev/pts
mkdir -p /sys/kernel/debug
mkdir -p /sys/kernel/debug
mount -t devpts devpts /dev/pts
#mount -n -t usbfs none /proc/bus/usb
#echo /sbin/mdev > /proc/sys/kernel/hotplug
mdev -s
mkdir -p /var/lock

ifconfig lo 127.0.0.1
#ifconfig eth0 10.238.233.5

/bin/hostname -F /etc/hostname

```

#### 创建rootfs

```shell

dd if=/dev/zero of=rootfs.ext bs=1M count=256

mkfs.ext4 rootfs.ext

mkdir tmpfs

sudo mount -t ext4 rootfs.ext tmpfs/ -o loop

sudo cp rootfs/* tmpfs/ -rvf

sudo umount tmpfs

```

## run

```shell
qemu-system-arm -M vexpress-a9 -m 512M -kernel linux/arch/arm/boot/zImage -dtb linux/arch/arm/boot/dts/vexpress-v2p-ca9.dtb -nographic -append "root=/dev/mmcblk0 rw console=ttyAMA0" -sd rootfs.ext
```