# 《奔跑吧Linux内核（第二版）》第一章笔记

## [Linux](https://so.csdn.net/so/search?q=Linux&spm=1001.2101.3001.7020)发展简介

1991年Linus Torvalds正式发布第一版内核。  
1994年，采用GPL（General Public License）协议的Linux 1.0正式发布。  
1996年，Linux 2.0发布，该版本可以支持多种处理器，如alpha，mips，powerpc等，内核代码量大约是40万行。  
2003年，Linux 2.6发布。与Linux 2.4相比，该版本增加了很多性能优化的新特性，使它成为真正意义上的现代操作系统。  
2008年，谷歌正式发布Android 1.0，Android系统基于Linux内核来构建。在之后的十年里，Android系统占据了手机系统的霸主地位。  
2011年，Linux 3.0发布。  
2015年，Linux 4.0发布。  
2019年3月，Linux 5.0发布。

## Linux 5.0 内核目录结构：

-   arch：包含内核支持的所有处理器架构，比如 x86、ARM32、ARM64、RISC-V 等。这个目录包含和处理器架构紧密相关的底层代码。
    
-   block：包含块设备抽象层的实现。
    
-   certs：包含用于签名与检查的相关证书机制的实现。
    
-   crypto：包含加密机制的相关实现。
    
-   Documentation：包含内核的文档。
    
-   drivers：包含设备驱动。Linux 内核支持绝大部分的设备（比如 USB 设备、网卡设备、显卡设备等）驱动。
    
-   fs：内核支持的文件系统，包括虚拟文件系统层以及多种文件系统类型，比如 ext4文件系统、xfs文件系统等。
    
-   include：内核头文件。
    
-   init：包含内核启动的相关代码。
    
-   ipc：包含进程通信的相关代码。
    
-   kernel：包含内核核心机制的代码，比如进程管理、进程调度、锁机制等。
    
-   lib：包含内核用到的一些共用的库，内核不会调用 libc 库，而是实现了 libc 库的类似功能。
    
-   mm：包含与内存管理相关的代码。
    
-   net：包含与网络协议相关的代码。
    
-   samples：包含例子代码。
    
-   scripts：包含内核开发者使用的一些脚本，比如内核编译相关的脚本、检查内核补丁格式的脚本等。
    
-   security：包含与安全相关的代码。
    
-   sound：包含与声卡相关的代码。
    
-   tools：包含内核各个子模块提供的一些开发用的工具，比如 slabinfo 工具、perf工具等。
    
-   usr：包含创建 initramfs 文件系统的相关工具。
    
-   virt：包含虚拟化的相关代码。
    

## 重要实验

1、使用QEMU虚拟机来运行Linux系统  
2、创建基于Ubuntu Linux的根文件系统  
3、编译和运行一个ARM64 Linux最小系统  
4、动手DIY一个RISC-V的Debian系统



## 参考

[《奔跑吧Linux内核（第二版）》第一章笔记_奔跑吧linux内核入门篇(第2版) pdf-CSDN博客](https://blog.csdn.net/weixin_51760563/article/details/119766006)