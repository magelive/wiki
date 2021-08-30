# meson 

## 什么是Meson
Meson（The Meson Build System）是个项目构建系统，如Makefile，automake，CMake...。Meson是一个Python实现的开源项目，其思想是，开发人员花费在构建调试上的每一秒都是浪费，同样等待构建过程直到真正开始编译都是不值得的。

因此，Meson的设计目的是在用户友好的同时不损害性能，Meson提供客户语言（custom language）作为主要工具，用户可以使用它完成项目构建的描述。客户语言的设计目标是简单（simplicity）、清晰（clarity）、简洁（conciseness），其中很多灵感来源于Python语言。

Meson的另个一主要设计目的是为现代编程工具提供优秀的支持和最好的实现。这包括一些特性如：单元测试（unit testing）、代码覆盖率报告（code coverage reporting）、头文件预编译（precompiled headers）。用户不需要寻找三方宏指令（third party macros）或编写shell脚本来实现这些特性，Meson只要开箱即用（work out of the box）。

## Meson有什么特点
对Linux，macOS，Windows，GCC，Clang，Visual Studio等提供多平台支持
* 支持的语言包括C，C ++，D，Fortran，Java，Rust
* 在非常易读且用户友好的非图灵完整DSL中构建定义
* 适用于许多操作系统和裸机的交叉编译
* 针对极快的完整和增量构建进行了优化，而不会牺牲正确性
* 内置的多平台依赖提供程序，可与发行版软件包一起使用
* 好玩！
以上这些特征均来自官网的介绍，我们在接下来的使用过程中只会涉及部分特性。

## 如何使用Meson
这一章将会包含比较多的使用细节，会在一系列的文章中去完善该部分内容。

### Meson安装
首先是安装Python3.x版本，而且版本尽可能的高，我用的是Python3.5。一般来说安装Python是默认带有pip，但是如果系统缺一些库的话pip会不能成功安装，我踩到zlib的坑（pip3的安装可以参考https://www.cnblogs.com/fyly/p/11112169.html）

具备pip3之后直接安装Meson Ninja
```shell
pip3 install meson ninja
```

这边多出一个Ninja工具，简单介绍一下。Ninja是一个轻量的构建系统，主要关注构建的速度。它与其他构建系统的区别主要在于两个方面：

1. Ninja被设计成需要一个输入文件的形式，这个输入文件则由高级别的构建系统生成；
2. Ninja被设计成尽可能快速执行构建的工具。

一般将Meson和Ninja配合使用，Meson负责构建项目依赖关系，Ninja进行编译。

### 简单的Meson构建样例

构建项目首先需要对项目的构建需求进行描述，前面介绍过Meson提供custom language用于描述项目构建需求。

custom language包含诸多部分：变量，数值，字符串，数组，辞典，函数调用，方法调用，控制语句，功能函数，内置对象，返回对象...。暂时不对这些细节进行展开，从简单的示例开始。

#### 构建一个可执行项目
创建一个项目目录，包含一个main.c文件

```c
#include <stdio.h>
 
int main(void)
{
        printf("hellow project01\n");
        return 0;
}
```
创建Meson构建描述文件meson.build（指定文件名）
```meson.build
project('project01', 'c')
executable("project", 'src/main.c')
```
项目目录结构：
```shell
project01/
├── meson.build
└── src
    └── main.c
```
项目构建关系描述完成，接下来就需要通过调用Meson来生成构建目录及构建系统。这就涉及到meson command line的使用，当meson安装完成后可以通过meson -v查看Meson 版本，这就是命令行。
```shell
meson -h / --help
```
输出：
```shell
usage: meson [-h] {setup,configure,dist,install,introspect,init,test,wrap,subprojects,help,rewrite,compile} ...
 
optional arguments:
  -h, --help                                                                           show this help message and exit
 
Commands:
  If no command is specified it defaults to setup command.
 
  {setup,configure,dist,install,introspect,init,test,wrap,subprojects,help,rewrite,compile}
    setup                                                                              Configure the project
    configure                                                                          Change project options
    dist                                                                               Generate release archive
    install                                                                            Install the project
    introspect                                                                         Introspect project
    init                                                                               Create a new project
    test                                                                               Run tests
    wrap                                                                               Wrap tools
    subprojects                                                                        Manage subprojects
    help                                                                               Print help of a subcommand
    rewrite                                                                            Modify the project definition
    compile                          

```
接下来我们要使用setup 命令，它也是meson的默认命令，即 `meson xxx `与 `meson setup xxx `等价。

