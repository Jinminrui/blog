---
title: React Hooks概览
tags: react
abbrlink: 52968
date: 2022-05-09 16:38:34
---

## 为什么引入Hooks
* 在组件之间复用状态逻辑很难; 
* 复杂组件变得难以理解;  
* 难以理解的 class;
为了解决这些实际开发痛点, 引入了Hook。

<!-- more -->

## Hook与Fiber
Hook最终也是为了控制fiber节点的状态和副作用. 从fiber视角, 状态和副作用相关的属性如下，使用Hook的任意一个api, 最后都是为了控制上述这几个fiber属性。
```ts
export type Fiber = {|
  // 1. fiber节点自身状态相关
  pendingProps: any,
  memoizedProps: any,
  updateQueue: mixed,
  memoizedState: any,

  // 2. fiber节点副作用(Effect)相关
  flags: Flags,
  nextEffect: Fiber | null,
  firstEffect: Fiber | null,
  lastEffect: Fiber | null,
|};
```

## Hooks 的构造
在一个函数组件中，如果使用了 useState 或者 useEffect等 Hook api，就会就会创建一个与之对应的Hook对象。
而useState, useEffect在fiber初次构造时分别对应 [mountState](https://github.com/facebook/react/blob/v17.0.2/packages/react-reconciler/src/ReactFiberHooks.old.js#L1113-L1136) 和 [mountEffect->mountEffectImpl](https://github.com/facebook/react/blob/v17.0.2/packages/react-reconciler/src/ReactFiberHooks.old.js#L1193-L1248) 。
```ts
function mountState<S>(
  initialState: (() => S) | S,
): [S, Dispatch<BasicStateAction<S>>] {
  const hook = mountWorkInProgressHook();
  // ...省略部分讨论
  return [hook.memoizedState, dispatch];
}

function mountEffectImpl(fiberFlags, hookFlags, create, deps): void {
  const hook = mountWorkInProgressHook();
  // ...省略部分不讨论
}
```
无论 useState，useEffect ，内部都通过mountWorkInProgressHook 创建一个 hook 。
```ts
function mountWorkInProgressHook(): Hook {
  const hook: Hook = {
    memoizedState: null,
    baseState: null,
    baseQueue: null,
    queue: null,
    next: null,
  };

  if (workInProgressHook === null) {
    // 链表中首个hook
    currentlyRenderingFiber.memoizedState = workInProgressHook = hook;
  } else {
    // 将hook添加到链表末尾
    workInProgressHook = workInProgressHook.next = hook;
  }
  return workInProgressHook;
}
```
mountWorkInProgressHook 创建Hook，以链表的形式将 Hook 对象挂在fiber.memoizedState上。

**无论状态 Hook 或副作用 Hook 都按照调用顺序存储在 fiber.memoizedState链表中。**
![](mount-fiber-memoizedstate.png)

## Hook 数据结构
从定义来看，Hook 对象共有5个属性：
1. **hook.memoizedState**: 保持在内存中的局部状态.
2. **hook.baseState**: hook.baseQueue中所有update对象合并之后的状态.
3. **hook.baseQueue**: 存储update对象的环形链表, 只包括高于本次渲染优先级的update对象.
4. **hook.queue**: 存储update对象的环形链表, 包括所有优先级的update对象.
5. **hook.next**: next指针, 指向链表中的下一个hook.
![](hook-linkedlist.png)
所以Hook是一个链表, 单个Hook拥有自己的状态hook.memoizedState和自己的更新队列hook.queue。

## Hooks状态持久化
* 以双缓冲技术为基础, 将 current.memoizedState 按照顺序克隆到了workInProgress.memoizedState中。
* Hook经过了一次克隆, 内部的属性(hook.memoizedState等)都没有变动, 所以其状态并不会丢失。