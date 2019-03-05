---
title: JavaScript——执行上下文栈
tags: JavaScript原理
abbrlink: 58701
date: 2019-01-22 10:49:28
---

## 顺序执行

如果要问到 JavaScript 代码执行顺序的话，想必写过 JavaScript 的开发者都会有个直观的印象，那就是顺序执行
看下面两个例子

```
var foo = function () {
    console.log('foo1');
}

foo();  // foo1

var foo = function () {
    console.log('foo2');
}

foo(); // foo2
```

<!-- more -->

顺序执行没毛病～
然而看下面这个：

```
function foo() {
    console.log('foo1');
}

foo();  // foo2

function foo() {
    console.log('foo2');
}

foo(); // foo2
```

输出结果是两个 foo2
这是因为 JavaScript 引擎并非一行一行地分析和执行程序，而是一段一段地分析执行。当执行一段代码的时候，会进行一个“准备工作”，比如第一个例子中的变量提升，和第二个例子中的函数提升。
第二个例子中，函数的声明被提升了，输出 foo2 的函数覆盖了前一个函数，所以两次调用的输出都是 foo2。

## 可执行代码

JavaScript 的可执行代码有三种：全局代码、函数代码、eval 代码。
当遇到可执行代码时，就会进行准备工作，这里的“准备工作”，用个更专业一点的说法，就叫做"执行上下文(execution context)"。

## 执行上下文栈

JavaScript 引擎创建了执行上下文栈（Execution context stack，ECS）来管理执行上下文。
我们可以通过一个数组来模拟：`ECStack = [];`
试想当 JavaScript 开始要解释执行代码的时候，最先遇到的就是全局代码，所以初始化的时候首先就会向执行上下文栈压入一个全局执行上下文，我们用 globalContext 表示它，并且只有当整个应用程序结束的时候，ECStack 才会被清空，所以程序结束之前， ECStack 最底部永远有个 globalContext：

```
ECStack = [
    globalContext
];
```

接下来遇到了这样一段代码：

```
function fun3() {
    console.log('fun3')
}

function fun2() {
    fun3();
}

function fun1() {
    fun2();
}

fun1();
```

那么执行上下文栈的变化如何呢？

```
// fun1()
ECStack.push(<fun1> functionContext);

// fun1中调用了fun2，创建fun2的执行上下文
ECStack.push(<fun2> functionContext);

// fun2还调用了fun3，创建fun3的执行上下文
ECStack.push(<fun3> functionContext);

// fun3执行完毕
ECStack.pop();

// fun2执行完毕
ECStack.pop();

// fun1执行完毕
ECStack.pop();

// javascript接着执行下面的代码，但是ECStack底层永远有一个globalContext
```
