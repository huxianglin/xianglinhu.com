# 前端之web上传文件的方式
## 本节内容
1. web上传文件方式介绍
2. form上传文件
3. 原生js实现ajax上传文件
4. jquery实现ajax上传文件
5. form+iframe构造请求上传文件

## 1. web上传文件方式介绍
在web浏览器上传文件一般有以下几种方式：
- form表单上传文件
- 原生js实现ajax上传文件
- jquery实现ajax上传文件
- form+iframe上传文件

其中form提交数据之后会整个刷新页面；
js通过ajax上传文件虽然不会刷新整个页面，但是他们都是通过使用formdata对象实现的，formdata对象在老版本的浏览器中并不支持；
为了兼容老版本浏览器，使用iframe方式提交；

下面几节就分别就这几种方式实现上传文件来举例说明。

## 2. form上传文件
这是最原始的一种方式，最开始学习web的时候就是使用这种方式提交。
注意：在form表单中如果要上传文件，一定要设置这个参数： **enctype="multipart/form-data"**表示封装数据类型，把数据分成一个一个小段传输。

html代码：
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        .show-img{
            display: inline-block;
            width: 200px;
            height:200px;
        }
    </style>
</head>
<body>
    <h1>form表单上传文件</h1><hr>
    <form action="/test/" method="POST" enctype="multipart/form-data">
        <p>名称：<input type="text" name="user"></p>
        <p>文件：<input type="file" name="testfile"></p>
        <p><input type="submit" value="提交"></p>
    </form>
    {% for i in imglist %}
        <img class="show-img" src="/{{ i.0 }}">
    {% endfor %}
</body>
</html>
```

在后端要注意上传过来的文件是通过**req.FILES.get() **方法接收的。
后端代码:
```python
def test(req):
    if req.method=="GET":
        imglist=models.Img.objects.all().values_list("img_path")
        return render(req,"test.html",{"imglist":imglist})
    elif req.method=="POST":
        user=req.POST.get("user")
        file=req.FILES.get("testfile")
        path=os.path.join("static","imgs",file.name)
        with open(path,"wb") as f:
            for chunk in file.chunks():
                f.write(chunk)
        print(user,file.name,file.size)
        models.Img.objects.create(img_path=path)
        return redirect("/test/")
```

## 3. 原生js实现ajax上传文件
js源码：
```javascript
document.getElementById("js_post").onclick=function(){
    var xml=new XMLHttpRequest();
    var data=new FormData; //创建formdata对象
    data.append("testfile",document.getElementById("file_upload").files[0]);//找到对象之后的file[0]对应的就是文件对象
    xml.open("POST","/test/",true);
    xml.onreadystatechange=function(){
        if(xml.readyState==4 && xml.status==200){  //判断状态到4了并且返回状态码是200时才做操作
            var rsp_data=JSON.parse(xml.responseText);  //反序列化
            if (rsp_data.state){
                var url="/"+rsp_data.data;  //拼接路径
                var obj=document.createElement("img");  //创建标签
                obj.className="show-img";  //给标签加样式
                obj.src=url;  //给标签加url
                document.getElementById("imgs").appendChild(obj);
            }
        }
    };
    xml.send(data)
}
````
html源码：
```html
<hr><h1>ajax上传文件</h1><hr>
<p>文件：<input id="file_upload" type="file" name="testfile"></p>
<p><button id="js_post">原生js提交</button>  <button id="jquery_post">jquery提交</button></p>
<div id="imgs">
    {% for i in imglist %}
        <img class="show-img" src="/{{ i.0 }}">
    {% endfor %}
</div>
```
后端源码：
```python
def test(req):
	if req.method=="GET":
        imglist=models.Img.objects.all().values_list("img_path")
        return render(req,"test.html",{"imglist":imglist})
    elif req.method=="POST":
        user=req.POST.get("user",None)
        file=req.FILES.get("testfile")
        path=os.path.join("static","imgs",file.name)
        with open(path,"wb") as f:
            for chunk in file.chunks():
                f.write(chunk)
        print(user,file.name,file.size)
        models.Img.objects.create(img_path=path)
        # return redirect("/test/")
        msg={"code":200,"state":True,"data":path}
        return HttpResponse(json.dumps(msg))
```
注意：**FormData对象在添加文件对象的时候并不是把标签直接给append进去，而是找到标签之后.file(0)才是文件对象**

## 4. jquery实现ajax上传文件
这里的html代码和后端代码相对上面没有改变，这里就不列出来了。
jquery代码：
```javascript
$("#jquery_post").on("click",function(){
            var data=new FormData;
            data.append("testfile",document.getElementById("file_upload").files[0]);
            $.ajax({
                url:"/test/",
                type:"POST",
                dataType:"JSON",
                data:data,
                contentType: false,
                processData: false,
                success:function(rst){
                    if(rst.state){
                        var url="/"+rst.data;
                        $('<img class="show-img" src="'+url+'">').appendTo("#imgs")
                    }
                }
            })
        })
```
注意：**jquery的ajax会自动把我们的数据转换成字符串，不想转换时需要在ajax里面写入：contentType: false,processData: false,**

## 5. form+iframe构造请求上传文件
html代码：
```html
<hr><h1>form+iframe上传文件</h1><hr>
    <p><iframe id="uploadfile" name="uploadfile" style="display: none"></iframe></p>  <!--注意这里的name要和form中的target一致-->
    <form target="uploadfile" action="/test/" method="POST" enctype="multipart/form-data">
        <p>名称：<input type="text" name="user"></p>
        <p>文件：<input type="file" name="testfile"></p>
        <p><input type="submit" value="提交"></p>
    </form>
    <div id="imgs">
        {% for i in imglist %}
            <img class="show-img" src="/{{ i.0 }}">
        {% endfor %}
    </div>
```
注意：**这里的iframe的name的值一定要和form的target的值一样，这样才能实现绑定**

js代码：
```javascript
$("#uploadfile").on("load",function(){  //iframe里面有个方法是onload，当上传数据成功之后服务器返回数据时才会触发该事件
            var rst=JSON.parse(this.contentDocument.body.textContent);//拿到iframe里面的内容需要通过contentDocument才能拿到里面的dom对象
            if (rst.state){
                var url="/"+rst.data;
                        $('<img class="show-img" src="'+url+'">').appendTo("#imgs")
            }
        })
```
注意：**拿到iframe里面的内容需要通过contentDocument才能拿到里面的dom对象**
经过实际检测，当鼠标点击提交速度比较快时，js通过ajax提交数据没有问题，但是iframe就无法提交所有数据。。。
原因是ajax是异步加载的，而iframe更像是通过浏览器打开了一个新标签，等待服务端传输回来的数据。所以，当提交速度过快时无法达到相应的效果。
使用iframe的唯一一个好处目前来看也就是兼容老版本的浏览器罢了。。。
随着时间的流逝，这种方式终将被淘汰。