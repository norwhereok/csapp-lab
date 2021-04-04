### bomb lab

1. 反编译bomb

   ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps1.jpg)

2. 找到phase_1

   ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps22.jpg)

   由汇编代码可知是字符串比较，并且eax中可能存放着与输入比较的字符，用gdb查看地址内字符

   ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps46.jpg)

3. phase_2:

   ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps66.jpg)

   发现调用了read_six_numbers函数，发现是读入并确认读入的数字为六个整数否则引爆。

   cmpl  $0x0,0xffffffe4(%ebp)将第一个数字与0比较相等否则jne   8048d20 <phase_2+0x25>引爆

   cmpl  $0x1,0xffffffe8(%ebp) 第二个数字与1进行比较

   ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps95.jpg)![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps131.jpg)

   一个循环将ebx+fffffff8地址的数字和ebx+ffffffc地址的数字相加与ebx比较不相同则引爆，然后将ebx的值加4并比较是否已经达到最后一个数字

   可以看出第三个数字要为一二数字的和，第四个要为二三的和，所以推算得到密码为六个数字0 1 1 2 3 5

4. phase_3:

   ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps151.jpg)

   ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps172.jpg)![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps190.jpg)

   查询以后发现是一个C语言内部函数，查到定义int sscanf(const char *str, const char *format,…)，发现3中接受了两个参数，一个为用户输入的参数，一个为系统内存中的参数，预计为格式控制的字符。返回一个值为实际有效的参数个数

   在8048e61p 设置断点

   

![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps225.jpg)

​							![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps252.jpg)



​			查看内存，发现需求值为一个整数，一个字符，一个整数

![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps299.jpg)

​			存入的地址分别是ebp+fffffff8 ebp+fffffff7 ebp+fffffffc

![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps326.jpg)

​			如果读入的有效参数小于等于2则引爆，然后将7与读入的第一个参数比较，如果超过则引爆，因此第一个参数小于7.

​			![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps370.jpg)

​			![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps410.jpg)

​		查看内存发现804a0a8中存的指令地址，为switch case的跳转表，因此发现是根据输入的一个参数进行指令跳转，决定执行那一部分代码，由此判断其C语言结构应该为switch case

​		单独取一个case

​		![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps444.jpg)

​		首先讲eax赋一个值，将0x170=368与输入的第三个值比较，因此第三个值应该为411跳转之后

​	![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps466.jpg)

​		要求输入的字符的ascii码为之前eax中al的值即a

​		其他几个case同理得到答案，任意一组均可通过。

​		2 a 368

5. Phase_4

   ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps486.jpg)

   调用sscanf@plt函数和fun4函数。

   ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps515.jpg)

   设置gdb断点查看内存0x804a392中的内容

   ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps637.jpg)

   确认函数需要的输入值为两个整数

   ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps681.jpg)

   将第一个是与0 14 进行比较要求其大于0小于等于14

   ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps722.jpg)

   将第一数字作为第一个参数，0为第二个参数，14为第三个参数传入func4，

   ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps773.jpg)

   要求返回值与第二数字等于10

   ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps813.jpg)

   现fun4调用自身，是一个递归调用。

   ```c++
   1.int func4(int a1, int a2, int a3)  
   2.{  
   3.  int v3;  
   4.  v3 = (a3 - a2) / 2 + a2;  
   5.  if ( v3 <= a1 )  
   6.  {  
   7.    if ( v3 < a1 )  
   8.      v3 += func4(a1, v3 + 1, a3);  
   9.  }  
   10.  else  
   11.  {  
   12.    v3 += func4(a1, a2, v3 - 1);  
   13.  }  
   14.  return v3;  
   } 
   ```

   根据代码分析得 答案3 10

   6. Phase_5

      ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps859.jpg)

      这一关用到了string_length和string_not_equal，可以推断该关与字符串有关，

      ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps877.jpg)

      目的应该为输入一个长度为6的字符串，

      ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps889.jpg)

      此处为一个循环

      ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps954.jpg)取每一个字符的后四位，

      ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps955.jpg)根据后四位进行一个寻址操作将内存中的值取出，并放进ecx所在的一个新字符串。

      ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps956.jpg)

      比较新字符串与内存中的预设字符串，要求相等。

      使用GDB调试

      ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps1080.jpg)设置断点

      ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps1081.jpg)查看系统中用来变化的字符串

      ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps1082.jpg)系统用来比较的预置字符串，根据比较发现需要选取ASCLL码后四位 d 6 3 4 8 7的字符组成的字符串，这里选用jtoefg

