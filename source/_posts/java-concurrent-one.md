---
title: java并发-java内存模型、底层实现原理
date: 2018-09-03 21:33:54
tags:
---
## java内存模型

### 1. 内存模型

JMM定义了线程和主内存之间的抽象关系：
线程之间的共享变量存储在主内存中，每一个线程都有一个私有的本地内存，
本地内存存储了该线程用来读/写共享变量的副本。
JMM通过控制主内存与每个线程的本地内存之间的交互，来为java程序提供内存可见性的保证。

<!-- more -->

### 2. volatile

#### &emsp;&emsp;特性

* 可见性。对一个volatile变量的读，总是能看到对这个volatile变量最后的写入。
* 原子性。对于单个volatile变量的读写具有原子性，但类似于volatile++的复合操作不具有原子性。
_《java高并发编程详解》把“禁止重排序（内存屏障）”也当做是volatile的语义，《java并发编程的艺术》把“禁止重排序（内存屏障）”当做是volatile的语义的实现_

#### &emsp;&emsp;写-读的内存语义

* 当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值刷新到主内存。
* 当读一个volatile变量时，JMM会把该行程对应的本地内存的值置为无效，线程接下来将从主内存中读取共享变量。

#### &emsp;&emsp;内存语义实现原理

为了实现volatile的内存语义，编译器在生成字节码时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序。
下面是基于保守策略的JMM内存屏障插入策略：

* 在每一个volatile写操作的前面插入一个StoreStore屏障。
* 在每一个volatile写操作的后面插入一个StoreLoad屏障。
* 在每一个volatile读操作的后面插入一个LoadLoad屏障。
* 在每一个volatile读操作的后面插入一个LoadStore屏障。

### 3. synchronized

#### &emsp;&emsp;特性

锁可以让临界区互斥执行。

#### &emsp;&emsp;释放-获取的内存语义

* 当线程释放锁时，JMM会把该线程对应的本地内存中的共享变量刷新到主内存中。
* 当线程获取锁时，JMM会把该线程对应的本地内存置为无效。从而使线程中的代码必须从主内存中读取共享变量。 

#### &emsp;&emsp;内存语义实现原理

1. 利用volatile变量的读-写所具有的的内存语义。
2. 利用CAS所附带的volatile读和volatile写的内存语义。

### 4. final

1. 在构造函数内对一个final变量的赋值（写），与随后把“这个被构造的对象的引用”赋值给“一个引用
变量”，这两个操作之间不能重排序。
2. 初次读一个包含“final变量的对象”的引用，与随后初次读“这个final变量”，这两个操作之间不能
重排序。

#### &emsp;&emsp;注意点：
_final只能在两种情况下初始化：1，定义时；2，构造函数内。_
_final引用不能从构造函数内“溢出”_

#### &emsp;&emsp;作用
1. 禁止继承重写，防止破坏类内部的线程安全性。
2. 禁止改变变量的值，保证在多线程环境下的数据安全。


### 5. JMM的三个特性

1. 原子性

一次操作或多次操作。要么都执行成功，要么都不执行。
synchronized、各种 Lock 以及各种原子类实现原子性

2. 可见性

一个线程对共享变量进行了修改，其他线程也可以立刻看到修改后的值。
synchronized、volatile 以及各种 Lock 实现可见性

3. 有序性

由于指令重排序的问题，代码的执行顺序未必是代码编写时的顺序。
volatile 关键字可以禁止指令进行重排序优化



## 并发底层实现原理

### synchronized

#### &emsp;&emsp;对象头：

1. Mark Word：主要存储对象的hashCode、分代年龄、锁标志位

2. 类的元数据指针

3. 数组长度

#### &emsp;&emsp;锁的升级：

1. 无锁：标志位是`001`。

2. 偏向锁：标志位是`101`；存在于的情况是：锁不存在多线程竞争，并且锁总是由同一个线程多次获取。
偏向锁撤销存在于的情况是：调用了锁对象的hashCode()（升级成重量级锁）、wait(timeout)（升级成重量级锁）、notify()（升级成轻量级锁）方法。

3. 轻量级锁：标志位是`00`；存在于的情况是：两个线程交替执行同步代码块。

4. 重量级锁：标志位是`10`;存在于的情况是：多个线程同时获取锁。
