# fat 文件系统

## 创建fat32 镜像

1. 创建镜像

```shell
dd if=/dev/zero of=test.img count=256 bs=1K
fdisk test.img
mkfs.vfat test.img
```

2. 挂载文件系统

```shell
mkdir ./test
mount test.img ./test
```

3. copy data

```shell
cp data ./test
```

4. umount

```shell
umount ./test
```

## Fat格式详解



