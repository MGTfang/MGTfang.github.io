title: D3.js数据可视化实践（精简）
author: Peng Fang
tags:
  - D3.js
categories:
  - 数据可视化
date: 2018-07-01 19:03:00
---
## D3数据可视化实践

> 方鹏 2018/07/05

### 前言
在我们的APP上线后，经过数据分析，实时获取反映使用情况的活跃用户数；活跃数可分为总的日活或月活，分APP的日活或月活，分APP分省的日活或月活等等。想把这些数据表达清楚，可能会涉及实时的动态的变化数据；数据的按维度分类展示；如何按数字大小或者属性进行区分，着重展示我们感兴趣的数据。以上涉及到的就是我们常说的数据可视化。数据可视化的目的就是对数据进行可视化处理，以使得明确有效地传递信息。  
利用图表来对数据进行可视化是我们熟知的方式。产品经理对可视化需求进行分析，选择合适的图表来表达和传递信息。而我们作为前端开发人员，也需要理解需求及设计，选用能满足需求的图表组件或可视化方案来实现产品设计。   
选择技术方案的时候，我们或许会发现，因为图表内容复杂，没有现成的图表组件能够利用；或者现有的图表组件不能满足我们的部分需求；或者因为大数据量的场景，造成图表渲染较慢。解决这些需求中的痛点，我们就得思考用何种技术方案，怎样解决遇到的问题。思考为什么选择这个方案，它能帮助我们解决什么问题，或者说它有什么优势。

### 怎么选择
现有的可视化库很多，像Echarts，Highcharts，D3这些我们多少都在项目中经常使用或者听说过大名。Echarts等可视化库封装层次很高，能够简单地制作图表，但是给予开发者控制和设计的空间较少。D3在这一点上取得了平衡。D3提供了极度灵活的Web标准化能力，例如CSS3, HTML5, SVG。
试想用原生的HTML、SVG、Canvas来实现数据绘图是困难和繁琐的。D3封装了这些能力，使开发者专注布局和逻辑。那么方案怎么选择呢？我们应该熟悉各可视化库的优势，在满足需求的同时也要考虑开发成本。

### 关于D3.js

#### 什么是D3.js
D3 全称是Data-Driven Document，直译为<strong style="color:red;">数据驱动文档</strong>。

> 数据由用户或开发者提供  
> 文档指的是基于web的文档，即Web浏览器可以渲染的任何元素，例如HTML、SVG、Canvas   
> 由D3来驱动数据和文档。从某种意义上说，它将文档和数据联系起来。

<strong style="color:red;">D3是一个JavaScript函数库，是用来做数据可视化的</strong>。

用D3.js的创始人<a href="https://bl.ocks.org/mbostock">Mike Bostock</a>

其开源地址：
<a href="https://github.com/d3/d3">https://github.com/d3/d3</a>

#### 历史及版本比较
1. 2011年2月，Mike Bostock发布了v1.0.0版本。
2. 2012年12月，v3.0.0版本发布，网上3.x的资料比较多。
3. 2016年6月，v4.0.0版本发布，开始支持Canvas，采用模块化设计模式。

| v3.x | v4.x |
| :-------------: | :-------------: |
| 嵌套结构 | 模块化|
| 稳定，资料多 | 有优化，资料相对较少 |
| 只支持SVG渲染 | 支持Canvas渲染 |

> 注意：<strong style="color:red;">两个版本的代码不兼容</strong>

#### D3的优势

- 相对比较底层   
数据和元素捆绑。DOM里含有数据，数据更新时重绘。同时支持SVG和Canvas。
- 更像数学库   
强大的图形计算能力，D3的“布局”封装了提供力直方图、饼图、树图、力导向图等。D3的“比例尺”提供线性、指数、对数、序数等多种关于对应关系的计算。还提供“地图”功能。
- 即封装操作，也给予自由   
计算和绘图相互独立。计算是算出节点的位置、线段端点、弧线角度等；绘图是将计算所得的节点、线段绘制到网页上。

