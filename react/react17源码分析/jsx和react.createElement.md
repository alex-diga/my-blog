# jsx 和 react.createElement

## react.createElement

用于 jsx 转换

React.createElement 接收三个或以上参数：

- type: 要创建的 React 元素类型，可以是标签名称字符串，如`div`和`span`等、也可以是 React 组件类型（class 组件或者函数组件）、或者是 React Fragmant 类型；
- config: 写在标签上的属性集合，js 对象格式，若标签上未添加任何属性则为 null；
- children: 从第三个参数开始后的参数为当前创建的 React 元素的子元素，每个参数的类型，若是当前元素节点的 textContent 则为字符串类型，否则为新的 React.createElement 创建的元素。

```ts
export function createElement(type, config, children) {
  let propName;

  // 记录标签上的属性集合
  const props = {};

  let key = null;
  let ref = null;
  let self = null;
  let source = null;

  // config 不为 null 时，说明标签上有属性，将属性添加到 props 中
  // 其中，key 和 ref 为 react 提供的特殊属性，不加入到 props 中，而是用 key 和 ref 单独记录
  if (config != null) {
    if (hasValidRef(config)) {
      // 有合法的 ref 时，则给 ref 赋值
      ref = config.ref;

      if (__DEV__) {
        warnIfStringRefCannotBeAutoConverted(config);
      }
    }
    if (hasValidKey(config)) {
      // 有合法的 key 时，则给 key 赋值
      key = "" + config.key;
    }

    // self 和 source 是开发环境下对代码在编译器中位置等信息进行记录，用于开发环境下调试
    self = config.__self === undefined ? null : config.__self;
    source = config.__source === undefined ? null : config.__source;
    // 将 config 中除 key、ref、__self、__source 之外的属性添加到 props 中
    for (propName in config) {
      if (
        hasOwnProperty.call(config, propName) &&
        !RESERVED_PROPS.hasOwnProperty(propName)
      ) {
        props[propName] = config[propName];
      }
    }
  }

  // 将子节点添加到 props 的 children 属性上
  const childrenLength = arguments.length - 2;
  if (childrenLength === 1) {
    // 共 3 个参数时表示只有一个子节点，直接将子节点赋值给 props 的 children 属性
    props.children = children;
  } else if (childrenLength > 1) {
    // 3 个以上参数时表示有多个子节点，将子节点 push 到一个数组中然后将数组赋值给 props 的 children
    const childArray = Array(childrenLength);
    for (let i = 0; i < childrenLength; i++) {
      childArray[i] = arguments[i + 2];
    }
    // 开发环境下冻结 childArray，防止被随意修改
    if (__DEV__) {
      if (Object.freeze) {
        Object.freeze(childArray);
      }
    }
    props.children = childArray;
  }

  // 如果有 defaultProps，对其遍历并且将用户在标签上未对其手动设置属性添加进 props 中
  // 此处针对 class 组件类型
  if (type && type.defaultProps) {
    const defaultProps = type.defaultProps;
    for (propName in defaultProps) {
      if (props[propName] === undefined) {
        props[propName] = defaultProps[propName];
      }
    }
  }

  // key 和 ref 不挂载到 props 上
  // 开发环境下若想通过 props.key 或者 props.ref 获取则 warning
  if (__DEV__) {
    if (key || ref) {
      const displayName =
        typeof type === "function"
          ? type.displayName || type.name || "Unknown"
          : type;
      if (key) {
        defineKeyPropWarningGetter(props, displayName);
      }
      if (ref) {
        defineRefPropWarningGetter(props, displayName);
      }
    }
  }

  // 调用 ReactElement 并返回
  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props
  );
}
```

React.createElement 做的事情主要有:

- 解析 config 参数是否有 key、ref、\_self、 \_source 属性，若存在分别赋值给 key、ref、self 和 source，将剩余的属性解析挂载到 props 上；
- 除了 type 和 config 外的其余参数，挂载到 props.children 上
- 针对类组件，如果 type.defaultProps 存在，遍历 type.defaultProps 的属性，如果 props 不存在该属性，则添加到 props 上；
- 将 type、key、ref、self、sourece、props 等信息，调用 ReactElement 函数创建虚拟 dom，ReactElement 主要是在开发环境下通过`Object.defineProperty`将 \_store、\_self、\_source 设置为不可枚举，提高 element 比较时的性能；

```js
const ReactElement = function(type, key, ref, self, source, owner, props) {
const element = {
// 用于表示是否为 ReactElement
$$typeof: REACT_ELEMENT_TYPE,

    // 用于创建真实 dom 的相关信息
    type: type,
    key: key,
    ref: ref,
    props: props,

    _owner: owner,

};

if (**DEV**) {
element.\_store = {};

    // 开发环境下将 _store、_self、_source 设置为不可枚举，提高 element 的比较性能
    Object.defineProperty(element._store, 'validated', {
      configurable: false,
      enumerable: false,
      writable: true,
      value: false,
    });

    Object.defineProperty(element, '_self', {
      configurable: false,
      enumerable: false,
      writable: false,
      value: self,
    });

    Object.defineProperty(element, '_source', {
      configurable: false,
      enumerable: false,
      writable: false,
      value: source,
    });
    // 冻结 element 和 props，防止被手动修改
    if (Object.freeze) {
      Object.freeze(element.props);
      Object.freeze(element);
    }

}

return element;
};
```

![createElement 流程](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d5195bfc57b2438a9e34eee55573b48e~tplv-k3u1fbpfcp-zoom-in-crop-mark:1304:0:0:0.awebp?)

## 参考

[jsx 转换及 React.createElement](https://juejin.cn/post/7015855371847729166)
