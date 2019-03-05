---
title: JavaScript——闭包
tags: JavaScript原理
abbrlink: 50156
date: 2019-01-30 11:02:36
---

## 什么是闭包

MDN 对闭包的定义为：

> 闭包是指那些能够访问自由变量的函数  
> 自由变量是指在函数中使用的，但既不是函数参数也不是函数的局部变量的变量。

由此，我们可以看出闭包共有两部分组成：
**闭包 = 函数 + 函数能够访问的自由变量**

<!-- more -->

> ECMAScript 中，闭包指的是：  
> 1、从理论角度：所有的函数。因为它们都在创建的时候就将上层上下文的数据保存起来了。哪怕是简单的全局变量也是如此，因为函数中访问全局变量就相当于是在访问自由变量，这个时候使用最外层的作用域。  
> 2、从实践角度：以下函数才算是闭包：  
> （1）即使创建它的上下文已经销毁，它仍然存在（比如，内部函数从父函数中返回）  
> （2）在代码中引用了自由变量

## 举例分析

在《JavaScript 权威指南》中关于闭包有这样一个例子

```
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f;
}
var foo = checkscope();
foo();
```

1. 根据之前所学习的，不难分析出这段代码的执行过程
2. 进入全局代码，创建全局执行上下文，全局执行上下文压入执行上下文栈
3. 全局执行上下文初始化
4. 执行 checkscope 函数，创建 checkscope 函数执行上下文，checkscope 执行上下文被压入执行上下文栈
5. checkscope 执行上下文初始化，创建变量对象、作用域链、this 等
6. checkscope 函数执行完毕，checkscope 执行上下文从执行上下文栈中弹出
7. 执行 f 函数，创建 f 函数执行上下文，f 执行上下文被压入执行上下文栈
8. f 执行上下文初始化，创建变量对象、作用域链、this 等
9. f 函数执行完毕，f 函数上下文从执行上下文栈中弹出

可以发现，当 f 函数执行的时候，checkscope 函数上下文已经从执行上下文栈弹出了，那怎么怎么还会读取到 checkscope 作用域下的 scope 值呢？
我们分析一下 f 的作用域链可以发现

```
fContext = {
    Scope: [AO, checkscopeContext.AO, globalContext.VO],
}
```

就是因为这个作用域链，f 函数依然可以读取到 checkscopeContext.AO 的值，说明当 f 函数引用了 checkscopeContext.AO 中的值的时候，即使 checkscopeContext 被销毁了，但是 JavaScript 依然会让 checkscopeContext.AO 活在内存中，f 函数依然可以通过 f 函数的作用域链找到它，正是因为 JavaScript 做到了这一点，从而实现了闭包这个概念。

## 一道面试题

```
var data = [];

for (var i = 0; i < 3; i++) {
  data[i] = function () {
    console.log(i);
  };
}

data[0]();
data[1]();
data[2]();
```

答案是都是 3。
当执行到 data[0] 函数之前，此时全局上下文的 VO 为：

```
globalContext = {
    VO: {
        data: [...],
        i: 3
    }
}
```

当执行 data[0] 函数的时候，data[0] 函数的执行上下文的活动对象中没有 i，所以会从 globalContext.VO 中查找，i 为 3，所以打印的结果就是 3。
data[1] 和 data[2] 是一样的道理。

那么如何得到我们想要的结果呢？

- 第一种方法：可以将 var 改成 ES6 的 let
- 第二种方法：使用闭包

```
var data = [];

for (var i = 0; i < 3; i++) {
  data[i] = (function (i) {
        return function(){
            console.log(i);
        }
  })(i);
}

data[0]();
data[1]();
data[2]();
```

当执行 data[0] 函数的时候，data[0] 函数的作用域链发生了改变

```
data[0]Context = {
    Scope: [AO, 匿名函数Context.AO globalContext.VO]
}
```

匿名函数执行上下文的 AO 为：

```
匿名函数Context = {
    AO: {
        arguments: {
            0: 0,
            length: 1
        },
        i: 0
    }
}
```

data[0]Context 的 AO 并没有 i 值，所以会沿着作用域链从匿名函数 Context.AO 中查找，这时候就会找 i 为 0，找到了就不会往 globalContext.VO 中查找了，即使 globalContext.VO 也有 i 的值(值为 3)，所以打印的结果就是 0。
