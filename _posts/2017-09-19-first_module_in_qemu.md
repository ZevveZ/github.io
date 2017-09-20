---
layout: post
title: "在QEMU上第一次成功编写模块"
categories: 日记
---
* content
{:toc}
为了更好地学习《Linux Device Drivers》这本书，我在QEMU上跑了Ubuntu5.04，想着它的内核是2.6.10的，照着书编写模块应该没什么问题（因为书上讲的内核也是这个版本），但今天第一次编写模块还是遇到了一些麻烦。





# Q&A
- 在Ubuntu上没有make和gcc工具，于是就想着可以通过apt-get的方式安装，但是提示要插入光盘，也就是说安装系统的光盘上已经有这些软件了，它不从网上下载。解决方法当然是插入光盘啦，但是在QEMU上要如何插入光盘呢?  
在QEMU的启动命令中加入-cdrom选项即可，启动Ubuntu后直接使用apt-get的方式安装所需软件即可，不需要挂载光盘~
- 编写好Makefile之后，make就报了一堆头文件没找到，包括stdio.h，怎么这些基本文件都找不到呢?  
需要通过apt-get安装build-essential，安装一些编译必须的东西
- 继续make，这回头文件都找到了，就是报了一堆类型错误，不应该呀。于是我就想着是不是源码树（从kernel.org下载的）和我现在Ubuntu上的内核不对应呀，虽然内核版本都是一样的，但是说不定Ubuntu修改了内核呢。于是我就开始了编译内核的道路
- 之前我在Ubuntu17.04上编译过内核没能成功，原因是我的编译工具版本不对，这回在Ubuntu5.04下一切就比较顺利了，主要过程如下,等对编译内核有进一步了解后再详细说明这个过程：
```
cd linux-2.6.10
make defconfig
//修改根目录下的.config,修改CONFIG_MODULES_UNLOAD=y，允许内核卸载模块；修改CONFIG_CIFS=y,因为主机与虚拟机之间通过samba共享文件；按照ubuntu5.04里面的设置修改千兆以太网，这样虚拟机才能够识别以太网
make
make bzImage
make modules
make modules_install
make install
mkinitrd -o /boot/initrd.img-2.6.10 2.6.10
修改/boot/grub/menu.lst，照着里面的内容新建一个引导项即可
重启按Esc选择内核
```
- 终于可以开心地编写内核模块了，但是我发现装载模块之后无法卸载模块，提示"Device busy"?  
将初始化函数的返回值设置为0即可，之前没有设置返回值
- 我照着书上在Makefile中只写了`obj-m:=hello.o`，但是使用书上的命令`make -C ~/linux-2.6.10 M =\`pwd\` modules`却不能成功编译模块?  
原来我在make命令中M和=号之间多了一个空格，导致不能成功编译模块，去掉空格之后就正常了

# 参考资料
- [ldd3学习笔记--环境搭建(构建linux2.6.10源码树)](http://blog.csdn.net/u013162593/article/details/45252383)
