---
title: java并发-并发编程基础
date: 2018-09-13 21:29:13
tags:
---

## java并发编程基础

### 1. 线程简介

线程都拥有各自的计数器、堆栈和局部变量等属性，并且能够访问共享的内存变量。

<!-- more -->

#### &emsp;&emsp;线程状态：

NEW：初始状态，线程被构建，但是还没有调用`start()`方法。
RUNNABLE：运行状态，java将操作系统中的“就绪”和“运行”两种状态统称为“运行中”。此时处于`Thread.start()`方法调用之后。
处于`WAITING`，`TIME_WAITING`状态的线程在调用`Object.notify()`，`Object.notifyAll()`，`LockSupport.unpark(Thread)`方法之后，也会进入RUNNABLE状态。
BLOCKED：阻塞状态，表示线程阻塞于锁。此时线程等待进入`synchronized`代码块或方法。获取到锁后进入`RUNNABLE`状态。
WAITING：等待状态，进入该状态表示当前线程需要其他线程做出一些特定的动作（通知或中断），此时处于`Object.wait()`，`Object.join()`或`LockSupport.park()`方法调用之后。
TIME_WAITING：超时等待状态，它与`WAITING`不同，它可以在指定的时间自行返回。此时处于`Thread.sleep(long)`，`Object.wait(long)`，
`Thread.join(long)`，`LockSupport.parkNanos()`，`LockSupport.parkUntil()`方法调用之后。
TERMINATED：终止状态，表示线程已经执行完毕。
![](java-concurrent-two/thread-status.png)

### 2. 启动和终止线程

##### &emsp;&emsp;理解中断

1. 中断好比其他线程对该线程打了个招呼，其他线程通过调用该线程的interrupt()
方法对其进行中断操作（设置了`中断标识位`）。
2. 许多声明抛出InterruptedException的方法（例如Thread.sleep(long millis)方法）这些方法在抛出InterruptedException之前，
JVM会先将该线程的`中断标识位`清除，然后抛出InterruptedException，此时调用线程对象的isInterrupted()方法依旧会返回false。

> 如何安全地中断运行中的线程
> 虽然java里面提供了一个stop()方法，可以强行终止线程，但是这个方式是不安全的，造成在运行的线程产生不正确的结果。
> 想要安全的中断线程，只能在线程内部埋下一个钩子，线程外部通过这个钩子触发线程的中断命令。
> java提供的一个interrupt()方法，这个方法要配合isInterrupted()方法来使用。

### 3. 线程间通信

#### &emsp;&emsp;等待通知范式

* 等待方遵循的原则：

1. “线程”获取对象的锁。wait的是“线程”对象。
2. 如果条件不满足则调用对象的wait()方法，被通知后仍要检查条件。
3. 条件满足则执行对应的逻辑。
代码：
```
synchronized(对象) {
	while(条件不满足) {
		对象.wait();
	}
	处理对应的逻辑
}
```
> 当“对象锁”是Thread对象时，此时wait的是获取“对象锁”的线程，而不是Thread对象。

* 通知方遵循的原则：
1. “线程”获取对象的锁。
2. 改变条件。
3. 通知所有等待在对象上的线程。
代码：
```
synchronized(对象){
	改变条件
	对象.notifyAll();
}
```

> 当`main线程`调用t.join时候，`main线程`会获得`线程对象t`的锁（wait意味着拿到该对象的锁)，调用该对象的wait(等待时间)，直到该对象唤醒`main线程`，比如退出后。
> 这就意味着`main线程`调用t.join时，必须能够拿到线程t对象的锁。

#### Thread.join的原理解释：
```
//场景：
main() {
	thread.join();
}
```

```
//解释：
//此时的“对象锁”是thread对象；相当于范式中的“synchronized(对象)”。
public final synchronized void join() throws InterruptedException {
	// 条件不满足，继续等待
	while (isAlive()) {
		//thread对象调用自身的wait方法。
		//相当于范式中“对象.wait()”。所以此时等待的是“main线程对象”。
		wait(0);
	}
	// 条件符合，方法返回
}
```

### 4. 线程应用实例

_待完成_