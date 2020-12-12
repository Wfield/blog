## React fiber
每个工作单元（fiber）不仅存储节点信息（stateNode），还通过child和sibling表征当前工作单元的下一个工作单元，return表示处理完成后返回结果所要合并的目标，通常指向父节点。整个结构是一个*链表*。
**fiber 节点的数据结构：**
```
 {
 ...
  // 浏览器环境下指 DOM 节点
  stateNode: any,

  // 形成列表结构
  return: Fiber | null, // 父节点
  child: Fiber | null,
  sibling: Fiber | null,

  // 更新相关
  pendingProps: any,  // 新的 props
  memoizedProps: any,  // 旧的 props
  // 存储 setState 中的第一个参数
  updateQueue: UpdateQueue<any> | null,
  memoizedState: any, // 旧的 state

  // 调度相关
  expirationTime: ExpirationTime,  // 任务过期时间

  // 大部分情况下每个 fiber 都有一个替身 fiber
  // 在更新过程中，所有的操作都会在替身上完成，当渲染完成后，
  // 替身会代替本身
  alternate: Fiber | null,

  // 先简单认为是更新 DOM 相关的内容
  effectTag: SideEffectTag, // 指这个节点需要进行的 DOM 操作
  // 以下三个属性也会形成一个链表
  nextEffect: Fiber | null, // 下一个需要进行 DOM 操作的节点
  firstEffect: Fiber | null, // 第一个需要进行 DOM 操作的节点
  lastEffect: Fiber | null, // 最后一个需要进行 DOM 操作的节点，同时也可用于恢复任务
  ....
}
```
#### type and key

fiber的type和key与React元素的作用相同。 （实际上，当从一个元素创建一个fiber时，这两个字段直接被复制过来。）

fiber的type描述了它对应的组件。对于复合组件，类型是函数或类组件本身。对于host组件（div，span等），类型是一个字符串。

从概念上讲，type是函数（如在v = f（d）中），其执行被栈帧跟踪。

随着type的不同，在reconciliation期间使用key来确定fiber是否可以重新使用。

#### child and sibling

这些字段指向其他fiber，描述fiber的递归树状结构。

子fiber对应于组件渲染方法返回的值。所以在下面的例子中

```
function Parent() {
  return <Child />
}
```

Parent的child fiber对应于Child。

兄弟领域说明了渲染返回多个children的情况（Fiber中的一个新特性）：

```
function Parent() {
  return [<Child1 />, <Child2 />]
}
```

child的fiber形成一个单一的链表，head是第一个child。所以在这个例子中，Parent的child是Child1，而Child1的兄弟是Child2。

#### return

return fiber是程序在处理完当前fiber后返回的fiber。它在概念上与堆栈帧的返回地址相同。它也可以被认为是parent fiber。

如果fiber具有多个子fiber，则每个子fiber的return fiber是parent。所以在前面的例子中，Child1和Child2的return fiber是Parent。

#### pendingProps 和 memoizedProps

从概念上讲，props是一个函数的arguments。一个fiber的pendingProps在执行开始时被设置，memoizedProps被设置在最后。

当传入的pendingProps等于memoizedProps时，它表明fiber的先前输出可以被重复使用，避免不必要的工作。

#### pendingWorkPriority

一个数字，表示fiber所代表的工作的优先级。 ReactPriorityLevel模块列出了不同的优先级以及它们代表的内容。

除NoWork为0外，较大的数字表示较低的优先级

scheduler使用优先级字段来搜索要执行的下一个工作单元。

#### 备用

**flush**

```
flush fiber是将其输出呈现在屏幕上。

```

**work-in-progress**

```
尚未完成的fiber;从概念上说，一个尚未返回的堆栈帧。

```

在任何时候，一个组件实例最多只有两条fiber对应：当前的，flushed fiber和work-in-progress fiber。

当前fiber的交替是work-in-progress，work-in-progress的交替是当前的fiber。

使用名为cloneFiber的函数，可以创建一个fiber的替代品。 cloneFiber不会总是创建一个新的对象，而是尝试重用fiber的备用，如果它存在，最小化分配。

