# Django之Form组件

## 本节内容
1. 基本使用
2. form中字段和插件
3. 自定义验证规则
4. 动态加载数据到form中

## 1. 基本使用
django中的Form组件有以下几个功能：
- 生成HTML标签
- 验证用户数据（显示错误信息）
- HTML Form提交保留上次提交数据
- 初始化页面显示内容

要使用form类，首先需要创建这个类，方法如下：
```python
from django.forms import Form
from django.forms import widgets  # 插件
from django.forms import fields  # 字段

class MyForm(Form):  # 继承自Form类
    user = fields.CharField(
        widget=widgets.TextInput(attrs={'id': 'i1', 'class': 'c1'})
        # 定义生成的html标签类型是input的text框，attrs={'id': 'i1', 'class': 'c1'}代表在这个标签中添加属性ID为i1，添加class为c1
    )

    gender = fields.ChoiceField(
        choices=((1, '男'), (2, '女'),),  # 定义下拉框的选项，元祖第一个值为option的value值，后面为html里面的值
        initial=2,  # 默认选中第二个option
        widget=widgets.RadioSelect  # 插件表现形式为单选按钮
    )

    city = fields.CharField(
        initial=2,  # 初始值为2
        widget=widgets.Select(choices=((1,'上海'),(2,'北京'),))  # 插件表现形式为下拉框
    )

    pwd = fields.CharField(
        widget=widgets.PasswordInput(attrs={'class': 'c1'}, render_value=True)  # 插件表现形式为密码输入框
    )
```
之后在views视图中定义：
```python
from django.shortcuts import render, redirect
from .forms import MyForm

def index(request):
    if request.method == "GET":
        obj = MyForm()  # 没有值，在页面上渲染form中的标签
        return render(request, 'index.html', {'form': obj})
    elif request.method == "POST":
        obj = MyForm(request.POST, request.FILES)  # 将post提交过来的数据作为参数传递给自定义的Form类
        if obj.is_valid():  # obj.is_valid()返回一个bool值，如果检查通过返回True，否则返回False
            values = obj.clean()  # 拿到处理后的所有数据，键值对的形式
            print(values)
        else:
            errors = obj.errors  # 拿到未通过的错误信息，里面封装的都是对象
            print(errors)
        return render(request, 'index.html', {'form': obj})
    else:
        return redirect('http://www.google.com')
```
模板定义：
```html
<form action="/" method="POST" enctype="multipart/form-data">
    <p>{{ form.user }} {{ form.errors.user.0 }}</p>  <!--{{ form.user }}生成标签，{{ form.user.errors }}生成第一个错误信息-->
    <p>{{ form.gender }} {{ form.errors.gender.0 }}</p>
    <p>{{ form.city }} {{ form.errors.city.0 }}</p>
    <p>{{ form.pwd }} {{ form.errors.pwd.0 }}</p>
    <input type="submit"/>
</form>
```
其他相关标签
```html
<form method="POST" enctype="multipart/form-data">
        {% csrf_token %}
            {{ form.xxoo.label }}
            {{ form.xxoo.id_for_label }}
            {{ form.xxoo.label_tag }}
            {{ form.xxoo.errors }}
            <p>{{ form.user }} {{ form.user.errors }}</p>
            <input type="submit" />
    </form>
```

## 2. form中字段和插件
创建Form类时，主要涉及到 【字段】 和 【插件】，字段用于对用户请求数据的验证，插件用于自动生成HTML;

