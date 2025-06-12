---
tags:
  - 知识笔记/基础知识
---
>[!Tip]
>**React中的位运算主要用在Lane模型计算,Lane(单个更新优先级),Lanes(批任务优先级),两个都是32位无符号整型**
>
>_Lane模型相对于老的过期时间模型优势在于解耦单个Fiber和一个Batch更新,性能也更好,比如之前的过期时间模型想要知道当前新添加的Fiber是否在当前正在执行的batch中,需要做很多判断甚至会进入递归,切换到Lane模型之后位运算即可搞定_

>[!Info] React中部分Lane定义如下
>![[Pasted image 20250110130757.png]]

#### 以权限判断作为例子:

```javascript
    var a = 0b100;
    var b = 0b010;
    var c = 0b001;
    
    // 那么假如user权限值如下:
    var user = 0b010;
    // 判断改用户是否有b权限
    user&b === 0
    // 为用户赋予a权限
    user|a
    // 为用户取消b权限
    user&(~b)
```