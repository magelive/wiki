#ubuntu18.04 3.5mm 耳机无声音解决

在台式机中的ubuntu18.04 系统中插入3.5mm的耳机后，ubuntu系统无任何反应，使用ffplay播放音乐，耳机口没有任何声音出现。
针对此问题的现象进行排查与解决。

## 问题排查

### 耳机孔与耳机是否可用？
将耳机插入其它的同台式机的口中，播放音乐是正常的，因此排除硬件问题。

### 系统驱动是否正常？
将同事usb耳机插入至台式机中，发现其耳机使用ffplay播放的音乐是正常的，使用系统的声音面板播放测试声音是有声音的，因此基础的驱动是正常的。

### 软件是否正常？
同上，ffplay播放是正常的，因此软件的原因也先排除。


### 查看系统声音配置？

查看`/dev/snd`下的音频节点信息：
```
dev@ubuntu:~$ ls /dev/snd/
seq  timer
dev@ubuntu:~$
```

发现节点信息只有`seq`和`timer`。

因此初步定位是驱动的问题。

## 问题定位与解决
因为是ubuntu系统，因此首先使用系统自带的alsa进行设置。
```
dev@ubuntu:~$ alsamixer
cannot open mixer: No such file or directory"
dev@ubuntu:~$
```
`alsamixer` 直接报错。

```
dev@ubuntu:~$ lspci -vv |grep -iA5 audio
00:1f.3 Audio device: Intel Corporation Cannon Lake PCH cAVS (rev 10) (prog-if 80)
        Subsystem: Lenovo Device 3133
        Control: I/O- Mem+ BusMaster+ SpecCycle- MemWINV- VGASnoop- ParErr- Stepping- SERR- FastB2B- DisINTx+
        Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- <TAbort- <MAbort- >SERR- <PERR- INTx-
        Latency: 32, Cache Line Size: 64 bytes
        Interrupt: pin A routed to IRQ 142
        Region 0: Memory at b1230000 (64-bit, non-prefetchable) [size=16K]
        Region 4: Memory at b1000000 (64-bit, non-prefetchable) [size=1M]
        Capabilities: <access denied>
        Kernel driver in use: snd_hda_intel
        Kernel modules: snd_hda_intel, snd_soc_skl, snd_sof_pci
dev@ubuntu:~$
```
`lspci`是可以看到声卡信息的。
因此怀疑是alsa出问题了，重新至官网下载了alsa的驱动并重新安装。

### 重新安装alsa

