---
tags:
  - 知识笔记/基础知识
---
1. 字面量
2. new Object
3. Object.create(proto, configure)

>[!Warning] 
>1,2都会把新对象的__proto__原型链挂到Object.prototype上，这两种方式没有区别; 而3的原型链由传入的proto参数决定，如果传null那么新对象的__proto__原型链就是空的，但是这种方式的优势就是可以配置，比如配置**writable，configurable，value**

