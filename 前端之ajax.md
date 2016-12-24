# 前端之ajax
## 本节内容
1. ajax介绍
2. 原生js实现ajax
3. jquery实现ajax
4. json
5. 跨域请求

## 1. ajax介绍
AJAX（Asynchronous Javascript And XML）翻译成中文就是“异步Javascript和XML”。即使用Javascript语言与服务器进行异步交互，传输的数据为XML（当然，传输的数据不只是XML）。

AJAX还有一个最大的特点就是，当服务器响应时，不用刷新整个浏览器页面，而是可以局部刷新。这一特点给用户的感受是在不知不觉中完成请求和响应过程。
- 与服务器异步交互；
- 浏览器页面局部刷新；

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

<script type="text/javascript">
window.onload = function() {//当文档加载完毕时执行本函数
    var form = document.getElementById("form1");//获取表单元素对象
    form.onsubmit = function() {//给表单元素添加一个监听，监听表单被提交事件
        var usernameValue = form.username.value;//获取表单中名为username的表单元素值
        if(!usernameValue) {//判断该值是否为空
            var usernameSpan = document.getElementById("usernameSpan");//得到username元素后的<span>元素
            usernameSpan.innerText = "用户名不能为空！";//设置span元素内容！
            return false;//返回false，表示拦截了表单提交动作
        }
        return true;//不拦截表单提交动作
    };
};
</script>
</head>
 <body>
<h1>注册页面</h1>
<form action="" method="post" id="form1">
用户名：<input type="text" name="username"/>
<span id="usernameSpan"></span>
<br/>
密　码：<input type="password" name="password"/>
<span id="passwordSpan"></span>
<br/>
<input type="submit" value="注册"/>

</form>
  </body>
</html>
```

### 同步交互与异步交互
- 同步交互：客户端发出一个请求后，需要等待服务器响应结束后，才能发出第二个请求；
- 异步交互：客户端发出一个请求后，无需等待服务器响应结束，就可以发出第二个请求。

### ajax使用场景
当我们在百度中输入一个“老”字后，会马上出现一个下拉列表！列表中显示的是包含“传”字的4个关键字。

其实这里就使用了AJAX技术！当文件框发生了输入变化时，浏览器会使用AJAX技术向服务器发送一个请求，查询包含“传”字的前10个关键字，然后服务器会把查询到的结果响应给浏览器，最后浏览器把这4个关键字显示在下拉列表中。

- 整个过程中页面没有刷新，只是刷新页面中的局部位置而已！
- 当请求发出后，浏览器还可以进行其他操作，无需等待服务器的响应！

当输入用户名后，把光标移动到其他表单项上时，浏览器会使用AJAX技术向服务器发出请求，服务器会查询名为zhangSan的用户是否存在，最终服务器返回true表示名为lemontree7777777的用户已经存在了，浏览器在得到结果后显示“用户名已被注册！”。

- 整个过程中页面没有刷新，只是局部刷新了；
- 在请求发出后，浏览器不用等待服务器响应结果就可以进行其他操作；

### ajax的优缺点
优点：
- AJAX使用Javascript技术向服务器发送异步请求；
- AJAX无须刷新整个页面；
- 因为服务器响应内容不再是整个页面，而是页面中的局部，所以AJAX性能高；

缺点：
- AJAX并不适合所有场景，很多时候还是要使用同步交互；
- AJAX虽然提高了用户体验，但无形中向服务器发送的请求次数增多了，导致服务器压力增大；
- 因为AJAX是在浏览器中使用Javascript技术完成的，所以还需要处理浏览器兼容性问题；

## 2. 原生js实现ajax

ajax技术的四步操作
- 创建核心对象；
- 使用核心对象打开与服务器的连接；
- 发送请求
- 注册监听，监听服务器响应。

核心对象XMLHTTPRequest提供的方法
- open(请求方式, URL, 是否异步)
- send(请求体)  # 键值对方式{key:value}
- onreadystatechange，指定监听函数，它会在xmlHttp对象的状态发生变化时被调用  # (状态在0-4之间变化，状态一共有四种变化，1-4)
- readyState，当前xmlHttp对象的状态，其中4状态表示服务器响应结束  # (状态从0-4)
- status：服务器响应的状态码，只有服务器响应结束时才有这个东东，200表示响应成功；
- responseText：获取服务器的响应体

### 原生js实现ajax的get请求
#### 准备工作：
```python
def login(request):
    print('hello ajax')
    return render(request,'index.html')
    # return HttpResponse('helloxiaoming')

