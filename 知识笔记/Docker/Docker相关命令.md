---
tags:
  - 知识笔记/Docker
---
#### 构建镜像
>[!Note] 命令
>```
>docker build -t athena-web -f dockerfile/Dockerfile .
>```

>[!Tip] 命令解释
>**-t athena-web** 镜像的名字
>**-f dockerfile/Dockerfile** Dockerfile位置
>**.** 指定上下文，比如：我的文件夹结构是
>-- parent_folder
>--- athena
>---- main
>---- subApps
>--- dockerfile
>---- Dockerfile
>---- xxxx
>我执行命令的pwd在xxx/xxx/parent_folder 那么我使用上面的命令 Context就是parent_folder,Docker相关路径则需要注意,比如下面的例子:

>[!Example] 例子, 执行命令的路径为parent_filder
>``` Dockerfile
>FROM nginx:1.12.1
COPY ./athena /usr/share/nginx/html/athena/
RUN cd /usr/share/nginx/html/athena/main  && mv -f * ../../
RUN cd /usr/share/nginx/html/athena/subApps  && mv -f * ../../
ADD ./dockerfile/nginx.conf /etc/nginx/
ADD ./dockerfile/default.conf /etc/nginx/conf.d/
ADD ./dockerfile/frontendShell.sh /usr/share/nginx/html/
RUN chmod +x /usr/share/nginx/html/frontendShell.sh
ADD ./dockerfile/dockerEnv.sh /usr/share/nginx/html/
RUN chmod +x /usr/share/nginx/html/dockerEnv.sh
WORKDIR /usr/share/nginx/html
EXPOSE 80
ENTRYPOINT ["/usr/share/nginx/html/frontendShell.sh"]
#CMD ["nginx", "-g", "daemon off;"]
>```

#### 启动容器
>[!Note] 命令
>docker run -p 9527:80 -d athena-web

>[!Tip] 说明
>-p 指定端口 本地端口 : 容器端口
>athena-web 镜像名称/镜像ID
>docker run -p 9527:80 -d athena-web 镜像名称/镜像ID之前加上-d可以静默启动

#### 其余常用命令记录
1. `docker ps` 可以查看当前运行中的容器，加上参数 -a 可以查看所有容易，包括停止的;找到container ID 则可以通过 `docker stop containerId`停止容器
2. `docker images`可以查看所有镜像
3. 删除指定容器`docker rm containerId`
4. 删除所有已停止的容器 `docker rm prune`,可以加-f取消确认
5. 当然，可以`docker stop containerId && docker rm containerId`停止并删除容器
6. 删除镜像 `docker rmi imageId`配合`docker images`获取imageId
7. `docker image prune`删除所有悬虚镜像，什么是悬虚镜像见问题记录第一条
8. `docker-compose build` 构建镜像
9. `docker-compose up -d` 启动容器，-d表示静默启动

#### 问题记录
1. docker build构建镜像时，如果-t指定的名称已经存在了，这时候镜像构建不会有问题，依然会生成正确的镜像，但是之前已经存在的同名镜像会变成一个名为<none>的悬虚镜像

>[!Info] 解释
>这个问题其实没有什么大的影响，但是docker images命令会出现越来越多的无效镜像被列出来，所以处理方式有两种
>1. docker build之前先docker images看一下现有镜像，尽量不要出现同名镜像，如果必须使用，可以先docker rmi删除那个同名镜像之后，再docker build
>2. 直接docker build，发现悬虚镜像多了之后，通过docker image prune删除所有悬虚镜像