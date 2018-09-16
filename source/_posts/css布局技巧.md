---
title: css布局技巧
date: 2018-07-21 17:53:21
tags: [css,布局技巧]
categories: [css,布局技巧]
---

## 清除浮动
```css
.clearfix:after{
  content: "020"; 
  display: block; 
  height: 0; 
  clear: both; 
  visibility: hidden;  
}
.clearfix {
  zoom: 1; 
}
```
