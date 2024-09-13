# 实验1-2 给优麒麟Linux更换心脏--更换内核

## 一.实验目的

（1）学会如何给Linux系统更新最新版本的Linux内核

（2）学习如何编译和安装Linux内核

## 二.实验详解

### 1.安装软件依赖包

在编译Liunx内核之前，需要通过命令安装相关软件包，终端输入以下命令

```
sudo apt update
sudo apt install -y libncurses5-dev libssl-dev libelf-dev build-essential openssl flex bison
```

![image-20240913094808194](image/image-20240913094808194.png)

**可以多安装些依赖，省的后面报错**

```
我也遇到缺少bzImage，导致make install，我的问题可能是确实依赖项，我重新多安装了些依赖项。

sudo apt-get install autoconf bison build-essential dkms dwarves fakeroot flex libelf-dev libiberty-dev libidn11 libidn11-dev libncurses-dev libpci-dev libssl-dev libudev-dev minizip openssl zlibc zstd

然后重新再 make bzImage。然后再make install
```



### 2.下载最新稳定版本Linux内核源码

内核下载官网：https://mirrors.edge.kernel.org/pub/linux/kernel/

选择稳定版本的压缩包下载到本地，一般为`linux-x.xx.xx.tar.gz` 或 `linux-x.xx.xx.tar.xz`  
我这里选择下载的是`linux-5.15.10.tar.xz

![image-20240913102432770](image/image-20240913102432770.png)

### 3.内核压缩包解压缩

进入内核压缩包所在目录，右键选择打开终端，运行以下命令：

    xz -d linux-5.15.10.tar.xz
    sudo tar -xf linux-5.15.10.tar -C /usr/src

或者

```
tar -xvJf linux-5.15.10.tar.xz -C /usr/src
```

在这个命令中：

- `-x`：解压缩文件。
- `-v`：显示解压的详细过程。
- `-J`：使用 `xz` 处理 `.xz` 压缩格式。
- `-f`：指定文件名。

所以，`tar -J` 主要是为了处理 `.xz` 压缩格式的文件

将内核文件解压缩至`/usr/src`目录下

![image-20240913102548723](image/image-20240913102548723.png)

![image-20240913102612172](image/image-20240913102612172.png)

### 4 配置内核

先进入内核文件目录

    cd /usr/src/linux-5.15.10

拷贝已有的配置文件

    sudo cp /boot/config-5.15.0-119-generic .config

![image-20240913103823302](image/image-20240913103823302.png)

注：具体配置文件名根据自己系统中的名字为准（可以通过`cd /boot` 然后`ls`查看系统已有配置文件）

![image-20240913095854755](image/image-20240913095854755.png)

使用下面命令，确定哪一个是现在使用内核配置文件

```
uname -r
```

![image-20240913095941735](image/image-20240913095941735.png)

后补：==经过第一次配置，我觉得先要修改.config文件==

然后通过make menuconfig进行配置

    sudo make menuconfig

![image-20240913104008360](image/image-20240913104008360.png)

弹出配置界面后，选择load,再选save

![image-20240913110728182](image/image-20240913110728182.png)

![image-20240913110801830](image/image-20240913110801830.png)

![image-20240913110823153](image/image-20240913110823153.png)

![image-20240913104111884](image/image-20240913104111884.png)

然后对config文件进行修改：

    sudo chmod 777 /usr/src/linux-5.15.10/.config
    vim .config

![image-20240913104248593](image/image-20240913104248593.png)

输入`/CONFIG_SYSTEM_TRUSTED`查找对应选项，然后回车定位  
输入`i`进入输入模式：  
将 CONFIG\_SYSTEM\_TRUSTED\_KEYS 选项中的 debian/canonical-certs.pem 删掉,打成分号

![image-20240913112329350](image/image-20240913112329350.png)

参考[make[1\]: *** 没有规则可制作目标“debian/canonical-certs.pem”，由“certs/x509_certificate_list” 需求。 停止。-CSDN博客](https://blog.csdn.net/Chenciyuan_nj/article/details/115099040?ops_request_misc=&request_id=&biz_id=102&utm_term=make[1]: *** 没有规则可制作目标“debian/&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-1-115099040.142^v100^pc_search_result_base5&spm=1018.2226.3001.4187)

### 5 开始编译内核

    sudo make -j4

注：-jn参数是表示使用多少个CPU核心来并行编译，具体根据自己的CPU核心数来指定

主要是根据目录下的Makefile来编

![image-20240913104849540](image/image-20240913104849540.png)

```
cat /proc/cpuinfo
```

我这里有4个，从0开始计数

![image-20240913104720045](image/image-20240913104720045.png)



全部都按得回车，编译中

![image-20240913112437999](image/image-20240913112437999.png)



编译完成后可以进入`/usr/src/linux-5.9.10/arch/x86/boot`目录查看是否有编译好的`bzImage`文件，如果有就说明编译成功。

**没找到，编译失败了，见 补充：重新编译**

### 6 安装内核模块

在/usr/src/linux-5.9.10/目录下执行：

```
make modules          #生成相应的模块
```

![image-20240913121028776](image/image-20240913121028776.png)

报错，再次修改.config

 删掉双引号里的内容，保留双引号，参考[编译内核报错——*** 没有规则可制作目标“debian/canonical-revoked-certs.pem”，由“certs/x509_revocation_list” 需求。 停止。-CSDN博客](https://blog.csdn.net/weixin_41608556/article/details/125617265?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_utm_term~default-1-125617265-blog-117435073.235^v43^control&spm=1001.2101.3001.4242.2&utm_relevant_index=4)

![image-20240913121117562](image/image-20240913121117562.png)

![image-20240913121144719](image/image-20240913121144719.png)

#### 补充：我这里再编译一次，以后直接修改.config这两处，现在我去make menuconfig..

```
make clean          # 清理编译生成的文件
make mrproper       # 彻底清理，包括配置文件
sudo cp /boot/config-5.15.0-119-generic .config
sudo chmod 777 /usr/src/linux-5.15.10/.config
vim .config#改两处
make menuconfig
make -j4
```

![image-20240913121607136](image/image-20240913121607136.png)

![image-20240913123820217](image/image-20240913123820217.png)

![image-20240913124359647](image/image-20240913124359647.png)

### 出现新问题

![image-20240913134541687](image/image-20240913134541687.png)

参考 [编译内核之BTF报错_failed to generate btf for vmlinux-CSDN博客](https://blog.csdn.net/fish332/article/details/134884806?ops_request_misc=%7B%22request%5Fid%22%3A%222001F14C-8462-421B-9618-9AE67D0B3556%22%2C%22scm%22%3A%2220140713.130102334..%22%7D&request_id=2001F14C-8462-421B-9618-9AE67D0B3556&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-2-134884806-null-null.142^v100^pc_search_result_base5&utm_term=BTF%3A .tmp_vmlinux.btf%3A pahole (pahole) is not available Failed to generate BTF for vmlinux Try to disable CONFIG_DEBUG_INFO_BTF make%3A *** [Makefile%3A1183：vmlinux] 错误 1&spm=1018.2226.3001.4187)

[内核错误：BTF: .tmp_vmlinux.btf: pahole (pahole) is not available_pahole is not available-CSDN博客](https://blog.csdn.net/qq_36393978/article/details/124274364?ops_request_misc=%7B%22request%5Fid%22%3A%222001F14C-8462-421B-9618-9AE67D0B3556%22%2C%22scm%22%3A%2220140713.130102334..%22%7D&request_id=2001F14C-8462-421B-9618-9AE67D0B3556&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-124274364-null-null.142^v100^pc_search_result_base5&utm_term=BTF%3A .tmp_vmlinux.btf%3A pahole (pahole) is not available Failed to generate BTF for vmlinux Try to disable CONFIG_DEBUG_INFO_BTF make%3A *** [Makefile%3A1183：vmlinux] 错误 1&spm=1018.2226.3001.4187)

一般是系统缺少dwarves[软件包](https://so.csdn.net/so/search?q=软件包&spm=1001.2101.3001.7020)导致，下载安装dwarves软件包即可。

```
sudo apt-get install dwarves
```

![image-20240913134809973](image/image-20240913134809973.png)

继续编译，仍然没有bzImage

可能需要使用sudo make bzImage   编译内核，内核较小时可以用make zImage，暂时忽略

```
sudo make bzImage
```



## 编译完成继续

    sudo make modules

![image-20240913135943842](image/image-20240913135943842.png)

```
sudo make modules_install
```

可以进入/lib/modules目录下查看模块安装是否成功

![image-20240913140339287](image/image-20240913140339287.png)

![image-20240913140405463](image/image-20240913140405463.png)

### 7 安装编译好的内核镜像

在/usr/src/linux-5.9.10/目录下执行：

    sudo make install

![image-20240913141427933](image/image-20240913141427933.png)

第一个问题缺少Binutils 工具 ，解决方法，但发现已经安装(参考下面的连接，可以先忽略)

```
sudo apt-get install binutils
```

![image-20240913141949653](image/image-20240913141949653.png)

那先解决 Missing file: arch/x86/boot/bzImage

```
make bzImage
```

报错

![image-20240913142348770](image/image-20240913142348770.png)

继续解决，参考[Ubuntu 20.04配置RT-PREEMPT问题解决_*** you need to run "make" before "make install".-CSDN博客](https://blog.csdn.net/weixin_51923165/article/details/124746981?ops_request_misc=&request_id=&biz_id=102&utm_term=arch/x86/Makefile:142: CONFIG_&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-6-124746981.142^v100^pc_search_result_base5&spm=1018.2226.3001.4187)

![image-20240913142329768](image/image-20240913142329768.png)

安装成功，检查

![image-20240913142549357](image/image-20240913142549357.png)

![image-20240913142620277](image/image-20240913142620277.png)

再次make install，成功

![image-20240913143107535](image/image-20240913143107535.png)



### 8 修改grub配置

打开grub配置文件

    sudo vim /etc/default/grub

把`GRUB_TIMEOUT_STYLE=hidden`给删除或者注释掉，把`GRUB_CMDLINE_LINUX_DEFAULT`的值修改为`text`。  
将`GRUB_TIMEOUT = 0`修改为`GRUB_TIMEOUT = 30`，保存退出

参考[Ubantu中 如何设置开机进入grub界面_linux_热爱高数的小牛马-GitCode 开源社区 (csdn.net)](https://gitcode.csdn.net/65ec52771a836825ed798625.html?dp_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpZCI6MTAwNDM0NSwiZXhwIjoxNzI2NzQwMjcxLCJpYXQiOjE3MjYxMzU0NzEsInVzZXJuYW1lIjoicXFfNDQ4MTc2MzgifQ.ZGZMWTM2SUbvRliuXH30_af6BezYee0_OTsyYHRz-cM&spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~activity-1-132077906-blog-130701551.235^v43^control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~activity-1-132077906-blog-130701551.235^v43^control&utm_relevant_index=2)

![image-20240913140538029](image/image-20240913140538029.png)

这样修改后，让开机时出现内核选择，设置开机进入grub界面

![image-20240913140723736](image/image-20240913140723736.png)



更新grub配置

    sudo update-grub

![image-20240913140844728](image/image-20240913140844728.png)

### 9 重新启动，选择新内核启动

![image-20240913141008614](image/image-20240913141008614.png)

##### 成功

![image-20240913143157220](image/image-20240913143157220.png)

登录试试

![image-20240913143224834](image/image-20240913143224834.png)

检查内核版本

![image-20240913143442601](image/image-20240913143442601.png)

![image-20240913143459780](image/image-20240913143459780.png)