---
tags:
  - 知识笔记/经验
---
#### 前置环境准备

>[!Info] VSCode安装通义
>1. 配置Lingma Domail Url 
>![[Pasted image 20250529085733.png]]
>2. 登陆即可

#### 尝试使用场景

##### MasterGo MCP试用

###### 前置准备
1. 获取MasterGo Token
   > `mg_4ae5822e619d41d39600a6645d0d5629`
2. 安装MasterGo MCP
3. 配置MCP,将上面获取到的Token配置到参数上
   ![[Pasted image 20250603090710.png]]
4. 测试MCP可用行
	- 找到MasterGo官网提供的测试页面
	- 选中最简单的图层![[Pasted image 20250603093132.png]]
	- 复制URL,交给MCP生成代码文件
	- 使用官网提供的话术![[Pasted image 20250603093247.png]]
	- MCP前置最重要的两步
		1. getMeta![[Pasted image 20250603093331.png]]
		2. getDsl![[Pasted image 20250603093400.png]]
		3. 上面两步完成，MCP就有了生成代码的基础数据，后续大模型就开始去生成代码了，整个过程会生成一系列代码文件![[Pasted image 20250603093500.png]]
		4. 这里其实只是官方的DEMO,也只是测试整个MCP工具是否可用，不关注到底生成了什么
5. 至此，MCP工具可用
###### 尝试使用
1. 添加上下文，主要是项目组件和angular配置![[Pasted image 20250603100804.png]]
2. 生成时间较长，生成完成之后，发现html文件中代码为空。。。而且存在angular18的代码
3. 鉴于可能是MCP存在异常（中途出现调用异常），重新跑一次，这一次添加更多的上下文，以及更详细的要求![[Pasted image 20250603102922.png]]
4. 首先MCP还是首先跑getMeta和geDsl![[Pasted image 20250603103111.png]]
5. 检查一下DSL结果看起来没啥问题
   >[!Note] DSL结果记录
