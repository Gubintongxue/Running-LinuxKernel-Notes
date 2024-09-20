# version1-内核状态下参数传递-module_param

1.简介

在用户态下编程可以通过main()的参数来传递[命令行](https://so.csdn.net/so/search?q=%E5%91%BD%E4%BB%A4%E8%A1%8C&spm=1001.2101.3001.7020)参数，而编写一个内核模块则通过module\_param()来传递参数。

在用户态下编程可以通过main(intargc,char\*argv\[\])的参数来传递命令行参数，而编写一个内核模块则通过module\_param()来传递参数。

例如，应用程序命令行传参:

    int main(intargc,char*argv[])/*argc:命令行参数个数，argv:命令行参数信息*/
    {
    /* 函数体 */
         return0;
    }
    运行：./a.out100200
    其中：argc=3
    argv[0]=“./a.out”
    argv[1]=“100”
    argv[2]=“200”

  


module\_param(name, type, perm)是一个宏，表示向当前模块传入参数。参数用 module\_param 宏定义来声明, 它定义在 moduleparam.h中。

这个 [宏定义](http://baike.baidu.com/view/2076445.htm)应当放在任何函数之外, 典型地是出现在 [源文件](http://baike.baidu.com/view/385166.htm)的前面.定义如：

    MODULE_LICENSE("Dual BSD/GPL");
     
    static char *whom = "world";
    static int howmany = 1;
     
    module_param(howmany,int,S_IRUGO);
    module_param(whom,charp,S_IRUGO);

2.内核参数

module\_param(name,type,perm);

功能：指定模块参数，用于在加载模块时或者模块加载以后传递参数给模块。

参数：

name：模块参数的名称

type： 模块参数的数据类型

perm： 模块参数的访问权限

其中参数type可以取以下任意一种情况：

bool : 布尔型

inbool : 布尔反值

charp: 字符指针（相当于char \*,不超过1024字节的字符串）

short: 短整型

ushort : 无符号短整型

int : 整型

uint : 无符号整型

long : 长整型

ulong: 无符号长整型

参数perm表示此参数在sysfs文件系统中所对应的文件节点的属性，其权限在include/linux/stat.h中有定义。它的取值可以用宏定义，也可以有数字法表示。

**宏定义**有：

#defineS\_IRUSR 00400文件所有者可读

#defineS\_IWUSR00200文件所有者可写

#defineS\_IXUSR 00100文件所有者可执行

#defineS\_IRGRP00040与文件所有者同组的用户可读

#defineS\_IWGRP00020

#defineS\_IXGRP 00010

#defineS\_IROTH 00004与文件所有者不同组的用户可读

#defineS\_IWOTH00002

#defineS\_IXOTH 00001

将数字最后三位转化为二进制:xxx xxx xxx,高位往低位依次看,第一位为1表示文件所有者可读,第二位为1表示文件所有者可写,第三位为1表示文件所有者可执行;接下来三位表示文件所有者同组成员的权限;再下来三位为不同组用户权限.

00400 ==> 400 ==> 100 000 000

**数字法**：1表示执行权限，2表示写入权限，4表示读取权限。

一般用8进制表示即可，如0664。从左向右看，第一位的0表示八进制的意思，第二位的6表示文件所有者的权限为可读可写，第三位的6表示文件同组用户的权限为可读可写，第四位的4表示文件其他用户的权限为只读。

例如：

intirq;

char\*pstr;

module\_param(irq,int,0664);

module\_param(pstr,charp,0000);

3.测试代码

    /*************************************************************************
    	> File Name: hello.c
    	> Author:  
    	> Mail: 
    	> Created Time: 2015年03月05日 星期四 22时17分43秒
     ************************************************************************/
     
    #include <linux/init.h>
    #include <linux/module.h>
    #include <linux/moduleparam.h>
     
     
    MODULE_LICENSE("Dual BSD/GPL");
     
    static char *whom = "world";
    static int howmany = 1;
     
    module_param(howmany,int,S_IRUGO);
    module_param(whom,charp,S_IRUGO);
     
    static int hello_init(void)
    {
        int i;
        for(i = 0; i < howmany;++i)
            printk(KERN_ALERT"Hello,%s\n",whom);
        return 0;
    }
     
    static void hello_exit(void)
    {
        printk(KERN_ALERT"Goodbye,cruel %s\n",whom);
    }
     
    module_init(hello_init);
    module_exit(hello_exit);

测试结果：

    sudo insmod hellop.ko howmany=10 whom="Mom"
    dmesg
    ---------------------------------------------------------------------
    [ 3844.046504] Hello,Mom
    [ 3844.046513] Hello,Mom
    [ 3844.046516] Hello,Mom
    [ 3844.046518] Hello,Mom
    [ 3844.046520] Hello,Mom
    [ 3844.046522] Hello,Mom
    [ 3844.046524] Hello,Mom
    [ 3844.046526] Hello,Mom
    [ 3844.046528] Hello,Mom
    [ 3844.046530] Hello,Mom
    ----------------------------------------------------------------------
    sudo rmmod hellop
    demsg
    ----------------------------------------------------------------------
    [ 3886.029760] Goodbye,cruel Mom
    ----------------------------------------------------------------------
     

代码下载：

[http://git.oschina.net/OpenWrt-X/ldd3/tree/master/driver/2.hellop](http://git.oschina.net/OpenWrt-X/ldd3/tree/master/driver/2.hellop)

## 参考

[内核状态下参数传递-module_param_linux用户态怎么向mod传参数-CSDN博客](https://blog.csdn.net/dai_xiangjun/article/details/44103827?ops_request_misc=&request_id=&biz_id=102&utm_term=向内核模块传递参数&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-4-44103827.nonecase&spm=1018.2226.3001.4187)

# version2-从命令行传递参数给内核模块 (MODULE_PARM)

[http://blog.chinaunix.net/u1/54524/showart\_470321.html](http://blog.chinaunix.net/u1/54524/showart_470321.html)

模块也可以从命令行获取参数。但不是通过以前你习惯的argc/argv。

要传递参数给模块，首先将获取参数值的变量声明为全局变量。然后使用宏`MODULE_PARM()`(在头文件linux/module.h)。运行时，insmod将给变量赋予命令行的参数，如同 **./insmod mymodule.o myvariable=5**。为使代码清晰，变量的声明和宏都应该放在 模块代码的开始部分。以下的代码范例也许将比我公认差劲的解说更好。

宏`MODULE_PARM()`需要两个参数，变量的名字和其类型。支持的类型有" b": 比特型，"h": 短整型, "i": 整数型，" l: 长整型和 "s": 字符串型,其中正数型既可为signed也可为unsigned。 字符串类型应该声明为"char \*"这样insmod就可以为它们分配内存空间。你应该总是为你的变量赋初值。 这是内核编程，代码要编写的十分谨慎。举个例子：

```
int myint = 3;
char *mystr;

MODULE_PARM(myint, "i");
MODULE_PARM(mystr, "s");
```

​	

数组同样被支持。在宏MODULE\_PARM中在类型符号前面的整型值意味着一个指定了最大长度的数组。 用'-'隔开的两个数字则分别意味着最小和最大长度。下面的例子中，就声明了一个最小长度为2,最大长度为4的整形数组。

```
int myshortArray[4];
MODULE_PARM (myintArray, "3-9i");
```

将初始值设为缺省使用的IO端口或IO内存是一个不错的作法。如果这些变量有缺省值，则可以进行自动设备检测， 否则保持当前设置的值。我们将在后续章节解释清楚相关内容。在这里我只是演示如何向一个模块传递参数。

最后，还有这样一个宏，`MODULE_PARM_DESC()`被用来注解该模块可以接收的参数。该宏 两个参数：变量名和一个格式自由的对该变量的描述。

**Example 2-7. hello-5.c**

```C
/*
 *  hello-5.c - Demonstrates command line argument passing to a module.
 */
#include <linux/module.h>
#include <linux/moduleparam.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/stat.h>

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Peter Jay Salzman");

static short int myshort = 1;
static int myint = 420;
static long int mylong = 9999;
static char *mystring = "blah";

/* 
 * module_param(foo, int, 0000)
 * The first param is the parameters name
 * The second param is it's data type
 * The final argument is the permissions bits, 
 * for exposing parameters in sysfs (if non-zero) at a later stage.
 */

module_param(myshort, short, S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP);
MODULE_PARM_DESC(myshort, "A short integer");
module_param(myint, int, S_IRUSR | S_IWUSR | S_IRGRP | S_IROTH);
MODULE_PARM_DESC(myint, "An integer");
module_param(mylong, long, S_IRUSR);
MODULE_PARM_DESC(mylong, "A long integer");
module_param(mystring, charp, 0000);
MODULE_PARM_DESC(mystring, "A character string");

static int __init hello_5_init(void)
{
	printk(KERN_ALERT "Hello, world 5/n=============/n");
	printk(KERN_ALERT "myshort is a short integer: %hd/n", myshort);
	printk(KERN_ALERT "myint is an integer: %d/n", myint);
	printk(KERN_ALERT "mylong is a long integer: %ld/n", mylong);
	printk(KERN_ALERT "mystring is a string: %s/n", mystring);
	return 0;
}

static void __exit hello_5_exit(void)
{
	printk(KERN_ALERT "Goodbye, world 5/n");
}

module_init(hello_5_init);
module_exit(hello_5_exit);
```

我建议用下面的方法实验你的模块：

```shell
satan# insmod hello-5.o mystring="bebop" mybyte=255 myintArray=-1
mybyte is an 8 bit integer: 255
myshort is a short integer: 1
myint is an integer: 20
mylong is a long integer: 9999
mystring is a string: bebop
myintArray is -1 and 420

satan# rmmod hello-5
Goodbye, world 5

satan# insmod hello-5.o mystring="supercalifragilisticexpialidocious" /
> mybyte=256 myintArray=-1,-1
mybyte is an 8 bit integer: 0
myshort is a short integer: 1
myint is an integer: 20
mylong is a long integer: 9999
mystring is a string: supercalifragilisticexpialidocious
myintArray is -1 and -1

satan# rmmod hello-5
Goodbye, world 5

satan# insmod hello-5.o mylong=hello
hello-5.o: invalid argument syntax for mylong: 'h'
```

module\_param(name, type, perm)是一个宏，向当前模块传入参数,对源码分析如下  

```C
在include/linux/moduleparam.h中

#define module_param(name, type, perm)                /
    module_param_named(name, name, type, perm)

#define module_param_named(name, value, type, perm)               /
    param_check_##type(name, &(value));                   /
    module_param_call(name, param_set_##type, param_get_##type, &value, perm); /
    __MODULE_PARM_TYPE(name, #type)



#define module_param_call(name, set, get, arg, perm)                  /
    __module_param_call(MODULE_PARAM_PREFIX, name, set, get, arg, perm)


#define __module_param_call(prefix, name, set, get, arg, perm)        /
    /* Default value instead of permissions? */            /
    static int __param_perm_check_##name __attribute__((unused)) =    /
    BUILD_BUG_ON_ZERO((perm) < 0 || (perm) > 0777 || ((perm) & 2));    /
    static char __param_str_##name[] = prefix #name;        /
    static struct kernel_param const __param_##name            /
    __attribute_used__                        /
    __attribute__ ((unused,__section__ ("__param"),aligned(sizeof(void *)))) /
    = { __param_str_##name, perm, set, get, arg }

```

  

\_\_attibute\_\_ 是gcc的关键字，可参考  
[http://gcc.gnu.org/onlinedocs/gcc-4.0.0/gcc/Variable-Attributes.html](http://gcc.gnu.org/onlinedocs/gcc-4.0.0/gcc/Variable-Attributes.html)  
\_\_attibute\_\_将参数编译到\_\_param段中，  

module\_param是一步一步展开的,  
上面几个宏在include/linux/moduleparam.h中的顺序刚好相反  

module\_param宏的类似函数调用的顺序  
module\_param->module\_param\_named->module\_param\_call->\_\_module\_param\_call  

展开的顺序正好相反  
\_\_module\_param\_call->module\_param\_call->module\_param\_named->module\_param  

  

type类型可以是byte,short,ushort,int,  
uint,long,ulong,charp(注：字符指针),bool,invbool,   
perm表示此参数在sysfs文件系统中所对应的文件节点的属性。  
权限在include/linux/stat.h中有定义  
比如：  

```
#define S_IRWXU 00700
#define S_IRUSR 00400
#define S_IWUSR 00200
#define S_IXUSR 00100

#define S_IRWXG 00070
#define S_IRGRP 00040
#define S_IWGRP 00020
#define S_IXGRP 00010

#define S_IRWXO 00007
#define S_IROTH 00004
#define S_IWOTH 00002
#define S_IXOTH 00001
```

当perm为0时，表示此参数不存在 sysfs文件系统下对应的文件节点。  
模块被加载后，在/sys/module/ 目录下将出现以此模块名命名的目录。  

测试一下，插入一个驱动模块mp.ko,  
sudo insmod mp.ko   
发现 /sys/module下出现mp这个文件夹  
mp中有drivers/,sections/,parameters/,initstate,refcnt,srcversion  

其中initstate 里面内容为：live  
refcnt里面内容为：0  
srcversion里面内容为：F9BDEBF706329B443C28E08,很长，应该是一个唯一数字  
driver文件夹里面是空的  
sections有\_\_param,\_\_versions  
\_\_param里面内容为0xd081e0a8  
\_\_versions里面内容为0xd081e100  

parameters有count,info(注：自己定义的参数)  
count里面的内容为1(注：输出次数)  
info里面的内容为a site about linux driver.(注：输出内容)  

如果此模块存在perm不为0的命令行参数，  
在此模块的目录下将出现parameters目录，  
包含一系列以参数名命名的文件节点，  
这些文件的权限值等于perm，文件的内容为参数的值。  


在/sys/module/mp/出现的参数，在module结构中的对应如下  


    struct module
    {
        enum module_state state;
    
        /* Member of list of modules */
        struct list_head list;
    
        ......
    
        /* Sysfs stuff. */
        struct module_kobject mkobj;
        struct module_param_attrs *param_attrs;
        struct module_attribute *modinfo_attrs;
        const char *version;
        const char *srcversion;
        struct kobject *drivers_dir;
    
        /* Exported symbols */
        const struct kernel_symbol *syms;
        unsigned int num_syms;
        const unsigned long *crcs;
    
        ......
    
        /* Section attributes */
        struct module_sect_attrs *sect_attrs;
    #endif
    
        ......
    };
    
    
    enum module_state
    {
        MODULE_STATE_LIVE,
        MODULE_STATE_COMING,
        MODULE_STATE_GOING,
    };



state对应/sys/module/mp/initstate  
drivers\_dir对应/sys/module/mp/driver文件夹  
version对应/sys/module/mp/sections/\_\_version  
srcversion对应/sys/module/mp/srcversion  


测试模块，源程序mp.c内容如下：  

```C
#include <linux/module.h>
#include <linux/moduleparam.h>             
 
static char *info= "a site about linux driver.";           
static int count= 1;    
static int mp= 0;       
    
module_param(mp,int,0);

module_param(count,int,S_IRUSR);     

module_param(info,charp,S_IRUSR);   
 
MODULE_AUTHOR("ioctrl");
MODULE_LICENSE("Dual BSD/GPL");
 
static int hello_init(void)       
{
    int i;
    for(i=0;i<count;i++)
    {
       printk(KERN_NOTICEwww.linuxdriver.cn is ");
       printk(info);
        printk("/n");
      }
   return 0;
 }
 
static void hello_exit(void) 
{
    printk(KERN_NOTICE"exit!/n");
}

module_init(hello_init);
module_exit(hello_exit);
```

编译  

手工插入,插入时传递参数 count=10,info="驱动开发网站。"  
命令如下：  
sudo insmod mp.ko count=10 info="驱动开发网站。"  
然后用dmesg查看消息  
看到如下，共10条记录  
\[16122.280000\][www.linuxdriver.cn](http://www.linuxdriver.cn/) is 驱动开发网站  
\[16122.280000\][www.linuxdriver.cn](http://www.linuxdriver.cn/) is 驱动开发网站  
\[16122.280000\][www.linuxdriver.cn](http://www.linuxdriver.cn/) is 驱动开发网站  
\[16122.280000\][www.linuxdriver.cn](http://www.linuxdriver.cn/) is 驱动开发网站  
\[16122.280000\][www.linuxdriver.cn](http://www.linuxdriver.cn/) is 驱动开发网站  
\[16122.280000\][www.linuxdriver.cn](http://www.linuxdriver.cn/) is 驱动开发网站  
\[16122.280000\][www.linuxdriver.cn](http://www.linuxdriver.cn/) is 驱动开发网站  
\[16122.280000\][www.linuxdriver.cn](http://www.linuxdriver.cn/) is 驱动开发网站  
\[16122.280000\][www.linuxdriver.cn](http://www.linuxdriver.cn/) is 驱动开发网站  
\[16122.280000\][www.linuxdriver.cn](http://www.linuxdriver.cn/) is 驱动开发网站