# django之一些feature
## 本节内容
1. cookie
2. session
3. 跨站请求保护
4. 分页
5. 序列化
6. model模块
7. CBV和FBV
8. 模板渲染对象

## 1. cookie
cookie 是一种发送到客户浏览器的文本串句柄，并保存在客户机硬盘上，可以用来在某个WEB站点会话间持久的保持数据。
cookie特性：
1.  一组一组键值对保存在用户浏览器
2. 可以主动清除
3. 也可以被“伪造”
4. 跨域名cookie不共享
5. 浏览器可以设置不接受cookie

cookie在浏览器中将会被放在一个一个源下，可以把浏览器中保存cookie的地方想象成一棵树，如果没有设置cookie的Domain和Path,则，该cookie将会被浏览器使用在该域名的顶级域名及它的所有子域名下。

django设置cookie通过接受的req对象来设置：
```python
req.set_cookie(key, value='', max_age=None, expires=None, path='/',
                   domain=None, secure=None, httponly=None)
expires属性：
	指定了coolie的生存期，默认情况下coolie是暂时存在的，他们存储的值只在浏览器会话期间存在，当用户退出浏览器后这些值也会丢失。
    如果想让cookie存在一段时间，就要为expires属性设置为未来的一个过期日期。
    现在已经被max-age属性所取代，max-age用秒来设置cookie的生存期。
	max_age设置的值是数字，单位是秒，例如：max_age=10代表超时时间为10秒钟。
    expires对应的值是datetime对象，可以设置成expires=datetime.datetime.now()+datetime.timedelta(seconds=10)也是将超时时间延长10s

path属性:
	它指定与cookie关联在一起的网页。
    在默认的情况下cookie会与创建它的网页，与该网页处于同一目录下的网页以及与这个网页所在目录下的子目录下的网页关联。

domain属性：
	domain属性可以使多个web服务器共享cookie。domain属性的默认值是创建cookie的网页所在服务器的主机名。
    不能将一个cookie的域设置成服务器所在的域之外的域。
	例如让位于order.example.com的服务器能够读取catalog.example.com设置的cookie值。
    如果catalog.example.com的页面创建的cookie把自己的path属性设置为“/”，把domain属性设置成“.example.com”，那么所有位于catalog.example.com的网页和所有位于orlders.example.com的网页，以及位于example.com域的其他服务器上的网页都可以访问这个coolie。

secure属性：
	它是一个布尔值，指定在网络上如何传输cookie，默认是不安全的，通过一个普通的http连接传输。

httponly属性：
	只允许这个cookie使用在http协议里面，也就是说不允许客户端js什么的修改这个cookie的值，但是这个cookie是能够整体被替换的，所以这个安全性并没有什么卵用。。。。。。
```
django设置签名cookie方式：
```python
rep.set_signed_cookie(key,value,salt='加密盐',**kwrgs)
	参数：
        key,              键
        value='',         值
        salt			  加密盐
```

django获取cookie
```python
request.COOKIES['key']
request.get_signed_cookie(key, default=RAISE_ERROR, salt='', max_age=None)
    参数：
        default: 默认值
           salt: 加密盐  //注意这里的salt要和签名cookie时候的salt设置一样才能拿到解密的cookie
        max_age: 后台控制过期时间
```

## 2. session
session机制：
session机制是一种服务器端的机制，服务器使用一种类似于散列表的结构（也可能就是使用散列表）来保存信息。

当程序需要为某个客户端的请求创建一个session时，服务器首先检查这个客户端的请求里是否已包含了一个session标识（称为session id），如果已包含则说明以前已经为此客户端创建过session，服务器就按照session id把这个session检索出来使用（检索不到，会新建一个），如果客户端请求不包含session id，则为此客户端创建一个session并且生成一个与此session相关联的session id，session id的值应该是一个既不会重复，又不容易被找到规律以仿造的字符串，这个session id将被在本次响应中返回给客户端保存。保存这个session id的方式可以采用cookie，这样在交互过程中浏览器可以自动的按照规则把这个标识发送给服务器。一般这个cookie的名字都是类似于SEEESIONID。但cookie可以被人为的禁止，则必须有其他机制以便在cookie被禁止时仍然能够把session id传递回服务器。

生成随机字符串，并在服务端保存特定信息；

Django中默认支持Session，其内部提供了5种类型的Session供开发者使用：

使用以下的类型只需要在引擎修改下配置换成相应的类型就可以了。
1. 数据库（默认）
2. 缓存
3. 文件
4. 缓存+数据库
5. 加密cookie

### 1. 数据库
```python
配置 settings.py

    SESSION_ENGINE = 'django.contrib.sessions.backends.db'   # 引擎（默认）

    SESSION_COOKIE_NAME ＝ "sessionid"                       # Session的cookie保存在浏览器上时的key，即：sessionid＝随机字符串（默认）
    SESSION_COOKIE_PATH ＝ "/"                               # Session的cookie保存的路径（默认）
    SESSION_COOKIE_DOMAIN = None                             # Session的cookie保存的域名（默认）
    SESSION_COOKIE_SECURE = False                            # 是否Https传输cookie（默认）
    SESSION_COOKIE_HTTPONLY = True                           # 是否Session的cookie只支持http传输（默认）
    SESSION_COOKIE_AGE = 1209600                             # Session的cookie失效日期（2周）（默认）
    SESSION_EXPIRE_AT_BROWSER_CLOSE = False                  # 是否关闭浏览器使得Session过期（默认）
    SESSION_SAVE_EVERY_REQUEST = False                       # 是否每次请求都保存Session，默认修改之后才保存（默认）

使用

    def index(request):
        # 获取、设置、删除Session中数据
        request.session['k1']  # 获取
        request.session.get('k1',None)
        request.session['k1'] = 123  # 设置
        request.session.setdefault('k1',123) # 设置，存在则不设置

        del request.session['k1']

        # 获取所有 键、值、键值对
        request.session.keys()
        request.session.values()
        request.session.items()
        request.session.iterkeys()
        request.session.itervalues()
        request.session.iteritems()


        # 用户session的随机字符串（存在客户端浏览器中，也存在服务端的数据库中）
        request.session.session_key

        # 将所有Session失效日期小于当前日期的数据删除
        request.session.clear_expired()

        # 检查 用户session的随机字符串 在数据库中是否
        request.session.exists("session_key")

        # 删除当前用户的所有Session数据
        request.session.delete("session_key")

示例：

def session(request):
    # request.session
    request.session['k1'] = 123  # 设置session
    # request.session['k1']
    print(request.session.session_key)  # 获得用户session的随机字符串（存在客户端浏览器中，也存在服务端的数据库中）
    return HttpResponse('session')


def index(request):  # 获得session
    return HttpResponse(request.session['k1'])  # 这里可以直接读取到session
```

