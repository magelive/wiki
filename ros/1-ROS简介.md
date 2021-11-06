#ROS简介
##What's ROS
ROS(Robot Operating System)即机器人操作系统，它是一面向机器人的开源的元操作系统（meta-operating system）。它能够提供类似传统操作系统的诸多功能，如硬件抽象、底层设备控制、常用功能实现、进程间消息传递和程序包管理等。此外，它还提供相关工具和库，用于获取、编译、编辑代码以及多个计算机之间运行程序完成分布式计算。

##ROS support
* 分布式计算
* 软件复用
* 快速测试

##ROS target 
* 小型化
* ROS不敏感库
* 方便测试
* 可扩展

##ROS special description
* ROS不是一种编程语言，其主要由C++编写，其客户端库可使用python、jave、lisp等其它语言编写
* ROS不是一个函数库，除客户端库（Client Libraries）外还包含一个中心服务器（Central Server)、一系统命令行工具、图形化界面工具以及编译环境。
* ROS不是集成开发环境，可使用任何主流IDE进行ROS的软件开发，还可使用文本编辑器和命令行来完成相应的开发。

##ROS version
ROS的主要版本称为发行版，其版本号以顺序字母作为版本名的首字母来命名。
由新至旧版本号分别为：jade、indigo、hydro、groovy、feurte、
electric、diamondback、C Turtle和box turtle。

##ROS Build system
从groovy版本开始，ROS对软件的编译进行了重大改动。
在groovy及之前的版本中，ROS采用rosbuild系统来完成软件的编译，而在新的版本中，则改用catkin编译系统。
了解这一点非常重要，尤其在阅读参考教程时需要注意，编译系
统的改变导致很多教程分为rosbuild和catkin两个版本。
