# Fiber 核心

react16 版本之后引入了 fiber，整个架构层面的 调度、协调、diff 算法以及渲染等都与 fiber 密切相关.

## fiber 出现的原因

### 出现前

在 react15 及之前 fiber 未出现时，React 的一系列执行过程例如生命周期执行、虚拟 dom 比较、dom 树的更新等都是同步的，一旦开始执行就不会中断，直到所有工作流程全部结束为止。
React 所有的状态更新是从根组件开始的，当应用组件树比较庞大时，一旦状态开始变更，组件树层层递归更新，js 主线程就不得不停止其他工作，用户的点击输入等交互事件、页面动画等都不会等到响应，体验会非常的差。

![函数堆栈的调用](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8e6738e56dad4ecd935ba9ea40ef9ca6~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

### 出现后

React 引入 fiber 这种数据结构，将更新渲染耗时长的大任务，分为许多的小片，每个小片的任务执行完成后，都先去执行其他高优先级的任务(用户输入点击事件、动画等)

fiber 分片模式下，浏览器主线程能够定期被释放，保证渲染的帧率，使页面能够及时响应高优先级任务，显得不会卡顿。

![函数堆栈的调用](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a96db90f08de4405a865b54c67e22e5f~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

## fiber 数据结构

fiber 是一种数据结构，每个 fiber 节点的内部，都保存了 dom 相关信息、fiber 树相关的引用、要更新的副作用等。

```ts
// packages/react-reconciler/src/ReactInternalTypes.js

export type Fiber = {|
  // 作为静态数据结构，存储节点 dom 相关信息
  tag: WorkTag, // 组件的类型，取决于 react 的元素类型
  key: null | string,
  elementType: any, // 元素类型
  type: any, // 定义与此fiber关联的功能或类。对于组件，它指向构造函数；对于DOM元素，它指定HTML tag
  stateNode: any, // 真实 dom 节点

  // fiber 链表树相关
  return: Fiber | null, // 父 fiber
  child: Fiber | null, // 第一个子 fiber
  sibling: Fiber | null, // 下一个兄弟 fiber
  index: number, // 在父 fiber 下面的子 fiber 中的下标

  ref:
    | null
    | (((handle: mixed) => void) & { _stringRef: ?string, ... })
    | RefObject,

  // 工作单元，用于计算 state 和 props 渲染
  pendingProps: any, // 本次渲染需要使用的 props
  memoizedProps: any, // 上次渲染使用的 props
  updateQueue: mixed, // 用于状态更新、回调函数、DOM更新的队列
  memoizedState: any, // 上次渲染后的 state 状态
  dependencies: Dependencies | null, // contexts、events 等依赖

  mode: TypeOfMode,

  // 副作用相关
  flags: Flags, // 记录更新时当前 fiber 的副作用(删除、更新、替换等)状态
  subtreeFlags: Flags, // 当前子树的副作用状态
  deletions: Array<Fiber> | null, // 要删除的子 fiber
  nextEffect: Fiber | null, // 下一个有副作用的 fiber
  firstEffect: Fiber | null, // 指向第一个有副作用的 fiber
  lastEffect: Fiber | null, // 指向最后一个有副作用的 fiber

  // 优先级相关
  lanes: Lanes,
  childLanes: Lanes,

  alternate: Fiber | null, // 指向 workInProgress fiber 树中对应的节点

  actualDuration?: number,
  actualStartTime?: number,
  selfBaseDuration?: number,
  treeBaseDuration?: number,
  _debugID?: number,
  _debugSource?: Source | null,
  _debugOwner?: Fiber | null,
  _debugIsCurrentlyTiming?: boolean,
  _debugNeedsRemount?: boolean,
  _debugHookTypes?: Array<HookType> | null,
|};
```

### dom 相关属性

#### `tag`

用于标记不同的 React 组件类型

#### `key`和`type`

用于 React diff 过程中确定 fiber 是否可以复用

1. key 为用户定义的唯一值；
2. type 为定义与此 fiber 关联的功能或类，对于组件它指向函数或者类本身，对于 DOM 它指向 HTML tag

#### `stateNode`

用于记录当前 fiber 所对应的真实 dom 节点或者当前虚拟组件的实例：

1. 为了实现 ref，2. 为了实现真实 dom 追踪

### 链表树相关属性

- return：指向父 fiber，若没有父 fiber 则为 null
- child： 指向第一个子 fiber，若没有任何子 fiber 则为 null
- sibling：指向下一个兄弟 fiber，若没有下一个兄弟 fiber 则为 null

![fiber 链表树结构](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dff49d5527094ed99d39f17618e9cbc1~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

### 副作用相关属性

#### `flags`

react 通过 flags 记录每个节点 diff 后需要变更的状态，例如 dom 的添加、替换、删除等

#### `Effect list`

在 render 阶段，react 对 fiber 树进行遍历，把每一个有副作用的 fiber 筛选出来，最后构建生成一个只带副作用的 Effect list 链表。和该链表相关的字段有 firstEffect、nextEffect 和 lastEffect。
在 commit 阶段，React 拿到 Effect list 链表中的数据后，根据每个 fiber 节点的 flags 类型，对相应的 dom 进行更改

## Fiber 执行原理

从根节点开始渲染和调度的过程可以分为 2 个阶段：render 阶段和 commit 阶段。

- render 阶段：这个阶段是可中断的，会找出所有节点的变更
- commit 阶段，这个阶段是不可中断的，会执行所有的变更

### render 阶段

```js
function beginUnitOfWork(currentFiber) {
  console.log("start");
}

function completeUnitOfWork(currentFiber) {
  let returnFiber = currentFiber.return;
  if (returnFiber) {
    if (!returnFiber.firstEffect) {
      returnFiber.firstEffect = currentFiber.firstEffect;
    }
    if (currentFiber.lastEffect) {
      if (returnFiber.lastEffect) {
        returnFiber.lastEffect.nextEffect = currentFiber.firstEffect;
      }
      returnFiber.lastEffect = currentFiber.lastEffect;
    }
    let effectTags = currentFiber.effectTags;
    if (effectTags) {
      if (returnFiber.lastEffect) {
        returnFiber.lastEffect.nextEffect = currentFiber;
      }
      returnFiber.lastEffect = currentFiber;
    }
  }
}

function performUnitOfWork(currentFiber) {
  beginUnitOfWork(currentFiber);
  if (currentFiber.child) {
    return currentFiber.child;
  }
  while (currentFiber) {
    completeUnitOfWork(currentFiber);
    if (currentFiber.sibling) {
      return currentFiber.sibling;
    }
    currentFiber = currentFiber.return;
  }
}

function workloop(deadline) {
  let shouldYield = false;
  while (!shouldYield && nextUnitOfWork) {
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    shouldYield = deadline.timeRemaining() < 1;
  }
  if (!nextUnitOfWork && workInProgressRoot) {
    commitRoot();
  }
  requestIdleCallback(workloop, { timeout: 1000 });
}
```

### commit 阶段

```js
function commitRoot() {
  let currentFiber = workInProgressRoot.firstEffect;
  while (currentFiber) {
    commitWork(currentFiber);
    currentFiber = currentFiber.nextEffect;
  }
  currentRoot = workInProgressRoot;
  workInProgressRoot = null;
}

function commitWork(currentFiber) {
  if (!currentFiber) return;
  let returnFiber = currentFiber.return;
  let returnDOM = returnFiber.stateNode;
  if (currentFiber.effectTag === INSERT) {
    returnDOM.appendChild(currentFiber.stateNode);
  } else if (currentFiber.effectTag === DELETE) {
    returnDOM.removeChild(currentFiber.stateNode);
  } else if (currentFiber.effectTag === UPDATE) {
    if (currentFiber.type === ELEMENT_TYPE) {
      if (currentFiber.alternate.props.text !== currentFiber.props.text) {
        currentFiber.stateNode.textContent = currentFiber.props.text;
      }
    }
  }
  currentFiber.effectTag = null;
}
```

## 参考

1. [走进 React Fiber 的世界](https://juejin.cn/post/6943896410987659277)
2. [深入理解 fiber](https://juejin.cn/post/7016512949330116645)
