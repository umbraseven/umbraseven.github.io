---
layout: post
title:  "汇编：一些基本主题"
date:   2020-06-11 18:30:00 +0800
categories: 编程语言
tags: 编程语言 汇编
description: 
---

*  目录
{:toc}

***
## 汇编：一些基本主题的讨论

2018.05

***
### 函数调用过程

c语言源程序 example.c：

```c
int add_a_and_b(int a, int b) {
   return a + b;}

int main() {
   return add_a_and_b(2, 3);}
```

编译： gcc -S example.c，生成exapmle.s（大概的样子）：

```
_add_a_and_b:
   push   %ebx
   mov    %eax, [%esp+8] 
   mov    %ebx, [%esp+12]
   add    %eax, %ebx 
   pop    %ebx 
   ret  

_main:
   push   3
   push   2
   call   _add_a_and_b 
   add    %esp, 8
   ret
```

注意点：
* 栈底在高地址，往下生长。所以push时会将esp减小，恢复时需要add %esp, 8。
* 默认的调用方式约定，由调用方清理栈，所以该恢复操作是由main函数做的。
* 默认的调用方式约定，后来的参数先入栈，在函数中取出参数时，使用mov %eax, [%esp+8]。
* intel语法习惯 op destination source。比如mov时，是将后者得到的值写入前者指向的内存；比如add时，是将两者相加后，将值写入前者。
* call/ret，会有一系列操作，比如压pc入栈，更新pc等等。（详情参考指令集架构专题）

***
### 和c交互

汇编程序foo.asm：
```
extern bar_func;

[section .data]
arg1  dd 3
arg2  dd 4

[section .text]
global _start
global foo_print

_start:
mov   eax, dword[arg1]
push  eax
mov   eax, dword [arg2]
push  eax
call  bar_func
add   esp, 8

mov   ebx,0
mov   eax, 1
int   0x80

foo_print:
mov   edx, [esp + 8]
mov   ecx, [esp + 4]
mov   ebx, 1
mov   eax, 4
int   0x80
ret
```

c程序bar.c：

```c
void foo_print(char* a, int len);

int  bar_func(int a, int b) {
    if (a > b) {
       foo_print("the 1st one\n", 13);
    } else {
       foo_print("the 2nd one\n", 13);
    }

    return 0;
}
```

* foo_print，这是在asm里定义的函数，可以给外部的c程序调用。只要按照编译器对函数解析、定义的方式去写就可以了，比如在合适的地方取参数，返回。

* \_start，这是汇编代码的入口函数，在其中调用了c函数bar_func。同样需要按照约定来，按照一定的顺序压入参数，调用函数，清理栈等。

* bar_func，c函数，在其中调用了汇编函数foo_print，正常调用即可。

所以，不管是c实现，还是汇编实现，函数本身是没什么区别的，都是在栈上进行的一系列的计算操作。这也就为c和汇编的协作提供了基础。能这么玩的原因是c的编译过程：c语言的编译是以c文件为单位的，对于外部变量/函数，只要有声明而不一定需要有定义。所以实际上，和任何外部函数的链接都是在二进制文件的基础上做的，完全可以用汇编编写。
 
***
**补充**

* 要想真正理解、记住，最后是自己动手写一写。
* c函数和汇编函数在本质上是一致的——它们都会被转化为汇编的形式，并在这种形式上进行交互。所以只要理解了c函数的调用与被调用，是如何转化为汇编代码的，我们就能很自如地完成它们之间的交互。
* 调用方和被调用方有约定上的配合：调用方负责压入参数，并丢出参数（清理堆栈），这两个工作是由同一方完成的，以确保堆栈的正确性。而被调用方原则上只使用堆栈上的数据，而不改变堆栈——如果有下级调用，那它作为调用方就要确保堆栈的正确性。因为堆栈一旦错误，就可能在ret时返回到未知的地址，造成系统崩溃，或者受到攻击。

***
### 反汇编

《HelloOS》教程里，为了同时用汇编和c开发，又得到一体式的汇编代码，借用了反汇编的方式，还挺有意思的，记录一下：

