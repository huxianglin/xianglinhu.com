# python高级之面向对象高级

## 本节内容
1. 成员修饰符
2. 特殊成员
3. 类与对象
4. 异常处理
5. 反射/自省
6. 单例模式

## 1.成员修饰符
python的类中只有私有成员和公有成员两种，不像c++中的类有公有成员（public），私有成员(private)和保护成员(protected).并且python中没有关键字去修饰成员，默认python中所有的成员都是公有成员，但是私有成员是以两个下划线开头的名字标示私有成员，私有成员不允许直接访问，只能通过内部方法去访问，私有成员也不允许被继承。

    class a:  # 说明父类的私有成员无法在子类中继承
        def __init__(self):
            self.ge=123
            self.__gene=456
    
    class b(a):
        def __init__(self,name):
            self.name=name
            self.__age=18
            super(b,self).__init__()  # 这一行会报错
        def show(self):
            print(self.name)
            print(self.__age)
            print(self.ge)
            print(self.__gene)  # 这一行也会报错
    obj=b("xiaoming")
    print(obj.name)
    print(obj.ge)
    # print(obj.__gene)  # 这个也会报错
    obj.show()

上面就是类里面的私有成员了。

## 2.特殊成员
### 1.\_\_init__
\_\_init__方法可以简单的理解为类的构造方法（实际并不是构造方法，只是在类生成对象之后就会被执行），之前已经在上一篇博客中说明过了。
### 2.\_\_del__
\_\_del__方法是类中的析构方法，当对象消亡的时候（被解释器的垃圾回收的时候会执行这个方法）这个方法默认是不需要写的，不写的时候，默认是不做任何操作的。因为你不知道对象是在什么时候被垃圾回收掉，所以，除非你确实要在这里面做某些操作，不然不要自定义这个方法。
### 3.\_\_call__
\_\_call__方法在类的对象被执行的时候（obj()或者 类()()）会执行。
### 4.\_\_int__
\_\_int\_\_方法，在对象被int()包裹的时候会被执行，例如int(obj)如果obj对象没有、\_\_int__方法，那么就会报错。在这个方法中返回的值被传递到int类型中进行转换。
### 5.\_\_str__
\_\_str__方法和int方法一样，当对象被str(obj)包裹的时候，如果对象中没有这个方法将会报错，如果有这个方法，str()将接收这个方法返回的值在转换成字符串。
### 6.\_\_add__
\_\_add__方法在两个对象相加的时候，调用第一个对象的\_\_add\_\_方法，将第二个对象传递进来，至于怎么处理以及返回值，那是程序员自定义的，就如下面的例子：

    class abc:
        def __init__(self,age):
            self.age=age
        def __add__(self,obj):
            return self.age+obj.age
    a1=abc(18)
    a2=abc(20)
    print(a1+a2)
    #执行结果：38

### 7.\_\_dict__
\_\_dict__方法在类里面有，在对象里面也有，这个方法是以字典的形式列出类或对象中的所有成员。就像下面的例子：

    class abc:
        def __init__(self,age):
            self.age=age
        def __add__(self,obj):
            return self.age+obj.age
    a1=abc(18)
    print(abc.__dict__)
    print(a1.__dict__)
    #执行结果：
    {'__add__': <function abc.__add__ at 0x0000020666C9E2F0>, '__module__': '__main__', '__weakref__': <attribute '__weakref__' of 'abc' objects>, '__init__': <function abc.__init__ at 0x0000020666C9E268>, '__doc__': None, '__dict__': <attribute '__dict__' of 'abc' objects>}
    {'age': 18}

### 8.\_\_getitem\_\_ \_\_setitem\_\_ \_\_delitem\_\_
\_\_getitem__方法匹配 对象[索引] 这种方式，\_\_setitem\_\_匹配 对象[索引]=value 这种方式，\_\_delitem\_\_匹配 del 对象[索引] 这种方式，例子如下:

    class Foo:
        def __init__(self,name,age):
            self.name=name
            self.age=age
        def __getitem__(self, item):  # 匹配:对象[item]这种形式
            return item+10
        def __setitem__(self, key, value):  # 匹配:对象[key]=value这种形式
            print(key,value)
        def __delitem__(self, key):  # 匹配:del 对象[key]这种形式
            print(key)
    
    li=Foo("alex",18)
    print(li[10])
    li[10]=100
    del li[10]
    执行结果：
    20
    10 100
    10

