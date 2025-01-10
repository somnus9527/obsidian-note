---
tags:
  - 知识笔记/经验
---
>[!Tip] 
>1. 本地开发使用npm link 联调时，由于link会将本地library整个项目链接到全局node_modules中，那么在app中npm link package-name之后，一旦library跟app同时使用一个依赖，则会报错Duplicate dependencies.这时需要在使用library的app中通过npm link appPath/to/libraryPath/node_modules/module_name。比如library和app在同一个文件夹中，并且同级，且假如library中peerDependencies依赖了react，app应用项目中也依赖react，那么就需要在app目录中执行：npm link ../libraryName/node_modules/react；可以消除报错。
>2. 针对复杂包（有额外依赖）tree-shakeable失效的处理
>3. 优化
>4. library开发联调还是以library: npm link app中: npm link libraryName进行比较方便
>5. 测试发布可以通过本地包工具yalc进行
>6. 打包脚本中直接`process.env.NODE_ENV`设置环境变量无效，因为rollup打包是多进程，还是通过cross_env包设置相对方便
>7. 针对分包之后，是否排除出依赖包视情况而定，可以放到peerDependencies中，也可以直接加入dependencies中，建议放入peerDependencies中，减少因为app和library同依赖但是版本号不同造成的打包进去同一个依赖的不同版本号。只是使用peerDependencies时需要注意external和globals的配置，如果你使用了rollup-plugin-peer-deps-external则只需要关注globals的配置，**注意不是因为不需要处理external，而是这个插件帮你自动处理了**

