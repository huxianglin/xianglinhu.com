# python 高级之面向对象初级
## 本节内容
1. 类的创建
2. 类的构造方法
3. 面向对象之封装
4. 面向对象之继承
5. 面向对象之多态
6. 面向对象之成员
7. property

## 1.类的创建

面向对象：对函数进行分类和封装，让开发“更快更好更强...”

在python2.7中有两种类，一种是经典类，一种是新式类。2.7中经典类和新式类区别在于类的定义的时候在类名后面加上(object)的是新式类，否则就是经典类。经典类和新式类在继承上面有区别，之后在继承的时候会提及。但是在python3.5中，已经不区分新式类和经典类了。统一都是新式类。
新式类的定义如下：

    class a:
        def fun1(self):
            print("hello world!")
    obj=a()

这样就定义好了一个基本的类，使用class 类名:定义一个类。

类的适用场景：如果在多个函数里面有多个相同的参数是，那么将这些函数封装成类将减少代码量，使逻辑更加清晰。

## 2.类的构造方法：

类定义好之后使用：变量 = 类名() 就将这个类进行实例化了。就像上面的例子中obj=a() obj就是类a的一个实例化对象，那么在类实例化的时候做了什么操作呢？首先，在解释器的命名空间里面生成一个变量，这个变量指向一个内存地址，这个内存地址就是类的实例化对象。这个对象中又有一个指针指向了类加载在内存中的地址。之后，解释器还会做一件事，就是调用这个类的构造方法，进行初始化。类的构造方法固定使用__init__()进行定义，如果在类中没有手动指定构造方法，那么python会自动生成一个构造方法，但是这个构造方法什么都不做。例子如下：

    class a():
        def __init__(self):
            print("hello world")

    obj=a()
    #执行结果: hello world

从上面的执行结果可以看出来，在类进行实例化的时候，调用了构造方法。在这里大家会注意到在__init__方法中传递了一个参数self，这个是python的类中的一个特殊参数，这个参数不一定要写成self，但是大家约定俗成的把这个变量写成self，如果写成别的名字，别人可能不一定明白。python中的self相当于c++中的this关键字，指代的是在类生成对象时这个对象的内存地址。在python的类中，普通函数都需要传递第一个参数self进去，这一块不如c++中做得好，如果不写self参数，那么是类中的一类特殊方法，这个在后面会进行讲解。虽然在定义类中的方法的时候需要在第一个参数上面写上self，但是在调用类中的方法的时候不需要传递参数给这个self，python中会自动的把这个对象的内存地址传递给self。。。个人感觉这个东西比较low。。。（不同意该观点的请轻喷。。。纯属个人观点）

## 3.面向对象之封装

封装，说的就是把某些东西封装到一个地方去，在python中，封装就是在类的对象中封装一些值，前面说过，在类生成对象时，会在解释器的命名空间中创建一个变量指向对象的内存地址，这个对象中不只是包含了类在内存空间中的地址，还可以存储一些内容。而封装就是在这个对象中存储一些内容的过程。看下面的例子：

    class a():
        def __init__(self):
            print("hello world",end="\t")

    obj=a()
    obj.name="xiaoming"
    obj.age=18
    print(obj.name,obj.age)
    #执行结果: hello world  xiaoming 18

通过对象名.属性名=值可以在对象中生成一个属性并对其赋值，这是对象的属性的一种赋值方式，还有另一种方式，那就是我们前面提到过的类的构造方法。既然对象在生成的时候会调用构造方法，那么为什么不能将对对象的属性赋值这些操作放到类的构造方法中去实现呢？答案是肯定可以的，下面就是在构造方法中对类的属性进行初始化的例子：

    class a():
        def __init__(self,name,age):
            print("hello world",end="\t")
            self.name=name
            self.age=age

    obj=a("xiaoming",18)
    print(obj.name,obj.age)
    #执行结果: hello world  xiaoming 18

上面的类的构造函数中传递进去了一些参数，这些参数被接收后可以赋值给对象，前面说过self这个特殊的参数代指的就是这个对象的内存地址，那么就相当于这个self就是obj这个对象的一个别名，那么可以直接通过self.属性名=值的方式对对象进行赋值。好了，这就是类的三大特性之一，类的封装。

