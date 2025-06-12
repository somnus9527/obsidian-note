---
tags:
  - 知识笔记/经验
---
#### 使用记录

>[!Info] 记录如下：
>下载安装依赖之后，可以发现gulpfile.js里有一个定义如下：
>```
>gulp.task(
  'dapDebugServer',
   gulp.series('clean', 'compile:static', 'dapDebugServer:webpack-bundle'),
);
>```
>所以在package.json的scripts中添加`"compile:dapServer": "gulp dapDebugServer"`,然后执行compile:dapServer,就可以打包dap的可执行文件，主要入口文件是dist/src/dapDebugServer.js，然后我们就可以cd进dist/src中执行命令`node dapDebugServer.js 4711`即可启动DAP服务，注意这个服务是一个socket服务。

