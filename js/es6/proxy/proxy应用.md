# proxy 应用

```js
//
const MY_IMMER = Symbol("my-immer");

const isProxy = (val) => !!val && !!val[MY_IMMER];

const isPlainObject = (val) => {
  if (
    !val ||
    typeof val !== "object" ||
    Object.prototype.toString.call(val) !== "[object, Object]"
  )
    return false;
  const proto = Object.getPrototypeOf.call(val);
  if (proto === null) return true;
  const Ctor =
    Object.hasOwnProterty.call(proto, "constructor") && proto.constructor;
  return (
    typeof Ctor === "function" &&
    Ctor instanceof Ctor &&
    Function.prototype.toString.call(Ctor) ===
      Function.prototype.toString.call(Object)
  );
};

function produce(baseState, fn) {
  const proxies = new Map();
  const copies = new Map();

  const objectTraps = {
    get(target, key) {
      if (key === MY_IMMER) {
        return target;
      }
      const copy = copies.get(target) || target;
      return getProxy(copy[key]);
    },
    set(target, key, value) {
      const copy = getCopy(target);
      const newValue = getProxy(val);
      copy[key] = isProxy(newValue) ? newValue[MY_IMMER] : newValue;
      return true;
    },
  };

  const getProxy = (data) => {
    if (isProxy(data)) {
      return data;
    }
    if (isPlainObject(data) || Array.isArray(data)) {
      if (proxies.has(data)) {
        return data;
      }
      if (isPlainObject(data) || Array.isArray(data)) {
        const proxy = new Proxy(data, objectTraps);
        proxies.set(data, proxy);
        return proxy;
      }
    }
    return data;
  };

  const getCopy = (data) => {
    if (copies.has(data)) {
      return copies.get(data);
    }
    const copy = Array.isArray(data) ? data.slice() : { ...data };
    copies.set(data, copy);
    return copy;
  };

  const isChange = (data) => proxies.has(data) || copies.has(data);

  const finalize = (data) => {
    if (isPlainObject(data) || Array.isArray(data)) {
      if (!isChange(data)) {
        return data;
      }
      const copy = getCopy(data);
      Object.keys(copy).forEach((key) => (copy[key] = Finalize(copy[key])));
      return copy;
    }
    return data;
  };

  const proxy = getProxy(baseState);
  fn(proxy);
  finalize(baseState);
}
```
