# 实验 9-6：mmap

## 1．实验目的

理解 mmap 系统调用的使用方法以及实现原理。

## 2．实验要求

1）编写一个简单的字符设备程序。分配一段物理内存，然后使用 mmap 方法把

这段物理内存映射到进程地址空间中，用户进程打开这个驱动程序之后就可以读写这

段物理内存了。需要实现 mmap、read 和 write 方法。

2）写一个简单的用户空间的测试程序，来测试这个字符设备驱动，比如测试 open、

mmap、read 和 write 方法。

## 3．实验步骤

下面是本实验的实验步骤。

启动 QEMU+runninglinuxkernel。

```
$ ./run_rlk_arm64.sh run
```

进入本实验的参考代码。

```
# cd /mnt/rlk_lab/rlk_basic/chapter_9_mm/lab6
```

编译内核模块。

```
benshushu:lab6_mmap# make
make -C /lib/modules/`uname -r`/build 
M=/mnt/rlk_lab/rlk_basic/chapter_9_mm/lab6_mmap modules;
make[1]: Entering directory '/usr/src/linux'
 CC [M] /mnt/rlk_lab/rlk_basic/chapter_9_mm/lab6_mmap/mydev_mmap.o
 LD [M] /mnt/rlk_lab/rlk_basic/chapter_9_mm/lab6_mmap/mydevdemo_mmap.o
 Building modules, stage 2.
 MODPOST 1 modules
make[2]: Warning: File 
'/mnt/rlk_lab/rlk_basic/chapter_9_mm/lab6_mmap/mydevdemo_mmap.mod.c' has 
modification time 0.13 s in the future
 CC /mnt/rlk_lab/rlk_basic/chapter_9_mm/lab6_mmap/mydevdemo_mmap.mod.o
 LD [M] /mnt/rlk_lab/rlk_basic/chapter_9_mm/lab6_mmap/mydevdemo_mmap.ko
make[2]: warning: Clock skew detected. Your build may be incomplete.
make[1]: Leaving directory '/usr/src/linux'
```

安装内核模块。

```
benshushu:lab6_mmap# insmod mydevdemo_mmap.ko 
[ 1191.122203] succeeded register char device: mydemo_mmap_dev
benshushu:lab6_mmap#
```

编译和运行测试程序。

```
benshushu:lab6_mmap# gcc test.c -o test
benshushu:lab6_mmap# ./test 
[ 1237.395078] demodrv_open: major=10, minor=58
driver max buffer size=40960
[ 1237.402272] demodrv_mmap: mapping 40960 bytes of device buffer at offset 0
mmap driver buffer succeeded: 0xffffbd688000
[ 1237.419639] demodrv_read: read nbytes=40960 done at pos=40960
data modify and compare succussful
benshushu:lab6_mmap#
```

![image-20240924151826910](image/image-20240924151826910.png)

## 4．参考代码

驱动程序的参考代码如下。

