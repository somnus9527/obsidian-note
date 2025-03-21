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
>1. brew install openjdk@17 *安装jdk*
>2. 配置JAVA_HOME
>> ```bash
>> export JAVA_HOME="/opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk/Contents/Home"
> export PATH="$JAVA_HOME/bin:$PATH"
>>```
>3. brew install maven *安装maven*
>4. cd app/server *cd进目录*
>5. cp ./envs/dev.env.example ./.env *复制一份环境变量到正式的.env配置*
>> 具体参数主要需要修改**APPSMITH_DB_URL**
>6. vn clean compile *先清理之前的编译，主要是让maven安装依赖，顺便清理一下*
>7. /build.sh -DskipTests *先编译，参数是跳过测试，目的是编译更快，理由是刚fork的代码不需要测试*
>8. 启动mongo服务和redis服务不要忘了
>9. /scripts/start-dev-server.sh *启动后端项目，dev模式*

#### 问题记录
1. JDK版本冲突
> 重新安装openjdk@17
> 因为server的配置中要求17版本：java.runtime.version=17 （在system.properties）

2. 安装mongosh时，同步安装了18版本的jdk
> 强制移除自带的18版本 *brew uninstall openjdk --ignore-dependencies openjdk*
> 打开.zshrc *nvim ~/.zshrc*
> 增加JAVA_HOME配置 
> `export JAVA_HOME="/opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk/Contents/Home"
> export PATH="\$JAVA_HOME/bin:\$PATH"`

3. service启动报错：[[#^168173|连接mongo报错]]
> 根据报错信息，我去.env文件中修改了**APPSMITH_DB_URL**，但是依然没有效果
> 后来发现server需要连分片集群，示例配置中的参数 *--replSet=rs0*就是连接rs0的副本集
> 而mongo默认是单例服务，[[#^76bf48|所以需要处理一下mango服务]]
#### 风险记录

>[!Error] node版本
>appsmith源码前端部分要求node版本20.11.1和目前athena-web依赖的node版本冲突???

#### Tips

>[!Note] mongo安装配置
>1. brew install mongo-community *安装mongo(官方包)*
>2. mongosh *进入命令行*
>3. use admin *进入admin db*
>4. [[#^8f885f|创建admin用户]]
>5. exit 退出
>6. 打开mongo配置 *mongo.conf*
>7. [[#^8f567b|顶层增加配置开启鉴权]]
>8. [[#^448da6|重启mongo服务]]

>[!Note] mongosh 命令行创建用户
>```bash
>db.createUser({
>  user: 'admin',
>  pwd: 'your_password',
>  roles: [{ role: 'root', db: 'admin' }]
>})
>```

^8f885f

>[!Note] mongo.conf 配置开启鉴权
>```yaml
>security:
>  authorization: enabled
>```

^8f567b

>[!Note] 重启mongo服务命令
>```bash
>brew services restart mongo-community
>```

^448da6
>[!Error] server连接mongo报错信息
>```plaintext
>[2025-03-20 09:21:32,147] [main] requestId= userEmail= traceId= spanId= - Error in mongock process. ABORTED MIGRATION
io.mongock.api.exception.MongockException: io.mongock.api.exception.MongockException: Error in method[DatabaseChangelog0.initializeSchemaVersion] : This MongoDB deployment does not support retryable writes. Please add retryWrites=false to your connection string.
>```

^168173

>[!Note] 分片集群启动mongo服务
>先停掉服务: `brew services stop mongodb-community`
>再重新启动，并开启副本集: `mongod --replSet rs0 --bind_ip localhost --dbpath /opt/homebrew/var/mongodb --logpath /opt/homebrew/var/log/mongodb.log --fork`
>参数说明：
>- --replSet rs0  启用副本集，名称为 rs0
>- --bind_ip localhost 仅允许本地连接
>- --dbpath /opt/homebrew/var/mongodb MongoDB 数据路径
>- --logpath /opt/homebrew/var/log/mongodb.log 日志路径
>- --fork 后台运行

^76bf48

