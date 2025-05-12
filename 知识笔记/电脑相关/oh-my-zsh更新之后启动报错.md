---
tags:
  - 知识笔记/电脑相关
---
>[!Error] compinit:527: no such file or directory: /opt/homebrew/share/zsh/site-functions/_brew_services
>原因是：**oh-my-zsh**更新之后brew_services被废弃了
>**oh-my-zsh**官网issue中给出的解决方案：
>`brew cleanup && rm -f $ZSH_COMPDUMP && omz reload`