在meson.build目录执行
```shell
meson setup build
```
执行后有如下打印：
```shell
The Meson build system
Version: 0.55.3
Source dir: /home/yu.xinrong/meson_demo/project01
Build dir: /home/yu.xinrong/meson_demo/project01/build
Build type: native build
Project name: project01
Project version: undefined
C compiler for the host machine: cc (gcc 4.8.5 "cc (GCC) 4.8.5 20150623 (Red Hat 4.8.5-39)")
C linker for the host machine: cc ld.bfd 2.25.1-22
Host machine cpu family: x86_64
Host machine cpu: x86_64
Build targets in project: 1
 
Found ninja-1.10.0.git.kitware.jobserver-1 at /usr/bin/ninja
NOTICE: You are using Python 3.5 which is EOL. Start。ing with v0.57, Meson will require Python 3.6 or newer
```

信息有Meson版本，源码目录和构建目录，一部分构建选项的参数，并且最后提示ninja的存在，可以直接使用ninja编译。

Ninja编译项目：
```shell
cd build
ninja
```
编译完成发现build目录中出现project可执行文件。
```shell
[root@localhost build]# ls -al
total 28
drwxr-xr-x. 6 root root  180 Sep 20 14:34 .
drwxr-xr-x. 4 root root   49 Sep 20 14:32 ..
-rw-r--r--. 1 root root 2581 Sep 20 14:32 build.ninja
-rw-r--r--. 1 root root  341 Sep 20 14:32 compile_commands.json
drwxr-xr-x. 2 root root  264 Sep 20 14:32 meson-info
drwxr-xr-x. 2 root root   27 Sep 20 14:32 meson-logs
drwxr-xr-x. 2 root root  207 Sep 20 14:32 meson-private
-rw-r--r--. 1 root root  760 Sep 20 14:34 .ninja_deps
-rw-r--r--. 1 root root  131 Sep 20 14:34 .ninja_log
-rwxr-xr-x. 1 root root 9600 Sep 20 14:34 project
drwxr-xr-x. 2 root root   26 Sep 20 14:34 project.p
[root@localhost build]# ./project
hellow project01
```

#### 构建静态库项目
创建库文件项目，目录结构如下
```shell
project02
├── meson.build
└── src
    ├── third_lib.c
    └── third_lib.h
```

third_lib.c:
```c
#include <stdio.h>
#include "third_lib.h"
 
void info_print()
{
        printf("hellow third library\n");
}
third_lib.h:

#ifndef _THIRD_LIB_
#define _THIRD_LIB_
 
void info_print();
 
#endif
```

meson.build:
```meson
project('project02', 'c')
static_library('thirdinfo', 'src/third_lib.c')
```
构建及编译完成后生成静态库libthirdinfo.a：
```shell
[root@localhost build]# ls -al
total 20
drwxr-xr-x. 6 root root  194 Sep 20 14:45 .
drwxr-xr-x. 4 root root   49 Sep 20 14:45 ..
-rw-r--r--. 1 root root 2657 Sep 20 14:45 build.ninja
-rw-r--r--. 1 root root  412 Sep 20 14:45 compile_commands.json
-rw-r--r--. 1 root root 3556 Sep 20 14:45 libthirdinfo.a
drwxr-xr-x. 2 root root   31 Sep 20 14:45 libthirdinfo.a.p
drwxr-xr-x. 2 root root  264 Sep 20 14:45 meson-info
drwxr-xr-x. 2 root root   27 Sep 20 14:45 meson-logs
drwxr-xr-x. 2 root root  207 Sep 20 14:45 meson-private
-rw-r--r--. 1 root root  808 Sep 20 14:45 .ninja_deps
-rw-r--r--. 1 root root  150 Sep 20 14:45 .ninja_log
```

#### 构建加载三方库的可执行项目
```shell
project03
├── meson.build
└── src
    ├── include
    │   └── third_lib.h
    ├── main.c
    └── third
        └── libthirdinfo.a
```
此处用的静态库和头文件来自上一步的构建结果。