> `{"dsl":{"styles":{"paint_34:6367":{"value":["#FFFFFF"],"token":"常用色/纯白"},"effect_155:219235":{"value":["box-shadow: 0px 8px 10px -5px rgba(0, 0, 0, 0.06);","box-shadow: 0px 16px 20px 2px rgba(0, 0, 0, 0.04);","box-shadow: 0px 6px 24px 5px rgba(0, 0, 0, 0.05);"],"token":"阴影/上层投影"},"paint_34:5715":{"value":["#1D1C33"],"token":"文字图标色/主要文字"},"font_32:8227":{"value":{"family":"PingFang SC","size":18,"style":"{\"fontStyle\":\"中粗体\",\"opsz\":\"auto\"}","decoration":"none","case":"none","lineHeight":"28","letterSpacing":"auto"},"token":"标题/一级"},"paint_155:57161":{"value":["#DFDFE5"],"token":"中性色/G8-常规分割线"},"font_155:57415":{"value":{"family":"PingFang SC","size":14,"style":"{\"fontStyle\":\"常规体\",\"opsz\":\"auto\"}","decoration":"none","case":"none","lineHeight":"20","letterSpacing":"auto"},"token":"正文/常规"},"paint_32:8216":{"value":["#FFFFFF"],"token":"文字图标色/白色"}},"nodes":[{"type":"INSTANCE","id":"285:059745","layoutStyle":{"width":720,"height":512,"relativeX":0,"relativeY":0},"fill":"paint_34:6367","effect":"effect_155:219235","componentInfo":{"properties":{"尺寸":"中-720","内容居中":"是"}},"children":[{"type":"INSTANCE","id":"285:059745/498:089245","layoutStyle":{"width":720,"height":48},"flexShrink":1,"componentInfo":{"properties":{"类型":"默认","对齐方式":"居中对齐"}},"children":[{"type":"FRAME","id":"285:059745/498:089245/459:288632","name":"占位区","layoutStyle":{"width":96.0009994506836,"height":28,"relativeX":24,"relativeY":10},"flexShrink":1,"children":[],"flexContainerInfo":{"flexDirection":"row","alignItems":"center","mainSizing":"fixed","crossSizing":"fixed","gap":"12px"}},{"type":"FRAME","id":"285:059745/498:089245/459:288634","name":"标题配置区","layoutStyle":{"width":231.99798583984375,"height":28,"relativeX":126.0009994506836,"relativeY":10},"children":[{"type":"TEXT","id":"285:059745/498:089245/459:288635","name":"模态标题","layoutStyle":{"width":72,"height":28,"relativeX":79.99899291992188},"text":[{"text":"模态标题","font":"font_32:8227"}],"textColor":[{"start":0,"end":4,"color":"paint_34:5715"}],"textAlign":"center","textMode":"single-line"}],"flexContainerInfo":{"flexDirection":"column","alignItems":"center","mainSizing":"fixed","crossSizing":"fixed","padding":"0px 131.99948120117188px"}},{"type":"FRAME","id":"285:059745/498:089245/459:288638","name":"操作区","layoutStyle":{"width":96.0009994506836,"height":28,"relativeX":363.9989929199219,"relativeY":10},"flexShrink":1,"children":[{"type":"INSTANCE","id":"285:059745/498:089245/459:288644","name":"关闭","layoutStyle":{"width":20,"height":20.000152587890625,"relativeX":76.0009994506836,"relativeY":3.9999237060546875},"componentInfo":{"description":"关闭；叉；取消"},"children":[{"type":"PATH","id":"285:059745/498:089245/459:288644/250:014190","name":"联集","layoutStyle":{"width":10,"height":10,"relativeX":3,"relativeY":3},"path":[{"fill":"paint_34:5715","data":"M0 0.5C0 0.632608 0.0526784 0.759785 0.146447 0.853553L4.29289 5L0.146886 9.14601L0.146886 9.14601C0.0526783 9.24022 0 9.36739 0 9.5C0 9.77614 0.223858 10 0.5 10C0.632608 10 0.759785 9.94732 0.853553 9.85355L5 5.70711L9.14645 9.85355C9.24022 9.94732 9.36739 10 9.5 10C9.77614 10 10 9.77614 10 9.5C10 9.36739 9.94732 9.24022 9.85355 9.14645L5.70711 5L9.85355 0.853553C9.94732 0.759785 10 0.632608 10 0.5C10 0.223858 9.77614 0 9.5 0C9.36739 0 9.24022 0.0526784 9.14645 0.146447L9.14645 0.146447L5 4.29289L0.853553 0.146447C0.759785 0.0526784 0.632608 0 0.5 0C0.223858 0 0 0.223858 0 0.5Z"}]}],"componentId":"250:014189"}],"flexContainerInfo":{"flexDirection":"row","justifyContent":"flex-end","alignItems":"center","mainSizing":"fixed","crossSizing":"fixed","gap":"12px"}}],"componentId":"459:288631"},{"type":"FRAME","id":"285:059745/498:089252","name":"弹窗/内容","layoutStyle":{"width":720,"height":208,"relativeY":48},"flexShrink":1,"strokeColor":"paint_155:57161","strokeType":"dashed","strokeAlign":"outside","strokeWidth":"0px 0px 1px","children":[{"type":"FRAME","id":"285:059745/498:089253","name":"内容区","layoutStyle":{"width":672,"height":164,"relativeX":24,"relativeY":20},"flexGrow":1,"children":[],"flexContainerInfo":{"flexDirection":"row","flexWrap":"wrap","mainSizing":"fixed","crossSizing":"auto"}}],"flexContainerInfo":{"flexDirection":"row","justifyContent":"center","alignItems":"center","mainSizing":"fixed","crossSizing":"auto","padding":"20px 24px 24px"}},{"type":"INSTANCE","id":"285:059745/498:089237","layoutStyle":{"width":720,"height":64,"relativeY":256},"flexShrink":1,"componentInfo":{"properties":{"对齐方式":"居中"}},"children":[{"type":"INSTANCE","id":"285:059745/498:089237/250:060074","layoutStyle":{"width":60,"height":32,"relativeX":172,"relativeY":16},"componentInfo":{"properties":{"样式":"实线按钮","纯图标按钮":"否","尺寸":"大-32","层级":"默认","交互":"默认"}},"children":[{"type":"TEXT","id":"285:059745/498:089237/250:060074/178:057750","name":"按钮文字","layoutStyle":{"width":28,"height":20,"relativeX":16,"relativeY":6},"text":[{"text":"取消","font":"font_155:57415"}],"textColor":[{"start":0,"end":2,"color":"paint_34:5715"}],"textAlign":"center","textMode":"single-line"}],"componentId":"178:057748"},{"type":"INSTANCE","id":"285:059745/498:089237/250:060082","layoutStyle":{"width":60,"height":32,"relativeX":248,"relativeY":16},"componentInfo":{"properties":{"样式":"填充按钮","纯图标按钮":"否","尺寸":"大-32","层级":"默认","交互":"默认"}},"children":[{"type":"TEXT","id":"285:059745/498:089237/250:060082/178:057769","name":"按钮文字","layoutStyle":{"width":28,"height":20,"relativeX":16,"relativeY":6},"text":[{"text":"确定","font":"font_155:57415"}],"textColor":[{"start":0,"end":2,"color":"paint_32:8216"}],"textAlign":"center","textMode":"single-line"}],"componentId":"178:057766"}],"componentId":"250:060073"}],"componentId":"498:089236"}],"components":[]},"componentDocumentLinks":[],"rules":["token filed must be generated as a variable (colors, shadows, fonts, etc.) and the token field must be displayed in the comment","\n componentDocumentLinks is a list of frontend component documentation links used in the DSL layer, designed to help you understand how to use the components.\n When it exists and is not empty, you need to use mcp__getComponentLink in a for loop to get the URL content of all components in the list, understand how to use the components, and generate code using the components.\n For example: \n ``js \n const componentDocumentLinks = [\n 'https://example.com/ant/button.mdx',\n 'https://example.com/ant/button.mdx'\n ]\n for (const url of componentDocumentLinks) {\n const componentLink = await mcp__getComponentLink(url);\n console.log(componentLink);\n }\n ``\n "]}`