>做法是这样的： 
>1. 先写好汇编代码和对应的C代码。 
>2. 用以下命令编译C代码模块，以便后面反汇编：gcc -m32 -fno-asynchronous-unwind-tables -s -c -o bar.o bar.c 
>3. 下载一个好用的反汇编工具objconv，通过如下命令下载：git clone https://github.com/vertis/objconv.git 
>4. 下载后进入objconv目录，编译该工具，运行下面的命令：g++ -o objconv -O2 src/\*cpp ， -O2中的圆圈是大写字母O. 
>5. 用objconv 反汇编C语言生成的目标文件bar.o,命令如下：objconv -fnasm bar.o -o bar.asm,于是目录下便有一个反汇编文件bar.asm 
>6. 打开foo.asm, 将里面的_start, 修改成main, 这一步在后面我们编译系统内核时可以不用，现在这么做，主要是想编译成linux可执行文件 
>7. 在foo.asm末尾，通过语句：%include “bar.asm” 将第五步反汇编的C模块代码引入foo.asm。 
>8. 运行命令编译foo.asm: nasm -f elf32 foo.asm, 执行这一步后，目录上会出现foo.o二进制文件 
>9. 执行命令：gcc -m32 foo.o -o foo. 这一步将foo.o与系统模块连接成可执行文件，编译系统内核时，这一步就不需要。 
>10. 运行结果：./foo, 就可以看到运行结果了。

