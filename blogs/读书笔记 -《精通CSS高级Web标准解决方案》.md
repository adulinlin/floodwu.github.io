# 《精通CSS高级Web标准解决方案》读书笔记
“CSS规范”本身十分复杂，常常还自相矛盾，它的目标读者是浏览器厂商，而不是网页开发人员。

## 第1章 基础知识
“有意义的标签”也称为`“语义标签”`，“有意义”不仅指方便开发人员立即，同时程序或其他设备也可以理解有意义的标记，如搜素引擎可以识别出标题、屏幕阅读器的用户可以依靠标题进行页面导航。

为元素命名，即分配ID和类名时，一定要尽可能保持名称与表现方式无关，应该根据“它们是什么”来为元素命名，而不应该根据“它们的外观如何”来命名。
差的命名：red、leftColumn、topNav、firstPara
好的命名：error、secondaryContent、mainNav、intro

一个ID名只能应用于页面上的一个元素，而同一个类名可以应用于页面上任意多个元素。只有在目标元素非常独特，绝对不会对网站上其他地方别的东西使用这个名称时，才会使用ID。

如果类名中出现了重复的单词，比如news-head、news-link，就应该考虑是否可以把这些元素分解成它们的组成部分，这会让代码更加“组件化”，提高灵活性。如：
```html
<h2 class=’news-head’> ...
<p class=’news-link’> ...</p>
<p class=’news-link’> ...</p>
```
像上面这种对类名的过度依赖是完全不必要的，应该改成：
```html
<h2 class=’news’> ...
<p> ...</p>
<p> ...</p>
```
div用于对块级元素进行分组，span用于对行内元素进行分组。不能过度使用div标签，应该使用div根据条目的意义或功能对相关条目进行分组，不应该根据表现方式或布局来使用div进行分组。



## 第2章 为样式找到应用目标
`类型选择器`\`元素选择器`\`简单选择器`：p {color:black;}

`后代选择器`：blockquote p{color:black;}

`ID选择器`：#intro{color:black;}

`类选择器`：.intro{color:black;}

`伪类`：根据文档结构之外的其他条件对元素应用样式，例如表单元素或链接的状态
tr:hover{background-color:red;}
input:focus{background-color:red;}
a:hover,a:focus,a:active{color:red;}
:link和:visited称为链接伪类，只能应用于锚元素。:hover、:active和:focus称为动态伪类，理论上可以应用于任何元素。
可以把伪类连接在一起，创建更复杂的行为：
a:visited:hover{color:red;}

`通用选择器`：匹配所有可用元素
```css
*{
padding:0;
margin:0;
}
```
通用选择器与其他选择器结合使用时，可以用来对某个元素的所有后代应用样式。

`子选择器`：只选择元素的直接后代，而不是像后代选择器一样选择元素的所有后代。
```css
#nav>li{
padding-left:20px;
color:red;
}
```

`相邻同胞选择器`：用于定位同一个父元素下与某个元素相邻的下一个元素。
```css
h2 + p{
font-size:1.4em;
}
```

`属性选择器`：根据某个属性是否存在或者属性的值来寻找元素。
```css
abbr[title]{
border-bottom:1px dotted #999;
}

abbr[title]:hover{
cursor:help;
}

a[rel=’nofollow’]{
color:red;
}
```
注意：属性名无引号，属性值有引号。

对于属性可以有多个值的情况（空格分割），属性选择器允许根据属性值之一来寻找元素：
.blogroll a[rel~=’co-worker’]{...}

`层叠`：通常会有多个规则能够寻找到同一个元素，CSS通过层叠的过程来处理这种冲突。层叠会给每个规则分配一个重要度，`重要度次序如下`：
（1）标有!important的用户样式
（2）标有!important的作者样式
（3）作者样式
（4）用户样式 //即通过浏览器指定的CSS规则，IE中的设置方式为：选项-常规-辅助功能
（5）浏览器的样式

`特殊性`：在层叠重要度次序的基础上，会根据选择器的特殊性决定规则的次序。具有更特殊选择器的规则优先于具有一般选择器的规则，如果两个规则的特殊性相同，那么后定义的规则优先。
选择器的特殊性分为abcd共4个成分等级：
（1）如果样式是行内样式，a=1。行内样式即直接在元素上应用style属性的样式
（2）b等于ID选择器的总数
（3）c等于类、伪类和属性选择器的总数
（4）d等于类型选择器和伪元素选择器的数量
总结起来就是：style > ID > 类 > 元素

