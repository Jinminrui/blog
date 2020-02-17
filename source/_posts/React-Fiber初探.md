---
title: React Fiber调度机制
tags: react
abbrlink: 35045
date: 2019-10-23 16:58:50
---

## Fiber 的使命

我们都知道 JavaScript 是单线程的，同时只能有一个任务运行。然而浏览器除了执行 JavaScript 还有很多事情要做，比如事件处理、界面渲染和绘制、网络资源的加载和处理等等。所以当一个 JavaScript 任务长时间占据 CPU 时，后面什么事情就都做不了了，这时页面就会非常的卡顿，用户的体验会很差。

<!-- more -->

**在 React 中渲染时会递归比对 VirtualDOM 树，找出需要变动的节点，然后同步更新它们, 一气呵成。这个过程 React 称为 Reconcilation**。在这个过程中，React 会长时间占用浏览器的资源，导致用户触发的事件得不到响应和掉帧。React 的 Fiber 架构就是为了解决这个问题而生。

**为了给用户制造一种应用很快的’假象’，我们不能让一个程序长期霸占着资源. 你可以将浏览器的渲染、布局、绘制、资源加载(例如 HTML 解析)、事件响应、脚本执行视作操作系统的’进程’，我们需要通过某些调度策略合理地分配 CPU 资源，从而提高浏览器的用户响应速率, 同时兼顾任务执行效率**。

所以 React 通过 Fiber 架构，让自己的 Reconcilation 过程变成可被**中断**。 适时地让出 CPU 执行权，让浏览器能够及时地响应用户的交互。

## 何为 Fiber

fiber 在英文中的意思是“纤维”。在 React 中，它更像是一种数据结构或者执行单元，每当执行完一个执行单元，React 就会检查现在还剩下多少时间，如果没有时间就把控制权交出去。
Chrome 浏览器提供了一个接口——requestIdleCallback

### requestIdleCallback

页面是一帧一帧绘制出来的，当每秒绘制的帧数（FPS）达到 60 时，页面是流畅的，小于这个值时，用户会感觉到卡顿。
1s60 帧，所以每一帧分到的时间是 1000/60 ≈ 16 ms。所以我们书写代码时力求不让一帧的工作量超过 16ms。
**浏览器在 1 帧的时间内需要完成如下的工作：**

- 处理用户的交互
- JS 解析执行
- 帧开始。窗口尺寸变更，页面滚去等的处理
- requestAnimationFrame
- 布局
- 绘制
  上面六个步骤完成后没超过 16 ms，说明时间有富余，此时就会执行 requestIdleCallback 里注册的任务。此外如果浏览器每一帧都被安排满了，requestIdleCallback 注册的任务有可能永远不会执行。此时可通过设置 timeout 来保证执行。

