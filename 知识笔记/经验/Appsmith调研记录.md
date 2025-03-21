---
tags:
  - 知识笔记/经验
---
#### 前置准备

>[!Note] 前端服务
>1. node版本20.11.1
>2. yarn install
>3. yarn run start

>[!Note] 后端服务
>1. brew install openjdk@17
>2. 配置JAVA_HOME
>> ```bash
>> export JAVA_HOME="/opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk/Contents/Home"
> export PATH="$JAVA_HOME/bin:$PATH"
>>```
>3. brew install maven
>4. cd app/server
>5. mvn clean compile
>6. ./build.sh -DskipTests
>7. ./scripts/start-dev-server.sh


#### 风险记录

>[!Error] node版本
>appsmith源码前端部分要求node版本20.11.1和目前athena-web依赖的node版本冲突???