#### D3部分模块
| 模块 | 描述 | 依赖 | 
| -------- | --------------- | ----------- | 
| d3-axis | 坐标轴| d3-scale, d3-selection, d3-transition | 
| d3-color | 颜色集操作和呈现 | None. | 
| d3-format | 数字格式化 | None. |
| d3-interpolate | 插值函数 | d3-color |
| d3-path | 路径生成 | None. |
| d3-polygon | 二维多边形几何操作 | None. |
| d3-request | XMLHttpRequest封装 | d3-dsv, d3-dispatch |
| d3-scale | XMLHttpRequest封装 | d3-dsv, d3-dispatch |
| d3-request | 从抽象到具体数据的映射 | d3-array, d3-collection, d3-color, d3-format, d3-interpolate, d3-time, d3-time-format |
| d3-selection | 通过选择和加入数据进行DOM转换 | d3-dsv, d3-dispatch |
| d3-transition | D3.js选择集的动画过渡 | d3-ease, d3-timer, d3-interpolate, d3-selection |

### D3基础
#### SVG基础
SVG（可缩放矢量图形），除了IE8之前的版本外，绝大部分浏览器支持SVG，可以直接嵌入HTML显示。   
位图与矢量图的区别：   
位图缩放后失真，矢量图缩放不失真；位图色彩表现力较丰富，矢量图图形色彩较简单；矢量图占用空间较小，位图较大；矢量图容易转化为位图，反之不容易。
##### SVG图形元素
SVG预定义七种形状元素：   
矩形`<rect>`、圆形`<circle>`、椭圆形`<ellipse>`、线段`<line>`、折线`<polyline>`、多边形`<polygon>`、路径`<path>`   
路径指令：   
- M=moveto
- L=lineto
- H=horizontal lineto
- V=vertical lineto
- C=curveto 画三次贝塞尔曲线经<strong>两个</strong>指定控制点到达终点
- S=smooth curveto 与前一条三次贝塞尔曲线相连，第一个控制点与前一条曲线的第二个控制点对称
- Q=quadratic curveto 画二次贝塞尔曲线经<strong>一个</strong>指定控制点到达终点
- T=smooth quadratic Bezier curvto 与前一条二次贝塞尔曲线相连，第一个控制点与前一条曲线的第二个控制点对称，只需输入终点，即可绘制一条二次贝塞尔曲线   
<a href="http://blockbuilder.org/MGTfang/7bf8068b2ff08cf98c83a5a7e73a9106">SVG 示例</a>

##### 文字
在SVG中可以使用`<text>`标签绘制文字   
dx: 相对当前位置在x方向上平移的距离，dy同理。   
<a href="https://segmentfault.com/a/1190000009293590">text-anchor属性</a>
``` html
<text class="label-2" text-anchor="end" dy=".35em" rank="B">
  <tspan x="0" dy="0.35em">(不知情定制B004)内容战</tspan>
  <tspan x="0" dy="1.0499999999999998em">略合作伙伴分类不合规</tspan>
</text>
```

### D3数据选择和属性设定
我们要对DOM中的元素执行一个行为，例如移动位置，改变颜色,更新数据中的值，首先要选中它。
d3.select()是选中单个元素；d3.selectAll()是选中多个元素。
选择集是一个或多个页面元素的组合，能够与数据集相关联。
利用D3提供的方法设置元素属性和样式值：
``` javascript
d3.selectAll("circle.a").style("fill", "red").attr("cx", 100)
d3.select("circle").attr("class", "active");
d3.select("circle").classed("active", true); // 添加或者移除命名的样式

```
有部分属性不能用attr()设定和获取，最典型的就是文本框的value属性，这样情况可以用property()来设定，例如：

