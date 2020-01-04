---
title: 学习java8的笔记
date: 2019-11-12 10:35:14
tags:
---

jdk1.8的更新内容，相对于前面的版本来说有了很多重大的改变。

<!-- more -->


### lambda表达式

#### lambda表达式语法
```java
(String first, String second) 
    -> {Integer.compare(first.length(), second.length())}
```

如果lambda表达式的参数类型是可以被推导出来的，则可以省略参数类型。下面的类型是根据泛型参数推导出来的。
```java
Comparator<String> com = 
    (first, second) -> Integer.compare(first.length(), second.length());
```

#### 变量作用域

```java
/**
* lambda表达式必须存储这两个变量的值。
* @param text
* @param count
*/
public static void m(String text, int count) {
    Runnable r = () -> {
        /**
        * lambda表达式被转换为只含有一个方法的对象，两个自由变量的值会被复制到该对象的实例变量中。
        * 所以被lambda表达式引用的变量的值是不能被改变的，下面的两句是错误的。
        */
//      count++;
//      text = "a";
        System.out.println(text);
    };
    new Thread(r).start();
}
```