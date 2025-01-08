---
tags:
  - 知识笔记/基础知识
---
```html
// 声明文档编码
<meta charset="utf-8">
//页面关键字
<meta name="keywords" content="your keywords">
// 页面描述内容
<meta name="description" content="your description">
// 定义网页作者
<meta name="author" content="author,email address">
//移动端布局参数设置 这个最常用
<meta name="viewport" content="width=device-width, initial-scale=1.0">
content参数如下：
-   width viewport 宽度(数值/device-width)
-   height viewport 高度(数值/device-height)
-   initial-scale 初始缩放比例
-   maximum-scale 最大缩放比例
-   minimum-scale 最小缩放比例
-   user-scalable 是否允许用户缩放(yes/no)
//各个浏览器的meta标签有不同的自定义功能 包括 chrome 360 UC 等等

<!-- 优先使用最新的chrome版本 --> 
<meta http-equiv="X-UA-Compatible" content="chrome=1" /> 
<!-- 禁止自动翻译 -->
<meta name="google" value="notranslate">
```