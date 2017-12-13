---
layout: post
title:  "编译linux内核的失败之旅"
categories: 日记
---
* content
{:toc}
因为最近在看《Linux Device Drivers》，这本书是按照2.6.10的内核来讲的，所以就打算自己编译一个内核方便学习，但因为宿主机自带的gcc版本太高不能够直接编译内核，总是会出现各种各样奇怪的问题。最后放弃编译内核，采用qemu跑ubuntu5.04虚拟机的方式。





## Q&A
1. 因为gcc6的版本默认会开启PIE，导致编译内核时会出现错误：code model kernel does not support PIC mode;解决方法是在内核的Makefile文件中添加关闭PIE的标志，但是在2.6的内核中，需要添加的标志是
```
CFLAGS += $(call cc-option, -fno-pie)
CFLAGS += $(call cc-option, -no-pie)
AFLAGS += $(call cc-option, -fno-pie)
CPPFLAGS += $(call cc-option, -fno-pie)
```
2. 虽然顺利解决了gcc的一个bug，但是继续编译下去还是出现了很多error，感觉这应该是一个无底坑，因此我决定安装一个低版本的gcc
3. 通过搜索发现有人成功用gcc3.3.5编译内核，于是在我的机器上编译gcc，还是出现了很多error，改了一些之后，发现这也是一个无底坑，果断放弃
4. 其实我只是想使用2.6.10的内核更好地学习《Linux Device Drivers》这本书，最后采取了一个折中的做法，我通过qemu安装了ubuntu5.04，因为这个系统的内核恰好就是2.6.10，后面的工作就在虚拟机上完成吧

## qemu使用虚拟机的过程
1. 下载qemu的源码编译，安装自己需要的东西，我使用的configure命令是：`./configure --enable-kvm --enable-vnc --target-list="i386-softmmu"`，可以开启其他选项，例如sdl
2. 制作磁盘映像：  `qemu-img create -f qcow2 ubuntu.img 4G`
3. 安装iso：  `qemu-system-i386  -m 1024 -enable-kvm -monitor stdio ubuntu.img -cdrom ubuntu-5.04-install-i386.iso`，默认是使用vnc进行显示；我如果使用sdl会出现键盘重复输入的问题
4. 安装vnc客户端Vinagre
5. 安装完iso后，下次直接使用  `qemu-system-i386  -m 1024 -enable-kvm -monitor stdio ubuntu.img`启动虚拟机

## 有价值的链接
- [ldd3学习笔记--环境搭建(构建linux2.6.10源码树)](http://blog.csdn.net/u013162593/article/details/45252383)
- [通过QEMU调试Linux内核](http://terenceli.github.io/%E6%8A%80%E6%9C%AF/2016/06/21/gdb-linux-kernel-by-qemu)
- [利用qemu进行内核源码级调试](http://blog.csdn.net/newnewman80/article/details/10154111)