## 4.面向对象之继承

在java和c++以及python中都支持类的继承，但是，java中不支持类的多继承，c++和python中是可以支持类的多继承的，但是类的多继承在有些时候可能会造成继承不明确（PS:对于不是很理解其内部原理的新手来说：就比如说我这菜鸟，哈哈），所以，java中不支持类的多继承，但是python中可以支持类的多继承，这个东西，仁者见仁智者见智吧，说不清哪个好哪个不好。前面提到过的python中的新式类和经典类在多继承的时候会有区别，将会在这一节进行讲解。

首先，我们先来介绍类的继承吧，类的继承其实就是这里有两个类a和b，每个类中都可以定义不同的方法，但是假如有些方法是相同的话，那么重复写起来比较麻烦，这样就出现了类的继承，把a当做父类（也叫基类），b当做子类（也叫派生类），这样，父类a中的所有方法在子类b中都被继承过去了，就像现实生活中的孩子继承了父亲的特性一样，但是子类不只能继承父类的方法，也能够自己定义一些方法，表现出和父类不一样的特性。具体看下面的例子：

    class a:
        def f(self):
            print("this is a.f")
    class b(a):
        def g(self):
            print("this is b.g")
    obj=b()
    obj.f()
    obj.g()
    #执行结果:this is a.f \n this is b.g

说明子类b继承了父类a中得方法f。并且子类b中也有自己的方法g。

好了，下面介绍类的多继承了，那么，什么是多继承呢？多继承就可以简单的理解为一个孩子有多个爸爸，就像有一个隔壁老王一样，哈哈。
多继承其实就是可以继承多个父类的方法，下面是多继承的例子：

    class a:
        def f(self):
            print("this is a.f")
    class b:
        def g(self):
            print("this is b.g")
    class c(a,b):
        def h(self):
            print("this is c.h")
    obj=c()
    obj.f()
    obj.g()
    obj.g()
    #执行结果:this is a.f \n this is b.g \n this is c.h

既然有多继承存在的话，那么就存在继承顺序的问题。

1. 在python的经典类中，继承顺序会按照深度优先方式去继承。
2. 在python的新式类中，继承顺序会按照广度优先方式去继承。

其实上面说的两句话并不全对，因为这里的深度优先和广度优先和图以及树中的概念有些不一样，举个例子来说明吧：

    class a:
        def f(self):
            print("this is a.f")
    class b(a):
        def f(self):
            print("this is b.f")
    class c(a):
        def f(self):
            print("this is c.f")
    class d(b,c):
        pass
