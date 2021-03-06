---
title: java并发-java中的锁
date: 2018-09-19 20:32:25
tags:
---

## java中的锁

围绕两个方面：“使用”，“实现”。

<!-- more -->

### 1. Lock接口

Lock接口提供了synchronized所不具备的特性：

特性|描述
-|-
尝试非阻塞地获取锁|当线程尝试获取锁，如果这一时刻锁没有被其他线程获取到，则成功获取到锁。
能被中断的获取锁|与synchronized不同，获取到锁的线程能够响应中断，当获取到锁的线程被中断，中断异常会被抛出，同时锁会被释放。
超时获取锁|在指定的时间内获取锁，如果时间到了仍无法获取锁，则返回。

### 2. 队列同步器（QAS）

AbstractQueuedSynchronizer是用来构建锁或其他同步组件的基础框架，它使用一个int成员变量表示同步状态；通过内置的FIFO队列来完成资源获取线程的排队工作。

#### &emsp;&emsp;使用

自定义同步组件（CustomLock）需要聚合一个AQS的子类（SpecificSynchronizer）。
SpecificSynchronizer可重写的方法：

方法名称|描述
-|-
protected boolean tryAcquire (int arg)|独占式获取同步状态。实现该方法需要查询当前状态并判断同步状态是否符合预期，然后再进行 CAS 设置同步状态。
protected boolean tryRelease (int arg)|独占式释放同步状态。等待获取同步状态的线程将有机会获取同步状态。
protected int tryAcquireShared (int arg)|共享式获取同步状态。返回大于等于0的值，表示获取成功，反之获取失败。
protected boolean tryReleaseShared (int arg)|共享式释放同步状态。
protected boolean isHeldExclusively()|当前同步器是否在独占模式卜被线程占用，一般该方法表示是否被当前线程所独占。

CustomLock可调用的SpecificSynchronizer提供的部分模板方法如下：

方法名称|描述
-|-
void acquire (int arg)|独占式获取同步状态。如果当前线程获取同步状态成功，则由该方法返回；否则，将会进入同步队列等待，该方法将会调用重写的tryAcquire(int arg)方法。
void acquireInterruptibly (int arg)|与acquire (int arg)相同，但是该方法响应中断，当前线程未获取到同步状态而进人同步队列中，如果当前线程被中断，则该方法会抛出InterruptedException并返回。
boolean tryAcquireNanos (int arg, long nanos)|在acquireInterruptibly (int arg)的基础上增加了超时限制，如果当前线程在超时时间内没有获取到同步状态，那么将会返回false，如果获取到了返回true。
void acquireShared(int arg)|共享式的获取同步状态，如果当前线程未获取到同步状态，将会进入同步队列等待，与独占式获取的主要区别是在同一时刻可以有多个线程获取到同步状态。
void acquireSharedInterruptibly(int arg)|与acquireShared(int arg)相同，该方法响应中断。
boolean tryAcquireSharedNanos(int arg, long nanos)|与acquireSharedInterruptibly(int arg)基础上增加了超时等待限制。
boolean release(int arg)|独占式释放同步状态，该方法会在释放同步状态后，将同步队列中第一个节点包含的线程唤醒。
boolean releaseShared(int arg)|共享式的释放同步状态。
Collection<Thread> getQueuedThreads()|获取等待在同步队列上的线程集合。

#### &emsp;&emsp;实现：

##### &emsp;&emsp;独占式获取同步状态的关键点

1. AQS依赖内部的同步队列（一个FIFO的双向队列）来完成同步状态的管理。同步队列中节点（Node）用来保存获取同步状态失败的线程引用、等待状态以及前驱节点和后继节点。
2. 通过调用AQS的acquire(int arg)方法可以获取同步状态。
3. 首先调用SpecificSynchronizer实现的tryAcquire(int arg)，该方法保证线程安全的获取同步状态。
4. 如果获取同步状态失败，通过调用addWaiter(Node node)方法将该节点加入到同步队列的尾部。主要是有一段快速尝试在尾部添加节点的操作。
5. 在enq(final Node node)方法中，主要是通过CAS将节点正确地设置成尾节点。
6. 节点加入同步队列后，调用acquireQueued(final Node node, int arg)方法进入一个自旋的过程。每个节点（或者说每个线程）都在自省的观察；
当条件满足，获取到了同步状态，就会从自旋过程中退出；否则依旧停留在自旋过程中。
7. acquireQueued(final Node node, int arg)的主要操作有两个：
一是判断是否当前节点的前驱节点是head节点，并且成功获取到了同步状态；则将当前节点设置为head节点，并将前驱节点设置为null。最后是返回并退出自旋，继续执行线程任务。
二是根据当前节点的前驱节点的waitStatus是否是SIGNAL来决定是否park当前节点中的线程。这里有一段将前驱节点的waitStatus设置为SIGNAL的操作，所以同步队列中的任意节点的前驱节点的waitStatus为SIGNAL，则任意节点中的线程就会park。
8. release(int arg)的作用是调用tryRelease(int arg)将state设置为0，如果成功则唤醒（unpark）在队列自旋中park的，并且是head节点的后继节点中的线程。

> 参考测试代码：
> https://github.com/legend9207/concurrent_code/blob/master/src/com/roocon/thread/ta4/Main.java

