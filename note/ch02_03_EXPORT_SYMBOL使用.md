[TOC]
# EXPORT_SYMBOLʹ��

��˵,2.6֮ǰ���ں�,Ĭ�Ϸ�static�����ͱ��������Զ������ں˿ռ乩����ģ��ʹ��,����EXPORT_SYMBOL���
��2.6��ʼ,����ʹ��EXPORT_SYMBOL�����ŵ����ں˿ռ�

# ʾ��
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
����/����/ж��
```shell
make modules
insmod mod1.ko
insmod mod2.ko
rmmod mod2
rmmod mod1
```

���
```shell
[  306.992716] Module 1,Init!
[  311.645936] Module 2,Init!
[  311.645968] In Func: func1...
[  311.645983] In Func: func2...
[  321.452650] Module 2,Exit!
[  326.679747] Module 1,Exit!
```

# ��ע
- printk �б������ KERN_ALERT, ���򿴲������
- rmmod ʱ,�����rmmod mod1 �ῴ�� mod1 is in use
- EXPORT_SYMBOL �� System.map������:
	- System.map ������ʱ�ĺ�����ַ,������ɺ�,���ں����й�����,�ǲ�֪���Ǹ��������ĸ���ַ��. ������ļ��Ǹ�����ʹ�õ�,���е�����, kernel����֪��
	- EXPORT_SYMBOL�ķ���,�ǰ���Щ���źͶ�Ӧ�ĵ�ַ��������,���ں����еĹ�����,�����ҵ���Щ���Ŷ�Ӧ�ĵ�ַ
	- module�ڼ��ع�����,�䱾���Ƕ�̬���ӵ��ں�. �����ģ�����������ں˻�����ģ��ķ���, ��Ҫ���ں˺�����ģ����EXPORT_SYMBOL��Щ����EXPORT_SYMBOL, ���������ҵ���Ӧ�ĵ�ַ����. ��ͽ�����ΪʲôҪ�ȼ���mod1