---
tags:
  - 知识笔记/经验
---
>[!Tip] ts-node执行需要tsconfig module:是commonjs，可以在tsconfig中直接提供针对ts-node的重写
> ```JSON
> {
>   "ts-node": {
>     // these options are overrides used only by ts-node
>     // same as our --compilerOptions flag and our TS_NODE_COMPILER_OPTIONS environment variable
>     "compilerOptions": {
>       "module": "commonjs"
>     }
>   }
> }
> ```

