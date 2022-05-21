# 模拟实现 call

## es5

```js
Function.prototype.ICall = function (context) {
  context = context || window;
  var fn = "$$callFn";
  context[fn] = this;
  var args = [];
  for (var i = 0; i < arguments.length; i++) {
    args.push("arguments[" + i + "]");
  }
  var result = eval("context[fn](" + args + ")");
  delete context[fn];
  return result;
};
```

## es6

```js
Function.prototype.ICall = function (context, ...args) {
  context = context ? Object(context) : window;
  const fn = Symbol("callFn");
  context[fn] = this;
  const result = context[fn](...args);
  Reflect.deleteProperty(context, fn);
  return result;
};
```

## 参考

1. [JavaScript 深入之 call 和 apply 的模拟实现](https://github.com/mqyqingfeng/Blog/issues/11)