### 2. 缓存
```python
配置 settings.py

    SESSION_ENGINE = 'django.contrib.sessions.backends.cache'  # 引擎
    SESSION_CACHE_ALIAS = 'default'                            # 使用的缓存别名（默认内存缓存，也可以是memcache），此处别名依赖缓存的设置


    SESSION_COOKIE_NAME ＝ "sessionid"                        # Session的cookie保存在浏览器上时的key，即：sessionid＝随机字符串
    SESSION_COOKIE_PATH ＝ "/"                                # Session的cookie保存的路径
    SESSION_COOKIE_DOMAIN = None                              # Session的cookie保存的域名
    SESSION_COOKIE_SECURE = False                             # 是否Https传输cookie
    SESSION_COOKIE_HTTPONLY = True                            # 是否Session的cookie只支持http传输
    SESSION_COOKIE_AGE = 1209600                              # Session的cookie失效日期（2周）
    SESSION_EXPIRE_AT_BROWSER_CLOSE = False                   # 是否关闭浏览器使得Session过期
    SESSION_SAVE_EVERY_REQUEST = False                        # 是否每次请求都保存Session，默认修改之后才保存

使用和数据库Session使用一样，只需要修改下配置就可以。
```

### 3. 文件
```python
配置 settings.py

    SESSION_ENGINE = 'django.contrib.sessions.backends.file'    # 引擎
    SESSION_FILE_PATH = None                                    # 缓存文件路径，如果为None，则使用tempfile模块获取一个临时地址tempfile.gettempdir()    #如：/var/folders/d3/j9tj0gz93dg06bmwxmhh6_xm0000gn/T


    SESSION_COOKIE_NAME ＝ "sessionid"                          # Session的cookie保存在浏览器上时的key，即：sessionid＝随机字符串
    SESSION_COOKIE_PATH ＝ "/"                                  # Session的cookie保存的路径
    SESSION_COOKIE_DOMAIN = None                                # Session的cookie保存的域名
    SESSION_COOKIE_SECURE = False                               # 是否Https传输cookie
    SESSION_COOKIE_HTTPONLY = True                              # 是否Session的cookie只支持http传输
    SESSION_COOKIE_AGE = 1209600                                # Session的cookie失效日期（2周）
    SESSION_EXPIRE_AT_BROWSER_CLOSE = False                     # 是否关闭浏览器使得Session过期
    SESSION_SAVE_EVERY_REQUEST = False                          # 是否每次请求都保存Session，默认修改之后才保存
使用和数据库Session使用一样，只需要修改下配置就可以。
```

### 4. 缓存+数据库
```python
数据库用于做持久化，缓存用于提高效率
a. 配置 settings.py
    SESSION_ENGINE = 'django.contrib.sessions.backends.cached_db'        # 引擎
b. 使用
    同上
```

### 5. 加密cookie
```python
a. 配置 settings.py
    SESSION_ENGINE = 'django.contrib.sessions.backends.signed_cookies'   # 引擎
b. 使用
    同上
```

### 登录认证示例
```html
<form class="common_form" id="Form" method="post" action="/app01/login/">
    <div><h1 class="login_title">登录</h1></div>
    <div style="width: 600px">
        <div class="form_group"><input name="username" class="form-control" label='用户名' type="text" placeholder="用户名" require='true'></div>
    </div>
    <div style="width: 600px">
        <div class="form_group"><input name="password" class="form-control" label='密码' type="password" placeholder="密码" require='true'></div>
    </div>
    <div class="form_group"><input class="btn btn-info form_btn" type="submit" value="登录"></div>
</form>
```

```python
def login(request):
    if request.method == "POST":
        username = request.POST.get("username")
        password = request.POST.get("password")
        if username == "zhangsan" and password == "123456":
            request.session["IS_LOGIN"] = True  #创建session
            return redirect("/app01/home/")
    return render(request,"app01/login.html")

def home(request):
    islogin = request.session.get("IS_LOGIN",False)
    if islogin:#如果用户已登录
        return render(request,"app01/menus.html")
    else:
        return redirect("/app01/login/")

def logout(request):#退出
    try:
        del request.session['IS_LOGIN']
    except KeyError:
        pass
    return redirect("/app01/login/")
```

## 3. 跨站请求保护
一、简介

django为用户实现防止跨站请求伪造的功能，通过中间件 django.middleware.csrf.CsrfViewMiddleware 来完成。而对于django中设置防跨站请求伪造功能有分为全局和局部。

全局：
　　中间件 django.middleware.csrf.CsrfViewMiddleware

局部：

@csrf_protect，为当前函数强制设置防跨站请求伪造功能，即便settings中没有设置全局中间件。
@csrf_exempt，取消当前函数防跨站请求伪造功能，即便settings中设置了全局中间件。
注：from django.views.decorators.csrf import csrf_exempt,csrf_protect

二、应用

1、普通表单
```python
veiw中设置返回值：
　　return render_to_response('Account/Login.html',data,context_instance=RequestContext(request))
     或者
     return render(request, 'xxx.html', data)

html中设置Token:
　　{% csrf_token %}
```

2、Ajax

对于传统的form，可以通过表单的方式将token再次发送到服务端，而对于ajax的话，使用如下方式。

view.py
```python
from django.template.context import RequestContext
# Create your views here.
def test(request):

    if request.method == 'POST':
        print request.POST
        return HttpResponse('ok')
    return  render_to_response('app01/test.html',context_instance=RequestContext(request))

```

text.html
```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title></title>
</head>
<body>
    {% csrf_token %}
    <input type="button" onclick="Do();"  value="Do it"/>
    <script src="/static/plugin/jquery/jquery-1.8.0.js"></script>
    <script src="/static/plugin/jquery/jquery.cookie.js"></script>
    <script type="text/javascript">
        var csrftoken = $.cookie('csrftoken');

        function csrfSafeMethod(method) {
            // these HTTP methods do not require CSRF protection
            return (/^(GET|HEAD|OPTIONS|TRACE)$/.test(method));
        }
        $.ajaxSetup({  //设置ajax发送之前做的操作，在请求头里面加上一个键值对，django配置里面默认键是X-CSRFToken，可以自行在配置文件中修改
            beforeSend: function(xhr, settings) {
                if (!csrfSafeMethod(settings.type) && !this.crossDomain) {
                    xhr.setRequestHeader("X-CSRFToken", csrftoken);
                }
            }
        });
        function Do(){

            $.ajax({
                url:"/app01/test/",
                data:{id:1},
                type:'POST',
                success:function(data){
                    console.log(data);
                }
            });

        }
    </script>
</body>
</html>
```