```
1 #include <linux/module.h>
2 #include <linux/fs.h>
3 #include <linux/uaccess.h>
4 #include <linux/init.h>
5 #include <linux/miscdevice.h>
6 #include <linux/device.h>
7 #include <linux/slab.h>
8 #include <linux/kfifo.h>
9 
10 #define DEMO_NAME "my_demo_dev"
11 static struct device *mydemodrv_device;
12 
13 /*virtual FIFO device's buffer*/
14 static char *device_buffer;
15 #define MAX_DEVICE_BUFFER_SIZE (10 * PAGE_SIZE)
16 
17 #define MYDEV_CMD_GET_BUFSIZE 1 /* defines our IOCTL cmd */
18 
19 static int demodrv_open(struct inode *inode, struct file *file)
20 {
21 int major = MAJOR(inode->i_rdev);
22 int minor = MINOR(inode->i_rdev);
23 
24 printk("%s: major=%d, minor=%d\n", __func__, major, minor);
25 
26 return 0;
27 }
28 
29 static int demodrv_release(struct inode *inode, struct file *file)
30 {
31 return 0;
32 }
33 
34 static ssize_t
35 demodrv_read(struct file *file, char __user *buf, size_t count, loff_t 
*ppos)
36 {
37 int nbytes =
38 simple_read_from_buffer(buf, count, ppos, device_buffer, 
MAX_DEVICE_BUFFER_SIZ E);
39 
40 printk("%s: read nbytes=%d done at pos=%d\n",
41 __func__, nbytes, (int)*ppos);
42 
43 return nbytes;
44 }
45 
46 static ssize_t
47 demodrv_write(struct file *file, const char __user *buf, size_t count, 
loff_t *ppos)
48 {
49 int nbytes =
50 simple_write_to_buffer(device_buffer, 
MAX_DEVICE_BUFFER_SIZE, ppos, buf, count );
51 
52 printk("%s: write nbytes=%d done at pos=%d\n",
53 __func__, nbytes, (int)*ppos);
54 
55 return nbytes;
56 }
57 
58 static int
59 demodrv_mmap(struct file *filp, struct vm_area_struct *vma)
60 {
61 unsigned long pfn;
62 unsigned long offset = vma->vm_pgoff << PAGE_SHIFT;
63 unsigned long len = vma->vm_end - vma->vm_start;
64 
65 if (offset >= MAX_DEVICE_BUFFER_SIZE)
66 return -EINVAL;
67 if (len > (MAX_DEVICE_BUFFER_SIZE - offset))
68 return -EINVAL;
69 
70 printk("%s: mapping %ld bytes of device buffer at offset %ld\n",
71 __func__, len, offset);
72 
73 /* pfn = page_to_pfn (virt_to_page (ramdisk + offset)); */
74 pfn = virt_to_phys(device_buffer + offset) >> PAGE_SHIFT;
75 vma->vm_page_prot = pgprot_noncached(vma->vm_page_prot);
76 if (remap_pfn_range(vma, vma->vm_start, pfn, len, 
vma->vm_page_prot))
77 return -EAGAIN;
78 
79 return 0;
80 }
81 
82 static long
83 demodrv_unlocked_ioctl(struct file *filp, unsigned int cmd, unsigned long 
arg)
84 {
85 unsigned long tbs = MAX_DEVICE_BUFFER_SIZE;
86 void __user *ioargp = (void __user *)arg;
87 
88 switch (cmd) {
89 default:
90 return -EINVAL;
91 
92 case MYDEV_CMD_GET_BUFSIZE:
93 if (copy_to_user(ioargp, &tbs, sizeof(tbs)))
94 return -EFAULT;
95 return 0;
96 }
97 }
98 
99 static const struct file_operations demodrv_fops = {
100 .owner = THIS_MODULE,
101 .open = demodrv_open,
102 .release = demodrv_release,
103 .read = demodrv_read,
104 .write = demodrv_write,
105 .mmap = demodrv_mmap,
106 .unlocked_ioctl = demodrv_unlocked_ioctl,
107};
108
109static struct miscdevice mydemodrv_misc_device = {
110 .minor = MISC_DYNAMIC_MINOR,
111 .name = DEMO_NAME,
112 .fops = &demodrv_fops,
113};
114
115static int __init simple_char_init(void)
116{
117 int ret;
118
119 device_buffer = kmalloc(MAX_DEVICE_BUFFER_SIZE, GFP_KERNEL);
120 if (!device_buffer)
121 return -ENOMEM;
122
123 ret = misc_register(&mydemodrv_misc_device);
124 if (ret) {
125 printk("failed register misc device\n");
126 kfree(device_buffer);
127 return ret;
128 }
129
130 mydemodrv_device = mydemodrv_misc_device.this_device;
131
132 printk("succeeded register char device: %s\n", DEMO_NAME);
133
134 return 0;
135}
136
137static void __exit simple_char_exit(void)
138{
139 printk("removing device\n");
140
141 kfree(device_buffer);
142 misc_deregister(&mydemodrv_misc_device);
143}
144
145module_init(simple_char_init);
146module_exit(simple_char_exit);
147
148MODULE_AUTHOR("Benshushu");
149MODULE_LICENSE("GPL v2");
150MODULE_DESCRIPTION("simpe character device");
```

