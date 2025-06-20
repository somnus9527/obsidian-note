---
tags:
  - 知识笔记/基础知识
---
#### 原码 反码 补码
>[!Note]
>去了解这个的原因主要是在react原码中大量的 a |= b;的操作,以及算法题,通过每一bit去表示一个值是否存在

>[!Example] 数字1
>- 原码: 0000 0001
>- 反码: 0000 0001
>- 补码: 0000 0001

>[!Example] 数字-1
>- 原码: 1000 0001
>- 反码: 1111 1110  **除符号位取反**
>- 补码: 1111 1111  **反码+1**

#### 位运算
>[!Warning]
>**注意事项: 核心点就是系统存储二进制数据的时候是采用补码的形式存储,**_**并且所有运算均是针对补码**_**,所以一定注意原码->补码,补码->原码的转化**

##### 取反 **(取反操作包括符号位)**
>[!Info]
>操作符 ~

>[!Example] ~11
>```
>11的原码 0000 1011 (正数补码相同所以直接取反)
操作取反 1111 0100 (这个东西就是结果值,这时是补码,要转成数值,必须要先转回原码)
那么先-1 1111 0011 (这个是我们要算的值的反码)
然后反码 1000 1100 (这个是要算的值的原码)
那么结果 = -12
>```

>[!Example] ~-12
>```
>-12的原码 1000 1100 (注意所有操作都是针对补码,所以先转成补码)
先取反码  1111 0011 (-12的反码)
反码+1    1111 0100 (-12的补码,ok可以开始计算了)
操作取反  0000 1011 (结果值,可以看到是正数,证书补码原码一致,那么可以直接得到数值)
那么结果 = 11
>```

##### 或 **(有一个1就等于1,否则等于0)**
>[!Info]
>操作符 |

>[!Example] 1|-2
>```
>只要注意所有操作都是针对补码就不会有问题
1的补码   0000 0001
-2的原码  1000 0010 -> 反码 1111 1101 -> 补码 1111 1110
或操作    1111 1111
补码再转原码
先-1      1111 1110
取反码    1000 0001
那么结果 = -1
>```

##### 与 **(全部是1就等于1,否则等于0)**
>[!Info] 
>操作符 &

##### 异或 **(相同等于0,不同等于1,即 0^0 = 0 1^1=0 1^0=1 0^1=1)**
>[!Info]
>操作符 ^

##### 左移 **(左移,右边空出来的用0补)**
>[!Info]
>操作符 <<

>[!Example] -1<<1
>```
>-1原码   1000 0001
反码     1111 1110
补码     1111 1111
左移一位 1111 1110
-1       1111 1101
取反码   1000 0010
那么结果 = -2
>```

##### 有符号右移 **(右移,右边空出来的用符号位补)**
>[!Info] 
>操作符 >>

>[!Example] -1>>1
>```
>-1原码   1000 0001
反码     1111 1110
补码     1111 1111
右移一位 1111 1111
-1       1111 1110
取反码   1000 0001
那么结果 = -1
>```

##### 无符号右移 **(右移,右边空出来的用0补)**
>[!Info]
>操作符 >>>