#### 下载alsa驱动
在[alsa ftp服务器上](ftp://ftp.alsa-project.org/pub/)下载alsa对应的package，分别需要下载`alsa-fireware` `alsa-lib` `alsa-pulgin` `alsa-utils` 4个安装包。

#### 移除系统中原有的alsa库及包
```
sudo apt purge alsa-base alsa-utils
```

#### 安装新的alsa package
先安装`alsa-fireware` `alsa-lib` 和`alsa-pulgin`，再安装`alsa-utils`。
安装方式都是一样
```
./configure
make 
sudo make install
```

在安装`alsa-utils`时, `./configure`阶段报如下错误：
```
configure: error: No linkable libatopology was found. 
```

但在检测`alsa-lib`安装情况时，发现`/usr/lib`中是有该库的，
```
dev@ubuntu:~$ ls /usr/lib/libatopology.* -hal
-rwxr-xr-x 1 root root  992 1月  18 16:00 /usr/lib/libatopology.la
lrwxrwxrwx 1 root root   21 1月  18 16:00 /usr/lib/libatopology.so -> libatopology.so.2.0.0
lrwxrwxrwx 1 root root   21 1月  18 16:00 /usr/lib/libatopology.so.2 -> libatopology.so.2.0.0
-rwxr-xr-x 1 root root 510K 1月  18 16:00 /usr/lib/libatopology.so.2.0.0
dev@ubuntu:~$
```
同时`/usr/lib/pkgconfig/`下也有对应的信息：
```
dev@ubuntu:~$ cat /usr/lib/pkgconfig/alsa-topology.pc
Name: alsa-topology
Description: Advanced Linux Sound Architecture (ALSA) - Topology Library
Version: 1.2.4
Requires: alsa >= 1.2.4
Libs: -latopology
dev@ubuntu:~$ cat /usr/lib/pkgconfig/alsa.pc
prefix=/usr
exec_prefix=/usr
libdir=${exec_prefix}/lib
includedir=${prefix}/include

Name: alsa
Description: Advanced Linux Sound Architecture (ALSA) - Library
Version: 1.2.4
Requires:
Libs: -L${libdir} -lasound
Libs.private: -lm -ldl -lpthread -lrt
Cflags: -I${includedir}
dev@ubuntu:~$
```
但`configure`过程中，却发现检测到`alsa`库的版本不是该pc文件中对应的1.2.4版本。

因此，在`/usr/lib/x86_64-linux-gnu/`下还发现低版本的`alsa`库，因此需要先清除掉目录下的alsa相关库。

后重新`configure` `alsa-utils`就正常了。

但在使用`alsaconf`配置时，还提示如下错误：
```
No supported PnP or PCI card found.
```

使用`alsamixer`还是报一样的错，问题还未解决，因此问题的原因不在`alsa`。

既然问题不在`alsa`，那问题可能就在更底层的驱动上。但Linux一般都带有声卡驱动的。

### 驱动排查
从前面的`lspci`中可以看出，音频所需要的驱动为`snd_hda_intel`, `snd_soc_skl`和`snd_sof_pci`。
```
dev@ubuntu:~$ lsmod  |grep snd
snd_hda_codec_hdmi     65536  1
snd_hda_codec_realtek   139264  1
snd_hda_codec_generic    81920  1 snd_hda_codec_realtek
ledtrig_audio          16384  1 snd_hda_codec_generic
snd_hda_intel          53248  6
snd_intel_dspcfg       24576  1 snd_hda_intel
soundwire_intel        40960  1 snd_intel_dspcfg
snd_hda_codec         147456  4 snd_hda_codec_generic,snd_hda_codec_hdmi,snd_hda_intel,snd_hda_codec_realtek
snd_hda_core           94208  5 snd_hda_codec_generic,snd_hda_codec_hdmi,snd_hda_intel,snd_hda_codec,snd_hda_codec_realtek
snd_hwdep              16384  1 snd_hda_codec
snd_soc_core          282624  1 soundwire_intel
snd_compress           28672  1 snd_soc_core
ac97_bus               16384  1 snd_soc_core
snd_pcm_dmaengine      16384  1 snd_soc_core
snd_pcm               118784  8 snd_hda_codec_hdmi,snd_hda_intel,snd_hda_codec,soundwire_intel,snd_compress,snd_soc_core,snd_hda_core,snd_pcm_dmaengine
snd_timer              40960  1 snd_pcm
snd                    94208  22 snd_hda_codec_generic,snd_hda_codec_hdmi,snd_hwdep,snd_hda_intel,snd_hda_codec,snd_hda_codec_realtek,snd_timer,snd_compress,snd_soc_core,snd_pcm
soundcore              16384  1 snd
```

使用`lsmod`查看load到内核的驱动模块，发现除了`snd_sof_pci`，其它都是正常的。
因此尝试升级系统内核试试。
### 内核升级
升级内核的原因，在于使用google搜索该问题时，发现在kernel-5.2~5.3中alsa好像有异常。因此尝试升级内核来解决此问题。
升级脚本如下：
```sh
#!/bin/sh

# 64bit 升级方法如下，32bit的将下面链接中的amd64换成i386
kernel=($(wget -qO- https://kernel.ubuntu.com/~kernel-ppa/mainline/ | awk -F'\"v' '/v[4-9]./{print $2}'|cut -d/ -f1|grep -v - |sort -V | awk 'END {print}'))

deb_name=($(wget -qO- https://kernel.ubuntu.com/~kernel-ppa/mainline/v${kernel}/ | grep "linux-image" | grep "generic" | awk -F'\">' '/amd64.deb/{print $2}'| cut -d'<' -f1 | head -1))
deb_kernel_url="https://kernel.ubuntu.com/~kernel-ppa/mainline/v${kernel}/${deb_name}"
deb_kernel_name="linux-image-${kernel}-amd64.deb"
modules_deb_name=($(wget -qO- https://kernel.ubuntu.com/~kernel-ppa/mainline/v${kernel}/ | grep "linux-modules" | grep "generic" | awk -F'\">' '/amd64.deb/{print $2}' | cut -d'<' -f1 | head -1))
deb_kernel_modules_url="https://kernel.ubuntu.com/~kernel-ppa/mainline/v${kernel}/${modules_deb_name}"
deb_kernel_modules_name="linux-modules-${kernel}-amd64.deb"

wget -c -t3 -T60 -O ${deb_kernel_name} ${deb_kernel_url}
wget -c -t3 -T60 -O ${deb_kernel_modules_name} ${deb_kernel_modules_url}
dpkg -i ${deb_kernel_modules_name} ${deb_kernel_name}
update-grub
```

使用`modprobe --show-depends`将内核依赖项全部打印出来，看是否有依赖项没有安装。
```
dev@ubuntu:~$ modprobe --show-depends snd_hda_intel
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/soundcore.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/core/snd.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/core/snd-timer.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/core/snd-pcm.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/core/snd-pcm-dmaengine.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/ac97_bus.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/core/snd-compress.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/soc/snd-soc-core.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/drivers/soundwire/soundwire-bus.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/core/snd-hwdep.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/hda/snd-hda-core.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/pci/hda/snd-hda-codec.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/drivers/soundwire/soundwire-cadence.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/drivers/soundwire/soundwire-generic-allocation.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/drivers/soundwire/soundwire-intel.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/hda/snd-intel-dspcfg.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/pci/hda/snd-hda-intel.ko
dev@ubuntu:~$ modprobe --show-depends snd_soc_skl
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/soundcore.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/core/snd.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/core/snd-timer.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/core/snd-pcm.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/core/snd-pcm-dmaengine.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/ac97_bus.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/core/snd-compress.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/soc/snd-soc-core.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/drivers/soundwire/soundwire-bus.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/hda/snd-hda-core.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/drivers/soundwire/soundwire-cadence.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/drivers/soundwire/soundwire-generic-allocation.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/drivers/soundwire/soundwire-intel.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/hda/snd-intel-dspcfg.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/soc/snd-soc-acpi.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/soc/intel/common/snd-soc-acpi-intel-match.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/hda/ext/snd-hda-ext-core.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/soc/intel/common/snd-soc-sst-dsp.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/soc/intel/common/snd-soc-sst-ipc.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/soc/intel/skylake/snd-soc-skl.ko
dev@ubuntu:~$ modprobe --show-depends snd_sof_pci
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/soundcore.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/core/snd.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/core/snd-timer.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/core/snd-pcm.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/core/snd-pcm-dmaengine.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/ac97_bus.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/core/snd-compress.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/soc/snd-soc-core.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/drivers/leds/trigger/ledtrig-audio.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/drivers/soundwire/soundwire-bus.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/core/snd-hwdep.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/hda/snd-hda-core.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/pci/hda/snd-hda-codec.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/drivers/soundwire/soundwire-cadence.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/drivers/soundwire/soundwire-generic-allocation.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/drivers/soundwire/soundwire-intel.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/hda/snd-intel-dspcfg.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/soc/snd-soc-acpi.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/soc/intel/common/snd-soc-acpi-intel-match.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/hda/ext/snd-hda-ext-core.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/soc/sof/snd-sof.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/soc/sof/intel/snd-sof-intel-hda.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/soc/sof/xtensa/snd-sof-xtensa-dsp.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/soc/codecs/snd-soc-hdac-hda.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/soc/sof/intel/snd-sof-intel-hda-common.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/soc/sof/intel/snd-sof-intel-ipc.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/soc/sof/intel/snd-sof-intel-byt.ko
insmod /lib/modules/5.10.8-051008-generic/kernel/sound/soc/sof/snd-sof-pci.ko
```

如上显示的驱动信息，这三个驱动安装都是正常的。
再使用`alsamixer`播放还是出现同样的问题。没办法再度google。

### 问题解决
在如下链接[sound-stopped-working-after-upgrading-to-linux-5-4-intel-hd-audio](https://superuser.com/questions/1509312/sound-stopped-working-after-upgrading-to-linux-5-4-intel-hd-audio)中发现类似的问题。
试着用下面的方法：

1. `sudo vim /etc/default/grub`
2. 查找 `GRUB_CMDLINE_LINUX_DEFAULT` 行并且添加 `snd_hda_intel.dmic_detect=0` 在其后面. (例如: `GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 snd_hda_intel.dmic_detect=0"`)
3. `sudo grub-mkconfig -o /boot/grub/grub.cfg` 更新grub命令。
4. Reboot the system.

重启系统后再`ls /dev/snd`下发现：
```
dev@ubuntu:ubuntu_kernel$ ls /dev/snd/
by-path    hwC0D0  pcmC0D0c  pcmC0D10p  pcmC0D3p  pcmC0D8p  seq
controlC0  hwC0D2  pcmC0D0p  pcmC0D2c   pcmC0D7p  pcmC0D9p  timer
```

### 原因分析
goolge一圈没找着，猜测应该是内核接口变动，但驱动未来得及变动导致。

# 参考：
[解决Linux只有Nvidia HDM声卡输出的现象](https://blog.csdn.net/weixin_43594034/article/details/104855855)

[Sound stopped working after upgrading to Linux 5.4 (Intel HD Audio)](https://superuser.com/questions/1509312/sound-stopped-working-after-upgrading-to-linux-5-4-intel-hd-audio)

[Fix No Sound (Dummy Output) Issue In Ubuntu With SND HDA Intel](https://www.linuxuprising.com/2018/06/fix-no-sound-dummy-output-issue-in.html)

[Linux 音频驱动实验遇到的问题 ](http://47.111.11.73/thread-309511-1-1.html)

[Ubuntu18.04声卡无声音解决方案](https://blog.csdn.net/Fenglin6165/article/details/89311402)

[声卡在Ubuntu 18.04中显示为Dummy Output](https://qastack.cn/ubuntu/1059619/sound-card-shown-as-dummy-output-in-ubuntu-18-04)

[alsa ubuntu声卡驱动重新安装](https://blog.csdn.net/utstarm/article/details/6832184)

[记一次解决在Ubuntu 18.04下声卡没有声音的经历](https://ywnz.com/linuxjc/2331.html)