def ajax_get(request):
    return HttpResponse('helloxiaoming')
```

#### 创建ajax核心对象：
其实AJAX就是在Javascript中多添加了一个对象：XMLHttpRequest对象。所有的异步交互都是使用XMLHttpRequest对象完成的。也就是说，我们只需要学习一个Javascript的新对象即可。

注意，各个浏览器对XMLHttpRequest的支持也是不同的！

- 大多数浏览器都支持DOM2规范，都可以使用：  var xmlHttp = new XMLHttpRequest()来创建对象；
- 但IE有所不同，IE5.5以及更早版本需要：var xmlHttp = new ActiveXObject(“Microsoft.XMLHTTP”)来创建对象；
- 而IE6中需要：var xmlHttp = new ActiveXObject(“Msmxl2.XMLHTTP”)来创建对象；
- 而IE7以及更新版本也支持DOM2规范。

为了处理浏览器兼容问题，给出下面方法来创建XMLHttpRequest对象：
```javascript
function createXMLHttpRequest() {
        var xmlHttp;
        // 适用于大多数浏览器，以及IE7和IE更高版本
        try{
            xmlHttp = new XMLHttpRequest();
        } catch (e) {
            // 适用于IE6
            try {
                xmlHttp = new ActiveXObject("Msxml2.XMLHTTP");
            } catch (e) {
                // 适用于IE5.5，以及IE更早版本
                try{
                    xmlHttp = new ActiveXObject("Microsoft.XMLHTTP");
                } catch (e){}
            }
        }
        return xmlHttp;
    }
```

#### 打开与服务器的连接
当得到XMLHttpRequest对象后，就可以调用该对象的open()方法打开与服务器的连接了。open()方法的参数如下：

open(method, url, async)：
- method：请求方式，通常为GET或POST；
- url：请求的服务器地址，例如：/ajaxdemo1/AServlet，若为GET请求，还可以在URL后追加参数；
- async：这个参数可以不给，默认值为true，表示异步请求；

```javascript
var xmlHttp = createXMLHttpRequest();
xmlHttp.open("GET", "/ajax_get/", true);
```
#### 发送请求
当使用open打开连接后，就可以调用XMLHttpRequest对象的send()方法发送请求了。send()方法的参数为POST请求参数，即对应HTTP协议的请求体内容，若是GET请求，需要在URL后连接参数。

注意：若没有参数，需要给出null为参数！若不给出null为参数，可能会导致FireFox浏览器不能正常发送请求！
```javascript
xmlHttp.send(null);
```
#### 接收服务器响应
当请求发送出去后，服务器端Servlet就开始执行了，但服务器端的响应还没有接收到。接下来我们来接收服务器的响应。

XMLHttpRequest对象有一个onreadystatechange事件，它会在XMLHttpRequest对象的状态发生变化时被调用。下面介绍一下XMLHttpRequest对象的5种状态：

- 0:初始化未完成状态，只是创建了XMLHttpRequest对象，还未调用open()方法；
- 1:请求已开始，open()方法已调用，但还没调用send()方法；
- 2：请求发送完成状态，send()方法已调用；
- 3：开始读取服务器响应；
- 4：读取服务器响应结束。

onreadystatechange事件会在状态为1、2、3、4时引发。

下面代码会被执行四次！对应XMLHttpRequest的四种状态！
```javascript
xmlHttp.onreadystatechange = function() {
			alert('hello');
		};
```
但通常我们只关心最后一种状态，即读取服务器响应结束时，客户端才会做出改变。我们可以通过XMLHttpRequest对象的readyState属性来得到XMLHttpRequest对象的状态。
```javascript
xmlHttp.onreadystatechange = function() {
            if(xmlHttp.readyState == 4) {
                alert('hello');
            }
        };
