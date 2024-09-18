# 实验 4-1：通过 QEMU 调试 ARM64 Linux 内核

## 一．实验目的

熟悉如何使用 QEMU 调试 Linux 内核。

本实验调试 ARM64 的处理器。

## 二．实验步骤

​		首先，确保在 Linux 主机上安装了 aarch64-linux-gnu-gcc 和 QEMU 工具包。

```
$sudo apt-get install qemu qemu-system-arm gcc-aarch64-linux-gnu build-essential bison flex bc
```

​		然后，安装 gdb-multiarch 工具包。

```
$sudo apt-get install gdb-multiarch
```

![image-20240919004412484](image/image-20240919004412484.png)

​		接下来，运行 run_rlk_arm64.sh 脚本以启动 QEMU 虚拟机和 GDB 服务。

```
./run_rlk_arm64.sh run debug
```

​		上述脚本会运行如下命令，不过建议读者直接使用 run_rlk_arm64.sh 脚本。

```
$ qemu-system-aarch64 -m 1024 -cpu max,sve=on,sve256=on -M virt,gic-version=3,its=on,iommu=smmuv3 -nographic -kernel arch/arm64/boot/Image -append "noinitrd nokaslr loglevel=8 sched_debug root=/dev/vda rootfstype=ext4 rw crashkernel=256M vfio.dyndbg=+pflmt irq_gic_v3_its.dyndbg=+pflmt iommu.dyndbg=+pflmt irqdomain.dyndbg=+pflmt" -drive if=none,file=/root/runninglinuxkernel_5.0/rootfs_debian_arm64.ext4,id=hd0 -device virtio-blk-device,drive=hd0 --fsdev local,id=kmod_dev,path=./kmodules,security_model=none -device virtio-9p-pci,fsdev=kmod_dev,mount_tag=kmod_mount -s -S
```

- -S：表示 QEMU 虚拟机会冻结 CPU，直到远程的 GDB 输入相应的控制命

令。

- -s：表示在 1234 端口接收 GDB 的调试连接。



终端好像是暂停了

![image-20240919004612602](image/image-20240919004612602.png)



接下来，在另外一个超级终端中启动 GDB。

```
$ cd runninglinuxkernel_5.0
$ gdb-multiarch --tui vmlinux
(gdb) set architecture aarch64 <= 设置Aarch64架构
(gdb) target remote localhost:1234 <= 通过1234端口远程连接到QEMU虚拟机
(gdb) b start_kernel <= 在内核的start_kernel处设置断点
(gdb) c
```

如图 4.5 所示，GDB 开始接管 Linux 内核的运行，并且在断点处暂停，这时即可使用 GDB 命令来调试内核。

![image-20240919003629298](image/image-20240919003629298.png)

实操：

```
cd runninglinuxkernel_5.0
gdb-multiarch --tui vmlinux
```

![image-20240919004730494](image/image-20240919004730494.png)

```
(gdb) set architecture aarch64
(gdb) target remote localhost:1234
```

![image-20240919004953210](image/image-20240919004953210.png)

```
b start_kernel
```

![image-20240919005019784](image/image-20240919005019784.png)

```
c
```

![image-20240919005051459](image/image-20240919005051459.png)

再按n

![image-20240919005208417](image/image-20240919005208417.png)

离开gdb

![image-20240919005227709](image/image-20240919005227709.png)

另一个终端开始运行

![image-20240919005249455](image/image-20240919005249455.png)

![image-20240919005432412](image/image-20240919005432412.png)