``` javascript 
d3.select("#someCheckbox").property("checked", true);
```
方法链，也叫链式调用。和JQuery写法类似。
``` javascript 
d3.selectAll("div").data(someData).enter().append("div").html("Wow").append("span").html("Even More Wow").style("font-weight", "900");
```
可以给.style(), .attr(), .property(), .html()设置匿名函数调用或者其他提供数据绑定的选择器的函数。
``` javascript 
var someColors = ["blue", "red", "chartreuse", "orange"];
someColors = someColors.filter(function(d) {return d.length < 5});
d3.select("body").selectAll("div")
  .data(someColors)
  .enter()
  .append("div")
  .style("background", function(d) {return d})
  .attr("cx", function(d,i) {return i})
  .html(function(d) {return d})
```
<a href="http://blockbuilder.org/MGTfang/ac3f1dee9143e3874398df498483ed31"></a>


### D3数据处理
- 数据映射  
数字数据仅仅同图形化元素在屏幕上的尺寸和位置相关。Scales（比例尺）有一个定义域domain和一个值域range。我们使用d3.scale()函数来归一化数据。例如我们通过线性比例尺，将 500,000到13,000,000的城市人口相同的线性变化映射到0到500px宽的画布上。
![线性比例尺示意](https://mmbiz.qpic.cn/mmbiz_png/GWicdfXT7twRA5heUgAGibFCfBW6IB2XsToicK4bbZXicsHTkMajMTP75ibRcXZlFSdKxXQczGXJ64CcSG51074fl1A/0?wx_fmt=png)
``` javascript 
var newRamp = d3.scaleLinear().domain([500000,13000000]).range([0, 500]);
newRamp(1000000); // 返回20，可以将一千万人口的国家放在20px处
newRamp(9000000); // 返回340
newRamp.invert(313); // 求逆，返回8325000
var newRamp = d3.scaleLinear().domain([500000,13000000]).range(["blue", "red"]);
newRamp(1000000); // 返回"#0a00f5"，可以将一百万人口的城市用深紫色表示
newRamp(9000000); // 返回"#ad0052"
newRamp.invert("#ad0052"); // 因为invert函数只接受数字，因此返回NaN
```
我们也能使用d3.scaleLog(), d3.scalePow(), d3.scaleOrdinal()等其他对数据集来说更加适合的比例尺来映射数据。
- 数据分类  
将定量数据分类，是将值按范围分或者组装到一起。一种分类方法是将数组均分几份。
![定量数据分类](https://mmbiz.qpic.cn/mmbiz_png/GWicdfXT7twRA5heUgAGibFCfBW6IB2XsTvC0bxjDkN7K9SW4BRglwcTu3DsK1yA6h5P7Pa9j0DGVuSaXTc8gn9Q/0?wx_fmt=png)
``` javascript 
var sampleArray = [423,124,66,424,58,10,900,44,1];
var qScale = d3.scaleQuantile().domain(sampleArray).range([0,1,2]);
qScale(423); // 返回2
qScale(20); // 返回0
qScale(10000); // 返回2
```
嵌套允许数组中的元素被组织为分层树型结构；类似SQL语句里面的GROUP BY方法。
下面的例子，将示例数据首先按year分组再按variety分组，如下：
``` javascript
var yields = [{yield: 27.00, variety: "Manchuria", year: 1931, site: "University Farm"},
              {yield: 48.87, variety: "Manchuria", year: 1931, site: "Waseca"},
              {yield: 27.43, variety: "Manchuria", year: 1931, site: "Morris"}, 
               ...]
var nest = d3.nest()
    .key(function(d) { return d.year; })
    .key(function(d) { return d.variety; })
    .entries(yields);
```
返回的嵌套数组中,以键值对的形式对数据进行分组:
``` javascript 
[{key: 1931, values: [
    {key: "Manchuria", values: [
        {yield: 27.00, variety: "Manchuria", year: 1931, site: "University Farm"},
        {yield: 48.87, variety: "Manchuria", year: 1931, site: "Waseca"},
        {yield: 27.43, variety: "Manchuria", year: 1931, site: "Morris"}, ...]},
    {key: "Glabron", values: [
        {yield: 43.07, variety: "Glabron", year: 1931, site: "University Farm"},
        {yield: 55.20, variety: "Glabron", year: 1931, site: "Waseca"}, ...]}, ...]},
 {key: 1932, values: ...}]
```
在数据格式化以后，需要测量它，以确保创建的图形尺寸合适，位置是基于数据集的参数。那你将会一直用到d3.extent，d3.min，d3.max，d3.mean。

- 数据测量  
在加载你的数据以后，首要的事情之一是应该对数据进行测量和排序。很重要的是知道特殊属性值的分布，以及最大最小值和属性名称。D3提供一个数组的函数集合能帮助理解数据。
加入想从cities.csv获取城市人口的最小，最大，平均值：
``` javascript
d3.csv("cities.csv", data => {
  d3.min(data, el => +el.population); 
  d3.max(data, el => +el.population);
  d3.mean(data, el => +el.population);
});
```
d3.extent方便地将d3.min()和d3.max()在一个数组中返回：
``` Javascript
d3.extent(data, el => +el.population); // 返回[500000, 1300000]
```
现在，我们已经加载、格式化、测量了我们的数据，那我们就可以创建数据可视化了。

<a href="http://blockbuilder.org/MGTfang/3d2d4fccbbc8023bd00f94451abf50ac">简单数据可视化示例</a>

### D3数据绑定过程
一个选择集是由DOM中一个或多个元素构成。能使用选择器创建和删除元素，更改样式和内容。
``` javascript
d3.csv("cities.csv", (error,data) => {
  if (error) {
      console.error(error)
  } else {
      dataViz(data)
  }
});
function dataViz(incomingData) {
  d3.select("body").selectAll("div.cities")
      .data(incomingData) // 绑定数据到选择集
      .enter() // 定义当选择集中的数据多于DOM元素时如何响应
      .append("div") // 在当前选择集中创建一个元素
      .attr("class", "cities") // 设置新创建元素class
      .html(d => d.label); // 设置创建div的内容
}
```
经常传入id没有匹配到任何元素，那么称其为空选择集。
当数据值的数量大于选择器中元素的数量，.enter()函数触发，允许你为每一个没有相应DOM元素的值定义一个执行行为。
这里需要将选中的DOM元素同一个数组联系起来。数据集里的每一个城市同选择集中的一个DOM元素相联系，关联数据是存在元素的data属性中。
``` javascript
document.getElementsByClassName("cities")[0].__data__ // 返回一个指向对象的指针
```
- 如果数组长度大于元素数量，则部分还不存在的元素“即将进入可视化（enter）”
- 如果数组长度小于元素数量，则多余的元素“即将退出可视化（exit）”
- 如果数组长度等于元素数量，则绑定数据的元素“即将被更新（update）”

大多数情况，.enter()函数触发，使用.append()来添加元素；.exit()函数触发，使用.remove()来移除元素。
![进入退出更新示意图](https://mmbiz.qpic.cn/mmbiz_png/GWicdfXT7twRA5heUgAGibFCfBW6IB2XsTyUz4Pa5o4PUA4ia9iaU3VX4nRnbPx1SuEbndtj97bWiaov6yvIU1ImwTA/0?wx_fmt=png)

### 交互

#### 颜色插值
D3支持颜色插值
``` javascript
var ybRamp = d3.scaleLinear()
  .interpolate(d3.interpolateHsl)
  .domain([0, maxValue]).range(["yellow", "blue"]);
```
离散颜色比例尺：`d3.schemeCategory10, d3.schemeCategory20, d3.schemeCategory20b, and d3.schemeCategory20c`   
序数比例尺，映射离散值为特殊的颜色。一个有用的特征是它的unknown方法，当传入一个不存在的值的时候，返回设定值。   
<a href="https://bl.ocks.org/pstuffa/3393ff2711a53975040077b7453781a9">Color Scales</a>
``` javascript
var tenColorScale = d3.scaleOrdinal()
  .domain(["UEFA", "CONMEBOL"])
  .range(d3.schemeCategory10)
  .unknown("#c4b9ac")
```

#### 事件监听
D3选择器可以通过on来为事件添加监听器：`selection.on(type[, listener[, capture]])`
``` javascript
selection.on("click",  function () {
    console.log(d3.mouse(this)); // 输出相对坐标
})
```
在当前选择的每个元素，为指定的类型type，添加或删除事件监听器listener 。type是一个字符串事件类型的名称，如“click”、“mouseover”、“keydown”、“touchstart”。基本上支持任何DOM事件，如鼠标、键盘、触屏事件。为了在侦听器内访问当前事件，使用全局函数d3.event。

如果所选择的元素相同类型的一个事件监听已经注册了，新的侦听加入之前的现有侦听被除去。为注册相同事件类型的多个监听器，该类型可以跟一个可选的命名空间，如“click.first”和“click.second”。 要删除一个监听器，传递null给listener如：selection.on(click", null)。

#### 行为
下面给出拖拽和缩放的例子：  
<a href="http://blockbuilder.org/MGTfang/8608aa25a5f44feefa2f1ee5c1c399dc">拖拽简单示例</a>   
<a href="http://blockbuilder.org/MGTfang/8203f1fe95d2afd455729c4cf8384549">缩放简单示例</a>

#### 布局
D3 3.x提供了12种布局：饼状图（Pie）、力导向图（Force）、弦图（Chord）、树图（Tree）、集群图（Cluster）、捆图（Bundle）、打包图（Pack）、直方图（Histogram）、分区图（Partition）、堆栈图（Stack）、矩阵树图（Treemap）、层级图（Hierarchy）。   
力导向图（Force-Directed Graph），是一种常用的绘图算法。d3-force, 力布局这个模块基于Verlet integration(韦尔莱积分法)实现了物理粒子之间的作用力的仿真。模拟的作用力有电荷之间的吸引排斥力，重力，链接吸引力。

> 力导向图的绘制
> 1. 生成数据
> 2. 设置力导向图
> 3. 添加绘制方法
> 4. 添加交互


<a href="http://blockbuilder.org/MGTfang/4080ee9dc10804fd7a362565e54c78da">力导向图示例</a>

### 总结
Ben Fry提出的数据可视化七个步骤：获取、解析、过滤、挖掘、表现、改善、交互。D3数据可视化也会经历这几个过程。首先D3读取源数据，进行解析得到需要处理的对象数组，然后将得到的数据进行过滤返回我们感兴趣的数据，然后对过滤得到的数据进行格式化，分类或分组，方便利用数据来驱动文档。
通常我们根据数据数值大小或时间远近等熟悉转换为图形上元素的大小或者位置。如果需要生成网络图谱等可视化图表，我们可以利用D3提供的布局来绘制。以力导向图为例，首先构建节点和连线的对象数据，然后将其设定给力导向图，力导向图为给节点连线生成初始的位置信息，然后根据节点和连线对象数据来自定义（绘制）节点和内容。同时我们利用tick机制绑定tick回调函数，在回调函数中定义机制对节点和连线的坐标进行更新，并更新图表。
D3.js是一种很强大的可视化利器，在GitHub数据可视化分类中关注度最高的。在AI+物联网时代，数据可视化或者说人机交互将会越来越重要。掌握常见的数据可视化技术，理解可视化中的图形图像算法对于想在可视化方向深耕的小伙伴来说是很有必要的。

### 参考资料
- [D3 API 中文文档](https://github.com/d3/d3/wiki/API--%E4%B8%AD%E6%96%87%E6%89%8B%E5%86%8C)
- [D3 js in action](https://www.manning.com/books/d3-js-in-action)
- [精通D3.js（第二版）](http://www.broadview.com.cn/book/4786)
- [D3.js 4.x Data Visualization(Third Edition)](https://www.amazon.com/D3-js-4-x-Data-Visualization-Third/dp/178712035X)
- [图说D3数据可视化利器从入门到进阶](http://www.broadview.com.cn/book/638)
