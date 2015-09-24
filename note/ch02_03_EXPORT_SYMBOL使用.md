[TOC]
# EXPORT_SYMBOL使用

据说,2.6之前的内核,默认非static函数和变量都会自动导入内核空间供各个模块使用,不用EXPORT_SYMBOL标记
从2.6开始,必须使用EXPORT_SYMBOL将符号导入内核空间

# 示例
mod1.c
``` C
#include<linux/init.h>
#include<linux/module.h>
#include<linux/kernel.h>

static int func1(void)
{
        printk(KERN_ALERT "In Func: %s...\n",__func__);
        return 0;
}

EXPORT_SYMBOL(func1);

static int __init hello_init(void)
{
        printk(KERN_ALERT "Module 1,Init!\n");
        return 0;
}

static void __exit hello_exit(void)
{
        printk(KERN_ALERT "Module 1,Exit!\n");
}

module_init(hello_init);
module_exit(hello_exit);
```
mod2.c
```c
#include<linux/init.h>
#include<linux/kernel.h>
#include<linux/module.h>

static int func2(void)
{
        extern int func1(void);
        func1();
        printk(KERN_ALERT "In Func: %s...\n",__func__);
        return 0;
}

static int __init hello_init(void)
{
        printk(KERN_ALERT "Module 2,Init!\n");
        func2();
        return 0;
}

static void __exit hello_exit(void)
{
        printk(KERN_ALERT "Module 2,Exit!\n");
}

module_init(hello_init);
module_exit(hello_exit);
```
编译/加载/卸载
```shell
make modules
insmod mod1.ko
insmod mod2.ko
rmmod mod2
rmmod mod1
```

输出
```shell
[  306.992716] Module 1,Init!
[  311.645936] Module 2,Init!
[  311.645968] In Func: func1...
[  311.645983] In Func: func2...
[  321.452650] Module 2,Exit!
[  326.679747] Module 1,Exit!
```

# 备注
- printk 中必须加入 KERN_ALERT, 否则看不到输出
- rmmod 时,如果先rmmod mod1 会看到 mod1 is in use
- EXPORT_SYMBOL 和 System.map的区别:
	- System.map 是链接时的函数地址,连接完成后,在内核运行过程中,是不知道那个符号在哪个地址的. 而这个文件是给调试使用的,其中的内容, kernel并不知道
	- EXPORT_SYMBOL的符号,是把这些符号和对应的地址保存起来,在内核运行的过程中,可以找到这些符号对应的地址
	- module在加载过程中,其本质是动态链接到内核. 如果在模块中引用了内核或其他模块的符号, 就要在内核和其他模块中EXPORT_SYMBOL这些符号EXPORT_SYMBOL, 这样才能找到对应的地址链接. 这就解释了为什么要先加载mod1