## 4. 分页
在django中内置了分页功能：Paginator
但是其他web框架中没有该功能，所以自己实现一个分页功能以后可以拿到其他框架中直接使用。

自定义分页

分页功能在每个网站都是必要的，对于分页来说，其实就是根据用户的输入计算出应该在数据库表中的起始位置。
1. 设定每页显示数据条数
2. 用户输入页码（第一页、第二页...）
3. 根据设定的每页显示条数和当前页码，计算出需要取数据表的起始位置
4. 在数据表中根据起始位置取值，页面上输出数据

需求又来了，需要在页面上显示分页的页面。如：［上一页］［1］［2］［3］［4］［5］［下一页］

1. 设定每页显示数据条数
2. 用户输入页码（第一页、第二页...）
3. 设定显示多少页号
4. 获取当前数据总条数
5. 根据设定显示多少页号和数据总条数计算出，总页数
6. 根据设定的每页显示条数和当前页码，计算出需要取数据表的起始位置
7. 在数据表中根据起始位置取值，页面上输出数据
8. 输出分页html，如：［上一页］［1］［2］［3］［4］［5］［下一页］

分页示例:
```python
def paging(p,num,count_data):
    btn_num=11  #定义的按钮个数
    num_list=[5,10,15,20,30,50,100]  # 定义每页显示的行数
    page_num=(count_data//num) if count_data%num==0 else (count_data//num+1)  #拿到所有数据分页的页数
    if p==0:  #为了防止上一页和下一页在边界越界情况
        p+=1
    elif p==page_num+1:
        p-=1
    page_list=None
    if btn_num>=page_num:  #当总数小于最小图标数时
        page_list=range(1,page_num+1)
    else:
        if p<=btn_num//2:  #当选中图标小于图标数的一半时
            page_list=range(1,btn_num+1)
        elif page_num-p<=btn_num//2:  #当选中图标之后剩下图标少于图标数一半时
            page_list=range(page_num-btn_num+1,page_num+1)
        else:
            if btn_num%2==0:  #如果是偶数
                page_list=range(p-btn_num//2+1,p+btn_num//2+1)
            else:  #如果是奇数
                page_list=range(p-btn_num//2,p+btn_num//2+1)
    p_dict={"first":1,"last":page_num,"current":p,"up":p-1,"down":p+1}
    return page_list,p_dict,num_list

def show_teacher(req):
    if req.method=="GET":
        p=int(req.GET.get("p",1))  #第n页
        num=int(req.GET.get("num",10))  #每页展示的数据条数
        count_data=models.Teacher.objects.all().count()
        page_list,p_dict,num_list=utils.paging(p,num,count_data)  # page_list页面显示的数字按钮，p_dict字典，存放首页尾页，上页下页及当前页的值，num_list返回一个列表，设置每页显示多少行数据
        data = models.Teacher.objects.all()[(p_dict["current"]-1)*num:p_dict["current"]*num]
        return render(req, "teacher.html", {"data": data,"page_list":page_list,"p":p_dict,"num_list":num_list,"num":num})
```

```html
    {% if data %}
        <table class=" table table-striped table-bordered table-hover table-condensed">
        <thead>
            <tr class="info">
                <th>ID</th>
                <th>教师姓名</th>
                <th>所教班级</th>
                <th>选项</th>
            </tr>
        </thead>
        <tbody>
            {% for i in data %}
            <tr>
                <td>{{ i.id }}</td>
                <td>{{ i.name }}</td>
                <td>{% for j in i.cls.all %}{{ j.caption }} {% endfor %}</td>
                <td style="width: 200px">
                    <button id="modify" choice_id={{ i.id }} type="button" class="btn btn-primary" data-toggle="modal" data-target="#Mymodal">
                        编辑
                    </button>
                    <button id="del" choice_id={{ i.id }} type="button" class="btn btn-danger" data-toggle="modal" data-target="#Mymodal">
                        {% csrf_token %}
                        删除
                    </button>
                </td>
            </tr>
            {% endfor %}
        </tbody>
        </table>
        <div>
            <button id={{ p.first }} paging="paging" class="btn btn-primary">首页</button>
            <button id={{  p.up }} paging="paging" class="btn btn-primary">上一页</button>
            {% for i in page_list %}
                <button id={{ i }} paging="paging" class="btn {% if i == p.current %}btn-primary{% endif %}">{{ i }}</button>
            {% endfor %}
            <button id={{  p.down }} paging="paging" class="btn btn-primary">下一页</button>
            <button id={{ p.last }} paging="paging" class="btn btn-primary">尾页</button>
            <select class="form-control" style="display:inline-block;width: 120px;">
                {% for i in num_list %}
                <option value="{{ i }}" {% if i == num %}selected="selected"{% endif %}>每页{{ i }}条</option>
                {% endfor %}
            </select>
        </div>
    {% endif %}
```

```javascript
//点击分页按钮获取相关数据并发送ajax向服务器请求数据
function paging_action(ev){
    var p=$(ev.target).attr("id");
    var num=$($(ev.target).nextAll("select")).val();
    var tbl_name=$(".body_left .check").attr("id");
    ajax_paging({p:p,num:num},tbl_name)
}

//用来发送ajax请求获取分页数据并放到相应位置
function ajax_paging(data,tbl_name){
    if (!(data && tbl_name)){
        var p=$("#content button[paging=paging]")
    }
    if(tbl_name=="classes"){
        $.ajax("/show_classes/",{
            type:"GET",
            data:data,
            dataType:"html",
            success:ajax_body
        })
    }else if(tbl_name=="teacher"){
        $.ajax("/show_teacher/",{
            type:"GET",
            data:data,
            dataType:"html",
            success:ajax_body
        })
    }else if(tbl_name=="student"){
        $.ajax("/show_student/",{
            type:"GET",
            data:data,
            dataType:"html",
            success:ajax_body
        })
    }
}

//点击分页按钮获取相关数据并发送ajax向服务器请求数据
function paging_action(ev){
    var p=$(ev.target).attr("id");
    var num=$($(ev.target).nextAll("select")).val();
    var tbl_name=$(".body_left .check").attr("id");
    ajax_paging({p:p,num:num},tbl_name)
}

//添加按钮的点击事件，通过button的id值判断去执行不同的函数
$(".body_right").click(".btn",function (ev){
    var btn=$(ev.target);
    if(btn.attr("paging")=="paging"){paging_action(ev);}
});
```

总结，分页时需要做三件事：

