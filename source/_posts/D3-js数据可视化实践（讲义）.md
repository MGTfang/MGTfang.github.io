title: D3.js数据可视化实践（讲义）
author: Peng Fang
tags:
  - D3.js
categories:
  - 数据可视化
date: 2018-06-29 19:01:00
---
# D3数据可视化实践
> 方鹏 2018/06/29

- 目标  
了解D3常用API和重要知识点，能阅读和编写简单的D3.js可视化程序。

- 内容
 1. D3简介
 2. 基础知识点（SVG + D3）
 3. 数据处理（涉及加载数据、处理数据）
 4. 数据绑定
 5. 交互
 6. 布局

### 什么是D3
D3 全称是Data-Driven Document，直译为<strong style="color:red;">数据驱动文档</strong>。

> 数据由用户或开发者提供  
> 文档指的是基于web的文档，即Web浏览器可以渲染的任何元素，例如HTML、SVG   
> 它们由D3来驱动。从某种意义上说，它将文档和数据联系起来。

只要记住：<strong style="color:red;">D3是一个JavaScript函数库，是用来做数据可视化的</strong>。


其开源地址：
[https://github.com/d3/d3](https://github.com/d3/d3)

用D3.js的创始人[Mike Bostock](https://bl.ocks.org/mbostock)的话说：D3提供了极度灵活，Web标准化的能力，例如CSS3, HTML5, SVG。
试想用原生的HTML、SVG、Canvas来实现数据变成图形是困难和繁琐的。D3封装了这些能力，使开发者专注布局和逻辑。

### D3简史

1. 2011年2月，Mike Bostock发布了v1.0.0版本。
2. 2012年12月，v3.0.0版本发布，网上3.x的资料比较多。
3. 2016年6月，v4.0.0版本发布，开始支持Canvas，采用模块化设计模式。

### D3的优势
Echarts等封装层次很高，能够简单地制作图表，但是给予开发者控制和设计的空间较少。
D3在这一点上取得了平衡。
- 相对比较底层   
数据和元素捆绑。DOM里含有数据，数据更新时重绘。同时支持SVG和Canvas。
- 更像数学库   
强大的图形计算能力，D3的“布局”封装了提供力直方图、饼图、树图、力导向图等。D3的“比例尺”提供线性、指数、对数、序数等多种关于对应关系的计算。还提供“地图”功能。
- 即封装操作，也给予自由   
计算和绘图相互独立。计算是算出节点的位置、线段端点、弧线角度等；绘图是将计算所得的节点、线段绘制到网页上。

应用截图如下：  
![Alt text](http://baapps.com/theme/images/cargo-analytics-apps-thumb.jpg "cargo analytics")
![Alt text](http://baapps.com/theme/images/sct-app-thumb.jpg "sct app")
![Alt text](http://baapps.com/theme/images/kpi-tracker-thumb.jpg "kpi tracker")
![Alt text](http://baapps.com/theme/images/eval-app-thumb.jpg "eval app")

### 适用范围
数据可视化七个步骤：获取、解析、过滤、挖掘、表现、改善、交互   
<strong style="color:red;">表现、改善、交互</strong>属于D3的适用范围

### 它不能干什么
  D3不能生成预定义的或者封装好的可视化效果，也就是不提供预先配置的图表类型。    
  D3不支持老的浏览器。   
  D3代码在客户端执行，因此不能隐藏源数据。如果担心数据泄露，最好不要可视化，可视化的目的是交流数据。


### SVG
SVG（可缩放矢量图形），除了IE8之前的版本外，绝大部分浏览器支持SVG，可以直接嵌入HTML显示。   
位图与矢量图的区别：   
位图缩放后失真，矢量图缩放不失真；位图色彩表现力较丰富，矢量图图形色彩较简单；矢量图占用空间较小，位图较大；矢量图容易转化为位图，反之不容易。

#### SVG图形元素
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
[示例](http://blockbuilder.org/MGTfang/7bf8068b2ff08cf98c83a5a7e73a9106)

#### 文字
在SVG中可以使用`<text>`标签绘制文字   
dx: 相对当前位置在x方向上平移的距离，dy同理。   
[text-anchor属性](https://segmentfault.com/a/1190000009293590)
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
[Hello World 示例](http://blockbuilder.org/MGTfang/ac3f1dee9143e3874398df498483ed31)

### 使用数据
D3数据处理流程：
![数据处理流程](http://pic.caigoubao.cc/604149/blog/visualization/%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86%E6%B5%81%E7%A8%8B.png)

#### 加载数据
D3提供几个方法来导入和处理数据。一样的是，d3.csv()和d3.json()生成一个JSON对象数组，而d3.xml()会创建一个XML文档。
``` javascript 
d3.csv("cities.csv", (error, data) => { console.log(error, data) });
d3.json("tweets.json", data => console.log(data));
```
error变量是可选项

#### 数据格式化
- 数据映射  
数字数据仅仅同图形化元素在屏幕上的尺寸和位置相关。Scales（比例尺）有一个定义域domain和一个值域range。我们使用d3.scale()函数来归一化数据。例如我们通过线性比例尺，将 500,000到13,000,000的城市人口相同的线性变化映射到0到500px宽的画布上。
![线性比例尺示意](http://pic.caigoubao.cc/604149/blog/visualization/%E7%BA%BF%E6%80%A7%E6%AF%94%E4%BE%8B%E5%B0%BA%E7%A4%BA%E6%84%8F.png)
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
![定量数据分类](http://pic.caigoubao.cc/604149/blog/visualization/%E5%AE%9A%E9%87%8F%E6%95%B0%E6%8D%AE%E5%88%86%E7%B1%BB.png)
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

#### 测量数据
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

[简单数据可视化示例](http://blockbuilder.org/MGTfang/3d2d4fccbbc8023bd00f94451abf50ac)

### 数据可视化
接下来将更深入解释选择器是如何与数据绑定一起创建元素的，以及创建后如何改变这些元素。

#### data工作过程
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
![进入退出更新示意图](http://pic.caigoubao.cc/604149/blog/visualization/%E8%BF%9B%E5%85%A5%E9%80%80%E5%87%BA%E6%9B%B4%E6%96%B0%E7%A4%BA%E6%84%8F%E5%9B%BE.png)

### 文档驱动设计及交互
D3v4 有几个方法帮助DOM上下移动：selection.raise 和 selection.lower. 使用这些方法来移动选择的元素到DOM兄弟节点的末尾或者移动到开头位置。
``` javascript 
d3.select("g.overallG").raise()
d3.select("g.overallG").lower()
```
禁用元素的鼠标事件：
``` javascript
teamG.select("text").style("pointer-events","none");
```

#### 使用颜色
d3.rbg()的使用如下：
``` javascript
teamColor = d3.rgb("red");
teamColor = d3.rgb("#ff0000");
teamColor = d3.rgb("rgb(255,0,0)");
teamColor = d3.rgb(255,0,0);
```
这些颜色对象有两个有用的方法：.darker() 和 .brighter()，返回比初始值更亮或更暗的值。
``` javascript
d3.selectAll("g.overallG").select("circle")
  .style("fill", p => p.region === d.region ?
  teamColor.darker(.75) : teamColor.brighter(.5)) // rgb.brighter([k]),每个颜色通道值将乘以0.7 ^ -k
```
`d3.hsl, d3.lab, d3.cubehelix, d3.hcl`可以呈现不同的颜色空间
D3支持颜色插值
``` javascript
var ybRamp = d3.scaleLinear()
  .interpolate(d3.interpolateHsl)
  .domain([0, maxValue]).range(["yellow", "blue"]);
```
离散颜色比例尺：`d3.schemeCategory10, d3.schemeCategory20, d3.schemeCategory20b, and d3.schemeCategory20c`
序数比例尺，映射离散值为特殊的颜色。一个有用的特征是它的unknown方法，当传入一个不存在的值的时候，返回设定值。   
[Color Scales](https://bl.ocks.org/pstuffa/3393ff2711a53975040077b7453781a9)
``` javascript
var tenColorScale = d3.scaleOrdinal()
  .domain(["UEFA", "CONMEBOL"])
  .range(d3.schemeCategory10)
  .unknown("#c4b9ac")
```

#### 监听器
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
[拖拽简单示例](http://blockbuilder.org/MGTfang/8608aa25a5f44feefa2f1ee5c1c399dc)
[缩放简单示例](http://blockbuilder.org/MGTfang/8203f1fe95d2afd455729c4cf8384549)

### 布局
D3 3.x提供了12种布局：饼状图（Pie）、力导向图（Force）、弦图（Chord）、树图（Tree）、集群图（Cluster）、捆图（Bundle）、打包图（Pack）、直方图（Histogram）、分区图（Partition）、堆栈图（Stack）、矩阵树图（Treemap）、层级图（Hierarchy）

#### 力导向图
力导向图（Force-Directed Graph），是绘图的一种算法。d3-force, 力布局这个模块基于Verlet integration(韦尔莱积分法)实现了物理粒子之间的作用力的仿真。模拟的作用力有电荷之间的吸引排斥力，重力，链接吸引力。

[力导向图示例](http://blockbuilder.org/MGTfang/4080ee9dc10804fd7a362565e54c78da)

### 项目案例分析


### 参考资料
- [D3 API 中文文档](https://github.com/d3/d3/wiki/API--%E4%B8%AD%E6%96%87%E6%89%8B%E5%86%8C)
- [D3 js in action](https://www.manning.com/books/d3-js-in-action)
- [精通D3.js（第二版）](http://www.broadview.com.cn/book/4786)
- [D3.js 4.x Data Visualization(Third Edition)](https://www.amazon.com/D3-js-4-x-Data-Visualization-Third/dp/178712035X)
- [图说D3数据可视化利器从入门到进阶](http://www.broadview.com.cn/book/638)