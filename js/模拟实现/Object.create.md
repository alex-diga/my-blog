# 模拟实现 Object.create

```js
function create(obj) {
  function F() {}
  F.prototype = obj;
  return new F();
}
```