创建处理分页数据的函数或类
根据分页数据获取数据
输出分页HTML，即：［首页］［上一页］［1］［2］［3］［4］［5］［下一页］［尾页］［每页显示条数］

## 5. 序列化
关于Django中的序列化主要应用在将数据库中检索的数据返回给客户端用户，特别的Ajax请求一般返回的为Json格式。

serializers

```python
from django.core import serializers  #django内部提供的一个序列化对象的方法
ret = models.BookType.objects.all()  #通过all或者filter这种方式拿到的queryset对象可以通过serializers进行序列化
data = serializers.serialize("json", ret)
```
json.dumps

```python
import json
#ret = models.BookType.objects.all().values('caption')  #通过values或者values_list方式拿到的数据，可以直接被list强转之后再通过json.dumps进行序列化
ret = models.BookType.objects.all().values_list('caption')
ret=list(ret)
result = json.dumps(ret)
```

由于json.dumps时无法处理datetime日期，所以可以通过自定义处理器来做扩展，如：

```python
import json
from datetime import date
from datetime import datetime

class JsonCustomEncoder(json.JSONEncoder):

    def default(self, field):
        if isinstance(field, datetime):
            return field.strftime('%Y-%m-%d %H:%M:%S')
        elif isinstance(field, date):
            return field.strftime('%Y-%m-%d')
        else:
            return json.JSONEncoder.default(self, field)

print(json.dumps(datetime.now(),cls=JsonCustomEncoder))
```

## 6. model模块
### 6.1 model字段
#### 6.1.1 所有字段
```python
AutoField(Field)
    - int自增列，必须填入参数 primary_key=True

BigAutoField(AutoField)
    - bigint自增列，必须填入参数 primary_key=True

    注：当model中如果没有自增列，则自动会创建一个列名为id的列
    from django.db import models

    class UserInfo(models.Model):
        # 自动创建一个列名为id的且为自增的整数列
        username = models.CharField(max_length=32)

    class Group(models.Model):
        # 自定义自增列
        nid = models.AutoField(primary_key=True)
        name = models.CharField(max_length=32)

SmallIntegerField(IntegerField):
    - 小整数 -32768 ～ 32767

PositiveSmallIntegerField(PositiveIntegerRelDbTypeMixin, IntegerField)
    - 正小整数 0 ～ 32767
IntegerField(Field)
    - 整数列(有符号的) -2147483648 ～ 2147483647

PositiveIntegerField(PositiveIntegerRelDbTypeMixin, IntegerField)
    - 正整数 0 ～ 2147483647

BigIntegerField(IntegerField):
    - 长整型(有符号的) -9223372036854775808 ～ 9223372036854775807

自定义无符号整数字段

    class UnsignedIntegerField(models.IntegerField):
        def db_type(self, connection):
            return 'integer UNSIGNED'

    PS: 返回值为字段在数据库中的属性，Django字段默认的值为：
        'AutoField': 'integer AUTO_INCREMENT',
        'BigAutoField': 'bigint AUTO_INCREMENT',
        'BinaryField': 'longblob',
        'BooleanField': 'bool',
        'CharField': 'varchar(%(max_length)s)',
        'CommaSeparatedIntegerField': 'varchar(%(max_length)s)',
        'DateField': 'date',
        'DateTimeField': 'datetime',
        'DecimalField': 'numeric(%(max_digits)s, %(decimal_places)s)',
        'DurationField': 'bigint',
        'FileField': 'varchar(%(max_length)s)',
        'FilePathField': 'varchar(%(max_length)s)',
        'FloatField': 'double precision',
        'IntegerField': 'integer',
        'BigIntegerField': 'bigint',
        'IPAddressField': 'char(15)',
        'GenericIPAddressField': 'char(39)',
        'NullBooleanField': 'bool',
        'OneToOneField': 'integer',
        'PositiveIntegerField': 'integer UNSIGNED',
        'PositiveSmallIntegerField': 'smallint UNSIGNED',
        'SlugField': 'varchar(%(max_length)s)',
        'SmallIntegerField': 'smallint',
        'TextField': 'longtext',
        'TimeField': 'time',
        'UUIDField': 'char(32)',

BooleanField(Field)
    - 布尔值类型

NullBooleanField(Field):
    - 可以为空的布尔值

CharField(Field)
    - 字符类型
    - 必须提供max_length参数， max_length表示字符长度

TextField(Field)
    - 文本类型

EmailField(CharField)：
    - 字符串类型，Django Admin以及ModelForm中提供验证机制

IPAddressField(Field)
    - 字符串类型，Django Admin以及ModelForm中提供验证 IPV4 机制

GenericIPAddressField(Field)
    - 字符串类型，Django Admin以及ModelForm中提供验证 Ipv4和Ipv6
    - 参数：
        protocol，用于指定Ipv4或Ipv6， 'both',"ipv4","ipv6"
        unpack_ipv4， 如果指定为True，则输入::ffff:192.0.2.1时候，可解析为192.0.2.1，开启刺功能，需要protocol="both"

URLField(CharField)
    - 字符串类型，Django Admin以及ModelForm中提供验证 URL

SlugField(CharField)
    - 字符串类型，Django Admin以及ModelForm中提供验证支持 字母、数字、下划线、连接符（减号）

CommaSeparatedIntegerField(CharField)
    - 字符串类型，格式必须为逗号分割的数字

UUIDField(Field)
    - 字符串类型，Django Admin以及ModelForm中提供对UUID格式的验证

FilePathField(Field)
    - 字符串，Django Admin以及ModelForm中提供读取文件夹下文件的功能
    - 参数：
            path,                      文件夹路径
            match=None,                正则匹配
            recursive=False,           递归下面的文件夹
            allow_files=True,          允许文件
            allow_folders=False,       允许文件夹

FileField(Field)
    - 字符串，路径保存在数据库，文件上传到指定目录
    - 参数：
        upload_to = ""      上传文件的保存路径
        storage = None      存储组件，默认django.core.files.storage.FileSystemStorage

ImageField(FileField)
    - 字符串，路径保存在数据库，文件上传到指定目录
    - 参数：
        upload_to = ""      上传文件的保存路径
        storage = None      存储组件，默认django.core.files.storage.FileSystemStorage
        width_field=None,   上传图片的高度保存的数据库字段名（字符串）
        height_field=None   上传图片的宽度保存的数据库字段名（字符串）

DateTimeField(DateField)
    - 日期+时间格式 YYYY-MM-DD HH:MM[:ss[.uuuuuu]][TZ]

DateField(DateTimeCheckMixin, Field)
    - 日期格式      YYYY-MM-DD

TimeField(DateTimeCheckMixin, Field)
    - 时间格式      HH:MM[:ss[.uuuuuu]]

DurationField(Field)
    - 长整数，时间间隔，数据库中按照bigint存储，ORM中获取的值为datetime.timedelta类型

FloatField(Field)
    - 浮点型

DecimalField(Field)
    - 10进制小数
    - 参数：
        max_digits，小数总长度
        decimal_places，小数位长度

BinaryField(Field)
    - 二进制类型
```

