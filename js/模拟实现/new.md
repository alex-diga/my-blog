# 模拟实现 new

`new`会进行如下操作：

1. 创建一个空的简单 JavaScript 对象
2. 为步骤 1 新创建的对象添加属性**proto**，将该属性链接至构造函数的原型对象
3. 将步骤 1 新创建的对象作为 this 的上下文
4. 如果该函数没有返回对象，则返回 this

## 法 1

```js
function myNew(context, ...args) {
  const obj = new Object(); // 创建空的简单js对象
  obj.__proto__ = context.prototype; // 给空对象添加新属性 __proto__，并链接到构造函数的原型对象
  const result = context.apply(obj, args); // 把obj作为上下文的this，执行构造函数
  return result && typeof result === "object" ? result : obj; // 如果构造函数没有返回对象，则返回this
}
```

## 法 2

```js
function myNew() {
  const args = [].slice.call(arguments);
  const construtor = args.shift();
  const newObj = Object.create(construtor);
  const res = construtor.apply(newObj, args);
  return typeof res === "object" && res !== null ? res : newObj;
}
```

## 参考

[MDN-new 运算符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new)