​		本实验是基于第 5 章的实验代码修改过来的，接下来我们只看和 mmap 相关的部分。

​		第 99 行，每个字符设备驱动都需要实现一个设备文件操作方法集 struct file_operations。我们这个实验也不例外。本实验需要实现 mmap 方法集，因此在第 105行，添加了读 mmap 方法的实现。 

```
.mmap = demodrv_mmap,
```

​		实现 mmap 方法的函数是 demodrv_mmap，实现在第 59 行。

​		第 59 行。demodrv_mmap()函数有两个参数，一个是 filp，设备文件操作符，另外一个是 vma，表示要映射的用户空间的区间。vma 这个概念在本章实验 5 中已经有说明了。

​		**读者可能会问，第一个参数** **filp** **好理解，那第二个参数** **vma** **是从哪里来的呢？**

​		要弄明白这个问题，需要读懂 Linux 内核的缺页异常和 mmap 机制的实现。有精

力的读者可以阅读蓝色版本《奔跑吧 Linux 内核》一书，第 2 章相关内容。

​		第 62 行，vma->vm_pgoff 表示在 VMA 区域中的偏移，通常这个值为 0。

​		第 63 行，len 表示 VMA 区域的长度。

​		第 65~68 行，做一些必要的检查。

​		第 74 行，本实验中 FIFO 设备的内存是 device_buffer，它在 init 时候通过 kmalloc来分配的。kmalloc 分配的内存是物理内存，并且是线性映射的内存，而且是物理上连续的内存。那么我们可以通过 virt_to_phys()来查找到该物理内存的起始的页幁号。

​		**读者需要注意，若 device_buffer 的大小不是以页对齐的话，那么 mmap 做映射的时候需要考虑起始地址对齐的问题，以及 buffer 大小和页对齐的问题。本实验设置的 buffer 大小是 10 \* PAGE_SIZE，而在实际开发过程中，需要考虑对齐的问题。读者需要注意，若 device_buffer 的大小不是以页对齐的话，那么 mmap 做映射的时候需要考虑起始地址对齐的问题，以及 buffer 大小和页对齐的问题。本实验设置的 buffer 大小是 10 \* PAGE_SIZE，而在实际开发过程中，需要考虑对齐的问题。**

​		第 75 行，pgprot_noncached()函数用来关闭 cache。cache 的属性设置是在 PTE 页表中，该函数根据体系结构不的同，来设置硬件 PTE 页表中关于 cache 的相关属性。在 ARM32 和 ARM64 中，它们的 PTE 页表就不相同，该函数的实现也有差异。

​		第 76 行，调用 remap_pfn_range()函数来完成用户空间虚拟内存和物理内存的映射关系。

```
int remap_pfn_range(struct vm_area_struct *vma, unsigned long addr,
 unsigned long pfn, unsigned long size, pgprot_t prot)
```

remap_pfn_range()函数实现在 mm/memory.c 文件中，它的主要功能是把内核态的

物理内存映射到用户空间。它一共有 5 个参数。

-  vma：描述用户空间的虚拟内存

-  addr：要映射的用户空间虚拟内存的起始地址，这个地址必须在 vma 区域里。

-  pfn：物理内存的起始页幁号

-  size：映射的大小

-  prot：映射的属性

测试程序的参考代码如下。