#### 6.1.2 参数
```python
null                数据库中字段是否可以为空
db_column           数据库中字段的列名
db_tablespace
default             数据库中字段的默认值
primary_key         数据库中字段是否为主键
db_index            数据库中字段是否可以建立索引
unique              数据库中字段是否可以建立唯一索引
unique_for_date     数据库中字段【日期】部分是否可以建立唯一索引
unique_for_month    数据库中字段【月】部分是否可以建立唯一索引
unique_for_year     数据库中字段【年】部分是否可以建立唯一索引

verbose_name        Admin中显示的字段名称
blank               Admin中是否允许用户输入为空
editable            Admin中是否可以编辑
help_text           Admin中该字段的提示信息
choices             Admin中显示选择框的内容，用不变动的数据放在内存中从而避免跨表操作
                    如：gf = models.IntegerField(choices=[(0, '何穗'),(1, '大表姐'),],default=1)

error_messages      自定义错误信息（字典类型），从而定制想要显示的错误信息；
                    字典健：null, blank, invalid, invalid_choice, unique, and unique_for_date
                    如：{'null': "不能为空.", 'invalid': '格式错误'}

validators          自定义错误验证（列表类型），从而定制想要的验证规则
                    from django.core.validators import RegexValidator
                    from django.core.validators import EmailValidator,URLValidator,DecimalValidator,\
                    MaxLengthValidator,MinLengthValidator,MaxValueValidator,MinValueValidator
                    如：
                        test = models.CharField(
                            max_length=32,
                            error_messages={
                                'c1': '优先错信息1',
                                'c2': '优先错信息2',
                                'c3': '优先错信息3',
                            },
                            validators=[
                                RegexValidator(regex='root_\d+', message='错误了', code='c1'),
                                RegexValidator(regex='root_112233\d+', message='又错误了', code='c2'),
                                EmailValidator(message='又错误了', code='c3'), ]
                        )

```

#### 6.1.3 元信息
```python
class UserInfo(models.Model):
    nid = models.AutoField(primary_key=True)
    username = models.CharField(max_length=32)
    class Meta:
        # 数据库中生成的表名称 默认 app名称 + 下划线 + 类名
        db_table = "table_name"

        # 联合索引
        index_together = [
            ("pub_date", "deadline"),
        ]

        # 联合唯一索引
        unique_together = (("driver", "restaurant"),)

        # admin中显示的表名称
        verbose_name

        # verbose_name加s
        verbose_name_plural
'''
更多：https://docs.djangoproject.com/en/1.10/ref/models/options/
注:
1.触发Model中的验证和错误提示有两种方式：
    a. Django Admin中的错误信息会优先根据Admiin内部的ModelForm错误信息提示，如果都成功，才来检查Model的字段并显示指定错误信息
    b. 调用Model对象的 clean_fields 方法，如：
        # models.py
        class UserInfo(models.Model):
            nid = models.AutoField(primary_key=True)
            username = models.CharField(max_length=32)

            email = models.EmailField(error_messages={'invalid': '格式错了.'})

        # views.py
        def index(request):
            obj = models.UserInfo(username='11234', email='uu')
            try:
                print(obj.clean_fields())
            except Exception as e:
                print(e)
            return HttpResponse('ok')

       # Model的clean方法是一个钩子，可用于定制操作，如：上述的异常处理。

2.Admin中修改错误提示
'''
    # admin.py
    from django.contrib import admin
    from model_club import models
    from django import forms


    class UserInfoForm(forms.ModelForm):
        username = forms.CharField(error_messages={'required': '用户名不能为空.'})
        email = forms.EmailField(error_messages={'invalid': '邮箱格式错误.'})
        age = forms.IntegerField(initial=1, error_messages={'required': '请输入数值.', 'invalid': '年龄必须为数值.'})

        class Meta:
            model = models.UserInfo
            # fields = ('username',)
            fields = "__all__"


    class UserInfoAdmin(admin.ModelAdmin):
        form = UserInfoForm

    admin.site.register(models.UserInfo, UserInfoAdmin)
```

