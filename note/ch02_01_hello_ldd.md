[TOC]
# 从这里开始深入理解Linux内核 hello.c

```C
#include <linux/init.h>
#include <linux/module.h>
MODULE_LICENSE("Dual BSD/GPL");

static int hello_init(void){
       printk(KERN_ALERT "Hello, world\n");
       return 0;
}
static void hello_exit(void){

       printk(KERN_ALERT "Goodbye, cruel world\n");
}

module_init(hello_init);
module_exit(hello_exit);
```

这个模块代码都做了些什么?很简单, 仅仅使用了一下Linux模块编程架构.
Linux设备驱动的开发, 无论是这个HelloWorld, 还是驱动USB, 还是驱动Android平台的RF模块收发消息, 都利用的是Linux模块编程.

在Linux模块编程中, 无论任何模块代码, 首先要找的是module_init和module_exit这两个宏. 
- module_init宏的参数是模块被装载时要调用的函数, 这是模块的入口;
- module_exit宏的参数, 则是模块被卸载时要调用的函数, 在这里要完成卸载模块前的全部清理工作.

所以对于hello.c, 代码编译后, 装载模块时调用hello_init;卸载模块时, 调用hello_exit

## 编译模块
编译hello.c, 需要指定内核源码树(从内核代码编译得到), 并提供一个Makefile文件.

### Makefile 文件
Makefile只需要一句话:
```
obj-m := hello.o
```
### 使用Make编译模块以及编译过程
``` shell
make -C /usr/src/linux-2.6.10 M=`pwd` modules
```
命令说明:
- -C指定之前提到的Linux内核源码树的路径
这个路径下Linux内核顶层的Makefile将被作为整个内核构建系统的入口
- M指定module目标的生成路径
M=后面是反引号(ESC键下面)而不是单引号, 里面的pwd表示把pwd命令执行的结果(即hello.c所在路径)赋值给M

执行这个命令,  可以看到生成了hello.ko, 即hello模块

问题是, 我仅仅指定了linux内核的路径以及输出路径.那么是什么帮我完成了全部的工作?

我们再看看命令输出的日志
``` shell
$ make -C /usr/src/linux-2.6.10 M=`pwd` modules
make[1]: Entering directory `/usr/src/linux-2.6.10'
 CC [M] /home/ldd3/src/misc-modules/hello.o
 Building modules,  stage 2.
 MODPOST
 CC /home/ldd3/src/misc-modules/hello.mod.o
 LD [M] /home/ldd3/src/misc-modules/hello.ko 
make[1]: Leaving directory `/usr/src/linux-2.6.10'
```
再看一遍LDD3 ch02 s4 ,  发现有这么一句: "是内核构建系统处理了余下的工作", 好像明白了些什么...

在往后看, 找到了对其全过程的描述, 
…… 
obj-m := hello.o表明有一个模块要冲目标文件hello.o建立
……
makefile, 它需要在更大的内核构建系统的环境中被调用
……
这条命令开始是改变他的目录到用-C选项提供的目录下, 在那里会发现内核的顶层makefile, M=选项使makefile在试图建立目标前, 回到模块源码目录
……
原来, Linux模块的开发是依赖整个Linux构建环境的. 理解到这里, 上面的问题就迎刃而解.

### 完善Makefile文件
仅仅 make hello.c这个模块还好,  但是以后还要make更多的模块,  需要一遍一遍的复制粘贴makefile, 和这个复杂的make命令吗?答案当然是: no
LDD3的作者为我们提供了一个比较好的Makefile模板:
```shell

# To build modules outside of the kernel tree,  we run "make"
# in the kernel source tree; the Makefile these then includes this
# Makefile once again.
# This conditional selects whether we are being included from the
# kernel Makefile or not.
ifeq ($(KERNELRELEASE), )

	# Assume the source tree is where the running kernel was built
	# You should set KERNELDIR in the environment if it's elsewhere
	KERNELDIR ?= /lib/modules/$(shell uname -r)/build
	# The current directory is passed to sub-makes as argument
	PWD := $(shell pwd)

modules:
	$(MAKE) -C $(KERNELDIR) M=$(PWD) modules

modules_install:
	$(MAKE) -C $(KERNELDIR) M=$(PWD) modules_install

clean:
	rm -rf *.o *~ core .depend .*.cmd *.ko *.mod.c .tmp_versions

.PHONY: modules modules_install clean

else
	# called from kernel build system: just declare what our modules are
	obj-m := hello.o
endif
```
这个makefile提供这几个功能:
- make 或者 make modules: 编译 obj-m指定的模块
- make modules_install: 编译并安装模块
- make clean: 清理上面两条命令的执行执行后的临时文件. 如果需要清理make modules_install,  需要su权限
- make .PHONY:执行一次make modules_install,  然后清理临时文件

## 测试模块
剩下就是要测试模块了.
模块建立后,  下一步是加载到内核, 测试模块的加载/卸载需要su权限

### 加载模块
安装模块使用insmod命令,  以hello.ko为例:
```shell
sudo insmod hello.ko
```
- 这个程序加载模块的代码段和数据段到内核, 接着,  执行一个类似 ld 的函数,  它连接模块中任何未解决的符号连接到内核的符号表上. 但是不象连接器,  内核不修改模块的磁盘文件,  而是内存内的拷贝.
- 另外还有一个工具需要提一下: modprobe
modprobe和 insmode功能相同,  区别在于 modprobe会查看要加载的模块, 看它是否引用了当前内核中没有定义的符号. modprobe 在定义相关符号的当前模块搜索路径中寻找其他模块. 当 modprobe 找到这些模块( 要加载模块需要的 ),  它也把它们加载到内核. 如果你在这种情况下代替以使用 insmod,  命令会失败,  在系统日志文件中留下一条"nresolved symbols "消息. 

### 卸载模块
卸载模块使用rmmod命令, 以hello.ko为例:
```shell
sudo rmmod hello
```
注意,  如果内核认为模块还在用( 就是说,  一个程序仍然有一个打开文件对应模块输出的设备 ),  或者内核被配置成不允许模块去除,  模块去除会失败. 可以配置内核允许"强行"去除模块,  甚至在它们看来是忙的. 如果你到了需要这选项的地步,  但是,  事情可能已经错的太严重以至于最好的动作就是重启了. 

### 查看模块列表
lsmod 程序生成一个内核中当前加载的模块的列表, 以及一些其他信息, 例如使用了一个特定模块的其他模块. lsmod 通过读取 /proc/modules 虚拟文件工作. 当前加载的模块的信息也可在位于 /sys/module 的 sysfs 虚拟文件系统找到. 

### 查看输出
使用dmsg查看输出日志
```shell
desg
```
可以看到，除了刚才安装模块时hello_init打印的”Hello, world”，又多了一条语句”Goodbye, cruel world”，这句话就是模块卸载函数hello_exit函数打印的.

## 总结
从学习LDD3开始正式进入系统学习Linux内核之旅. 这里先了解了Linux模块编程的概念, 学习了Linux模块编程架构以及加载和卸载的方法, 并利用makefile模板来完成编译工作. 感觉C语言几乎都要还给老师了, 并且对于makefile几乎没有概念. 穿插复习C语言以及学习makefile编写将是学习LDD3过程中非常重要的事.