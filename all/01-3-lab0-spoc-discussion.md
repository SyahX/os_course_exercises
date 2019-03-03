# lec2：lab0 SPOC思考题

## **提前准备**
（请在上课前完成，option）

- 完成lec2的视频学习
- git pull ucore_os_lab, os_tutorial_lab, os_course_exercises  in github repos。这样可以在本机上完成课堂练习。
- 了解代码段，数据段，执行文件，执行文件格式，堆，栈，控制流，函数调用,函数参数传递，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在不同操作系统（如linux, ucore,etc.)与不同硬件（如 x86, riscv, v9-cpu,etc.)中是如何相互配合来体现的。
- 安装好ucore实验环境，能够编译运行ucore labs中的源码。
- 会使用linux中的shell命令:objdump，nm，file, strace，gdb等，了解这些命令的用途。
- 会编译，运行，使用v9-cpu的dis,xc, xem命令（包括启动参数），阅读v9-cpu中的v9\-computer.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
- 了解基于v9-cpu的执行文件的格式和内容，以及它是如何加载到v9-cpu的内存中的。
- 在piazza上就学习中不理解问题进行提问。

---

## 思考题

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？

进程的切换需要硬件支持时钟中断；虚存管理需要地址映射机制，从而需要MMU等硬件；对于文件系统，需要硬件有稳定的存储介质来保证操作系统的持久性。 对应的，应当提供中断使能，触发软中断等中断相关的，设置内存寻址模式，设置页表等内存管理相关的，执行I/O操作等文件系统相关的特权指令。

对于现代操作系统（每个进程占一个时间片）时钟中断是非常需要的。存储介质当然也是非常必要的。当然，事实上，MMU没有也行，可以用用户态函数库来实现地址转换，但这样可能就保证不了安全性了。

Intel手册第3卷2.8节“System Instruction Summary”中给出了一个系统指令列表。“系统指令完成的是系统级的功能，包括加载系统寄存器、管理Cache、管理中断和设置调试寄存器。其中的大部分指令都必须由操作系统或特权级为0的进程执行；另一部分可以由任何特权级的进程执行。”

- 你理解的x86的实模式和保护模式有什么区别？物理地址、线性地址、逻辑地址的含义分别是什么？

保护模式和实模式的根本区别是进程内存是否受保护。

物理地址：是处理器提交到总线上用于访问计算机系统中的内存和外设的最终地址。
逻辑地址：在有地址变换功能的计算机中，访问指令给出的地址叫逻辑地址。
线性地址：线性地址是逻辑地址到物理地址变换之间的中间层，是处理器通过段机制控制下的形成的地址空间。

- 你理解的risc-v的特权模式有什么区别？不同 模式在地址访问方面有何特征？

RISC-V定义了四种特权级模式。不同的特权级包含多个 CSR（control and status register，控制和状态寄存器）寄存器。

- 理解ucore中list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）

- 对于如下的代码段，请说明":"后面的数字是什么含义
```
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
 };
```

表示所占用的bit位数

- 对于如下的代码段，

```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```
如果在其他代码段中有如下语句，
```
unsigned intr;
intr=8;
SETGATE(intr, 1,2,3,0);
```
请问执行上述指令后， intr的值是多少？

0x20003

### 课堂实践练习

#### 练习一

1. 请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。
```
.text
.globl kernel_thread_entry
kernel_thread_entry:        # void kernel_thread(void)
    pushl %edx              # push arg
    call *%ebx              # call fn
    pushl %eax              # save the return value of fn(arg)
    call do_exit            # call do_exit to terminate current thread
```
通过参数调用函数

2. (option)请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。

+ 利用宏进行复杂数据结构中的数据访问；
+ 利用宏进行数据类型转换；如 to_struct
+ 常用功能的代码片段优化；如 ROUNDDOWN, SetPageDirty

#### reference
 - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
 - [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
 - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
 - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
 - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)