#### 6.1.4 多表关系
```python
ForeignKey(ForeignObject) # ForeignObject(RelatedField)
    to,                         # 要进行关联的表名
    to_field=None,              # 要关联的表中的字段名称
    on_delete=None,             # 当删除关联表中的数据时，当前表与其关联的行的行为
                                    - models.CASCADE，删除关联数据，与之关联也删除
                                    - models.DO_NOTHING，删除关联数据，引发错误IntegrityError
                                    - models.PROTECT，删除关联数据，引发错误ProtectedError
                                    - models.SET_NULL，删除关联数据，与之关联的值设置为null（前提FK字段需要设置为可空）
                                    - models.SET_DEFAULT，删除关联数据，与之关联的值设置为默认值（前提FK字段需要设置默认值）
                                    - models.SET，删除关联数据，
                                                  a. 与之关联的值设置为指定值，设置：models.SET(值)
                                                  b. 与之关联的值设置为可执行对象的返回值，设置：models.SET(可执行对象)

                                                    def func():
                                                        return 10

                                                    class MyModel(models.Model):
                                                        user = models.ForeignKey(
                                                            to="User",
                                                            to_field="id"
                                                            on_delete=models.SET(func),)
    related_name=None,          # 反向操作时，使用的字段名，用于代替 【表名_set】 如： obj.表名_set.all()
    related_query_name=None,    # 反向操作时，使用的连接前缀，用于替换【表名】     如： models.UserGroup.objects.filter(表名__字段名=1).values('表名__字段名')
    limit_choices_to=None,      # 在Admin或ModelForm中显示关联数据时，提供的条件：
                                # 如：
                                        - limit_choices_to={'nid__gt': 5}
                                        - limit_choices_to=lambda : {'nid__gt': 5}

                                        from django.db.models import Q
                                        - limit_choices_to=Q(nid__gt=10)
                                        - limit_choices_to=Q(nid=8) | Q(nid__gt=10)
                                        - limit_choices_to=lambda : Q(Q(nid=8) | Q(nid__gt=10)) & Q(caption='root')
    db_constraint=True          # 是否在数据库中创建外键约束
    parent_link=False           # 在Admin中是否显示关联数据


OneToOneField(ForeignKey)
    to,                         # 要进行关联的表名
    to_field=None               # 要关联的表中的字段名称
    on_delete=None,             # 当删除关联表中的数据时，当前表与其关联的行的行为

                                ###### 对于一对一 ######
                                # 1. 一对一其实就是 一对多 + 唯一索引
                                # 2.当两个类之间有继承关系时，默认会创建一个一对一字段
                                # 如下会在A表中额外增加一个c_ptr_id列且唯一：
                                        class C(models.Model):
                                            nid = models.AutoField(primary_key=True)
                                            part = models.CharField(max_length=12)

                                        class A(C):
                                            id = models.AutoField(primary_key=True)
                                            code = models.CharField(max_length=1)

ManyToManyField(RelatedField)
    to,                         # 要进行关联的表名
    related_name=None,          # 反向操作时，使用的字段名，用于代替 【表名_set】 如： obj.表名_set.all()
    related_query_name=None,    # 反向操作时，使用的连接前缀，用于替换【表名】     如： models.UserGroup.objects.filter(表名__字段名=1).values('表名__字段名')
    limit_choices_to=None,      # 在Admin或ModelForm中显示关联数据时，提供的条件：
                                # 如：
                                        - limit_choices_to={'nid__gt': 5}
                                        - limit_choices_to=lambda : {'nid__gt': 5}

                                        from django.db.models import Q
                                        - limit_choices_to=Q(nid__gt=10)
                                        - limit_choices_to=Q(nid=8) | Q(nid__gt=10)
                                        - limit_choices_to=lambda : Q(Q(nid=8) | Q(nid__gt=10)) & Q(caption='root')
    symmetrical=None,           # 仅用于多对多自关联时，symmetrical用于指定内部是否创建反向操作的字段
                                # 做如下操作时，不同的symmetrical会有不同的可选字段
                                    models.BB.objects.filter(...)

                                    # 可选字段有：code, id, m1
                                        class BB(models.Model):

                                        code = models.CharField(max_length=12)
                                        m1 = models.ManyToManyField('self',symmetrical=True)

                                    # 可选字段有: bb, code, id, m1
                                        class BB(models.Model):

                                        code = models.CharField(max_length=12)
                                        m1 = models.ManyToManyField('self',symmetrical=False)

    through=None,               # 自定义第三张表时，使用字段用于指定关系表
    through_fields=None,        # 自定义第三张表时，使用字段用于指定关系表中那些字段做多对多关系表
                                    from django.db import models

                                    class Person(models.Model):
                                        name = models.CharField(max_length=50)

                                    class Group(models.Model):
                                        name = models.CharField(max_length=128)
                                        members = models.ManyToManyField(
                                            Person,
                                            through='Membership',
                                            through_fields=('group', 'person'),
                                        )

                                    class Membership(models.Model):
                                        group = models.ForeignKey(Group, on_delete=models.CASCADE)
                                        person = models.ForeignKey(Person, on_delete=models.CASCADE)
                                        inviter = models.ForeignKey(
                                            Person,
                                            on_delete=models.CASCADE,
                                            related_name="membership_invites",
                                        )
                                        invite_reason = models.CharField(max_length=64)
    db_constraint=True,         # 是否在数据库中创建外键约束
    db_table=None,              # 默认创建第三张表时，数据库中表的名称

练习：使用ForeignKey 、ManyToManyField、OneToOneField创建表
```
#### 6.1.5 操作
```python
基本操作：
    # 增
    #
    # models.Tb1.objects.create(c1='xx', c2='oo')  增加一条数据，可以接受字典类型数据 **kwargs

    # obj = models.Tb1(c1='xx', c2='oo')
    # obj.save()

    # 查
    #
    # models.Tb1.objects.get(id=123)         # 获取单条数据，不存在则报错（不建议）
    # models.Tb1.objects.all()               # 获取全部
    # models.Tb1.objects.filter(name='seven') # 获取指定条件的数据
    # models.Tb1.objects.exclude(name='seven') # 获取指定条件的数据

    # 删
    #
    # models.Tb1.objects.filter(name='seven').delete() # 删除指定条件的数据

    # 改
    # models.Tb1.objects.filter(name='seven').update(gender='0')  # 将指定条件的数据更新，均支持 **kwargs
    # obj = models.Tb1.objects.get(id=1)
    # obj.c1 = '111'
    # obj.save()                                                 # 修改单条数据

进阶操作：
    # 获取个数
    #
    # models.Tb1.objects.filter(name='seven').count()

    # 大于，小于
    #
    # models.Tb1.objects.filter(id__gt=1)              # 获取id大于1的值
    # models.Tb1.objects.filter(id__gte=1)              # 获取id大于等于1的值
    # models.Tb1.objects.filter(id__lt=10)             # 获取id小于10的值
    # models.Tb1.objects.filter(id__lte=10)             # 获取id小于10的值
    # models.Tb1.objects.filter(id__lt=10, id__gt=1)   # 获取id大于1 且 小于10的值

    # in
    #
    # models.Tb1.objects.filter(id__in=[11, 22, 33])   # 获取id等于11、22、33的数据
    # models.Tb1.objects.exclude(id__in=[11, 22, 33])  # not in

    # isnull
    # Entry.objects.filter(pub_date__isnull=True)

    # contains
    #
    # models.Tb1.objects.filter(name__contains="ven")
    # models.Tb1.objects.filter(name__icontains="ven") # icontains大小写不敏感
    # models.Tb1.objects.exclude(name__icontains="ven")

    # range
    #
    # models.Tb1.objects.filter(id__range=[1, 2])   # 范围bettwen and

    # 其他类似
    #
    # startswith，istartswith, endswith, iendswith,

    # order by
    #
    # models.Tb1.objects.filter(name='seven').order_by('id')    # asc
    # models.Tb1.objects.filter(name='seven').order_by('-id')   # desc

    # group by
    #
    # from django.db.models import Count, Min, Max, Sum
    # models.Tb1.objects.filter(c1=1).values('id').annotate(c=Count('num'))
    # SELECT "app01_tb1"."id", COUNT("app01_tb1"."num") AS "c" FROM "app01_tb1" WHERE "app01_tb1"."c1" = 1 GROUP BY "app01_tb1"."id"

    # limit 、offset
    #
    # models.Tb1.objects.all()[10:20]

    # regex正则匹配，iregex 不区分大小写
    #
    # Entry.objects.get(title__regex=r'^(An?|The) +')
    # Entry.objects.get(title__iregex=r'^(an?|the) +')

    # date
    #
    # Entry.objects.filter(pub_date__date=datetime.date(2005, 1, 1))
    # Entry.objects.filter(pub_date__date__gt=datetime.date(2005, 1, 1))

    # year
    #
    # Entry.objects.filter(pub_date__year=2005)
    # Entry.objects.filter(pub_date__year__gte=2005)

    # month
    #
    # Entry.objects.filter(pub_date__month=12)
    # Entry.objects.filter(pub_date__month__gte=6)

    # day
    #
    # Entry.objects.filter(pub_date__day=3)
    # Entry.objects.filter(pub_date__day__gte=3)

    # week_day
    #
    # Entry.objects.filter(pub_date__week_day=2)
    # Entry.objects.filter(pub_date__week_day__gte=2)

    # hour
    #
    # Event.objects.filter(timestamp__hour=23)
    # Event.objects.filter(time__hour=5)
    # Event.objects.filter(timestamp__hour__gte=12)

    # minute
    #
    # Event.objects.filter(timestamp__minute=29)
    # Event.objects.filter(time__minute=46)
    # Event.objects.filter(timestamp__minute__gte=29)

    # second
    #
    # Event.objects.filter(timestamp__second=31)
    # Event.objects.filter(time__second=2)
    # Event.objects.filter(timestamp__second__gte=31)

其他操作：
    # extra
    #
    # extra(self, select=None, where=None, params=None, tables=None, order_by=None, select_params=None)
    #    Entry.objects.extra(select={'new_id': "select col from sometable where othercol > %s"}, select_params=(1,))
    #    Entry.objects.extra(where=['headline=%s'], params=['Lennon'])
    #    Entry.objects.extra(where=["foo='a' OR bar = 'a'", "baz = 'a'"])
    #    Entry.objects.extra(select={'new_id': "select id from tb where id > %s"}, select_params=(1,), order_by=['-nid'])

    # F
    #
    # from django.db.models import F
    # models.Tb1.objects.update(num=F('num')+1)


    # Q
    #
    # 方式一：
    # Q(nid__gt=10)
    # Q(nid=8) | Q(nid__gt=10)
    # Q(Q(nid=8) | Q(nid__gt=10)) & Q(caption='root')

    # 方式二：
    # con = Q()
    # q1 = Q()
    # q1.connector = 'OR'
    # q1.children.append(('id', 1))
    # q1.children.append(('id', 10))
    # q1.children.append(('id', 9))
    # q2 = Q()
    # q2.connector = 'OR'
    # q2.children.append(('c1', 1))
    # q2.children.append(('c1', 10))
    # q2.children.append(('c1', 9))
    # con.add(q1, 'AND')
    # con.add(q2, 'AND')
    #
    # models.Tb1.objects.filter(con)


    # 执行原生SQL
    #
    # from django.db import connection, connections
    # cursor = connection.cursor()  # cursor = connections['default'].cursor()
    # cursor.execute("""SELECT * from auth_user where id = %s""", [1])
    # row = cursor.fetchone()
```

