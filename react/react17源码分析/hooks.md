---
title: React hook 源码
---

## Hook 数据结构

每一个 hooks 方法都会生成一个类型为 Hook 的对象，用来存储一些信息，函数组件 fiber 中的 memoizedState 会存储 hooks 链表，每个链表节点的结构是 Hook

### Hook 类型

```ts
export type Hook = {|
  memoizedState: any,
  baseState: any,
  baseQueue: Update<any, any> | null,
  queue: UpdateQueue<any, any> | null,
  next: Hook | null,
|};
```

#### 示例

```js
const [name, setName] = useState("tom");
const [age, setAge] = useState(23);
```

实际的 Hook 结构:

```js
{
  memoizedState: 'tom',
  baseState: 'tom',
  baseQueue: null,
  queue: null,
  next: {
    memoizedState: 23,
    baseState: 23,
    baseQueue: null,
    queue: null,
  },
};
```

> 不同的 hooks 方法，memoizedState 存储的内容不同

- useState: state
- useEffect: effect 对象
- useMemo、useCallback: [callback, deps]
- useRef: { current: xxx }

### Update & UpdateQueue

`Update` 和`UpdateQueue`是存储`useState`的 state 和`useReducer`的 reducer 相关内容的数据结构

```ts
type Update<S, A> = {|
  lane: Lane,
  action: A,
  eagerReducer: ((S, A) => S) | null,
  eagerState: S | null,
  next: Update<S, A>,
  priority?: ReactPriorityLevel,
|};

type UpdateQueue<S, A> = {|
  pending: Update<S, A> | null,
  dispatch: (A => mixed) | null,
  lastRenderedReducer: ((S, A) => S) | null,
  lastRenderedState: S | null,
|};
```

每次调用`setState`或者`useReducer`的 dispatch 时，都会生成一个 Update 类型的对象，并将其添加到 UpdateQueue 队列中

#### 示例

```js
const [name, setName] = useState("tom");
setName("lucas");
```

hook 数据:

```js
{
  memoizedState: 'tom',
  baseState: 'tom',
  baseQueue: null,
  queue: {
    pending: {
      lane: 1,
      action: 'lucas',
      eagerState: 'lucas',
      // ...
    },
    lastRenderedState: 'tom',
    // ...
  },
  next: null,
};
```

最后 react 会遍历 UpdateQueue 中的每个 Update 去进行更新

### Effect

```ts
export type Effect = {|
  tag: HookFlags, // 标记是否有 effect 需要执行
  create: () => (() => void) | void, // 回调函数
  destroy: (() => void) | void, // 销毁时触发的回调
  deps: Array<mixed> | null,  // 依赖的数组
  next: Effect, // 下一个要执行的 Effect
|};
```

#### 示例

```js
useEffect(() => {
  console.log("hello");
  return () => {
    console.log("bye");
  };
}, []);
```

effect 的 hook 数据:

```js
{
  memoizedState: {
    create: () => { console.log('hello') },
    destroy: () => { console.log('bye') },
    deps: [],
    // ...
  },
  baseState: null,
  baseQueue: null,
  queue: null,
  next: null,
}
```

## 参考

1. [搞懂 hooks 原理](https://juejin.cn/post/7023568411963686920)
