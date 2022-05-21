# 模拟实现 bind

```js
Function.prototype.IBind = function (context) {
  if (typeof this !== "function") {
    throw new TypeError("bind with function");
  }
  const self = this;
  context = context ? Object(context) : window;
  const args = Array.prototype.slice.call(arguments, 1);
  const FNop = function () {};
  const FBound = function () {
    const bindArgs = Array.prototype.slice.call(arguments);
    return self.apply(
      self instanceof FNop ? self : context,
      args.concat(bindArgs)
    );
  };
  FNop.prototype = self.prototype;
  FBound.prototype = new FNop();
  return FBound;
};
```

## 参考

1. [JavaScript 深入之 bind 的模拟实现](https://github.com/mqyqingfeng/Blog/issues/12)
