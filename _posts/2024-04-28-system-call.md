---
layout: post
title: "系统调用概述"
categories: operating-system
tags: [system call]
---

系统调用原理（Linux32位）

参考素材：【操作系统 (九)：Linux 系统调用是如何实现的？】

https://www.bilibili.com/video/BV1TX4y1p7pc?vd_source=ca3e56f329e1a3ac9a87638d40ee40a7

![My helpful screenshot](/assets/system-call/1.png)

以write()为例：

从启动加载程序向linux内核的跳转是由汇编语句：__enter_kernel完成。

write()产生中断指令，中断号为0x80并被送入cpu

cpu查询中断向量表（中断号：中断服务程序内存地址），得到中断服务程序内存地址为0x003498，PC指向这个内存地址

![My helpful screenshot](/assets/system-call/2.png)

中断服务程序中有许多不同的系统调用接口（系统调用号：系统调用函数），如1:sys_open,2:sys_fork,3:sys_write

早在系统在用户态时，便存在存在着方法名和系统调用号的映射关系，比如write()对应3

之后系统调用号送入cpu寄存器中，使得EAX=3,从而根据系统调用号得到系统调用函数

此外write函数的参数也被保存到了cpu寄存器中，保存参数的寄存器最多有6个

![My helpful screenshot](/assets/system-call/3.png)

将用户态的寄存器保存到pt_regs的数据结构中

在sys_call_table中根据系统调用号找到对应函数并执行

将函数返回值写入pt_regs的ax位置，ax在写回到EAX寄存器中

通过指令iret根据pt_regs恢复用户态程序，读取EAX寄存器中的值，得到write()函数结果

![My helpful screenshot](/assets/system-call/4.png)