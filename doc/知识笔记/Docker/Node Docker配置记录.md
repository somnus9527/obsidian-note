---
tags:
  - 知识笔记/Docker
---
>[!Info] 项目目录
>- yuan-framework
>	- back-end
>		- docker-compose.yaml
>		- test-app 项目目录
>			- Dockerfile-test
>	- front-end

>[!Example] Dockerfile-test示例
> ```Dockerfile
> FROM node:16
> WORKDIR /yuan-framework/back-end/test-app
> 
> # COPY package*.json .
> # 执行cache clean是为了使镜像尽可能的小
> RUN yarn install && yarn cache clean --force
> # COPY . .
> CMD npm start
> ```

>[!Example] docker-compose.yaml示例
> ```yaml
> version: '3.9'
> services:
>   yuan-framework-test:
>     container_name: yuan-framework-test
>     image: "yuan-framework-test:1.0.0"
>     build:
>       context: ./test-app
>       dockerfile: Dockerfile-test
>       labels:
>         - "yuan-framework-test-app-name=test"
>         - "yuan-framework-author=somnus9527"
>         - "yuan-framework-email=somnus9527@yeah.net"
>     ports:
>       - "9000:8000"
>     env_file:
>       - ./test-app/yuan-framework-test.develop.env
>     volumes:
>       - .:/yuan-framework/back-end
>     depends_on:
>       - db
>       - redis
> 
>   db:
>     image: mysql:8
>     #    platform: linux/x86_64 #m1芯片的mac需要添加改行配置
>     restart: always
>     env_file:
>       - ./test-app/yuan-framework-test.develop.mysql.env
>     # environment:
>     #  MYSQL_DATABASE: yuan-framework-test
>     #  MYSQL_ROOT_PASSWORD: rosiky9527
>     volumes:
>       - .dbdata/test-app:/var/lib/mysql
>     ports:
>       - "10000:3306"
> 
>   redis:
>     image: redis:6.0
>     ports:
>       - "15000:6379"
> ```