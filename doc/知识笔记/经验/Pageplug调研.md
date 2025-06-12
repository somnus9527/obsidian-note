---
tags:
  - 知识笔记/经验
---
#### 项目本地启动
> 细节建议参照之前的AppSmith本地启动文档
> 这玩意靠不靠谱，就看前端启动时那一堆报错就很慌，虽然几乎都是prettier报错

##### PC端本地启动
> 后面会说明为什么不安官网步骤来
- `cd app/client`
- 增加npm启动命令
	- `"start:dev": "BROWSER=none EXTEND_ESLINT=true REACT_APP_ENVIRONMENT=DEVELOPMENT REACT_APP_CLIENT_LOG_LEVEL=debug HOST=0.0.0.0 craco --max_old_space_size=7168 start"`
- 修改dev的webpack配置
> [!Info] craco.dev.config.js 增加proxy配置
> ```
> /* eslint-disable @typescript-eslint/no-var-requires */
> const { merge } = require("webpack-merge");
> 
> const common = require("./craco.common.config.js");
> 
> module.exports = merge(common, {
>   devServer: {
>     hot: true,
>     client: {
>       overlay: {
>         warnings: false,
>         errors: false,
>       },
>     },
>     proxy: {
>       "/favicon.ico": {
>         target: "http://localhost:3000",
>       },
>       "/api": {
>         target: "http://localhost:8080",
>         changeOirgin: true,
>       },
>       "/oauth2": {
>         target: "http://localhost:8080",
>         changeOirgin: true,
>       },
>       "/login": {
>         target: "http://localhost:8080",
>         changeOirgin: true,
>       },
>       "/data-gl": {
>         target: "http://localhost:8080",
>         changeOirgin: true,
>       },
>       "/f": {
>         target: "https://cdn.optimizely.com",
>         changeOirgin: true,
>       },
>     },
>   },
>   optimization: {
>     minimize: false,
>   },
> });
> ```

- `yarn install && yarn run start:dev`启动项目

##### 后端本地启动
> 后端本地启动和AppSmith一致
> 但是需要注意，这边链接的mongo名称和之前AppSmith连接的一致
> 所以，如果你之前操作过AppSmith，那么需要先清理数据库中的appsmith库
> 否则migrate data时会出现无法迁移数据的问题
> **如何清理很简单，可以直接vscode安装一个database client，配置不用动（因为我们设置mongo时也是使用的默认），连接之后，找到appsmith这个db，右键删除即可**

#### 组件开发
> 开发方法和AppSmith一致
##### taro项目
> 如果需要玩taro项目，就需要配置nginx服务，并设置https了，因为小程序配置的请求根域名必须要是https
> 这玩意更离谱，最新版的代码里面都没有这块的代码，什么玩意
> 这边不管了，找到上一个Tag**ce-v1.9.35**版本的代码，下下来开整
- `yarn install`
- `yarn dev:weapp`
- 修改`config/dev.js`中的**API_BASE_URL**为本地后端服务地址
>[!Warning] 目前进度
>看起来需要建一个小程序，然后修改配置，将应用id和请求地址改掉，才能正常访问，暂时还不成功（目前我直接用的测试应用）

#### 详细看一下整个dsl的流转逻辑
- [ ] 整体数据流程流转逻辑需要捋清楚
>[!Note] 核心相关文件（叫法沿用了之前lowcode的叫法）
>- `/app/clientsrc/pages/Editor/loader.tsx` 设计器入口文件
>- `/app/clientsrc/pages/Editor/index.tsx` 设计器实际入口文件
>- `/app/client/src/api/PageApi.tsx` 核心service文件 **重点关注savePage**
>- `/app/client/src/sagas/PageSagas.tsx` 核心处理前后端交互的地方 
>	- **savePageSaga**保存dsl入口
>	- **getLayoutSavePayload**dsl转换器入口
>		- **nestDSL** 核心转化逻辑入口
>- `/app/client/src/pages/Editor/PropertyPane/index.tsx`设置器面板入口
>

>[!Note] craco.dev.config.js配置说明
>直接理解成webpack.dev.config.js即可

>[!Warning] 异常记录
>之前的AppSmith其实还存在一个叫RTS的服务，专门处理实时通信（ws）,但是在Pageplug这边，在后端服务启动时可以看见，RTS会报连接失败，所以很明显，后端源码中依然保留了RTS的代码，但是这个项目中RTS的服务源代码却被移除了。。。
>*找taro代码时发现上个版本rts的文件夹还在，最新版本没了。。。taro文件夹也是。。。*

>[!Warning] 未按照官网要求启动本地服务的理由
>- 官网的做法，PC和后端服务在本地，但是代理却做到了nginx配置中(nginx在docker容器启动,然后反向代理宿主机的3000,8080端口)
>- nginx配置做了两件事
>	- 代理
>	- https的配置
>- 我这边的处理很简单，在craco的启动配置中增加了代理（dev配置增加就行，其余环境配置不用动），将请求代理到8080（后端服务的端口），这样直接访问3000即可，为何一定要经过nginx转发，哪怕前后端服务都在docker中，通过volume配置监听文件变更做hotreload，这种场景，在docker中做nginx转发，我都觉得可以接受
>- nginx中还有一个https的配置,这个目前看起来是小程序用的，因为小程序请求地址只能是https域名，暂时小程序看不了，所以直接本地起