### 1.Django内置字段如下：
```python
Field
            required=True,               是否允许为空
            widget=None,                 HTML插件
            label=None,                  用于生成Label标签或显示内容
            initial=None,                初始值
            help_text='',                帮助信息(在标签旁边显示)
            error_messages=None,         错误信息 {'required': '不能为空', 'invalid': '格式错误'}
            show_hidden_initial=False,   是否在当前插件后面再加一个隐藏的且具有默认值的插件（可用于检验两次输入是否一直）
            validators=[],               自定义验证规则
            localize=False,              是否支持本地化（根据不同语言地区访问用户显示不同语言）
            disabled=False,              是否可以编辑
            label_suffix=None            Label内容后缀


        CharField(Field)
            max_length=None,             最大长度
            min_length=None,             最小长度
            strip=True                   是否移除用户输入空白

        IntegerField(Field)
            max_value=None,              最大值
            min_value=None,              最小值

        FloatField(IntegerField)
            ...

        DecimalField(IntegerField)       小数，举例，涉及金钱计算保留小数点后两位
            max_value=None,              最大值
            min_value=None,              最小值
            max_digits=None,             总长度
            decimal_places=None,         小数位长度

        BaseTemporalField(Field)
            input_formats=None          时间格式化

        DateField(BaseTemporalField)    格式：2015-09-01
        TimeField(BaseTemporalField)    格式：11:12
        DateTimeField(BaseTemporalField)格式：2015-09-01 11:12

        DurationField(Field)            时间间隔：%d %H:%M:%S.%f
            ...

        RegexField(CharField)
            regex,                      自定制正则表达式
            max_length=None,            最大长度
            min_length=None,            最小长度
            error_message=None,         忽略，错误信息使用 error_messages={'invalid': '...'}

        EmailField(CharField)
            ...

        FileField(Field)
            allow_empty_file=False     是否允许空文件

        ImageField(FileField)
            ...
            注：需要PIL模块，pip3 install Pillow
            以上两个字典使用时，需要注意两点：
                - form表单中 enctype="multipart/form-data"
                - view函数中 obj = MyForm(request.POST, request.FILES)

        URLField(Field)
            ...


        BooleanField(Field)
            ...

        NullBooleanField(BooleanField)
            ...

        ChoiceField(Field)
            ...
            choices=(),                选项，如：choices = ((0,'上海'),(1,'北京'),)
            required=True,             是否必填
            widget=None,               插件，默认select插件
            label=None,                Label内容
            initial=None,              初始值
            help_text='',              帮助提示

        TypedChoiceField(ChoiceField)
            coerce = lambda val: val   对选中的值进行一次转换，通过lambda函数实现
            empty_value= ''            空值的默认值

        MultipleChoiceField(ChoiceField)多选框
            ...

        TypedMultipleChoiceField(MultipleChoiceField)
            coerce = lambda val: val   对选中的每一个值进行一次转换
            empty_value= ''            空值的默认值

        ComboField(Field)
            fields=()                  使用多个验证，如下：即验证最大长度20，又验证邮箱格式
                                       fields.ComboField(fields=[fields.CharField(max_length=20), fields.EmailField(),])

        MultiValueField(Field)
            PS: 抽象类，子类中可以实现聚合多个字典去匹配一个值，要配合MultiWidget使用，提供接口，需要自己实现

        SplitDateTimeField(MultiValueField)
            input_date_formats=None,   格式列表：['%Y--%m--%d', '%m%d/%Y', '%m/%d/%y']
            input_time_formats=None    格式列表：['%H:%M:%S', '%H:%M:%S.%f', '%H:%M']

        FilePathField(ChoiceField)     文件选项，目录下文件显示在页面中
            path,                      文件夹路径
            match=None,                正则匹配
            recursive=False,           递归下面的文件夹
            allow_files=True,          允许文件
            allow_folders=False,       允许文件夹
            required=True,
            widget=None,
            label=None,
            initial=None,
            help_text=''

        GenericIPAddressField
            protocol='both',           both,ipv4,ipv6支持的IP格式
            unpack_ipv4=False          解析ipv4地址，如果是::ffff:192.0.2.1时候，可解析为192.0.2.1， PS：protocol必须为both才能启用

        SlugField(CharField)           数字，字母，下划线，减号（连字符）
            ...

        UUIDField(CharField)           uuid类型
            ...
```
注：UUID是根据MAC以及当前时间等创建的不重复的随机字符串
```python
>>> import uuid

    # make a UUID based on the host ID and current time
    >>> uuid.uuid1()    # doctest: +SKIP
    UUID('a8098c1a-f86e-11da-bd1a-00112444be1e')

    # make a UUID using an MD5 hash of a namespace UUID and a name
    >>> uuid.uuid3(uuid.NAMESPACE_DNS, 'python.org')
    UUID('6fa459ea-ee8a-3ca4-894e-db77e160355e')

    # make a random UUID
    >>> uuid.uuid4()    # doctest: +SKIP
    UUID('16fd2706-8baf-433b-82eb-8c7fada847da')

    # make a UUID using a SHA-1 hash of a namespace UUID and a name
    >>> uuid.uuid5(uuid.NAMESPACE_DNS, 'python.org')
    UUID('886313e1-3b8a-5372-9b90-0c9aee199e5d')

    # make a UUID from a string of hex digits (braces and hyphens ignored)
    >>> x = uuid.UUID('{00010203-0405-0607-0809-0a0b0c0d0e0f}')

    # convert a UUID to a string of hex digits in standard form
    >>> str(x)
    '00010203-0405-0607-0809-0a0b0c0d0e0f'

    # get the raw 16 bytes of the UUID
    >>> x.bytes
    b'\x00\x01\x02\x03\x04\x05\x06\x07\x08\t\n\x0b\x0c\r\x0e\x0f'

    # make a UUID from a 16-byte string
    >>> uuid.UUID(bytes=x.bytes)
    UUID('00010203-0405-0607-0809-0a0b0c0d0e0f')
```

