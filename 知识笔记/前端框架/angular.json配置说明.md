---
tags:
  - 知识笔记/前端框架/angular
---
#### angular.json
```json
{
  "version": "string", // 配置文件的版本
  "newProjectRoot": "string", // 新创建的工程的位置 [1]
  "cli": {
    // 自定义 Angular CLI 的选项，用于在这个工作空间中执行的 Angular CLI 命令
    "analytics": "boolean", // 给 Angular 团队分享使用数据
    "analyticsSharing": {
      // 分享使用数据的选项
      "tracking": "string", // 分析共享信息跟踪 ID
      "uuid": "string" // 分析共享信息 UUID（通用唯一标识符）
    },
    "cache": {
      // 构建缓存
      "enabled": "boolean = true", // 是否启用缓存
      "environment": "string: local | ci | all = local", // 在哪个环境启用缓存
      "path": "string = .angular/cache" // 缓存路径
    },
    "schematicCollections": "string[]", // 默认原理图集
    "packageManager": "string: npm | cnpm | pnpm | yarn", // 包管理工具
    "warnings": {
      // 警告信息的显示控制
      "versionMismatch": "boolean = true" // 当全局 Angular CLI 版本比本地版本更新时显示警告
    }
  },
  "schematics": "object", // 原理图
  "projects": {
    // 项目配置
    "<project>": {
      "root": "string", // 根目录
      "sourceRoot": "string",
      "projectType": "string: application | library",
      "prefix": "string", // Angular CLI 创建组件或指令时添加选择器前缀
      "schematics": "object", // Angular CLI 创建组件或指令时的原理图
      "architect": {
        // 构建器目标配置默认值
        "build": "object", // 构建，ng build 命令的配置
        "serve": "object", // 服务，覆盖构建，ng serve 命令的配置
        "e2e": "object", // E2E，覆盖构建，ng e2e 命令的配置
        "test": "object", // 测试，覆盖构建，ng test 命令的配置
        "lint": "object", // 格式，ng lint 命令的配置
        "extract-i18n": "object", // I18N，ng extract-i18n 命令的配置
        "server": "object", // 服务器，ng run <project>:server 命令的配置
        "app-shell": "object" // PWA，ng run <project>:app-shell 命令的配置
      }
    }
  }
}
```

#### builder相关
```json
{
  "builder": "string = @angular-devkit/build-angular:browser", // 构建工具使用的 NPM 包
  "options": {
    // 命令选项
    "main": "string", // 入口文件路径
    "index": "string|object", // index.html 文件路径
    "outputPath": "string", // 输出路径
    "resourcesOutputPath": "string", // 资源输出路径，相对于 outputPath
    "polyfills": "string|string[]", // 垫片文件
    "tsConfig": "string", // tsc 配置文件路径
    "inlineStyleLanguage": "string: css|less|sass|scss", // 内敛样式的格式
    "optimization": "object", // 优化
    "sourceMap": {
      // 源文件地图
      "scripts": "boolean = true", // 脚本
      "styles": "boolean = true", // 样式
      "vendor": "boolean = false", // node_modules 脚本和样式
      "hidden": "boolean = false" // 错误报告工具
    },
    "aot": "boolean = true", // 预先编译
    "vendorChunk": "boolean = false", // 为 node_modules 中的文件创建一个单独的包
    "commonChunk": "boolean = true", // 为多次被用到的常用代码创建一个单独的包
    "baseHref": "string", // 基地址
    "verbose": "boolean = false", // 输出日志更加详细
    "progress": "boolean = true", // 输出构建进度
    "i18nMissingTranslation": "string: warning|error|ignore", // I18N 翻译缺失的处理方式
    "i18nDuplicateTranslation": "string: warning|error|ignore", // I18N 翻译重复的处理方式
    "localize": "boolean|string[]", // 需要本地化的语言
    "watch": "boolean = false", // 监听模式，文件变化时自动重新构建
    "outputHashing": "string: none|all|media|bundles = none", // 定义输出文件名的缓存无效哈希的模式
    "poll": "number", // 监听模式的轮询周期
    "deleteOutputPath": "boolean = true", // 构建前删除输出目录
    "preserveSymlinks": "boolean", // 当解析模块时，不要使用真实路径。如果未设置，则如果 NodeJS 启用了 --preserve-symlinks 选项，就默认为 true
    "extractLicense": "boolean = true", // 提取所有证书到一个单独文件中
    "buildOptimizer": "boolean = true", // 使用 AOT 时是否启用构建优化
    "namedChunks": "boolean = false", // 对于惰性模块，使用文件名命名
    "serviceWorker": "boolean = false", // 在生产环境中生成一个 Service Worker
    "ngswConfigPath": "string", // Service Worker 配置文件路径
    "statsJson": "boolean = true", // 生成打包分析数据文件
    "webWorkerTsConfig": "string", // Web Worker 的 Typescript 配置文件路径
    "crossOrigin": "string: none|anonymous|use-credentials = none", // 支持 CORS 的元素的跨域属性的默认值
    "allowedCommonJsDependencies": "string[]", // 允许使用且不会在构建时发出警告的 CommonJs 包

    // 无法从命令行传递的额外选项
    "assets": "(string|object)[]", // 全局静态文件路径
    "styles": "(string|object)[]", // 全局样式文件路径
    "stylePreprocessorOptions": "object", // 样式预处理器选项
    "scripts": "(string|object)[]", // 全局脚本路径
    "budgets": "object[]", // 输出文件大小预算
    "fileReplacements": [
      // 文件路径替换 [1]
      {
        "replace": "string", // 原本文件路径
        "with": "string" // 目标文件路径
      }
    ]
  },
  "configuration": {
    // 不同环境下的配置，默认可选：development 和 production [2]，用于在不同环境下覆盖`options`中的选项
    "development": "object",
    "production": "object"
  },
  "defaultConfiguration": "string" // 默认环境
}
```

