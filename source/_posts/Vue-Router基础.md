---
title: Vue-Router基础
date: 2018-10-12 11:32:02
tags: Vue.js
---

# Vue-Router

路由中有三个基本的概念 route, routes, router。

1、 route，它是一条路由，由这个英文单词也可以看出来，它是单数， Home 按钮 => home 内容， 这是一条 route, about 按钮 => about 内容， 这是另一条路由。

2、 routes 是一组路由，把上面的每一条路由组合起来，形成一个数组。[{home 按钮 =>home 内容 }， { about 按钮 => about 内容}]

<!--more-->

3、router 是一个机制，相当于一个管理者，它来管理路由。因为 routes 只是定义了一组路由，它放在哪里是静止的，当真正来了请求，怎么办？ 就是当用户点击 home 按钮的时候，怎么办？这时 router 就起作用了，它到 routes 中去查找，去找到对应的 home 内容，所以页面中就显示了 home 内容。

4、客户端中的路由，实际上就是 dom 元素的显示和隐藏。当页面中显示 home 内容的时候，about 中的内容全部隐藏，反之也是一样。客户端路由有两种实现方式：基于 hash 和基于 html5 history api.
![](/blog/images/4.jpg)

- vue-router 中，首先在 routes 数组中定义所需要的 route。
- 然后创建 router 对路由进行管理，它是由构造函数 new vueRouter() 创建，接受 routes 参数
- 最后将 router 实例注入到 vue 根实例中,就可以使用路由了
  ![](/blog/images/5.jpg)
  执行过程：当用户点击 router-link 标签时，会去寻找它的 to 属性， 它的 to 属性和 js 中配置的路径{ path: '/home', component: Home} path 一一对应，从而找到了匹配的组件， 最后把组件渲染到 <router-view> 标签所在的地方。所有的这些实现才是基于 hash 实现的。

      编程式导航：这主要应用到按钮点击上。当点击按钮的时候，跳转另一个组件, 这只能用代码，调用`rourter.push()` 方法。 当们把router 注入到根实例中后，组件中通过 this.$router 可以获取到router, 所以在组件中使用

  `this.$router.push("home”)`就可以跳转到 home 界面