### 6.2 ORM基本操作
```python
##################################################################
# PUBLIC METHODS THAT ALTER ATTRIBUTES AND RETURN A NEW QUERYSET #
##################################################################

def all(self)
    # 获取所有的数据对象

def filter(self, *args, **kwargs)
    # 条件查询
    # 条件可以是：参数，字典，Q

def exclude(self, *args, **kwargs)
    # 条件查询
    # 条件可以是：参数，字典，Q

def select_related(self, *fields)
     性能相关：表之间进行join连表操作，一次性获取关联的数据。
     model.tb.objects.all().select_related()
     model.tb.objects.all().select_related('外键字段')
     model.tb.objects.all().select_related('外键字段__外键字段')

def prefetch_related(self, *lookups)
    性能相关：多表连表操作时速度会慢，使用其执行多次SQL查询在Python代码中实现连表操作。
            # 获取所有用户表
            # 获取用户类型表where id in (用户表中的查到的所有用户ID)
            models.UserInfo.objects.prefetch_related('外键字段')



            from django.db.models import Count, Case, When, IntegerField
            Article.objects.annotate(
                numviews=Count(Case(
                    When(readership__what_time__lt=treshold, then=1),
                    output_field=CharField(),
                ))
            )

            students = Student.objects.all().annotate(num_excused_absences=models.Sum(
                models.Case(
                    models.When(absence__type='Excused', then=1),
                default=0,
                output_field=models.IntegerField()
            )))

def annotate(self, *args, **kwargs)
    # 用于实现聚合group by查询

    from django.db.models import Count, Avg, Max, Min, Sum

    v = models.UserInfo.objects.values('u_id').annotate(uid=Count('u_id'))
    # SELECT u_id, COUNT(ui) AS `uid` FROM UserInfo GROUP BY u_id

    v = models.UserInfo.objects.values('u_id').annotate(uid=Count('u_id')).filter(uid__gt=1)
    # SELECT u_id, COUNT(ui_id) AS `uid` FROM UserInfo GROUP BY u_id having count(u_id) > 1

    v = models.UserInfo.objects.values('u_id').annotate(uid=Count('u_id',distinct=True)).filter(uid__gt=1)
    # SELECT u_id, COUNT( DISTINCT ui_id) AS `uid` FROM UserInfo GROUP BY u_id having count(u_id) > 1

def distinct(self, *field_names)
    # 用于distinct去重
    models.UserInfo.objects.values('nid').distinct()
    # select distinct nid from userinfo

    注：只有在PostgreSQL中才能使用distinct进行去重

def order_by(self, *field_names)
    # 用于排序
    models.UserInfo.objects.all().order_by('-id','age')

def extra(self, select=None, where=None, params=None, tables=None, order_by=None, select_params=None)
    # 构造额外的查询条件或者映射，如：子查询

    Entry.objects.extra(select={'new_id': "select col from sometable where othercol > %s"}, select_params=(1,))
    Entry.objects.extra(where=['headline=%s'], params=['Lennon'])
    Entry.objects.extra(where=["foo='a' OR bar = 'a'", "baz = 'a'"])
    Entry.objects.extra(select={'new_id': "select id from tb where id > %s"}, select_params=(1,), order_by=['-nid'])

 def reverse(self):
    # 倒序
    models.UserInfo.objects.all().order_by('-nid').reverse()
    # 注：如果存在order_by，reverse则是倒序，如果多个排序则一一倒序


 def defer(self, *fields):
    models.UserInfo.objects.defer('username','id')
    或
    models.UserInfo.objects.filter(...).defer('username','id')
    #映射中排除某列数据

 def only(self, *fields):
    #仅取某个表中的数据
     models.UserInfo.objects.only('username','id')
     或
     models.UserInfo.objects.filter(...).only('username','id')

 def using(self, alias):
     指定使用的数据库，参数为别名（setting中的设置）


##################################################
# PUBLIC METHODS THAT RETURN A QUERYSET SUBCLASS #
##################################################

def raw(self, raw_query, params=None, translations=None, using=None):
    # 执行原生SQL
    models.UserInfo.objects.raw('select * from userinfo')

    # 如果SQL是其他表时，必须将名字设置为当前UserInfo对象的主键列名
    models.UserInfo.objects.raw('select id as nid from 其他表')

    # 为原生SQL设置参数
    models.UserInfo.objects.raw('select id as nid from userinfo where nid>%s', params=[12,])

    # 将获取的到列名转换为指定列名
    name_map = {'first': 'first_name', 'last': 'last_name', 'bd': 'birth_date', 'pk': 'id'}
    Person.objects.raw('SELECT * FROM some_other_table', translations=name_map)

    # 指定数据库
    models.UserInfo.objects.raw('select * from userinfo', using="default")

    ################### 原生SQL ###################
    from django.db import connection, connections
    cursor = connection.cursor()  # cursor = connections['default'].cursor()
    cursor.execute("""SELECT * from auth_user where id = %s""", [1])
    row = cursor.fetchone() # fetchall()/fetchmany(..)


def values(self, *fields):
    # 获取每行数据为字典格式

def values_list(self, *fields, **kwargs):
    # 获取每行数据为元祖

def dates(self, field_name, kind, order='ASC'):
    # 根据时间进行某一部分进行去重查找并截取指定内容
    # kind只能是："year"（年）, "month"（年-月）, "day"（年-月-日）
    # order只能是："ASC"  "DESC"
    # 并获取转换后的时间
        - year : 年-01-01
        - month: 年-月-01
        - day  : 年-月-日

    models.DatePlus.objects.dates('ctime','day','DESC')

def datetimes(self, field_name, kind, order='ASC', tzinfo=None):
    # 根据时间进行某一部分进行去重查找并截取指定内容，将时间转换为指定时区时间
    # kind只能是 "year", "month", "day", "hour", "minute", "second"
    # order只能是："ASC"  "DESC"
    # tzinfo时区对象
    models.DDD.objects.datetimes('ctime','hour',tzinfo=pytz.UTC)
    models.DDD.objects.datetimes('ctime','hour',tzinfo=pytz.timezone('Asia/Shanghai'))

    """
    pip3 install pytz
    import pytz
    pytz.all_timezones
    pytz.timezone(‘Asia/Shanghai’)
    """

def none(self):
    # 空QuerySet对象


####################################
# METHODS THAT DO DATABASE QUERIES #
####################################

def aggregate(self, *args, **kwargs):
   # 聚合函数，获取字典类型聚合结果
   from django.db.models import Count, Avg, Max, Min, Sum
   result = models.UserInfo.objects.aggregate(k=Count('u_id', distinct=True), n=Count('nid'))
   ===> {'k': 3, 'n': 4}

def count(self):
   # 获取个数

def get(self, *args, **kwargs):
   # 获取单个对象

def create(self, **kwargs):
   # 创建对象

def bulk_create(self, objs, batch_size=None):
    # 批量插入
    # batch_size表示一次插入的个数
    objs = [
        models.DDD(name='r11'),
        models.DDD(name='r22')
    ]
    models.DDD.objects.bulk_create(objs, 10)

def get_or_create(self, defaults=None, **kwargs):
    # 如果存在，则获取，否则，创建
    # defaults 指定创建时，其他字段的值
    obj, created = models.UserInfo.objects.get_or_create(username='root1', defaults={'email': '1111111','u_id': 2, 't_id': 2})

def update_or_create(self, defaults=None, **kwargs):
    # 如果存在，则更新，否则，创建
    # defaults 指定创建时或更新时的其他字段
    obj, created = models.UserInfo.objects.update_or_create(username='root1', defaults={'email': '1111111','u_id': 2, 't_id': 1})

def first(self):
   # 获取第一个

def last(self):
   # 获取最后一个

def in_bulk(self, id_list=None):
   # 根据主键ID进行查找
   id_list = [11,21,31]
   models.DDD.objects.in_bulk(id_list)

def delete(self):
   # 删除

def update(self, **kwargs):
    # 更新

def exists(self):
   # 是否有结果

```
## 7. CBV和FBV
django里面的views里面支持两种处理请求的方式：
- CBV--->views里面处理请求使用类
- FBV--->views里面处理请求使用函数

