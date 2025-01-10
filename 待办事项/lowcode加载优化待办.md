---
tags:
  - 待办/工作待办
创建日期: 2024-12-28T09:30:00
done:
---
- [ ] 当前主应用打包存在同时打es2015和es5两个版本代码的情况
>[!Success] 背景&解决方式
>- 背景：
>angular12项目,使用`@angular-builders/custom-webpack:browser`进行打包，发现生成的资源文件中，js文件都有两份，一份es5一份es2015，但是项目只需要支持现代浏览器，所以可以取消es5那一份的打包
>- 解决方式
>	- 增加browserslist文件，内容见Tip
>	- tsconfig.json中，将compilerOptions.target改为es2015
- [x] [[lowcode加载缓慢优化记录|lowcode加载目前的缓存逻辑没有加载到lowcode引擎内部的部分资源]]


>[!Tip] browserlist文件内容
>```browserlist
>
>```