# 最简Qemu运行Linux一条龙

进行[内核开发](https://so.csdn.net/so/search?q=%E5%86%85%E6%A0%B8%E5%BC%80%E5%8F%91&spm=1001.2101.3001.7020)的过程中，难免需要频繁重启内核，直接在物理机上反复重启非常耗时，一旦出错还难以解决。网上的Qemu配置教程要么过于繁琐，要么缺斤少两运行不通。这里，以x86为例。  
已测试版本：linux\-4.18.1

### 依赖、[qemu](https://so.csdn.net/so/search?q=qemu&spm=1001.2101.3001.7020)安装

    sudo apt install git build-essential kernel-package fakeroot libncurses5-dev libssl-dev ccache bison flex qtbase5-dev libelf-dev -y 
    sudo apt install qemu -y


### Linux内核编译

下载源码，[进入目录](https://so.csdn.net/so/search?q=%E8%BF%9B%E5%85%A5%E7%9B%AE%E5%BD%95&spm=1001.2101.3001.7020)

    make  x86_64_defconfig
    make menuconfig


设置如下两项

    General setup  --->
           ----> [*] Initial RAM filesystem and RAM disk (initramfs/initrd) support
    Device Drivers  --->
       [*] Block devices  --->
               <*>   RAM block device support
               (65536) Default RAM disk size (kbytes)


启动编译

    make -j8


编译成功后的内核位于：`arch/x86_64/boot/bzImage`

### 根文件系统制作与烧录

制作文件（确定~/qemu/busybox目录可以一键运行）

    cd ~/qemu
    cd busybox
    make menuconfig
    # 进入 Settings， 勾选 Build static binary (no shared libs)
    make install -j
    cd _install
    mkdir proc sys dev etc etc/init.d
    cd ..


写入文件镜像（确定~/qemu/busybox目录可以一键运行）

    cd ~/qemu
    qemu-img create qemu_rootfs.img  500m
    mkfs.ext4 rootfs.img
    mkdir mnt_rootfs
    sudo mount -o loop rootfs.img mnt_rootfs/
    sudo cp -r busybox/_install/* mnt_rootfs/
    sudo umount mnt_rootfs


### 启动qemu

    qemu-system-x86_64  \
        -kernel /home/selina/qemu/linux-4.18.1/arch/x86/boot/bzImage  \
        -hda /home/selina/qemu/rootfs.img \
        -append "root=/dev/sda rootfstype=ext4 rw console=ttyS0" \
        -nographic


一个脚本`start_qemu.sh`，可以指定kernel和rootfs路径

    #!/bin/bash
    
    # 默认值
    kernel=/home/selina/qemu/linux-4.18.1/arch/x86/boot/bzImage
    rootfs=/home/selina/qemu/rootfs.img


​    
    # 输出结果
    echo "using kernel : $kernel"
    echo "using rootfs: $rootfs"
    echo "other params using: $@"
    sleep 1
    qemu-system-x86_64  \
        -kernel $kernel  \
        -hda $rootfs \
        -append "root=/dev/sda rootfstype=ext4 rw console=ttyS0 nokaslr" \
        -nographic \
        $@


### GDB调试内核代码

内核配置

    Kernel hacking > Compile-time checks and compiler options ->
    	[*] Compile the kernel with debug info 


      │ Symbol: ARCH_WANT_FRAME_POINTERS [=n]   
      │ Type  : bool 
      │   Defined at lib/Kconfig.debug:352


    config ARCH_WANT_FRAME_POINTERS
    	bool
    	default y


保存，再检查

    Kernel hacking > Compile-time checks and compiler options ->
        [*] Compile the kernel with frame pointers (NEW)


重新编译

    make -j8


等待的时候，以调试模式启动qemu

    ./start_qemu.sh -S -s
    # 或者1234端口可能别人正在使用，则
    ./start_qemu.sh -S -gdb tcp::2345


新建终端，进入内核目录，等待内核编译完成后

    gdb vmlinux


进入gdb交互界面后，连接qemu，添加断点，继续执行

    target remote localhost:1234
    b start_kernel
    c


### Buglist

如果打断点不停  
根据https://blog.csdn.net/gatieme/article/details/104266966  
启动加上nokaslr就可以了

    qemu-system-x86_64 -append "xxxx nokaslr"



## 参考

[最简Qemu运行Linux一条龙_qemu linux-CSDN博客](https://blog.csdn.net/Piamen/article/details/131731040?ops_request_misc=%7B%22request%5Fid%22%3A%220D81D017-EB16-49E5-B4F4-980DF331B971%22%2C%22scm%22%3A%2220140713.130102334.pc%5Fall.%22%7D&request_id=0D81D017-EB16-49E5-B4F4-980DF331B971&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~rank_v31_ecpm-10-131731040-null-null.142^v100^pc_search_result_base5&utm_term=qemu运行Linux&spm=1018.2226.3001.4187)