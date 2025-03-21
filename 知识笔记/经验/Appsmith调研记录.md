---
tags:
  - 知识笔记/经验
---
#### 环境准备

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

>[!Success] 至此，本地已经可以正常访问appsmith服务了
>![[Pasted image 20250321151610.png]]
>**那么到这里可以说明，后端服务有机会剥离 *当然只是有机会***

#### 尝试开发一个自定义组件

1. 需要在`src/widgets`中新建一个组件
2. 组件开发需要继承`BaseWidget`,这个BaseWidget是一个抽象类内容比较复杂，这就带来一个限制
3. 查看BaseWidget源码，再结合文档开发完成
4. 开发完成之后，在`src/widgets/index.ts`中引入并注册
5. 完成之后设计器中就可以选择这个组件了![[Pasted image 20250321165713.png]]
6. 可以简单测试一下设置器![[Pasted image 20250321165739.png]]![[Pasted image 20250321165759.png]]

#### 总结

1. 集成环境，如果不需要适配原有dsl，一切以现有appsmith系统为准，那么可用度还行
2. 当然集成环境的问题就在于，一旦需要剥离后端服务，对接现有dsl，那么几乎不可用，理由：
	1. 现有系统很复杂，内部数据结构固定，（这里说的数据结构固定是指，现有前端源码和现有后端服务的对接，不是指dsl。哪怕删除这些）对接别的后端服务代价会很大，哪怕最小的代价去写一个转换器来专门处理数据结构，这个转换器会非常恐怖。当然如果新的后端能够适配现有的请求则另说
	2. 现有前端服务存在一些不纯粹的东西，比如登陆界面，鉴权，分享，git连接等等，如果需要一个纯粹的设计器，这些东西的剥离也有消耗

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
> 而mongo默认是单例服务，[[#^76bf48|处理一下mango服务]]

4. 前端访问的后端服务不是本地的8080
>[[#^833803|处理一下前端代码，让请求发到本地的8080后端服务]]

5. 请求报错，关键信息：**AE-BAD-4002: Appsmith version mismatch**
> [[#^824d76|处理这个报错，本地不需要关心版本号是否匹配]]

6. 一堆tracing报错
> 很明显这个是埋点请求，所以很明显也不需要，找一下源码，主要处理点 **这个应该是源码的bug，原来是startsWith("}")** 
> ![[Pasted image 20250321150813.png]]
> 这样就可以取消埋点接口调用,因为源码中会读tracingUrl,读不到就会禁用这个功能,disabled代码就不贴了


7. 打开编辑器页面，加载失败
> [[#^8260a6|处理编辑器页面加载失败]]

8. 自定义开发组件需要遵循**BaseWidget**,这是一个抽象类，内部做了大量的事情，那么就意味着，如果想开发自定义组件时使用 function component形式，前置就需要改写BaseWidget, 做成HOC,否则就必须以class component形式开发。*当然，这也不是什么大问题，用class component形式开发也不无不可。实在不行将BaseWidget改写成HOC,即可支持使用function component形式开发，但毕竟存在时间消耗*
#### 风险记录

>[!Error] node版本
>appsmith源码前端部分要求node版本^20.11.1和目前athena-web依赖的node版本冲突

>[!Error] 剥离前后端服务难度不小
>1. CI/CD流程和现有的不是配，这块也是要动的
>2. rts服务的剥离
>3. 现在的appsmith其实有一套自己的数据结构，如果剥离后端，数据结构的适配工作量很大(前后端交互，不是dsl)
>4. 前端不是类似lowcode那种单纯的设计器，存在不少类似登录页的内容，这块也需要删除
>5. 前端请求并未设置baseURL，如需剥离后端服务这块就需要去改

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
>启动之后去mongosh初始化副本集

^76bf48
>[!Note] mongosh命令记录
>- 副本集初始化命令: `rs.initiate()`
>- 查看当前副本集状态: `rs.status()`
>- 删除数据库：进入要删除的数据库执行`db.dropDatabase()`
>- 鉴权命令：`db.auth("username", "password")`
>- 查看当前所以连接: `db.adminCommand({ getCmdLineOpts: 1 })`
>- 创建用户: db.createUser({ user: 'username', pwd: 'password', roles: [{ role: 'dbOwner', db: 'dbname' }] })
>- 查看当前所有用户：`db.getUsers()`

>[!Note] 前端请求地址处理
>1. 先看前端源码，发请求用的是axios，但是源码中却没有baseURL的设置，所以源码中使用的是相对路径发请求
>2. 那么很明显，本地部署的场景下不行，所以简单处理，直接配转发
>3. 看了下源码![[Pasted image 20250321145859.png]]，源码获取代理的地方是package.json，所以直接为package.json增加proxy配置![[Pasted image 20250321145945.png]],这样前端请求就打到8080端口了

^833803

>[!Note] 后端服务报错：版本不匹配
>1. 先通过报错去源码中找,先找到报错定义：![[Pasted image 20250321150422.png]]
>2. 再找使用的地方:![[Pasted image 20250321150446.png]]
>3. 服务端我们的版本号也没配，因为本地不关心这个，所以简单处理直接把判断注释，走后续逻辑即可(注释的代码就是上面注释的部分)



^824d76

>[!Note] 编辑器加载失败
>1. 报错信息：![[Pasted image 20250321151112.png]]
>2. 很明显连接8081端口的rts服务加载失败
>3. 所以启动一下rts服务，大致看了一下，rts服务是一个websocket服务，用于实时通信
>4. rts服务在**app/client/packages/rts**文件夹下，同样`cp .env.example .env`，有了配置文件（默认就够用），直接`yarn run start`启动即可，当然`./start-server.sh`也可以

^8260a6
