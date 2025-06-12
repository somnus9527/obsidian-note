---
tags:
  - 知识笔记/前端框架/angular
---
>[!Tip] 正常编译命令：
>`ngcc --properties es2015 browser module main`
>如果更换了angular目标版本或者其它影响目标js的处理，可能回需要重新编辑ngcc，这时候就需要加上参数来避免ngcc编译缓存，完整命令：
>`ngcc --properties es2015 browser module main --first-only --create-ivy-entry-points`
>一般这种操作需要在install之后执行，需要只需要在package.json中增加一个postinstall即可：
>`"postinstall": "ngcc --properties es2015 browser module main --first-only --create-ivy-entry-points"`
