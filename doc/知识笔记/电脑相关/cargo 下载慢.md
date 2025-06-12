---
tags:
  - 知识笔记/电脑相关
---
>[!Success] 具体操作：
>在`$HOME/.cargo/config.toml`下添加如下内容（没有就新增）：
>```toml
># 新建名为ustc的镜像源
[source.crates-io]
replace-with = 'ustc'
># 将默认的crates-io替换为ustc
>[source.ustc]
>registry = "git://mirrors.ustc.edu.cn/crates.io-index"
>```

