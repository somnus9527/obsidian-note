---
tags:
  - 知识笔记/基础知识
---
#### 基本类型
>存储在栈上， 新建一个变量，用原来的变量赋值，实际是copy了一份数据，两者之间变更不会相互影响

1. Undefinded
2. Null
3. Number
4. String
5. Bool
6. Symbol
7. BigInt

#### 引用类型
>存储在堆上，变量持有的是数据在堆上的地址

>Object _包括常见的Array,Set，Function都是对象_

>当然对象也分函数对象和普通对象，区别在于函数对象才有prototype，而所有对象都有__proto__