```
1 #include <stdio.h>
2 #include <fcntl.h>
3 #include <unistd.h>
4 #include <sys/mman.h>
5 #include <string.h>
6 #include <errno.h>
7 #include <fcntl.h>
8 #include <sys/ioctl.h>
9 #include <malloc.h>
10
11#define DEMO_DEV_NAME "/dev/my_demo_dev"
12
13#define MYDEV_CMD_GET_BUFSIZE 1 /* defines our IOCTL cmd */
14
15int main()
16{
17 int fd;
18 int i;
19 size_t len;
20 char message[] = "Testing the virtual FIFO device";
21 char *read_buffer, *mmap_buffer;
22
23 len = sizeof(message);
24
25 fd = open(DEMO_DEV_NAME, O_RDWR);
26 if (fd < 0) {
27 printf("open device %s failded\n", DEMO_DEV_NAME);
28 return -1;
29 }
30
31 if (ioctl(fd, MYDEV_CMD_GET_BUFSIZE, &len) < 0) {
32 printf("ioctl fail\n");
33 goto open_fail;
34 }
35
36 printf("driver max buffer size=%d\n", len);
37
38 read_buffer = malloc(len);
39 if (!read_buffer)
40 goto open_fail;
41
42 mmap_buffer = mmap(NULL, len, PROT_READ | PROT_WRITE, MAP_SHARED, 
fd, 0);
43 if (mmap_buffer == (char *)MAP_FAILED) {
44 printf("mmap driver buffer fail\n");
45 goto map_fail;
46 }
47
48 printf("mmap driver buffer succeeded: %p\n", mmap_buffer);
49
50 /* modify the mmaped buffer */
51 for (i = 0; i < len; i++)
52 *(mmap_buffer + i) = (char)random();
53
54 /* read the buffer back and compare with the mmap buffer*/
55 if (read(fd, read_buffer, len) != len) {
56 printf("read fail\n");
57 goto read_fail;
58 }
59
60 if (memcmp(read_buffer, mmap_buffer, len)) {
61 printf("buffer compare fail\n");
62 goto read_fail;
63 }
64
65 printf("data modify and compare succussful\n");
66
67 munmap(mmap_buffer, len);
68 free(read_buffer);
69 close(fd);
70
71 return 0;
72
73read_fail:
74 munmap(mmap_buffer, len);
75map_fail:
76 free(read_buffer);
77open_fail:
78 close(fd);
79 return -1;
80
81}
```

​		第 25 行，打开设备文件。

​		第 31 行，获取 buffer 的大小。

​		第 38 行，分配一个读 buffer。

​		第 42 行，通过 mmap 函数来把设备驱动的 buffer 映射到用户空间 mmap_buffer。

​		第 51~52 行，修改设备驱动 buffer 的内容。

​		第 55 行，通过 read 方法，把设备 buffer 读到用户空间的读 buffer 中。

​		第 60 行，比较读 buffer 和 mmap_buffer 的数据是否完全相同。

## 5．进阶思考

本实验在内核空间申请一个 buffer，然后把这个 buffer 映射到用户空间中。那么用户空间就可以读写这个内核空间申请的 buffer 的数据了。

读者可以深入思考两个问题：

1. 先来考察内核空间申请的 buffer。在本实验中使用 kmalloc()函数来分配内存。

-  站在 CPU 角度来看，CPU 访问这个 buffer，这个 buffer 中的页（page）

对应的物理内存和对应的虚拟内存分别指向哪里？它对应的 PTE 页表

又在哪里？

-  它的 cache 是打开还是关闭的？

-  若这时候有一个硬件设备也需要来访问这个 buffer，比如硬件设备想通

过 DMA 来访问我们这个 buffer，我们是否考虑 cache 的问题？CPU 和

DMA 同时访问这个 buffer，怎么办？如何关闭这个 buffer 的 cache。

-  内核中有哪些接口函数可以分配关闭 cache 的 buffer？

2. 我们来考察通过 mmap 映射到用户空间的这个 user buffer。

-  这个 user buffer 的物理内存是在哪里？对应的虚拟内存是在哪里？

-  对应的 PTE 页表项是在哪里？

-  CPU 访问这个 user buffer，它对应的 cache 是关闭还是打开的？