##### &emsp;&emsp;共享式获取同步状态的关键点

_待完成_

### 3. 重入锁

#### &emsp;&emsp;实现：
包含3个要素：
1. 原子状态。原子状态使用CAS操作来存储当前锁的状态，进而判断锁是否被别的线程所持有。
2. 等待队列。所有没有请求到锁的线程，会进入等待队列进行等待；带有线程释放锁后，jvm就能从等待队列中唤醒一个线程，继续工作。
3. 阻塞原语park()和unpark()。用来挂起和恢复线程。

* nonfairTryAcquire增加了再次获取同步状态的逻辑：判断当前线程是否为获取锁的线程；如果是，则将同步状态的值增加并返回true，表示获取同步状态成功。
* tryAcquire增加了同步队列中的当前节点是否有前驱节点的判断，如果有，则表示有线程比当前线程更早的请求锁。所以当前还要继续等待，不能成功获取锁。
* tryRelease会判断，在经过前面n-1次的tryRelease后；判断此次的同步状态是否为0，如果是，则将占有线程设为null，并返回true，表示释放成功。

#### &emsp;&emsp;公平性
大多数情况下，锁的申请都是非公平的。例如，线程1请求了锁A，线程2也请求了锁A，当锁A可用时，是线程1还是线程2获取到锁是不确定的；这种情况下会产生饥饿现象。
ReentrantLock支持对获取锁时公平性的设置。

### 4. 读写锁   
读写锁允许多个线程同时访问。

#### &emsp;&emsp;实现：
* 读写锁的同步器需要在同步状态（变量-state）上维护多个读线程和一个写线程的状态。这样就需要“按位切割使用”这个变量：高16位表示读，低16位表示写。
* 源码解读

```java
/**
ReentrantReadWriteLock中的部分源码，主要分析“写锁”。
@see java.util.concurrent.locks.ReentrantReadWriteLock
*/
static final int SHARED_SHIFT   = 16;
static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1;

/**
下面的公式转成二进制的运算：
= 0000 0000 0000 0001 0000 0000 0000 0000 - 1
= 1*2^16 - 1
= 1*2^15
= 0000 0000 0000 0000 1111 1111 1111 1111
意思是取低位的最大值。
*/
static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;

/** 
Returns the number of shared holds represented in count  
*/
static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }
/** 
Returns the number of exclusive holds represented in count 

返回持有写锁的线程的数量
如果c = 0；则：
0000 0000 0000 0000 0000 0000 0000 0000 & 0000 0000 0000 0000 1111 1111 1111 1111 = 0000 0000 0000 0000 0000 0000 0000 0000
如果c = 1；则：
0000 0000 0000 0000 0000 0000 0000 0001 & 0000 0000 0000 0000 1111 1111 1111 1111 = 0000 0000 0000 0000 0000 0000 0000 0001
可以得知，其意思是计算低位的值。
 */
static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }

/**
该方法最终被ReentrantReadWriteLock.WriteLock调用。
@see java.util.concurrent.locks.ReentrantReadWriteLock.WriteLock#lock
*/
protected final boolean tryAcquire(int acquires) {
	/*
	 * Walkthrough:
	 * 1. If read count nonzero or write count nonzero
	 *    and owner is a different thread, fail.
	 * 2. If count would saturate, fail. (This can only
	 *    happen if count is already nonzero.)
	 * 3. Otherwise, this thread is eligible for lock if
	 *    it is either a reentrant acquire or
	 *    queue policy allows it. If so, update state
	 *    and set owner.
	 */
	Thread current = Thread.currentThread();
	int c = getState();
	//获取持有写锁的线程数量。
	int w = exclusiveCount(c);
	if (c != 0) {
		// (Note: if c != 0 and w == 0 then shared count != 0)
		/**
		在取到写锁线程的数目后，首先判断是否已经有线程持有了锁，如果已经有线程持有了锁(c!=0)，
		则看当前持有写锁的线程的数目，如果写线程数（w）为0（那么读线程数就不为0，因为上面的c和此处的w都是取的state，而读锁和写锁都依赖state。）
		或者独占锁线程（持有锁的线程）不是当前线程就返回失败；
		如果写入锁的数量（其实是重入数）大于65535就抛出一个Error异常。
		*/
		if (w == 0 || current != getExclusiveOwnerThread())
			return false;
		if (w + exclusiveCount(acquires) > MAX_COUNT)
			throw new Error("Maximum lock count exceeded");
		// Reentrant acquire
		setState(c + acquires);
		return true;
	}
	/**
	writerShouldBlock如果返回true，则表示当前线程是在运行在公平锁下，并且有前驱节点，需要阻塞；
					如果返回false，则表示当前线程没有前驱节点或者运行在非公平锁下，不需要阻塞。
	@see java.util.concurrent.locks.ReentrantReadWriteLock.FairSync#writerShouldBlock
	
	compareAndSetState如果返回true，则表示成功获取到锁；
	*/
	if (writerShouldBlock() ||					
		!compareAndSetState(c, c + acquires))
		return false;
	setExclusiveOwnerThread(current);
	return true;
}
```

> 源码解读参考：
> https://blog.csdn.net/MeituanTech/article/details/84138163