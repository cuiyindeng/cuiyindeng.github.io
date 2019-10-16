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

var x = 1;
var y = 2;

`${x} + ${y} = ${x + y}`
//"1 + 2 = 3"
```

## Module的语法
Module可以将大的程序拆分成互相依赖的小文件，再用简单的方法将它们拼接起来。
ES6的模块默认使用严格模式，不管有没有在头部加"use strict"。

### export命令
模块功能只要有两个命令构成：export和import；export命令用于声明模块的对外接口，import命令用于输入其他模块提供的功能。

一个模块就是一个独立文件，文件内的变量，外部无法获取。如果外部想要读取模块内部的某个变量，模块必须使用export关键字输出该变量。

```JavaScript
//写法1
//profile.js
export var firstName = 'Michael';
export var lastName = 'Jackson';
export var year = 1983;

//写法2
//profile.js
var firstName = 'Michael';
var lastName = 'Jackson';
var year = 1983;

export {firstName, lastName, year};

//写法3
//profile.js
var v1 = 'Michael';
var v2 = 'Jackson';
var v3 = 1983;

export {v1 as firstName, v2 as lastName, v3 as year};

//export还可以导出函数和类。

```

### import命令
使用了export定义了模块的对外接口后，在其它文件中就可以使用import加载这个模块了。
```JavaScript
//写法1
//main.js
import {firstName, lastName, year} from './profile';

function setName(element) {
    element.textContent = firstName + ' ' + lastName;
}

//写法2
//为导入的变量声明别名
import {firstName as surnamae} from './profile';

/**
import是静态执行，所以不能使用变量（let x）和表达式（x + 'a'），只有在运行时才能得到结果。
*/

```
### export default命令
使用import命令时必须知道要加载的变量名或函数名，否则无法加载。
为了能不使用变量名就能加载变量和函数，可以使用export default为模块指定默认输出。
一个模块只能有一个export default。
本质上export default就是输出一个叫做default的变量或方法，系统允许我们为它取任意的名字。

```JavaScript
//export-default.js
export default function() {
    console.log('foo');
}

//import-default.js
//加载该模块时，import命令可以为该匿名函数指定任意的名字，此时import后面不适用大括号。
import customName from './export-default';
customName();//foo
```

### export和import的复合写法
```JavaScript
export {foo, bar} from 'my_module';
//等价于
import {foo, bar} from 'my_module';
export {foo, bar};
```
