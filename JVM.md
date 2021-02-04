[TOC]

# JVM

## 一. 内存与垃圾回收

### 1.JVM与Java体系结构

- Java API是基于JVM
- JVM的作用：项目管理，调优的需要
- Java内存动态分配，垃圾收集技术
- JVM字节码
- 虚拟机，程序虚拟机的典型代表是Java虚拟机，专门为执行单个计算机程序而设计
- Java虚拟机就是二进制字节码的运行环境
  - 一次编译，自动内存管理，自动垃圾回收
  - 硬件-》操作系统-》JVM-》字节码文件-》用户
- HotSpot VM 是高性能的虚拟机代表
- 运行时数据区（Runtime Data Area）:方法区，堆，Java栈，本地方法栈，程序计数器
- 字节码  ***.class
- Java编译器的指令流逝一种基于栈的指令集架构（大部分是零地址指令），另一种是基于寄存器的指令集架构（Android）
- Java虚拟机的启动是引导类加载器加载的
- 默认虚拟机是Oracle HotSpot VM虚拟机，HotSpot热点代码探测技术

### 2.类加载子系统

- Class Loader SubSystem
- class文件有特定标识
- 虚拟机必须保证同一个类的方法在多线程下被同步加锁
- 双亲委派机制
  - 如果一个类加载器收到类加载请求，它并不会自己先去加载，而是将这个委托给父类的加载器去执行；如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归，请求最终将达到顶层的启动类加载器
  - 优点：避免类的重复加载；保护程序安全，防止核心API被随意篡改（自定义类：java.lang.String或者java.lang.ShkStart）；
- 沙箱安全机制：自定义String类，加载String类时优先使用引导类加载器（ClassLoader）加载，保证对源代码的保护
- 对类的使用方式：主动和被动（会被自动初始化）

### 3.运行时数据区概述以及线程

- 进程一份：方法区、堆（优化）
- 线程一份：程序计数器、本地方法栈、虚拟机栈
- 线程：是一个程序运行的单元，Hotspot JVM中的线程与操作系统的本地线程直接映射
- 虚拟机线程、周期任务线程、GC线程、编译线程（字节码-》本地代码）、信号调度线程

### 4.程序计数器

- JVM中的PC寄存器是对物理PC内寄存器的一种抽象模拟，PC寄存器是用来存储指向下一条指令的地址，也就是即将执行的指令代码。

- PC寄存器存储字节码指令地址有什么用？或为什么用PC寄存器记录当前线程的执行地址？

  - 因为CPU需要不断切换各个线程，这时候切换回来就知道从哪里开始继续

  - JVM的字节码解释器就需要通过改变PC寄存器的值来明确下一条应该执行什么样的字节码指令

- PC寄存器为什么是线程私有

  - 为了能够准确记录各个线程正在执行的当前字节码指令地址，最好的办法自然就是为每个线程分配一个PC寄存器，这样各个线程之间可以进行独立计算，不会出现相互干扰的情况，由于CPU时间片轮限制，众多线程在并发执行过程中，任何一个确定的时刻，一个处理器或者多核处理器的一个内核，只会执行某个线程中的一个指令。
  - 并行和并发

### 5.虚拟机栈

- Java指令基于栈来设计。
  - 优点：跨平台，指令集小，编译器容易实现
  - 缺点：性能下降，实现同样的功能需要更多指令
- 栈是运行时单位，堆是存储的单位
- Java虚拟机栈是线程私有的
- 一个栈帧对应一个方法
- 生命周期和线程一致
- 作用：主管Java程序的运行，保存方法的局部变量、部分结果，并参与方法的调用和返回
- JVM直接对Java栈的操作只有出栈和进栈，对栈来说没有垃圾回收问题（无GC, 有OOM）
- 开发中可能遇到的异常：（由于Java栈的大小是动态的或者固定不变的）
  - 对于固定不变的栈，若线程分配的栈容量超过了Java虚拟机栈允许的最大容量，则Java虚拟机会抛出一个StackOverflowError异常
  - 对于动态扩展的栈，若扩展时无法申请到足够的内存，或者在创建新线程的时候没有足够的内存去创建对应的Java虚拟机栈，则会抛出OutOfMemoryError异常
- 栈的存储单位：栈帧 Stack Frame
- 栈帧和方法一一对应，一个栈帧的结束意味着一个方法的结束
- 栈帧是一个内存区块，是一个数据集
- 不同线程所包含的栈帧是不允许存在相互引用的，因为若当前方法调用了其他方法，则方法返回之际，当前栈帧会传回此方法的执行结果给前一个栈帧，虚拟机会丢弃当前栈帧，使得前一个栈帧重新成为当前栈帧
- Java方法返回函数有两种方式：1.正常函数return返回 2.抛出异常；两个方法都会使栈帧被弹出
- 栈帧里面有：局部变量表（数字数组，大小编译时确定），操作数栈，动态链接（指向运行时常量池的方法引用），方法返回地址（方法正常退出或异常退出）
  - 局部变量表的基本单元：Slot是变量槽
  - 若当前帧是由构造方法或者实例方法创建，该对象引用this将会存放在index为0的slot处
  - 静态变量和局部变量的对比：
  - 基本数据类型《-》引用数据类型，成员变量《-》局部变量
  - 局部变量表中的变量也是重要的垃圾回收根节点，只要被局部变量表中直接或间接引用的对象不会被回收
  - 操作数栈：可以使用数组和链表来实现，push，pop
  - 

### 6.本地方法接口

### 7.本地方法栈

### 8.堆

### 9.方法区

### 10.直接内存

### 11.执行引擎

### 12.StringTable

### 13.垃圾回收概述

### 14.垃圾回收相关算法

### 15.垃圾回收相关概念

### 16.垃圾回收器

## 二. 字节码与类的加载

## 三. 性能监控与调优

## 四. 大厂面试
