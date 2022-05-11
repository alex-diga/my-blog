# react diff 核心算法

## 40 行代码实现 React 核心 Diff 算法

Diff 砍掉处理常见情况的算法，保留处理不常见情况的算法

```ts
type Flag = "Placement" | "Deletion";

type Node1 = {
  key: string;
  flag?: Flag;
  index?: number;
};

type NodeList1 = Node1[];

function diff(before: NodeList1, arfer: NodeList1): NodeList1 {
  const result: NodeList1 = [];
  let lastPlacedIndex = 0;
  const beforeMap = new Map<string, Node1>();
  before.forEach((node, index) => {
    node.index = index;
    beforeMap.set(node.key, node);
  });

  for (let i = 0; i < arfer.length; i++) {
    const afterNode = arfer[i];
    afterNode.index = i;
    const beforeNode = beforeMap.get(afterNode.key);
    if (beforeNode) {
      beforeMap.delete(beforeNode.key);
      const oldIndex = beforeNode.index;
      if (oldIndex < lastPlacedIndex) {
        afterNode.flag = "Placement";
        result.push(afterNode);
        continue;
      } else {
        lastPlacedIndex = oldIndex;
      }
    } else {
      afterNode.flag = "Placement";
      result.push(afterNode);
    }
  }

  beforeMap.forEach((node) => {
    node.flag = "Deletion";
    result.push(node);
  });
  return result;
}

// 更新前
const before = [{ key: "a" }, { key: "b" }, { key: "c" }, { key: "d" }];
// 更新后
const after = [{ key: "d" }, { key: "c" }, { key: "b" }, { key: "a" }];

diff(before, after);

// diff 结果
// [
//   { key: 'c', index: 1, flag: 'Placement' },
//   { key: 'b', index: 2, flag: 'Placement' },
//   { key: 'a', index: 3, flag: 'Placement' }
// ]
```

## 参考

1. [40 行代码实现 React 核心 Diff 算法](https://juejin.cn/post/7086634898953338911)
