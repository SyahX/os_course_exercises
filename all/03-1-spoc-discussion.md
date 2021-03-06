# lec5: SPOC思考题

##**提前准备**
（请在上课前完成）

- 完成lec５的视频学习和提交对应的在线练习
- git pull ucore_os_lab, v9_cpu, os_course_spoc_exercises in github repos。这样可以在本机上完成课堂练习。
- 理解连续内存动态分配算法的实现（主要自学和网上查找）

NOTICE
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。


## 思考题
---

## “连续内存分配”与视频相关的课堂练习

### 5.1 计算机体系结构和内存层次

1.操作系统中存储管理的目标是什么？
+ 抽象
+ 保护
+ 共享
+ 虚拟化

### 5.2 地址空间和地址生成
1.描述编译、汇编、链接和加载的过程是什么？
+ 编译：将程序源代码转换为汇编代码
+ 汇编：将汇编代码转为二进制的机器码
+ 链接：将多个二进制的机器码结合成一个可执行环境
+ 加载：将程序从外存中加载到内存中

2.动态链接如何使用？尝试在Linux平台上使用LD_DEBUG查看一个C语言Hello world的启动流程。  (optional)


### 5.3 连续内存分配
1.什么是内碎片、外碎片？
+ 外碎片：两个进程之间的空余的无法利用空间
+ 内碎片：每个进程内部分配的多余的空间

2.最先匹配会越用越慢吗？请说明理由（可来源于猜想或具体的实验）？
+ 会越来越慢，因为最先匹配总是先找低地址空间的内存，所以到后期需要申请大内存时，需要从前往后扫描大量的小的不连续的空间

3.最差匹配的外碎片会比最优适配算法少吗？请说明理由（可来源于猜想或具体的实验）？
+ 猜测可能会的，因为每次找最大的，而最大的分配之后往往还能剩余一个较大仍可以继续使用的空间。

4.理解0:最优匹配，1:最差匹配，2:最先匹配，3:buddy systemm算法中分区释放后的合并处理过程？ (optional)


### 5.4 碎片整理
1.对换和紧凑都是碎片整理技术，它们的主要区别是什么？为什么在早期的操作系统中采用对换技术？  
+ 主要区别：紧凑是在内存中搬动进程占用的内存位置，以合并出大块的空闲块；对换是把内存中的进程搬到外存中，以空出更多的内存空闲块。
+ 原因在于早期的操作系统的内存空间都很小，相比较空间价值，时间开销是可以接受的。

2.一个处于等待状态的进程被对换到外存（对换等待状态）后，等待事件出现了。操作系统需要如何响应？
+ 将进程从硬盘中读取到内存中

### 5.5 伙伴系统
1.伙伴系统的空闲块如何组织？
+ 将空闲块分解成二的次幂，且只分配二的次幂的大小的内存空间

2.伙伴系统的内存分配流程？伙伴系统的内存回收流程？
+ 分配流程：找到最小比待分配空间S大的内存块，设其大小为M，若M > 2 * S则将M不断二分，直到新的S < M < 2S，并将M分配给S
+ 回收流程：回收S之后，判断是否可以和相邻的等大的空闲空间合并（需要满足一定的合并条件：两个等大的空间必须是二叉树上的两个兄弟节点）

## 课堂实践

观察最先匹配、最佳匹配和最差匹配这几种动态分区分配算法的工作过程，并选择一个例子进行分析分析整个工作过程中的分配和释放操作对维护数据结构的影响和原因。

  * [算法演示脚本的使用说明](https://github.com/chyyuu/os_tutorial_lab/blob/master/ostep/ostep3-malloc.md)
  * [算法演示脚本](https://github.com/chyyuu/os_tutorial_lab/blob/master/ostep/ostep3-malloc.py)

例如：
```
python ./ostep3-malloc.py -S 100 -b 1000 -H 4 -a 4 -l ADDRSORT -p BEST -n 5 -c
python ./ostep3-malloc.py -S 100 -b 1000 -H 4 -a 4 -l ADDRSORT -p FIRST -n 5 -c
python ./ostep3-malloc.py -S 100 -b 1000 -H 4 -a 4 -l ADDRSORT -p WORST -n 5 -c
```

### 扩展思考题 (optional)

1. 请参考xv6（umalloc.c），ucore lab2代码，选择四种（0:最优匹配，1:最差匹配，2:最先匹配，3:buddy systemm）分配算法中的一种或多种，在Linux应用程序/库层面，用C、C++或python来实现malloc/free，给出你的设计思路，并给出可以在Linux上运行的malloc/free实现和测试用例。


2. 阅读[slab分配算法](http://en.wikipedia.org/wiki/Slab_allocation)，尝试在应用程序中实现slab分配算法，给出设计方案和测试用例。