### 2.Django内置插件：
```python
TextInput(Input)  #input type="text"
NumberInput(TextInput)  # 数字输入框
EmailInput(TextInput)  # 邮箱输入框
URLInput(TextInput)  # url输入框
PasswordInput(TextInput)  # 密码输入框
HiddenInput(TextInput)  # 隐藏输入框
Textarea(Widget)  # textarea文本区
DateInput(DateTimeBaseInput)  # 日期输入框
DateTimeInput(DateTimeBaseInput)  # 日期时间输入框
TimeInput(DateTimeBaseInput)  # 时间输入框
CheckboxInput  # 多选框
Select  # 下拉框
NullBooleanSelect  # 非空布尔值下拉框
SelectMultiple  # 多选下拉框
RadioSelect  # 单选框
CheckboxSelectMultiple  # 多选checkbox ？？？
FileInput  # 文件上传
ClearableFileInput
MultipleHiddenInput  # 多隐藏输入框
SplitDateTimeWidget  # 时间分割框（两个input框）
SplitHiddenDateTimeWidget
SelectDateWidget
```
#### 常用的选择插件
```python
# 单radio，值为字符串
# user = fields.CharField(
#     initial=2,
#     widget=widgets.RadioSelect(choices=((1,'上海'),(2,'北京'),))
# )

# 单radio，值为字符串
# user = fields.ChoiceField(
#     choices=((1, '上海'), (2, '北京'),),
#     initial=2,
#     widget=widgets.RadioSelect
# )

# 单select，值为字符串
# user = fields.CharField(
#     initial=2,
#     widget=widgets.Select(choices=((1,'上海'),(2,'北京'),))
# )

# 单select，值为字符串
# user = fields.ChoiceField(
#     choices=((1, '上海'), (2, '北京'),),
#     initial=2,
#     widget=widgets.Select
# )

# 多选select，值为列表
# user = fields.MultipleChoiceField(
#     choices=((1,'上海'),(2,'北京'),),
#     initial=[1,],
#     widget=widgets.SelectMultiple
# )



# 单checkbox
# user = fields.CharField(
#     widget=widgets.CheckboxInput()
# )


# 多选checkbox,值为列表
# user = fields.MultipleChoiceField(
#     initial=[2, ],
#     choices=((1, '上海'), (2, '北京'),),
#     widget=widgets.CheckboxSelectMultiple
# )
```

在使用选择标签时，需要注意choices的选项可以从数据库中获取，但是由于是静态字段 **获取的值无法实时更新**，那么需要自定义构造方法从而达到此目的。

```python
from django.forms import Form
from django.forms import widgets
from django.forms import fields
from django.core.validators import RegexValidator

class MyForm(Form):

    user = fields.ChoiceField(
        # choices=((1, '上海'), (2, '北京'),),
        initial=2,
        widget=widgets.Select
    )

    def __init__(self, *args, **kwargs):
        super(MyForm,self).__init__(*args, **kwargs)
        # self.fields['user'].widget.choices = ((1, '上海'), (2, '北京'),)
        # 或
        self.fields['user'].widget.choices = models.Classes.objects.all().value_list('id','caption')
```

