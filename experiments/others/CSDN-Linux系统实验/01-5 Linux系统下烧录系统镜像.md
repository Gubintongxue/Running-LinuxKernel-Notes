# Linux系统下烧录系统镜像

## 1\. `.img` `.iso`格式镜像

通过简单的dd命令，即可将预安装镜像烧录到SD卡或U盘中

    dd if=xxxx.img of=/dev/sdX status=progress


    dd if=xxxx.iso of=/dev/sdX status=progress


其中 if= 后面的是要安装的镜像的名称（注意路径）  
of = 后面的是你的SD卡或U盘，sdX的X需要改为存储卡实际映射值，如sda，sdb等。

查看方式，可以直接使用`lsblk`命令。

在烧录之前，最好先将sd卡的挂载点卸载掉，以防有其他程序读写。比如：

    umount /dev/sda1
    umount /dev/sda2


## 2 . `.img.xz`格式镜像

如果镜像是xxx.img.xz格式，可以先解压成img格式，再使用dd烧写。

    xz -d xxx.img.xz


也可以使用如下指令直接烧写到SD卡或U盘：

    sudo xz -cd xxx.img.xz > /dev/sdX


sdX的X需要改为SD卡或U盘实际映射值，如sda，sdb等。