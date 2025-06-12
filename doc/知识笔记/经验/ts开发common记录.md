---
tags:
  - 知识笔记/经验
---
#### 业务需要抽出私有common包，整体采用typescript开发
>[!Note] 恶心点相关文件删除敏感信息之后如下 package.json
> ```JSON
> {
>     "name": "@xxx/xxxxx",
>     "version": "1.0.0",
>     "main": "lib/index.js",
>     "types": "lib/index.d.ts",
>     "repository": "gitxxxxxxxxx",
>     "author": "xxxxxxxxxxxxxx",
>     "license": "MIT",
>     "files": [
>         "lib/**/*"
>     ],
>     "devDependencies": {
>         "@types/jest": "^26.0.23",
>         "@types/node": "^16.0.0",
>         "@types/react": "^17.0.18",
>         "@types/react-dom": "^17.0.9",
>         "@types/react-test-renderer": "^17.0.1",
>         "jest": "^27.0.6",
>         "prettier": "^2.3.2",
>         "react-test-renderer": "^17.0.2",
>         "ts-jest": "^27.0.3",
>         "ts-node": "^10.0.0",
>         "tsconfig-paths": "^3.10.1",
>         "tscpaths": "^0.0.9",
>         "tslint": "^6.1.3",
>         "tslint-config-prettier": "^1.18.0",
>         "typescript": "^4.3.5"
>     },
>     "scripts": {
>         "build:w": "tsc -w",
>         "prebuild": "yarn run format && yarn run lint",
>         "build": "tsc -p tsconfig.json && tscpaths -p tsconfig.json -s ./src -o ./lib",
>         "format": "prettier --write \"src/**/*.ts\" \"package.json\"",
>         "lint": "tslint -p tsconfig.json",
>         "test": "jest --config jestconfig.json",
>         "preversion": "npm run lint",
>         "version": "npm run format && git add -A src",
>         "postversion": "git push && git push --tags",
>         "prepare": "npm run build",
>         "prepublishOnly": "npm test && npm run lint",
>         "login:oppo-npm": "npm login --registry http://xxxxxxxxxxxx",
>         "prepublish:oppo-npm": "node prepublish.js",
>         "publish:oppo-npm": "npm publish --registry http://xxxxxxxxxxx",
>         "postpublish:oppo-npm": "node postpublish.js"
>     },
>     "dependencies": {
>         "react": "16.12.0",
>         "react-dom": "16.12.0"
>     }
> }
> ```

>[!Note] tsconfig.json
> ```JSON
> {
>     "compilerOptions": {
>         "module": "commonjs",
>         "moduleResolution": "Node",
>         "target": "es5",
>         "sourceMap": false,
>         "baseUrl": "./",
>         "outDir": "./lib",
>         "declaration": true,
>         "noImplicitAny": false,
>         "paths": {
>             "@/*": ["src/*"],
>             "@header/*": ["src/header/*"],
>             "@badjs/*": ["src/badjs/*"]
>         },
>         "lib": ["es5","es6","es7","dom"],
>         "typeRoots": [
>             "node_modules/@types"
>         ],
>         "allowJs": true,
>         "checkJs": true,
>         "allowSyntheticDefaultImports": true,
>         "strict": true,
>         "experimentalDecorators": true,
>         "jsx": "react",
>         "esModuleInterop": true
>     },
>     "include": ["src/**/*","tests/**/*"],
>     "exclude": [
>         "node_modules",
>         "**/tests/*"
>     ]
> }
> ```


#### 坑点记录
1. tsc打包不会把我全局定义的声明打到自动生成的声明文件中，导致生成的声明文件中一些原本在自定义的声明文件中的声明丢失，造成报错。    💡 多方查阅资料无果，tsc确实不会在自动生成声明文件时，加入自定义的声明文件，唯一的办法就是将原本的声明文件定义成非.d.ts文件，这样tsc打包时就会去处理自定义声明文件了，但是依然不是一个完美的解决方案，因为打包后会多出很多不必要的文件，但是这个目前比较靠谱的方式。
2. - tsconfig.json中定义paths属性之后，jest测试时却并不能识别别名    💡 办法是在jest配置文件中的moduleNameMapper属性配置相应别名，例如上面配置的别名在jest配置文件中需要配置如下，否则jest会报找不到文件。  
>[!Tip]
> ```
> "moduleNameMapper": {
>     "@/(.*)": "rootDir/src/$1",
>     "@header/(.*)": "rootDir/src/header/$1",
>     "@badjs/(.*)": "rootDir/src/badjs/$1"
>   }
> ```


3. common发布之后，发现子应用安装会把devDependencies内的包也进行安装，这肯定不需要安装。    💡 解决办法目前识别到的有两种  1. 子应用安装时添加production参数，如：npm install xxx —production  2. 像package.json中的操作，将发布命令加到scripts中，然后通过前后钩子，pre钩子里面读取package.json内容，先进行备份，然后重写package.json将devDependencies清空。post钩子里面将备份文件内容再写到package.json中，同时删除备份的package.json即可。推荐这种，尽量不要要求使用者。（当然重写会出现格式问题，可以在合适的钩子里面执行一下format即可）  
4. 发布npm包可以通过.gitignore,.npmignore,package.json的files字段忽略/指定上传文件，优先级从前到后，所以一般都会直接通过package.json的files字段指定上传文件。
5. 三方包开发测试非常麻烦，后来发现一个本地测试的方式    💡 在开发的三方包目录下npm link发布到全局node_modules下，在测试项目里npm link module-name;完成之后在测试目录下即可使用本地的三方包了。如果需要取消link 在开发的三方包目录npm unlink module-name,同时测试项目里也同样需要执行该命令。  这里会出现一个问题，比如library中依赖react，app同时也依赖react，这时启动app，运行页面会报错，原因就是项目中同一个依赖（这里是react）出现了两次（可以通过npm list查看）。（深层原因是这样的，npm link是把library的整个本地文件夹的索引copy到了全局node_modules中，那么app link之后，结果就是node_modules树中确实出现了两个react。那为什么发布之后app安装没问题，是因为真正的发布是不会发布node_modules文件夹的）所以处理办法就是在app中执行命令：npm link /node_modules/react即可解决。当然通过yalc测试也可以，只是相对npm link使用比较复杂，但是不会有这个问题。