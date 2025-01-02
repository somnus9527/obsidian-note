---
tags:
  - 知识笔记
---
#### 需求
>希望可以在主应用中实现一个agInnerLable指令，用来在相应节点上增加浮动标题效果

#### 前期分析

##### 思路
添加浮动标题效果，首先尝试如下方式：

1. 在需要添加样式的节点内添加样式节点或样式伪类
2. 添加一个包裹节点并增加样式
3. 前两种如果都不行，添加sibling节点，通过定位处理样式

##### 思路分析：

1. 首先因为要处理的节点是input节点，而input节点是自闭合节点，所以在自闭合节点内部增加节点或伪类必定不会生效。第一个思路不行
2. 添加包裹节点需要将现有input挪到一个新建的dom节点中，这可能会存在如下问题：所以待测试？？？

1. angular中，手动操作dom会不会造成angular原生数据流和事件触发丢失？
2. 如果原input节点存在`.testStyle > input {}`的类似样式，增加了一层包裹节点后，这种样式会失效，那么就需要手动处理，会很麻烦，而且难以保证ng-zorro的样式不丢失

3. 添加兄弟节点并手动定位+处理样式也存在如下问题：所以待测试？？？

1. 如何准确将新添加的节点定位到input节点上，如果input节点存在移动的效果，如何实时定位？
2. 定位如果能解决，则还需要将原input的placeholder样式添加到新增加的节点上

#### 思路尝试：

1. 添加包裹节点：

1. 经测试，手动添加的节点不会对原组件数据流和事件流造成影响，但是是否所有的都没有影响不敢保证
2. ant-input确实存在`.testStyle > input {}`这种类似样式，那么部分样式就丢失了，而且新增加的包裹节点也会受到样式影响，由于希望这个指令支持在所有input上生效，按么就需要处理所有这种类似的样式，工作量过于恐怖了，加上a可能存在的影响，只能暂时先放弃思路2
3. 如何获取placeholder样式成为问题，问题在于无法获取，或者说老版本支持，现在版本已经被取消，因为获取placeholder样式还是通过`getComputedStyle`来获取，代码如：`const placeholderStyle = window.getComputedStyle(inputElement, '::placeholder');`,但是测试并不生效，原因在于最新的w3标准，已经不支持placeholder了，相关文档地址：[API说明](https://developer.mozilla.org/en-US/docs/Web/API/Window/getComputedStyle)

2. 添加兄弟节点：

1. 准确定位可以通过`getComputedStyle`方法获取input位置并给予新节点定位，但是由于新增节点需要脱离文档流，那么上层节点的position:relative会对新增节点的位置造成影响，那么处理的时候就需要递归往上找最近的position:relative节点并且根据找到的这个节点和目标节点的位置计算新增节点的位置，一旦出现dom节点位置变更处理更麻烦
2. 当然当input位置变更，同样可以通过`MutationObserver`API来监听位置变更来重置新节点位置
3. 同样存在placeholder样式这个问题

#### 尝试结论：

由于两个可行思路都存在一些问题，但是兄弟节点毕竟不如父子节点可控性高，且目前未发现存在原生数据流和事件存在问题，且样式不多可以稍微补一下，所以决定采用思路二

#### 正式尝试开发：

##### 实现思路：

1. 在指令中主要通过`**ElementRef**`和`**Renderer2**`来处理dom操作
2. `ngAfterViewInit`内获取dom节点类型，根据是否是input，构建需要处理nodes数据，是input则只需处理一个，不是则有可能需要处理多个
3. 遍历nodes数组，重新构建dom节点->新建一个节点，作为wrapper，并将原节点赛道wrapper中，通过新建一个inner-label节点专门处理浮动标题效果
4. 处理node时，同时根据不同情况设置不同class样式，保证css样式正常
5. 使用:not(:placeholder-shown)控制innerLabel动画样式

##### 问题记录&处理方式：

1. 除纯input外会出现focus时的border，这个就是由于原节点加深了一层，所以之前的一些样式应用不到，需要通过样式隐藏边框
2. textarea高度异常，原逻辑采用原节点高度，而textarea应该使用单行高度，所以针对textarea实时构建dom并设置原节点的样式，获取单行高度，并将innerLabel设置为这个单行高度
3. select 高度问题，原因是nz-select将内部input的placeholder进行了移除，并且自己塞了一个placeholder的自定义dom节点，造成原本的通用动画控制逻辑不生效了，所以针对select需要特殊处理

1. 构建innerLabel的DOM节点逻辑可重用，但是高度设置需要使用select高度，否则定位不准
2. 另外存在部分场景select会存在使用absolute定位的场景，那么左侧偏移量应该使用left值而不是paddingLeft值
3. 当select是多选时，高度并不一定等于选择框高度，因为还是select使用了absolute定位问题，这时候其实默认状态的定位不会有问题，但是上浮时会缺少高度差，所以需要针对浮动高度做补偿，默认4px

4. select缺少了:placeholder-shown支持，如何触发样式动画更新：

1. 发现select会在需要placeholder时，向dom树插入placeholder节点，不需要时删除placeholder节点
2. 针对a的情况，考虑使用MutationObserver进行dom树监听，判断placeholder的状态，如果被移除就添加上浮动画，如果被添加，就移除上浮动画
3. 另外由于新增了innerLabel，所以需要在使用innerLabel时，隐藏placeholder

1. 发现还不能设置为display:none;会造成js取不到高度等属性

5. tree-select同样存在高度问题，但是可以使用select逻辑
6. date-picker和range-picker同样存在高度问题，但是相对简单，使用最大高度即可
7. 如果存在手动设置了样式的情况，默认设置可能就会样式上出问题了，所以将原css进行抽取，必要的值改成css变量的形式，并支持通过入参手动设置，最大限度提供可兼容性