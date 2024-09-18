# 实验2-5：把Vim打造成一个强大的IDE编辑工具

## 一．实验目的

通过配置把Vim打造成一个和Source Insight相媲美的IDE工具。

## 二．实验步骤

请参考《奔跑吧Linux内核入门篇》第二版第2.6.5节相关内容。

## 三．Problem

有读者在使用vim查看linuxkernel代码的时候遇到：无法补全一些数据结构的问题，可以尝试如下办法：

### 1.安装python-is-python3。YCM默认使用python3

```
$ sudo apt install python-is-python3
```

### 2.重新编译YCM。

```
$ cd /home/rlk/.vim/bundle/YouCompleteMe/
$ python3 install.py --clangd-completer
```

### 3.使 用 YCM-Generator 来为runninglinuxkernel_5.0 目录生成一个.ycm_extra_conf.py 配置文件，这个配置文件已经上传到 git 上，大家只要git pull 一下 runninglinuxkernel_5.0 即可。

如果读者想自己重新生成.ycm_extra_conf.py 文件，可以通过如下方法。

```
$sudo apt install clang exuberant-ctags
$ git clone https://github.com/rdnetto/YCM-Generator.git
$ cd YCM-Generator
$ ./config_gen.py /home/rlk/rlk/runninglinuxkernel_5.0
```

测试 YCM。打开 vim，然后打开 mm/memory.c 文件，在第 370 行，输入 vma->

![image-20240918173634260](image/image-20240918173634260.png)