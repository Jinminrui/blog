---
title: 关于Vue生命周期
date: 2018-10-12 10:27:11
tags: Vue.js
---

### vue 提供了如下的钩子函数供我们在 vue 生命周期的不同时刻调用：

- beforeCreate
- created
- beforeMount
- mounted
- beforeUpdate
- updated
- beforeDestroy
- destroyed
  <!--more-->

---

### 1、在 beforeCreate 和 created 钩子函数之间的生命周期

在这个生命周期之间，进行初始化事件，进行数据的观测，可以看到在 created 的时候数据已经和 data 属性进行绑定（放在 data 中的属性当值发生改变的同时，视图也会改变）。
~注意看下：此时还是没有 el 选项~

---

### 2、created 钩子函数和 beforeMount 间的生命周期

![](/hexo/images/1.jpg)

首先会判断对象是否有 el 选项。如果有的话就继续向下编译，如果没有 el 选项，则停止编译，也就意味着停止了生命周期，直到在该 vue 实例上调用 vm.\$mount(el)。
**template 参数对生命周期的影响**：
（1）如果 vue 实例对象中有 template 参数选项，则将其作为模板编译成 render 函数。
（2）如果没有 template 选项，则将外部 HTML 作为模板编译。
（3）可以看到 template 中的模板优先级要高于 outer HTML 的优先级。
_在 vue 对象中还有一个 render 函数，它是以 createElement 作为参数，然后做渲染操作，而且我们可以直接嵌入 JSX._

```
new Vue({
    el: '#app',
    render: function(createElement) {
        return createElement('h1', 'this is createElement')
    }
})
```

综合排名优先级：
**render 函数选项 > template 选项 > outer HTML.**

---

### 3、beforeMount 和 mounted 钩子函数间的生命周期

此时是给 vue 实例对象添加\$el 成员，并且替换掉挂在的 DOM 元素。
![](/hexo/images/2.jpg)

---

### 4、mounted

在 mounted 之前 h1 中还是通过{{message}}进行占位的，因为此时还有挂在到页面上，还是 JavaScript 中的虚拟 DOM 形式存在的。在 mounted 之后可以看到 h1 中的内容发生了变化。
![](/hexo/images/3.jpg)

---

### 5、beforeUpdate 钩子函数和 updated 钩子函数间的生命周期

当 vue 发现 data 中的数据发生了改变，会触发对应组件的重新渲染，先后调用 beforeUpdate 和 updated 钩子函数。在 beforeUpdate,可以监听到 data 的变化但是 view 层没有被重新渲染，view 层的数据没有变化。等到 updated 的时候 view 层才被重新渲染，数据更新。

---
