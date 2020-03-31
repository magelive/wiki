#ubuntu 安装xrdp

在用RDP远程的时候总是卡在登录界面，connection problem,giving up， some problem...

经过一阵折腾，好像是Ubuntu 18.04.2本身有改动，让XRDP不能正常运行。所以需要做一些小改动。

第一步：

sudo apt-get install xserver-xorg-core
sudo apt-get -y install xserver-xorg-input-all



第二步，安装XRDP

sudo apt-get install xrdp



第三步：

sudo apt-get install xorgxrdp



#参考
[ubuntu远程桌面实现（包括解决connection problem,giving up问题）](https://zhuanlan.zhihu.com/p/93438433)
[Issues with xRDP and Ubuntu 18.04.2](https://c-nergy.be/blog/?p=13390)

