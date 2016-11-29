# 前端之float的几种清除浮动方式

## 本节内容
- 1.float清除方式1
- 2.float清除方式2
- 3.float清除方式3
- 4.float清除方式4

## 1.float清除方式1
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .content{
            background: red;
        }
        .left{
            float: left;
            height: 200px;
            width: 200px;
            background: black;
            border: 1px solid red;
        }
    </style>
</head>
<body>
    <div class="content">
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
    </div>
    <div style="clear: both">asdf</div>
</body>
</html>
```
在父div的下面div中设置clear:both方式，content的div没有被撑起来，高度还是0，但是下面的asdf排在了最下面，缺点是外部的div高度为0，则外部div的颜色属性无法展示出来。

## 2.float清除方式2
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .content{
            background: red;
        }
        .left{
            float: left;
            height: 200px;
            width: 200px;
            background: black;
            border: 1px solid red;
        }
    </style>
</head>
<body>
    <div class="content">
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div style="clear: both"></div>
    </div>
    <div>asdf</div>
</body>
</html>
```
在所有浮动标签的最后一个标签后面加上一个空标签，里面的style设置为clear:both，这样就把父div撑起来了。但是如果想在父div中添加新的浮动标签将可能出现问题，因为append新元素进去之后，空标签可能就不在最后一个位置了。

## 3.float清除方式3
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .content{
            background: red;
        }
        .left{
            float: left;
            height: 200px;
            width: 200px;
            background: black;
            border: 1px solid red;
        }

    </style>
</head>
<body>
    <div class="content" style="overflow: hidden">
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
    </div>
    <div>asdf</div>
</body>
</html>
```
在父标签中设置style为overflow: hidden也能将父标签撑起来，hidden的含义是超出的部分要裁切隐藏，float的元素虽然不在普通流中，但是他是浮动在普通流之上的，可以把普通流元素+浮动元素想象成一个立方体。如果没有明确设定包含容器高度的情况下，它要计算内容的全部高度才能确定在什么位置hidden，这样浮动元素的高度就要被计算进去。这样包含容器就会被撑开，清除浮动。

## 4.float清除方式4
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .content{
            background: red;
        }
        .left{
            float: left;
            height: 200px;
            width: 200px;
            background: black;
            border: 1px solid red;
        }
        .clearfix:after{
            content: ".";  /*设置内容，必须有内容，没有，则无法实现效果*/
            visibility: hidden;  /*将标签隐藏*/
            height:0;  /*设置标签的高度为0*/
            display: block;  /*设置标签为块级标签*/
            clear: both;  /*设置清除float浮动*/
        }
    </style>
</head>
<body>
    <div class="content clearfix">
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
        <div class="left"></div>
    </div>
    <div>asdf</div>
</body>
</html>
```
第四种方式是在父标签上面建一个伪类，设置如上面那样，这样将能够撑起父div标签，并且动态在里面添加float标签将不会影响父标签的撑开。

### 推荐使用这种方式