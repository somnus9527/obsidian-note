---
tags:
  - 知识笔记
---
#### 调研记录
1. 分别下载code-server和vscode源码
	- vscode: [https://github.com/microsoft/vscode.git](https://github.com/microsoft/vscode.git)
	- code-server: [https://github.com/coder/code-server.git](https://github.com/coder/code-server.git)
2. 初始化vscode源码项目
3. 安装依赖失败，原因是electron和@playwright/test需要下载Chromium内核项目，下载非常慢，所以使用镜像下载,但是淘宝镜像已经证书失效，这里采用腾讯云镜像，但是由于@playwright/test包写死了下载Chromium内核的地址，所以没办法，只能等。。。（会很慢，甚至报网络错误，疯狂retry🤪，耐心！耐心！耐心！）

4. 初始化code-server源码项目

5. 发现需要将vscode源码拷贝到code-server源码目录中的lib文件夹中，替换掉下载源码之后就会有的vscode目录，所以建议还是先替换，再分别安装包，先装vscode再装code-server
6. 开始编译code-server和vscode
7. 在code-server中编译vscode，这边两个问题：
8. build-vscode.sh中存在git操作，但是由于之前git clone非常慢，所以我是直接下载的zip包，这里就需要注视掉所有git相关的shell命令（当然注释的时候，注意一下你注释的命令干了啥，有一些是通过git获取最新的打包配置文件的，那么相应的文件就一定不要修改）
9. 发现build-vscode.sh中还存在jq操作，所以还需要安装jq命令行工具。。。
	- mac: brew install jq
	- win: choco install jq,当前这边依赖choco,如果你使用的scoop需要去查一下scoop文档怎么装jq，如果choco和scoop都没有，那么自己去下载jq并安装
10. 可以开始编译了。。。
11. 先yarn run build:vscode开始编译vscode源码（啰嗦一句，这边前提是vscode源码，已经放到了code-server/lib/vscode中，并且执行成功yarn install），编译还是很慢。。。是非常非常非常慢。。。
12. 编译50min报错。。。
>[!Error] 报错信息：
>```
[16:13:11] Error: /Users/wangxuepei/projects/dingjie-work/athena-vscode/code-server/lib/vscode/src/vs/platform/telemetry/test/browser/telemetryService.test.ts(20,31): Argument of type 'sinon.SinonStatic' is not assignable to parameter of type 'import("/Users/wangxuepei/projects/dingjie-work/athena-vscode/code-server/lib/vscode/node_modules/@types/sinon-test/node_modules/@types/sinon/index").SinonStatic'.
>Type 'SinonStatic' is not assignable to type 'SinonSandbox'.
>The types of 'usingPromise(...).replace' are incompatible between these types.
>Property 'usingAccessor' is missing in type '<T, TKey extends keyof T, R extends T[TKey] = T[TKey]>(obj: T, prop: TKey, replacement: R) => R' but required in type
>'SandboxReplace'.
>```

13. 发现是ts类型异常，但是实际类型确实是匹配的，只是由于namespace声明+union定义造成了typescript类型推断失败，所以先使用const sinon = require('sinon');放弃sinon相关的typescript类型校验能力，开始重新编译。。。
14. 同样ts校验问题，ide中提示并没有typescript类型校验错误，但是打包时却发现typescript校验就出现异常；（问题原因见记录第三条）

#### 备注
1. 或许electron安装慢的问题可以通过设置electron_mirror加快electron安装，但是不确定，我这边是挂的梯子，注意打开verbose，查看下载地址，如果下载的地址和你设置的不一致，检查一下代理设置是否正确，如果确认无误，那么就把yarn.lock删了吧，就是这玩意影响了下载地址
2. 建议所有命令有verbose就开verbose，大部分命令都很慢，否则长时间不动，都不知道到底是卡住了还是慢
3. ts校验问题来自从code-server启动造成。就是说，首先vscode源码依赖的typescript版本是**^5.6.0-dev.20240618，**但是从code-server启动时typescript版本却不一致，那么gulp启动时，由于是从code-server启动，采用的就是code-server依赖的版本，而不是vscode自己依赖的版本，造成类型校验异常。
4. 鉴于第三条的问题，且code-server又不能改为和vscode源码依赖的typescript版本，因为同样会报错，那么之前的初始化预想就出问题了。最后发现vscode源码中直接存在code-server.sh，也就是说不需要依赖code-server源码，所以直接删除code-server，直接使用vscode源码统一处理。另外打包脚本为`yarn run compile-web`,dev脚本为`yarn run watch-web`,同时需要运行本次自定义命令code-server`"code-server": "./scripts/code-server.sh --launch"`，来启动code-server的服务，这样就可以通过浏览器`[http://localhost:9888/](http://localhost:9888/)`访问我们本地的vscode了。