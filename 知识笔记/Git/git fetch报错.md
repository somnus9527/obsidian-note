---
tags:
  - 知识笔记/Git
---
>[!Error] 报错信息
>错误：RPC 失败。curl 18 transfer closed with outstanding read data remaining
fetch-pack: unexpected disconnect while reading sideband packet
致命错误：协议错误：坏的包头

>[!Success] 解决方案
>`git gc --prune=now`