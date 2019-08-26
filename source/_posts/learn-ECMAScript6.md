---
title: 记录一些ECMAScript6的语法
date: 2019-08-23 17:45:58
tags:
---

ECMAScript6简称ES6。

<!-- more -->

## 变量的解构赋值
ES6允许按照一定的模式从数组和对象中提取值，然后对变量进行赋值，这称为解构。

### 数组的解构赋值
```JavaScript
//ES6之前
let a = 1;
let b = 2;
let c = 3;
console.log(a); //1

//ES6之后
let [a, b, c] = [1, 2, 3];
console.log(a); //1

let [ , , third] = [1, 2, 3];
console.log(third); //3
```

### 对象的解构赋值
```JavaScript
//代码-1
let {foo, bar} = {foo: "aaa", bar: "bbb"};
console.log(foo); //aaa
console.log(bar);//bbb

//变量没有对应的同名属性，导致取不到值。
let {baz} = {foo: "aaa", bar: "bbb"};
console.log(baz); //undefined

//如果变量名和属性名不一致，必须使用如下语法
let {foo：baz} = {foo: "aaa", bar: "bbb"};
console.log(baz); //aaa

//代码-1是下面语法的简写，大括号中的前者是匹配的模式，后者是真正的变量。
let {foo: foo, bar: bar} = {foo: "aaa", bar: "bbb"};
console.log(foo); //aaa
console.log(bar);//bbb

//解构嵌套解构的对象


```