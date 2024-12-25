#知识笔记/Obsidian/语法 
> [!Info]
> 除Markdown语法外，Obsidian还支持自己的一些扩展语法，以下只记录Obsidian扩展语法

#### 标签（Tag）
> [!Tip]
> 标签目前看来是Obsidian中组织笔记的核心点
##### 可用的标签写法
- 使用驼峰式大小写：`#TwoWords`
- 使用下划线：`#two_words`
- 使用连字符：`#two-words`
##### 子标签
> [!TIp]
> 子标签就是通过`/`分割线在主标签之后增加，比如：
> `#TwoWords/subTag/subSubTag`

#### 引用
##### 直接引用
1. `![[笔记文件名#标题名]]`
2. `![[笔记文件名^文本块]]`
##### 增加链接到Obsidian中的其它笔记
1. `[[Obsidian扩展语法#标签（Tag）]]`
2. `[[12.23-12.27工作小记#工作内容]]`

#### Callout
##### 语法
>[!Tip]

>[!Note] 标题

>[!Info] Info标题
>这是标题内容1
>这是标题内容2

##### 列举一下常用的Callout内置标记
>[!Note]

>[!Todo]

>[!Info]

>[!Tip]

>[!Success]

>[!Question]

>[!Warning]

>[!Failure]

>[!Danger]

>[!Bug]

>[!Example]

>[!Quote]

>[!Abstract]

#### 注释
##### 语法
>[!Example]
>单行：`%%这是单行注释%%`
>多行：
>`%%`
>`这是`
>`多行`
>`注释`
>`%%`

#### 图片
##### 语法
>[!Example]
>引用网络图片并裁剪：`![图片名称|宽度数值](链接地址)`
>引用本地图片（仓库中）并裁剪：`![[图片名|宽度数值]]`

#### 键盘文本
##### 语法
>[!Info]
>主要是用于描述**快捷键**，**按键**有关的时候
>比如：
>- `<kbd>**键盘文本**</kbd>`
>- `**<kbd>ctrl + x</kbd>**`

#### 插入音频视频
##### 语法
>[!Info]
>`<iframe src='https://xxxx' scrolling='no' border='0' frameborder='no' framespacing='0' allowfullscreen='true'></iframe>`
