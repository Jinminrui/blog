---
title: JavaScript——继承
tags: JavaScript原理
abbrlink: 45068
date: 2019-02-20 15:03:39
---

## 前言

这篇文章将记录 JavaScript 中的几种继承方式（不含 es6），并分析其中的优缺点。另一方面也加深一下对原型链的理解。

<!-- more -->

## 原型链继承

```
function Parent () {
    this.name = 'xiaoming';
}

Parent.prototype.getName = function () {
    console.log(this.name);
}

function Child () {

}

Child.prototype = new Parent();

var child1 = new Child();
console.log(child1.getName()) // xiaoming
```

简而言之，原型链继承就是就是直接将父类的一个实例赋给子类的原型。
如代码中所示，这种方法是直接 new 了一个父类的实例，然后赋给子类的原型。这样也就相当于直接将父类原型中的方法属性以及挂在 this 上的各种方法属性全赋给了子类的原型，简单粗暴！最后在子类实例中寻找 getName 方法时，子类实例中没有，便会到原型链上去找。
这种继承方式有两个缺点：

1. 引用类型的属性被所有实例共享
2. 在创建 Child 的实例时，不能向 Parent 传参

## 借用构造函数（经典继承）

```
function Parent () {
    this.names = ['one', 'two'];
}

function Child () {
    Parent.call(this);
}

var child1 = new Child();

child1.names.push('three');

console.log(child1.names); // ["one", "two", "three"]

var child2 = new Child();

console.log(child2.names); // ["one", "two"]
```

这种方式，避免了引用类型的属性被所有实例共享，并且可以在 Child 中向 Parent 传参。

```
function Parent (name) {
    this.name = name;
}

function Child (name) {
    Parent.call(this, name);
}
```

**缺点：**
方法都在构造函数中定义，每次创建实例都会创建一遍方法。

### 组合继承

组合继承结合了上述两种继承方法

```
function Parent (name) {
    this.name = name;
    this.colors = ['red', 'blue', 'green'];
}

Parent.prototype.getName = function () {
    console.log(this.name)
}

function Child (name, age) {

    Parent.call(this, name);

    this.age = age;

}

Child.prototype = new Parent();
Child.prototype.constructor = Child;

var child1 = new Child('kevin', '18');

child1.colors.push('black');

console.log(child1.name); // kevin
console.log(child1.age); // 18
console.log(child1.colors); // ["red", "blue", "green", "black"]

var child2 = new Child('daisy', '20');

console.log(child2.name); // daisy
console.log(child2.age); // 20
console.log(child2.colors); // ["red", "blue", "green"]
```

## 寄生组合式继承

组合继承最大的缺点是会调用两次父构造函数。
一次是设置子类型实例的原型的时候：
`Child.prototype = new Parent();`
一次在创建子类型实例的时候：
`var child1 = new Child('kevin', '18');`
为了解决重复调用的问题，在这里我们不使用 `Child.prototype = new Parent()`，而是间接的让 Child.prototype 访问到 Parent.prototype

```
function Parent (name) {
    this.name = name;
    this.colors = ['red', 'blue', 'green'];
}

Parent.prototype.getName = function () {
    console.log(this.name)
}

function Child (name, age) {
    Parent.call(this, name);
    this.age = age;
}

// 关键的三步
var F = function () {};

F.prototype = Parent.prototype;

Child.prototype = new F();
```

其实这三行代码就是对 ES5 中 Object.create()的模拟

```
var F = function () {};
F.prototype = Parent.prototype;
Child.prototype = new F();
```

也可以直接用 ES5 的 Object.create()
`Child.prototype = Object.create(Parent.prototype)`

> 这种方式的高效率体现它只调用了一次 Parent 构造函数，并且因此避免了在 Parent.prototype 上面创建不必要的、多余的属性。与此同时，原型链还能保持不变；因此，还能够正常使用 instanceof 和 isPrototypeOf。开发人员普遍认为寄生组合式继承是引用类型最理想的继承范式。
