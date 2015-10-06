[TOC]
# Scull 学习笔记
## Scull 文件结构
|	文件					|	描述			|
| :-----------------:	| :-----------: 	|
|	main.c				|	主程序，scull设备的初始化、卸载、open、 write等实现都在这里面	|	
|	Makefile			|					|
|	pipe.c				|	第六章《高级字符驱动程序操作》会用到，用来讲解阻塞型设 备	|
|	scull.h				|	头文件		|
|	scull.init			|	加载scull模块的脚本	|
|	scull_unload	|	卸载scull模块的脚本	|

## 编译
- make : 编译,最终生成scull.ko
- 安装: 脚本scull_load， scull设备就会自动加载到内核. 此时可以通过查看/proc/devices文件找到scull模块，还有内核为其分配的主设备号,同时在/dev/中也增加很多以scull开头的字符设备.
- 卸载: 脚本scull_unload, 清除/proc/devices 以及/dev/的scull设备
- make clean : 清理make输出的文件

## 运行
1. 在main.c中,分别在scull_init_module(), scull_read(), scull_write() 加入调试语句, make
2. 使用scull_load 将scull加载进内核,此时内核应该会调用scull_init_module()函数, 可以在 dmesg中看到scull_init_module()输出的日志.
3. 使用ls -l > /dev/scull 向scull写入数据, 会调用scull_write().
4. 使用cat /dev/scull 或者 dd if=/dev/scull 读取 scull, 会调用scull_read() 函数