---
tags:
  - 知识笔记/电脑相关
---
#### 问题记录

在公司安装electron时出现,原因是公司的防火墙造成大量证书失效,而electron在安装时会跑node_modules\electron\install.js,这个方法使用了node的https模块去github上下载,出现ssl证书鉴权失败然后报错RequestError: unable to verify the first certificate;

使用了很多方式,都无法解决:

1. npm config set strict-ssl false
2. 设置环境变量ELECTRON_MIRROR=https://npm.taobao.org/mirrors/electron/
3. npm cache clean —force
4. 安装指定版本的electron

最后安装cnpm解决

npm install cnpm -g

npm install cnpm -g --registry=https://registry.npm.taobao.org 安装并设置镜像