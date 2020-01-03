---
title: java中的泛型
date: 2019-06-29 11:52:39
tags:
---

泛型实现了`参数化类型`的概念，使代码可以应用于多种类型；使程序有更好的可读性和安全性。

<!-- more -->

可读性：可以很直观的看到泛型类或方法接收什么类型的参数
安全性：在使用泛型是必须指定类型参数，避免了在获取类型参数时产生的类型转换异常。

### 泛型类

```java
/**
 * 这是一个泛型类，T就是类型参数
 * @param <T>
 */
public class Tool<T> {

    private T a;

    public T get() {
        return this.a;
    }

    public void set(T a) {
        this.a = a;
    }

}
```

### 泛型接口

```java
public interface Generator<T> {
    //返回类型是参数化的T
    T next();
}
```

```java
public class Fibonacci implements Generator<Integer> {
    private int count = 0;

    /**
     * 参数化的Generator接口确保next的返回值是参数的类型。
     * @return
     */
    @Override
    public Integer next() {
        return fib(count++);
    }
    private int fib(int n) {
        if (n < 1) {
            return 1;
        }
        return fib(n - 2) + fib(n - 1);
    }

    public static void main(String[] args) {
        Fibonacci gen = new Fibonacci();
        for (int i = 0; i < 8; i++) {
            System.out.println(gen.next() + "");
        }
    }
}
```

### 泛型方法

泛型方法使得方法能够独立于类而产生定义上的变化。
1. 如果泛型方法可以取代整个类的泛型化，就应该尽量使用泛型方法。
2. static方法不能访问类的类型参数，所以static方法如果要使用泛型能力，必须将其定义为独立的泛型方法。

```java
public class GenericMethods {
    public <T> void f(T x) {
        System.out.println(x.getClass().getName());
    }

    public static void main(String[] args) {
        GenericMethods gm = new GenericMethods();
        gm.f("");
        gm.f(1);
        gm.f(gm);
    }
}
```

### 擦除

1. 在泛型代码内部，无法获取泛型参数类型的任何信息。
2.  ArrayList<Demo>.class 替换为new ArrayList<Demo>().getClass()。