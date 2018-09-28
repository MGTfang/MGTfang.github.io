title: webcola介绍
author: Peng Fang
tags:
  - webcola
categories:
  - 可视化
date: 2018-09-12 20:18:00
---
[cola.js](https://ialab.it.monash.edu/webcola/)是一个开源的JavaScript库，使用基于约束的优化技术组织HTML5文档和图表。它可以和D3.js、SVG.js以及Cytoscape.js类似的很好配合。核心布局是基于libcola的C++库重写的。   
它也适配于d3.js，允许你使用cola作为D3力布局的一个核心替代。不同于D3力布局，通过一个简单的退火策略使其布局收敛到局部最优，因此与D3力布局相比：   
- Cola实现更高质量的布局
- 在交互式应用程序中更稳定（没有"抖动"）
- 它允许用户指定约束，例如对齐和分组
- 它能自动生成约束来
    - 避免节点重叠
    - 为有向图提供流布局
- 对于非常大的图，它可能不太好扩展  
然而，图上节点少于100个的情况下工作得很好，见[Unix家族树](https://ialab.it.monash.edu/webcola/examples/unix.html)   

注意，虽然使用D3力布局，为了获得合理的节点分离，你可能要必须处理"charge"这样的参数。cola在最终布局中指定的链接距离方面做得更好。这是因为cola直接试着最小化理想链接距离和图中实际链接距离中的方差。换句话说，仅仅根据节点大小设置合理的链接距离。

为了了解一些可选的参数，我们可以像这样开始布局：
``` javascript
d3cola
    .nodes(graph.nodes)
    .links(graph.links)
    .constraints(graph.constraints)
    .symmetricDiffLinkLengths(5)
    .avoidOverlaps(true)
    .start(10,15,20);
```
我们像D3力布局一样指定节点和链接。constraints是一个新参数，它是一个包含约束的数组，像这样：   
``` javascript
{"axis":"y", "left":0, "right":1, "gap":25}
```
这就是说，`graph.nodes[0]`的中心必须与`graph.nodes[1]`的中心距离25个像素以上。更精确地讲，约束要求满足这样的不等式：
``` javascript
graph.nodes[0].y + gap <= graph.nodes[1].y
```
设置`avoidOverlaps(true)`，在布局进行时动态生成约束，以防止节点的边框彼此重叠。   
`symmetricDiffLinkLengths(5)` ，使用5作为基本长度，计算每个链路上的理想长度，为的是在高度节点周围创造额外的空间。或者，你也可以将自己的函数f传递给linkDistance，返回每个链接的特定长度。   
`start()`方法现在包含多达三个整数参数。在上面的例子中，start最初将应用10次没有约束的布局迭代，15次仅具有结构（用户指定的）约束的迭代，以及20次包含反重叠约束的具有所有约束的布局迭代。指定这样一个计划是有用的，使得图形在其被严格约束前展开。

#### 在cytoscape.js中使用cola.js
Cytoscape.js通过扩展已经完全支持Cola.js.Cytoscape.js拥有一个完全的图理论模型，高度可定制的样式和布局。Cola.js在Cytoscape.js中运行只需要一行代码：   
``` javascript
cy.layout({ 
    name: 'cola' /* and maybe some other options */ 
});
```

##### 翻译自
https://ialab.it.monash.edu/webcola 