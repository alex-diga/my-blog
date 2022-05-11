# 模拟实现 promise

## Promise A+

```js
const pending = "PENDING";
const fulfilled = "FULFILLED";
const rejected = "REJECTED";

function resolvePromise(promise2, x, resolve, reject) {
  if (promise2 === x) {
    return reject(
      new TypeError("chaining cycle detected for promise #<promise>")
    );
  }
  if (x instanceof MyPromise) {
    x.then(resolve, reject);
  } else {
    resolve(x);
  }
}

class MyPromise {
  status = pending;
  value = null;
  reason = null;
  onFulfilledCallbacks = [];
  onRejectedCallbacks = [];
  constructor(executor) {
    try {
      executor(this._resolve, this._reject);
    } catch (error) {
      this._reject(error);
    }
  }

  _resolve = (value) => {
    if (this.status === pending) {
      this.status = fulfilled;
      this.value = value;
      while (this.onFulfilledCallbacks.length) {
        this.onFulfilledCallbacks.split()(value);
      }
    }
  };

  _reject = (reason) => {
    if (this.status === pending) {
      this.status = rejected;
      this.reason = reason;
      while (this.onRejectedCallbacks.length) {
        this.onRejectedCallbacks.split()(reason);
      }
    }
  };

  then(onFulfilled, onRejected) {
    const realOnFulfilled =
      typeof onFulfilled === "function" ? onFulfilled : (value) => value;
    const realOnRejected =
      typeof onRejected === "function"
        ? onRejected
        : (reason) => {
            throw reason;
          };
    const promise2 = new MyPromise((resolve, reject) => {
      const fulfilledMicrotask = () => {
        queueMicrotask(() => {
          try {
            const x = realOnFulfilled(this.value);
            resolvePromise(promise2, x, resolve, reject);
          } catch (error) {
            reject(error);
          }
        });
      };
      const rejectedMicrotask = () => {
        queueMicrotask(() => {
          try {
            const x = realOnRejected(this.reason);
            resolvePromise(promise2, x, resolve, reject);
          } catch (error) {
            reject(error);
          }
        });
      };
      if (this.status === fulfilled) {
        fulfilledMicrotask();
      } else if (this.status === rejected) {
        rejectedMicrotask();
      } else if (this.status === pending) {
        this.onFulfilledCallbacks.push(fulfilledMicrotask);
        this.onRejectedCallbacks.push(rejectedMicrotask);
      }
    });
    return promise2;
  }

  static resolve(parameter) {
    if (parameter instanceof MyPromise) {
      return parameter;
    }
    return new MyPromise((resolve) => {
      resolve(parameter);
    });
  }

  static reject(reason) {
    return new MyPromise((_, reject) => {
      reject(reason);
    });
  }

  catch(onRejected) {
    return this.then(null, onRejected);
  }

  finally(callback) {
    return this.then(
      (value) => {
        return MyPromise.resolve(callback()).then(() => value);
      },
      (error) => {
        return MyPromise.resolve(callback()).then(() => {
          throw error;
        });
      }
    );
  }

  all(promises) {
    return new MyPromise((resolve, reject) => {
      const len = promises.length;
      const result = [];
      if (len === 0) {
        return resolve(result);
      }
      let index = 0;
      for (let i = 0; i < len; i++) {
        MyPromise.resolve(promises[i])
          .then((data) => {
            result[i] = data;
            index++;
            if (index === len) {
              resolve(result);
            }
          })
          .catch((error) => reject(error));
      }
    });
  }

  race(promises) {
    return new MyPromise((resolve, reject) => {
      const len = promises.length;
      if (len === 0) {
        return;
      }
      for (let i = 0; i < len; i++) {
        MyPromise.resolve(promises[i])
          .then((data) => {
            resolve(data);
            return;
          })
          .catch((error) => {
            reject(error);
            return;
          });
      }
    });
  }
}
```

## 参考

1. [MDN-Promise](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)
2. [Promise 对象](https://es6.ruanyifeng.com/#docs/promise)
3. [深入分析 Promise 实现细节](https://juejin.cn/post/6945319439772434469)
4. [45 道 Promise 面试题](https://juejin.cn/post/6844904077537574919)
5. [Promise 之问](https://juejin.cn/post/6844904004007247880#heading-36)
