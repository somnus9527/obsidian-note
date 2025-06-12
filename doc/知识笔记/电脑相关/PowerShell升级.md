---
tags:
  - 知识笔记/电脑相关
---
1. 首先查看当前PowerShell版本：`$PSVersionTable.PSVersion`
2. 通过命令`winget search Microsoft.PowerShell`查询当前可用的最新版本
3. 通过命令 `winget install --id Microsoft.PowerShell --source winget`下载安装
4. 最后如果发现重打开终端发现还是老版本，可以在设置中修改启动 -> 默认配置文件，修改成新安装的powershell版本

#### 问题记录

- 我从5升级到7之后，oh-my-posh的配置不生效了。。。找了很久发现，7和5的配置不使用同一份默认配置下

- 5的配置文件地址`C:\Users\Administrator\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1`
- 7的配置文件地址`C:\Users\Administrator\Documents\PowerShell\Microsoft.PowerShell_profile.ps1`

#### 其它配置

1. 安装oh-my-posh `winget install JanDeDobbeleer.OhMyPosh -s winget`
2. 安装Nerd-Fonts
3. 安装Icon `Install-Module -Name Terminal-Icons -Repository PSGallery`
4. 最后在powershell的外观设置中设置透明度，效果比较好的时60%，同时打开下面的启用亚克力材料

#### ps配置文件最终如下

```powershell
oh-my-posh init pwsh --config 'C:\Users\Administrator\AppData\Local\Programs\oh-my-posh\themes\1_shell.omp.json' | Invoke-Expression
Import-Module -Name Terminal-Icons
```
