# 杂项之python描述符协议
## 本节内容
1. 由来
2. 描述符协议概念
3. 类的静态方法及类方法实现原理
4. 类作为装饰器使用

## 1. 由来
闲来无事去看了看django中的内置分页方法，发现里面用到了类作为装饰器来使用，由于之前就看到过这一类的用法，但是一直没有明白具体是如何实现的，今天本着打破砂锅问到底的精神去网上搜资料，在这里不得不吐槽下百度搜索的垃圾了。。。。。竞价排名做的那么6，搜一些技术文档。。。。。各种坑爹。。。就是找不到想要的资源。。。
于是翻墙上google搜了搜，找到了python官网的文档。。。里面详细解释了原理以及用法，表示英语渣是硬伤。。。。囫囵吞枣的理解了里面的一部分内容。。。于是就有了本文的诞生。。。。。python官网文档连接在这：
[python官方文档地址](https://docs.python.org/2/howto/descriptor.html)
英语比较好的小伙伴可以去拜读下。

## 2. 描述符协议概念
在python中有个概念叫做描述符（descriptors）
描述符在python中是一个协议，协议的定义如下：
如果为某个对象定义了\_\_get\_\_，\_\_set\_\_及\_\_delete\_\_中的任意一种方法，那么就称这个对象是描述符。
描述符是一个功能强大的通用协议，类中的**属性，方法，静态方法，类方法甚至super()背后都是基于描述符**它是在python2.2中引入的新式类中实现的，描述符使底层的C代码实现更简单，并且为python编程提供了一个灵活的新工具。

官方说明如下：
```python
descr.__get__(self, obj, type=None) --> value

descr.__set__(self, obj, value) --> None

descr.__delete__(self, obj) --> None
```
如果对象中定义了\_\_get\_\_()和\_\_set\_\_()，它被认为是一个数据描述符。只定义\_\_get\_\_()的描述符称为非数据描述符（它们通常用于方法，但其他用途也是可能的）。
数据和非数据描述符的区别在如何重写对象的字典计算实例。
- 如果实例字典具有与数据描述符相同的项，则数据描述符将优先。
- 如果实例字典具有与非数据描述符同名的项，则字典条目优先。

描述符的调用：
描述符在被调用时区别在于调用描述符的是对象还是类：
对于对象：
- 数据描述符被优先调用
- 非数据描述符可能会被实例字典所覆盖

解释：
非数据描述符在对象调用中可能会被实例字典覆盖的举例：
```python
#-------------------------------------被实例的字典覆盖的例子-------------------------------------------------------------
class RevealAccess:
    """A data descriptor that sets and returns values
       normally and prints a message logging their access.
    """

    def __init__(self, initval=None, name='var'):
        self.val = initval
        self.name = name

    def __get__(self, obj, objtype):
        print('Retrieving', self.name)
        return self.val

class MyClass:
    x=RevealAccess(10,'var"x"')

m=MyClass()
m.x=3
print(m.x)
# 3
# 这里的非数据描述符被实例中的字典给覆盖了，当调用实例m的属性x的时候，x对应的是数字3

#---------------------------------------数据描述符被优先调用的例子-------------------------------------------------------
class RevealAccess:
    """A data descriptor that sets and returns values
       normally and prints a message logging their access.
    """

    def __init__(self, initval=None, name='var'):
        self.val = initval
        self.name = name

    def __get__(self, obj, objtype):
        print('Retrieving', self.name)
        return self.val

    def __set__(self, obj, val):
        print('Updating', self.name)
        print(obj,val)
        self.val = val

class MyClass:
    x=RevealAccess(10,'var"x"')

m=MyClass()
m.x=3
print(m.x)
# Updating var"x"
# <__main__.MyClass object at 0x0000021E5F33A7B8> 3
# Retrieving var"x"
# 3

# 这里的数据描述符无法被实例中的字典覆盖。。。当实例m给属性x赋值的时候，实际是调用了x对应的RevealAccess对象的__set__方法给属性x赋值，这里的数据描述符的优先级比对象的属性的优先级更高
```

会将对象b.x 转换成 type(b).\_\_dict\_\_['x'].\_\_get\_\_(obj, type(b))
对于类：
会将class.x 转换成 class.\_\_dict\_\_['x'].\_\_get\_\_(None, class)

python模拟\_\_getattribute\_\_()的实现
```python
def __getattribute__(self, key):
    "Emulate type_getattro() in Objects/typeobject.c"
    v = object.__getattribute__(self, key)
    if hasattr(v, '__get__'):
        return v.__get__(None, self)
    return v
```
需要重点记住的点：
- 描述符只有在\_\_getattribute\_\_()的时候才会被调用
- 重写\_\_get\_\_将会覆盖默认的\_\_getattribute\_\_()中的调用
- \_\_getattribute\_\_()只适用于新式类和对象
- object.\_\_getattribute\_\_() 和 type.\_\_getattribute\_\_()调用\_\_get\_\_()的方式不同
- 数据描述符总是覆盖实例中的字典
- 非数据描述符可能被实例字典所覆盖

例子：
```python
class RevealAccess:
    """A data descriptor that sets and returns values
       normally and prints a message logging their access.
    """

    def __init__(self, initval=None, name='var'):
        self.val = initval
        self.name = name

    def __get__(self, obj, objtype):
        print('Retrieving', self.name)
        return self.val

    def __set__(self, obj, val):
        print('Updating', self.name)
        print(obj,val)
        self.val = val

class MyClass:
    x=RevealAccess(10,'var"x"')

m=MyClass()
print(m.x)
# Retrieving var"x"  通过对象调用属性x，将会执行RevealAccess对象的__get__方法(描述符)。
# 10
m.x = 20  # 将20赋值给MyClass的对象m的属性x-->RevealAccess对象-->调用RevealAccess对象的__set__方法
# Updating var"x"
# <__main__.MyClass object at 0x0000013064DAA7F0> 20  # __set__方法接收的第一个参数是MyClass对象，第二个参数是=后面的值
print(m.x)
# Retrieving var"x"
# 20
```
## 3. 类的静态方法及类方法实现原理
类的静态方法及类方法作用可以参考我之前的博客：
[python 高级之面向对象初级](http://www.cnblogs.com/huxianglin/p/5906455.html)

类的静态方法实现原理：
```python
class StaticMethod(object):
    "Emulate PyStaticMethod_Type() in Objects/funcobject.c"

    def __init__(self, f):
        self.f = f

    def __get__(self, obj, objtype=None):
        return self.f

class E(object):
	@staticmethod
    def f(x):
        print x
    # 上面的装饰器相当于 f = staticmethod(f)

print E.f(3)
# 3
print E().f(3)
# 3
```

类方法实现原理：
```python
class ClassMethod(object):
    "Emulate PyClassMethod_Type() in Objects/funcobject.c"
    def __init__(self, f):
        self.f = f

    def __get__(self, obj, klass=None):
        if klass is None:
            klass = type(obj)
        def newfunc(*args):
            return self.f(klass, *args)
        return newfunc

class Dict(object):

	@classmethod
    def fromkeys(klass, iterable, value=None):
        "Emulate dict_fromkeys() in Objects/dictobject.c"
        d = klass()
        for key in iterable:
            d[key] = value
        return d
    # 上面的装饰器相当于 fromkeys = classmethod(fromkeys)

print(Dict.fromkeys('abracadabra'))
# {'a': None, 'r': None, 'b': None, 'c': None, 'd': None}

#执行Dict.fromkeys相当创建了classmethod(fromkeys)对象，并执行了该对象的__get__方法，该方法返回一个新的函数，而后面的('abracadabra')相当于传递参数'abracadabra'执行了这个新函数，返回的是fromkeys的执行结果。
```
## 4. 类作为装饰器使用
代码如下：
```python
class MyClassDescriptor:
    def __init__(self, func, name=None):
        self.func = func
        self.name = name or func.__name__
        self.__doc__=getattr(func, '__doc__')

    def __get__(self,instance,cls=None):
        if instance is None:
            print(instance,cls)
            return self
        print(instance,cls)
        res = instance.__dict__[self.name] = self.func(instance)
        return res

class MyClass:
    @MyClassDescriptor
    def foo(self):
        print("MyClass foo")
	# foo=MyClassDescriptor(foo)

m.foo
# <__main__.MyClass object at 0x000001BE9597A828> <class '__main__.MyClass'>  # 传递进__get__方法的两个参数，第一个是MyClass对象m，第二个是MyClass类的内存地址，在__get__方法中返回的是foo函数的执行结果，也就打印出来了'MyClass foo'
# MyClass foo
print(MyClass.foo)
# None <class '__main__.MyClass'>  # 当是类调用foo时，传递进__get__方法中的第一个参数为None,第二个参数是MyClass类的内存地址
# <__main__.MyClassDescriptor object at 0x000001BE9597A7F0>
```