#### 具体配置如下
```
### Rollup打包Library记录

1. 项目目录结构

- __project_root_path__
   - __config__
     - [rollup.config.ts](config/rollup.config.ts)
   - __lib__
   - [node\_modules](node_modules)
   - __scripts__
   - __src__
     - __@types__
     - __assets__
     - __component__
       - [index.tsx]
       - __scss__
         - [component.module.scss]
         - [component.scss]
     - [index.ts](src/index.ts)
     - __tools__
   - [tsconfig.json](tsconfig.json)
   - [tslint.json](tslint.json)
   - [yarn.lock](yarn.lock)
   - [.gitignore](.gitignore)
   - [.npmrc](.npmrc)
   - [.prettierrc](.prettierrc)
   - [package.json](package.json)
   - [README.md](README.md)

2. 目录结构说明
   - config 
     - > rollup打包配置相关文件
   - lib
     - > 打包生成的文件
   - scripts
     - > 打包脚本文件
   - src
     - > 项目源代码
       - > @types 全局声明文件
       - > assets 静态资源文件
       - > header 具体模块代码
         - > scss 样式文件
       - > tools 工具文件夹
       - > index.ts 入口文件

3. 单独文件说明
    
    - rollup.config.ts
      - > 配置文件
        ```typescript
            // 打包commonjs插件
            import commonjs from '@rollup/plugin-commonjs';
            // 处理json插件
            import json from '@rollup/plugin-json';
            // 处理图片插件 这里是把图片打包成base64，打进js中，如果需要输出静态资源，应该通过rollup-plugin-copy输出
            import image from '@rollup/plugin-image';
            // 打包ts插件
            import typescript from 'rollup-plugin-typescript2';
            // 压缩代码插件
            import { terser } from 'rollup-plugin-terser';
            // 删除文件夹插件
            import del from 'rollup-plugin-delete';
            // 处理项目中对于node模块的引用插件
            import resolve from 'rollup-plugin-node-resolve';
            // 自动处理peerDependencies中的模块，加入到rollup配置的external中
            import peerDeps from 'rollup-plugin-peer-deps-external';
            // 处理ts声明文件中的别名插件
            import { typescriptPaths } from 'rollup-plugin-typescript-paths';
            // 处理ts代码中的别名插件
            import pathsTransformer from "ts-transform-paths";
            // 处理对于css,less,scss等样式文件的打包，这里只使用了scss所以只安装了node-sass如果需要处理less需要安装less模块，其余样式如果有依赖也是如此
            import css from 'rollup-plugin-postcss';
            // 同webpack的autoprefixer，增加各个浏览器样式的hack
            import autoprefixer from 'autoprefixer';
            // 处理ts声明文件中的别名
            import ttypescript from 'ttypescript';
            
            import packageJson from '../package.json';

            export default {
                // 打包入口文件
                input: "src/index.ts",
                // 打包输出文件列表
                output: [
                    // 输出commonjs模块，声明的globals是依赖于应用项目中react
                    {
                        file: packageJson.main,
                        format: "cjs",
                        globals: {
                            react: "React"
                        }
                    },
                    // 输出esmodule模块
                    {
                        file: packageJson.module,
                        format: "es",
                        globals: {
                            react: "React"
                        }
                    },
                    // 输出umd模块
                    {
                        file: packageJson.umd,
                        format: "umd",
                        name: "LibraryName",
                        globals: {
                            react: "React"
                        }
                    }
                ],
                // 插件顺序执行
                plugins: [
                    // 打包前先行清除之前的输出目录
                    del({
                        targets: "lib/*"
                    }),
                    // 先行处理ts,tsx文件中的别名
                    typescriptPaths({
                        preserveExtensions: true
                    }),
                    // 处理json文件的引用
                    json(),
                    // 增加node依赖
                    resolve({
                        mainFields: ["jsnext", "preferBuiltins", "browser"]
                    }),
                    // 处理peerDependencies
                    peerDeps(),
                    // 处理commonjs输出
                    commonjs(),
                    // 处理图片打包
                    image(),
                    // 处理样式文件打包
                    css({
                        plugins: [
                            // 打包增加浏览器hack
                            autoprefixer()
                        ]
                    }),
                    // 打包ts
                    typescript({
                        // 增加ttc专门处理别名
                        typescript: ttypescript,
                        useTsconfigDeclarationDir: true,
                        // transformers处理别名的转换
                        transformers: [
                            (service) => pathsTransformer()
                        ]
                    }),
                    // 压缩输出的代码
                    terser(),
                ]
            }
        ```
    - build_path.ts
      - > 打包地址集合
        ```typescript
            import path from 'path';
            export default {
                // rollup配置路径 watch.ts中需要加载配置
                config: path.resolve(__dirname,'../config/rollup.config.ts')
            }
        ```
    - load_config.ts
      - > watch.ts中加载rollup配置的方法
        ```typescript
            import loadConfigFile from "rollup/dist/loadConfigFile";
            import buildConfig from "./build_path";

            interface IBuildConfig {
                options: any;
                warnings: {
                    add: (warning: any) => void;
                    readonly count: number;
                    flush: () => void;
                    readonly warningOccurred: boolean;
                }
            }

            async function loadConfig(): Promise<IBuildConfig> {
                return await loadConfigFile(buildConfig.config);
            }

            export {
                loadConfig,
                IBuildConfig,
            }  
        ```
    - tools.ts 
      - > 时间和日志输出工具方法
        ```typescript
            import { RollupError } from 'rollup';
            import chalk from 'chalk';

            enum ELogType {
                INFO,
                ERROR,
            }

            function getNow(): string {
                const now: Date = new Date();
                const year: number = now.getFullYear();
                const month: number = now.getMonth() + 1;
                const date: number = now.getDate();
                const hour: number = now.getHours();
                const minutes: number = now.getMinutes();
                const seconds: number = now.getSeconds();
                return `${year}-${month}-${date} ${hour}:${minutes}:${seconds}`;
            }

            function log(info: string | RollupError,type: ELogType = ELogType.INFO) {
                const colorInfo: string = type === ELogType.INFO ? chalk.blue(info) : chalk.yellow(info);
                console.log(chalk.green(`[${getNow()}]`),colorInfo);
            }

            export {
                ELogType,
                getNow,
                log,
            }
        ```
    - watch.ts
      - > 以监听模式启动rollup打包脚本
    
        ```typescript
            import { RollupWatcher , RollupWatcherEvent, watch } from "rollup";
            import { loadConfig , IBuildConfig } from './load_config';
            import { log , ELogType } from './tools';

            let firstStartUp: boolean = true;
            let buildTimes: number = 1;

            async function watchBuild() {
                const buildConfig: IBuildConfig = await loadConfig();
                const watcher: RollupWatcher = watch({
                    ...buildConfig.options[0],
                    watch: {
                        // 监听到文件变化，重新打包的延迟时间
                        buildDelay: 500
                    }
                });

                // watch模式打包触发的事件处理
                watcher.on('event',async ( event: RollupWatcherEvent ) => {
                    // rollup启动
                    if(event.code === 'START') {
                        if(firstStartUp)
                            log('-------------------------------------------------------');
                        log(firstStartUp ? '打包开始...' : '监听到变更，重新开始打包...');
                    }
                    // 打包出错
                    if(event.code === 'ERROR') {
                        log(event.error,ELogType.ERROR);
                        if(watcher.close) {
                            watcher.close();
                        }
                    }
                    // 打包开始
                    if(event.code === 'BUNDLE_START') {
                        // log('开始进行声明文件别名转换...');
                        // await transformAlias();
                        log(`打包入口：${event.input && event.input.length > 0 ? event.input : '空'} -> 即将输出文件：${event.output.join(', ')}...`);
                    }
                    // 打包结束
                    if(event.code === 'BUNDLE_END') {
                        log(`成功创建文件：${event.output.join(', ')}`);
                        log(`打包耗时：${event.duration/1000}秒`);
                        log(`打包完成,已完成${buildTimes}次...`);
                        log('-------------------------------------------------------');
                        log('监听文件修改中...');
                        log('-------------------------------------------------------');
                        buildTimes++;
                        firstStartUp = false;
                    }
                })
            }
            watchBuild();
        ```
    - package.json
        ```json
            {
              "name": "project-name",
              "main": "lib/index.cjs.js",
              "module": "lib/index.esm.js",
              "umd": "lib/index.umd.js",
              "types": "lib/index.d.ts",
              "version": "1.0.0",
              "repository": "",
              "path": ".",
              "author": "",
              "license": "MIT",
              "files": [
                "lib/**/*"
              ],
              // 不打包的，但是使用的app中必须要有的包
              "peerDependencies": {
                "react": "^17.0.2"
              },
              // 这些包会打包到library中
              "dependencies": {
              },
              "devDependencies": {
                "@rollup/plugin-commonjs": "^21.0.0",
                "@rollup/plugin-image": "^2.1.1",
                "@rollup/plugin-json": "^4.1.0",
                "autoprefixer": "^10.3.7",
                "chalk": "^4.1.2",
                "node-sass": "^6.0.1",
                "postcss": "^8.3.9",
                "react": "^17.0.2",
                "rollup": "^2.58.0",
                "rollup-plugin-delete": "^2.0.0",
                "rollup-plugin-node-resolve": "^5.2.0",
                "rollup-plugin-peer-deps-external": "^2.2.4",
                "rollup-plugin-postcss": "^4.0.1",
                "rollup-plugin-terser": "^7.0.2",
                "rollup-plugin-typescript-paths": "^1.3.0",
                "rollup-plugin-typescript2": "^0.30.0",
                "ts-node": "^10.3.0",
                "ts-transform-paths": "^2.0.3",
                "tslib": "^2.3.1",
                "ttypescript": "^1.5.12",
                "typescript": "^4.4.3",
                "typescript-transform-paths": "^3.3.1"
              },
              "scripts": {
                "build": "rollup -c config/rollup.config.ts && ttsc --emitDeclarationOnly",
                "dev": "ts-node scripts/watch.ts"
              }
            }

        ```
    - tsconfig.json
        ```json
            {
                "compilerOptions": {
                    "module": "esnext",
                    "target": "es5",
                    "sourceMap": false,
                    "baseUrl": "./",
                    "outDir": "lib",
                    // 是否自动打包除声明文件
                    "declaration": true,
                    "declarationDir": "lib",
                    "noImplicitAny": false,
                    "moduleResolution": "Node",
                    "paths": {
                        "@rosiky/*": ["src/rosiky/*"],
                    },
                    "lib": ["es5","es6","es7","dom"],
                    "typeRoots": [
                        "src/@types",
                        "node_modules/@types"
                    ],
                    "allowJs": true,
                    "checkJs": true,
                    // 允许默认导入
                    "allowSyntheticDefaultImports": true,
                    // 严格模式
                    "strict": true,
                    // 允许装饰器
                    "experimentalDecorators": true,
                    "jsx": "react",
                    "esModuleInterop": true,
                    // 转换申明文件别名
                    "plugins": [
                        {
                          "transform": "ts-transform-paths",
                          "afterDeclarations": true
                        }
                      ]
                },
                "ts-node": {
                    // ts-node执行时使用的配置
                    "compilerOptions": {
                        "module": "commonjs"
                    }
                },
                "include": ["src/**/*"],
                "exclude": [
                    "node_modules",
                    "**/tests/*",
                    "lib"
                ]
            }
        ```
4. 问题修复
   - tree-shakeable无效
     - > 按官网所说，esModule是tree-shakeable的必要条件，其余 package.json中需要设置 ```sideEffects:  false```,意思是我的包没有副作用，可以通过依赖搜索确定tree-shakeable删除的内容；但是全部设置完成后，一旦我引入依赖包，无论是dependencies还是peerDependencies，经过测试发现tree-shakeable都会失效，也就是说如果我的包没有任何依赖，这种情况下，只要package.json的设置和esmodule都正确，webpack打包的app已经可以正常tree-shakeable（这边的测试只是简单的方法，是否真的一定可以，并不确定），而有依赖则发现无效。
     - > 由于有依赖的前提，这是根据rollup官网的文档说明以及stackoverflow上的建议，进行分包。首次通过```preserveModule: true```进行分包，但是发现分包的场景，打包之后，对于依赖包的引用都会出现问题，找不到依赖包。原因是包发布之后node_modules并不会跟着发布，那么app在引用时就会出现找不到依赖的情况。并且preserveModule也会造成对于node_modules依赖引用的路径错误。
     - > 鉴于上面的尝试不成功，重新进行多入口分包的尝试，结果发现可行。多入口之后，配合之前的设置，app引入之后不仅代码有效的tree-shaking，并且不同模块的依赖也有效的tree-shaking了。
    
    > 具体修改内容如下：
    > 首先是配置修改为多入口
    ```typescript
        import commonjs from '@rollup/plugin-commonjs';
        import json from '@rollup/plugin-json';
        import image from '@rollup/plugin-image';
        import typescript from 'rollup-plugin-typescript2';
        import { terser } from 'rollup-plugin-terser';
        import del from 'rollup-plugin-delete';
        import buildin from 'rollup-plugin-node-builtins';
        import resolve from 'rollup-plugin-node-resolve';
        import peerDeps from 'rollup-plugin-peer-deps-external';
        import { typescriptPaths } from 'rollup-plugin-typescript-paths';
        import pathsTransformer from "ts-transform-paths";
        // 增加打包进度详情
        import progress from 'rollup-plugin-progress';
        import css from 'rollup-plugin-postcss';
        import autoprefixer from 'autoprefixer'
        import ttypescript from 'ttypescript';
        import path from 'path';
        import { Plugin, PreRenderedChunk, RollupOptions } from 'rollup';

        interface IStringConfig {
            [propName: string]: string;
        }

        const plugins = (): (Plugin | null | false | undefined)[] => {
            const env: string|undefined = process.env.NODE_ENV;
            const basePlugins:(Plugin | null | false | undefined)[] = [
                progress({
                    clearLine: false,
                }),
                buildin(),
                typescriptPaths({
                    preserveExtensions: true
                }),
                json(),
                resolve({
                    mainFields: ["jsnext", "preferBuiltins", "browser"]
                }),
                peerDeps(),
                commonjs(),
                image(),
                css({
                    plugins: [
                        autoprefixer()
                    ]
                }),
                typescript({
                    typescript: ttypescript,
                    useTsconfigDeclarationDir: true,
                    transformers: [
                        (service) => pathsTransformer()
                    ]
                })
            ];

            if(env === 'development') {
                basePlugins.unshift(del({
                    targets: 'lib/*'
                }));
                return basePlugins;
            } 
            return basePlugins.concat(terser());
        };

        const globals: IStringConfig = {
            "react": "React",
            "axios-esm": "axios",
        };

        const input: IStringConfig = {
            "index": path.resolve(__dirname,'../src/index.ts'),
            "header": path.resolve(__dirname,'../src/header/index.tsx'),
            "other": path.resolve(__dirname,'../src/other/index.ts')
        };

        const getEntryFile = (chunkInfo: PreRenderedChunk): string => {
            if(chunkInfo.name === 'index') {
                return '[name].[format].js';
            }
            return '[name]/[name].[format].js';
        }

        const esmConfig: RollupOptions = {
            input,
            output: {
                // file: packageJson.module,
                dir: 'lib',
                entryFileNames: getEntryFile,
                format: "es",
                globals: {
                    ...globals
                }
            },
            plugins: plugins()
        };

        const cjsConfig: RollupOptions = {
            input,
            output: {
                // file: packageJson.main,
                dir: 'lib',
                entryFileNames: getEntryFile,
                format: "cjs",
                globals: {
                    ...globals
                }
            },
            plugins: plugins()
        };

        const umdConfig: RollupOptions = {
            input: input.index,
            output: {
                file: 'lib/index.umd.js',
                format: "umd",
                name: "MyLibrary",
                globals: {
                    ...globals
                }
            },
            plugins: plugins()
        }

        export default [
            esmConfig,
            cjsConfig,
            umdConfig,
        ];
    ```
    > build_path.ts增加多入口配置
    ```typescript
        import path from 'path';

        export default {
            config: path.resolve(__dirname,'../config/rollup.config.ts'),
            entries: {
                "index": path.resolve(__dirname,'../src/index.ts'),
                "header": path.resolve(__dirname,'../src/header/index.tsx'),
                "other": path.resolve(__dirname,'../src/other/index.ts')
            }
        }
    ```
    > watch.ts 打包配合优化
    ```typescript
        import { InputOption, OutputOptions, RollupOptions, RollupWatcher , RollupWatcherEvent, watch } from "rollup";
        import rollupConfig from '../config/rollup.config';
        import { log , ELogType } from './tools';

        // 是否初次启动
        let firstStartUp: boolean = true;
        // 打包次数记录
        let buildTimes: number = 1;
        // 打包esModule的配置
        let esmOptions: RollupOptions;

        async function watchBuild() {
            rollupConfig.forEach(element => {
                if((element.output as OutputOptions).format === 'es') {
                    esmOptions = element;
                }
            });
            console.log('esModuleConfig: ',esmOptions);
            const watcher: RollupWatcher = watch({
                ...esmOptions,
                watch: {
                    buildDelay: 500
                }
            });
    
            watcher.on('event',async ( event: RollupWatcherEvent ) => {
                if(event.code === 'START') {
                    if(firstStartUp)
                        log('-----------------[打包模式为esModule]---------------------');
                    log(firstStartUp ? '打包开始...' : '监听到变更，重新开始打包...');
                }

                if(event.code === 'ERROR') {
                    log(event.error,ELogType.ERROR);
                    if(watcher.close) {
                        watcher.close();
                    }
                }

                if(event.code === 'BUNDLE_START') {
                    const inputKeys: string[] = Object.keys(event.input as InputOption);
                    const inputPaths: string[] = [];
                    if(inputKeys.length > 0) {
                        inputKeys.forEach((key:string) => {
                            inputPaths.push(( event.input as InputOption)[key]);
                        })
                    }
                    log(`打包入口：${inputPaths.length > 0 ? inputPaths.join(',') : '空'} -> 即将输出文件：${event.output}...`);
                }

                if(event.code === 'BUNDLE_END') {
                    log(`成功创建文件：${event.output}`);
                    log(`打包耗时：${event.duration/1000}秒`);
                    log(`打包完成,已完成${buildTimes}次...`);
                    log('-------------------------------------------------------');
                    log('监听文件修改中...');
                    log('-------------------------------------------------------');
                    buildTimes++;
                    firstStartUp = false;
                }
            })
        }

        watchBuild();
    ```
    > package.json 入口处理，新增打包脚本依赖包
    ```json
        {
            "name": "my-library",
            "main": "lib/index.cjs.js",
            "module": "lib/index.es.js",
            "types": "lib/index.d.ts",
            "version": "1.0.0",
            "repository": "",
            "path": ".",
            "author": "",
            "license": "MIT",
            "sideEffects": false,
            "files": [
                "lib/**/*"
            ],
            "peerDependencies": {
                "axios-esm": "^1.0.0",
                "react": "^17.0.2"
            },
            "devDependencies": {
                "@rollup/plugin-commonjs": "^21.0.0",
                "@rollup/plugin-image": "^2.1.1",
                "@rollup/plugin-json": "^4.1.0",
                "autoprefixer": "^10.3.7",
                "axios-esm": "^1.0.0",
                "chalk": "^4.1.2",
                "cross-env": "^7.0.3",
                "del": "^6.0.0",
                "node-sass": "^6.0.1",
                "postcss": "^8.3.9",
                "react": "^17.0.2",
                "rollup": "^2.58.0",
                "rollup-plugin-delete": "^2.0.0",
                "rollup-plugin-node-builtins": "^2.1.2",
                "rollup-plugin-node-resolve": "^5.2.0",
                "rollup-plugin-peer-deps-external": "^2.2.4",
                "rollup-plugin-postcss": "^4.0.1",
                "rollup-plugin-terser": "^7.0.2",
                "rollup-plugin-typescript-paths": "^1.3.0",
                "rollup-plugin-typescript2": "^0.30.0",
                "ts-node": "^10.3.0",
                "ts-transform-paths": "^2.0.3",
                "tslib": "^2.3.1",
                "ttypescript": "^1.5.12",
                "typescript": "^4.4.3",
                "typescript-transform-paths": "^3.3.1"
            },
            "scripts": {
                "build": "cross-env NODE_ENV=production ts-node scripts/build.ts",
                "dev": "cross-env NODE_ENV=development ts-node scripts/watch.ts"
            }
        }
    ```
    > tsconfig.json 提供打包脚本引入json的能力
    ```json
        {
            "compilerOptions": {
                "target": "es6",
                "sourceMap": false,
                "baseUrl": "./",
                "outDir": "lib",
                "declaration": true,
                "declarationDir": "lib",
                "noImplicitAny": false,
                "moduleResolution": "Node",
                "paths": {
                    "@/*": ["src/*"],
                    "@header/*": ["src/header/*"],
                    "@other/*": ["src/other/*"],
                    "@assets/*": ["src/assets/*"],
                    "@tools/*": ["src/tools/*"]
                },
                "lib": ["es5","es6","es7","dom"],
                "typeRoots": [
                    "src/@types",
                    "node_modules/@types"
                ],
                "allowJs": true,
                "checkJs": true,
                "allowSyntheticDefaultImports": true,
                "strict": true,
                "experimentalDecorators": true,
                "jsx": "react",
                "esModuleInterop": true,
                "plugins": [
                    {
                        "transform": "typescript-transform-paths",
                        "afterDeclarations": false
                    }
                ]
            },
            "ts-node": {
                // ts-node执行时使用的配置
                "compilerOptions": {
                    "module": "commonjs",
                    "resolveJsonModule": true
                }
            },
            "include": ["src/**/*"],
            "exclude": [
                "node_modules",
                "**/tests/*",
                "lib"
            ]
        }
    ```
5. 增加打包脚本build.ts
   
   ```typescript
        import { OutputOptions, rollup, RollupBuild } from "rollup";
        import rollupConfig from "../config/rollup.config";
        import del from 'del';
        import { log } from './tools';

        async function build() {

            log('----------开始打包----------')
            log('开始删除lib文件夹')
            del(["lib/*"]);
            log('清理lib文件夹完成')

            for await (const rollupOption of rollupConfig) {
                const moduleType: string | undefined = (rollupOption.output as OutputOptions).format;
                log(`----------开始打包${moduleType}模块----------`);
                const bundle: RollupBuild = await rollup(rollupOption);
                await bundle.write(rollupOption.output as OutputOptions);
                log(`----------${moduleType}模块输出完成----------`);
            }

            log('打包完成');
        }

        build();