```

其实我们还要关心服务器响应的状态码是否为200，其服务器响应为404，或500，那么就表示请求失败了。我们可以通过XMLHttpRequest对象的status属性得到服务器的状态码。

最后，我们还需要获取到服务器响应的内容，可以通过XMLHttpRequest对象的responseText得到服务器响应内容。
```javascript
xmlHttp.onreadystatechange = function() {
            if(xmlHttp.readyState == 4 && xmlHttp.status == 200) {
                alert(xmlHttp.responseText);
            }
        };
```

#### 原生js实现ajax小结
- 创建XMLHttpRequest对象；
- 调用open()方法打开与服务器的连接；
- 调用send()方法发送请求；
- 为XMLHttpRequest对象指定onreadystatechange事件函数，这个函数会在XMLHttpRequest的1、2、3、4，四种状态时被调用；

XMLHttpRequest对象的5种状态，通常我们只关心4状态。
XMLHttpRequest对象的status属性表示服务器状态码，它只有在readyState为4时才能获取到。
XMLHttpRequest对象的responseText属性表示服务器响应内容，它只有在readyState为4时才能获取到！
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>


</head>
 <body>
<h1>XMLHttpRequest - Ajax请求</h1>
<input type="button" onclick="XmlGetRequest();" value="Get发送请求" />

    <script>
        function XmlGetRequest(){
            function createXMLHttpRequest() {
        var xmlHttp;
        // 适用于大多数浏览器，以及IE7和IE更高版本
        try{
            xmlHttp = new XMLHttpRequest();
        } catch (e) {
            // 适用于IE6
            try {
                xmlHttp = new ActiveXObject("Msxml2.XMLHTTP");
            } catch (e) {
                // 适用于IE5.5，以及IE更早版本
                try{
                    xmlHttp = new ActiveXObject("Microsoft.XMLHTTP");
                } catch (e){}
            }
        }
        return xmlHttp;
    }       //alert('ok');
            var xmlHttp = createXMLHttpRequest();
            xmlHttp.open("GET", "/ajax_get/", true);
            xmlHttp.send(null);
            xmlHttp.onreadystatechange = function() {
            if(xmlHttp.readyState == 4 && xmlHttp.status == 200) {
                var test=xmlHttp.responseText;
                alert(test);
            }

        };


        }

    </script>
  </body>
</html>

#------------------------------------------
def login(request):
    print('hello ajax')
    return render(request,'index.html')
    # return HttpResponse('helloxiaoming')

def ajax_get(request):

    return HttpResponse('helloxiaoming')
```
思考：启动后台后，直接运行html，会怎么样？这就涉及到咱们一会要讲到的通源策略机制和跨域请求；

### 原生js实现ajax的post请求
发送POST请求

- 需要设置请求头：ContentType，其值为：application/x-www-form-urlencoded
- 使用xmlHttp对象的setRequestHeader(“Content-Type”, “application/x-www-form-urlencoded”)；
- 在发送时可以指定请求体了：xmlHttp.send(“username=xiaoming&password=123”)

#### 发送POST请求注意事项
POST请求必须设置ContentType请求头的值为application/x-www.form-encoded。表单的enctype默认值就是为application/x-www.form-encoded！因为默认值就是这个，所以大家可能会忽略这个值！当设置了<\form>的enctype=” application/x-www.form-encoded”时，等同与设置了Cotnent-Type请求头。

但在使用AJAX发送请求时，就没有默认值了，这需要我们自己来设置请求头：
```javascript
xmlHttp.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
```
当没有设置Content-Type请求头为application/x-www-form-urlencoded时，Web容器会忽略请求体的内容。所以，在使用AJAX发送POST请求时，需要设置这一请求头，然后使用send()方法来设置请求体内容。

