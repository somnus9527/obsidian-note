---
tags:
  - 知识笔记/CSS
---
>[!Tip] :not(:placeholder-shown)一直被匹配
>假设存在一个input节点`<input type='text' class='testNode' />`
>那么一旦通过js`inputNode.removeAttribute('placeholder')`将placeholder属性移除掉
>就会出现如下css选择器，一直处于匹配状态
>```css
>.testNode:not(:placeholder-shown) {
  color: red;
}
>```