7. Phase_6

   ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps1159.jpg)

   ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps1191.jpg)

   ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps1227.jpg)

   该关是要输入六个整数。

   ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps1335.jpg)

   这是一个大的循环过程，其中还包含一个循环

   ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps1336.jpg) 

   这里可知外部循环首先第一个数字减去1要比5小，即第一个数字要小于6；

   然后循环查看其他5个数字有没有重复的，由重复则炸。然后再次循环发现是第二个数字要比6小，且不能跟后面4个数字重复。因此它这个大循环就是要检查6个数字是否相等。

    

   ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps1337.jpg)接着又是一个循环，它根据输入的数字，去找预置在对应次序的数字。

   ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps1338.jpg)

   最后一步就是对取出来的数字进行验证，是否为从大到小排序。

   ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps1378.jpg) 

   发现地址0x0804B1E4放的是预置的第一个数字，接着+8放的是下一个数字的地址，因此可推断它是一个链表。

   ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps1379.jpg) 

   则从大到小顺序应该是 3 4 5 1 2 6

   8. secret_phase

      在查看汇编代码时发现有一个secret_phase，而在C语言源代码中没有发现该函数，搜索汇编代码后发现在phase_defused函数中有调用secret_phase，观察C语言代码，发现每关结束后均调用phase_defused函数。

      ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps1380.jpg) 

      在该函数设置断点查看后发现0x804b894上的值每关结束后均加1判断为关卡计数，因此只有通过六关才有机会进入隐藏关卡。

      ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps1381.jpg) 

      sscanf@plt函数的值，发现一个为格式控制，另外一个为在第四关输入的参数，因此推断进入隐藏关卡的要求为在第四关额外输入一个字符串，

      ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps1382.jpg) 

      查看传入strings_not_equal的内存地址，确定要求额外输入的字符串应该为DrEvil,比较成功后，便进入secret_phase

      ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps1383.jpg) 

      发现其中调用![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps1384.jpg)函数，发现类型与sscanf@plt类型相似，应该为C语言自带函数，进行查找后确定为long int strtol(const char *nptr,char **endptr,int base)函数，效果为把一个字符串，转换成long int型的整数，因此该函数的作用应该为将隐藏关卡输入的字符串转换为长整数。然后用这个数和36调用fun7函数。

      ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps1385.jpg) 

      fun7函数发现其为一个递归函数。

      分析函数出口edx是第一参数所指向的 ecx是第二参数，ebx=0时传回-1，要求最后递归传回的结果eax=5。根据需要的结果，追溯函数，发现要求三次循环时输入![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps1386.jpg)

      因此输入的数字要大于36，进入

       

      ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps1387.jpg)该块函数![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps1388.jpg)

      利用GDB查看传入的参数发现为50，根据追溯的结果需求要求数字小于50，

      ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps1389.jpg) 

      进入该块，查看内存得到![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps1390.jpg)

      因此输入数字大于45，最后一次传入要求edx=ecx

      ![img](https://norwhere-1301825653.cos.ap-shanghai.myqcloud.com/markdown/wps1391.jpg)查看参数确定最后要求输入的数字应该为47，同时也符合上方所有要求，因此确认隐藏关卡的答案为47。

      

