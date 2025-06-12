---
tags:
  - 知识笔记/基础知识
---
#### 方式1
```javascript
JSON.parse(JSON.stringify(obj))
```
>[!Warning] 
>- 会忽略 undefined、function、Symbol
>- 对象内部有循环引用会报错

#### 方式2
```javascript
MessageChannel
```
>[!Warning] 可以正确复制 undefined 以及循环引用, 但是函数和 Symbol 属性还是会报错
> ```javascript
> function deepClone(obj) {
>   return new Promise((resolve,reject) => {
>     const {port1: receivePort, port2: sendPort} = new MessageChannel();
>     receivePort.onmessage = (info) => resolve(info.data);
>     sendPort.postMessage(obj);
>   })
> }
> const info = {
>   a: 1,
>   b: [
>     {
>       c: "abc",
>       d: [1,2,3,4]
>     }
>   ]
> }
> deepClone(info).then(v => {
>   const deepV = v;
>   info.b[0].d[1] = 4;
>   console.log("deepV = ", deepV);
>   console.log("info = ", info);
> })
> ```

#### 方式3
>手写, 当然还有一些例如WeakMap,WeakSet没有处理，理解就行，需要的时候再加即可

>[!Example] 示例
> ```javascript
> function deepClone(obj) {
> 
>   if (!obj instanceof Object) {
>     return obj
>   } else {
>     const type = Object.prototype.toString.call(obj);
>     if (type === '[object Array]') {
>       const nArray = []
>       obj.forEach(item => {
>         nArray.push(deepClone(item));
>       })
>       return nArray;
>     } else if (type === '[object Object]') {
>       const nObj = {};
>       Object.keys(obj).map(key => {
>         nObj[key] = deepClone(obj[key]);
>       })
>       return nObj;
>     } else if (type === '[object Set]') {
>       const nSet = new Set();
>       obj.forEach(ob => nSet.add(deepClone(ob)));
>       return nSet;
>     } else if (type === '[object Map]') {
>       const nMap = new Map();
>       obj.keys().forEach(key => {
>         const v = obj.get(key);
>         nMap.set(key, deepClone(v));
>       })
>       return nMap;
>     } else {
>       return obj;
>     }
>   }
> }
> 
> const info = {
>   a: 1,
>   b: {
>     c: "c",
>     d: [
>       {
>         a: 1,
>         b: {
>           c: "c",
>           d: ["q", "q"]
>         }
>       }
>     ]
>   }
> }
> 
> const nInfo = deepClone(info);
> 
> info.b.d[0].a = 2;
> info.b.d[0].b.d[0] = "cc";
> 
> console.log("nInfo = ", nInfo);
> console.log("info = ", info);
> ```