## 3. 自定义验证规则
### 方式1：在字段中自定义validators设计正则匹配
```python
from django.forms import Form
from django.forms import widgets
from django.forms import fields
from django.core.validators import RegexValidator

class MyForm(Form):
    user = fields.CharField(
        validators=[RegexValidator(r'^[0-9]+$', '请输入数字'), RegexValidator(r'^159[0-9]+$', '数字必须以159开头')],
    )
```
### 方式2：自定义规则函数处理数据
```python
import re
from django.forms import Form
from django.forms import widgets
from django.forms import fields
from django.core.exceptions import ValidationError


# 自定义验证规则
def mobile_validate(value):
    mobile_re = re.compile(r'^(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$')
    if not mobile_re.match(value):
        raise ValidationError('手机号码格式错误')


class PublishForm(Form):


    title = fields.CharField(max_length=20,
                            min_length=5,
                            error_messages={'required': '标题不能为空',
                                            'min_length': '标题最少为5个字符',
                                            'max_length': '标题最多为20个字符'},
                            widget=widgets.TextInput(attrs={'class': "form-control",
                                                          'placeholder': '标题5-20个字符'}))


    # 使用自定义验证规则
    phone = fields.CharField(validators=[mobile_validate, ],
                            error_messages={'required': '手机不能为空'},
                            widget=widgets.TextInput(attrs={'class': "form-control",
                                                          'placeholder': u'手机号码'}))

    email = fields.EmailField(required=False,
                            error_messages={'required': u'邮箱不能为空','invalid': u'邮箱格式错误'},
                            widget=widgets.TextInput(attrs={'class': "form-control", 'placeholder': u'邮箱'}))
```
### 方式3：同时生成多个标签并进行验证
```python
from django.forms import Form
from django.forms import widgets
from django.forms import fields

from django.core.validators import RegexValidator


############## 自定义字段 ##############
class PhoneField(fields.MultiValueField):
    def __init__(self, *args, **kwargs):
        # Define one message for all fields.
        error_messages = {
            'incomplete': 'Enter a country calling code and a phone number.',
        }
        # Or define a different message for each field.
        f = (
            fields.CharField(
                error_messages={'incomplete': 'Enter a country calling code.'},
                validators=[
                    RegexValidator(r'^[0-9]+$', 'Enter a valid country calling code.'),
                ],
            ),
            fields.CharField(
                error_messages={'incomplete': 'Enter a phone number.'},
                validators=[RegexValidator(r'^[0-9]+$', 'Enter a valid phone number.')],
            ),
            fields.CharField(
                validators=[RegexValidator(r'^[0-9]+$', 'Enter a valid extension.')],
                required=False,
            ),
        )
        super(PhoneField, self).__init__(error_messages=error_messages, fields=f, require_all_fields=False, *args,
                                         **kwargs)

    def compress(self, data_list):
        """
        当用户验证都通过后，该值返回给用户
        :param data_list:
        :return:
        """
        return data_list

############## 自定义插件 ##############
class SplitPhoneWidget(widgets.MultiWidget):
    def __init__(self):
        ws = (
            widgets.TextInput(),
            widgets.TextInput(),
            widgets.TextInput(),
        )
        super(SplitPhoneWidget, self).__init__(ws)

    def decompress(self, value):
        """
        处理初始值，当初始值initial不是列表时，调用该方法
        :param value:
        :return:
        """
        if value:
            return value.split(',')
        return [None, None, None]
```

## 4. 动态加载数据到form中
在Web应用程序中开发编写功能时，时常用到获取数据库中的数据并将值初始化在HTML中的标签上。
### 1.form：
```python
from django.forms import Form
from django.forms import widgets
from django.forms import fields
from django.core.validators import RegexValidator


class InitialForm(DForms.Form):
    username = fields.CharField()
    user_type = fields.IntegerField(
        widget=widgets.Select(choices=[])
    )

    def __init__(self,*args, **kwargs):
        # 执行父类的构造方法
        super(InitialForm,self).__init__(*args, **kwargs)

        self.fields['user_type'].widget.choices = models.UserType.objects.all().values_list('id','caption')
```

### 2.views:
```python

def initial(request):
    from app01 import forms
    from app01 import models
    if request.method == 'GET':
        nid = request.GET.get('nid')
        m = models.UserInfo.objects.filter(id=nid).first()
        dic = {'username': m.name, 'user_type': m.ut_id, 'favor': [1,2,3]}
        obj = forms.InitialForm(dic)
        return render(request, 'initial.html', {'obj': obj})
```
### 3.html:
```python
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    {{ obj.username }}
    {{ obj.user_type }}
</body>
</html>
```