main.c:
```c
#include <stdio.h>
#include "third_lib.h"
 
int main(void)
{
        printf("hellow project03\n");
        info_print();
        return 0;
}
```
meson.build:
```meson
project('project03', 'c')
libs=meson.get_compiler('c').find_library('thirdinfo', dirs : join_paths(meson.source_root(),'src/third'))
executable('project03', 'src/main.c', dependencies : libs, include_directories : 'src/include')
```
构建编译之后：
```shell
[root@localhost build]# ls -al
total 28
drwxr-xr-x. 6 root root   184 Sep 20 16:34 .
drwxr-xr-x. 4 root root    49 Sep 20 16:34 ..
-rw-r--r--. 1 root root  2774 Sep 20 16:34 build.ninja
-rw-r--r--. 1 root root   368 Sep 20 16:34 compile_commands.json
drwxr-xr-x. 2 root root   264 Sep 20 16:34 meson-info
drwxr-xr-x. 2 root root    27 Sep 20 16:34 meson-logs
drwxr-xr-x. 2 root root   207 Sep 20 16:34 meson-private
-rw-r--r--. 1 root root   800 Sep 20 16:34 .ninja_deps
-rw-r--r--. 1 root root   135 Sep 20 16:34 .ninja_log
-rwxr-xr-x. 1 root root 10144 Sep 20 16:34 project03
drwxr-xr-x. 2 root root    26 Sep 20 16:34 project03.p
[root@localhost build]# ./project03
hellow project03
hellow third library
```

该篇文章主要对meson官网的句法部分和对象部分进行简单摘要，具体方法的详细用法还要参考官网描述。对象部分写的相对简单，在后续文章中，会以示例的形式逐一讲解。

## Meson句法
### 变量

Meson中的变量的工作方式与其他高级编程语言相同。变量可以包含任何类型的值，例如整数或字符串。变量无需预先声明，只需将其赋值即可出现。这是将值分配给两个不同变量的方法。
```meson
var1 = 'hello'

var2 = 102
```
Meson中变量如何工作的一个重要区别是所有对象都是不可变的。
```meson
var1 = [1, 2, 3]

var2 = var1

var2 += [4]

# var2 is now [1, 2, 3, 4]

# var1 is still [1, 2, 3]
```
### 数值
Meson仅支持整数。只需将其写出即可声明它们。支持基本的算术运算。
```meson
x = 1 + 2

y = 3 * 4

d = 5 % 3 # Yields 2.
```
 

自0.45.0版开始支持十六进制文字：

```meson
int_255 = 0xFF
```

自0.47.0版开始支持八进制和二进制文字：

```meson
int493 = 0o755

int_1365 = 0b10101010101
```
 

字符串可以转换为如下数字：
```meson
string_var = '42'

num = string_var.to_int()
```
 

数字可以转换为字符串：
```meson
int_var = 42

string_var = int_var.to_string()
```

### 布尔值
布尔值是true或false。
```meson
truth = true
```
### 字符串
Meson中的字符串用单引号声明。要输入文字单引号，请执行以下操作：
```meson
single quote = 'contains a \' character'
```

转义序列的完整列表为：
```meson
\\ 反斜杠

\' 单引号

\a 钟

\b 退格键

\f 换页

\n 新队

\r 回车

\t 水平制表符

\v 垂直标签

\ooo 具有八进制值的字符

\xhh 十六进制值hh的字符

\uxxxx 具有16位十六进制值xxxx的字符

\Uxxxxxxxx 具有32位十六进制值xxxxxxxx的字符

\N{name} Unicode数据库中名为name的字符
```

### 字符串连接
可以使用+符号将字符串连接起来以形成新的字符串。
```meson
str1 = 'abc'

str2 = 'xyz'

combined = str1 + '_' + str2 # combined is now abc_xyz
```

### 字符串路径构建
您可以使用/运算符来连接任意两个字符串以构建路径。/在所有平台上，它将始终用作路径分隔符。
```meson
joined = '/usr/share' / 'projectname' # => /usr/share/projectname

joined = '/usr/local' / '/etc/name' # => /etc/name

joined = 'C:\\foo\\bar' / 'builddir' # => C:/foo/bar/builddir

joined = 'C:\\foo\\bar' / 'D:\\builddir' # => D:/builddir
```

### 字符串跨越多行
跨越多行的字符串可以用三个单引号声明，如下所示：
```meson
multiline_string = '''#include <foo.h>

int main (int argc, char ** argv) {
return FOO_SUCCESS;

}'''
```

### 字符串格式化
可以使用字符串格式化功能来构建字符串。
```meson
template = 'string: @0@, number: @1@, bool: @2@'

res = template.format('text', 1, true)

# res now has value 'string: text, number: 1, bool: true'
```

