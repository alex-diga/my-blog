---
title: BFC
date: 2022-06-09
---

BFC(Block Formatting Context)即块格式化上下文，是页面可视 CSS 渲染的一部分，是块级盒子的布局过程发生的区域，也是浮动元素与其他元素交互的区域

具有BFC特性的元素可以看做是隔离了的独立容器，容器里面的元素不会在布局上影响容器外面的元素。

以下方式会创建 BFC:

1. 根元素
2. 浮动元素
3. 绝对定位元素
4. 行内块元素
5. overflow 值不为 visible、clip 的块元素
6. diplay 值为 flow-root 的元素
7. contain 值为 layout、content 或 paint 的元素
8. 弹性元素

格式化上下文影响布局：

1. 包含内部浮动
2. 排出外部浮动
3. 阻止外边距重叠

## 参考

1. [MDN-BFC](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context)