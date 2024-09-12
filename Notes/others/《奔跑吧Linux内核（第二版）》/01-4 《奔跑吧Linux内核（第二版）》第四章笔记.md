# 《奔跑吧Linux内核（第二版）》第四章笔记

## 内核配置

内核配置工具常见的有：

-    make config
-    make oldconfig
-    make menuconfig

内核配置工具最终会在[Linux](https://so.csdn.net/so/search?q=Linux&spm=1001.2101.3001.7020)内核源码的根目录下生成一个隐藏文件——`.config`文件，这个文件包含了内核所有配置信息。

`.config`文件的每个配置选项都以“CONFIG”开头，后面的y表示内核会把这个特性编译进内核，m表示这个特性会被编译成内核模块。如果不需要编译到内核中，就要在前面用“#”进行注释，并在后面用“is not set”标识。

`.config`文件通常有几千行，每一行都手动配置显然不现实，那实际项目中如何生成这个`.config`文件呢？

#### 1\. 使用板级配置文件

一些芯片公司通常会提供基于某款SoC的开发板，读者可以基于此开发板来快速开发产品原型。芯片公司同时会提供板级开发板包，其中包含移植好的Linux内核。以ARM公司的Vexpress板子为例，该板子对应的Linux内核的配置文件存放在`arch/arm/configs`目录中。

我们可以使用它提供的配置文件配置内核。

    export ARCH=arm
    export CROSS_COMPILE=arm-linux-gnueabi-
    make vexpress_defconfig


#### 2\. 使用系统已有的配置文件

当我们需要编译计算机中的Linux内核时，可以使用系统自带的配置文件。以Ubuntu 20.04为例，boot目录下有一个config-5.4.0-26-generic文件。但我们想要编译一个新的内核时（如Linux 5.6内核），可以通过如下命令生成一个新的`.config`文件。

    cd linux-5.6
    cp /boot/config-5.4.0-26-generic ./.config

## 参考

[《奔跑吧Linux内核（第二版）》第四章笔记_奔跑linux-CSDN博客](https://blog.csdn.net/weixin_51760563/article/details/119973416)