### Attack lab 完



```c
void test()
{
    int val;
    val = getbuf();
    printf("No exploit. Getbuf returned 0x%x\n", val);
}
```

正常执行的话是调用getbuf，然后从屏幕中输入字符串，如果正常退出的话，则会执行,而这个lab就是不让它正常退出。

#### level 1

```c
void touch1() {
    vlevel = 1;
    printf("Touch!: You called touch1()\n");
    validate(1);
    exit(0);
}
```

level1要求输入字符串使程序跳转到函数`touch1`
`touch1`里面没啥东西，所以我们要做的只是使程序跳转。
要使程序跳转到touch1，我们需要将test栈帧顶部存放的地址改写成`touch1`的起始地址

首先反汇编

![20201229183523473](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/20201229183523473.png)

可以看到`touch1`的地址为`0x4017c0`
接下来需要做的是填充getbuf分配的空间，查看`getbuf`的反汇编如下：![20201229183734620](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/20201229183734620.png)

可以看到`%rsp`减了`40`，故我们需要填满这40个字节的缓冲区，然后在接下来的8个字节中，填入`touch1`的地址`0x4017c0`。
这里我选择用字节`00`填充。所以输入字节为

![](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/20201229184756493.png)

![20201229185038129](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/20201229185038129.png)

验证

#### level2

```
void touch2(unsigned val){
    vlevel = 2;
    if (val == cookie){
        printf("Touch2!: You called touch2(0x%.8x)\n", val);
        validate(2);
    } else {
        printf("Misfire: You called touch2(0x%.8x)\n", val);
        fail(2);
    }
    exit(0);
}
```

它判断参数val是否等于cookie，要等于才算过关。
所以我们不仅要使程序跳转到touch2，还得保证传给touch2的参数val必须与cookie相等。
其中cookie是十六进制表示的无符号整数，值为0x59b997fa，
level2与level1的差别之一在于，level2在跳转之前需要将0x59b997fa传给touch2，这一步可以由汇编指令
movq 0x59b997fa, %rdi完成。
我们可以将指令写入getbuf分配的空间中，并将test栈帧中的return address改写成getbuf的栈顶(注入字节序列的起始地址)，这样一来，当getbuf执行ret的时候，CPU会跳转到getbuf的栈顶，执行写入的指令。
而getbuf的栈顶可以通过反汇编得到：

![20201229152111281](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/20201229152111281.png)

这里显示的是执行完第一条sub指令后的状态，这时候的%rsp即为注入字符串的首地址=0x5561dc78
为了使程序执行touch2，需要将touch2的地址push到栈中，然后执行ret指令
使用反汇编得到touch2的地址为：0x4017ec![](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/20201229191947485.png)

对应指令

```
movq    $0x59b997fa, %rdi
pushq   0x4017ec
ret
```

最后将汇编转化为字节码

![](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/2020122919242097.png)

验证

![20201229192524825](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/20201229192524825.png)



#### level 4

```c
/* Compare string to hex represention of unsigned value */
int hexmatch(unsigned val, char *sval) {
    char cbuf[110];
    /* Make position of check string unpredictable */
    char *s = cbuf + random() % 100;
    sprintf(s, "%.8x", val);
    return strncmp(sval, s, 9) == 0;
}

void touch3(char *sval) {
    vlevel = 3; /* Part of validation protocol */
    if (hexmatch(cookie, sval)) {
        printf("Touch3!: You called touch3(\"%s\")\n", sval);
        validate(3);
    } else {
        printf("Misfire: You called touch3(\"%s\")\n", sval);
        fail(3);
    }
    exit(0);
}
```

level3要求输入字符串使程序跳转到函数`touch3`,首先看看函数touch3要干嘛，它调用了函数hexmatch，要使它返回true才算过关。
那么看看hexmatch函数要干嘛，它首先分配了一个110字节的缓冲区，然后在这个缓冲区里面随机取一个地址作为指针s的值，sprintf将参数val存放在s所指的缓冲区内，最后比较参数sval所指的字符串与s所指的字符串是否相等。

所以为了使hexmatch返回true，我们需要使转化成字符串后的cookie，与字符串sval相等。
其中cookie是十六进制表示的无符号整数，值为0x59b997fa，
使用man ascii指令查询cookie的ascii码表示为：
35 39 62 39 39 37 66 61 00

现在我们知道了sval字符串的内容，还要解决一个问题，就是该把它放在哪里？
当调用hexmatch和strncmp时，他们会把数据压入到栈中，有可能会覆盖getbuf栈帧的数据，所以不要放到getbuf的栈帧里面，要放到test的栈帧里。那么就放在test栈帧里返回地址的上面就好了。
使用反汇编工具，查看getbuf函数的汇编代码如下图

![](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/20201229161841627.png)

现在的状态是刚进入函数，还没执行第一条语句，所以rsp指向的是test栈帧的顶部，里面存放的是本该回到的地址，
`rsp`指向`0x5561dca0`，由于地址占8个字节，所以我们的sval字符串应该放在`0x5561dca0+8=0x5561dca8`，这个位置

总的来说，栈的初始状态很简单，如果没有buffer overflow，那么执行完getbuf函数，CPU将回到test函数中紧跟着getbuf()的语句继续执行；这里我们的目的就是使CPU执行完getbuf后，跳转到touch3函数，为了达到这个目的，我们需要把原本应该回到的地址改成我们注入字符串的首地址，如此一来，getbuf函数ret后并不会回到test，而是跳转到我们getbuf的栈顶，开始执行我们写入的指令，这些指令的任务就是使CPU去执行函数touch3。

