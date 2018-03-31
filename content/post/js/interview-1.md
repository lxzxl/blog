---
title: "面试知识点总结 (1)"
date: 2018-03-31T07:50:49+08:00
keywords: ["面试", "javascript", "作用域", "作用域链", "闭包"]
description: "总结面试中常问的 JavaScript 知识点：关于作用域和闭包"
tags: ["javascript"]
categories: ["javascript"]
author: "lxzxl"
---

总结面试中常问的 JavaScript 知识点之 - 作用域和闭包

# 执行上下文

先看 3 个例子：

```javascript
console.log(a); // ReferenceError

console.log(a); // undefined
var a;

console.log(a); // undefined
var a = 10;

console.log(f1); // function f1(){}
function f1() {}

console.log(f2); // undefined
function f2() {}
```

解释：

一段 js 代码在真正运行前会做一些准备工作:

* 变量、函数表达式的声明，默认赋值为 undefined
* 给 this 赋值
* 函数声明

这三种数据的准备工作就叫做 ** 执行上下文 **，** 每次调用 ** 函数都会生成 ** 新的 ** 执行上下文。

如果代码段是 ** 函数体 **，那么还要准备另外两种数据：

* arguments
* 作用域

# this

`this` 是在函数真正被调用时确定的，函数定义时无法确定。

1.  使用 `new`

new 的原理是，创建一个空对象，将这个空对象当做 this。

2.  函数作为属性

如果函数作为对象的一个属性时，并且作为对象的一个属性被调用时，函数中的 this 指向该对象。

```javascript
var obj = {
  value: 123,
  fn: function() {
    console.log(this.value);
  }
};
console.log(obj.fn()); // 10
var fn = obj.fn;
console.log(fn()); // undefined
```

3.  用 call、apply 调用

this 的值就取传入的对象的值

```javascript
var obj = {
  value: 123
};

function fn() {
  console.log(this.value);
}
fn(); // undefined
fn.call(obj); // 123
```

4.  全局 和 作为普通函数调用

this 是 window

```javascript
var obj = {
  value: 123,
  fn: function() {
    function f() {
      console.log(this.value);
    }
    f();
  }
};

obj.fn(); // undefined
```

# 作用域

变量作用域

> 程序中定义这个变量的区域，有全区变量和局部变量

函数作用域

> js 中没有块级作用域，只有函数作用域
>
> 函数内声明的所有变量在函数体内始终可见
>
> ** 静态作用域 ** 又叫做词法作用域

作用域的用处就是隔离变量，不同作用域下的同名变量不冲突。

#作用域链

## 自由变量

在 A 作用域中使用的变量 x，却没有在 A 作用域中声明，对于 A 作用域来说，x 就是一个自由变量。

查找自由变量时，要到 ** 创建这个函数的作用域中 ** 取值。

```javascript
var x = 10;

function fn() {
  console.log(x);
}

function other(f) {
  var x = 20;
  f();
}

other(fn); // 10
```

## 作用域链

像上面说的去 ** 创建这个函数的作用域中 ** 查找，一直到全局作用域的查找路线，就是作用域链。

每段 js 代码 (全局代码) 都有一个与之关联的作用域链，是一个列表或链表，包含了这个函数所有的可访问的变量。当 js 需要查找变量时，会从第一个对象开始找相同名字，找不到就去下一个找，知道最后抛出 ReferenceError。

当定义一个函数时，因为函数也是对象，它会保存一个作用域链。从函数内部向外遍历，每当碰到一个 function 时，就将其对应的变量对象添加至作用域链中去，如此下去，直到 global 对象，然后将作用域链的引用赋给 `[[Scope]]` 属性

当调用这个函数时，会创建一个新的对象存储它的局部变量，并将这个对象放在作用域链的前端。

### 其他

全局变量存在于运行期上下文作用域链的最末端，查找最慢，所以我们应该尽可能少使用全局变量，如果使用，就先用局部变量缓存下来；

`with` 会在作用域链前端添加一个对象。

# 闭包

概念不好记，只需知道两种使用情况：

1.  函数作为返回值

```javascript
function func() {
  var num = 100;
  return function(input) {
    if (num > 100) {
      console.log(input);
    }
  };
}
var check = func();
check(300); // 300
```

2.  函数作为参数被传递

```javascript
var name = 'global';
function func() {
  console.log(name);
}

(function fn(f) {
  var name = 'fn';
  f();
})(func); // global
```

闭包常用来隔离变量，缓存。

使用闭包会增加内存消耗。
