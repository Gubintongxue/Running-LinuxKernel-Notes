实验 13-1：使用 cppcheck 检查代码

1．实验目的

学会使用代码缺陷静态检测工具完善代码质量。

2．实验详解

cppcheck 是一个 C/C++的代码缺陷静态检测工具。它不仅可以检测代码中的语法

错误，还可以检测出编译器检查不出来的缺陷类型，从而帮助程序员提升代码质量。

cppcheck 支持的检测功能如下。

 野指针。

 整型变量溢出。

 无效的移位操作数。

 无效的转换。

 无效使用 STL 库。

 内存泄漏检测。

 代码格式错误以及性能原因检查。

cppcheck 的安装方法如下。

$sudo apt install cppcheck

cppcheck 的使用方法如下。

cppcheck 选项

 

文件或者目录

默认情况下只显示错误信息，可以通过“--enable”命令来启动更多检查。

--enable=warning #打开警告消息

--enable=performance #打开性能消息

--enable=information #打开信息消息

--enable=all #打开所有消息