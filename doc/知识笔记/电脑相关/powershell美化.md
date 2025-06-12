---
tags:
  - 知识笔记/电脑相关
---
1. 根据官网安装**oh_my_posh**, 代码`https://ohmyposh.dev/docs/installation/windows`
2. 终端输入 `New-Item -Path $PROFILE -Type File -Force`创建配置文件，可以通过`code $PROFILE`打开
3. 修改主题 `oh-my-posh init pwsh --config 'https://raw.githubusercontent.com/JanDeDobbeleer/oh-my-posh/main/themes/jandedobbeleer.omp.json' | Invoke-Expression`
>[!Tip]
>注意这块的地址，可以改成本地的主题地址，一般来说主题都在这边**C:\Users\Administrator\AppData\Local\Programs\oh-my-posh\themes**,也可以自己去下载放在这边，下载地址在这边[oh my posh主题地址](https://github.com/JanDeDobbeleer/oh-my-posh/tree/main/themes)

>[!Note] 备注
>- `Get-PoshThemes`,获取本地可以读取到的所有主题
>- 如果第二步走完，打开终端出现报错，_此系统上禁止运行脚本_,一般都是权限不够造成的，以管理员权限运行终端，跑一下命令`Set-ExecutionPolicy -ExecutionPolicy RemoteSigned`即可
>- 另附oh-my-posh地址[oh-my-posh官网地址](https://ohmyposh.dev/)