### 9.\_\_getslice\_\_ \_\_setslice\_\_ \_\_delslice\_\_
这三种方式在python2.7中还存在，用来对对象进行切片的，但是在python3之后，将这些特殊方法给去掉了，统一使用上面的方式对对象进行切片，因此在使用\_\_getitem\_\_ \_\_setitem\_\_ 这两个方法之前要先判断传递进参数的类型是不是slice对象。例子如下：

    class Foo:
        def __init__(self,name,age):
            self.name=name
            self.age=age
            self.li=[1,2,3,4,5,6,7]
        def __getitem__(self, item):  # 匹配:对象[item]这种形式
            if isinstance(item,slice):  # 如果是slice对象，返回切片后的结果
                return self.li[item]  # 返回切片结果
            elif isinstance(item,int):  # 如果是整形，说明是索引
                return item+10
        def __setitem__(self, key, value):  # 匹配:对象[key]=value这种形式
            print(key,value)
        def __delitem__(self, key):  # 匹配:del 对象[key]这种形式
            print(key)
        def __getslice__(self,index1,index2):
            print(index1,index2)
    
    li=Foo("alex",18)
    print(li[3:5])
    #执行结果：
    [4, 5]

### 10. \_\_iter__
类的对象如果想要变成一个可迭代对象，那么对象中必须要有\_\_iter\_\_方法，并且这个方法返回的是一个迭代器。

for 循环的对象如果是一个可迭代的对象，那么会先执行对象中的\_\_iter\_\_方法，获取到迭代器，然后再执行迭代器中的\_\_next\_\_方法获取数据。如果for循环的是一个迭代器，那么直接执行迭代器中的\_\_next\_\_方法。

    class Foo:
        def __init__(self,name,age):
            self.name=name
            self.age=age
        def __iter__(self):
            return iter([1,2,3,4,5])  # 返回的是一个迭代器
    li=Foo("alex",18)
    
    # 1.如果类中有__iter__方法，他的对象就是可迭代对象
    # 2.对象.__iter()的返回值是一个迭代器
    # 3.for循环的如果是迭代器，直接执行.next方法
    # 4.for循环的如果是可迭代对象，先执行对象.__iter()，获取迭代器再执行next
    
    for i in li:
        print(i)
    #执行结果：
    1
    2
    3
    4
    5

### 11.isinstance和issubclass
之前讲过isinstance可以判断一个变量是否是某一种数据类型，其实，isinstance不只可以判断数据类型，也可以判断对象是否是这个类的对象或者是这个类的子类的对象，代码如下：

    class Foo:
        def __init__(self,name,age):
            self.name=name
            self.age=age
    class Son(Foo):
        pass
    obj=Son("xiaoming",18)
    print(isinstance(obj,Foo))
    执行结果：True
issubclass用来判断一个类是否是某个类的子类，返回的是一个bool类型数据，代码如下：
    
    class Foo:
        def __init__(self,name,age):
            self.name=name
            self.age=age
    class Son(Foo):
        pass
    obj=Son("xiaoming",18)
    print(issubclass(Son,Foo))
    执行结果：True