[tyler_download 反汇编](https://blog.csdn.net/tyler_download/article/details/52468520) 

***
### 硬件端口读写操作

c语言无法直接读写硬件，所以对端口的操作必须通过汇编完成。在汇编中有指令in/out可以读写端口：


```
io_in32:
  mov edx, [esp + 4]
  in  eax, dx
  ret

io_out32:
    mov edx, [esp + 4]
    mov eax, [esp + 8]
    out dx, eax
    ret

io_cli:
  CLI
  RET
  
io_load_eflags:
    pushfd
    pop  eax
    ret

io_store_eflags:
    mov eax, [esp + 4]
    push eax
    popfd
    ret
```

上面的代码提供了可以供c语言调用的函数接口，分别为：

* int io_in32(int port);
* void io_out32(int port, int data);
* void io_cli(void);
* int io_load_eflags(void);
* void io_store_eflags(int eflags);

***
### 从“org 0x7c00”到编址


在写boot程序时，我们都需要声明：org 07c00h，为什么是它？

计算机的启动过程中，首先运行bios系统，它会查找可启动硬件（根据第一个扇区的末尾字节是否为055aah来判定。），找到之后会将该扇区载入到内存中的07c00h处。所以对于boot代码来说，不管你怎么写这个值，bios都会把它载入到7c00去，所以必须声明，让汇编器和操作系统之间保持同步，正确设置目标文件中代码和数据的地址。

阮一峰有一个考古文，说了0x7c00的历史渊源，很有意思：

>当时，搭配的操作系统是86-DOS。这个操作系统需要的内存最少是32KB。我们知道，内存地址从0x0000开始编号，32KB的内存就是0x0000～0x7FFF。8088芯片本身需要占用0x0000～0x03FF，用来保存各种中断处理程序的储存位置。（主引导记录本身就是中断信号INT 19h的处理程序。）所以，内存只剩下0x0400～0x7FFF可以使用。为了把尽量多的连续内存留给操作系统，主引导记录就被放到了内存地址的尾部。由于一个扇区是512字节，主引导记录本身也会产生数据，需要另外留出512字节保存。所以，它的预留位置就变成了：
  0x7FFF - 512 - 512 + 1 = 0x7C00 
0x7C00就是这样来的。

***
### 结构体

文件pm.inc:

``` 
%macro Descriptor 3
    dw    %2  &  0FFFFh
    dw    %1  &  0FFFFh
    db   (%1>>16) & 0FFh
    dw   ((%2 >> 8) & 0F00h) | (%3 & 0F0FFh)
    db   (%1 >> 24) & 0FFh
%endmacro

DA_32       EQU 4000h   ; 32 位段
DA_C        EQU 98h ; 存在的只执行代码段属性值
DA_DRW      EQU 92h ; 存在的可读写数据段属性值
```

结构体
* 格式为 %macro Descriptor 3，%macro类似于#define；Descriptor为结构体名字；3为参数个数。
* %1表示第一个参数，以此类推。
* db/dw/dd分别为在内存里写1/2/4个字节。
* EQU 表示 定值，类似于const。


***
### Hello, world（AT&T vs Intel）

AT&T格式：

```
#hello.s
.data                    # 数据段声明
        msg : .string "Hello, world!\\n" # 要输出的字符串
        len = . - msg                   # 字串长度
.text                    # 代码段声明
.global _start           # 指定入口函数
         
_start:                  # 在屏幕上显示一个字符串
        movl $len, %edx  # 参数三：字符串长度
        movl $msg, %ecx  # 参数二：要显示的字符串
        movl $1, %ebx    # 参数一：文件描述符(stdout)
        movl $4, %eax    # 系统调用号(sys_write)
        int  $0x80       # 调用内核功能
         
                         # 退出程序
        movl $0,%ebx     # 参数一：退出代码
        movl $1,%eax     # 系统调用号(sys_exit)
        int  $0x80       # 调用内核功能

```

Intel格式：

```
; hello.asm
section .data            ; 数据段声明
        msg db "Hello, world!", 0xA     ; 要输出的字符串
        len equ $ - msg                 ; 字串长度
section .text            ; 代码段声明
global _start            ; 指定入口函数
_start:                  ; 在屏幕上显示一个字符串
        mov edx, len     ; 参数三：字符串长度
        mov ecx, msg     ; 参数二：要显示的字符串
        mov ebx, 1       ; 参数一：文件描述符(stdout)
        mov eax, 4       ; 系统调用号(sys_write)
        int 0x80         ; 调用内核功能
                         ; 退出程序
        mov ebx, 0       ; 参数一：退出代码
        mov eax, 1       ; 系统调用号(sys_exit)
        int 0x80         ; 调用内核功能
```

***
### 汇编工具：汇编器/链接器

1.汇编器：GAS/NASM

常见的用于Linux平台的有GAS和Nasm。其中GCC配套的是GAS，面向可读性很差的AT&T语法（现在似乎加了扩展），Nasm则是面向Intel语法。典型指令：
* as -o hello.o hello.s
* nasm -f elf hello.asm

2.链接器:LD
* 典型指令：ld -s -o hello hello.o bar.o
    
3.调试器：GDB/ALD
使用GDB进行调试时，和调试c程序基本一样，设置断点，单步执行，查看寄存器、变量的值等等。注意在链接的时候不要加入-s参数，否则会将符号表（symbol table）删除，不利于调试。比如：
* as --gstabs -o hello.o hello.s
* ld -o hello hello.o

***
### 内联汇编

基本格式：

```
__asm__ ( "movl %eax, %ebx\n\t"
                 "movl $56, %esi\n\t"
                 "movl %ecx, $label(%edx,%ebx,$4)\n\t"
                 "movb %ah, (%ebx)");
```

扩展格式：

```
asm ( assembler template
        : output operands                /* optional */
        : input operands                   /* optional */
        : list of clobbered registers   /* optional */
);
```

实例1：

```
int a=10, b;
asm ( "movl %1, %%eax;
           movl %%eax, %0;"
          :"=r"(b)           /* output */
          :"r"(a)              /* input */
          :"%eax"         /* clobbered register */
);
```

说明（来源网上文章）：

* “b”是输出操作数，用%0来访问，”a”是输入操作数，用%1来访问。
* “r” 是一个constraint, 关于constraint后面有详细的介绍。这里我们只要记住这里”r”的意思就是让GCC自己去选择一个寄存器去存储变量a。
* 输出部分constraint前必须要有个 ”=”修饰，用来说明是一个这是一个输出操作数，并且是只写(write only)的。你可能已经注意到，有的寄存器名字前面用了”％%”，这是用来让GCC区分操作数和寄存器的：操作数已经用了一个%作为前缀，寄存器只能用“%%”做前缀了。
* 第三个冒号后面的clobbered register部分有个%eax，意思是内联汇编代码中会改变寄存器eax的内容，如此一来GCC在调用内联汇编前就不会依赖保存在寄存器eax中的内容了。[3]


Volatile：

* 如果我们要求汇编代码必须在被放置的位置执行(例如不能被循环优化而移出循环)，我们就要在asm之后的“()”前，放一个volatile关键字。 这样可以禁止这些代码被移动或删除，像这样声明:
* asm volatile ( ... : ... : ... : ...);
* 同样，如果担心volatile有变量冲突，可以使用__volatile__关键字。
* 如果汇编代码只是做一些运算而没有什么附加影响的时候最好不要使用volatile修饰。不用volatile能给GCC留下优化代码的空间。[3]

***
### 实例：段机制解析

```
主要汇编代码如下：
%include "pm.inc"

org   0x9000

jmp   LABEL_BEGIN

[SECTION .gdt]
 ;                                  段基址          段界限                属性
LABEL_GDT:          Descriptor        0,            0,                   0  
LABEL_DESC_CODE32:  Descriptor        0,      SegCode32Len - 1,       DA_C + DA_32
LABEL_DESC_VIDEO:   Descriptor     0B8000h,         0ffffh,            DA_DRW

GdtLen     equ    $ - LABEL_GDT
GdtPtr     dw     GdtLen - 1
           dd     0

SelectorCode32    equ   LABEL_DESC_CODE32 -  LABEL_GDT
SelectorVideo     equ   LABEL_DESC_VIDEO  -  LABEL_GDT

[SECTION  .s16]
[BITS  16]
LABEL_BEGIN:
     mov   ax, cs
     mov   ds, ax
     mov   es, ax
     mov   ss, ax
     mov   sp, 0100h

     xor   eax, eax
     mov   ax,  cs
     shl   eax, 4
     add   eax, LABEL_SEG_CODE32
     mov   word [LABEL_DESC_CODE32 + 2], ax
     shr   eax, 16
     mov   byte [LABEL_DESC_CODE32 + 4], al
     mov   byte [LABEL_DESC_CODE32 + 7], ah

     xor   eax, eax
     mov   ax, ds
     shl   eax, 4
     add   eax,  LABEL_GDT
     mov   dword  [GdtPtr + 2], eax

     lgdt  [GdtPtr]

     cli   ;关中断

     in    al,  92h
     or    al,  00000010b
     out   92h, al

     mov   eax, cr0
     or    eax , 1
     mov   cr0, eax

     jmp   dword  SelectorCode32: 0

     [SECTION .s32]
     [BITS  32]
LABEL_SEG_CODE32:
    mov   ax, SelectorVideo
    mov   gs, ax
    mov   si, msg
    mov   ebx, 10
    mov   ecx, 2
showChar:
    mov   edi, (80*11)
    add   edi, ebx
    mov   eax, edi
    mul   ecx
    mov   edi, eax
    mov   ah, 0ch
    mov   al, [si]
    cmp   al, 0
    je    end
    add   ebx,1
    add   si, 1
    mov   [gs:edi], ax
    jmp    showChar
end: 
    jmp   $
    msg:
    DB     "Protect Mode", 0

SegCode32Len   equ  $ - LABEL_SEG_CODE32
```

文件pm.inc:
``` 
%macro Descriptor 3
    dw    %2  &  0FFFFh
    dw    %1  &  0FFFFh
    db   (%1>>16) & 0FFh
    dw   ((%2 >> 8) & 0F00h) | (%3 & 0F0FFh)
    db   (%1 >> 24) & 0FFh
%endmacro

DA_32       EQU 4000h   ; 32 位段
DA_C        EQU 98h ; 存在的只执行代码段属性值
DA_DRW      EQU 92h ; 存在的可读写数据段属性值
```

gdt：
* 全局描述符表。里面包含了段描述符，每一个描述符占用八字节。
* 描述符的规则如上段代码所示，传入段基址、段界限、属性，分别写入2347、016、56几个字节中。
* gdtlen是gdt的总长度（必须在编译完之后才知道），决定于有多少个描述符。在本例中有3个描述符（第一个为空描述符），一共24byte（$表示当前行代码地址）。gdtptr分两部分，前二字节为gdtlen-1，后四字节为一个地址。应该是给bios系统用的。

程序流程为：
* 入口初始化；
* 获得程序段LABEL_SEG_CODE32 的真实物理地址（用实模式的计算方式），并根据保护模式规则写入gdt描述符LABEL_DESC_CODE32中。
* 获得gdt的真实物理地址，写入gtdptr结构中。
* lgdt，关闭中断，一顿操作，进入保护模式。
* 进入保护模式后，根据新的寻址模式，跳转到程序段。  

***

### 参考文章

[阮一峰 《汇编语言入门教程》](http://www.ruanyifeng.com/blog/2018/01/assembly-language-primer.html)
[IBM开发社区 《Linux汇编语言开发指南》](https://www.ibm.com/developerworks/cn/linux/l-assembly/index.html)
[简书《GCC内联汇编基础》](https://www.jianshu.com/p/1782e14a0766)