#### 一些细节配置
>[!Info] index **projects.project.architect.build.options.index**
>```jsonc
>{
  "input": "string", // 输入路径
  "output": "string" // 输出路径
}
>```

>[!Info] assets **projects.project.architect.build.options.assets**
>```jsonc
>{
>  "glob": "string", // 匹配模式
>  "input": "string", // 基准路径
>  "output": "string", // 输出路径，相对于 outputPath/assets
>  "ignore": "string", // 忽略的匹配模式
>  "followSymlinks": "boolean = false" // 允许跟踪符号链接
>}
>```

>[!Info] styles&scripts **projects.project.architect.build.options.[styles|scripts]**
>```jsonc
>{
  "input": "string", // 路径
  "inject": "boolean", // 是否注入到 HTML 文件
  "bundleName": "打包名称"
}
>```

>[!Info] optimization **projects.project.architect.build.options.optimization**
>```jsonc
>{
>    "scripts": "boolean = true", // 脚本输出优化
>     "styles": {
>	"minify": "boolean = true", // 移除多余的空格和注释、合并标识符
>	"inlineCritical": "boolean = true" // 提取并内联一些关键 CSS 定义
>    },
>    "fonts": {
> 	   // 也可以为 true
> 	   "inline": "boolean = true" // 内联外部 Google 字体和 Adobe 字体的 CSS 定义
>    }
>}
>```

>[!Info] budgets **projects.project.architect.build.options.budgets**
>```jsonc
>{
  "type": "string: bundle|initial|allScript|all|anyComponentStyle|anyScript|any", // 类型
  "name": "string", // type=bundle 时有效
  "baseline": "string", // 一个表示基准大小的绝对值，用做比例值的基数
  "maximumWarning": "string", // 当大小超过基线的这个阈值百分比时给出警告
  "maximumError": "string", // 当大小超过基线的这个阈值百分比时报错
  "minimumWarning": "string", // 当大小小于基线的这个阈值百分比时给出警告
  "minimumError": "string", // 当大小小于基线的这个阈值百分比时报错
  "warning": "string", // 当大小达到或小于基线的这个阈值百分比时都给出警告
  "error": "string" // 当大小达到或小于基线的这个阈值百分比时都报错
}
>```

>[!Info] ng server命令的配置
>```jsonc
>{
>  "builder": "string = @angular-devkit/build-angular:dev-server", // 构建工具使用的 NPM 包
>  "options": {
>    // 命令选项
>    "browserTarget": "string", // 构建目标，project:target:configuration的格式，configuration 可以用逗号连接多个值
>   "port": "number = 4200", // 端口号
>    "host": "string = localhost", // 域名
>    "proxyConfig": "string", // 代理配置文件路径
>    "ssl": "boolean = false", // 使用 HTTPS
>    "sslCert": "string", // HTTPS 证书路径
>    "headers": "object", // Headers
>    "open": "boolean = false", // 自动打开浏览器
>    "liveReload": "boolean = true", // 文件变更时自动重载
>    "publicHost": "string", // 开发服务器使用的域名
>    "allowedHosts": "string[]", // 允许访问开发服务器的域名
>    "servePath": "string", // 应用所在路径
>    "disableHostCheck": "boolean = false", // 禁用域名检查
>    "hmr": "boolean = false" // 模块热替换
>  }
>}
>```

