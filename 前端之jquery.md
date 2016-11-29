# 前端之jquery

## 本节内容
1. jquery简介
2. 选择器和筛选器
3. 操作元素
4. 示例

## 1. jquery简介
1 jquery是什么

- jQuery由美国人John Resig创建，至今已吸引了来自世界各地的众多 javascript高手加入其team。
- jQuery是继prototype之后又一个优秀的Javascript框架。其宗旨是——WRITE LESS,DO MORE,写更少的代码,做更多的事情。
- 它是轻量级的js库(压缩后只有21k) ，这是其它的js库所不及的，它兼容CSS3，还兼容各种浏览器 （IE 6.0+, FF 1.5+, Safari 2.0+, Opera 9.0+）。
- jQuery是一个快速的，简洁的javaScript库，使用户能更方便地处理HTML documents、events、实现动画效果，并且方便地为网站提供AJAX交互。
- jQuery还有一个比较大的优势是，它的文档说明很全，而且各种应用也说得很详细，同时还有许多成熟的插件可供选择。
- jQuery能够使用户的html页保持代码和html内容分离，也就是说，不用再在html里面插入一堆js来调用命令了，只需定义id即可。

2 什么是jQuery对象？

-  jQuery 对象就是通过jQuery包装DOM对象后产生的对象。
-  jQuery 对象是 jQuery 独有的. 如果一个对象是 jQuery 对象, 那么它就可以使用 jQuery 里的方法: $(“#test”).html();

比如:
```javascript
$("#test").html()   意思是指：获取ID为test的元素内的html代码。其中html()是jQuery里的方法
```
这段代码等同于用DOM实现代码： document.getElementById(" test ").innerHTML;

虽然jQuery对象是包装DOM对象后产生的，但是jQuery无法使用DOM对象的任何方法，同理DOM对象也不能使用jQuery里的方法.乱使用会报错

约定：如果获取的是 jQuery 对象, 那么要在变量前面加上$.

var $variable = jQuery 对象

var variable = DOM 对象

## 2. 选择器和筛选器
### 2.1.   选择器

#### 2.1.1 基本选择器
- $("*")  //选择所有元素
- $("#id")  //选择ID元素
- $(".class")  //选择class名
- $("element")  //选择标签名
- $(".class,p,div")  //多个选择同时选中

#### 2.1.2 层级选择器
- $(".outer div")  //.outer选中下的子孙辈的div标签
- $(".outer>div")  //.outber选中下的儿子标签中的div标签
- $(".outer+div")  //.outer选中下的毗邻的div标签===>(".outer").next("div")等价
- $(".outer~div")  //.outer选中下的下面兄弟中的div标签===>$(".outer").nextAll("div")等价

#### 2.1.3 基本筛选器
- $("li:first")  //找到的所有li标签中的第一个标签
- $("li:eq(2)")  //找到的所有li标签中的第二个标签
- $("li:even")  //找到的所有li标签中计数为偶数的标签
- $("li:gt(1)")  //找到的所有li标签中计数大于1的标签

#### 2.1.4 属性选择器
- $('[id="div1"]')  //标签中的id属性等于div1的标签
- $('["alex="sb"][id]') //标签中的alex属性等于sb的标签中含有id属性的标签

#### 2.1.5 表单选择器
- $("[type='text']")----->$(":text")                    注意只适用于input标签 $("input:checked")

### 2.2. 筛选器

#### 2.2.1. 过滤筛选器
- $("li").eq(2)  //找到的所有li标签的第二个li标签
- $("li").first()  //找到的所有li标签的第一个li标签
- $("ul li").hasclass("test")  //找到的标签中的class中含有test的标签

#### 2.2.2. 查找筛选器

- $("div").children(".test")  //div标签中的儿子辈标签中class属性中有test的标签
- $("div").find(".test")  //div标签中的子孙辈标签中class属性中有test的标签
- 
- $(".test").next()  //找到的标签的下一个标签
- $(".test").nextAll()  //找到的标签的下面所有兄弟标签
- $(".test").nextUntil()  //找到的标签的下面直到util标签为止的所有标签，util里面的标签不包含
- 
- $("div").prev()  //找到的标签的上一个标签
- $("div").prevAll()  //找到的标签的上面所有兄弟
- $("div").prevUntil()  //找到的标签的上面直到util标签为止的所有标签，util里面的标签不包含
- 
- $(".test").parent()  //找到的标签的父类标签
- $(".test").parents()  //找到的标签的祖宗标签，一直找到根部
- $(".test").parentUntil()  //找到的标签的祖宗标签，直到某个祖宗为止，util里面的标签不包含
- 
- $("div").siblings()  //找到的标签的所有相邻同级标签（兄弟标签），不包含自己

### 2.3. 左侧菜单实现
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>left_menu</title>

    <script src="js/jquery-2.2.3.js"></script>
    <script>
          function show(self){
//              console.log($(self).text())
              $(self).next().removeClass("hide")
              $(self).parent().siblings().children(".con").addClass("hide")

          }
    </script>
    <style>
          .menu{
              height: 500px;
              width: 30%;
              background-color: gainsboro;
              float: left;
          }
          .content{
              height: 500px;
              width: 70%;
              background-color: rebeccapurple;
              float: left;
          }
         .title{
             line-height: 50px;
             background-color: #425a66;
             color: forestgreen;}


         .hide{
             display: none;
         }


    </style>
</head>
<body>

<div class="outer">
    <div class="menu">
        <div class="item">
            <div class="title" onclick="show(this);">菜单一</div>
            <div class="con">
                <div>111</div>
                <div>111</div>
                <div>111</div>
            </div>
        </div>
        <div class="item">
            <div class="title" onclick="show(this);">菜单二</div>
            <div class="con hide">
                <div>111</div>
                <div>111</div>
                <div>111</div>
            </div>
        </div>
        <div class="item">
            <div class="title" onclick="show(this);">菜单三</div>
            <div class="con hide">
                <div>111</div>
                <div>111</div>
                <div>111</div>
            </div>
        </div>

    </div>
    <div class="content"></div>

</div>

</body>
</html>
```

### 2.4. 实例tab切换
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>tab</title>
    <script src="js/jquery-2.2.3.js"></script>
    <script>
           function tab(self){
               var index=$(self).attr("xxx")
               $("#"+index).removeClass("hide").siblings().addClass("hide")
               $(self).addClass("current").siblings().removeClass("current")

           }
    </script>
    <style>
        *{
            margin: 0px;
            padding: 0px;
        }
        .tab_outer{
            margin: 0px auto;
            width: 60%;
        }
        .menu{
            background-color: #cccccc;
            /*border: 1px solid red;*/
            line-height: 40px;
        }
        .menu li{
            display: inline-block;
        }
        .menu a{
            border-right: 1px solid red;
            padding: 11px;
        }
        .content{
            background-color: tan;
            border: 1px solid green;
            height: 300px;
        }
        .hide{
            display: none;
        }

        .current{
            background-color: darkgray;
            color: yellow;
            border-top: solid 2px rebeccapurple;
        }
    </style>
</head>
<body>
      <div class="tab_outer">
          <ul class="menu">
              <li xxx="c1" class="current" onclick="tab(this);">菜单一</li>
              <li xxx="c2" onclick="tab(this);">菜单二</li>
              <li xxx="c3" onclick="tab(this);">菜单三</li>
          </ul>
          <div class="content">
              <div id="c1">内容一</div>
              <div id="c2" class="hide">内容二</div>
              <div id="c3" class="hide">内容三</div>
          </div>

      </div>
</body>
</html>
```

## 3. 操作元素
### 3.1 属性操作
- $("p").text()  //p标签的内容text字符串
- $("p").html()  //p标签的内部html内容，和innerhtml一样
- $(":checkbox").val()  //拿到checkbox的value
- $(".test").attr("alex")  //拿到class中有test的标签的属性alex的值
- $(".test").attr("alex","sb")  //给class中有test的标签设置属性alex的值为sb。。。
- $(".test").attr("checked","checked")  //给class中有test的标签设置checked，打上勾
- $(":checkbox").removeAttr("checked")  //移除打上的勾
- $(".test").prop("checked",true)  //给class中含有test的标签的checked属性设置为true，打上勾
- $(".test").addClass("hide")  //添加class名为hide

### 3.2 选择框正反选
```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="js/jquery-2.2.3.js"></script>
    <script>

             function selectall(){

                 $("table :checkbox").prop("checked",true)
             }
             function che(){

                 $("table :checkbox").prop("checked",false)
             }

             function reverse(){


//                 var idlist=$("table :checkbox")
//                 for(var i=0;i<idlist.length;i++){
//                     var element=idlist[i];
//
//                     var ischecked=$(element).prop("checked")
//                     if (ischecked){
//                         $(element).prop("checked",false)
//                     }
//                     else {
//                         $(element).prop("checked",true)
//                     }
//
//                 }



                 $("table :checkbox").each(function(){
                     if ($(this).prop("checked")){
                         $(this).prop("checked",false)
                     }
                     else {
                         $(this).prop("checked",true)
                     }

                 });



//                 li=[10,20,30,40]
////                 dic={name:"yuan",sex:"male"}
//                 $.each(li,function(i,x){
//                     console.log(i,x)
//                 })

             }

    </script>
</head>
<body>

     <button onclick="selectall();">全选</button>
     <button onclick="che();">取消</button>
     <button onclick="reverse();">反选</button>

     <table border="1">
         <tr>
             <td><input type="checkbox"></td>
             <td>111</td>
         </tr>
         <tr>
             <td><input type="checkbox"></td>
             <td>222</td>
         </tr>
         <tr>
             <td><input type="checkbox"></td>
             <td>333</td>
         </tr>
         <tr>
             <td><input type="checkbox"></td>
             <td>444</td>
         </tr>
     </table>



</body>
</html>
```

### 3.3 模态对话框
```javascript
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8" />
        <title>批量编辑</title>
        <!--<link rel="stylesheet" href="css/mycss.css" />-->
        <style>
            *{
    margin: 0;
    padding: 0;
}
body {
  font-family: 'Open Sans', sans-serif;
  font-weight: 300;
  line-height: 1.42em;
  color:rebeccapurple;
  background-color:goldenrod;
}

h1 {
  font-size:3em;
  font-weight: 300;
  line-height:1em;
  text-align: center;
  color: #4DC3FA;
}
.blue {
    color: #185875;
}
.yellow {
    color: #FFF842;
    }

/*!*弹出层罩子*!*/
#full {
    background-color:gray;
    left:0;
    opacity:0.7;
    position:absolute;
    top:0;
    filter:alpha(opacity=30);
}

.modified {
    background-color: #1F2739;
    border:3px solid whitesmoke;
    height:400px;
    left:50%;
    margin:-200px 0 0 -200px;
    padding:1px;
    position:fixed;
    /*position:absolute;*/
    top:50%;
    width:400px;
    display:none;
}
li {
    list-style: none;
    margin: 20px 0 0 50px;
    color: #FB667A;
}
input[type="text"] {
  padding: 10px;
  border: solid 1px #dcdcdc;
  /*transition: box-shadow 3s, border 3s;*/

}

.iputbutton {
    margin: 60px 0 0 50px;
    color: whitesmoke;
    background-color: #FB667A;
    height: 30px;
    width: 100px;
    border: 1px dashed;

}




.container th h1 {
    font-weight: bold;
    font-size: 1em;
      text-align: left;
      color: #185875;
}

.container td {
    font-weight: normal;
    font-size: 1em;
}

.container {

    width: 80%;
    margin: 0 auto;

}

.container td, .container th {
    padding-bottom: 2%;
    padding-top: 2%;
      padding-left:2%;
}

/*单数行*/
.container tr:first-child {
    background-color: red;
}

/*偶数行*/
.container tr:not(first-child){
      background-color: cyan;
}

.container th {
    background-color: #1F2739;
}

.container td:last-child {
    color: #FB667A;
}
/*鼠标进过行*/
.container tr:hover {
   background-color: #464A52;
}
/*鼠标进过列*/
.container td:hover {
  background-color: #FB667A;
  color: #403E10;
  font-weight: bold;
  transform: translate3d(5px, -5px, 0);
}





        </style>
        <script src="jquery-2.2.3.js"></script>
        <script>
            //点击【编辑】显示

$(function () {


    $("table[name=host] tr:gt(0) td:last-child").click(function (event) {

        alert("234");
//        $("#full").css({height:"100%",width:"100%"});

        $(".modified").data('current-edit-obj', $(this));

        $(".modified,#full").show();

        var hostname = $(this).siblings("td[name=hostname]").text();
        $(".modified #hostname").val(hostname);
        var ip = $(this).siblings("td[name=ip]").text();
        $(".modified #ip").val(ip);
        var port = $(this).siblings("td[name=port]").text();
        $(".modified #port").val(port);
    });
    //取消编辑
    $(".modified #cancel").bind("click", function () {
        $(".modified,#full").hide();
    });

//    确定修改
    $(".modified #ok").bind("click", function (event) {
        var check_status = true;
        var ths = $(".modified").data('current-edit-obj');
        var hostname = $(".modified #hostname").val().trim();
        var ip = $(".modified #ip").val().trim();
        var port = $(".modified #port").val().trim();
        var port = parseInt(port);
        console.log(port);
        //        端口为空设置为22
        if (isNaN(port)) {
            alert("您没有设置正常的SSH端口号，将采用默认22号端口");
            var port = 22;
        }else if ( port > 65535) {
        //            如果端口不是一个数字 或者端口大于65535
            var check_status = false;
            $(".modified #port").css("border-color","red");
            alert("端口号超过范围!")
        };
        // 主机和ip不能是空
        if ( hostname == ""){
            var check_status = false;
            $(".modified #hostname").css("border-color","red");
        }else if (ip == "") {
            var check_status = false;
            $(".modified #ip").css("border-color","red");
        };
        if (check_status == false){
            return false;
        }else{
            //$(this).css("height","60px")   为什么不用$(this),而用$()
            $(ths).siblings("td[name=hostname]").text(hostname);
            $(ths).siblings("td[name=ip]").text(ip);
            $(ths).siblings("td[name=port]").text(port);
            $(".modified,#full").hide();
        };

    });

});
            
        </script>
    </head>
    <body>
        <h1>
            <span class="blue">&lt;</span>Homework1<span class="blue">&gt;</span> <span class="yellow">HostList</span>
        </h1>
        <div id="full">
            <div class="modified">
                <li>主机名：<input id="hostname" type="text" value="" />*</li>
                <li>ip地址：<input id="ip" type="text" value="" />*</li>
                <li>端口号：<input id="port" type="text" value="" />[0-65535]</li>
                    <div id="useraction">
                     <input class="iputbutton" type="button" name="确定" id="ok" value="确定"/>
                    <input class="iputbutton" type="button" name="取消" id="cancel" value="取消"/>
                    </div>
            </div>
        </div>
        <table class="container" name="host">
            <tr>
                <th><h1>主机名</h1></th>
                <th><h1>IP地址</h1></th>
                <th><h1>端口</h1></th>
                <th><h1>操作</h1></th>

            </tr>
            <tr>
                <td name="hostname">web01</td>
                <td name="ip">192.168.2.1</td>
                <td name="port">22</td>
                <td>编辑 </td>
            </tr>
            <tr>
                <td name="hostname">web02</td>
                <td name="ip">192.168.2.2</td>
                <td name="port">223</td>
                <td>编辑 </td>
            </tr>
            <tr>
                <td name="hostname">web03</td>
                <td name="ip">192.168.2.3</td>
                <td name="port">232</td>
                <td>编辑 </td>
            </tr>
            <tr>
                <td name="hostname">web04</td>
                <td name="ip">192.168.2.4</td>
                <td name="port">232</td>
                <td>编辑 </td>
            </tr>
        </table>


    </body>
</html>
```
### 3.4 CSS操作
- (样式)   css("{color:'red',backgroud:'blue'}")
- (位置)   offset()    position()  scrollTop()  scrollLeft()
- (尺寸)   height()  width()

### 3.5 返回顶部
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="js/jquery-2.2.3.js"></script>
    <script>


          window.onscroll=function(){

              var current=$(window).scrollTop();
              console.log(current)
              if (current>100){

                  $(".returnTop").removeClass("hide")
              }
              else {
              $(".returnTop").addClass("hide")
          }
          }


           function returnTop(){
//               $(".div1").scrollTop(0);

               $(window).scrollTop(0)
           }




    </script>
    <style>
        body{
            margin: 0px;
        }
        .returnTop{
            height: 60px;
            width: 100px;
            background-color: darkgray;
            position: fixed;
            right: 0;
            bottom: 0;
            color: greenyellow;
            line-height: 60px;
            text-align: center;
        }
        .div1{
            background-color: orchid;
            font-size: 5px;
            overflow: auto;
            width: 500px;
        }
        .div2{
            background-color: darkcyan;
        }
        .div3{
            background-color: aqua;
        }
        .div{
            height: 300px;
        }
        .hide{
            display: none;
        }
    </style>
</head>
<body>
     <div class="div1 div">
         <p>hello</p>
         <p>hello</p>
         <p>hello</p>
         <p>hello</p>
         <p>hello</p>
         <p>hello</p>
         <p>hello</p>
         <p>hello</p>
         <p>hello</p>
         <p>hello</p>
         <p>hello</p>
         <p>hello</p>
         <p>hello</p>
         <p>hello</p>
         <p>hello</p>
         <p>hello</p>
         <p>hello</p>
         <p>hello</p>

     </div>
     <div class="div2 div"></div>
     <div class="div3 div"></div>
     <div class="returnTop hide" onclick="returnTop();">返回顶部</div>
</body>
</html>
```

### 3.6 滚动菜单
```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
    <style>

        body{
            margin: 0px;
        }
        img {
            border: 0;
        }
        ul{
            padding: 0;
            margin: 0;
            list-style: none;
        }
        .clearfix:after {
            content: ".";
            display: block;
            height: 0;
            clear: both;
            visibility: hidden;
        }

        .wrap{
            width: 980px;
            margin: 0 auto;
        }

        .pg-header{
            background-color: #303a40;
            -webkit-box-shadow: 0 2px 5px rgba(0,0,0,.2);
            -moz-box-shadow: 0 2px 5px rgba(0,0,0,.2);
            box-shadow: 0 2px 5px rgba(0,0,0,.2);
        }
        .pg-header .logo{
            float: left;
            padding:5px 10px 5px 0px;
        }
        .pg-header .logo img{
            vertical-align: middle;
            width: 110px;
            height: 40px;

        }
        .pg-header .nav{
            line-height: 50px;
        }
        .pg-header .nav ul li{
            float: left;
        }
        .pg-header .nav ul li a{
            display: block;
            color: #ccc;
            padding: 0 20px;
            text-decoration: none;
            font-size: 14px;
        }
        .pg-header .nav ul li a:hover{
            color: #fff;
            background-color: #425a66;
        }
        .pg-body{

        }
        .pg-body .catalog{
            position: absolute;
            top:60px;
            width: 200px;
            background-color: #fafafa;
            bottom: 0px;
        }
        .pg-body .catalog.fixed{
            position: fixed;
            top:10px;
        }

        .pg-body .catalog .catalog-item.active{
            color: #fff;
            background-color: #425a66;
        }

        .pg-body .content{
            position: absolute;
            top:60px;
            width: 700px;
            margin-left: 210px;
            background-color: #fafafa;
            overflow: auto;
        }
        .pg-body .content .section{
            height: 500px;
        }
    </style>
</head>
<body>

    <div class="pg-header">
        <div class="wrap clearfix">
            <div class="logo">
                <a href="#">
                    <img src="images/3.jpg">
                </a>
            </div>
            <div class="nav">
                <ul>
                    <li>
                        <a  href="#">首页</a>
                    </li>
                    <li>
                        <a  href="#">功能一</a>
                    </li>
                    <li>
                        <a  href="#">功能二</a>
                    </li>
                </ul>
            </div>

        </div>
    </div>
    <div class="pg-body">
        <div class="wrap">
            <div class="catalog">
                <div class="catalog-item" auto-to="function1"><a>第1张</a></div>
                <div class="catalog-item" auto-to="function2"><a>第2张</a></div>
                <div class="catalog-item" auto-to="function3"><a>第3张</a></div>
            </div>
            <div class="content">

                <div menu="function1" class="section">
                    <h1>第一章</h1>
                </div>
                <div menu="function2" class="section">
                    <h1>第二章</h1>
                </div>
                <div menu="function3" class="section">
                    <h1>第三章</h1>
                </div>
            </div>
        </div>
    </div>

    <script type="text/javascript" src="js/jquery-2.2.3.js"></script>
    <script type="text/javascript">


     window.onscroll=function(){
         var ws=$(window).scrollTop()
         if (ws>50){
             $(".catalog").addClass("fixed")
         }
         else {
             $(".catalog").removeClass("fixed")
         }
         $(".content").children("").each(function(){

          var offtop=$(this).offset().top;
//             console.log(offtop)
             var total=$(this).height()+offtop;

             if (ws>offtop && ws<total){

                 if($(window).height()+$(window).scrollTop()==$(document).height()){
                     var index=$(".catalog").children(" :last").css("fontSize","40px").siblings().css("fontSize","12px")
                     console.log(index)
                 target='div[auto-to="'+index+'"]';
                 $(".catalog").children(target).css("fontSize","40px").siblings().css("fontSize","12px")

                 }
                 else{
                      var index=$(this).attr("menu");
                 target='div[auto-to="'+index+'"]';
                 $(".catalog").children(target).css("fontSize","40px").siblings().css("fontSize","12px")
                 }


             }

         })

     }

    </script>


</body>
</html>
```

### 3.7 文档处理
内部插入
- $("#c1").append("<b>hello</b>")  //在选中元素的子元素中追加一个元素
- $("p").appendTo("div")  //将一个元素追加到选中元素的子元素中
- prepend()  //在选中元素的子元素中第一个位置插入一个元素
- prependTo()  //将某个元素插入到选中元素中的第一个子元素位置

外部插入
- before()  //在选中元素上面插入某个元素
- insertBefore()  //将某个元素插入到选中元素的上面
- after() //在选中元素下面插入某个元素
- insertAfter()  //将某个元素插入到选中元素的下面
- replaceWith()  //将选中的标签替换成新的dom标签
- remove()  //删除一个标签（彻底删除）
- empty()  //清空某个标签下的所有子标签
- clone()  //克隆某个标签，括号中有个参数设置为true时代表将标签绑定的事件也一起克隆

### 3.8 clone方法的应用
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

</head>
<body>
            <div class="outer">
                <div class="condition">

                        <div class="icons" style="display:inline-block">
                            <a onclick="add(this);"><button>+</button></a>
                        </div>

                        <div class="input" style="display:inline-block">
                            <input type="checkbox">
                            <input type="text" value="alex">
                        </div>
                </div>
            </div>

<script src="js/jquery-2.2.3.js"></script>
    <script>

            function add(self){
                var $duplicate = $(self).parent().parent().clone();
                $duplicate.find('a[onclick="add(this);"]').attr('onclick',"removed(this)").children("").text("-");
                $duplicate.appendTo($(self).parent().parent().parent());

            }
           function removed(self){

               $(self).parent().parent().remove()

           }

    </script>
</body>
</html>
```

### 3.9 事件

- $(document).ready(function(){}) -----------> $(function(){})  //文档加载完成
- $("p").click(function(){})  //单击事件
- $("p").bind("click",function(){})  //绑定事件（将要被废弃，已经有on事件可以替代）
- $("ul").delegate("li","click",function(){})  //jquery2版本中用它动态绑定子元素事件，在jquery3中用on替代
- on(events,[selector],[data],fn) //可以动态绑定子元素的某个事件，当插入一个子元素时，该事件会被绑定上

## 4 示例
### 4.1 拖动面板
```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
    <div style="border: 1px solid #ddd;width: 600px;position: absolute;">
        <div id="title" style="background-color: black;height: 40px;color: white;">
            标题
        </div>
        <div style="height: 300px;">
            内容
        </div>
    </div>
<script type="text/javascript" src="jquery-2.2.3.js"></script>
<script>
    $(function(){
        // 页面加载完成之后自动执行
        $('#title').mouseover(function(){
            $(this).css('cursor','move');
        }).mousedown(function(e){
            //console.log($(this).offset());
            var _event = e || window.event;
            // 原始鼠标横纵坐标位置
            var ord_x = _event.clientX;
            var ord_y = _event.clientY;

            var parent_left = $(this).parent().offset().left;
            var parent_top = $(this).parent().offset().top;

            $(this).bind('mousemove', function(e){
                var _new_event = e || window.event;
                var new_x = _new_event.clientX;
                var new_y = _new_event.clientY;

                var x = parent_left + (new_x - ord_x);
                var y = parent_top + (new_y - ord_y);

                $(this).parent().css('left',x+'px');
                $(this).parent().css('top',y+'px');

            })
        }).mouseup(function(){
            $(this).unbind('mousemove');
        });
    })
</script>
</body>
</html>
```
### 4.2 放大镜
```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>bigger</title>
    <style>
        *{
            margin: 0;
            padding:0;
        }
        .outer{
            height: 350px;
            width: 350px;
            border: dashed 5px cornflowerblue;
        }
        .outer .small_box{
            position: relative;
        }
        .outer .small_box .float{
            height: 175px;
            width: 175px;
            background-color: darkgray;
            opacity: 0.4;
            fill-opacity: 0.4;
            position: absolute;
            display: none;

        }

        .outer .big_box{
            height: 400px;
            width: 400px;
            overflow: hidden;
            position:absolute;
            left: 360px;
            top: 0px;
            z-index: 1;
            border: 5px solid rebeccapurple;
            display: none;


        }
        .outer .big_box img{
            position: absolute;
            z-index: 5;
        }


    </style>
</head>
<body>


        <div class="outer">
            <div class="small_box">
                <div class="float"></div>
                <img src="small.jpg">

            </div>
            <div class="big_box">
                <img src="big.jpg">
            </div>

        </div>


<script src="js/jquery-2.2.3.js"></script>
<script>

    $(function(){

          $(".small_box").mouseover(function(){

              $(".float").css("display","block");
              $(".big_box").css("display","block")

          })
          $(".small_box").mouseout(function(){

              $(".float").css("display","none");
              $(".big_box").css("display","none")

          })



          $(".small_box").mousemove(function(e){

              var _event=e || window.event;

              var float_width=$(".float").width();
              var float_height=$(".float").height();

              console.log(float_height,float_width);

              var float_height_half=float_height/2;
              var float_width_half=float_width/2;
              console.log(float_height_half,float_width_half);


               var small_box_width=$(".small_box").height();
               var small_box_height=$(".small_box").width();



//  鼠标点距离左边界的长度与float应该与左边界的距离差半个float的width,height同理
              var mouse_left=_event.clientX-float_width_half;
              var mouse_top=_event.clientY-float_height_half;

              if(mouse_left<0){
                  mouse_left=0
              }else if (mouse_left>small_box_width-float_width){
                  mouse_left=small_box_width-float_width
              }


              if(mouse_top<0){
                  mouse_top=0
              }else if (mouse_top>small_box_height-float_height){
                  mouse_top=small_box_height-float_height
              }

               $(".float").css("left",mouse_left+"px");
               $(".float").css("top",mouse_top+"px")

               var percentX=($(".big_box img").width()-$(".big_box").width())/ (small_box_width-float_width);
               var percentY=($(".big_box img").height()-$(".big_box").height())/(small_box_height-float_height);

              console.log(percentX,percentY)

               $(".big_box img").css("left", -percentX*mouse_left+"px")
               $(".big_box img").css("top", -percentY*mouse_top+"px")


          })


    })


</script>
</body>
</html>
```
### 4.3 动画效果
隐藏与显示
```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="js/jquery-2.2.3.js"></script>
    <script>
    /**
 * Created by yuan on 16/5/5.
 */

$(document).ready(function(){
    $("#hide").click(function(){
        $("p").hide(1000);
    });
    $("#show").click(function(){
        $("p").show(1000);
    });

//用于切换被选元素的 hide() 与 show() 方法。
    $("#toggle").click(function(){
        $("p").toggle();
    })

 for (var i= 0;i<7;i++){
//            颠倒了常规的$(A).append(B)的操作，即不是把B追加到A中，而是把A追加到B中。
            $("<div>").appendTo(document.body);
        }
        $("div").click(function(){
          $(this).hide(2000);
        });
});

    </script>
    <link type="text/css" rel="stylesheet" href="style.css">
</head>
<body>
    <!--1 隐藏与显示-->
    <!--2 淡入淡出-->
    <!--3 滑动-->
    <!--4 效果-回调:每一个动画执行完毕之后所能执行的函数方法或者所能做的一件事-->


    <p>hello</p>
    <button id="hide">隐藏</button>
    <button id="show">显示</button>
    <button id="toggle">隐藏/显示</button>

</body>
</html>
```

淡入淡出
```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="js/jquery-2.2.3.js"></script>
    <script>
    $(document).ready(function(){
   $("#in").click(function(){
       $("#id1").fadeIn(1000);
       $("#id2").fadeIn(1000);
       $("#id3").fadeIn(1000);

   });
    $("#out").click(function(){
       $("#id1").fadeOut(1000);
       $("#id2").fadeOut(1000);
       $("#id3").fadeOut(1000);

   });
    $("#toggle").click(function(){
       $("#id1").fadeToggle(1000);
       $("#id2").fadeToggle(1000);
       $("#id3").fadeToggle(1000);

   });
    $("#fadeto").click(function(){
       $("#id1").fadeTo(1000,0.4);
       $("#id2").fadeTo(1000,0.5);
       $("#id3").fadeTo(1000,0);

   });
});

    
    
    </script>

</head>
<body>
      <button id="in">fadein</button>
      <button id="out">fadeout</button>
      <button id="toggle">fadetoggle</button>
      <button id="fadeto">fadeto</button>

      <div id="id1" style="display:none; width: 80px;height: 80px;background-color: blueviolet"></div>
      <div id="id2" style="display:none; width: 80px;height: 80px;background-color: orangered "></div>
      <div id="id3" style="display:none; width: 80px;height: 80px;background-color: darkgreen "></div>

</body>
</html>
```

滑动
```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="js/jquery-2.2.3.js"></script>
    <script>
    $(document).ready(function(){
     $("#flipshow").click(function(){
         $("#content").slideDown(1000);
     });
      $("#fliphide").click(function(){
         $("#content").slideUp(1000);
     });
      $("#toggle").click(function(){
         $("#content").slideToggle(1000);
     })
  });
    </script>
    <style>
        #flipshow,#content,#fliphide,#toggle{
            padding: 5px;
            text-align: center;
            background-color: blueviolet;
            border:solid 1px red;

        }
        #content{
            padding: 50px;
            display: none;
        }
    </style>
</head>
<body>

    <div id="flipshow">出现</div>
    <div id="fliphide">隐藏</div>
    <div id="toggle">toggle</div>

    <div id="content">helloworld</div>

</body>
</html>
```

回调函数

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="js/jquery-2.2.3.js"></script>
    <script>

    $(document).ready(function(){
   $("button").click(function(){
       $("p").hide(1000,function(){
           alert('动画结束')
       })

//       $("p").css('color','red').slideUp(1000).slideDown(2000)
   })
});
    </script>
</head>
<body>
  <button>隐藏</button>
  <p>helloworld helloworld helloworld</p>

</body>
</html>
```
京东轮播图
```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="js/jquery-2.2.3.js"></script>
    <script src="index.js"></script>
    <script>
        /**
 * Created by yuan on 2016/5/6.
 */

//手动轮播效果
$(function(){
    var size=$(".img li").size()
    for (var i= 1;i<=size;i++){
        var li="<li>"+i+"</li>"
        $(".num").append(li);
    }


    //$(".img li").eq(0).show();
    $(".num li").eq(0).addClass("active");
    $(".num li").mouseover(function(){
       $(this).addClass("active").siblings().removeClass("active");
       var index=$(this).index();
       i=index;
       $(".img li").eq(index).fadeIn(1000).siblings().fadeOut(1000);
   });


i=0;
var t=setInterval(move,1500)

    function move(){
    i++;
    if(i==size){
        i=0;
    }
    $(".num li").eq(i).addClass("active").siblings().removeClass("active");
    $(".img li").eq(i).stop().fadeIn(1000).siblings().stop().fadeOut(1000);
};

    function moveL(){
    i--;
    if(i==-1){
        i=size-1;
    }
    $(".num li").eq(i).addClass("active").siblings().removeClass("active");
    $(".img li").eq(i).stop().fadeIn(1000).siblings().stop().fadeOut(1000);
}

    $(".out").hover(function(){
        clearInterval(t);
    },function(){
        t=setInterval(move,1500);
    });

    $(".out .right").click(function(){
        move()
    });
    $(".out .left").click(function(){
       moveL()
    })

});
    </script>
    <link type="text/css" rel="stylesheet" href="style.css">
    <style>
        *{
    margin: 0;
    padding: 0;
}
ul{
    list-style: none;
}

.out{
    width: 730px;
    height: 454px;
    /*border: 8px solid mediumvioletred;*/
    margin: 50px auto;
    position: relative;
}

.out .img li{
    position: absolute;
    left: 0;
    top: 0;
}

.out .num{
    position: absolute;
    left:0;
    bottom: 20px;
    text-align: center;
    font-size: 0;
    width: 100%;
}


.out .btn{
    position: absolute;
    top:50%;
    margin-top: -30px;
    width: 30px;
    height: 60px;
    background-color: aliceblue;
    color: black;
    text-align: center;
    line-height: 60px;
    font-size: 40px;
    display: none;





}

.out .num li{
    width: 20px;
    height: 20px;
    background-color: grey;
    color: #fff;
    text-align: center;
    line-height: 20px;
    border-radius: 50%;
    display: inline;
    font-size: 18px;
    margin: 0 10px;
    cursor: pointer;
}
.out .num li.active{
    background-color: red;
}


.out .left{
    left: 0;
}
.out .right{
    right: 0;
}

.out:hover .btn{
    display: block;
}
    </style>
</head>
<body>
     <div class="out">
         <ul class="img">
             <li><a href="#"><img src="images/1.jpg" alt=""></a></li>
             <li><a href="#"><img src="images/2.jpg" alt=""></a></li>
             <li><a href="#"><img src="images/3.jpg" alt=""></a></li>
             <li><a href="#"><img src="images/4.jpg" alt=""></a></li>
             <li><a href="#"><img src="images/5.jpg" alt=""></a></li>
         </ul>

         <ul class="num">
             <!--<li class="active">1</li>-->
             <!--<li>2</li>-->
             <!--<li>3</li>-->
             <!--<li>4</li>-->
             <!--<li>5</li>-->
         </ul>

         <div class="left btn"><</div>
         <div class="right btn">></div>

     </div>

</body>
</html>
```

### 4.4 	扩展
- jquery.extend({})  //扩展jquery对象的方法
- jquery.fn.extend({})  //扩展jQuery中dom中的标签对象的方法

商城菜单
```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <meta name="viewport" content="width=device-width">
    <meta http-equiv="X-UA-Compatible" content="IE=8">
    <title>购物商城</title>
    <style>

*{
    margin: 0;
    padding: 0;
}
.hide{
    display:none;
}


.header-nav {
    height: 39px;
    background: #c9033b;
}
.header-nav .bg{
    background: #c9033b;
}

.header-nav .nav-allgoods .menuEvent {
    display: block;
    height: 39px;
    line-height: 39px;
    text-decoration: none;
    color: #fff;
    text-align: center;
    font-weight: bold;
    font-family: 微软雅黑;
    color: #fff;
    width: 100px;
}
.header-nav .nav-allgoods .menuEvent .catName {
    height: 39px;
    line-height: 39px;
    font-size: 15px;
}

.header-nav .nav-allmenu a {
    display: inline-block;
    height: 39px;
    vertical-align: top;
    padding: 0 15px;
    text-decoration: none;
    color: #fff;
    float: left;
}

.header-menu a{
    color:#656565;
}

.header-menu .menu-catagory{
    position: absolute;
    background-color: #fff;
    border-left:1px solid #fff;
    height: 316px;
    width: 230px;
     z-index: 4;
     float: left;
}
.header-menu .menu-catagory .catagory {
    border-left:4px solid #fff;
    height: 104px;
    border-bottom: solid 1px #eaeaea;
}
.header-menu .menu-catagory .catagory:hover {
    height: 102px;
    border-left:4px solid #c9033b;
    border-bottom: solid 1px #bcbcbc;
    border-top: solid 1px #bcbcbc;
}

.header-menu .menu-content .item{
    margin-left:230px;
    position:absolute;
    background-color:white;
    height:314px;
    width:500px;
    z-index:4;
    float:left;
    border: solid 1px #bcbcbc;
    border-left:0;
    box-shadow: 1px 1px 5px #999;
}

    </style>
</head>
<body>

<div class="pg-header">

    <div class="header-nav">
        <div class="container narrow bg">
            <div class="nav-allgoods left">
                <a id="all_menu_catagory" href="#" class="menuEvent">
                    <strong class="catName">全部商品分类</strong>
                    <span class="arrow" style="display: inline-block;vertical-align: top;"></span>
                </a>
            </div>
        </div>
    </div>
    <div class="header-menu">
        <div class="container narrow hide">
            <div id="nav_all_menu" class="menu-catagory">
                <div class="catagory" float-content="one">
                    <div class="title">家电</div>
                    <div class="body">
                        <a href="#">空调</a>
                    </div>
                </div>
                <div class="catagory" float-content="two">
                    <div class="title">床上用品</div>
                    <div class="body">
                        <a href="http://www.baidu.com">床单</a>
                    </div>
                </div>
                <div class="catagory" float-content="three">
                    <div class="title">水果</div>
                    <div class="body">
                        <a href="#">橘子</a>
                    </div>
                </div>
            </div>

            <div id="nav_all_content" class="menu-content">
                <div class="item hide" float-id="one">

                    <dl>
                        <dt><a href="#" class="red">厨房用品</a></dt>
                        <dd>
                            <span>| <a href="#" target="_blank" title="勺子">勺子</a> </span>
                        </dd>
                    </dl>
                    <dl>
                        <dt><a href="#" class="red">厨房用品</a></dt>
                        <dd>
                            <span>| <a href="#" target="_blank" title="菜刀">菜刀</a> </span>

                        </dd>
                    </dl>
                    <dl>
                        <dt><a href="#" class="red">厨房用品</a></dt>
                        <dd>
                            <span>| <a href="#">菜板</a> </span>
                        </dd>
                    </dl>
                    <dl>
                        <dt><a href="#" class="red">厨房用品</a></dt>
                        <dd>
                            <span>| <a href="#" target="_blank" title="碗">碗</a> </span>

                        </dd>
                    </dl>

                </div>
                <div class="item hide" float-id="two">
                    <dl>
                        <dt><a href="#" class="red">厨房用品</a></dt>
                        <dd>
                            <span>| <a href="#" target="_blank" title="">角阀</a> </span>

                        </dd>
                    </dl>
                    <dl>
                        <dt><a href="#" class="red">厨房用品</a></dt>
                        <dd>
                            <span>| <a href="#" target="_blank" title="角阀">角阀</a> </span>

                        </dd>
                    </dl>
                    <dl>
                        <dt><a href="#" class="red">厨房用品</a></dt>
                        <dd>
                            <span>| <a href="#" target="_blank" title="角阀">角阀</a> </span>

                        </dd>
                    </dl>

                </div>
                <div class="item hide" float-id="three">
                    <dl>
                        <dt><a href="#" class="red">厨房用品3</a></dt>
                        <dd>
                            <span>| <a href="#" target="_blank" title="角阀">角阀3</a> </span>

                        </dd>
                    </dl>
                    <dl>
                        <dt><a href="#" class="red">厨房用品3</a></dt>
                        <dd>
                            <span>| <a href="http://www.meilele.com/category-jiaofa/" target="_blank" title="角阀">角阀3</a> </span>

                        </dd>
                    </dl>
                </div>
            </div>
        </div>
    </div>

</div>


<script src="js/jquery-2.2.3.js"></script>

<script type="text/javascript">
    $(document).ready(function () {

        Change_Menu('#all_menu_catagory','#nav_all_menu', '#nav_all_content');

    });



    function Change_Menu(all_menu_catagory,menu, content) {
        $all_menu_catagory = $(all_menu_catagory);
        $menu = $(menu);
        $content = $(content);

        $all_menu_catagory.bind("mouseover", function () {
            $menu.parent().removeClass('hide');
        });
        $all_menu_catagory.bind("mouseout", function () {
            $menu.parent().addClass('hide');
        });

        $menu.children().bind("mouseover", function () {
            $menu.parent().removeClass('hide');
            $item_content = $content.find('div[float-id="' + $(this).attr("float-content") + '"]');
            $item_content.removeClass('hide').siblings().addClass('hide');
        });
        $menu.bind("mouseout", function () {
            $content.children().addClass('hide');
            $menu.parent().addClass('hide');
        });
        $content.children().bind("mouseover", function () {

            $menu.parent().removeClass('hide');
            $(this).removeClass('hide');
        });
        $content.children().bind("mouseout", function () {

            $(this).addClass('hide');
            $menu.parent().addClass('hide');
        });
    }
</script>


</body>
</html>
```

编辑框
```javascript
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
    <style>
        .edit-mode{
            background-color: #b3b3b3;
            padding: 8px;
            text-decoration: none;
            color: white;
        }
        .editing{
            background-color: #f0ad4e;
        }
    </style>
</head>
<body>

    <div style="padding: 20px">
        <input type="button" onclick="CheckAll('#edit_mode', '#tb1');" value="全选" />
        <input type="button" onclick="CheckReverse('#edit_mode', '#tb1');" value="反选" />
        <input type="button" onclick="CheckCancel('#edit_mode', '#tb1');" value="取消" />

        <a id="edit_mode" class="edit-mode" href="javascript:void(0);"  onclick="EditMode(this, '#tb1');">进入编辑模式</a>

    </div>
    <table border="1">
        <thead>
        <tr>
            <th>选择</th>
            <th>主机名</th>
            <th>端口</th>
            <th>状态</th>
        </tr>
        </thead>
        <tbody id="tb1">
            <tr>
                <td><input type="checkbox" /></td>
                <td edit="true">v1</td>
                <td>v11</td>
                <td edit="true" edit-type="select" sel-val="1" global-key="STATUS">在线</td>
            </tr>
            <tr>
                <td><input type="checkbox" /></td>
                <td edit="true">v1</td>
                <td>v11</td>
                <td edit="true" edit-type="select" sel-val="2" global-key="STATUS">下线</td>
            </tr>
            <tr>
                <td><input type="checkbox" /></td>
                <td edit="true">v1</td>
                <td>v11</td>
                <td edit="true" edit-type="select" sel-val="1" global-key="STATUS">在线</td>
            </tr>
        </tbody>
    </table>

    <script type="text/javascript" src="js/jquery-2.2.3.js"></script>
    <script>
        /*
         监听是否已经按下control键
         */
        window.globalCtrlKeyPress = false;

        window.onkeydown = function(event){
            if(event && event.keyCode == 17){
                window.globalCtrlKeyPress = true;
            }
        };
        window.onkeyup = function(event){
            if(event && event.keyCode == 17){
                window.globalCtrlKeyPress = false;
            }
        };

        /*
         按下Control，联动表格中正在编辑的select
         */
        function MultiSelect(ths){
            if(window.globalCtrlKeyPress){
                var index = $(ths).parent().index();
                var value = $(ths).val();
                $(ths).parent().parent().nextAll().find("td input[type='checkbox']:checked").each(function(){
                    $(this).parent().parent().children().eq(index).children().val(value);
                });
            }
        }
    </script>
    <script type="text/javascript">

$(function(){
    BindSingleCheck('#edit_mode', '#tb1');
});

function BindSingleCheck(mode, tb){

    $(tb).find(':checkbox').bind('click', function(){
        var $tr = $(this).parent().parent();
        if($(mode).hasClass('editing')){
            if($(this).prop('checked')){
                RowIntoEdit($tr);
            }else{
                RowOutEdit($tr);
            }
        }
    });
}

function CreateSelect(attrs,csses,option_dict,item_key,item_value,current_val){
    var sel= document.createElement('select');
    $.each(attrs,function(k,v){
        $(sel).attr(k,v);
    });
    $.each(csses,function(k,v){
        $(sel).css(k,v);
    });
    $.each(option_dict,function(k,v){
        var opt1=document.createElement('option');
        var sel_text = v[item_value];
        var sel_value = v[item_key];

        if(sel_value==current_val){
            $(opt1).text(sel_text).attr('value', sel_value).attr('text', sel_text).attr('selected',true).appendTo($(sel));
        }else{
            $(opt1).text(sel_text).attr('value', sel_value).attr('text', sel_text).appendTo($(sel));
        }
    });
    return sel;
}

STATUS = [
    {'id': 1, 'value': "在线"},
    {'id': 2, 'value': "下线"}
];

BUSINESS = [
    {'id': 1, 'value': "车上会"},
    {'id': 2, 'value': "二手车"}
];

function RowIntoEdit($tr){
    $tr.children().each(function(){
        if($(this).attr('edit') == "true"){
            if($(this).attr('edit-type') == "select"){
                var select_val = $(this).attr('sel-val');
                var global_key = $(this).attr('global-key');
                var selelct_tag = CreateSelect({"onchange": "MultiSelect(this);"}, {}, window[global_key], 'id', 'value', select_val);
                $(this).html(selelct_tag);
            }else{
                var orgin_value = $(this).text();
                var temp = "<input value='"+ orgin_value+"' />";
                $(this).html(temp);
            }

        }
    });
}

function RowOutEdit($tr){
    $tr.children().each(function(){
        if($(this).attr('edit') == "true"){
            if($(this).attr('edit-type') == "select"){
                var new_val = $(this).children(':first').val();
                var new_text = $(this).children(':first').find("option[value='"+new_val+"']").text();
                $(this).attr('sel-val', new_val);
                $(this).text(new_text);
            }else{
                var orgin_value = $(this).children().first().val();
                $(this).text(orgin_value);
            }

        }
    });
}

function CheckAll(mode, tb){
    if($(mode).hasClass('editing')){

        $(tb).children().each(function(){

            var tr = $(this);
            var check_box = tr.children().first().find(':checkbox');
            if(check_box.prop('checked')){

            }else{
                check_box.prop('checked',true);

                RowIntoEdit(tr);
            }
        });

    }else{

        $(tb).find(':checkbox').prop('checked', true);
    }
}

function CheckReverse(mode, tb){

    if($(mode).hasClass('editing')){

        $(tb).children().each(function(){
            var tr = $(this);
            var check_box = tr.children().first().find(':checkbox');
            if(check_box.prop('checked')){
                check_box.prop('checked',false);
                RowOutEdit(tr);
            }else{
                check_box.prop('checked',true);
                RowIntoEdit(tr);
            }
        });


    }else{
        //
        $(tb).children().each(function(){
            var tr = $(this);
            var check_box = tr.children().first().find(':checkbox');
            if(check_box.prop('checked')){
                check_box.prop('checked',false);
            }else{
                check_box.prop('checked',true);
            }
        });
    }
}

function CheckCancel(mode, tb){
    if($(mode).hasClass('editing')){
        $(tb).children().each(function(){
            var tr = $(this);
            var check_box = tr.children().first().find(':checkbox');
            if(check_box.prop('checked')){
                check_box.prop('checked',false);
                RowOutEdit(tr);

            }else{

            }
        });

    }else{
        $(tb).find(':checkbox').prop('checked', false);
    }
}

function EditMode(ths, tb){
    if($(ths).hasClass('editing')){
        $(ths).removeClass('editing');
        $(tb).children().each(function(){
            var tr = $(this);
            var check_box = tr.children().first().find(':checkbox');
            if(check_box.prop('checked')){
                RowOutEdit(tr);
            }else{

            }
        });

    }else{

        $(ths).addClass('editing');
        $(tb).children().each(function(){
            var tr = $(this);
            var check_box = tr.children().first().find(':checkbox');
            if(check_box.prop('checked')){
                RowIntoEdit(tr);
            }else{

            }
        });

    }
}


    </script>

</body>
</html>
```