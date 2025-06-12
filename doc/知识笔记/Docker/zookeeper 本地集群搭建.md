---
tags:
  - 知识笔记/Docker
---
>[!Tip] 
>network是为了服务可以和zookeeper集群统一网络环境,实际docker-compose配置如下示例

>[!Example] docker-compose.yaml示例
> ```yaml
> version: '2'
> services:
>   provider1:
>     container_name: provider1
>     image: provider1
>     build:
>       context: .
>       dockerfile: Dockerfile-provider
>     volumes:
>       - .:/dubbo-service
>     depends_on:
>       - zookeeper1
>       - zookeeper2
>       - zookeeper3
>     ports:
>       - "6000:6000"
>     networks:
>       - yuan-micro-net
>   consumer1:
>     container_name: consumer1
>     image: consumer1
>     build:
>       context: .
>       dockerfile: Dockerfile-client
>     volumes:
>       - .:/dubbo-service
>     depends_on:
>       - provider1
>     ports:
>       - "9527:8000"
>     networks:
>       - yuan-micro-net
>   zookeeper1:
>     image: 'bitnami/zookeeper:3.6.2'
>     ports:
>       - '2181:2181'
>       - '2888'
>       - '3888'
>     volumes:
>       - ./zookeeper-persistence:/zookeeper
>     environment:
>       - ALLOW_ANONYMOUS_LOGIN=yes
>       - ZOO_SERVER_ID=1
>       - ZOO_SERVERS=0.0.0.0:2888:3888,zookeeper2:2888:3888,zookeeper3:2888:3888
>     networks:
>       - yuan-micro-net
>   zookeeper2:
>     image: 'bitnami/zookeeper:3.6.2'
>     ports:
>       - '2182:2181'
>       - '2888'
>       - '3888'
>     volumes:
>       - ./zookeeper-persistence:/zookeeper
>     environment:
>       - ALLOW_ANONYMOUS_LOGIN=yes
>       - ZOO_SERVER_ID=2
>       - ZOO_SERVERS=zookeeper1:2888:3888,0.0.0.0:2888:3888,zookeeper3:2888:3888
>     networks:
>       - yuan-micro-net
>   zookeeper3:
>     image: 'bitnami/zookeeper:3.6.2'
>     ports:
>       - '2183:2181'
>       - '2888'
>       - '3888'
>     volumes:
>       - ./zookeeper-persistence:/zookeeper
>     environment:
>       - ALLOW_ANONYMOUS_LOGIN=yes
>       - ZOO_SERVER_ID=3
>       - ZOO_SERVERS=zookeeper1:2888:3888,zookeeper2:2888:3888,0.0.0.0:2888:3888
>     networks:
>       - yuan-micro-net
>   zoonavigator:
>     container_name: zoonavigator
>     image: elkozmon/zoonavigator
>     ports:
>       - "19000:9000"
>     networks:
>       - yuan-micro-net
> networks:
>   yuan-micro-net:
>     driver: bridge
> ```