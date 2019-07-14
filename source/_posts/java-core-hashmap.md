---
title: java中HashMap源码分析
date: 2019-06-07 10:26:54
tags:
---

## HashMap源码解析

<!-- more -->

### 存放新的map

```java
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
		//新map存放元素的个数
        int s = m.size();
        if (s > 0) {
			//原始map未初始化，也就是还没有元素
            if (table == null) { // pre-size
				//因为threshold = capacity * loadFactor，此时的threshold相当于就是元素的大小。
				//所以下面的计算相当于ft = threshold / loadFactor，ft也就是capacity。
                float ft = ((float)s / loadFactor) + 1.0F;
                int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                         (int)ft : MAXIMUM_CAPACITY);
				//容量大于阈值，则初始化阈值
                if (t > threshold)
                    threshold = tableSizeFor(t);
            }
			//新map的元素个数大于阈值，需要扩容
            else if (s > threshold)
                resize();
			//新map的添加到原始map中
            for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
                K key = e.getKey();
                V value = e.getValue();
                putVal(hash(key), key, value, false, evict);
            }
        }
    }
```

### put存放新的元素

```java

public V put(K key, V value) {
	return putVal(hash(key), key, value, false, true);
}

/**	
*/
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
				boolean evict) {
	Node<K,V>[] tab; Node<K,V> p; int n, i;
	//原始table未初始化或长度为0，需要调用扩容方法
	if ((tab = table) == null || (n = tab.length) == 0)
		n = (tab = resize()).length;
	/**
	(n - 1) & hash也即是hash值，用来定位key在数组中的位置，如果该位置为空，就创建新的node，并放在数组中。
	(n - 1) & hash保证获取的index一定在数组范围内，举个例子，默认容量16，n-1=15，hash=18,转换成二进制计算为
	 	1  0  0  1  0
    &   0  1  1  1  1
    __________________
        0  0  0  1  0    = 2
	*/
	if ((p = tab[i = (n - 1) & hash]) == null)
		tab[i] = newNode(hash, key, value, null);
	//数组里面有元素了
	else {
		Node<K,V> e; K k;
		//用查到的元素，也即是数组中的第一个元素；与加入的key做比较，比较他们的hash值、地址值。
		if (p.hash == hash &&
			((k = p.key) == key || (key != null && key.equals(k))))
			//如果相等就用一个临时变量记录查到的元素。
			e = p;
		//如果key不相等，并且查到的元素是个红黑树的节点，就将key和value放到红黑树中。
		else if (p instanceof TreeNode)
			e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
		else {
			//如果key不相等，并且查到元素的不是红黑树的节点，那就是链表的节点。
			//下面就是遍历节点所在的链表。
			for (int binCount = 0; ; ++binCount) {
				//遍历到链表的尾部
				if ((e = p.next) == null) {
					//将加入的key和value组成节点放到链表尾部
					p.next = newNode(hash, key, value, null);
					//节点数量达到红黑树阈值，将链表转成红黑树
					if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
						treeifyBin(tab, hash);
					break;
				}
				//如果链表中存在与新加入的key相等的节点，则跳出遍历，丢弃新加入的key
				if (e.hash == hash &&
					((k = e.key) == key || (key != null && key.equals(k))))
					break;
				p = e;
			}
		}
		//如果数组中有和新加入的key相等的节点
		if (e != null) { // existing mapping for key
			V oldValue = e.value;
			//用新的value替换旧的value
			if (!onlyIfAbsent || oldValue == null)
				e.value = value;
			afterNodeAccess(e);
			return oldValue;
		}
	}
	++modCount;
	//如果当前元素数量大于阈值，则进行扩容
	if (++size > threshold)
		resize();
	afterNodeInsertion(evict);
	return null;
}
```

### resize扩容

```java
final Node<K,V>[] resize() {
	Node<K,V>[] oldTab = table;
	int oldCap = (oldTab == null) ? 0 : oldTab.length;
	int oldThr = threshold;
	int newCap, newThr = 0;
	//数组中有元素时，进行扩容。
	if (oldCap > 0) {
		if (oldCap >= MAXIMUM_CAPACITY) {
			threshold = Integer.MAX_VALUE;
			return oldTab;
		}
		else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
					oldCap >= DEFAULT_INITIAL_CAPACITY)
			newThr = oldThr << 1; // double threshold
	}
	//创建HashMap时，指定了capacity和loadFactor，即调用了new HashMap(int initialCapacity)时，将threshold赋给capacity。
	else if (oldThr > 0) // initial capacity was placed in threshold
		newCap = oldThr;
	//创建默认的HashMap，即调用了new HashMap()时，设置默认的capacity和threshold。
	else {               // zero initial threshold signifies using defaults
		newCap = DEFAULT_INITIAL_CAPACITY;
		newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
	}
	//创建HashMap时，指定了capacity和loadFactor，即调用了new HashMap(int initialCapacity)时，设置新的threshold。
	if (newThr == 0) {
		float ft = (float)newCap * loadFactor;
		newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
					(int)ft : Integer.MAX_VALUE);
	}
	threshold = newThr;
	//用新的数组capacity初始化新的数组
	@SuppressWarnings({"rawtypes","unchecked"})
	Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
	table = newTab;
	if (oldTab != null) {
		//开始遍历原数组，进行数据迁移。
		for (int j = 0; j < oldCap; ++j) {
			Node<K,V> e;
			if ((e = oldTab[j]) != null) {
				oldTab[j] = null;
				//元素是单节点，直接复制元素到新的数组。
				if (e.next == null)
					newTab[e.hash & (newCap - 1)] = e;
				//元素是红黑树节点
				else if (e instanceof TreeNode)
					((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
				//元素是链表
				else { // preserve order
					Node<K,V> loHead = null, loTail = null;
					Node<K,V> hiHead = null, hiTail = null;
					Node<K,V> next;
					do {
						next = e.next;
						/**
						进行链表复制。
						将原链表拆分为两个链表，使元素在新的数组中分布的更松散。
						如果 (e.hash & oldCap) == 0 则该节点在新表的下标位置与旧表一致都为 j 
						如果 (e.hash & oldCap) == 1 则该节点在新表的下标位置 j + oldCap
						*/
						if ((e.hash & oldCap) == 0) {
							//保持元素在原链表中的顺序。
							if (loTail == null)
								loHead = e;
							else
								loTail.next = e;
							loTail = e;
						}
						else {
							if (hiTail == null)
								hiHead = e;
							else
								hiTail.next = e;
							hiTail = e;
						}
					} while ((e = next) != null);
					if (loTail != null) {
						loTail.next = null;
						newTab[j] = loHead;
					}
					if (hiTail != null) {
						hiTail.next = null;
						newTab[j + oldCap] = hiHead;
					}
				}
			}
		}
	}
	return newTab;
}
```

### 为何HashMap的数组长度一定是2的次幂？

1. 数组长度保持2的次幂，length-1的低位都为1，会使得获得的数组索引index更加均匀。

> https://www.cnblogs.com/chengxiao/p/6059914.html
> https://segmentfault.com/a/1190000015812438
> https://blog.csdn.net/login_sonata/article/details/76598675