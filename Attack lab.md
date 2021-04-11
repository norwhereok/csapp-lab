### Attack lab

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



剩下的还没来得及做  下周继续