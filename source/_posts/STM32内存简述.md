---
title: STM32内存简述
date: 2024-04-03 16:36:59
tags:
---

# STM32内存简述（Flash和SRAM）

## 1. STM32内存简述
### 1.1. STM32寻址范围
1. STM32是一个32位的单片机，因此，它有32根地址线，每个地址线有两种状态：导通 或 不导通。

2. 单片机内存的地址访问存储单元是按照字节编址的。

    按照字节编址，也就是说，访问一个地址上存储的数据，得到的是一个字节的数据。

根据上述两条，可以得出结论：

1. STM32的寻址（内存）大小为：2^32（字节） = 4G（字节）

2. STM32的寻址范围为：0X0000 0000 ~ 0XFFFF FFFF

## 1.2. 存储器功能划分
上述存储地址被分为了8个块（目前我用到的芯片F1、F4、L4的都是，其他的没用过不确定）。
如下表所示：

![](1.png)


比如，这里打开STM32L475的芯片手册上可以看到，从0X0000 0000 ~ 0XFFFF FFFF被分成了八份，也就是八个块。

![](./a836f07702ee116a356dea6bf8a17669.png)


# 2. SRAM、ROM位置
简单说：

- RAM：读写速度快、掉电数据丢失；类比于电脑内存的作用。

- ROM：读写速度相对慢、掉电数据仍在；类比于电脑硬盘的作用。

其中对于STM32而言，SRAM就是内存；Flash就是硬盘。
![](./3)


以最最最常见的STM32F103C8T6为例：
![](./4)
它有64K字节的Flash、20K字节的RAM。

使用CubeMX对该芯片创建一个工程，然后点击魔术棒可以看到：

1. 它的内置ROM起始地址为`0X0800 0000`，大小为`0X10000`，也就是FLASH的大小为：0X10000个字节 = 65536个字节 = 64k个字节 = 64kBytes。

2. 它的内置RAM起始地址为`0X2000 0000`，大小为`0X5000`，也就是RAM的大小为：0X5000个字节 = 20480个字节 = 20k个字节 = 20kBytes。
![](./5)


对比一下芯片的数据手册：可以看到，其中FLASH位于第一块（索引0）的中间区域，RAM位于第二块（索引1）的开头。
![](./6)


> 如果在KEIL软件中使用STLINK下载代码，就会发现代码是从0X0800 0000处开始保存的。

但是对于部分芯片来讲，其不止有一块RAM，比如**STM32L475VET6**，它有512K字节的Flash、128K字节的RAM。

![](./7)


同样的使用CubeMX对该芯片创建一个工程，然后点击魔术棒：

它的内置ROM起始地址为`0X0800 0000`，大小为`0X80000`，也就是FLASH的大小为：0X80000个字节 = 524288个字节 = 512k个字节 = 512kBytes。

它的内置RAM分为两块，第一块起始地址为`0X2000 0000`，大小为`0X18000`；第二块起始地址为`0X1000 000`，大小为`0X8000`，也就是RAM的大小为：0X18000+0X8000个字节 = 0X20000个字节 = 131072个字节 = 128k个字节 = 128kBytes。
![](./8)
同样对比一下芯片的数据手册：可以看到，其中FLASH位于第一块（索引0）的中间区域，RAM1位于第二块（索引1）的开头，RAM2位于第一块（索引0）的中间区域。
![](./9)



通过对比两个芯片的内存分布，可以看到，不同芯片的分布是有点差别的。

> 了解这部分的内容主要是因为：在使用内存池的时候，要知道内存池到底是怎么划分的，可以划分几个内存池，每个内存池的大小最大可以划分多大，都要参考这部分的内容。
> 
>  
> 另外关注一下：地址0x1FFFF 0000到0x1FFF 7000这部分，这部分是系统存储器，不需要我们用户来操作，但是要知道这部分出厂的时候就已经存了有东西的，后面看到Bootloader的时候会说到。



## 3. 程序占用内存大小
### 3.1. 查看程序大小
使用 CubeMX 创建一个 stm32 工程，创建完成后什么也不写，直接使用 Keil 对编写的程序进行编译，编译完成之后，软件最下方会提示编写的程序所占空间的大小。
![](./10)

也可以在工程目录中保存编译结果的文件夹中找到.map文件，在文件最下方查看更详细的信息。

![](./11)

### 3.2. 占用内存分析
数据类型解释：

- Code：代码，也就是编译之后产生的机器指令。

- RO_data：Read Only data，只读数据域，指程序中用到的只读数据，这些数据被存储在ROM区，因而程序不能修改其内容。C语言中const关键字定义的变量就是典型的RO-data。这部分在程序运行过程中不能被更改，因此在运行时只需要来读取即可，无需占用 RAM 空间。

- RW_data：Read Write data，可读写数据域，指初始化为“非0值”的可读写数据，程序刚运行时，这些数据具有非0的初始值，且运行的时候它们会常驻在RAM区，因而应用程序可以修改其内容。C语言中定义的全局变量，且定义时赋予“非0值”给该变量进行初始化。

- ZI_data：Zero Initialie data，即0初始化数据，它指初始化为“0值”的可读写数据域。它与RW-data的区别是程序刚运行时这些数据初始值全都为 0，而后续运行过程与RW-data的性质一样，它们也常驻在RAM区，因而应用程序可以更改其内容。例如C语言中使用定义的全局变量，且定义时赋予“ 0 值”给该变量进行初始化(若定义该变量时没有赋予初始值，编译器会把它当ZI-data来对待，初始化为 0)；



再补充一个

- ZI-data 的栈空间(Stack)及堆空间(Heap)：在C语言中，函数内部定义的局部变量属于栈空间，进入函数的时候会向栈空间申请内存给局部变量，退出时释放局部变量，归还内存空间。而使用malloc动态分配的变量属于堆空间。在程序中的栈空间和堆空间都是属于ZI-data区域的，这些空间都会被初始值化为0值。编译器给出的ZI-data占用的空间值中包含了堆栈的大小(经实际测试，若程序中完全没有使用malloc动态申请堆空间，编译器会优化，不把堆空间计算在内)。


上述四种类型的数据占用内存情况如下：

1. 在程序烧录完成之后如左图所示。Code + RO_data + RW_data（RO + RW）三种类型需要占用 Flash 空间。RAM不需要用到。

2. 在程序运行时的情况如右图所示。Flash 的空间未发生变化；对于RAM的空间，程序启动时首先需要把 Flash 中的 RW_data（RW）复制到 RAM 中，然后把 ZI_data 加载到 RAM中。
![](./12)
对应到具体的内存上，结合启动流程如下图所示。

![](./13)

结论，想要让一个程序正常运行。

芯片的 Flash 大小 要大于 Code + RO-data + RW-data 的大小；

芯片的 RAM 大小 要大于 RW-data + ZI_data 的大小。

## 4. text、data、bss
上面分析了程序占用的 ROM 和 RAM 大小。然后分析一下数据或变量的存储位置。
![](./14)

程序编译后的内容包括：代码段（text）、数据段(data)、bss段、堆栈段（head stack） 。
其中：

- text 段：存放代码程序的可执行指令

- data 段：存放已被初始化的 全局变量 和 常量

- bss 段：存放未被初始化的 全局变量



[文章来源](https://blog.csdn.net/weixin_46253745/article/details/130032941)

