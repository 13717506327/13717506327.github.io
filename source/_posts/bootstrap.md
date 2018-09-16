---
title: bootstrap
date: 2018-07-24 17:10:05
tags: [css,bootstrap]
categories: [css,bootstrap]
---

### 移动设备优先
添加元数据标签
```html
<meta name="viewport" content="width=device-width, initial-scale=1">

<!-- 禁止 缩放 -->
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
```

### Normalize.css
为了增强跨浏览器表现的一致性，我们使用了 [Normalize.css](http://necolas.github.io/normalize.css/)。

### 布局容器
`Bootstrap` 需要为页面内容和栅格系统包裹一个 `.container` 容器 和 `.container-fluid`(类用于 100% 宽度，占据全部视口（`viewport`）的容器。)。我们提供了两个作此用处的类。注意，由于 padding 等属性的原因，这两种 容器类不能互相嵌套。

### 栅格系统
> `Bootstrap` 提供了一套响应式、移动设备优先的流式栅格系统，随着屏幕或视口（`viewport`）尺寸的增加，系统会自动分为最多`12`列。它包含了易于使用的预定义类，还有强大的`mixin` 用于生成更具语义的布局。

栅格系统用于通过一系列的行（`row`）与列（`column`）的组合来创建页面布局，你的内容就可以放入这些创建好的布局中。下面就介绍一下 `Bootstrap` 栅格系统的工作原理：
* “行（`row`）”必须包含在 `.container` （固定宽度）或 `.container-fluid`（100% 宽度）中
* 通过“行（`row`）”在水平方向创建一组“列（`column`）”。
* 你的内容应当放置于“列（`column`）”内，并且，只有“列（`column`）”可以作为行（`row`）”的直接子元素。
* 类似 `.row` 和 `.col-xs-4` 这种预定义的类，可以用来快速创建栅格布局。Bootstrap 源码中定义的 mixin 也可以用来创建语义化的布局。

### 媒体查询
```javascript
  /* 超小屏幕（手机，小于 768px） */
  /* 没有任何媒体查询相关的代码，因为这在 Bootstrap 中是默认的（还记得 Bootstrap 是移动设备优先的吗？） */

  /* 小屏幕（平板，大于等于 768px） */
  @media (min-width: @screen-sm-min) { ... }

  /* 中等屏幕（桌面显示器，大于等于 992px） */
  @media (min-width: @screen-md-min) { ... }

  /* 大屏幕（大桌面显示器，大于等于 1200px） */
  @media (min-width: @screen-lg-min) { ... }
```
我们偶尔也会在媒体查询代码中包含 `max-width` 从而将 CSS 的影响限制在更小范围的屏幕大小之内。
```javascript
  @media (max-width: @screen-xs-max) { ... }
  @media (min-width: @screen-sm-min) and (max-width: @screen-sm-max) { ... }
  @media (min-width: @screen-md-min) and (max-width: @screen-md-max) { ... }
  @media (min-width: @screen-lg-min) { ... }
```

| 超小屏幕 手机 (<768px) | 小屏幕 平板 (≥768px) |  中等屏幕 桌面显示器 (≥992px) | 中等屏幕 桌面显示器 (≥992px) |
| --------------------- | ------------------- | ---------------------------- | -------------------------- |
| .col-xs- | .col-sm- | .col-md- | .col-lg- |