6. 结果上来说，MasterGo时UX设计稿，所以结果更关注HTML和LESS文件的生成效果，但是HTML内容几乎为空，less文件相对完整，但是存在大量冗余代码

>[!Note] 生成的HTML文件
>![[Pasted image 20250603110006.png]]
>结论：生成的HTML文件，内容约等于为空，最关注的中间部分内容，实际生成的文件只有一行`这是数据视图模态框的内容区域，可以添加各种表单组件和数据显示`的说明

>[!Note] 生成的LESS文件
>![[Pasted image 20250603105936.png]]
>less文件，对比了一下MasterGo设计稿，其实相对已经比较完整（HTML中没有的节点的样式也出现了），这就说明，实际MCP是知道存在相应的节点的，只是HTML中没有生成相应的节点，但是样式文件中却出现了

###### 其它问题记录
- 中途稳定出现调用次数过多的报错
- 存在大量循环调用（多为检查），造成上面的问题![[Pasted image 20250603105606.png]]

###### MasterGo MCP尝试结论

>[!Warning] MasterGo MCP尝试结论
>部分可用。对于从设计稿生成最重要的HTML文件中没有生成相应的节点，调整都没办法调整。样式生成近乎完整，如果只是用来生成样式规则，那么可以生成之后拷贝到自己的规则中，还具备可用性。

##### 简单功能使用

###### 注释补充

>[!Note] 找到子应用中的一个cookie工具方法文件测试注释能力
>![[Pasted image 20250603113638.png]]
>可以发现添加注释的能力可用性很高,前提是纯函数

###### 代码优化
>[!Note] 简单沟通之后优化代码
>![[Pasted image 20250603114049.png]]
>可以看到代码优化以及根据规范的调整都可以达到理想的效果

###### 复杂场景的代码片段生成
> 上面两个场景都偏简单，下面测试一下复杂场景

>[!Warn] 场景介绍 (场景来自于实际需求，目前已经解)
>当前有主应用A,子应用B
>B通过Module Federation暴露组件Comp
>Comp内部需要创建子线程（WebWorker）
>A会通过MF的方式加载组件Comp
>A应用开发模式起在4201端口
>B应用开发模式起在3000端口
>经过多次调整依然没有得到想到的结果：
> ![[Pasted image 20250603130551.png]]
> ![[Pasted image 20250603130603.png]]
> 最终实际给出的代码还是最开始的方式，只是细节方面有些差异：
> ![[Pasted image 20250603130634.png]]
> 实际，之前我这边处理的方式如下：
> ![[Pasted image 20250603130723.png]]

##### 规则尝试

- 下载规则并配置
- [官方DEMO规则下载地址](https://atomgit.com/lingma/lingma-project-rule-template)
- 下载完成之后在项目根目录进行配置![[Pasted image 20250603131729.png]]
- 针对规则内容进行修改 [[通义灵码AI规则记录]]
- 规则初步添加&修改完成，继续尝试代码片段的开发&修改&优化能力

##### 借助AI进行登录页改造需求开发

1. 先在主应用中配置上AI规则![[通义灵码angular项目AI规则记录]]
2. 添加本次需求开发需要的相关上下文![[Pasted image 20250605103910.png]]
3. 通过上述描述开始让AI去开发
4. ![[Pasted image 20250605104030.png]]
5. AI初步执行完成之后，查看页面发现它其实并没有按照UX设计稿来设计页面![[Pasted image 20250605104111.png]]
6. 这是设计稿![[Pasted image 20250605104132.png]]
7. 多次调整之后无果，页面结构还是不符合UX
8. 考虑到可能是Master Go MCP可能存在问题，所以将登录页截图交给AI让AI再次进行优化，结果页面直接爆炸了![[Pasted image 20250605105539.png]]
9. 整体来看，想要AI直接生成可用的页面结构效果不好

##### TODO
- [x] MasterGo MCP生成页面测试
- [x] 常规能力测试：注释生成，代码修改，代码片段生成
- [x] 添加代码规则，并根据项目实际情况优化
- [x] 借助AI能力，进行一个小需求的开发