xmlHttp.send("b=B");
这时django后台就可以获取到这个参数！！！
```html
<h1>AJAX2</h1>
<button onclick="send()">测试</button>
<div id="div1"></div>
```
```javascript
    <script>
       function createXMLHttpRequest() {
        try {
            return new XMLHttpRequest();//大多数浏览器
        } catch (e) {
            try {
                return new ActiveXObject("Msxml2.XMLHTTP");
            } catch (e) {
                return new ActiveXObject("Microsoft.XMLHTTP");
            }
        }
    }

    function send() {
        var xmlHttp = createXMLHttpRequest();
        xmlHttp.onreadystatechange = function() {
            if(xmlHttp.readyState == 4 && xmlHttp.status == 200) {
                var div = document.getElementById("div1");
                div.innerText = xmlHttp.responseText;
                div.textContent = xmlHttp.responseText;
            }
        };
        xmlHttp.open("POST", "/ajax_post/", true);
        xmlHttp.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
        xmlHttp.send("b=B");
    }
	</script>
```
```python
#--------------------------------views.py
def login(request):
    print('hello ajax')
    return render(request,'index.html')
    # return HttpResponse('helloxiaoming')

@csrf_exempt
def ajax_post(request):
    print('ok')
    return HttpResponse('helloxiaoming')

```
### 原生js实现动态检测用户名是否已注册
在注册表单中，当用户填写了用户名后，把光标移开后，会自动向服务器发送异步请求。服务器返回true或false，返回true表示这个用户名已经被注册过，返回false表示没有注册过。

客户端得到服务器返回的结果后，确定是否在用户名文本框后显示“用户名已被注册”的错误信息！

- 页面中给出注册表单；
- 在username表单字段中添加onblur事件，调用send()方法；
- send()方法获取username表单字段的内容，向服务器发送异步请求，参数为username；
- django 的视图函数：获取username参数，判断是否为“xiaoming”，如果是响应true，否则响应false；

```javascript
<script type="text/javascript">
function createXMLHttpRequest() {
    try {
        return new XMLHttpRequest();
    } catch (e) {
        try {
            return new ActiveXObject("Msxml2.XMLHTTP");
        } catch (e) {
            return new ActiveXObject("Microsoft.XMLHTTP");
        }
    }
}

function send() {
    var xmlHttp = createXMLHttpRequest();
    xmlHttp.onreadystatechange = function() {
        if(xmlHttp.readyState == 4 && xmlHttp.status == 200) {
            if(xmlHttp.responseText == "true") {
                document.getElementById("error").innerText = "用户名已被注册！";
                document.getElementById("error").textContent = "用户名已被注册！";
            } else {
                document.getElementById("error").innerText = "";
                document.getElementById("error").textContent = "";
            }
        }
    };
    xmlHttp.open("POST", "/ajax_check/", true, "json");
    xmlHttp.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");
    var username = document.getElementById("username").value;
    xmlHttp.send("username=" + username);
}
</script>
```
```html
//--------------------------------------------------

<h1>注册</h1>
<form action="" method="post">
用户名：<input id="username" type="text" name="username" onblur="send()"/><span id="error"></span><br/>
密　码：<input type="text" name="password"/><br/>
<input type="submit" value="注册"/>
</form>

```
```python
//--------------------------------------------------views.py
def login(request):
    print('hello ajax')
    return render(request,'index.html')
    # return HttpResponse('helloxiaoming')

@csrf_exempt
def ajax_check(request):
    print('ok')

    username=request.POST.get('username',None)
    if username=='xiaoming':
        return HttpResponse('true')
    return HttpResponse('false')
```

注意：
在Form元素的语法中，EncType表明提交数据的格式 用 Enctype 属性指定将数据回发到服务器时浏览器使用的编码类型。 例如： application/x-www-form-urlencoded： 窗体数据被编码为名称/值对。这是标准的编码格式。 multipart/form-data： 窗体数据被编码为一条消息，页上的每个控件对应消息中的一个部分，这个一般文件上传时用。 text/plain： 窗体数据以纯文本形式进行编码，其中不含任何控件或格式字符。

## 3. jquery实现ajax
jquery实现的ajax有三层封装，最底层是$.ajax(),上层封装是$.get()和$.post(),再上层封装是$.getScript()和$.getJson()接口。
用的最多的是下面两层封装接口

