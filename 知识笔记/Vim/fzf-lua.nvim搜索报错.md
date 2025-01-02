---
tags:
  - 知识笔记/Vim
---
>[!Error] 报错信息：
>**EMFILE: too many open files**

>[!Note] 原因
>fzf-lua搜索大项目时，会消耗大量文件句柄，造成这个报错

>[!Success] 处理方法
>1. fzf-lua配置中限制搜索深度
>```
>require('fzf-lua').setup {
>  files = {
>    rg_opts = "--files --hidden -g '!{.git,node_modules}/*'",
>    fd_opts = "--max-depth=5"  -- 限制查找深度
>  }
>}
>```
>2. 放大mac系统的文件句柄限制
>```
>ulimit -n 8192  # 适当增加限制，8192 是一个比较安全的值
>```

