---
title: JavaScript——作用域
tags: JavaScript原理
abbrlink: 53883
date: 2019-01-22 10:28:54
---

## 什么是作用域

作用域是指程序源代码中定义变量的区域。
作用域规定了如何查找变量，也就是确定当前执行代码对变量的访问权限。
JavaScript 采用词法作用域(lexical scoping)，也就是静态作用域。

<!-- more -->

## 静态作用域和动态作用域

- 因为 JavaScript 采用的是词法作用域，函数的作用域在**函数定义的时候**就决定了。
- 而与词法作用域相对的是动态作用域，函数的作用域是在**函数调用的时候**才决定的。
  （这里要划重点）

```
var value = 1;

function foo() {
    console.log(value);
}

function bar() {
    var value = 2;
    foo();
}

bar();
```

- 假设 JavaScript 采用静态作用域，让我们分析下执行过程：
  执行 foo 函数，先从 foo 函数内部查找是否有局部变量 value，如果没有，就根据书写的位置，查找上面一层的代码，也就是 value 等于 1，所以结果会打印 1。

- 假设 JavaScript 采用动态作用域，让我们分析下执行过程：
  执行 foo 函数，依然是从 foo 函数内部查找是否有局部变量 value。如果没有，就从调用函数的作用域，也就是 bar 函数内部查找 value 变量，所以结果会打印 2。

JavaScript 采用的是静态作用域，所以这个例子的结果是 1。
