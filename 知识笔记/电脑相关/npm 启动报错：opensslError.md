---
tags:
  - 知识笔记/电脑相关
---
>[!Error] 报错信息
>`opensslErrorStack: [ 'error:03000086:digital envelope routines::initialization error' ]`

>[!Success] 解决方案
>- 在npm脚本上设置环境变量`NODE_OPTIONS=--openssl-legacy-provider`
>- 降级node版本到16，因为这玩意就是node17以上可能会出现,但是不建议这么干
>- 最好是能找到出问题的三方包，但是。。。