- 为了使程序跳转到`touch3`，需要把`touch3`的起始地址`push`到栈中
  使用`disas touch3`得到函数touch3的反汇编代码

  ![](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/20201229143013399.png)

- 在执行`touch3`之前，需要把之前得到的与`cookie`相等的字符串sval的首地址作为参数1传递给`touch3`

- 最后只剩下注入字符串的首地址未知了，我们直接从栈顶注入字符串，所以首地址就是`getbuf`的栈顶，即执行完第一条汇编指令后`%rsp`所指的地址

  ![a2fc333659ada99b0cc2fb0b3246d76](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/a2fc333659ada99b0cc2fb0b3246d76.jpg)

这时候的`%rsp`即为我们注入字符串的首地址=`0x5561dc78`
结合上面我们可以得出存放在`getbuf`栈帧中的注入代码为

```
movq    $0x5561dca8, %rdi
pushq   0x4018fa
ret
```

将命令转化成数字序列![](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/20201229154014246.png)

验证

![7bc170612aeb3e2622463d4a9ea4aec](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/7bc170612aeb3e2622463d4a9ea4aec.jpg)

#### phase 4

`phase4`其实就是用另一种方法实现`phase2`的工作。
要将`cookie`作为参数传入`touch2`，可以用以下指令完成

```
popq %rax
movq %rax, %rdi
```

在getbuf ret之前，%rsp指向`popq %rax ret`指令所在的地址,

当getbuf `ret`后，`popq %rax ret`指令所在的地址被pop到%rip，指示CPU去执行语句`popq %rax`，并且将`%rsp`加8，使其指向栈中的前一个值

`popq %rax`这句话相当于`movq (%rsp) %rax``addq $8 %rsp`

`ret`后， `movq %rax, %rdi` 所在的地址被pop到%rip，指示CPU去执行语句 `movq %rax, %rdi` ，并且将`%rsp`加8，使其指向栈中的前一个值

这时我们把前一个值改写成cookie,那么%rsp指向的地址内存放的就是cookie的值。
即(%rsp)==cookie，这样一来，语句popq %rax将cookie的值赋给%rax，并将%rsp加8指向 movq %rax, %rdi 所在的地址。
movq %rax, %rdi 把%rax赋给%rdi，即将要调用的函数touch2的第一个参数，再通过ret指令跳转到touch2。
接下来要做的就是找到这两个gadget的地址了。
popq %rax对应的 字节是 58，
ret 是 c3
movq %rax, %rdi是48 89 c7
所以我们的任务就是将farm.c变成字节序列，然后从中找到58 c3以及48 89 c7 c3这两个序列，记下他们的地址

搜索`58`发现么有`58 c3`，只有`58 90 c3`，`90`其实代表的是指令no operation,就是啥都不做，相当于跳过的意思。
所以这里用`58 90 c3`效果是一样的。记下地址为`0x5c`

![](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/20201230153234587.png)

同理记下地址`0x51`

![](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/20201230153305770.png)

得到输入字节序列为

```
00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00
5c 00 00 00 00 00 00 00
fa 97 b9 59 00 00 00 00
51 00 00 00 00 00 00 00
ec 17 40 00 00 00 00
```

#### level 5

在这一阶段中，我们需要做的就是把字符串的起始地址传送到%rdi,然后调用touch3函数。

因为每次栈的位置是随机的，所以无法直接用地址来索引字符串的起始地址，只能用栈顶地址 + 偏移量来索引字符串的起始地址。从farm中我们可以获取到这样一个gadget :lea (%rdi,%rsi,1),%rax，这样就可以把字符串的首地址传送到%rax。

思路：

	- 首先获取到%rsp的地址，并且传送到%rdi 
	- 其二获取到字符串的偏移量值，并且传送到%rsi
	- lea (%rdi,%rsi,1),%rax, 将字符串的首地址传送到%rax, 再传送到%rdi
	- 调用touch3函数

第一步，获取到%rsp的地址![](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/20201230185012195.png)

`movq %rsp, %rax`的指令字节为：48 89 e0, 所以这一步的gadget地址为：`0x1c5`

第二步，将%rax的内容传送到%rdi![](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/20201230185202338.png)

`movq %rax, %rdi`的指令字节为：48 89 c7，所以这一步的gadget地址为：`0x1a`

第三步，将偏移量的内容弹出到%rax\

![](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/20201230153234587.png)

`popq %rax`的指令字节为：58， 其中90为nop指令, 所以这一步的gadget地址为：`0x5c`

第四步，将%eax的内容传送到%edx![](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/20201230185433337.png)

`movl %eax, %edx`的指令字节为:`89 c2`, 所以这一步的gadget地址为：`0x79`

第五步，将%edx的内容传送到%ecx

![](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/2020123019012831.png)

`movl %edx, %ecx`的指令字节为：`89 d1`，所以这一步的gadget地址为：`0x164`

第六步，将%ecx的内容传送到%esi

![0000000000401a11 <addval_436>:401a11: 8d 87 89 ce 90 90     lea    -0x6f6f3177(%rdi),%eax401a17: c3                    retq](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/20201230190202485.png)

`movl %ecx, %esi`的指令字节为：`89 ce`, 所以这一步gadget地址为：`0xcf`

第七步，将栈顶 + 偏移量得到字符串的首地址传送到%rax![](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/20201230190333236.png)

这一步的gadget地址为：`0x6a`

将字符串首地址%rax传送到%rdi

![](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/20201230190432854.png)

`movq %rax, %rdi`的指令字节为：`48 89 c7`，所以这一步的gadget地址为：`0x51`
综上，我们可以得到字符串首地址和返回地址之间隔了9条指令，所以偏移量为`72`个字节，也就是`0x48`，可以的到如下字符串的输入：![](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/20201230191256434.png)