### post和get接口
```javascript
$.get(url, [data], [callback], [type])
$.post(url, [data], [callback], [type])
```
type类型：type: text|html|json|script
```javascript
function test() {

        //sendToTest();
         //testWithData();
         //testWithCallback();
        //testWithDataAndCallback();
         //testType();
         //testGetScript();
         //testGetJson();
        //testLoad();
    }

    // only get|post
    function sendToTest() {
        $.get('/test');
        $.post('/test');
    }

    // with data
    function testWithData() {
        $.get('/test?x=1&y=4');
        $.get('/test', {y: 2,d:6});
        $.post('/test', {z: 3});
    }

    // with callback
    function testWithCallback() {
        $.get('/user/allusers', function (data, callbacktype, jqXHR) {
            console.log(data);
            console.log(callbacktype);
            console.log(jqXHR);
        });
    }

    // with data and callback
    // 请求参数应该尽量放在data参数中，因为可以自动编码，手动拼接url要注意编码问题
    function testWithDataAndCallback() {
        //
        $.get('/user/list', {type: 1}, function (data) {
            console.log(data);//将json字符串解析成json对象
        });
    }

    function testType() {
        $.get('/types', function (data, callbacktype, jqXHR) {
            console.log(data);
            console.log(data.a);
            console.log("xiaoming")
            console.log(typeof  data);
            console.log(jqXHR.getResponseHeader('Content-Type'));
        });
    }
```
### getScript和getJSON接口
$.getScript()使用 AJAX 请求，获取和运行 JavaScript:
```javascript
function testGetScript() {
        // alert(testFun(3, 4));
        $.getScript('test.js', function () {
            alert(add(1, 6));
        });
    }

// test.js
function add(a,b){
   return a+b
   }
```
$.getJSON()与$.get()是一样的，只不过就是做后一个参数type必须是json数据了。一般同域操作用$.get()就可以，$.getJson 最主要是用来进行jsonp跨域操作的。
### ajax接口
```javascript
function basicUsage() {
        $.ajax('/test', {
            success: function () {
                alert('ok');
            }
        });

        $.ajax({
            url: '/test',
            success: function () {
                alert('okk');
            }
        });
    }
```
处理响应结果的回调函数 success、error、statusCode、complete：
```javascript
function callbacksUsage() {
        $.ajax('/user/allusers', {

            success: function (data) {
                console.log(arguments);
                console.log("xiaoming")
            },

            error: function (jqXHR, textStatus, err) {

                // jqXHR: jQuery增强的xhr
                // textStatus: 请求完成状态
                // err: 底层通过throw抛出的异常对象，类型与值与错误类型有关
                console.log(arguments);
            },

            complete: function (jqXHR, textStatus) {
                // jqXHR: jQuery增强的xhr
                // textStatus: 请求完成状态 success | error
                console.log('statusCode: %d, statusText: %s', jqXHR.status, jqXHR.statusText);
                console.log('textStatus: %s', textStatus);
            },

            statusCode: {
                '403': function (jqXHR, textStatus, err) {
                    console.log(arguments);
                    console.log('status code 403');
                    console.log("ahha403")
                },
                '400': function () {
                    console.log('status code 400');
                    console.log("hhhh400")
                }
            }
        });
    }
```
注意：模拟error方式：
```python
def login(request):

    return render(request,'Ajax.html')

def ajax_get(request):
    HttpResponse.status_code=500
    l=['alex','little alex']

    #return HttpResponse(l)
    return HttpResponse(json.dumps(l))
```

#### 1. 请求数据: data, processData, contentType, traditional
data:
当前ajax请求要携带的数据，是一个json的object对象，ajax方法就会默认地把它编码成某种格式(urlencoded:?a=1&b=2)发送给服务端；此外，ajax默认以get方式发送请求。
```javascript
function testData() {
       $.ajax("/test",{／／//此时的data是一个json形式的对象
          data:{
            a:1,
            b:2
          }
       });

//?a=1&b=2
```
processData：
声明当前的data数据是否进行转码或预处理，默认为true，即预处理；if 为false，那么对data：{a:1,b:2}会调用json对象的toString()方法，即{a:1,b:2}.toString(),最后得到一个［object，Object］形式的结果。
```javascript
alert({"1":"111","2":"222","3":"333"}.toString());//[object Object]
```
该属性的意义在于，当data是一个dom结构或者xml数据时，我们希望数据不要进行处理，直接发过去，就可以讲其设为true。

contentType：

默认值: "application/x-www-form-urlencoded"。发送信息至服务器时内容编码类型。

用来指明当前请求的数据编码格式；urlencoded: ?a=1&b=2；如果想以其他方式提交数据，比如contentType:"application/json"，即向服务器发送一个json字符串：
```javascript
$.ajax("/ajax_get",{

          data:JSON.stringify({
               a:22,
               b:33
           }),
           contentType:"application/json",
           type:"POST",

       });

//{a: 22, b: 33}
```
注意：contentType:"application/json"一旦设定，data必须是json字符串，不能是json对象

