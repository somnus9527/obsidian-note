---
tags:
  - 知识笔记/经验
---
#### **vsCode Debug配置**
- 安装插件 C/C++ 和 Rust
- 增加配置 , 之后便可以debug了

>[!Example] 配置示例
> ```json
> {
>     // 使用 IntelliSense 了解相关属性。 
>     // 悬停以查看现有属性的描述。
>     // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
>     "version": "0.2.0",
>     "configurations": [
>         {
>             "name": "(Windows) Launch",
>             "type": "cppvsdbg",
>             "request": "launch",
>             "program": "${workspaceRoot}/target/debug/guessing_game.exe",
>             "args": [],
>             "stopAtEntry": false,
>             "cwd": "${workspaceRoot}",
>             "environment": []
>         }
>     ]
> }
> ```