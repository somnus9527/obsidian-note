---
tags:
  - 知识笔记/经验
---
#### 环境准备

>[!Info] 版本
>- node: **18.18.2**
>- npm: **9.8.1**

>[!Note] 依赖
>- PostgreSQL: **12.0.2**
>- Redis: 未见明确要求
>- Python3: 
>	- 如果按钮版本没有distutils需要重新安装，前端项目启动使用的node-gyp依赖这玩意
>	- 当然如果是较新的python3版本，比如3.12.x，那么distutils这个module已经遭弃用，这时候需要安装setuptools（这里面包含distutils）

> - 处理完以上内容，前端代码就已经准备好了,这时候直接启动会出现一堆报错，简单翻一下源码就可以发现整个ToolJet强依赖plugins系统
> - ToolJet前后端源码都依赖一个plugins的东西，需要在源码根目录执行`npm run build:plugins` （这里注意，其实需要先安装plugins，但是由于存在pre**prebuild:plugins**,这里才可以直接执行build. *plugins是另外一个文件夹，也就是说除了frontend,server,plugins这个文件夹也是核心源码的地方*）
> - 到这里，前端源码就全部准备好了，可以`npm start`启动
> - 但是后端源码还没有完全可用，后端依赖PostgreSQL和Redis，装完之后还需要执行`npm run db:setup`,这个是完成数据库初始化的(不走数据库初始化这一步，会丢失ee文件夹，项目也是跑不起来的)。初始化完成之后才能启动`npm run start:debug`,这里我就不装PostgreSQL了，考虑直接在镜像中起后端服务，同时把现有的本地前端项目打到容器中的后端服务中

>[!Note] 开始在镜像中启动后端服务
>经过大量测试行不通，存在大量版本不匹配以及ce和ee版本不匹配的问题，但是不想继续在这条路上浪费时间了，开始走纯本地部署

>[!Note] 走纯本地部署
>- 先详细看一下源码和文档，搞清楚到底ce和ee的源码上到底区别在哪儿，否则感觉走不下去了

>[!Note] 部署步骤
>- 从ToolJet仓库拉取**v3.2.2-ce**这个Tag的代码 **这步很重要。。。**
>- 安装**postgresql@12**,注意一定是12，别听官方文档扯淡，让装13`brew install postgresql@12`
>- 安装**postgrest** `brew install postgrest`,这是一个轻量级的 web 服务器，它可以直接将 **PostgreSQL 数据库** 暴露为一个 **RESTful API**，无需手写后端代码。
>- 安装完成之后需要先去看一下数据库是否有用户(如果执行下面的命令提示没有命令行，那么去看一下你的环境变量是否配置，postgres官网有教怎么配环境变量):
>	- `psql -d postgres` 登陆postgres数据库
>	- `\du` 查看当前所有用户，后面会使用postgres这个用户，所以需要确保存在这个用户，如果没有就创建，当然不创建直接使用现有的用户也行，记得改.env文件就行
>	- `CREATE ROLE postgres WITH SUPERUSER CREATEDB CREATEROLE LOGIN;`创建postgres这个用户
>	- `ALTER USER postgres WITH PASSWORD 123456;`修改密码，建议改一下，否则没有密码，还需要开启数据库的不需要鉴权，这个的操作复杂度还不是设置一下密码
>	- 完成之后，整个数据库的本地配置就完成了
>- 进入之前clone的源码的根目录：
>	- `cp .env.example .env` 基于example的环境配置，新建一个.env的配置文件
>	- 修改一下.env的配置，核心修改点如下，其余不需要动：
> ```dotenv
> TOOLJET_HOST=http://localhost:8082  
> LOCKBOX_MASTER_KEY=1d291a926ddfd221205a23adb4cc1db66cb9fcaf28d97c8c1950e3538e3b9281  
> SECRET_KEY_BASE=4229d5774cfe7f60e75d6b3bf3a1dbb054a696b6d21b6d5de7b73291899797a222265e12c0a8e8d844f83ebacdf9a67ec42584edf1c2b23e1e7813f8a3339041  
> NODE_ENV=development
> PG_HOST=localhost  
> PG_PORT=5432  
> PG_USER=postgres  
> PG_PASS=postgres  
> PG_DB=tooljet_development  
> TOOLJET_DB=tooljet_db  
> TOOLJET_DB_USER=postgres  
> TOOLJET_DB_HOST=localhost  
> TOOLJET_DB_PASS=postgres  
> ORM_LOGGING=all
> ```
> 	- `npm install`安装顶层依赖, 这个命令以及下面的命令都是在根目录执行
> 	- `npm install --prefix server` 安装服务端的依赖
> 	- `npm install --prefix frontend` 安装前端依赖
> 	- `npm run build:plugins` 安装并构建插件
> 	- `npm run --prefix server db:create` 创建数据库并执行迁移
> 	- `npm run --prefix server db:reset` 重置数据
> 	- 至此本地环境就ok了，可以启动本地服务：
> 		- `cd ./plugins && npm start`启动plugins开发服务
> 		- `cd ./server && npm run start:dev`启动server开发服务
> 		- `cd ./frontend && npm start`启动前端开发服务
> 	- 访问localhost:8082即可访问项目了
> - 配置postgrest
> ```ini
> # 必填配置 这里尤其需要注意db-schema的配置，找到你创建的workspace的id配置到里面，否则会出现一堆问题
> db-uri = "postgres://postgres:postgres@localhost:5432/tooljet_db"  # 示例：postgres://postgres:pass@localhost:5432/mydb
> db-schema = "postgres,workspace_4873691e-b275-4507-b8d7-2e9631bd17f3,public"
> db-anon-role = "anon"
> 
> # 可选配置
> server-port = 3001         # 默认 3000
> server-host = "::1"  # 默认仅监听本地
> jwt-secret = "6fd2f9e9f2edcbc93c730759a904722af80332744224f7e038891dc89d491af7"  # 如果需要 JWT 认证
> ```
> - `postgrest postgrest.conf地址` 启动postgrest服务
> - 至此，整个项目已经启动完成（纯本地）

#### ToolJet的闪光点
- 整个设计器内充斥着大量的变量注入，组件和数据源的链接就是在创建数据源之后，创建queries，queries的名称就会成为一个变量被注入到当前workspace中，所有数据的绑定就可以通过{{ queriesName.data }}注入组件内，非常方便。相比于AppSmith直接链接查询语句的设定，个人觉得这种方式更为清晰和灵活
- ToolJet在创建queries时还支持js代码转换，也就是说我们可以在查询语句执行完成的结果之上进行数据的处理，很方便
- ToolJet支持常量设置，这个常量可以在整个workspace内部通过上面的模版代码直接读取，放到组件中，这样的话对于各个版本的全局变量配置就是天然支持

#### ToolJet缺点或者相对于AppSmith不够好的点
- 虽然支持多平台，但是需要单独设计，也就是需要在PC和Mobile都进行一次设计，而不是直接通用

#### 尝试创建自定义组件

>自定义组件同样类似lowcode，分为json配置，以及画布渲染组件



>[!ERROR] ToolJet有ce和ee两个版本
>- ce为社区版
>- ee为企业版（也就是收费）
>- 那么ce版本会阉割多少功能，暂时未知