就像这个例子中，d类是b和c的子类，那b和c又是a的子类，在调用方法f的时候会调用哪个方法呢？用下面这个图来解释吧：
![](http://7xsn7l.com2.z0.glb.clouddn.com/python%E4%B8%AD%E7%BB%8F%E5%85%B8%E7%B1%BB%E4%B8%8E%E6%96%B0%E5%BC%8F%E7%B1%BB%E5%8C%BA%E5%88%AB.png)

上面就解释了经典类和新式类在多继承中的区别了。那么既然有继承，那么我又想继承父类，但是父类中的某些方法又不适应我子类中的场景，那么就可以对父类中的方法进行重写了，重写就是在子类中定义一个和父类中相同名字的方法名，并在里面实现想要实现的功能。这样就完成了重写，再调用子类的这个方法的时候就不会调用父类的方法了。例子如下：

    class father:
        def f1(self):
            print("f1.1")
        def f2(self):
            print("f2.2")
    class son(father):
        def f1(self):
            print("s1.1")
    
    z=son()
    z.f1()
    z.f2()
    #执行结果:s1.1 \n f2.2

那么，既然能够重写父类的方法，能不能在重写父类方法的时候调用父类方法呢？当然是可以的，在子类重写父类方法的时候调用父类方法有两种方式，一种是使用super关键字，还有一种是直接通过父类名.方法名调用，推荐使用第一种，例子如下:

    class father:
        def f1(self):
            print("f1.1")
        def f2(self):
            print("f2.2")
    class son(father):
        def s1(self):
            print("s1.1")
        def f1(self):
            print("s1.2")
            super(son,self).f1()  # 调用父类方法1 推荐
            father.f1(self)  # 调用父类方法2 不推荐
    
    z=son()
    z.f1()
    #执行结果：
    s1.2
    f1.1
    f1.1
使用super的时候格式是super(子类名,self).父类方法名。
好了，多继承里面还剩下一个东西，那就是构造方法了，多继承里面构造方法的找寻规则和普通方法的找寻规则是一样的，直到找到第一个父类构造方法，执行这个父类的构造方法，但是这里有一个问题我至今还没搞清楚，那就是在钻石继承里面的构造方法中调用super关键字的执行结果：

    class A(object):
        def __init__(self):
            print("In A's __init__()")
    
    
    class B(A):
        def __init__(self):
            print("Enter B's __init__()")
            super(B, self).__init__()
            print("Leave B's __init__()")
    
    
    class C(A):
        def __init__(self):
            print("Enter C's __init__()")
            super(C, self).__init__()
            print("Leave C's __init__()")
    
    
    class D(B, C):
        pass
    
    
    d = D()
    执行结果：
    Enter B's __init__()
    Enter C's __init__()
    In A's __init__()
    Leave C's __init__()
    Leave B's __init__()

上面的执行结果。。。抱歉，我至今也没法解释，诶，本人水平还没到那个境界，心塞中。。。

## 5.面向对象之多态

在其他语言中，类似于c++和java这一类强类型的语言中，函数的参数规定死了是什么数据类型就必须传递这种数据类型的数据进来，否则就会报错，但是在python中内置的就是支持多态的，python是一门弱类型语言，在函数的定义的参数中不需要指定参数的数据类型，可以传递任意数据类型的参数到函数中去。所以对于python来说不存在多态的问题，那么在java和c++中的多态是什么意思呢？说的是函数的参数中指定了参数类型是父类的对象，那么这个父类的子类的对象也可以传递进这个函数中，这样就实现了参数的多态性。

## 6.面向对象之成员

python的类中的所有东西都可以统称为类的成员，成员包含成员方法和成员字段。之前只是介绍了一部分的成员方法和字段，下面来详细说明类的成员方法和字段。

    成员字段
        - 普通字段，保存在对象中，执行只能通过对象访问
        - 静态字段，保存在类中，  执行 可以通过对象访问 也可以通过类访问
        
    成员方法
        - 普通方法，保存在类中，由对象来调用，self-->对象
        - 静态方法，保存在类中，由类直接调用
        - 类方法，保存在类中，由类直接调用，cls-->当前类

成员字段
普通字段就是之前说的对象中的字段，这些字段是保存在对象中的，只能通过对象访问
静态字段是在类定义的时候定义的，保存在类中，可以被类调用，也可以被对象调用

成员方法
普通方法就是之前说的在方法中含有self参数的一类方法
静态方法就是使用@staticmethod方法装饰的一类方法，这种方法不需要传递对象进来
类方法是使用@classmethod方法装饰的一类方法，这类方法里面必须带一个参数，默认使用cls，这个参数代指类本身

## 7.property
在定义普通方法的时候，在上面加上@property装饰器装饰时，生成的是一个类似于属性的方法。

    class foo:
        @property  # 属性，把方法当做属性来用，介于字段和方法之间
        def stat(self):
            return 1
        @stat.setter
        def stat(self,key):
            print(key)
        @stat.deleter
        def stat(self):
            print("abc")
    
    obj=foo()
    r=obj.stat
    print(r)
    obj.stat=123
    del obj.stat
    #执行结果：1 \n 123  \n  abc

类似于这样的有三个对应关系：
@property --> result=对象.方法
@方法.setter --> 对象.方法=key
@方法.delter  --> del 对象.方法
这样执行，方法就具有了字段一样的特点。
上面这种方式还可以使用静态字段的方式达到相同效果，代码如下。

    class foo:
        
        def get_stat(self):
            return 1
    
        def set_stat(self,key):
            print(key)
    
        def del_stat(self):
            print("abc")
        BAR=property(get_stat,set_stat,del_stat)
    obj=foo()
    r=obj.BAR
    print(r)
    obj.BAR=123
    del obj.BAR
    #执行结果：1 \n 123  \n  abc

上面的BAR=property(get_stat,set_stat,del_stat)里面的参数是按照顺序排列的，取值，设置值和删除值。