traditional：
一般是我们的data数据有数组时会用到
```javascript
$.ajax("/ajax_get",{
          data:{
               a:22,
               b:33,
               c:["x","y"]
           },
           type:"POST",
           traditional:"true",
       });
```
traditional为false会对数据进行深层次迭代；

#### 2. 响应数据: dataType、dataFilter
dataType：

预期服务器返回的数据类型,服务器端返回的数据会根据这个值解析后，传递给回调函数。

默认不需要显性指定这个属性，ajax会根据服务器返回的content Type来进行转换；比如我们的服务器响应的content Type为json格式，这时ajax方法就会对响应的内容进行一个json格式的转换，if转换成功，我们在success的回调函数里就会得到一个json格式的对象；转换失败就会触发error这个回调函数。

如果我们明确地指定目标类型，就可以使用data Type。比如服务器响应(Response Headers)的content Type是json格式(json对象)，但是我们希望得到的是一个json字符串，那就可以把dataType：text，这样ajax方法就不会把返回的数据进行json格式的转换。

实例：
```python
from django.shortcuts import render,HttpResponse
from django.views.decorators.csrf import csrf_exempt
# Create your views here.

import json

def login(request):

    return render(request,'Ajax.html')


def ajax_get(request):

    l=['alex','little alex']

    #return HttpResponse(l)
    return HttpResponse(json.dumps(l))
```
```javascript
//---------------------------------------------------
    function testData() {

        $.ajax('ajax_get', {
           success: function (data) {
           console.log(typeof(data));
                                     },
           dataType:"json",
                            }
                       )}

注解:Response Headers的content Type为text/html,所以返回的是String;但如果我们想要一个json对象
    设定dataType:"json"即可,相当于告诉ajax方法把服务器返回的数据转成json对象发送到前端.结果为object
```

dataType的可用值：html｜xml｜json｜text｜script

dataFilter

类型：Function 给 Ajax返回的原始数据的进行预处理的函数。
```python
function testData() {

         $.ajax('ajax_get', {
           success: function (data) {
               console.log(data);
            },


            dataType: 'json',
            dataFilter: function(data, type) {
                console.log(data);//["alex", "little alex"]
                console.log(type);//json
                //var tmp =  JSON.parse(data);
                return tmp.length;//2
            }
        });}
```

#### 3. 请求类型 type
类型：String 默认值: "GET")。请求方式 ("POST" 或 "GET")， 默认为 "GET"。注意：其它 HTTP 请求方法，如 PUT 和 DELETE 也可以使用，但仅部分浏览器支持。

#### 4. 前置处理 beforeSend(XHR)
类型：Function 发送请求前可修改 XMLHttpRequest 对象的函数，如添加自定义 HTTP 头。XMLHttpRequest 对象是唯一的参数。这是一个 Ajax 事件。如果返回 false 可以取消本次 ajax 请求。
```javascript
function testData() {
        $.ajax('ajax_get', {
         beforeSend: function (jqXHR, settings) {
                console.log(arguments);
                console.log('beforeSend');
                jqXHR.setRequestHeader('test', 'haha');
                jqXHR.testData = {a: 1, b: 2};
            },
            success: function (data) {
                console.log(data);
            },

            complete: function (xhr) {
                console.log(xhr);
                console.log(xhr.testData);
            },

        })};
```

#### 5. other parameter
jsonp

类型：String

在一个 jsonp 请求中重写回调函数的名字。这个值用来替代在 "callback=?" 这种 GET 或 POST 请求中 URL 参数里的 "callback" 部分，比如 {jsonp:'onJsonPLoad'} 会导致将 "onJsonPLoad=?" 传给服务器。

jsonpCallback

类型：String

为 jsonp 请求指定一个回调函数名。这个值将用来取代 jQuery 自动生成的随机函数名。这主要用来让 jQuery 生成度独特的函数名，这样管理请求更容易，也能方便地提供回调函数和错误处理。你也可以在想让浏览器缓存 GET 请求的时候，指定这个回调函数名。

## 4. json
### json介绍
JSON(JavaScript Object Notation) 是一种轻量级的数据交换格式。

