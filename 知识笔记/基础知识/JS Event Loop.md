---
tags:
  - 知识笔记/基础知识
---
>Event Loop相当玄乎并且恶心

#### 浏览器

浏览器的事件循环比较简单,首先分清宏任务&微任务  
宏任务:  
- script(整体代码)  
- setTimeout()  
- setInterval()  
- postMessage  
- I/O  
- UI交互事件  
- MessageChannel  
微任务:  
- MutationObserver(新的H5特性)  
- Promise(注意async/await也是Promsie,只是语法糖而已)
执行流程：
![[browser-excute-animate.gif]]

#### Node

Node事件循环相对复杂主要有六个阶段:  
![[Pasted image 20241231185718.png]]
代码主流程跑完之后开始六个阶段的执行(主要关注1,4,5阶段):

1. **timers**: `setTimeout`和`setInterval`的callback
2. Pending callbacks: **基本上 pending callback phase 就是执行上一轮 poll phase 没有成功触发的 callback**
3. idls,prepare: **仅node内部使用**
4. **poll**: **获取新的I/O事件, 适当的条件下node将阻塞在这里**
5. **check**: `setImmediate`的callback
6. close callback: **包括 socket的close回调**

执行流程:  
![[node-excute-animate.gif]]

每个阶段执行完成后会执行完全部微任务队列

#### 其余关注点

1. process.nextTick() VS setImmediate()
>**!!!首先process.nextTick的callback一定要清空调用栈才会开始下一个阶段.**process.nextTick 是在每个阶段中间执行,setImmediate在check阶段,也就是说,process.nextTick方法一定会比setImmediate方法先执行.(所以官方不推荐使用nextTick)

