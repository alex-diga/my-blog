---
title: flex布局
date: 2022-06-09
---

## flex 元素上的属性

为了更好的控制 flex 元素，有三个属性可以作用于它们：

- flex-grow
- flex-shrink
- flex-basis

### flex-grow

`flex-grow`属性规定了 flex 容器中分配剩余空间的相对比例。主尺寸是项的宽度或者高度，取决于`flex-direction`的值
值为正数，负值无效，默认为`0`

详细可看[MDN-flex-grow](https://developer.mozilla.org/zh-CN/docs/Web/CSS/flex-grow)

### flex-shrink

`flex-shrink`属性指定了 flex 元素的收缩规则，flex 元素仅在默认宽度之和大于容器的时候才会发生收缩，其收缩的大小是依据 flex-shrink 的值

值为正数，负值不被允许，默认为`1`

### flex-basis

`flex-basis`属性指定了flex元素在主轴上的初始大小。

## 参考

1. [MDN-flex 布局的基本概念](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Flexible_Box_Layout/Basic_Concepts_of_Flexbox)
2. [flex 属性深入理解](https://www.zhangxinxu.com/wordpress/2019/12/css-flex-deep/)
