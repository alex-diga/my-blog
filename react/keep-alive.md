---
title: 实现 keep-alive 效果
---

```typescript
import React, { useRef, useLayoutEffect } from "react";
import ReactDOM from "react-dom";

type IProps = {
  active: boolean;
  children: React.ReactNode;
};

const Conditional: React.FC<IProps> = (props) => {
  const { current: targetElement } = useRef(document.createElement("div"));
  const containerRef = useRef<HTMLDivElement>(null);
  const activateRef = useRef(false);
  activateRef.current = activateRef.current || props.active;
  useLayoutEffect(() => {
    if (!activateRef.current) {
      return;
    }
    if (props.active) {
      containerRef.current.appendChild(targetElement);
    } else {
      try {
        containerRef.current.removeChild(targetElement);
      } catch (e) {}
    }
  }, [props.active]);

  return (
    <>
      <div ref={containerRef} />
      {props.active && ReactDOM.createPortal(props.children, targetElement)}
    </>
  );
};
```

## 参考

1. [React 实现 keep-alive 效果](https://juejin.cn/post/7098601924731093022)
2. [50 行代码实现页面状态保持 keepalive](https://juejin.cn/post/7088705614603354142)
