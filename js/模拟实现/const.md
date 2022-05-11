# 模拟实现 const

es5 环境模拟实现 const 部分功能

```js
function __const(data, value) {
  window.data = value;
  Object.defineProperty(window, data, {
    enumerable: false,
    configurable: false,
    get: function () {
      return value;
    },
    set: function (val) {
      if (val !== value) {
        throw new TypeError("Assignment to constant variable.");
      } else {
        return value;
      }
    },
  });
}

__const("a", 10);
console.log(a); // 10
a = 20; // Uncaught TypeError: Cannot redefine property: a
```

## 参考

1. [ES5 环境下实现一个 const](https://juejin.cn/post/6844903746514714632)
