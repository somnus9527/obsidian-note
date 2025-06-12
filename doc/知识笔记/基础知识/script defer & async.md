---
tags:
  - 知识笔记/基础知识
---
>[!Danger]
>- defer: 不阻塞DOM加载&解析,按顺序执行,所有defer脚本执行完毕才会触发DOMContentLoaded
>- async: 同样不阻塞DOM加载&解析,按加载顺序执行(先加载完成先执行)

>[!Tip]
>适性: defer更适合项目依赖的Common包加载,async适合三方包(加载成功,执行成功与否并不对应用起决定性作用,比如埋点,广告相关js包)
>另外: js动态加载的js默认是async的,可以通过设置script.async=false变成defer的