### 数组
数组由方括号分隔。数组可以包含任意数量的任何类型的对象。
```meson
my_array = [1, 2, 'string', some_obj]
```

可以通过数组索引来访问数组的元素：
```meson
my_array = [1, 2, 'string', some_obj]

second_element = my_array[1]

last_element = my_array[-1] #类似python从后往前
```

您可以将更多项目添加到数组中，如下所示：
```meson
my_array += ['foo', 3, 4, another_obj]
```

添加单个项目时，不需要将其包含在数组中：
```meson
my_array += ['something']

# This also works

my_array += 'else'
```
注意追加到数组将始终创建一个新的数组对象并将其分配给它，my_array而不是修改原始数组对象，因为Meson中的所有对象都是不可变的。
 

从0.49.0开始，您可以检查数组是否包含如下元素。
```meson
my_array = [1, 2]

if 1 in my_array

# This condition is true

endif

if 1 not in my_array

# This condition is false

endif
```

### 数组方法
为所有数组定义了以下方法：

length，数组的大小

contains，true如果数组包含作为参数给出的对象，则返回，false否则返回

get，返回给定索引处的对象，负索引从数组的后面开始计数，超出范围索引是一个致命错误。提供向后兼容性，它与数组索引相同。

### 辞典
字典用花括号分隔。字典可以包含任意数量的键值对。键必须是字符串，值可以是任何类型的对象。在0.53.0之前，键必须是文字字符串。
```meson
my_dict = {'foo': 42, 'bar': 'baz'}
```
key必须唯一：
```meson
# This will fail

my_dict = {'foo': 42, 'foo': 43}

 ```

从0.49.0开始，您可以检查字典是否包含这样的键：
```meson
my_dict = {'foo': 42, 'bar': 43}

if 'foo' in my_dict

# This condition is true

endif

if 42 in my_dict

# This condition is false

endif

if 'foo' not in my_dict

# This condition is false

endif
```
 

由于0.53.0键可以是任何求值为字符串值的表达式，不再局限于字符串。
```meson
d = {'a' + 'b' : 42}

k = 'cd'

d += {k : 43}
```

### 函数调用
Meson提供了一组可用的函数。最常见的用例是创建构建对象。
```meson
executable('progname', 'prog.c')
```
大多数函数只接受很少的位置参数，而接受几个关键字参数，其指定方式如下：
```meson
executable('progname',

sources: 'prog.c',

c_args: '-DFOO=1')
```
从0.49.0版开始，可以动态指定关键字参数。这是通过传递表示要在关键字中设置的kwargs关键字的字典来完成的。前面的示例将这样指定：
```meson
d = {'sources': 'prog.c', 'c_args': '-DFOO=1'}

executable('progname', kwargs: d)
```

单个函数既可以直接在函数调用中使用kwargs关键字参数，也可以通过关键字参数间接获取关键字参数。唯一的限制是，将任何特定键作为直接参数和间接参数一起传递是错误的。
```meson
d = {'c_args': '-DFOO'}

executable('progname', 'prog.c', c_args: '-DBAZ=1', kwargs: d) # This is an error!
```
### 方法调用
对象可以具有用点运算符调用的方法。它提供的确切方法取决于对象。
```meson
myobj = some_function()

myobj.do_something('now')
```

### if语句

```meson
var1 = 1

var2 = 2

if var1 == var2 # Evaluates to false

something_broke()

elif var3 == var2

something_else_broke()

else

everything_ok()

endif

 

opt = get_option('someoption')

if opt != 'foo'

do_something()

endif
```

### 逻辑运算
Meson具有可在if语句中使用的标准逻辑操作范围 。

```meson
if a and b

# do something

endif

if c or d

# do something

endif

if not e

# do something

endif

if not (f or g)

# do something

endif
```

### Foreach语句

#### 用数组进行Foreach

这是一个示例，说明如何使用数组和foreach定义具有相应测试的两个可执行文件。
```meson
progs = [['prog1', ['prog1.c', 'foo.c']],

['prog2', ['prog2.c', 'bar.c']]]

foreach p : progs

exe = executable(p[0], p[1])

test(p[0], exe)

endforeach
```
 
#### 带有字典的Foreach

