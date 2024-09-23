# 实验 7-2：在 Ubuntu Linux 机器上新增一个系统调用

## 1．实验目的

​		通过新增一个系统调用，理解系统调用的实现过程。

## 2．实验详解

​		1）在 Ubuntu Linux 平台上新增一个系统调用，该系统调用不用传递任何参数，在该系统调用里输出当前进程的 PID 和 UID 值。该实验的目的是让读者学会如何在x86_64 里添加一个系统调用，并比较和 ARM32 系统的区别。

​		2）编写一个应用程序来调用这个新增的系统调用。实验要求添加的系统调用不传递任何参数，一般将 pid 和 uid 直接通过 printk 输出到 dmesg 中，但是这样非常的不优雅。该参考代码添加了一个系统调用。

```
long getpuid(pid_t *pid, uid_t *uid);
```

​		pid 和 uid 通过参数返回到用户空间。

​		为了方便进行实验，以下实验基于《奔跑吧 Linux 入门版》中第一章的实验 2，我们选定了最新的社区稳定版内核来修改添加系统调用，然后编译后，安装到 UbuntuLinux 机器上。

（1） 下载最新 Linux 内核

​		下载最新的社区稳定版内核。本实验采用 linux-5.1.16 版本，下载方法如下：

```
$ wget -c https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.1.16.tar.xz
$ xz -d linux-5.1.16.tar.xz 
$ tar -xf linux-5.1.16.tar 
$ cd linux-5.1.16/
```

（2） 添加系统调用 getpuid

​		打上我们提供的参考补丁。

```
$ patch –p1 < /
home/rlk/rlk_basic/runninglinuxkernel_5.0/rlk_lab/rlk_basic/chapter_6/lab2/
0001-x86-add-a-new-syscall-which-called-getpuid.patch
```

（3） 重新编译内核

为了方便，我们直接复制 Ubuntu Linux 系统中自带的配置文件，例如我的系统上

的配置文件为：/boot/config-5.4.0-26-generic，相关命令如下：

```
$ cd linux-5.1.16/
$ cp /boot/config-5.4.0-26-generic .config
$ make menuconfig
$ make -j4
```

编译时间取决于主机的处理能力。大概需要几十分钟。

编译完成之后，我们需要安装内核。

```
$ sudo make modules_install
$ sudo make install
```

安装完成后，重启电脑，用刚才编译的内核启动，登录系统。

（4） 编译测试程序。

进入到我们实验参考代码目录。

```
# cd 
/home/rlk/rlk_basic/runninglinuxkernel_5.0/rlk_lab/rlk_basic/chapter_6/lab2
# gcc test_getpuid_syscall.c -o test_getpuid_syscall
#./test_getpuid_syscall
```

![image-20240924011052671](image/image-20240924011052671.png)