---
tags:
  - 知识笔记/Godot
---
特殊通用能力可以通过autoload来实现，比如自定义事件，可以通过这个方式来实现。(注意autoload加载的会默认以文件名定义类名，并且定义中不允许出现class_name)

1. 先定义一个全局自定义事件

```
extends Node
# class_name GameEvent Godot会默认生成，且不允许手动定义
signal experience_vial_collected(expr: float)

func emit_experience_vial_collected(expr: float):
	experience_vial_collected.emit(expr)
```

2. 触发&接受事件

```
# 触发事件
GameEvent.experience_vial_collected.emit(10)
# 接收事件
GameEvent.experience_vial_collected.connect(on_experience_vial_collected)
```