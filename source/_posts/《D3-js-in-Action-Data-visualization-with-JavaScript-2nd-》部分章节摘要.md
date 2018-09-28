title: 《D3.js in Action Data visualization with JavaScript(2nd)》部分章节摘要
author: Peng Fang
tags:
  - D3.js
categories:
  - 数据可视化
date: 2018-06-27 18:58:00
---
## 什么是D3
为满足Web可访问的，复杂的数据可视化需求而生。
假如你公司使用了BI工具，但是他们不能满足团队需求用来显示数据的类型模式。
那么你就需要量身定制地构建一个客户化的看板来精确展示具体领域客户的行为。
这个数据看板需要快速、可交互、能在组织内分享。那么可以使用D3来做这个事情。
D3.js的创始人Mike Bostock，用他的话说：提供极度灵活，Web标准化的能力，例如CSS3,HTML5,SVG. 最新迭代版本D3.v4
## D3怎样工作
怎样使用D3来处理和呈现数据，也是增加交互性，优化数据可视化。
理解D3选择器、数据绑定、D3怎样与DOM中的SVG和HTML交互。
### 数据可视化不仅仅是图表
你或许认为数据可视化仅限于饼图、线图、以及其他各类图表。远远不止如此。D3.js核心能力之一是为传统图表创建矢量图形，创建地理空间和网络可视化，以及丰富的动画和交互性。数据可视化这种广泛的方法中，地图或者网络图谱或表格是数据的一种表现形式，也是D3.js库提供的核心能力。
学习D3是因为它提供能力实现几乎每个主要的数据可视化技术，也提供能力创建你自己的数据可视化技术。
### D3数据选择和绑定
selection是数据和元素的组合。我们对group中的元素执行一个行为，例如移动或者改变颜色。同样也可以更新数据中的值。尽管我们以页面元素和数据分离的方式工作，D3真正强大来自于使用选择器来合并数据和web页面元素。
``` javascript
d3.selectAll("circle.a").style("fill", "red").attr("cx", 100)
```
使用d3.select()选择单个元素；使用d3.selectAll()选中多个元素
选择器是一个或多个页面元素的组合能够与数据集相关联，例如下面的代码是绑定数组[1,5,11,3]元素到样式名为market的div：
``` javascript
d3.selectAll("div.market").data([1,5,11,3])
```
D3遍历选择器中的元素，使用绑定数据执行相同的行为，导致不同的图形化效果。行为相同，效果不同。
决定一个元素的行为和呈现分别是：样式、属性attributes、特性properties。样式决定不透明度、颜色、尺寸、边框等。属性包含ID、类名、交互行为。特性涉及状态，例如单选框的选中。D3有三个函数来定义这些值：
``` javascript
d3.select("#someDiv").style("border", "5px darkgray dashed");
d3.select("#someDiv").attr("id", "newID");
d3.select("#someCheckbox").property("checked", true);
```
DOM决定了元素在屏幕上的绘制顺序，孩子元素在父元素的里面及之后绘制。尽管传统HTML中可以使用z-index来控制，但是在SVG2规范实现之前，设置z-index对SVG元素来说不管用。
### SVG
HTML5一个主要的特性就是集成支持SVG。SVG允许图像简单的数字呈现如缩放，这并应用到动画和交互中。D3提供一个抽象层绘制SVG。SVG绘制复杂形状的指令如路径，从画布的一个点开始向另一个点画线。如果想绘制曲线，将绘制曲线的坐标设置为path的d属性。
``` html
<!doctype html>
<html>
<script src="d3.v4.min.js">
</script>
<body>
<div id="infovizDiv">
<svg style="width:500px;height:500px;border:1px lightgray solid;">
<path d="M 10,60 40,30 50,50 60,30 70,80"
style="fill:black;stroke:gray;stroke-width:4px;" />
<polygon style="fill:gray;"
points="80,400 120,400 160,440 120,480 60,460" />
<g>
<line x1="200" y1="100" x2="450" y2="225"
style="stroke:black;stroke-width:2px;"/>
<circle cy="100" cx="200" r="30"/>
<rect x="410" y="200" width="100" height="50"
style="fill:pink;stroke:black;stroke-width:1px;" />
</g>
</svg>
</div>
</body>
</html>
```
svg提供一系列的通用形状集合：`<CIRCLE>`, `<RECT>`, `<LINE>`, `<POLYGON>`，每个形状都有元素决定它的尺寸和位置。`<RECT>`有x和y属性决定形状左上角的位置。`<CIRCLE>`的x和y属性决定圆心的位置，r属性决定圆半径。`<LINE>`x1和y1作为起点坐标，x2和y2决定中的坐标位置。
任何形状的颜色、外轮廓、不透明度能被改变来适应形状的样式，fill决定形状区域的颜色，stroke,stroke-width,stroke-dasharray决定其外轮廓。
能像下面这样改变矩形的样式：
``` css
fill:purple;stroke-width:5px;stroke:cornflowerblue;
```
`<TEXT>`,SVG提供写文本的能力。如果想要实现基本的格式化，可以在`<TEXT>`中嵌套`<TSPAN>`。
`<G>`或者组元素，区别于之前的SVG元素，它没有图形化呈现的能力，不存在一个边界空间。可以使用它来表示一个逻辑分组。当创建一个图形对象，有几个形状的图形和文本组成，就可以将它们放在`<G>`标签内。在画板内移动`<G>`元素，可以通过transform属性，它接受一个结构化的描述就是translate()，传一对坐标x和y，表示其向右和向下移动的像素数。transform属性还接受scale()，用来改变图形渲染的比例。
![路径的属性设置](http://pic.caigoubao.cc/604149/blog/visualization/Path%E5%B1%9E%E6%80%A7%E8%AE%BE%E7%BD%AE%E5%B7%AE%E5%BC%82.png)
### CSS
.css文件能被引入到HTML页当中，或者直接嵌入HTML当中，或者同js控制。
``` javascript 
d3.select(#someElement).style(opacity, .5);
d3.select("circle").attr("class", "tentative");
d3.select("circle").classed("active", true);
```
通过`.classed()`，不必重写存在的样式，而是从样式列表中添加或者移除命名的样式。
### JavaScript
方法链，也叫链式调用。这和我们平时聊天有点像。
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
## 数据标准
因为不同目的数据被格式化为不同的形式，但是它倾向存在于我们认识的类型：表格数据，嵌套数据，网络数据，地理数据，裸数据，对象。
## 第一个D3 APP
``` html
<html>
<head>
<script src="https://d3js.org/d3.v4.min.js"></script>
</head>
<body>
<div id="vizcontainer">
<svg style="width:500px;height:500px;border:1px lightgray solid;" />
<script>
d3.select("svg")
.append("circle")
.attr("r", 20)
.attr("cx",20)
.attr("cy",20)
.style("fill","red");
d3.select("svg")
.append("text")
.attr("id", "a")
.attr("x",20)
.attr("y",20)
.style("opacity", 0)
.text("HELLO WORLD");
d3.select("svg")
.append("circle")
.attr("r", 100)
.attr("cx",400)
.attr("cy",400)
.style("fill","lightblue");
d3.select("svg")
.append("text")
.attr("id", "b")
.attr("x",400)
.attr("y",400)
.style("opacity", 0)
.text("Uh, hi.");
d3.select("#a").transition().delay(1000).style("opacity", 1);
d3.select("#b").transition().delay(3000).style("opacity", .75);
d3.selectAll("circle").transition().duration(2000).attr("cy", 200);
</script>
</div>
</body>
</html>
```
## 使用数据
![数据处理流程](http://pic.caigoubao.cc/604149/blog/visualization/%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86%E6%B5%81%E7%A8%8B.png)
### 加载数据
不管什么样的数据源，都可能被格式化成XML，CSV，或者JSON这样的格式。D3提供几个方法来导入和处理数据。一样的是，d3.csv()和d3.json()生成一个JSON对象数组，而d3.xml()会创建一个XML文档。
``` javascript 
d3.csv("cities.csv", (error,data) => { console.log(error, data) });
d3.json("tweets.json", data => console.log(data));
```
error变量是可选项
### 数据格式化
在我们加载数据集以后，我们需要定义方法以致于数据的属性同颜色、尺寸和位置的设定相关。比如想展示导入的CSV文件格式的城市数据，假如使用圆来表示，根据人口数量来设置圆的大小，根据几何坐标设置位置。在做数据可视化的时候，我们需要理解数据形式，通常可以分为数量的，类别的，几何的，临时的，拓扑的或者未加工的。
### 进一步更改数据
数字数据仅仅同图形化元素在屏幕上的尺寸和位置相关。使用d3.scale()函数来归一化数据。Scales（比例尺）有一个定义域domain和一个值域range。例如，从cities.csv里面取出人口数字的最小值和最大值，通过线性比例尺我们能将它们的不同很容易地展示到500px的画布上。下图所示，将 500,000到13,000,000相同的线性变化映射到0到500.
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
我们也能使用d3.scaleLog(), d3.scalePow(),d3.scaleOrdinal(), 以及其他不太通用但对数据集来说更加适合的比例尺来映射数据。
将定量数据分类，将值按范围分或者组装到一起，是挺有用的。一种分类方法是将数组均分几份。
![定量数据分类](http://pic.caigoubao.cc/604149/blog/visualization/%E5%AE%9A%E9%87%8F%E6%95%B0%E6%8D%AE%E5%88%86%E7%B1%BB.png)
``` javascript 
var sampleArray = [423,124,66,424,58,10,900,44,1];
var qScale = d3.scaleQuantile().domain(sampleArray).range([0,1,2]);
qScale(423); // 返回2
qScale(20); // 返回0
qScale(10000); // 返回2
```
嵌套背后的观点是数据的共享属性能被用来排序成离散的类别和子类。
``` javascript
d3.json("tweets.json", data => {
var tweetData = data.tweets;
var nestedTweets = d3.nest()
.key(d => d.user)
.entries(tweetData);
});
```
d3.nest()合并推特账户到新的对象下的数组，这些新对象由唯一的用户属性值标记。
在数据格式化以后，需要测量它，以确保创建的图形尺寸合适，位置是基于数据集的参数。那你将会一直用到d3.extent，d3.min，d3.max，d3.mean.
### 测量数据
在加载你的数据以后，首页的事情之一是你应该将它测量和排序。很重要的是知道特殊属性值的分布，以及最大最小值和属性名称。D3提供一个数组的函数集合能帮助你理解数据。
``` javascript
var testArray = [88,10000,1,75,12,35];
d3.min(testArray, el => el); // return 1
d3.max(testArray, el => el); // 返回10000
d3.mean(testArray, el => el); // 返回平均值1701.83
```
加入想从cities.csv获取人口的最小，最大，平均值：
``` javascript
d3.csv("cities.csv", data => {
    d3.min(data, el => +el.population);
    d3.max(data, el => +el.population);
    d3.mean(data, el => +el.population);
});
```
d3.extent方便地将d3.min()和d3.max()在一个数组中返回：
```
d3.extent(data, el => +el.population); // 返回[500000, 1300000]
```
现在，我们已经加载、格式化、测量了我们的数据，那我们就可以创建数据可视化了。
## 数据可视化
接下来将更深入解释选择器是如何与数据绑定一起创建元素的，以及创建后如何改变这些元素。
第一个例子使用cities.csv中的数据。
### 选择器及绑定
使用D3选择器来改变web页的结构和呈现。一个选择器是由DOM中一个或多个元素构成。你能使用选择器创建和删除元素，更改样式和内容。
``` javascript
d3.csv("cities.csv", (error,data) => {
    if (error) {
        console.error(error)
    }
    else {
        dataViz(data)
    }
});
function dataViz(incomingData) {
    d3.select("body").selectAll("div.cities")
        .data(incomingData) // 绑定数据到选择器
        .enter() // 定义当选择中的数据多于DOM元素时如何响应
        .append("div") // 在当前选择器中创建一个元素
        .attr("class", "cities") // 设置新创建元素class
        .html(d => d.label); // 设置创建div的内容
}
```
d3.selectAll()传入对应于DOM一部分的CSS id选择器。经常传入id没有匹配到任何元素，那么称其为空选择器，使用.enter()函数在页面上创建新元素。需要指明一个选择器如何创建或更改一个具体DOM元素的孩子元素。注意子选项不会自动生成父节点。父亲必须已经存在，或者需要使用.append()创建一个。
这里需要将选中的DOM元素同一个数组联系起来。数据集里的每一个城市通选中器中的一个DOM元素想联系，关联数据是存在元素的data属性中。
``` javascript
document.getElementsByClassName("cities")[0].__data__ // 返回一个指向对象的指针
```
当数据值的数量大于选择器中元素的数量，.enter()函数触发，允许你为每一个没有相应DOM元素的值定义一个执行行为。
当存在的数据值比较少，.enter()函数会触发
当两者数量一样，则都不会触发
当你知道将不会有更少的数据元素，则不必为.exit()定义处理行为。
.append()函数允许你添加更多元素，定义添加哪个元素。通过.insert()你能控制新元素添加的地方。
.attr()函数是用来改变样式和属性
.html()函数可以设置DOM元素的内容
### 使用内联函数访问数据
在选择器中使用一个内联匿名函数，自动提供两个变量的访问，这对数据呈现很关键。一个是数据值本身，一个是该数据在数组中的位置。
直方图是分类表达数据数值最简单最有效的方法。
``` javascript
d3.select("svg")
    .selectAll("rect")
    .data([15, 50, 22, 8, 100, 10])
    .enter()
    .append("rect")
    .attr("height", d => d)
    .style("fill", "#FE9922")
    .style("stroke", "#9A8B7A")
    .style("stroke-width", "1px")
    .attr("x", (d,i) => i * 10)
    .attr("y", d => 100 - d);
```
多线性比例尺是在domain和range带有多个点的线性比例尺。比如我们对1到100范围内的值特别感兴趣，对100到1000内的值有时候感兴趣。偶尔会得到相当大的异常值。我们能这样表示：
``` javascript
var yScale =
    d3.scaleLinear().domain([0,100,1000,24500]).range([0,50,75,100]);
```
例如，有一些调查返回的数据，我们认为大于500就表示成功。我们仅仅想显示0到500内的数据，同时用比例尺强调0到100的变化。
``` javascript
var yScale = d3.scaleLinear().domain([0,100,500]).range([0,50,100]);
yScale(1000); // 返回162.5
```
通常D3比例尺会推断最小值到最大值之外的值。如果想设置小于最小值的取最小值，大于最大值的取最大值。
可以使用.clamp()函数：
``` javascript
var yScale = d3.scaleLinear()
    .domain([0,100,500])
    .range([0,50,100])
    .clamp(true);
yScale(1000); // 返回100
```
scale函数使决定位置、尺寸、数据可视化元素颜色的关键。
## 数据呈现
``` javascript
d3.csv("cities.csv",(error, data) => { dataViz(data) });
function dataViz(incomingData) {
    var maxPopulation = d3.max(incomingData, d => parseInt(d.population)) // 人口数最大值
    var yScale = d3.scaleLinear().domain([0, maxPopulation]).range([0,460]);
    d3.select("svg").attr("style", "height: 480px; width: 600px;");
    d3.select("svg")
        .selectAll("rect")
        .data(incomingData)
        .enter()
        .append("rect")
        .attr("width", 50)
        .attr("height", d => yScale(parseInt(d.population)))
        .attr("x", (d,i) => i * 60)
        .attr("y", d => 480 - yScale(parseInt(d.population)))
        .style("fill", "#FE9922")
        .style("stroke", "#9A8B7A")
        .style("stroke-width", "1px")
}
```
``` javascript
d3.json("tweets.json",(error, data) => { dataViz(data.tweets) }); 
function dataViz(incomingData) {
    var nestedTweets = d3.nest()
        .key(d => d.user)
        .entries(incomingData);
    nestedTweets.forEach(d => {
        d.numTweets = d.values.length; // 创建一个新属性
    })
    var maxTweets = d3.max(nestedTweets, d => d.numTweets);
    var yScale = d3.scaleLinear().domain([0,maxTweets]).range([0,500]);
    d3.select("svg")
        .selectAll("rect")
        .data(nestedTweets)
        .enter()
        .append("rect")
        .attr("width", 50)
        .attr("height", d => yScale(d.numTweets))
        .attr("x", (d,i) => i * 60)
        .attr("y", d => 500 - yScale(d.numTweets))
        .style("fill", "#FE9922")
        .style("stroke", "#9A8B7A")
        .style("stroke-width", "1px");
}
```
![推特用户直方图](http://pic.caigoubao.cc/604149/blog/visualization/%E6%8E%A8%E7%89%B9%E7%94%A8%E6%88%B7%E7%9B%B4%E6%96%B9%E5%9B%BE.png)
### 设置通道channel
多变量是另一种表示具有多个数据特征的数据点的方式。channel是一个图形如何直观表达数据的技术术语，它取决于你正在使用的数据，不同的通道适用于来表达不同的数据可视化。
下面通过时间和影响因子两个维度来呈现数据：
``` javascript
function dataViz(incomingData) {
    incomingData.forEach(d => {
        d.impact = d.favorites.length + d.retweets.length;
        d.tweetTime = new Date(d.timestamp);
    })
    var maxImpact = d3.max(incomingData, d => d.impact);
    var startEnd = d3.extent(incomingData, d => d.tweetTime);
    var timeRamp = d3.scaleTime().domain(startEnd).range([20, 480]);
    var yScale = d3.scaleLinear().domain([0, maxImpact]).range([0, 460]);
    var radiusScale = d3.scaleLinear()
        .domain([0, maxImpact]).range([1, 20]);
    var colorScale = d3.scaleLinear()
        .domain([0, maxImpact]).range(["white", "#75739F"]);
    d3.select("svg")
        .selectAll("circle")
        .data(incomingData)
        .enter()
        .append("circle")
        .attr("r", d => radiusScale(d.impact))
        .attr("cx", d => timeRamp(d.tweetTime))
        .attr("cy", d => 480 - yScale(d.impact))
        .style("fill", d => colorScale(d.impact))
        .style("stroke", "black")
        .style("stroke-width", "1px");
    };
```
### Enter, update, mergem, exit
.enter()已经用到很多次了。现在让我们近距离看看它， 以及与它配对的.exit()。它们只有在绑定数据个数和选择器中DOM元素个数不匹配时才会触发。使用selection.enter() 定义怎样基于绑定数据创建新元素，使用selection.exit()定义元素相关的数据被删除时，如何移除选择器中的元素。更新绑定数据时，会基于数据重新创建图形元素。
大多数情况，使用.enter()事件，就会使用.append()来添加元素。
![进入退出更新示意图](http://pic.caigoubao.cc/604149/blog/visualization/%E8%BF%9B%E5%85%A5%E9%80%80%E5%87%BA%E6%9B%B4%E6%96%B0%E7%A4%BA%E6%84%8F%E5%9B%BE.png)
``` javascript
d3.selectAll("g").data([1,2,3,4]).exit().remove();
```
这个代码会删除四个<g>元素。选择器前四个元素绑定了新数据，其余的属于.exit()函数。大多数情况下，不会遇到这样的绑定数组完全不同情况，一般通过用户交互或者其他行为触发数据过滤界面呈现变化后会看到初始化数据的重新绑定。
d3.merge()允许合并两个选择器，以致于可以同时操作它们。
通常.data()绑定基于数据值在数组中的位置。如果不想依赖位置，而是依赖你绑定的键值，使用有意义的键值，例如数据对象本身的值。所有对象都被当做[object object]对待，所有可以使用JSON.stringify函数。
``` javascript
function dataViz(incomingData) {
    incomingData.forEach(d => {
        d.impact = d.favorites.length + d.retweets.length;
        d.tweetTime = new Date(d.timestamp);
    })
    var maxImpact = d3.max(incomingData, d => d.impact)
    var startEnd = d3.extent(incomingData, d => d.tweetTime)
    var timeRamp = d3.scaleTime().domain(startEnd).range([ 50, 450 ]);
    var yScale = d3.scaleLinear().domain([ 0, maxImpact ]).range([ 0, 460 ]);
    var radiusScale = d3.scaleLinear()
        .domain([ 0, maxImpact ])
        .range([ 1, 20 ]);
    d3.select("svg").selectAll("circle")
        .data(incomingData, JSON.stringify)
        .enter().append("circle")
        .attr("r", d => radiusScale(d.impact))
        .attr("cx", d => timeRamp(d.tweetTime))
        .attr("cy", d => 480 - yScale(d.impact))
        .style("fill", "#75739F ")
        .style("stroke", "black")
        .style("stroke-width", "1px");
    var filteredData = incomingData.filter(d => d.impact > 0)
    d3.selectAll("circle")
        .data(filteredData, d => JSON.stringify(d))
        .exit()
        .remove();
}
以上代码，会将影响因子不大于0的元素移除。
如果使用转化为字符串的对象，改变数据的话将不起作用，因为它不再与原始绑定的字符串相关。如果计划做重要的改变和更新，数据对象需要一个唯一ID作为绑定键值。
```

## 文档驱动设计及交互
D3v4 有几个方法帮助DOM上下移动：selection.raise 和 selection.lower. 使用这些方法来移动选择的元素到DOM兄弟节点的末尾或者移动到开头位置。
``` javascript 
d3.select("g.overallG").raise()
d3.select("g.overallG").lower()
```
禁用元素的鼠标事件：
``` javascript
teamG.select("text").style("pointer-events","none");
```
### 使用颜色
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
  teamColor.darker(.75) : teamColor.brighter(.5))
```
d3.hsl, d3.lab, d3.cubehelix, d3.hcl可以呈现不同的颜色空间
D3支持颜色插值
``` javascript
var ybRamp = d3.scaleLinear()
  .interpolate(d3.interpolateHsl)
  .domain([0,maxValue]).range(["yellow", "blue"]);
```
离散颜色比例尺：d3.schemeCategory10, d3.schemeCategory20, d3.schemeCategory20b, and d3.schemeCategory20c
序数比例尺，映射离散值为特殊的颜色。一个有用的特征是它的unknown方法，当传入一个不存在的值的时候，返回灰色值。
``` javascript
var tenColorScale = d3.scaleOrdinal()
  .domain(["UEFA", "CONMEBOL"])
  .range(d3.schemeCategory10)
  .unknown("#c4b9ac")
```
## 预生成内容
### 图片
使用insert()替代append()，是告诉D3在文本元素前面嵌入图片，使得标签文字在新添加的图片后被绘制。
``` javascript
d3.selectAll("g.overallG").insert("image", "text")
  .attr("xlink:href", d => `images/${d.team}.png`)
  .attr("width", "45px").attr("height", "20px")
  .attr("x", -22).attr("y", -10)
```
### SVG
```javascript
d3.html("resources/icon_1907.svg", loadSVG);
function loadSVG(svgData) {
  d3.selectAll("g").each(function() {
    var gParent = this;
    d3.select(svgData).selectAll("path").each(function() {
      gParent.appendChild(this.cloneNode(true))
    });
  });
};
d3.selectAll("path").style("fill", "#93C464")
  .style("stroke", "black").style("stroke-width", "1px");
```
效果图
![Football icons](http://pic.caigoubao.cc/604149/blog/visualization/footballs.png)

### 参考资料
- [D3 js in action](https://www.manning.com/books/d3-js-in-action)