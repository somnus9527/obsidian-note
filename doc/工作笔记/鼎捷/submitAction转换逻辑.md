---
tags:
  - 工作笔记/鼎捷/备忘
---
#### submitAction
- 核心转换逻辑
```typescript
const buttons = submitActions.map((item, index) => {
	// 取出如下字段，处理后放入action
    const {
      id,
      type,
      actionType,
      actionParams,
      actionId,
      serviceName,
      url,
      paras,
      lang,
      returnText,
      executeAfterCheckCompleted,
      extendParas,
      dispatchBPM,
      needProxyToken,
      dispatch,
      terminateProcess,
      defaultAction,
      trackCode,
      submitType,
      hidden,
      condition,
      confirm,
      combineActions,
      attachActions,
      sizeType,
      ...rest
    } = item;
    // 将returnText的多语言重新组装
    const { returnText: returnTextLang, ...restLang } = lang ?? {};
    // 上面取出的字段处理后组装到action中
    const action = {
      type,
      actionType,
      actionParams,
      actionId,
      serviceName,
      url,
      paras,
      returnText,
      lang: {
        returnText: returnTextLang,
      },
      executeAfterCheckCompleted,
      extendParas,
      dispatchBPM,
      needProxyToken,
      dispatch,
      terminateProcess,
      trackCode,
      submitType,
      attachActions,
      combineActions,
    };
    // 单独处理condition和hidden字段
    const isHiddenStr = typeof hidden === 'string';
    const isConditionStr = typeof condition === 'string';
    // rebuildHidden是将原来的hidden里的condition字段key，改为scipt,值不动
    const actualHidden = isHiddenStr ? { script: hidden } : rebuildHidden(hidden);
    const actualCondition = isConditionStr ? { script: condition } : { ...(condition ?? {}) };
    // 组装实际的动态按钮数据
    const info = {
      id: id ?? uuidv4(),
      // 原对象中多余的字段统一放在这边，避免丢失
      ...rest, 
      // 最后一个SubmitAction设为默认
      styleMode: defaultAction || index === submitActions.length - 1 ? 'primary' : 'default',
      // 大小以原对象的sizeType为准，没有就塞large
      size: sizeType || 'large',
      type: getButtonTypeByAction(action),
      // 重新组装后的hidden和condition放在这
      hiddenConfig: actualHidden,
      condition: actualCondition,
      confirm: { ...(confirm ?? {}) },
      // 其余的多语言
      lang: {
        ...restLang,
      },
      // 后端转换这边不用处理，将上面组装的action放进来就行
      action: handleInnerActions(cleanObject(action)),
    };
    return cleanObject(info);
  });
  // 外面再套一层button_group
  return [
    {
	  // 前端标记，后端不需要
      evolutionFromSubmitActions: true, // 标记：从submitActions转化而来
      id: uuidv4(),
      type: 'BUTTON_GROUP',
      headerName: '按钮组',
      verticalAlignment: 'center',
      block: true,
      justifyContent: 'center',
      gap: '12px',
      position: 'absolute',
      padding: ['12px', '0', '12px', '0'],
      bottom: '0px',
      lang: {
        headerName: {
          zh_CN: '按钮组',
          zh_TW: '按鈕組',
          en_US: 'Button Group',
        },
      },
      moreButtonConfig: {
        enable: false,
      },
      group: buttons ?? [],
    },
  ];
```
- 放置逻辑
```typescript
// 这里有平台的特殊逻辑，对于平台来说LAYOUT组件比较特殊，所以 有 LAYOUT 组件存在的 情况下
// 把 buttonGroup 放到 LAYOUT 组件的content的 group 的 末尾，而不是 放到 layout 末尾
// 判断第一层即可
    const layoutComponent = pageUIElementContent?.layout?.find((item) => item.type === 'LAYOUT');
    if (layoutComponent && layoutComponent?.content?.group) {
      layoutComponent.content.group = [...layoutComponent.content.group, ...buttonGroup];
    } else {
      pageUIElementContent.layout = [...pageUIElementContent.layout, ...buttonGroup];
    }

```