使用视图函数处理请求的方式就不赘述了，之前一直是这种方式。
下面介绍使用类处理请求的方式。

首先在urls里面定义方式需要改变：
```python
urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^login/', views.Login.as_view()),
]
```
要将之前的函数名称改成类名.as_view()方式，之后在views里面导入views模块：
```python
from django import views
from django.utils.decorators import method_decorator #用来在类里面的函数视图函数中加上装饰器时使用
```
之后定义类继承自views.View:
```python

def outer(func):
    def wrapper(req,*args,**kwargs):
        print("xxxxxxx")
        return func(req,*args,*kwargs)
    return wrapper

class Login(views.View):

    def dispatch(self, request, *args, **kwargs):
        if request.method=="GET":
            # return HttpResponse("xxx")
            pass
        ret=super(Login, self).dispatch(request, *args, **kwargs)
        print(111)
        return ret

    @method_decorator(outer)
    def get(self,req,*args,**kwargs):
        return render(req,"login.html")

    @method_decorator(outer)
    def post(self,req,*args,**kwargs):
        return render(req,"login.html",{"msg":"hello post"})
```

至于Login继承自views.View，而其父类中有一个方法dispatch，这个方法用来分发请求到不同的处理视图函数中，继承下这个类的子类中重写这个方法可以在视图函数处理请求之前和处理请求之后做一些操作。（甚至可以把装饰器装饰在这上面）

导入的method_decorator方法用来在类中的视图函数前面加上装饰器，把装饰器当做参数传递给method_decorator函数中。

## 8. 模板渲染对象
在django的模板语言中，可以使用.来获取对象的属性和方法，那么，当这个对象有同名的属性和方法时，模板语言调用对象的属性，而当对象中没有这个属性时，则调用该对象的方法。

```python
class testobj:
    def __init__(self,name):
        self.name=name
    def name(self):
        return "xiaoming2"

def test(req):
obj=testobj("xiaoming")

    return render(req,"test.html",{"obj":obj})
```
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>test template</h1>
    <h1>{{ obj.name }}</h1>
</body>
</html>
```
页面拿到的值是xiaoming，而当把self.name改成self.aname时则页面拿到的是xiaoming2
