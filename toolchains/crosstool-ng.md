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

