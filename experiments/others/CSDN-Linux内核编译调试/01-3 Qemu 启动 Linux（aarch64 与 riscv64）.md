# Qemu 启动 Linux（aarch64 与 riscv64）

### Qemu 启动 Linux（aarch64 与 riscv64）

> 我的平台架构为 x86\_64，[操作系统](https://so.csdn.net/so/search?q=%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F&spm=1001.2101.3001.7020)为 Ubuntu22.04.3
>
> 这部分涉及内核的编译以及文件系统的制作
>
> 本博客中涉及了aarch64以及riscv64，以及临时文件系统以及永久文件系统，按需选择

#### 零、前提：安装Qemu以及aarch64与riscv64的交叉工具链

-   Qemu的安装

可以参考我之前的博客：https://blog.csdn.net/jingyu\_1/article/details/135625477

-   ARM 官方的交叉编译工具链：

https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads

根据平台架构和操作系统（我这里是x86\_64，[ubuntu](https://so.csdn.net/so/search?q=ubuntu&spm=1001.2101.3001.7020)）选择如下：

**x86\_64 Linux hosted cross toolchains**

AArch64 GNU/Linux target (**aarch64-none-linux-gnu**)

链接为：[arm-gnu-toolchain-13.2.rel1-x86\_64-aarch64-none-linux-gnu.tar.xz](https://developer.arm.com/-/media/Files/downloads/gnu/13.2.rel1/binrel/arm-gnu-toolchain-13.2.rel1-x86_64-aarch64-none-linux-gnu.tar.xz?rev=22c39fc25e5541818967b4ff5a09ef3e&hash=E7676169CE35FC2AAECF4C121E426083871CA6E5)

ARM官方提供的是二进制可执行文件，可以直接执行，无需编译

-   RISCV 交叉编译工具链：

https://github.com/riscv-collab/riscv-gnu-toolchain  
【官网也提供了相应的编译好的工具链，可以直接下载】

```
# 1、官网的意思是，在构建时会自动下载需要的包
git clone https://github.com/riscv/riscv-gnu-toolchain
# 2、先 clone 仓库，然后添加仓库里的子模块
git clone https://github.com/riscv/riscv-gnu-toolchain
cd riscv-gnu-toolchain
git submodule update --init --recursive
# 3、尽管有魔法，但是我上述两种方式都不成功，在 clone 时指定 --recursive 参数能成功下载
git clone https://github.com/riscv/riscv-gnu-toolchain --recursive
```

其他老哥的方案（借助gitee）：https://blog.csdn.net/limanjihe/article/details/122373942

我使用的是 ubuntu，所以需要以下依赖（其他操作系统查仓库）

```
sudo apt-get install autoconf automake autotools-dev curl python3 python3-pip libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev ninja-build git cmake libglib2.0-dev
```

```
./configure --prefix=/opt/riscv
make linux	# 可以添加 -j4 参数加快编译的速度, 可能出现权限不够, 使用 sudo make 即可
```

-   将qemu以及交叉编译工具添加到环境变量

```
vim ~/.bashrc
# 添加如下内容, 注意这里的路径根据你自己实际安装的路径进行配置
export PATH=$PATH:/opt/qemu/bin
export PATH=$PATH:/home/jingyu/arm-gnu-toolchain-x86_64-aarch64-none-linux-gnu/bin
export PATH=$PATH:/opt/riscv/bin
```

> 如果需要构建裸机 aarch64 以及 riscv64 交叉编译工具链： **aarch64-none-elf-** 与 **riscv64-unknown-elf-**
>
> 可以参考我的另一篇博客：https://blog.csdn.net/jingyu\_1/article/details/135631512

#### 一、编译 linux 内核

Linux 内核官网：https://www.kernel.org/

可以在其中选择想要编译的版本，点击`tarball`下载对应的压缩包，我下载的是`linux-6.7.tar.xz`

```
# 解压刚刚下载的压缩包
tar -xf linux-6.7.tar.gz
```

交叉编译内核，需要指定一些参数，就比如 ARCH，在其 Makefile 中有下面的描述：

```
# 对其他架构进行交叉编译时，应将 ARCH 设置为目标架构（请参见 arch/*）
# ARCH 可以在调用 make 时设置：
# make ARCH=arm64
# 另一种方法是在环境中设置 ARCH, 默认的 ARCH 是执行 make 的主机
# CROSS_COMPILE 指定使用的前缀
```

所以 `ARCH` 可以指定什么架构要看源码下的 `arch/*`

其中包括了 `arm`、`arm64`、`riscv`

其中 `CROSS_COMPILE` 和指定的架构 `ARCH` 匹配就可以

> 没有 riscv64 的选项，编译 riscv 版本的内核只需选择 riscv 即可，不需要像 arm 那样区分
>
> **对于一般情况下的交叉编译内核，配置这两个内容即可开始编译内核了**（对于其他定制化的配置，按需配置）

##### 关于配置

可以参考：https://zhuanlan.zhihu.com/p/94221380

配置这里选择默认配置：`defconfig`

或者使用可视化的配置，可以选择 `menuconfig`

初学最常用的就是这两个配置，在 `menuconfig` 中可以对默认配置进一步进行配置（对我目前来说已经够了），两者都要使用配置 `ARCH` 以及 `CROSS_COMPILE`，否则会出现默认x86（默认与当前主机平台一致）

> 注意的一点是（尝试中发现的一点细节）：**默认的配置中将 ext4 文件系统的驱动配置进去了 linux 内核**，所以可以直接挂载 ext4 文件系统（随后会涉及）

##### ARM64 架构

```
# 配置并编译内核, defconfig 表示 default config 默认配置
cd linux-6.7
make ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu- defconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu- -j $(nproc)
```

完成后会显示：

```
Kernel: arch/arm64/boot/Image.gz is ready
```

##### RISCV 架构

```
# 配置并编译内核, defconfig 表示 default config 默认配置
cd linux-6.7
make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- defconfig
make ARCH=riscv CROSS_COMPILE=riscv64-unknown-linux-gnu- -j $(nproc)
```

完成后会显示：

```
Kernel: arch/riscv/boot/Image.gz is ready
<div class="hljs-button {2}" data-title="复制"></div>
```

#### 二、制作 rootfs（重要）

> 嵌入式系统三大部分：**bootloader(eg: uboot)、Linux内核、根文件系统**
>
> 制作根文件系统有三大神器：**busybox、buildroot、yocto**
>
> 1.  使用 busybox 构建文件系统，仅仅只是帮我们构建好了一些常用的命令和文件，像 lib 库、/etc 目录下的一些文件都需要自己手动创建，我们还要自己去移植一些第三方软件和库，比如 alsa、iperf、mplayer 等等，而且 busybox 构建的根文件系统默认没有用户名和密码设置
> 2.  如果想要构建完整的根文件系统，**大家一般都是使用buildroot，它不仅包含了 busybox 的功能**，而且里面还集成了各种软件，需要什么软件就选择什么软件，不需要我们去移植，buildroot 极大的方便了我们嵌入式 Linux 开发人员构建实用的根文件系统
> 3.  至于 yocto 构建根文件系统，过于复杂，需要时间也很久，我们一般不会选择这一种方式
>
> 参考：https://cloud.tencent.com/developer/article/1843036

`buildroot` 包含了 `busybox` 的功能，并且有类似 `linux` 配置的过程 `menuconfig`，主要使用 `buildroot`，并对 `busybox` 进行简单介绍，并不涉及 yocto

##### 方式一：纯 busybox

这部分主要参考：https://zhuanlan.zhihu.com/p/258394849

> 上面这篇博客是南航一个老师写的，在Qemu上运行RISC-V 64版本的Linux，其中采用了 busybox，很友好也很详细，下面主要是作了一些解释补充

-   下载源码：

```
git clone https://github.com/mirror/busybox.git
```

-   配置 busybox：

```
# 在 busybox 的源码目录中, 配置 busybox, menuconfig 表示 会打开一个图形化窗口进行配置
# 对于 riscv 架构:
make CROSS_COMPILE=riscv64-unknown-linux-gnu- menuconfig
# 对于 arm64 架构：
make ARCH=arm64 CROSS_COMPILE=aarch64-none-linux-gnu- menuconfig
```

打开菜单后, 选择 Settings->Build Options->Build static binary(no shared libs)，即静态编译（无需依赖其他库），保存并退出

-   编译 busybox

```
# 对于 riscv 架构：
make CROSS_COMPILE=riscv64-unknown-linux-gnu- install
# 对于 arm64 架构：
make CROSS_COMPILE=aarch64-none-linux-gnu- install
```

编译完成后，会在`_install` 下包含了生成的内容，busybox生成的文件系统

-   制作最小文件系统

```
# 创建 1g (这个根据实际需要进行配置) 的 raw 格式的空镜像磁盘
qemu-img create rootfs.img  1g
# 将创建的磁盘格式化成 ext4 类型的文件系统
mkfs.ext4 rootfs.img
# 创建一个文件用来挂载该镜像（用来将编译好的内容装入磁盘，载体）
mkdir rootfs
# 将 rootfs.img 挂载到 rootfs 文件夹
sudo mount -o loop rootfs.img  rootfs

# 将在 busybox 中编译的内容移动到刚新建的文件夹 rootfs 中(等价于拷贝到了 rootfs.img 磁盘中)
cd rootfs
sudo cp -r ../busybox/_install/* .
# 补充一些必要的文件夹
sudo mkdir proc sys dev etc etc/init.d
```

> 以下内容为查找到的资料，创建这些目录为了确保 Linux 正常运行
>
> proc：用于存储内核和进程的信息
>
> sys：提供了访问和更改内核参数的接口
>
> etc：包含系统的配置文件
>
> dev：用于存放设备文件，这些文件代表系统中的物理或虚拟设备
>
> etc/init.d：用于存放初始化脚本，在启动时运行，用于启动和停止服务，下面就介绍了一个简单的脚本
>
> **【挖个坑！学会了来填坑】更多的细节还有待补充**

```
cd etc/init.d/
sudo touch rcS
sudo vi rcS
```

脚本的内容如下：

```
#!/bin/sh
mount -t proc none /proc	# 挂载文件系统到 /proc 目录, 它提供了一个接口来访问内核和进程信息
mount -t sysfs none /sys	# 挂载sysfs文件系统到 /sys 目录, 提供了一种方式来访问和调整内核的运行时信息
/sbin/mdev -s				# 使用mdev（是BusyBox的一部分）来初始化/dev目录，它负责创建设备节点
							# -s 告诉 mdev 启动是扫描 /sys 目录并创建设备节点 
							# /sys中包含了当前系统中硬件设备的详细信息
```

> proc 和 sysfs 是两个特殊的[虚拟文件系统](https://so.csdn.net/so/search?q=%E8%99%9A%E6%8B%9F%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F&spm=1001.2101.3001.7020)设备
>
> mdev 是 BusyBox 提供的一个轻量级工具，用于在系统启动时或当新硬件被检测到时，动态地创建和管理`/dev`目录中的设备节点，但不是唯一方式

给 `rcS` 文件添加可执行权限，`busybox` 的 `init` 运行起来之后，就能运行 `/etc/init.d/rcS` 脚本

```
sudo chmod +x rcS
```

退出 `rootfs` 目录并卸载文件系统（注意，如果当前目录还在 `rootfs` 里面，是卸载不成功的）

```
cd ..
# 在 rootfs 文件夹所在的目录
sudo umount rootfs
```

文件系统制作完成，为 `rootfs.img`

##### 方式二：使用 buildroot （推荐）

`buildroot` 可以编译 boot、内核、根文件系统等，这里主要采用 `buildroot` 构建根文件系统

`buildroot` 支持构建多种类型的文件系统精细那个，包括不限于

-   `cpio`：通常用于创建内核启动时所需的初始根文件系统（`initramfs`），可以直接被内核解压到 `RAM` 中，适合临时或`RAM`文件系统
-   `ext2`、`ext3`、`ext4`：基于磁盘的文件系统格式，适合于持久存储

这一部分主要参考了 https://www.cnblogs.com/sun-ye/p/14992084.html

下载源码：

```
git clone https://github.com/buildroot/buildroot.git
cd buildroot
```

使用进行配置：

```
make menuconfig 
```

1.  `Target options` -> `Target Architecture` 设置为 `AArch64(little endian)`（根据需要选择特定的架构）
2.  `Toolchain` -> `Toolchain type` 设置为 `External toochain`
3.  `Toolchain` -> `Toolchian` 设置为`Custom toolchain`
4.  `Toolchain` -> `Toolchian origin` 设置为 `Pre-installed toolchian`
5.  `Toolchain` -> `Toolchain path` 设置为 `/home/jingyu/arm-gnu-toolchain-x86_64-aarch64-none-linux-gnu`，注意：这里的路径只需要设置到 `bin` 的上一级，上面的路径根据本地实际情况配置
6.  `Toolchain` -> `Toolchain prefix` 设置为 `$(ARCH)-none-linux-gnu`，其他交叉编译链类似
7.  `Toolchain` -> `External toolchain gcc version` 设置为 `13.x` ，这个要看情况，查看交叉编译工具的 `gcc` 版本选择
8.  `Toolchain` -> `External toolchain kernal headers series` 设置为 `4.20.x`，看本地情况，如果设置错了 `buildroot` 会报错给出版本建议
9.  `Toolchain` -> `External toolchain C library` 设置为 `glibc`
10.  `Toolchain` -> `Toolchain has SPP support?` 配置为 `selected`，未配置编译报错会提示
11.  `Toolchain` -> `Toolchain has C++ support?` 配置为 `selected`，未配置编译报错会提示
12.  `Toolchain` -> `Toolchain has Fortran support?` 配置为 `selected`，未配置编译报错会提示
13.  `Toolchain` -> `Toolchain has OpenMP support?` 配置为 `selected`，未配置编译报错会提示
14.  `System configuration` -> `Enable root login with password` 配置为 `selected`，然后在这里配置根用户名以及密码
15.  `System configuration` -> `Run a getty (login prompt) after boot` -> `TTY port` 设置为 `ttyAMA0` （这个名字是针对 arm 架构的，riscv 不一定适用）
16.  `Target packages` -> `Show packages that are also provided by busybox` 配置为 `selected`
17.  `Target packages` -> `Debugging, profiling and benchmark` -> `strace` 配置为 `selected`
18.  `Filsystem images` -> `cpio the root filesystem` 配置为 `selected` ，注意这里生成的是一个 RAM 文件系统，非持久性的

如果要生成一个持久性的 ext4 格式的根文件系统，则选择如下配置：

18.  `Filsystem images` -> `ext2/3/4 root filesystem` 配置为 `selected` ，然后配置其 `filesystem label` 和 `exact size`

配置完成之后进行编译：

```
make 		# 可以使用 -j4 等参数进行加速
```

生成的 `rootfs.cpio` （或 ext4 格式的根文件系统）在目录 `$BUILDROOT_PATH/output/images` 下面

> 自此，启动 linux 的前置工作都完成了，接下来即时借助 qemu 启动 linux 系统

#### 三、启动linux

##### 1、使用纯 busybox 构建的文件系统

> 这里使用的是经过 qemu-img 创建的虚拟磁盘，是永久性的根文件系统

-   启动 RISC-V 架构的 linux

```
qemu-system-riscv64 -M virt -m 256M -nographic -kernel ../linux-6.7/arch/riscv/boot/Image -drive file=rootfs.img,format=raw,id=hd0  -device virtio-blk-device,drive=hd0 -append "root=/dev/vda rw console=ttyS0"
```

`-M` 表示的指定机器为 `virt`，不同的板子有不同的内存地址布局，所以这个参数很必要，详细的地址可以参考 qemu 源代码下的 `hw/riscv/virt.c` （arm同理）

`-m` 表示的是指定机器的内存大小

`-kelnel` 指定编译出来的 Linux 内核

`-drive file=rootfs.img,format=raw,id=hd0` 创建了一个虚拟磁盘驱动，指定镜像文件以及镜像文件格式，并给驱动分配了一个标识 id

`-device virtio-blk-device,drive=hd0` 将一个虚拟 IO 块设备添加到虚拟机中，使用标识 id 为 hd0 的驱动

`-append` 的内容是传递给内核参数的参数

这里的`/dev/vda` 为 Qemu创建的第一个Virtio磁盘，告诉内核从这个设备加载根文件系统

`rw` 表示以读写模式挂载根文件系统

`console=ttyS0` 告诉 Linux 从哪个控制台终端输出信息，`ttyS0` 即执行 qemu 命令的这个终端

> 同时按下`ctrl` + `a`，然后松开，再按下 `x` ，即可退出 `qemu` 回到原本的终端窗口

-   启动 aach64 架构的 linux

```
qemu-system-aarch64 -M virt -cpu cortex-a57 -m 256M -nographic -kernel ../linux-6.7/arch/arm64/boot/Image -drive file=rootfs.img,format=raw,id=hd0,if=none -device virtio-blk-device,drive=hd0 -append "root=/dev/vda rw console=ttyAMA0"
```

> 注意：
>
> 使用的 qemu 命令要和构建文件系统、使用的 kernel时配置 的架构一致，尤其当需要同时使用 arm 和 riscv 时
>
> 命令中的文件地址要根据实际情况进行选择
>
> 对于 qemu-system-aarch64，如果不指定 -cpu，Linux 启动不起来，终端没有任何输出（迷惑了很久发现的）

##### 2、使用 buildroot 构建的文件系统

-   cpio 格式的根文件系统（RAM上的根文件系统，临时性）

```
# cpio格式, 注意替换路径
qemu-system-aarch64 -machine virt -cpu cortex-a53 -nographic -smp 1 -m 2048 -kernel ../../../linux-6.7/arch/arm64/boot/Image --append "console=ttyAMA0" -initrd ./rootfs.cpio
```

`-initrd` （即`init ram disk`）指定的是 `initramfs`，Linux 会使用 `rootfs.cpio` 作为根文件系统，为第二部分中使用 `buildroot` 构建的临时根文件系统

-   ext4 格式的根文件系统（磁盘上的根文件系统，永久性）

```
# ext4 格式, 注意替换路径
qemu-system-aarch64 -M virt -m 4G -cpu cortex-a53 -nographic -kernel linux-6.7/arch/arm64/boot/Image -drive file=/home/jingyu/buildroot/output/images/rootfs.ext4,format=raw,if=none,id=disk0 -device virtio-blk-device,drive=disk0 -append "root=/dev/vda rw console=ttyAMA0
```

这里的`/dev/vda` 为 Qemu创建的第一个Virtio磁盘，即命令中的那一个 virtio-blk-device

`root=/dev/vda` 告诉内核从这个设备加载根文件系统

`rw` 表示以读写模式挂载根文件系统

> 这里注意：
>
> 由于 Linux 内核内置了 ext4 格式根文件系统的驱动器，所以可以直接加载 root=/dev/vda 指定的文件系统进行挂载，无需加载临时根文件系统
>
> 如果在上述命令后添加，`-initrd /home/jingyu/buildroot/output/images/rootfs.cpio` Linux 内核会先使用该临时文件系统，但制作的临时文件系统中没有切换到永久文件系统的脚本，会停留在临时文件系统上，即（rootfs.cpio）

##### 补充

可以使用 ubuntu-base 的根文件系统, 优点制作简单,支持 `apt` 包管理工具,有网络的情况下,可以下载需要的包,如有需要,可以参考我的另一篇博客:  
https://blog.csdn.net/jingyu\_1/article/details/135822574

关于配置网络的部分先挖个坑(主要是我现在也不太会)



## 参考

[Qemu 启动 Linux（aarch64 与 riscv64）_qemu-system-aarch64-CSDN博客](https://blog.csdn.net/jingyu_1/article/details/135746098?ops_request_misc=%7B%22request%5Fid%22%3A%229EFA7D6B-DB2D-42FB-8F6D-19EE591E4BC5%22%2C%22scm%22%3A%2220140713.130102334..%22%7D&request_id=9EFA7D6B-DB2D-42FB-8F6D-19EE591E4BC5&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-135746098-null-null.142^v100^pc_search_result_base5&utm_term=qemu运行Linux&spm=1018.2226.3001.4187)