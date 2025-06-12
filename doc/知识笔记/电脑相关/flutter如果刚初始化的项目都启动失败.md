---
tags:
  - 知识笔记/电脑相关
---
flutter如果刚初始化的项目都启动失败，一般都是国内网的问题，可以设置两个全局变量

- `PUB_HOSTED_URL` 设置为 `https://mirrors.tuna.tsinghua.edu.cn/dart-pub`
- `FLUTTER_STORAGE_BASE_URL` 设置为 `https://mirrors.tuna.tsinghua.edu.cn/flutter`