`继承`：应用样式的元素的后代会继承样式的某些属性，比如颜色、字号等。可以直接给body设置color属性。但是当直接设置body的size属性时，对于h1、h2等却无效。这是因为浏览器的默认样式表设置了标题字号：直接应用于元素的任何样式总会覆盖继承而来的样式，因为继承而来的样式的特殊性为空。

在HTML文档中导入CSS文件：
```html
<link href=’css/basic.css’ rel=’stylesheet’ type=’text/css’ />
```

在CSS文件中导入CSS文件：
```html
<style type=’text/css’>
<!--
@import url(‘/css/advanced.css’);
-->
</style>
```

尽量使用单一的CSS文件而不是将其分为多个小文件：因为多个文件会导致多次服务器请求，这将影响下载时间。此外浏览器只能同时从一个域名下载数量有限的文件。

CSS`使用C风格的注释`，即 /* .. */，注释可以单行，也可以多行，而且可以出现在代码中的任何地方。
```css
/* Body Style */
body{
font-size:1em;  /* set the font size */
}
```

## 第3章 可视化格式模型
`盒模型`：
![image](https://github.com/woojean/woojean.github.io/blob/master/images/css_1.png)
（1）由内到外为：内容-内边距-边框-外边距。
（2）如果在元素上添加背景，那么背景会被应用于由内容和内边距组成的区域。外边距是透明的，一般用来控制元素之间的间隔。
（3）此外CSS2.1还支持outline属性，与border不同的是其将轮廓绘制在元素框上。
（4）内边距、边框、外边距都是可选的，默认值为0，但是许多元素会被浏览器设置外边距和内边距，可以通过如下方式重置：
```css
 	*{
margin:0;
padding:0;
}
```
（5）但是这种方式不区分元素，可能会对如option等元素造成不利影响，因此使用全局reset把内边距和外边距显式地设置为0可能更安全。
（6）width和height指的是内容区域的宽度和高度，增加内边距、边框和外边距不会影响内容区域的尺寸，但是会增加元素框的总尺寸。
（7）内边距、边框、外边距可以应用于一个元素的所有边，也可以应用于单独的边。外边距还可以是负值。

`外边距叠加`：
（1）当两个或更多个垂直外边距相遇时，它们将合并为一个外边距，这个新外边距的高度等于两个发生叠加的外边距的高度中的较大者。
（2）当一个元素包含在另一个元素中时，如果没有内边距或者边框将外边距分隔开，那么它们的顶、底外边距也会发生叠加。
（3）甚至同一个元素，如果没有内边距、边框以及内容，此时它的顶外边距与底外边距碰在一起，也会发生叠加。而且如果这个新的外边距碰到了另一个元素的外边距，它还会发生叠加。
（4）注意：只有普通文档流中块框的垂直外边距才会发生外边距叠加。行内框、浮动框或者绝对定位框之间的外边距不会叠加。


`可视化格式模型`：
（1）块级元素：显示为一块内容，即块框，如p、h1、div等。
（2）行内元素：内容显示在行中，即行内框，如strong、span等。
（3）可以使用display属性来改变生成的框的类型，如将a标签的display设置为block，从而让其表现的像块级元素一样；还可以设置display属性为none，让生成的元素根本没有框，不占用文档中的空间。
（4）CSS中有3种基本的定位机制：普通流、浮动、绝对定位。
（5）块级框从上到下一个接一个地垂直排列，框之间的垂直距离由框的垂直外边距计算出来。
（6）行内框在一行中水平排列。可以使用水平内边距、边框、外边距来调整它们的水平间距，但是行内框的垂直内边距、边框和外边距不会增加行高，设置显式的高度或宽度也不行。由一行形成的水平框称为行框，行框高度等于本行内所有元素中行高最大的值，可以通过设置行高（line-height）来修改这个高度。CSS2.1支持将display属性设置为inline-block，这将使元素像行内元素一样水平地依次排列，但是框的内容仍然符合块级框的行为，如能够显式地设置宽度、高度、垂直外边距和内边距。

`匿名块框`：当将文本添加到一个块级元素的开头时，即使没有把这些文本定义为块级元素，它也会被当成块级元素对待：
```html
<div>
 	some text
<p>other text</p>
</div>
```

`匿名行框`：块级元素内的文本，每一行都会形成匿名行框。无法直接对匿名块或者行框应用样式，除非使用:first-line伪元素。

`相对定位`：如果对一个元素进行相对定位，它将出现在它所在的位置上，然后可以通过设置top、left等属性让这个元素相对于它的起点移动。无论是否移动，元素仍然占据原来的空间，因此移动元素会导致它覆盖其他框。相对定位实际上是普通流定位模型的一部分。
![image](https://github.com/woojean/woojean.github.io/blob/master/images/css_2.png)

`绝对定位`：绝对定位的元素的位置是相对于距离它最近的那个已定位的祖先元素确定的，如果没有已定位的祖先元素，那么它的位置是相对于初始包含块的。元素定位后生成一个块级框，而不论原来它在正常流中生成何种类型的框。绝对定位使元素的位置与文档流无关。
![image](https://github.com/woojean/woojean.github.io/blob/master/images/css_3.png)

`固定定位`：相对于viewport进行定位。

`浮动`：浮动的框可以左右移动，直到它的外边缘碰到包含框或另一个浮动框的边缘。浮动框不在文档的普通流中。
![image](https://github.com/woojean/woojean.github.io/blob/master/images/css_4.png)
当框 1 向左浮动时，它脱离文档流并且向左移动，直到它的左边缘碰到包含框的左边缘。因为它不再处于文档流中，所以它不占据空间，实际上覆盖住了框 2，使框 2 从视图中消失：
![image](https://github.com/woojean/woojean.github.io/blob/master/images/css_5.png)
如果包含框太窄，无法容纳水平排列的三个浮动元素，那么其它浮动块向下移动，直到有足够的空间。如果浮动元素的高度不同，那么当它们向下移动时可能被其它浮动元素“卡住”：
![image](https://github.com/woojean/woojean.github.io/blob/master/images/css_6.png)

`行框和清理`：浮动框旁边的行框被缩短，从而给浮动框留出空间，行框围绕浮动框。
因此，创建浮动框可以使文本围绕图像：
![image](https://github.com/woojean/woojean.github.io/blob/master/images/css_7.png)
要想阻止行框围绕浮动框，需要对该框应用clear属性。clear 属性定义了元素的哪边上不允许出现浮动元素。在 CSS1 和 CSS2 中，这是通过自动为清除元素（即设置了clear属性的元素）增加上外边距实现的。在 CSS2.1 中，会在元素上外边距之上增加清除空间，而外边距本身并不改变。不论哪一种改变，最终结果都一样，如果声明为左边或右边清除，会使元素的上外边框边界刚好在该边上浮动元素的下外边距边界之下。（即浏览器会自动添加上外边距）
![image](https://github.com/woojean/woojean.github.io/blob/master/images/css_8.png)

## 第4章 背景图像效果
```css
/* 设置背景图像，且水平平铺 */
body{
background-image:url(/img/picture.gif);  /* 图片路径没有引号 */
background-repeat:repeat-x;
}

/* 一次性设置背景属性 */
h1{
background: #ccc url(/img/back.gif) no-repeat left center;
}

/* 使用背景图像实现各种圆角框 */
略：过时技术，使用新的border-radius、border-image属性可以轻松实现。

/* 使用背景图像实现图片投影效果 */
略：过时技术，使用新的box-shadow属性可以轻松实现。

/* 用opacity/filter实现透明提示框 */
.alert{
background-color:#000;
opacity:0.8;
filter:alpha(opacity=80);
}
这种方式的主要问题是，透明度除了对背景生效外，应用它的元素的内容也会继承透明度设置，造成提示框的文本也透明。
更好的方式是使用RGBa：
.alert{
background-color:rgba(0,0,0,0.8);
}
```
/* 按百分比使用background-position属性实现改变窗口大小时的视差效果 */
略：用到再看

/* 图像替换：将文本添加到文档中，设置背景图片，然后通过CSS隐藏文本，从而既可以显示文本，又不影响搜索引擎的语义解析 */
略：用到再看

## 第5章 对链接应用样式
`:active`动态伪类选择器用来寻找被激活的元素，对于链接来说，激活发生在链接被单击时。

在定义`:hover`状态时，最好也同时定义`:focus`，这样在通过键盘移动到链接上时，会让链接显示的样式与鼠标悬停时相同。

### 去掉链接的下划线，在悬停、激活时显示：
a:link, a:visited {text-decoration:none;}
a:hover,a:focus,a:active{text-decoration:underline;}
选择器的次序很重要，如果定义顺序反过来：
a:hover,a:focus,a:active{text-decoration:underline;}
a:link, a:visited {text-decoration:none;}
则鼠标悬停和激活的样式就不起作用了。这是因为两个规则具有相同的特殊性，所以a:link, a:visited将覆盖a:hover,a:focus,a:active。最好按如下顺序进行定义：
a:link、a:visited、a:hover、a:focus、a:active

### 为链接目标（同一页面的锚点）设置样式
```css
:target
{
border: 2px solid #D4D4D4;
background-image: url(img/fade.gif);  // 设置一个黄色渐变为白色的动画图片
}
```
### 在站点的所有外部链接的右上角显示一个图标
```css
a[href^=’http:’]{  /* 使用属性选择器 */
background:url(/img/external.gif) no-repeat right top;
padding-right:10px;
}

a[href^=’http://www.mysite.com’]{  /* 覆盖排除本站点的绝对链接 */
background:none;
padding-right:0;
}
```
### 为站点的所有下载.pdf文档的链接加上图标
```css
a[href$=’.pdf’]{
background:url(img/pdf.gif) no-repeat right top;
padding-right:10px;
}
```
### 创建类似按钮的链接
```css
a{
display:block;
width:6.6em;
line-height:1.4;  /* 这里使用line-height能够使文本垂直居中。如果使用height，则还需要结合padding来处置居中 */
text-align:center;
text-decoration:none;
border:1px solid #66a300;
background-color:#8cca12;
color:#fff;
}
```
### 实现按钮状态翻转
```css
a:hover,a:focus{
background-color:#f7a300;
border-color:#ff7400;
}
```
### pixy方法
使用多个图片来做不同状态下的背景，以实现按钮状态转换的方式，会在按钮状态切换时有闪烁。这可以通过使用pixy方法来解决。即按钮的不同状态使用同一张图片，仅通过切换不同位置来实现状态转换：
```css
a:link,a:visited{
display:block;
width:203px;
height:72px;
text-indent:-1000em;  /* 使按钮文字不可见 */
background:url(/img/buttons.png) -203px 0 no-repeat;  /* 正常状态下，使背景图像在中间显示 */
}

a:hover,a:focus{
background-position:right top;
}

a:active{
background-position:left top;
}
```
但是这种方式在IE下仍然会有轻微闪烁，解决方式略。

CSS精灵：pixy方法的进一步应用。即把站点的所有图标甚至导航等都包含在一个图像中，从而减少请求数量。

CSS3的新特性：text-shadow、box-shadow、border-radius等属性可以用来替换以上使用图片创建按钮切换的方式。

用纯CSS来创建tooltips效果
略

## 第6章 对列表应用样式和创建导航条
### 创建基本的垂直导航条
![image](https://github.com/woojean/woojean.github.io/blob/master/images/css_9.png)
<ul class="nav">
<li class="selected"><a href="home.htm">Home</a></li>
<li><a href="#">About</a></li>
<li><a href="#">Our Services</a></li>
<li><a href="#">Our Work</a></li>
<li><a href="#">News</a></li>
<li class="last"><a href="contact.htm">Contact</a></li>
</ul>

```css
ul.nav {
  	margin: 0;  /* 不同的浏览器对列表的缩进控制方式不同，所以需要去掉这个缩进，然后在列表项上定制 */
  	padding: 0;
  	width: 8em;
  	list-style-type: none;  /* 可以使用list-style-image来控制项目符号，但是这种方式的控制能力不强，更好的方式是关闭项目符号，然后在列表项上定制 */
float: left;
	background-color: #8BD400;
	border: 1px solid #486B02;
	border-bottom: none;
}

ul.nav a {
  	display: block;
	color: #2B3F00;
  	text-decoration: none;
	border-top: 1px solid #E4FFD3;
	border-bottom: 1px solid #486B02;
  	background: url(img/arrow.gif) no-repeat 5% 50%;
	padding: 0.3em 1em;
}

ul.nav a:hover,ul.nav a:focus,ul.nav .selected a {  /* 定义交互状态 */
	color: #E4FFD3;
	background-color: #6DA203;
}
```
### 在导航条中突出显示当前页
为每个页面的body添加一个ID，并在导航列表中的每一项上定义相应的类名，然后结合两者实现突出显示当前页，以主页home为例：
<body id=’home’>
<ul class="nav">
<li class="home"><a href="home.htm">Home</a></li>
<li class="about"><a href="#">About</a></li>
...
</ul>

然后可以在CSS中这样选择：
```css
#home .nav .home a, #about .nav .about a ...{
color: #E4FFD3;
	background-color: #6DA203;
}
```

### 创建简单的水平导航条
 ![image](https://github.com/woojean/woojean.github.io/blob/master/images/css_10.png)
 ```html
<ol class="pagination">
<li><a href="search.htm?page=1" rel="prev">Prev</a></li>
<li><a href="search.htm?page=1">1</a></li>
<li class="selected">2</li>
<li><a href="search.htm?page=3">3</a></li>
<li><a href="search.htm?page=4">4</a></li>
<li><a href="search.htm?page=5">5</a></li>
<li><a href="search.htm?page=3" rel="next">Next</a></li>
</ol>
```

```css
ol.pagination {  /* 去掉默认缩进 */
  margin: 0;
  padding: 0;
  list-style-type: none;
}

ol.pagination li {  /* 将列表项向左浮动，则列表将水平排列 */
	float: left;
	margin-right: 0.6em;
}

ol.pagination a,ol.pagination li.selected {  /* 去掉默认缩进 */
	display: block;
	padding: 0.2em 0.5em;
	border: 1px solid #ccc;
	text-decoration: none;
}

ol.pagination a[rel="prev"],ol.pagination a[rel="next"] {
	border: none;
}

ol.pagination a[rel="prev"]:before {   /* CSS注入 */
	content: "\00AB";
	padding-right: 0.5em;
}

ol.pagination a[rel="next"]:after {  	/* CSS注入 */
	content: "\00BB";
	padding-left: 0.5em;
}

ol.pagination a:hover,ol.pagination a:focus,ol.pagination li.selected {
	background-color: blue;
	color: white;
}
```
### 创建图形化导航条
略：使用图片，不优雅

### 简化的滑动门标签式导航
略：使用图片，不优雅

### 下拉菜单
略：can do

### CSS图像映射
略：原文“图像映射在几年前非常流行，但是近来不太常见了，部分原因是flash流行起来了，还有部分原因是发展出了更简单、表现性更低的标记”

### 远距离翻转
 ![image](https://github.com/woojean/woojean.github.io/blob/master/images/css_11.png)
实现方式是：在锚链接内嵌套一个或多个元素，然后使用绝对定位对嵌套的元素分别定位。尽管显示在不同的地方，但是它们都包含在同一个父锚中，所以可以对同一个鼠标悬停事件作出反应，当鼠标悬停在一个元素上时，可以影响另一个元素的样式。
```html
<div class="remote">
<img src="img/nerdcore.jpg" width="333" height="500" alt="Rich, Sophie, Cath, James and Paul" />
<ul>
<li class="rich">
	<a href="http://www.clagnut.com/" title="Richard Rutter">
		<span class="hotspot"></span>
		<span class="link">&raquo; Richard Rutter</span>
	</a>
</li>
```
如上，在一个a内部放两个span，然后通过绝对定位的方式将两个span分别定位到图片与列表中，并分别定义hover时的行为：显示边框和改变字体颜色：
```css
.remote a:hover .hotspot,.remote a:focus .hotspot {
  border: 1px solid #fff;
}

.remote a:hover .link,.remote a:focus .link {
  color: #0066FF;
}
```
## 第7章 对表单和数据表格应用样式
定义一个不带样式的表格：
![image](https://github.com/woojean/woojean.github.io/blob/master/images/css_12.png)

添加样式后效果：
![image](https://github.com/woojean/woojean.github.io/blob/master/images/css_13.png)
具体过程略。

简单的表单布局
具体细节略，需要注意的就是fieldset和legend这两个标签的使用。
```html
<fieldset>
	<legend>Your Contact Details</legend>
	<div>
	<label for="author">Name: <em class="required">(Required)</em></label>
	<input name="author" id="author" type="text" />
	</div>
	
	<div>
	<label for="email">Email Address:</label>
	<input name="email" id="email" type="text" />
	</div>
</fieldset>
```
效果：
![image](https://github.com/woojean/woojean.github.io/blob/master/images/css_14.png)


## 第8章 布局
所有CSS布局技术的根本都是3个基本概念：定位、浮动、外边距操作。

三种布局方式：（原文描述混乱不清，以下摘自网络）
`固定布局`：
![image](https://github.com/woojean/woojean.github.io/blob/master/images/css_15.png)

`流式布局`
![image](https://github.com/woojean/woojean.github.io/blob/master/images/css_16.png)

`弹性布局`：相当于以上两者的结合。其要点就在于使用单位em来定义元素宽度。em是相对长度单位。相对于当前对象内文本的字体尺寸。如当前对行内文本的字体尺寸未被人为设置，则相对于浏览器的默认字体尺寸。任意浏览器的默认字体高都是16px，所有未经调整的浏览器都符合: 1em=16px。


详略

## 第9章 捕捉bug
大部分内容是处理低版本IE上的BUG，略

## 第10章、第11章 两个实例
略

