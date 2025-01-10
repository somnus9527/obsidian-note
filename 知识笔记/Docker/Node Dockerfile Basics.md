---
tags:
  - 知识笔记/Docker
---
- 使用COPY而不是ADD,因为ADD可以处理网络资源并且遇到压缩包会进行解压,为了Dockerfile的简洁、语义清晰尽量使用COPY
- npm/yarn 是在build时进行安装的,一般我们在构建node镜像时,现在的node镜像都会自动有这两个工具,可以自己选择使用npm/yarn,但要保证一致性,需要一会用npm一会用yarn,这样会增加大量的镜像大小
- CMD 尽量直接使用node而不是npm start之类的指令,如CMD ["node","./bin/www"],原因是npm会额外有自己的一个进程,且npm可能会发送错误的singnal给node进程,不利于更精细的node操作

>[!Example] 简单的例子
> ```Dockerfile
> FROM node:16
> EXPOSE 3000
> WORKDIR /app
> COPY . .
> # npm cache clean的目的是清楚额外缓存,减小镜像的大小
> RUN npm install && npm cache clean --force
> CMD ["node","./bin/www"]
> ```

>[!Warning] 注意点
>使用node镜像,一定选择偶数版本,Netflix大佬经验之谈,说的原因是,所有稳定版本都是偶数,对于生产环境稳定是第一位的