JSON是用字符串来表示Javascript对象，例如可以在django中发送一个JSON格式的字符串给客户端Javascript，Javascript可以执行这个字符串，得到一个Javascript对象。

json就是js对象的一种表现形式(字符串的形式)
### json对象语法
JSON 语法：
- 数据在名称/值对中
- 数据由逗号分隔
- 花括号保存对象（对象需要使用大括号括起来）
- 方括号保存数组（数组使用中括号括起来）

```json
var person = {"name":"zhangSan", "age":"18", "sex":"male"};
alert(person.name + ", " + person.age + ", " + person.sex);
```
注意，key也要在双引号中！

JSON值：

- 数字  （整数或浮点数）
- 字符串（在双引号中）
- 逻辑值（true 或 false）
- 数组   （在方括号中）
- 对象   （在花括号中）
- null

```javascript
var person = {"name":"alex", "age":"18", "sex":"male", "hobby":["girl", "movie", "travle"]};
alert(person.name + ", " + person.age + ", " + person.sex + ", " + person.hobby);
```

带有方法的JSON对象：
```javascript
var person = {"name":"alex",
              "sex":"men",
              "teacher":{
                 "name":"tiechui",
                  "sex":"half_men",
              },
              "bobby":['basketball','running'],

               "getName":function() {return 80;}
              };
alert(person.name);
alert(person.getName());
alert(person.teacher.name);
alert(person.bobby[0]);
```
js接受python的json对象：
```html
在json的编码过程中，会存在从python原始类型向json类型的转换过程，具体的转换
    如下：

        python         -->        json
        dict                      object
        list,tuple                array
        str,unicode               string
        int,long,float            number
        True                      true
        False                     false
        None                      null
```
js与django的交互
```python
def login(request):
    obj={'name':"alex111"}
    return render(request,'index.html',{"objs":json.dumps(obj)})
```
```javascript
#----------------------------------
 <script>
     var temp={{ objs|safe }}
     alert(temp.name);
     alert(temp['name'])
 </script>
```

### JSON与XML比较
- 可读性：   XML胜出；
- 解码难度：JSON本身就是JS对象（主场作战），所以简单很多；
- 流行度：   XML已经流行好多年，但在AJAX领域，JSON更受欢迎。

### parse()和.stringify()
```html
parse用于从一个字符串中解析出json对象,如

var str = '{"name":"xiaoming","age":"23"}'

结果：

JSON.parse(str)

Object

age: "23"
name: "xiaoming"


注意：单引号写在{}外，每个属性名都必须用双引号，否则会抛出异常。


stringify()用于从一个对象解析出字符串，如

var
 a = {a:1,b:2}

结果：

JSON.stringify(a)

"{"a":1,"b":2}"
```

## 5. 跨域请求
### 同源策略机制
 浏览器有一个很重要的概念——同源策略(Same-Origin Policy)。所谓同源是指，域名，协议，端口相同。不同源的客户端脚本(javascript、ActionScript)在没明确授权的情况下，不能读写对方的资源。

简单的来说，浏览器允许包含在页面A的脚本访问第二个页面B的数据资源，这一切是建立在A和B页面是同源的基础上。

如果Web世界没有同源策略，当你登录淘宝账号并打开另一个站点时，这个站点上的JavaScript可以跨域读取你的淘宝账号数据，这样整个Web世界就无隐私可言了。
### jsonp的js实现
SONP是JSON with Padding的略称。可以让网页从别的域名（网站）那获取资料，即跨域读取数据。

它是一个非官方的协议，它允许在服务器端集成Script tags返回至客户端，通过javascript callback的形式实现跨域访问（这仅仅是JSONP简单的实现形式）。

JSONP就像是JSON+Padding一样(Padding这里我们理解为填充)
```python
#---------------------------http://127.0.0.1:8001/login

 def login(request):
    print('hello ajax')
    return render(request,'index.html')
 #---------------------------返回用户的index.html
 <h1>发送JSONP数据</h1>
```
```javascript
<script>
    function fun1(arg){
        alert("hello"+arg)
    }
</script>
<script src="http://127.0.0.1:8002/get_byjsonp/"></script>
```
```python
#-----------------------------http://127.0.0.1:8002/get_byjsonp

def get_byjsonp(req):
    print('8002...')
    return HttpResponse('fun1("xiaoming")')
```
这其实就是JSONP的简单实现模式，或者说是JSONP的原型：创建一个回调函数，然后在远程服务上调用这个函数并且将JSON 数据形式作为参数传递，完成回调。

