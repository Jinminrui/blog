---
title: CSS布局笔记
date: 2018-10-16 20:51:17
tags: CSS
---

## 一、盒模型

1. ie 盒模型算上 border、padding 及自身（不算 margin），标准的只算上自身窗体的大小 css 设置方法如下：

```
/* 标准模型 */
box-sizing:content-box;
 /*IE模型*/
box-sizing:border-box;
```

<!--more-->

2. margin、border、padding、content 由外到里
3. 最常用的获得宽高的方式`dom.offsetWidth/offsetHeight`
4. 各种获得宽高的方式
   - 获取屏幕的高度和宽度（屏幕分辨率）：window.screen.height/width
   - 获取屏幕工作区域的高度和宽度（去掉状态栏）window.screen.availHeight/availWidth
   - 网页全文的高度和宽度：document.body.scrollHeight/Width
   - 滚动条卷上去的高度和向右卷的宽度：document.body.scrollTop/scrollLeft
   - 网页可见区域的高度和宽度（不加边线）：document.body.clientHeight/clientWidth
   - 网页可见区域的高度和宽度（加边线）：document.body.offsetHeight/offsetWidth

## 二、居中方式

1. 水平居中
   内联元素：inline, 内联块 inline-block, 内联表 inline-table, inline-flex 元素及 img,span,button 等
   `text-align:center;`
   不定宽块元素居中：
   `margin:0 auto;//且需要设置父级宽度`
   通过给父元素设置 float，然后给父元素设置 position:relative 和 left:50%，子元素设置 position:relative 和 left: -50% 来实现水平居中。

```
.wrap{
    float:left;
    position:relative;
    left:50%;
    clear:both;
}
.wrap-center{
    left:-50%;
}
```

2. 垂直居中
   单行内联(inline-)元素垂直居中 通过设置内联元素的高度(height)和行高(line-height)相等，从而使元素垂直居中。

```
height: 120px;
line-height: 120px;
```

利用表布局

```
.father {
    display: table;
}
.children {
    display: table-cell;
    vertical-align: middle;
    text-align: center;
}
```

flex 布局

```
.center-flex {
    display: flex;
    flex-direction: column;//上下排列
    justify-content: center;
}
```

绝对布局方式
已知高度

```
.parent {
  position: relative;
}
.child {
  position: absolute;
  top: 50%;
  height: 100px;
  margin-top: -50px;
}
```

未知高度

```
.parent {
    position: relative;
}
.child {
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
}
```

3. 垂直水平居中
   flex 方式

```
.parent {
    display: flex;
    justify-content: center;
    align-items: center;
}
```

grid 方式

```
.parent {
  height: 140px;
  display: grid;
}
.child {
  margin: auto;
}
```