```

#### 2022-04-11更新 css按需加载尝试
1. css 分包有两种方式
	1. css打入js中,js模块做到tree-shaking,那么css自然就做到了
	2. css extract模式,这种模式是把css抽出来独立打成css文件,**这种方式需要依赖babel-plugin-import插件来帮你做转义**
>这里转义的意思是,babel-plugin-import插件会把类似`import { Button } from 'antd'`的代码转译成`import {default as Button} from 'antd/component/button'`,如果配置上还有`style: css|true`,那么还会加上类似`import 'antd/component/button/style/index.css'`的代码来引入样式,**这其实也是babel-plugin-import实现按需加载的原理**,当然这边牵扯到你的lib在打包时一定要分包
2. 缺陷(**暂无解决方案**)
> 无论是哪种方式,都有一个问题.正常开发一般都会使用scss|less等预编译css语言来扩展css的能力,类似的语言一般我们也会声明一个全局文件global.scss,这个文件,我们在每个模块可能都会引入`@import 'global.scss'`,这就导致打包之后global.scss文件中定义的全局样式会出现冗余,**但是,暂时还没有发现有什么办法可以将全局scss独立打包,这就导致冗余代码无法避免,目前唯一的解决方案就是global.scss中不写具体样式,只做定义,这样是不会出现冗余样式的**

3. css独立文件打包的配置修改如下
>这个做法就是将之前的一个css插件修改为一个文件一个插件打包流程,保证scss打包份文件

```typescript
// 依赖glob包
const bundleCss = (): Plugin[] => {
    let config: Plugin[] = [];
    const files = glob.sync(path.resolve(__dirname, '../src/**/*.scss'))
    files.forEach(file => {
        const filePath = path.parse(file);
        const basePath = filePath.dir.replace("src","lib");
        const fileName = `${filePath.name}.css`;
        const output = path.resolve(basePath,`./style/${fileName}`);
        config.push(
            css({
                include: file,
                minimize: true,
                extract: output,
                plugins: [
                    autoprefixer(),
                    cssnano(),
                ],
            })
        )
    })
    return config;
}
```
4. 为打包css增加浏览器前缀,只需要在根目录增加文件.browserslistrc,内容如下:
```
defaults
not ie < 8
last 2 versions
> 1%
iOS 7
last 3 iOS versions
```
5. 压缩css,需要cssnano插件