![1.png](http://ww1.sinaimg.cn/large/005ZR24Xgy1g888qqynzij30ss0gfwgv.jpg)
目前 requestIdleCallback 目前只有 Chrome 支持。所以目前 React [自己实现了一个](https://github.com/facebook/react/blob/master/packages/scheduler/src/forks/SchedulerHostConfig.default.js) 。它利用 [MessageChannel](https://developer.mozilla.org/zh-CN/docs/Web/API/MessageChannel) 模拟将回调延迟到绘制操作之后执行。

### 任务优先级

上面说了，为了避免任务被饿死，可以设置一个超时时间. **这个超时时间不是死的，低优先级的可以慢慢等待, 高优先级的任务应该率先被执行**. 目前 React 预定义了 5 个优先级：

1. Immediate(-1) - 这个优先级的任务会同步执行, 或者说要马上执行且不能中断
2. UserBlocking(250ms) 这些任务一般是用户交互的结果, 需要即时得到反馈
3. Normal (5s) 应对哪些不需要立即感受到的任务，例如网络请求
4. Low (10s) 这些任务可以放后，但是最终应该得到执行. 例如分析通知
5. Idle (没有超时时间) 一些没有必要做的任务 (e.g. 比如隐藏的内容), 可能会被饿死

那么如果我们使用 setState 了来更新组件，这个待更新的任务会被放进更新队列中，然后通过 requestIdleCallback 请求浏览器调度。如果当前余有时间，就会从更新队列循环弹出任务来执行，每执行完一个任务就会判断一下时间是否充足，如果充足就进行执行下一个执行单元，反之则停止执行，保存现场，等下一次有执行权时恢复。
![2.png](http://ww1.sinaimg.cn/large/005ZR24Xgy1g888ubh5pyj30qo0riglv.jpg)

## Fiber 的数据结构

在 react16 之前，Reconciliation 是同步的、递归执行的。也就是说这是基于函数’调用栈‘的 Reconciliation 算法。只不过这种依赖于调用栈的方式不能随意中断、也很难被恢复, 不利于异步处理。 React 目前的做法是采用**链表**的数据结构，每个 VirtualDOM 节点内部现在使用**Fiber**表示。
关键属性如下：

```js
const Fiber = {
  tag: HOST_COMPONENT,
  type: "div",
  return: parentFiber, // 指向父节点
  child: childFiber, // 指向子节点
  sibling: null, // 指向兄弟节点
  alternate: currentFiber,
  stateNode: document.createElement("div") | instance,
  props: { children: [], className: "foo" },
  partialState: null,
  effectTag: PLACEMENT,
  effects: []
};
```

return 存储的是当前节点的父节点（元素），child 存储的是第一个子节点（元素），sibling 存储的是他右边第一个的兄弟节点（元素）。alternate 保存是当更新发生时候同一个节点带有新的 props 和 state 生成的新 fiber 节点。 那么虚拟 dom 的存储结构用链表的形式描述了整棵树。

举个 🌰：
![](http://ww1.sinaimg.cn/large/005ZR24Xgy1g9n39f0iyqj30ra0c7mxl.jpg)

所以整个 Fiber Diff 的大致过程如下：
碰到一个节点和 alternate 比较并记录下需要更新的东西，并把这些更新提交到当前节点的父亲。当遍历完这颗树的时候，再通过 return 回溯到根节点。这个过程中把所有的更新全部带到根节点，再一次更新到真实的 dom 中去。

这个遍历的过程是可以中断的，由于是链表结构，react 只需用一个变量存储一下暂停的节点，等下一次浏览器有空余时间的时候，再继续执行 diff。

## 源码分析

最后从源码角度来梳理一下整个 setState 的过程：

1. setState 先把此次更新放到更新队列 updateQueue 里面，然后调用调度器开始做更新任务。

```js
// react/srs/ReactBaseClasses.js
Component.prototype.setState = function(partialState, callback) {
  this.updater.enqueueSetState(this, partialState, callback, "setState");
};
```

```js
// react-reconciler/src/ReactFiberClassComponent.js
enqueueSetState(inst, payload, callback) {
    ......
    enqueueUpdate(fiber, update);
    scheduleWork(fiber, expirationTime); // 开始进行调度
  },
```

2. 然后会执行到 workLoop 这个函数，其中 workInProgress 存储的就是一个 Fiber 节点的 alternate 属性。如果 workInProgress 存在就表示上次任务中断了，需要继续调度执行。
   （WIP 树就是一个缓冲，它在 Reconciliation 完毕后一次性提交给浏览器进行渲染。它可以减少内存分配和垃圾回收，WIP 的节点不完全是新的，比如某颗子树不需要变动，React 会克隆复用旧树中的子树。
   双缓存技术还有另外一个重要的场景就是异常的处理，比如当一个节点抛出异常，仍然可以继续沿用旧树的节点，避免整棵树挂掉。）

```js
function workLoopSync() {
  while (workInProgress !== null) {
    workInProgress = performUnitOfWork(workInProgress);
  }
}
```

3. 可以看到 performUnitOfWork 的逻辑就是对当前节点的 fiber 节点和 fiber 节点的 alternate 属性进行 diff 计算，也是旧树和新树的 diff。diff 完看看有没有子节点，如果没有子节点则把更新先提交到父节点。然后再看有没有兄弟节点，如果有则返回出去当作下次遍历的节点。如果还是没有，说明整个 fiber 树已经遍历完了，则进入到回溯过程，把所有的更新都集中到根节点进行更新真实 dom。（具体的 diff 算法过程等下一篇博客再写）

```js
function performUnitOfWork(unitOfWork: Fiber): Fiber | null {
  // The current, flushed, state of this fiber is the alternate. Ideally
  // nothing should rely on this, but relying on it here means that we don't
  // need an additional field on the work in progress.
  const current = unitOfWork.alternate;

  startWorkTimer(unitOfWork);
  setCurrentDebugFiberInDEV(unitOfWork);

  let next;
  if (enableProfilerTimer && (unitOfWork.mode & ProfileMode) !== NoMode) {
    startProfilerTimer(unitOfWork);
    next = beginWork(current, unitOfWork, renderExpirationTime);
    stopProfilerTimerIfRunningAndRecordDelta(unitOfWork, true);
  } else {
    next = beginWork(current, unitOfWork, renderExpirationTime);
  }

  resetCurrentDebugFiberInDEV();
  unitOfWork.memoizedProps = unitOfWork.pendingProps;
  if (next === null) {
    // If this doesn't spawn new work, complete the current work.
    next = completeUnitOfWork(unitOfWork);
  }

  ReactCurrentOwner.current = null;
  return next;
}
```