2. 查看JS调用堆栈动画网址. [JS调用栈动画](http://latentflip.com/loupe/?code=JC5vbignYnV0dG9uJywgJ2NsaWNrJywgZnVuY3Rpb24gb25DbGljaygpIHsKICAgIHNldFRpbWVvdXQoZnVuY3Rpb24gdGltZXIoKSB7CiAgICAgICAgY29uc29sZS5sb2coJ1lvdSBjbGlja2VkIHRoZSBidXR0b24hJyk7ICAgIAogICAgfSwgMjAwMCk7Cn0pOwoKY29uc29sZS5sb2coIkhpISIpOwoKc2V0VGltZW91dChmdW5jdGlvbiB0aW1lb3V0KCkgewogICAgY29uc29sZS5sb2coIkNsaWNrIHRoZSBidXR0b24hIik7Cn0sIDUwMDApOwoKY29uc29sZS5sb2coIldlbGNvbWUgdG8gbG91cGUuIik7!!!PGJ1dHRvbj5DbGljayBtZSE8L2J1dHRvbj4%3D)
3. NodeJS是单线程的吗? (不是)

首先有一个例子:
```javascript
const fs = require("fs")
fs.readFile("./a.js", () => {
    console.log("read file!")
})
console.log("test");
while(true) {}
// test
```

这个代码执行不会走到IO callback,简单来看是因为死循环阻塞了Event Loop,或者你说阻塞了线程也可以(从单线程来看).但是其实括号里的说法是错误的.那么是不是CPU密集型一定会阻塞Event Loop呢.  
例子:
```javascript
const crypto = require("crypto");
const start = Date.now();
crypto.pbkdf2('a', 'b', 100000, 512, 'sha512', () => {
    console.log('1:', Date.now() - start);
});
crypto.pbkdf2('a', 'b', 100000, 512, 'sha512', () => {
    console.log('2:', Date.now() - start);
});
console.log("done")
// done
// 1: 5xx ms
// 2: 5xx ms
```

这里你会发现时间竟然是差不多的.那么如果NodeJS真的是单线程,这个结果是不可能的,应该是500,1000左右才对,但事实确实是两个时间几乎一样.(所以其实这里我们可以认定,NodeJS并不是单线程).再看一个例子:
```javascript
process.env.UV_THREADPOOL_SIZE = 1;
const crypto = require("crypto");
const start = Date.now();
crypto.pbkdf2('a', 'b', 100000, 512, 'sha512', () => {
    console.log('1:', Date.now() - start);
});
crypto.pbkdf2('a', 'b', 100000, 512, 'sha512', () => {
    console.log('2:', Date.now() - start);
});
console.log("done")
// done
// 1: 5xx ms
// 2: 10xx ms
```

这个例子结果又很出人意料,结果变成了上一个例子猜想中的结果.原因出在第一行代码,我们通过环境变量把NodeJS(实际是libuv(libuv管理了NodeJS最核心的Event Loop 和 Threed Pool))的线程池大小限制成了1(默认是4),那么这时自然变成了时间叠加的效果.  
由于默认池子是4,那么下面的例子就会变成前四个500后四个1000:
```javascript
const start = Date.now();
const crypto = require("crypto");
const fs = require("fs");
function doHash() {
    crypto.pbkdf2('a', 'b', 100000, 512, 'sha512', () => {
        console.log('Hash:', Date.now() - start);
    });
}
doHash();
doHash();
doHash();
doHash();
doHash();
doHash();
doHash();
doHash();
// Hash: 595
// Hash: 609
// Hash: 613
// Hash: 615
// Hash: 1212
// Hash: 1227
// Hash: 1230
// Hash: 1244
```
**这里fs的单纯读取只需要3ms左右**,而加上crypto之后影响了fs的读取,由于之前说的默认只有四个进程所以一开始的进程分配如下:  
fs -> Thread1  
doHash // A -> Thread2  
doHash // B -> Thread3  
doHash // C -> Thread4  
Thread1开始执行之后就把任务交给了真正的执行者File System,这时候Thread1就空闲下来了,它就会开始执行hoHash,这时候进程分配如下:  
doHash // A -> Thread2  
doHash // B -> Thread3  
doHash // C -> Thread4  
doHash // D -> Thread1  
而doHash需要500ms左右才能执行完成,虽然File System只需要3ms就执行完成了,但是它需要等待空闲Thread,等一个doHash执行完成之后,File System才能通知Thread,我执行完成了,Thread才能把callback再扔给Event Loop去执行callback,造成fs的执行时间变成了500多ms.  
再看看Network的情况:
- 情况1
```javascript
const http = require("http")
const crypto = require("crypto")
const start = Date.now();
function doHash() {
    crypto.pbkdf2('a', 'b', 100000, 512, 'sha512', () => {
        console.log('Hash:', Date.now() - start);
    });
}
function doRequest() {
    http.request("http://localhost:4000", res => {
        res.on("data", () => {
        })
        res.on("end", () => {
            console.log("Request:", Date.now() - start);
        });
    }).end();
}
doRequest();
doHash();
doHash();
doHash();
doHash();
// Hash: 758
// Hash: 761
// Hash: 762
// Hash: 762
// Request: 3032
```
- 情况2
```javascript
const http = require("http")
const crypto = require("crypto")
const start = Date.now();
function doHash() {
    crypto.pbkdf2('a', 'b', 100000, 512, 'sha512', () => {
        console.log('Hash:', Date.now() - start);
    });
}
function doRequest() {
    http.request("http://localhost:4000", res => {
        res.on("data", () => {
        })
        res.on("end", () => {
            console.log("Request:", Date.now() - start);
        });
    }).end();
}
doHash(); // A
doHash(); // B
doHash(); // C
doHash(); // D
doRequest();
// Hash: 609
// Hash: 613
// Hash: 621
// Hash: 624
// Request: 3620
```
- 情况3
```javascript
const http = require("http")
const start = Date.now();
function doRequest() {
    http.request("http://localhost:4000", res => {
        res.on("data", () => {
        })
        res.on("end", () => {
            console.log("Request:", Date.now() - start);
        });
    }).end();
}
doRequest();
doRequest();
doRequest();
doRequest();
doRequest();
doRequest();
doRequest();
doRequest();
// Request: 3025
// Request: 3035
// Request: 3036
// Request: 3036
// Request: 3036
// Request: 3037
// Request: 3037
// Request: 3037
```

**这里单纯的Http请求需要3000ms的时间返回**第一种情况其实不难理解: Thread需要发送请求,它只是把这个任务交给了OS去执行,而等到OS执行完成返回的时候,Thread都已经空闲了,所以对Http请求的延迟时间没有影响.  
第二种情况: 四个进程被占满,虽然Thread不自己执行任务,但是任务还是需要Thread发送到OS,OS才会开始执行,所以需要等doHash执行完成之后,Thread才能给OS发送通知,所以延迟了500左右  
第三种情况: Thread不自己执行任务,所以不影响延迟  
这里相对第二种情况,看下面的代码:
```javascript
const http = require("http");
const crypto = require("crypto");
const start = Date.now();
function doHash() {
    crypto.pbkdf2("a", "b", 100000, 512, "sha512", () => {
        console.log("Hash:", Date.now() - start);
    });
}
function doRequest() {
    http.request("http://127.0.0.1:4000/test", res => {
        res.on("data", () => {
        });
        res.on("end", () => {
            console.log("Request:", Date.now() - start);
        });
    }).end();
}
doHash();
doHash();
doHash();
doHash();
doRequest();
// Hash: 697
// Hash: 700
// Hash: 700
// Hash: 703
// Request: 3014
```
很神奇,又变成3000左右了...  
**原因是当 http 模组发送请求是用『IP』而不是『hostname』的时候  
此时去指派任务的人, 就不是 Thread Pool, 而是 Event Loop 本身这条 Thread  
但为何会有这样的区别, 原因是 http 底层用了 dns.lookup 去解析 hostname  
而 dns.lookup 实际操作方式就是用 Thread Pool**  
所以才会有下面这张图的解释, 让大家了解 node 各模组所属的组别是什么:
![[Pasted image 20241231190746.png]]