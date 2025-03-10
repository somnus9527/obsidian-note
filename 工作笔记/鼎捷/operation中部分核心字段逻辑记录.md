---
tags:
  - 工作笔记/鼎捷/备忘
---
>[!Note] target
>隶属表格的fullPath

>[!Note] applyToField
>1. mode === 'all', applyToField为空 (可以手动选择字段)
>2. mode === 'row'
>	- pageCode === 'browse-page'则applyToField=BUTTON_GROUP
>	- pageCode === 'edit-page'则applyToField=UIBOT_BUTTON_GROUP

>[!Note] 开窗
>- target：隶属表格的fullPath
>- applyToField：所在列的字段的schema
>另：如何判断这个operation是不是开窗
>`['openwindow', 'open-task-window'].includes(operate)`
