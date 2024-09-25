# 实验 12-5：debugfs 

## 1．实验目的

1）写一个内核模块，在 debugfs 文件系统中创建一个名为“test”的目录。

2）在 test 目录下面创建两个节点，分别是“read”和“write”。从“read”节点中

可以读取内核模块的某个全局变量的值，向“write”节点写数据可以修改某个全局变

量的值。

## 2．实验要求

debufs文件系统中有不少API函数可以使用，它们定义在include/linux/debugfs.h头文件中。

```
struct dentry *debugfs_create_dir(const char *name,
 struct dentry *parent)
void debugfs_remove(struct dentry *dentry)
struct dentry *debugfs_create_blob(const char *name, umode_t mode,
 struct dentry *parent,
 struct debugfs_blob_wrapper *blob)
struct dentry *debugfs_create_file(const char *name, umode_t mode,
 struct dentry *parent, void *data,
 const struct file_operations *fops)
```

读者可以参照 Linux 内核中的例子来完成本实验。

## 3．实验步骤

### 下面是本实验的实验步骤。

### 启动 QEMU+runninglinuxkernel。

```
$ ./run_rlk_arm64.sh run
```



### 进入本实验的参考代码。

```
# cd /mnt/rlk_lab/rlk_basic/chapter_12_debug/lab5
```

编译内核模块。

```
benshushu:lab5_debugfs# make
make -C /lib/modules/`uname -r`/build 
M=/mnt/rlk_lab/rlk_basic/chapter_12_debug/lab5_debugfs modules;
make[1]: Entering directory '/usr/src/linux'
 CC [M] /mnt/rlk_lab/rlk_basic/chapter_12_debug/lab5_debugfs/debugfs_test.o
[ 890.252504] hrtimer: interrupt took 16687528 ns
 LD [M] /mnt/rlk_lab/rlk_basic/chapter_12_debug/lab5_debugfs/debugfs-test.o
 Building modules, stage 2.
 MODPOST 1 modules
make[2]: Warning: File 
'/mnt/rlk_lab/rlk_basic/chapter_12_debug/lab5_debugfs/debugfs-test.mod.c' has 
modification time 0.023 s in the future
 CC /mnt/rlk_lab/rlk_basic/chapter_12_debug/lab5_debugfs/debugfstest.mod.o
 LD [M] /mnt/rlk_lab/rlk_basic/chapter_12_debug/lab5_debugfs/debugfstest.ko
make[2]: warning: Clock skew detected. Your build may be incomplete.
make[1]: Leaving directory '/usr/src/linux'
```

安装内核模块。

```
benshushu:lab5_debugfs# insmod debugfs-test.ko 
[ 1003.631638] I created benshushu on debugfs
```

创建的目录会在哪里呢？debugfs 一般会 mount 到/sys/kernel/debug/这个目录。

如果大家发现这个目录不存在，或者里面没有东西，说明 debugfs 没有挂载，或

者输入“df”命令来查看。那么可以手工挂载。

```
# mount -t debugfs none /sys/kernel/debug/
```

进入到/sys/kernel/debug/目录后，你会发现有一个 benshushu 的目录。

![image-20240925155958242](image/image-20240925155958242.png)

进入 benshushu 目录之后，你发发现有一个 my_debug 的节点。

![image-20240925160005239](image/image-20240925160005239.png)

接下来就可以对这个节点进行读写操作了。

![image-20240925160010828](image/image-20240925160010828.png)

## 4．实验代码分析

```
1 #include <linux/module.h>
2 #include <linux/proc_fs.h>
3 #include <linux/uaccess.h>
4 #include <linux/init.h>
5 #include <linux/debugfs.h>
6 
7 #define NODE "benshushu"
8 
9 static int param = 100;
10struct dentry *debugfs_dir;
11
12#define KS 32
13static char kstring[KS]; /* should be less sloppy about overflows :) 
*/
14
15static ssize_t
16my_read(struct file *file, char __user *buf, size_t lbuf, loff_t *ppos)
17{
18 int nbytes = sprintf(kstring, "%d\n", param);
19 return simple_read_from_buffer(buf, lbuf, ppos, kstring, nbytes);
20}
21
22static ssize_t my_write(struct file *file, const char __user *buf, size_t 
lbuf,
23 loff_t *ppos)
24{
25 ssize_t rc;
26 rc = simple_write_to_buffer(kstring, lbuf, ppos, buf, lbuf);
27 sscanf(kstring, "%d", &param);
28 //pr_info("param has been set to %d\n", param);
29 return rc;
30}
31
32static const struct file_operations mydebugfs_ops = {
33 .owner = THIS_MODULE,
34 .read = my_read,
35 .write = my_write,
36};
37
38static int __init my_init(void)
39{
40 struct dentry *debug_file;
41
42 debugfs_dir = debugfs_create_dir(NODE, NULL);
43 if (IS_ERR(debugfs_dir)) {
44 printk("create debugfs dir fail\n");
45 return -EFAULT;
46 }
47
48 debug_file = debugfs_create_file("my_debug", 0444,
49 debugfs_dir, NULL, &mydebugfs_ops);
50 if (IS_ERR(debug_file)) {
51 printk("create debugfs file fail\n");
52 debugfs_remove_recursive(debugfs_dir);
53 return -EFAULT;
54 }
55
56 pr_info("I created %s on debugfs\n", NODE);
57 return 0;
58}
59
60static void __exit my_exit(void)
61{
62 if (debugfs_dir) {
63 debugfs_remove_recursive(debugfs_dir);
64 pr_info("Removed %s\n", NODE);
65 }
66}
67
68module_init(my_init);
69module_exit(my_exit);
70MODULE_LICENSE("GPL");
```

