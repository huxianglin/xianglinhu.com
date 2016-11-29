# 前端之css

## 本节内容
1. css概述及引入
2. css选择器
3. css常用属性

## 1.css概述及引入
### CSS概述
CSS是Cascading Style Sheets的简称，中文称为层叠样式表，用来控制网页数据的表现，可以使网页的表现与数据内容分离。

### css的四种引入方式
#### 1>. 行内式

行内式是在标记的style属性中设定CSS样式。这种方式没有体现出CSS的优势，不推荐使用。（但是一般通过js动态加载一些css属性都是通过调节行内style属性来实现的）

    <p style="background-color: rebeccapurple"\>hello yuan</p\>

#### 2>. 嵌入式

嵌入式是将CSS样式集中写在网页的<head\></head\>标签对的<style\></style\>标签对中。格式如下：

    <head>
        <meta charset="UTF-8">
        <title>Title</title>
        <style>
            p{
                background-color: #2b99ff;
            }
        </style>
    </head>

#### 3>. 链接式

将一个.css文件引入到HTML文件中,最常用方式

    <link href="mystyle.css" rel="stylesheet" type="text/css"/>

#### 4>. 导入式

将一个独立的.css文件引入HTML文件中，导入式使用CSS规则引入外部CSS文件，<style\>标记也是写在<head\>标记中，使用的语法如下：

    <style type="text/css">
              @import"mystyle.css"; 此处要注意.css文件的路径
    </style>

注意：

导入式会在整个网页装载完后再装载CSS文件，因此这就导致了一个问题，如果网页比较大则会儿出现先显示无样式的页面，闪烁一下之后，再出现网页的样式。这是导入式固有的一个缺陷。使用链接式时与导入式不同的是它会以网页文件主体装载前装载CSS文件，因此显示出来的网页从一开始就是带样式的效果的，它不会象导入式那样先显示无样式的网页，然后再显示有样式的网页，这是链接式的优点。

## 2. css选择器
“选择器”指明了{}中的“样式”的作用对象，也就是“样式”作用于网页中的哪些元素