这是一个示例，您可以迭代应根据某些配置编译的一组组件。它使用字典，自0.47.0起可用。
```meson
components = {
    'foo': ['foo.c'],
    'bar': ['bar.c'],
    'baz': ['baz.c'],
}

# compute a configuration based on system dependencies, custom logic

conf = configuration_data()

conf.set('USE_FOO', 1)

# Determine the sources to compile

sources_to_compile = []

foreach name, sources : components

if conf.get('USE_@0@'.format(name.to_upper()), 0) == 1

sources_to_compile += sources

endif

endforeach
```

#### Foreach break和continue

从0.49.0开始break，continue关键字可以在foreach循环中使用。
```meson
items = ['a', 'continue', 'b', 'break', 'c']

result = []

foreach i : items

if i == 'continue'

continue

elif i == 'break'

break

endif

result += i

endforeach

# result is ['a', 'b']
``` 

### 注释
```meson
#
```

### 三元运算符
```meson
x = condition ? true_value : false_value
```

### 包含关系
大多数源树都有多个子目录要处理。这些可以通过Meson的subdir命令来处理。它将切换到给定的子目录并执行该子目录中的内容meson.build。所有状态（变量等）都往返于子目录。效果大致与将子目录的Meson文件的内容写入include命令所在的位置相同。
```meson
test_data_dir = 'data'

subdir('tests')
```
## 内置对象
### meson对象
meson对象主要是提供系统内部的参数。方法比较多详细参看官网。

* build_root() 构建根目录，通常是xxx/build

* source_root() 项目的源码目录

* current_build_dir() 使用subdir深入源码目录时，build目录同步深入

* current_source_dir() 当前编译到的源码目录

以上四个方法是体现了meson 源码后和构建分开的原则，假设有一个`project/`项目目录，`project/build` 为构建根目录， `project/src`为源码根目录。当编译到`project/src/module`时，`current_source_dir()`返回`project/src/module`,`current_build_dir()`返回`project/build/module`

### build_machine对象
进行实际构建的机器的主机，有如下方法：

* cpu_family():返回CPU族的名字（https://mesonbuild.com/Reference-tables.html#cpu-families）；

* cpu():返回一个更加具体的CPU名称。

* system():返回操作系统的名称（https://mesonbuild.com/Reference-tables.html#operating-system-names）；

* endian():大端系统返回big，小端系统返回little。

### host_machine对象
编译好的二进制文件运行的主机对象，方法与build_machine一致。

### target_machine对象
编译好的二进制文件的输出的运行主机，仅在程序产生特定主机输出时才有效，方法与build_machine一致。

build_machine、host_machine、target_machine在交叉编译时才有效。

### string对象
有如下方法：

* contains(string) string对象包含参数中的string返回true

* endswith(string) string对象以参数string结尾返回true

* format() 格式化

* join(list_of_strings) 加入划分符号

* split(split_character) 按符号划分

* startswith(string) string对象以参数string开始返回true

* substring(start,end) 返回子串

* strip() 去除whitespace

* to_int() 返回整数

* to_lower() 返回小写

* to_upper() 返回大写

* underscorify() 非字母或数字的符号全部用下划线代替

* version_compare(comparison_string) 版本对比

### Number对象
* is_even() 偶数返回true

* is_odd() 奇数返回true

* to_string() 返回字符串

### boolean对象
* to_int() 返回1 or 0

* to_string() 返回true or false 或定制bool.to_string('yes', 'no')

### array对象
* contains(item) 包含返回true

* get(index, fallback) 根据脚标获取元素，超出返回fallback

* length() 返回数据长度

### dictionary对象
* has_key(key) 判断key是否存在

* get(key, fallback) 根据key获取value

* keys() 以数组的形式返回所有的key

## 返回对象
### compiler对象
由`meson.get_compiler(lang)`方法返回，该对象代表了指定语言的编译对象，并且可以通过它查询相关属性。

### build target对象
build target代表构建的任何目标，包括可执行文件，动态库，静态库等。

### configuration data对象
由`configuration_data()`返回，并且包含了生成配置文件的配置的内容。

### custom target对象
由`custom_target()`返回。

### dependency对象
由`dependency()`返回。

### enternal program对象
由`find_program()`返回。

### environment 对象
由`environment()`返回。

### external library对象
由`find_library()`返回。

###  generator对象
由`generator()`返回。

### subproject对象
由`subproject()`返回。

### run result对象
由`run_command()`返回。


# 参考

[meson build](https://mesonbuild.com/)
[Meson构建系统（一）](https://blog.csdn.net/u010074726/article/details/108695256)
[Meson构建系统（二）](https://blog.csdn.net/u010074726/article/details/109010222)