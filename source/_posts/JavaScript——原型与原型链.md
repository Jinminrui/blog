---
title: JavaScript——原型与原型链
tags: JavaScript原理
abbrlink: 568
date: 2019-01-21 15:35:45
---

## 构造函数

在 JavaScript 中构造函数和 Java 中的函数一样也可以创建对象实例。

<!-- more -->

例如下面的代码：

```
function Person() {
}
var person = new Person();
person.name = 'myname';
```

Person 是一个构造函数，我们使用 new 创建了一个实例对象 person。

## prototype

每一**函数**都有一个 prototype 属性，这个 prototype 属性指向一个对象，这个对象正是调用该构造函数而创建的实例的原型，也就是 person 的原型。

- 什么是原型：
  每一个 JavaScript 对象(null 除外)在创建的时候就会与之关联另一个对象，这个对象就是我们所说的原型，每一个对象都会从原型"继承"属性。
  那么 person 的原型该如何表示呢？

## **proto**

这是每一个 JavaScript 对象(除了 null )都具有的一个属性，叫`__proto__`，这个属性会指向该对象的原型。
在浏览器控制台中输入如下的代码可以证明这一点：

```
function Person() {

}
var person = new Person();
console.log(person.__proto__ === Person.prototype); // true
```

由此可以知道，构造函数和实例可以指向原型，那么原型是否有属性指向构造函数和实例呢？

## constructor

指向实例倒是没有，因为一个构造函数可以生成多个实例，但是原型指向构造函数倒是有的，这就要讲到第三个属性：constructor，每个原型都有一个 constructor 属性指向关联的构造函数。
在浏览器中控制台输入如下代码可以证明

```
function Person() {

}
console.log(Person === Person.prototype.constructor); // true
```

## 实例和原型

当读取实例的属性时，如果找不到，就会查找与对象关联的原型中的属性，如果还查不到，就去找原型的原型，一直找到最顶层为止。
形象一点描述，就是儿子的属性里找不到，就到爸爸的属性里找，爸爸的属性里找不到就到爸爸的爸爸的属性里去找……以此类推。知道找到原型的原型，那原型的原型又是什么呢？

## 原型的原型

在前面，我们已经讲了原型也是一个对象，既然是对象，我们就可以用最原始的方式创建它，那就是：

```
var obj = new Object();
obj.name = 'myname'
console.log(obj.name) // myname
```

其实原型对象就是通过 Object 构造函数生成的。

## 原型链

那 Object.prototype 的原型呢？
答案是：null，我们可以尝试：

```
console.log(Object.prototype.__proto__ === null) // true
```

null 表示没有对象，即该处不应该有值。
所以 `Object.prototype.__proto__`的值为 null 跟 Object.prototype 没有原型，其实表达了一个意思。
所以查找属性的时候查到 Object.prototype 就可以停止查找了。
盗用一下[伢羽大大博客](https://github.com/mqyqingfeng/Blog/issues/2)上的图片，关系图如下：
![](http://ww1.sinaimg.cn/large/005ZR24Xgy1g0ry7z7s96j30mm0k276k.jpg)
图中由相互关联的原型组成的链状结构就是原型链，也就是蓝色的这条线。