### 1. 基础选择器
    ＊ ：           通用元素选择器，匹配任何元素                    * { margin:0; padding:0; }
    
    E  ：          标签选择器，匹配所有使用E标签的元素               p { color:green; }
    
    .info和E.info: class选择器，匹配所有class属性中包含info的元素   .info { background:#ff0; }    p.info { background:blue; }
    
    #info和E#info  id选择器，匹配所有id属性等于footer的元素         #info { background:#ff0; }   p#info { background:#ff0; }

### 2. 组合选择器
     E,F         多元素选择器，同时匹配所有E元素或F元素，E和F之间用逗号分隔         div,p { color:#f00; }
    
     E F         后代元素选择器，匹配所有属于E元素后代的F元素，E和F之间用空格分隔    li a { font-weight:bold;
     E > F       子元素选择器，匹配所有E元素的子元素F                            div > p { color:#f00; }
     
     E + F       毗邻元素选择器，匹配所有紧随E元素之后的同级元素F                  div + p { color:#f00; } 

注意嵌套规则：


1. 块级元素可以包含内联元素或某些块级元素，但内联元素不能包含块级元素，它只能包含其它内联元素。
2. 块级元素不能放在p里面。
3. 有几个特殊的块级元素只能包含内联元素，不能包含块级元素。如h1,h2,h3,h4,h5,h6,p,dt
4. li内可以包含div
5. 块级元素与块级元素并列、内联元素与内联元素并列。（错误的：<div\><h2\></h2\><span\></span\></div\>）

### 3. 属性选择器
     E[att]         匹配所有具有att属性的E元素，不考虑它的值。（注意：E在此处可以省略，比如“[cheacked]”。以下同。）   p[title] { color:#f00; }
    
     E[att=val]     匹配所有att属性等于“val”的E元素                                 div[class=”error”] { color:#f00; }
     
     E[att~=val]    匹配所有att属性具有多个空格分隔的值、其中一个值等于“val”的E元素      td[class~=”name”] { color:#f00; }
    
     E[attr^=val]    匹配属性值以指定值开头的每个元素                     div[class^="test"]{background:#ffff00;}
    
     E[attr$=val]    匹配属性值以指定值结尾的每个元素                     div[class$="test"]{background:#ffff00;}
    
     E[attr*=val]    匹配属性值中包含指定值的每个元素                     div[class*="test"]{background:#ffff00;}
    
     p:before        在每个 <p> 元素的内容之前插入内容                    p:before{content:"hello";color:red}
    
     p:after         在每个 <p> 元素的内容之前插入内容                    p:after{ content:"hello"；color:red}

### 4. 伪类(Pseudo-classes)
CSS伪类是用来给选择器添加一些特殊效果。

anchor伪类：专用于控制链接的显示效果

    a:link（没有接触过的链接）,用于定义了链接的常规状态。
    
    a:hover（鼠标放在链接上的状态）,用于产生视觉效果。
    
    a:visited（访问过的链接）,用于阅读文章，能清楚的判断已经访问过的链接。
    
    a:active（在链接上按下鼠标时的状态）,用于表现鼠标按下时的链接状态。
    
    伪类选择器 : 伪类指的是标签的不同状态:
    
    a ==> 点过状态 没有点过的状态 鼠标悬浮状态 激活状态
    
    a:link {color: #FF0000} /* 未访问的链接 */
    
    a:visited {color: #00FF00} /* 已访问的链接 */
    
    a:hover {color: #FF00FF} /* 鼠标移动到链接上 */
    
    a:active {color: #0000FF} /* 选定的链接 */ 格式: 标签:伪类名称{ css代码; }

举例

    <style type="text/css">
        a:link{
            color: red;
        }
        a:visited {
            color: blue;
        }
        a:hover {
            color: green;
        }
        a:active {
            color: yellow;
        }
    </style>
    </head>
    <body>
        <a href="#">hello-world</a>
    </body>
    </html>

before after伪类 ：

    :before    p:before    在每个<p>元素之前插入内容
    :after    p:after        在每个<p>元素之后插入内容
        
     p:after{
                content: 'welcome!';
            }

## 3. css的常用属性

### 1>.颜色属性
    <div style="color:blueviolet">ppppp</div>
    <div style="color:#ffee33">ppppp</div>
    <div style="color:rgb(255,0,0)">ppppp</div>
    <div style="color:rgba(255,0,0,0.5)">ppppp</div>

### 2>.字体属性
    font-size: 20px/50%/larger
    font-family:'Lucida Bright'
    font-weight: lighter/bold/border/
    <h1 style="font-style: oblique">你好</h1>

### 3>.背景属性
    background-color: cornflowerblue;
    background-image: url('1.jpg');
    background-repeat: no-repeat;(repeat:平铺满)
    background-position: right top（20px 20px）;(横向：left center right)(纵向：top center bottom)
          简写： <body style="background: 20px 20px no-repeat #ff4 url('1.jpg')">
                <div style="width: 300px;height: 300px;background: 20px 20px no-repeat #ff4 url('1.jpg')"> 

注意：如果将背景属性加在body上，要记得给body加上一个height，否则结果异常，这是因为body为空，无法撑起背景图片；另外，如果此时要设置一个

width＝100px，你也看不出效果，除非你设置出html。  

### 4>.文本属性
    font-size: 10px;
    text-align: center;横向排列
    line-height: 200px;文本行高 通俗的讲，文字高度加上文字上下的空白区域的高度 50%:基于字体大小的百分比
    p
    { width: 200px;
    height: 200px;
    text-align: center;
    background-color: aquamarine;
    line-height: 200px; }
    text-indent: 150px; 首行缩进，50%：基于父元素（weight）的百分比
    letter-spacing: 10px;
    word-spacing: 20px;
    direction: rtl;
    text-transform: capitalize;

### 5>.边框属性
    border-style: solid;
    border-color: chartreuse;
    border-width: 20px;
    简写：border: 30px rebeccapurple solid;

### 6>.列表属性
    ul,ol{   list-style: decimal-leading-zero;
             list-style: none;
             list-style: circle;
             list-style: upper-alpha;
             list-style: disc; }

### 7>.dispaly属性
    none
    block
    inline
    display:inline-block可做列表布局，其中的类似于图片间的间隙小bug可以通过如下设置解决：
    #outer{
                border: 3px dashed;
                word-spacing: -5px;
            }

### 8>.外边距和内边 
![](http://7xsn7l.com2.z0.glb.clouddn.com/boxmodule.png)


- margin:            用于控制元素与元素之间的距离；margin的最基本用途就是控制元素周围空间的间隔，从视觉角度上达到相互隔开的目的。
- padding:           用于控制内容与边框之间的距离；   
- Border(边框)     围绕在内边距和内容外的边框。
- Content(内容)   盒子的内容，显示文本和图像。

元素的宽度和高度:

重要: 当你指定一个CSS元素的宽度和高度属性时，你只是设置内容区域的宽度和高度。要知道，完全大小的元素，你还必须添加填充，边框和边距。

下面的例子中的元素的总宽度为300px：

    width:250px;
    padding:10px;
    border:5px solid gray;
    margin:10px; 

练习： 300px*300px的盒子装着100px*100px的盒子，分别通过margin和padding设置将小盒子 移到大盒子的中间

    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
        <style>
    
            .div1{
                background-color: aqua;
                width: 300px;
                height: 300px;
            }
            .div2{
                background-color: blueviolet;
                width: 100px;
                height: 100px;
            }
        </style>
    </head>
    <body>
           <div class="div1">
               <div class="div2"></div>
               <div class="div2"></div>
           </div>
    </body>
    </html>

思考1:

边框在默认情况下会定位于浏览器窗口的左上角，但是并没有紧贴着浏览器的窗口的边框，这是因为body本身也是一个盒子（外层还有html），在默认情况下，   body距离html会有若干像素的margin，具体数值因各个浏览器不尽相同，所以body中的盒子不会紧贴浏览器窗口的边框了，为了验证这一点，加上：

    body{
        border: 1px solid;
        background-color: cadetblue;
    }

解决方法：

    body{
        margin: 0;
    }

思考2：

margin collapse（边界塌陷或者说边界重叠）

外边距的重叠只产生在普通流文档的上下外边距之间，这个看起来有点奇怪的规则，其实有其现实意义。设想，当我们上下排列一系列规则的块级元素（如段     落P）时，那么块元素之间因为外边距重叠的存在，段落之间就不会产生双倍的距离。又比如停车场

1兄弟div：上面div的margin-bottom和下面div的margin-top会塌陷，也就是会取上下两者margin里最大值作为显示值

2父子div    

if  父级div中没有 border，padding，inline content，子级div的margin会一直向上找，直到找到某个标签包括border，padding，inline content              中的其中一个，然后按此div 进行margin ；

    <!DOCTYPE html>
    <html lang="en" style="padding: 0px">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
        <style>
    
            body{
                margin: 0px;
            }
    
            .div1{
                background-color: aqua;
                width: 300px;
                height: 300px;
            }
            .div2{
                background-color: blueviolet;
                width: 100px;
                height: 100px;
                margin: 20px;
    
            }
        </style>
    </head>
    <body>
    
    <div style="background-color: cadetblue;width: 300px;height: 300px"></div>
    
           <div class="div1">
               <div class="div2"></div>
               <div class="div2"></div>
           </div>
    
    </body>
    </html>

 解决方法：

    1:border:1px solid transparent
    2:  padding:1px
    3: over-flow:hidden; 

### 9>.float属性
首先要知道，div是块级元素，在页面中独占一行，自上而下排列，也就是传说中的流。无论多么复杂的布局，其基本出发点均是：“如何在一行显示多个div元素”。浮动可以理解为让某个div元素脱离标准流，漂浮在标准流之上，和标准流不是一个层次。 

假如某个div元素A是浮动的，如果A元素上一个元素也是浮动的，那么A元素会跟随在上一个元素的后边(如果一行放不下这两个元素，那么A元素会被挤到下一行)；如果A元素上一个元素是标准流中的元素，那么A的相对垂直位置不会改变，也就是说A的顶部总是和上一个元素的底部对齐。

div的顺序是HTML代码中div的顺序决定的。

靠近页面边缘的一端是前，远离页面边缘的一端是后。

注意：当初float被设计的时候就是用来完成文本环绕的效果，所以文本不会被挡住，这是float的特性，即float是一种不彻底的脱离文档流方式。

清除浮动：

在非IE浏览器（如Firefox）下，当容器的高度为auto，且容器的内容中有浮动（float为left或right）的元素，在这种情况下，容器的高度不能自动伸长以适应内容的高度，使得内容溢出到容器外面而影响（甚至破坏）布局的现象。这个现象叫浮动溢出，为了防止这个现象的出现而进行的CSS处理，就叫CSS清除浮动。

    清除浮动的关键字是clear，官方定义如下：
    语法：
    clear : none | left | right | both
    取值：
    none  :  默认值。允许两边都可以有浮动对象
    left   :  不允许左边有浮动对象
    right  :  不允许右边有浮动对象
    both  :  不允许有浮动对象

方式1(推荐):

    .clearfix:after {             <----在类名为“clearfix”的元素内最后面加入内容； 
    content: ".";                 <----内容为“.”就是一个英文的句号而已。也可以不写。 
    display: block;               <----加入的这个元素转换为块级元素。 
    clear: both;                  <----清除左右两边浮动。 
    visibility: hidden;           <----可见度设为隐藏。注意它和display:none;是有区别的。visibility:hidden;仍然占据空间，只是看不到而已； 
    line-height: 0;               <----行高为0； 
    height: 0;                    <----高度为0； 
    font-size:0;                  <----字体大小为0； 
    } 
    .clearfix { *zoom:1;}         <----这是针对于IE6的，因为IE6不支持:after伪类，这个神奇的zoom:1让IE6的元素可以清除浮动来包裹内部元素。

整段代码就相当于在浮动元素后面跟了个宽高为0的空div，然后设定它clear:both来达到清除浮动的效果。 
之所以用它，是因为，你不必在html文件中写入大量无意义的空标签，又能清除浮动。 

话说回来，你这段代码真是个累赘啊，这样写不利于维护。 
只要写一个.clearfix就行了，然后在需要清浮动的元素中 添加clearfix类名就好了。 
如：

    <div class="head clearfix"></div>

方式2：

    overflow:hidden;<em id="__mceDel" style="background-color: #ffffff; font-family: verdana, Arial, Helvetica, sans-serif; font-size: 14px">

### 10>.position
1 static，默认值 static：无特殊定位，对象遵循正常文档流。

top，right，bottom，left等属性不 会被应用。 说到这里我们不得不提一下一个定义——文档流，文档流其实就是文档的输出顺序， 也就是我们通常看到的由左      到右、由上而下的输出形式，在网页中每个元素都是按照这个顺序进行排序和显示的，而float和position两个属性可以将元素从文档流脱离出来显示。 默认值就        是让元素继续按照文档流显示，不作出任何改变。

2  position: relative／absolute

relative：对象遵循正常文档流，但将依据top，right，bottom，left等属性在正常文档流中偏移位置。而其层叠通过z-index属性定义。

absolute：对象脱离正常文档流，使用top，right，bottom，left等属性进行绝对定位。而其层叠通过z-index属性定义。 如果设定 position:relative，就可以使用 top,bottom,left和 right 来相对于元素在文档中应该出现的位置来移动这个元素。[意思是元素实际上依然占据文档 中的原有位置，只是视觉上相对于它在文档中的原有位置移动了] 当指定 position:absolute 时，元素就脱离了文档[即在文档中已经不占据位置了]，可以准确的按照设置的 top,bottom,left 和 right 来定位了。 

注意1：如果没有设置left,top又没有设置right，bottom，它跟static时的位置一样,会被他的上一个static或relative兄弟挤下去，但是它本身不占位置

注意2：

If the element has 'position: absolute', the containing block is established bythe nearest ancestor with a 'position' of 'absolute', 'relative' or 'fixed', in the following way: 

1.In the case that the ancestor is an inline element, the containing block is the bounding box around the padding boxes of the first and the last inline boxes generated for that element. In CSS 2.1, if the inline element is split across multiple lines, the containing block is undefined. 

2.Otherwise, the containing block is formed by the padding edge of the ancestor.

If there is no such ancestor, the containing block is theinitial containing block.

对这个 initial containing block 规范前文有描述：

The containing block in which the root element lives is a rectangle called the initial containing block. For continuous media,it has the dimensions of the viewport and is anchored at the canvas origin; 

这就是说，initial containing block 是以整个 canvas (渲染内容的空间, http://www.w3.org/TR/CSS2/intro.html#canvas) 的坐标原点(左上)为基准，以 viewport (也就是浏览器视窗内渲染 HTML 的空间)为大小的矩形。

3  position:fixed

在理论上，被设置为fixed的元素会被定位于浏览器窗口的一个指定坐标，不论窗口是否滚动，它都会固定在这个位置。

 fixed：对象脱离正常文档流，使用top，right，bottom，left等属性以窗口为参考点进行定位，当出现滚动条时，对象不会随着滚动。而其层叠通过z-index属性      定义。 注意点： 一个元素若设置了 position:absolute | fixed; 则该元素就不能设置float。这 是一个常识性的知识点，因为这是两个不同的流，一个是浮动流，    另一个是“定位流”。但是 relative 却可以。因为它原本所占的空间仍然占据文档流。