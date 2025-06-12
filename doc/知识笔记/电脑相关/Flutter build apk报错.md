---
tags:
  - 知识笔记/电脑相关
---
#### 报错内容

flutter AAPT: error: resource android:attr/lStar not found.

#### 错误原因

依赖包依赖了androidx.core:core-ktx:最新版本

#### 解决办法

build.gradle中添加配置 跟android配置平级

>[!Tip] 配置如下：
> ```gradle
> configurations.all {
>     resolutionStrategy {
>         force 'androidx.core:core:1.6.0'
>         force 'androidx.core:core-ktx:1.6.0'
>     }
> }
> ```

