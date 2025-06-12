---
tags:
  - 知识笔记/基础知识
---
>[!Info] 一般常用的toLocaleString是Date对象上的，偶然发现的Number对象上的使用
>Number.prototype.toLocaleString有两个参数 _1.locales这个具体语言编码可以根据MDN查看2.options最常用参数为style,可以是decimal用于纯数字格式也是默认值，currency用于货币格式货币格式需要额外提供currency货币符号，常用的是人民币CNY 美元USD 欧元EUR。percent用于百分比格式_

>[!Example] 例子
>- 将 123456789转成三个数字加一个逗号的形式_一般用在金额展示_ `var number = 123456789; number.toLocaleString();`结果是'123,456,789'
>- `var number = 3500;number.toLocaleString('zh-CN',{style:'currency',currency:'CNY'});`结果是'￥3,500.00'
>- `var number = 0.1;number.toLocaleString('en-US',{style:'percent'})`结果是'10%'

