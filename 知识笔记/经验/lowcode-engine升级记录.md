---
tags:
  - 知识笔记/经验
  - 知识笔记/前端框架/lowcode
---
>[!Warning] 目标
> - [ ] 升级lowcode内部react版本到react18 **关键目标**
> 	- [ ] 升级webpack到webpack5  **附属目标**
> 		- [ ] 移除build-script  **附属目标**
> 	- [ ] 移除lerna  **附属目标**
> 	- [ ] 减少子包数量  **附属目标**
> 	- [x] 升级原主项目node版本到18
> 	- [ ] 移除fuison-ui (@alifd/next)改用antd
> 	- [ ] 增加lowcode内部国际化能力
> 	- [ ] 移除moment改用dayjs
> 	- [ ] 移除lodash改用lodash-es
> 	- [ ] 移除阿里原有打包系统的依赖，重构整个打包系统
> 	- [ ] lowcode事件系统从直接代理到document改为代理到挂载节点

### 前置准备

#### 升级node版本到18
> 理由：
> 	想升级react到18,虽然本身和node版本并不冲突，但是react升级之后，它背后可能用到的生态很大一部分都会强依赖node版本>=18(之前尝试MF发现的)，为了后续升级的可操作性更大，先把现有的node16升到18很有必要

>[!Error] 升级18报错
> ```
> Error: error:0308010C:digital envelope routines::unsuppor
> 
> ted
> 
>     at new Hash (node:internal/crypto/hash:69:19)
> 
>     at Object.createHash (node:crypto:133:10)
> 
>     at BulkUpdateDecorator.hashFactory (/Users/somnuszyy952
> 
> 7/workspace/dingjie/athena_designer_web/apps/main/node_modu
> 
> les/@angular-devkit/build-angular/node_modules/webpack/lib/
> 
> util/createHash.js:145:18)
> 
>     at BulkUpdateDecorator.update (/Users/somnuszyy9527/wor
> 
> kspace/dingjie/athena_designer_web/apps/main/node_modules/@
> 
> angular-devkit/build-angular/node_modules/webpack/lib/util/
> 
> createHash.js:46:50)
> ```
> 可以看到升到18，启动主应用报createHash的错，**根本原因就是node17+启用了OpenSSL3.0**,而我们使用到的@angular-devket等包没有兼容这个特性造成报错。
> 但是升级node18我并不希望对主项目有任何更改，所以通过配置node参数解决这个问题`NODE_OPTIONS=--openssl-legacy-provider`
> 加上这个参数之后，再次启动主应用发现运行堆栈溢出
> ```
> <--- JS stacktrace --->
> FATAL ERROR: Reached heap limit Allocation failed - JavaS
> 
> cript heap out of memory
> 
>  1: 0x100f53c40 node::Abort() [/usr/local/bin/node]
> 
>  2: 0x100f53e24 node::ModifyCodeGenerationFromStrings(v8:
> 
> :Local<v8::Context>, v8::Local<v8::Value>, bool) [/usr/lo
> 
> cal/bin/node]
> 
>  3: 0x1010ab608 v8::internal::V8::FatalProcessOutOfMemory
> 
> (v8::internal::Isolate*, char const*, bool) [/usr/local/b
> 
> in/node]
> ```
> 同样，node18启用了新的v8引擎，新的v8引擎默认就会限制最大内存（约 1.5 GB 左右），所以还要加载内存参数，增加运行内存，再加上之前的参数，合并如下：
> `NODE_OPTIONS='--max-old-space-size=4096 --openssl-legacy-provider'`
> 再次运行发现已经可以正常运行了，访问项目也没有发现问题。升级node18完成

#### 构建monorepo工程文件夹
- 利用pnpm的workspace取代源码中的lerna，减少心智负担
- 在工程目录顶层，新增babel.config.js,tsconfig.base.json,tsconfig.cjs.json,tsconfig.esm.json,webpack.base.js,webpack.config.js,pnpm-workspace.yaml，.eslintrc.js, prettierrc.js
- 以上除了pnpm-workspace.yaml是为了定义整个工作空间，其余都是为了子应用复用
- 优化pnpm-workspace.yaml中的catalog，保证整个工程的一些核心依赖的版本一致，后续出现新的也会同步更新
### 开始升级
#### 处理lowcode-types包
- 拷贝源码进来命名为@athena/lowcode-types包
- 处理源码内部报错
	- 源码内部存在@alifd/next的使用，暂时直接替换为antd和@ant-design/icons,后续实际使用的地方需要处理
- 优化package.json配置 （考虑后续打包成npm格式的dist输出）
- 打包处理
	- 发现整个lowcode的打包都借助原来阿里的一套打包脚本，那一套打包脚本并不适配，所以也需要自己处理打包
#### 处理打包脚本
>在monorepo中packages目录同级，新增scripts目录，内部统一放置整个monorepo工程的打包脚本

##### 开始编写打包脚本

	- 构建tsc输出申明文件脚本

#### 记录

- 整个工程是一个monorepo，采用lerna管理
- 内部packages拆分了大量的子工程 **核心内容都在packages**
- 另外还有一个modules目录，内部的两个应用是lowcode的配套工具，不属于核心包
- 考虑整个工程如何接入后续项目
	- 前置背景：不接入公共npm仓库、无私有npm仓库
	- 经过测试目前暂定如下方式（需要架构确认）
		- packages内部各个包打包成npm格式的资源结构
		- 将各个资源打包成zip包放到指定服务器上
		- 调用方通过package.json直接连接服务器上的zip包
		- 以上为生产模式的运行方式
		- 如果是开发模式，比如修改lowcode功能之类的
		- 通过配置webpack的resolve.alias指定地址到本地的monorepo各个子包目录的dist
		- 这样只要各个子包通过watch模式启动，实时打包生成到dist,这样就是实现hotreload
		- 以上的整个打包模式，其实和普通的打包区别不小，比如生产环境需要在打包完成之后自动生成zip包，比如开发模式也需要watch并实时将资源输出到dist，这样外部接入的应用才能实时监听到lowcode的代码变更。所以看下来，整个打包逻辑相对还是比较复杂，考虑在monorepo中增加scripts目录，统一处理各个package的打包和debug

### TODO
- [ ] @alifd/next使用的地方待处理