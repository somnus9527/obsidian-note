---
tags:
  - 知识笔记/电脑相关
---
>[!Danger] android studio加载flutter项目出现如下报错：
>Android studio Cause unable to find valid certification path to requested target

>[!Success] 2022-01-20解决方式
>1. [git地址](https://github.com/escline/InstallCert),首先去该地址clone仓库代码
>2. 进入项目目录编译java代码 `javac InstallCert.java`
>3. 然后运行,`dl.google.com`是你下载文件失败请求根域名,不包含**https://**  `java InstallCert dl.google.com`
>4. 注意运行第一行`Loading KeyStore ..........`,拷贝一下地址
>5. 将项目下生成的证书文件jssecacerts,拷贝至上一步拷贝的地址文件夹下
>6. 如果这样都解决不了,那么去改gradle的dependencies吧

