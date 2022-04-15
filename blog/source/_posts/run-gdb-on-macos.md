---
title: 在MacOS上使用GDB调试C程序
comment: true
date: 2022-04-09 09:47:03
categories:
  - C语言
tags:
  - C语言
  - GDB
---

## 编写C程序

编写一段简单的C程序 main.c ，并做好验证

```c
#include <stdio.h>

int main() {
  int i = 0;
  printf("hello world\n");
  i = i + 1;
  printf("%d\n", i);
}
```

编译（保证机器上有装gcc，若没有装可以采用 brew install gcc 完成安装），并执行

```bash
$ gcc -g main.c -o main.o
$ ./main.o
hello world
1
```

<!--more-->

## 如何调试

### 先安装GDB

```bash
$ brew install gdb
```

完成安装后，执行gdb看是否输出正常

```bash
$ gdb
GNU gdb (GDB) 11.2
Copyright (C) 2022 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-apple-darwin21.1.0".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word".
(gdb) 
```

### 开始调试

在c程序的目录下，执行如下命令进入gdb调试界面

```bash
$ sudo gdb main.o
GNU gdb (GDB) 11.2
Copyright (C) 2022 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-apple-darwin21.1.0".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from main.o...
Reading symbols from /Users/ahern88/cCodes/main.o.dSYM/Contents/Resources/DWARF/main.o...
(gdb) l # 通过 l 命令查看代码
1	#include <stdio.h>
2	
3	
4	int main() {
5	 int i = 0;
6	 printf("hello world\n");
7	 i = i + 1;
8	 printf("%d\n", i);
9	}
(gdb) layout src # 通过layout查看main.c的代码布局
(gdb) b 5 # 在第5行代码插入断点，这样在layout模式下能看到 B+ 的状态，代表断点位置
(gdb) run # 开始运行程序, 这里执行可以hang住，后面有解决办法
Starting program: /Users/ahern88/cCodes/main.o 
[New Thread 0x1603 of process 64447]
[New Thread 0x1903 of process 64447]
warning: unhandled dyld version (17)

Thread 2 hit Breakpoint 1, main () at main.c:5
5	 int i = 0;
(gdb) info break # 查看当前断点的信息
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000100003f48 in main at main.c:5
(gdb) next # 单步执行
(gdb) continue # 执行到下一个break断点
(gdb) print i # 输出当前栈中的变量i
```

上面能完成基本的调试操作，可以尝试着自己做一做

## 遇到的问题

在使用gdb的过程中我遇到了几个问题

1）run 操作后，提示Unable to find Mach task port for process-id

解决办法：可能执行gdb命令的时候没有加sudo，加上sudo gdb就可以了

2）run 操作后，命令行hang住不返回

解决办法：在当前用户目录下新增 .gdbinit文件，并加入```set startup-with-shell off``` 

```bash
$ touch ~/.gdbinit
$ echo "set startup-with-shell off" > ~/.gdbinit
```

> 记住修改完 .gdbinit 文件后一定要关闭term窗口后重新打开，不然会没有效果

## 参考资料

- [(gdb) 8.3.1 hangs after run command on mac Catalina #49631](https://github.com/Homebrew/homebrew-core/issues/49631)

- [MacOS Catalina下使用gdb进行调试遇到的几个问题](https://blog.csdn.net/donaldsy/article/details/106739316)
