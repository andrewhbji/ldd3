# 这是我第一次深入Linux内核 hello.c

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

我第一次都干了些什么呢?很简单,就尝了尝了一下Linux模块编程架构
而Linux设备驱动的开发,无论小到这个HelloWorld,还是驱动USB,还是大到驱动Android平台的RF模块收发消息,都利用的是Linux模块编程.

在Linux模块编程中,无论任何模块代码,首先要找的是module_init和module_exit这两个宏. 
- module_init宏的参数是模块被装载时要调用的函数,这是模块的入口;
- module_exit宏的参数,则是模块被卸载时要调用的函数,在这里要完成卸载模块前的全部清理工作.

所以对于hello.c,代码编译后,装载模块时调用hello_init;卸载模块时,调用hello_exit

## 编译模块
编译hello.c,需要指定内核源码树(从内核代码编译得到),并提供一个Makefile文件.

### Makefile 文件
Makefile只需要一句话:
```
obj-m := hello.o
```
### 使用Make编译模块以及编译过程分析
``` shell
make -C /usr/src/linux-2.6.10 M=`pwd` modules
```
命令说明:
- -C指定之前提到的Linux内核源码树的路径
这个路径下Linux内核顶层的Makefile将被作为整个内核构建系统的入口
- M指定module目标的生成路径
M=后面是反引号(ESC键下面)而不是单引号,里面的pwd表示把pwd命令执行的结果(即hello.c所在路径)赋值给M

执行这个命令,可以看到生成了hello.ko,即hello模块

怎么有一种直接将对手KO的即视感...

我们再看看输出日志,卧槽,真的是打出一记组合拳将直接对手KO啊,这Make命令太强大了
``` shell
$ make -C /usr/src/linux-2.6.10 M=`pwd` modules
make[1]: Entering directory `/usr/src/linux-2.6.10'
 CC [M] /home/ldd3/src/misc-modules/hello.o
 Building modules, stage 2.
 MODPOST
 CC /home/ldd3/src/misc-modules/hello.mod.o
 LD [M] /home/ldd3/src/misc-modules/hello.ko 
make[1]: Leaving directory `/usr/src/linux-2.6.10'
```
这短短的一瞬就发生了这么多,究竟是怎么做到的?

再看一遍LDD3 ch02 s4 , 发现有这么一句: "是内核构建系统处理了余下的工作",好像明白了些什么...

在往后看,找到了对全过程的描述,
…… 
obj-m := hello.o表明有一个模块要冲目标文件hello.o建立
……
makefile,它需要在更大的内核构建系统的环境中被调用
……
这条命令开始是改变他的目录到用-C选项提供的目录下,在那里会发现内核的顶层makefile,M=选项使makefile在试图建立目标前,回到模块源码目录
……
等等云云

### 完善Makefile文件
仅仅 make hello.c这个模块还好, 但是以后还要make更多的模块, 还需要一遍一遍重复的写makefile,并重复敲这么复杂的make命令吗?作为一个能少些代码就少写代码的程序猿,答案当然是: no
好在LDD3提供了一个比较好的Makefile模板:
```shell

# To build modules outside of the kernel tree, we run "make"
# in the kernel source tree; the Makefile these then includes this
# Makefile once again.
# This conditional selects whether we are being included from the
# kernel Makefile or not.
ifeq ($(KERNELRELEASE),)

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
- make clean: 清理上面两条命令的执行执行后的临时文件.如果需要清理make modules_install,需要su权限
- make .PHONY:执行一次make modules_install,然后清理临时文件

## 测试模块
测试模块的安装/卸载需要su权限
### 安装模块
安装模块使用insmod命令,以hello.ko为例:
```shell
sudo insmod hello.ko
```

### 卸载模块
卸载模块使用rmmod命令,以hello.ko为例:
```shell
sudo rmmod hello
```
### 查看输出
使用dmsg查看输出日志
```shell
desg
```
可以看到，除了刚才安装模块时hello_init打印的”Hello, world”，又多了一条语句”Goodbye, cruel world”，这句话就是模块卸载函数hello_exit函数打印的。

## 总结
从学习LDD3开始正式进入系统学习Linux内核之旅.这里先了解了Linux模块编程的概念,学习了Linux模块编程架构以及加载和卸载的方法,并利用makefile模板来完成编译工作.感觉C语言几乎都要还给老师了,并且对于makefile几乎没有概念.穿插复习C语言以及学习makefile编写将是学习LDD3过程中非常重要的事.