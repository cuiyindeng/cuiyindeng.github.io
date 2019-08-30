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
let obj = {
    p: [
        'Hello',
        {y: 'World'}
    ]
};
//p是模式，后面的数组是变量；
//x是变量，因为'Hello'是字面量，所以可以直接赋值给x；
//y是与对象属性同名的变量，如果用其他变量名，必须在变量名前面声明模式。
let {p: [x, {y}]} = obj;
console.log(x);//Hello
console.log(y);//World

//多次解构赋值对象。
//前面的p是第一次解构与对象属性名相同的变量。
//后面的p是第二次解构的模式。
let {p, p: [x, {y}]} = obj;
console.log(p);//['Hello', {y: 'World'}]
console.log(x);//Hello
console.log(y);//World
```

### 字符串的解构赋值
```JavaScript
const [a, b, c, d, e] = 'hello';
console.log(a);//h
console.log(b);//e
console.log(c);//l
...
```

### 数值和字符值的解构赋值
```JavaScript
let {toString: s} = 1123;
console.log(s)//ƒ toString() { [native code] } 
```

### 解构赋值的用途

##### 交换变量
```JavaScript
let x = 1;
let y = 2;

[x, y] = [y, x];
console.log(x);//2
console.log(y);//1
```

##### 函数返回多个值
```JavaScript
//返回一个数组
function example() {
    return [1, 2, 3];
}

let [a, b, c] = example();
console.log(a);//1
console.log(b);//2
console.log(c);//3

//返回一个对象
function example() {
    return {
        foo: 1,
        bar: 2
    };
}

let {foo, bar} = example();
console.log(foo);//1
console.log(bar);//2
```

##### 函数参数的定义
```JavaScript
//传递数组
function f([x, y, z]) {
    console.log(x);
    console.log(y);
    console.log(z);
}
f([1, 2, 3]]);
//1
//2
//3

//传递对象
function f({x, y, z}) {
    console.log(x);
    console.log(y);
    console.log(z);
}
f({'z': 3, 'y': 2, 'x': 1});
//1
//2
//3

```
##### 提取json数据
```JavaScript
let jsonData = {
    id: 42,
    status: "OK",
    data: [867,5039]
};

let {id, status, data: number} = jsonData;
console.log(id, status, number);//42, "OK", [867, 5039]
```

##### 函数参数的默认值
```JavaScript
jQuery.ajax = function(url, {
    async = true,
    beforeSend = function() {},
    cache = true,
    //more config
}) {
    //do stuff
};
```

##### 遍历map解构
```JavaScript
var map = new Map();
map.set('first', 'hello');
map.set('second', 'wolrd');
for (let [key, value] of map) {
    console.log(key + " is " + value);
}
//first is hello
//second is world
```

##### 输入模块的指定方法
```JavaScript
const {SourceMapConsumer, SourceNode} = require("source-map");
```

## 字符串的扩展 

### 模板字符串
```JavaScript
//传统的js写法：
$('#result').append(
    'There are <b>' + basket.count + '</b> ' + 
    'items in your basket, ' + 
    '<em>' + basket.onSale + 
    '</em> are on sale!'
);

//es6的写法
$('#result').append(`
    There are <b>${basket.count}</b> 
    items in your basket,  
    <em>${basket.onSale}</em> are on sale!
`);

var name = "Bob", time = "today";
`hello ${name}, how are you ${time}?`
//hello Bob, how are you today?

/**
模板中还能放表达式用于运算，以及引用对象属性。
模板字符串中还能调用函数。
*/
```

