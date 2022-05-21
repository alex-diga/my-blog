# 模拟实现 apply

## es5

```js
Function.prototype.IApply = function (context, params) {
  context = context || window;
  var fn = "$$applyFn";
  context[fn] = this;
  var result;
  if (!params) {
    result = context[fn]();
  } else {
    var args = [];
    for (var i = 0; i < params.length; i++) {
      args.push("params[" + i + "]");
    }
    result = eval("context[fn](" + args + ")");
  }
  delete context[fn];
  return result;
};
```

## es6

```js
Function.prototype.IApply = function (context, params) {
  context = context ? Object(context) : window;
  const fn = Symbol("applyFn");
  context[fn] = this;
  const result = context[fn](params);
  Reflect.deleteProperty(context, fn);
  return result;
};
```

## 参考

1. [JavaScript 深入之 call 和 apply 的模拟实现](https://github.com/mqyqingfeng/Blog/issues/11)
