[TOC]
# Scull ѧϰ�ʼ�
## Scull �ļ��ṹ
|	�ļ�					|	����			|
| :-----------------:	| :-----------: 	|
|	main.c				|	������scull�豸�ĳ�ʼ����ж�ء�open�� write��ʵ�ֶ���������	|	
|	Makefile			|					|
|	pipe.c				|	�����¡��߼��ַ�����������������õ������������������� ��	|
|	scull.h				|	ͷ�ļ�		|
|	scull.init			|	����scullģ��Ľű�	|
|	scull_unload	|	ж��scullģ��Ľű�	|

## ����
- make : ����,��������scull.ko
- ��װ: �ű�scull_load�� scull�豸�ͻ��Զ����ص��ں�. ��ʱ����ͨ���鿴/proc/devices�ļ��ҵ�scullģ�飬�����ں�Ϊ���������豸��,ͬʱ��/dev/��Ҳ���Ӻܶ���scull��ͷ���ַ��豸.
- ж��: �ű�scull_unload, ���/proc/devices �Լ�/dev/��scull�豸
- make clean : ����make������ļ�

## ����
1. ��main.c��,�ֱ���scull_init_module(), scull_read(), scull_write() ����������, make
2. ʹ��scull_load ��scull���ؽ��ں�,��ʱ�ں�Ӧ�û����scull_init_module()����, ������ dmesg�п���scull_init_module()�������־.
3. ʹ��ls -l > /dev/scull ��scullд������, �����scull_write().
4. ʹ��cat /dev/scull ���� dd if=/dev/scull ��ȡ scull, �����scull_read() ����