## 3.类与对象
### \_\_new\_\_和\_\_metaclass\_\_
在python中，一切皆对象，我们定义的类其实。。。也是一个对象，那么，类本身是谁的对象呢？在python2.2之前（或者叫经典类中），所有的类，都是class的对象，但是在新式类中，为了将类型（int,str,float等）和类统一，所以，所有的类都是type类型的对象。当然，这个规则可以被修改，在类中有一个属性 \_\_metaclass\_\_ 可以指定当前类该由哪个类进行实例化。而创建对象过程中，其实构造器不是\_\_init\_\_方法，而是\_\_new\_\_方法，这个方法会返回一个对象，这才是对象的构造器。下面是一个解释类实例化对象内部实现过程的代码段：
![](http://7xsn7l.com2.z0.glb.clouddn.com/类实例化对象的内部实现.png)

    class Mytype(type):
        def __init__(self, what, bases=None, dict=None):
            super(Mytype,self).__init__(what, bases, dict)
        def __call__(self, *args, **kwargs):
            obj=self.__new__(self)
            self.__init__(obj, *args, **kwargs)
            return obj
    class Foo:
        __metaclass__=Mytype
        def __init__(self,name,age):
            self.name=name
            self.age=age
        def __new__(cls, *args, **kwargs):
            return object.__new__(cls)
    obj=Foo("xiaoming",18)
    print(obj.name,obj.age)
    执行结果：xiaoming 18

## 4.异常处理
python中使用try except finally组合来实现异常扑捉，不像java中是使用try catch finally......其中，except中的Exception是所有异常的父类，下面是一个异常处理的示例：

    try:
        int("aaa")  #可能出现异常的代码
    except IndexError as e:  # 捕捉索引异常的子异常，注意，这里的as e在老版本的py中可以写成，e但是新版本中用as e,",e"未来可能会淘汰
        print("IndexError:",e)
    except ValueError as e:  # 捕捉value错误的子异常
        print("ValueError:",e)
    except Exception as e:  # 如果上面两个异常没有捕获到，那么使用Exception捕获，Exception能够捕获所有的异常
        print("Exception:",e)
    else:  # 如果没有异常发生，执行else中的代码块
        print("true")
    finally:  # 不管是否发生异常，在最后都会执行finally中的代码，假如try里面的代码正常执行，先执行else中的代码，再执行finally中的代码
        print("finally")
    执行结果：
    ValueError: invalid literal for int() with base 10: 'aaa'
    finally

那么既然Exception是所有异常的父类，我们可以自已定义Exception的子类，实现自定义异常处理，下面就是实现例子：
    
    class OldBoyError(Exception):  # 自定义错误类型
        def __init__(self,message):
            self.message=message
        def __str__(self):  # 打印异常的时候会调用对象里面的__str__方法返回一个字符串
            return self.message
    try:
        raise OldBoyError("我错了...")  # raise是主动抛出异常，可以调用自定义的异常抛出异常
    except OldBoyError as e:
        print(e)
    执行结果：我错了...

异常处理里面还有一个断言，一般用在判断执行环境上面，只要断言后面的条件不满足，那么就抛出异常，并且后面的代码不执行了。

    print(123)
    assert 1==2  # 断言，故意抛出异常，做环境监测用，环境监测不通过，报错并结束程序
    print("456")
    执行结果：
        assert 1==2  # 断言，故意抛出异常，做环境监测用，环境监测不通过，报错并结束程序
    123
    AssertionError

## 5.反射/自省
python中的反射/自省的实现，是通过hasattr、getattr、setattr、delattr四个内置函数实现的，其实这四个内置函数不只可以用在类和对象中，也可以用在模块等其他地方，只是在类和对象中用的很多，所以单独提出来进行解释。

1. hasattr(key)返回的是一个bool值，判断某个成员或者属性在不在类或者对象中
2. getattr(key,default=xxx)获取类或者对象的成员或属性，如果不存在，则会抛出AttributeError异常,如果定义了default那么当没有属性的时候会返回默认值。
3. setattr(key,value)假如有这个属性，那么更新这个属性，如果没有就添加这个属性并赋值value
4. delattr(key)删除某个属性

注意，上面的key都是字符串，而不是变量，也就是说可以通过字符串处理类中的成员或者对象中的属性。下面是一个例子代码：

    class Foo:
        def __init__(self,name,age):
            self.name=name
            self.age=age
        def show(self):
            return self.name,self.age
    obj=Foo("xiaoming",18)
    print(getattr(obj,"name"))
    setattr(obj,"k1","v1")
    print(obj.k1)
    print(hasattr(obj,"k1"))
    delattr(obj,"k1")
    show_fun=getattr(obj,"show")
    print(show_fun())
    执行结果：
    xiaoming
    v1
    True
    ('xiaoming', 18)

反射/自省能够直接访问以及修改运行中的类和对象的成员和属性，这是一个很强大的功能，并且并不像java中效率很低，所以用的很多。

下面是一个反射/自省用在模块级别的例子：

    import s2
    operation=input("请输入URL:")
    if operation in s2.__dict__:
        getattr(s2,operation)()
    else:
        print("404")
    
    #模块s2中的代码：
    def f1():
        print("首页")
    def f2():
        print("新闻")
    def f3():
        print("精选")
    执行结果：
    请输入URL:f1
    首页

## 6.单例模式
这里介绍一个设计模式，设计模式在程序员写了两三年代码的时候，到一定境界了，才会考虑到设计模式对于程序带来的好处，从而使用各种设计模式，这里只是简单的介绍一个简单的设计模式：单例模式。在面向对象中的单例模式就是一个类只有一个对象，所有的操作都通过这个对象来完成，这就是面向对象中的单例模式，下面是实现代码：

    class Foo:  # 单例模式
        __v=None
        @classmethod
        def ge_instance(cls):
            if cls.__v:
                return cls.__v
            else:
                cls.__v=Foo()
                return cls.__v
    obj1=Foo.ge_instance()
    print(obj1)
    obj2=Foo.ge_instance()
    print(obj2)
    obj3=Foo.ge_instance()
    print(obj3)
    执行结果：
    <__main__.Foo object at 0x000001D2ABA01860>
    <__main__.Foo object at 0x000001D2ABA01860>
    <__main__.Foo object at 0x000001D2ABA01860>

可以看到，三个对象的内存地址都是一样的，其实，这三个变量中存储的都是同一个对象的内存地址，这样有什么好处呢？能够节省资源，就比如在数据库连接池的时候就可以使用单例模式，只创建一个类的对象供其他程序调用，还有在web服务中接收请求也可以使用单例模式来实现，节省资源。