将JSON数据填充进回调函数，这就是JSONP的JSON+Padding的含义吧。

一般情况下，我们希望这个script标签能够动态的调用，而不是像上面因为固定在html里面所以没等页面显示就执行了，很不灵活。我们可以通过javascript动态的创建script标签，这样我们就可以灵活调用远程服务了。
```javascript
<script>
    function addScriptTag(src){
     var script = document.createElement('script');
         script.setAttribute("type","text/javascript");
         script.src = src;
         document.body.appendChild(script);
    }
    function fun1(arg){
        alert("hello"+arg)
    }

    window.onload=function(){
        addScriptTag("http://127.0.0.1:8002/get_byjsonp/")
    }
</script>
```
现在将你自己在客户端定义的回调函数的函数名传送给服务端，服务端则会返回以你定义的回调函数名的方法，将获取的json数据传入这个方法完成回调：
```javascript
<script>
    function addScriptTag(src){
     var script = document.createElement('script');
         script.setAttribute("type","text/javascript");
         script.src = src;
         document.body.appendChild(script);
    }
    function fetch(arg){
        alert("hello"+arg)
    }

    window.onload=function(){
        addScriptTag("http://127.0.0.1:8002/get_byjsonp?callback=fetch")
    }

</script>
```
```python
#--------------------------------http://127.0.0.1:8002/get_byjsonp
    def get_byjsonp(req):

    callback=req.GET.get('callback')
    print(callback)
    return HttpResponse('%s("xiaoming")'%callback)
```
###  jQuery对JSONP的实现
jQuery框架也当然支持JSONP，可以使用$.getJSON(url,[data],[callback])方法
```javascript
<script type="text/javascript">
    $.getJSON("http://127.0.0.1:8002/get_byjsonp?callback=?",function(arg){
        alert("hello"+arg)
    });
</script>
```
结果是一样的，要注意的是在url的后面必须添加一个callback参数，这样getJSON方法才会知道是用JSONP方式去访问服务，callback后面的那个问号是内部自动生成的一个回调函数名。

当然，如果说我们想指定自己的回调函数名，或者说服务上规定了固定回调函数名该怎么办呢？我们可以使用$.ajax方法来实现
```javascript
<script type="text/javascript" src="/static/jquery-2.2.3.js"></script>

<script type="text/javascript">
   $.ajax({
        url:"http://127.0.0.1:8002/get_byjsonp",
        dataType:"jsonp",
        jsonp: 'callbacks',
        jsonpCallback:"fetch"
   });
    function fetch(arg){
        alert(arg);
    }
</script>
```
```python
#--------------------------------- http://127.0.0.1:8002/get_byjsonp
 def get_byjsonp(req):

    callback=req.GET.get('callbacks')
    print(callback)
    return HttpResponse('%s("xiaoming")'%callback)
```
another way:
```javascript
<script type="text/javascript" src="/static/jquery-2.2.3.js"></script>

<script type="text/javascript">
   $.ajax({
        url:"http://127.0.0.1:8002/get_byjsonp",
        dataType:"jsonp",
        jsonp: 'callbacks',
        success:function(data){
            alert(data)
        }
   });

</script>
```
```python
 #-------------------------------------http://127.0.0.1:8002/get_byjsonp
def get_byjsonp(req):

    callback=req.GET.get('callbacks')
    print(callback) #jQuery223015502220591490135_1477560648881
return HttpResponse("%s('xiaoming')"%callback)
```
没错，jsonpCallback就是可以指定我们自己的回调方法名fetch，远程服务接受callback参数的值就不再是自动生成的回调名，而是fetch。dataType是指定按照JSOPN方式访问远程服务。callback必须有，因为服务端根据它来去回调函数的名字，如果是自已定义的，那么就得有自定义的名字：jsonpCallback:"fetch"；如果不加这个参数，则自动生成一个随机名字。

利用jQuery可以很方便的实现JSONP来进行跨域访问。