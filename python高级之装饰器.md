# python高级之装饰器

## 本节内容
1. 高阶函数
2. 嵌套函数及闭包
3. 装饰器
4. 装饰器带参数
5. 装饰器的嵌套
6. functools.wraps模块
7. 递归函数被装饰

## 1.高阶函数
高阶函数的定义：

满足下面两个条件之一的函数就是高阶函数：

- 接受一个或多个函数作为输入参数
- 输出一个函数

首先理解一个概念：函数名其实也是一个变量，一个函数其实就是一个对象，函数名就是对这个对象的引用。所以函数名也就和一个普通变量一样可以被当做函数的变量进行传递，当然也能够把函数名当做一个变量进行返回。

举个栗子：

	def foo(func,x,y):
	    return func(x,y)
	def add(x,y):
	    return x+y
	print(foo(add,3,4))
	#************************
	def foo():
	    x=3
	    def bar():
	        return x
	    return bar　
	a=foo()
	print(a())
上面的就是一些高阶函数。

## 2.嵌套函数及闭包
上面的例子里面的第二个例子，a=foo()的时候就是执行foo函数并且返回bar对象给a,这时候foo函数已经执行完了，a得到的是bar函数对应的内存地址。那么执行a()函数的时候为什么能够得到x的值呢？明明foo函数已经在前面执行完了，之后执行的是bar函数，但是bar函数能够获取到foo函数中定义的变量。

这种在内部嵌套函数里面能够获取到外部函数中的变量的现象在python里面叫做闭包，下面来说说闭包的定义：

定义：如果在一个内部函数里，对在外部作用域（但不是在全局作用域）的变量进行引用，那么内部函数就被认为是闭包(closure)。也可以叫做：闭包=函数块+定义函数时的环境。

## 3.装饰器
装饰器本质上是一个函数，该函数用来处理其他函数，它可以让其他函数在不需要修改代码的前提下增加额外的功能，装饰器的返回值也是一个函数对象。它经常用于有切面需求的场景，比如：插入日志、性能测试、事务处理、缓存、权限校验等应用场景。装饰器是解决这类问题的绝佳设计，有了装饰器，我们就可以抽离出大量与函数功能本身无关的雷同代码并继续重用。概括的讲，装饰器的作用就是为已经存在的对象添加额外的功能。

假如我们有一个之前已经写好了功能的函数，现在又想在上面添加计时功能，想统计这个函数执行花费了多少时间，但是又因为这个函数是一个基础函数，被很多其他地方调用了，所以不能修改这个函数中的代码。（这就是开发中的开放封闭原则：即写的代码对于需求来说是开放的，也就是说可以在原有需求上面添加需求，但是对于源代码来说又是封闭的，一个完成功能的函数，不能被修改。）这时候该怎么办呢？

举个栗子：

	def add(*args,**kwargs):
	    Sum = reduce(lambda x,y:x+y ,args)
	    print(Sum)
这是一个计算加法的函数，那么怎么给这个做加法的函数统计时间呢？

这时，我们想到了可以使用另一个函数调用这个函数，再去统计这个函数执行的时间。

栗子如下：

	import time
	def add(*args,**kwargs):
		    Sum = reduce(lambda x,y:x+y ,args)
		    print(Sum)

	def show_time(*args,**kwargs):
		start_time=time.time()
		add(*args,**kwargs)
		end_time=time.time()
		print("计算时间:%s" % (end_time-start_time))

	show_time()
这样，就能够统计到函数的执行时间了，但是。。。别人调用函数的时候就需要修改函数名了，那么该怎么办呢？这时候我们想到了闭包。将上面的函数进行改造下，成了下面的方式：

	import time
	def add(*args,**kwargs):
		    Sum = reduce(lambda x,y:x+y ,args)
		    print(Sum)

	def show_time(func):
		def wrapper(*args,**kwargs)
			start_time=time.time()
			func(*args,**kwargs)
			end_time=time.time()
			print("计算时间:%s" % (end_time-start_time))
		return wrapper

	wrap=show_time(add)
这时候执行show_time函数之后返回的是什么呢？没错，返回的是一个wrapper函数对象，这时候执行这个函数对象，是不是就能够既能够获取到函数执行时间，又能够执行函数了呢？

下面我们再对上面的东西进行改造一下，效果如下：

	import time

	def show_time(func):
		def wrapper(*args,**kwargs)
			start_time=time.time()
			func(*args,**kwargs)
			end_time=time.time()
			print("计算时间:%s" % (end_time-start_time))
		return wrapper

	def add(*args,**kwargs):
		    Sum = reduce(lambda x,y:x+y ,args)
		    print(Sum)
	add=show_time(add)
	add(1,2,3,4,5)
这时候是不是又没有改变函数调用名称，又能够给这个函数加上一个计算时间的功能呢？

其实上面的例子就是装饰器的概念，只不过在python中使用@语法糖，可以将add=show_time(add)这一句隐士的表达出来罢了，下面是真正的实现一个简单的装饰器了。


	import time

	def show_time(func):
		def wrapper(*args,**kwargs)
			start_time=time.time()
			func(*args,**kwargs)
			end_time=time.time()
			print("计算时间:%s" % (end_time-start_time))
		return wrapper
	@show_time  # 其实这一句的意思就是：add=show_time(add)
	def add(*args,**kwargs):
		    Sum = reduce(lambda x,y:x+y ,args)
		    print(Sum)
	
	add(1,2,3,4,5)
上面就是python里面实现的一个简单的装饰器了。

## 4.装饰器带参数

既然，我们可以使用装饰器，对带参数的函数进行装饰，那么，我们能不能对装饰器带上参数，使装饰器更加灵活呢？

答案是可以的，那就是在装饰器的外面再嵌套一层函数。

现在在之前的需求上再增加一个需求，在装饰器中传递一个参数，实现可以通过这个参数决定是否将计算时间写入到文件中去，下面是实现的代码：


	import time
	def logger(flag=Falce):
		def show_time(func):
			def wrapper(*args,**kwargs)
				start_time=time.time()
				func(*args,**kwargs)
				end_time=time.time()
				print("计算时间:%s" % (end_time-start_time))
				if flag:
					print("写入日志文件")
				else:
					print("不写入日志文件")
			return wrapper
		return showtime

	@logger(flag=True)  # 首先会执行logger函数，返回show_time对象，然后@show_time就和之前的装饰器一样了
	def add(*args,**kwargs):
		    Sum = reduce(lambda x,y:x+y ,args)
		    print(Sum)
	
	add(1,2,3,4,5)

上面的logger函数接受参数，执行这个函数之后，返回的是一个装饰器对象，但是装饰器对象是一个闭包，可以拿到logger函数传递进来的参数。其内部的wrapper函数也能够拿到logger中传递进来的参数。

## 5.装饰器的嵌套

可不可以对装饰器进行多层嵌套呢？一层装饰器实现一个新功能，这样，想实现什么功能，就在外面再套一层装饰器罢了。

下面是两个装饰器嵌套的源代码：

	def tag2(func):  # 对返回的函数值进行第二次装饰
	    def wrapper(*args,**kwargs):
	        rst=func(*args,**kwargs)
	        rst="".join(("<p>",rst,"</p>"))
	        return rst
	    return wrapper
	
	def tag1(func):  # 对返回的函数值进行第一次装饰
	    def wrapper(*args,**kwargs):
	        rst=func(*args,**kwargs)
	        rst="".join(("<a>",rst,"</a>"))
	        return rst
	    return wrapper
	
	@tag2
	@tag1
	def helloworld():
	    return "hello world"
	
	print(helloworld())

如上所示，定义了两个装饰器，tag1装饰器对helloworld函数进行装饰，在返回的helloworld字符串外面嵌套一层a标签，然后在tag2装饰器上，对前面使用第一个装饰器装饰过了的函数看成一个整体，再进行装饰，在最外面再嵌套一层p标签，这就是装饰器的嵌套。。。越上面的装饰器，表示越在外面，先执行里面的再执行外面的。

## 6.functools.wraps模块

使用装饰器极大地复用了代码，但是他有一个缺点就是原函数的元信息不见了，比如函数的docstring、__name__、参数列表，先看例子：

    def foo():
        print("hello foo")
    
    print(foo.__name__)
    #####################
    
    def logged(func):
        def wrapper(*args, **kwargs):
    
            print (func.__name__ + " was called")
            return func(*args, **kwargs)
    
        return wrapper
    
    
    @logged
    def cal(x):
       return x + x * x
    
    
    print(cal.__name__)
    
    ########
    # foo
    # wrapper

解释：

    @logged
    def f(x):
       return x + x * x
    等价于：
    
    def f(x):
        return x + x * x
    f = logged(f)
    不难发现，函数f被wrapper取代了，当然它的docstring，__name__就是变成了wrapper函数的信息了。
    
    print f.__name__    # prints 'wrapper'
    print f.__doc__     # prints None

这个问题就比较严重的，好在我们有functools.wraps，wraps本身也是一个装饰器，它能把原函数的元信息拷贝到装饰器函数中，这使得装饰器函数也有和原函数一样的元信息了。

    from functools import wraps
     
    def logged(func):
     
        @wraps(func)
     
        def wrapper(*args, **kwargs):
            print (func.__name__ + " was called")
            return func(*args, **kwargs)
        return wrapper
     
    @logged
    def cal(x):
       return x + x * x
     
    print(cal.__name__)  #cal

## 7.递归函数被装饰

在实际使用过程中发现，在递归函数中使用普通的装饰器得到的并不是我们想要的结果。

    ##----------------------------------------foo函数先加载到内存,然后foo变量指向新的引用,所以递归里的foo是wrapper函数对象
    # def show_time(func):
    #
    #     def wrapper(n):
    #         ret=func(n)
    #         print("hello,world")
    #         return ret
    #     return wrapper
    #
    # @show_time# foo=show_time(foo)
    # def foo(n):
    #     if n==1:
    #         return 1
    #     return n*foo(n-1)
    # print(foo(6))
    
这种现象是不对的，foo变量已经指向了wrapper函数在内存中的地址了。要解决这种问题必须在wrapper里面接收原func返回的结果，并把这个结果返回回去，代码如下所示。还有一种方式就是用另一个普通函数将递归函数包裹起来，然后使用普通装饰器去装饰该普通函数。

    def show_time(func):
    
        def wrapper(n):
            ret=func(n)
            print("hello,world")
            return ret
        return wrapper
    
    @show_time# foo=show_time(foo)
    def foo(n):
        def _foo(n):
            if n==1:
                return 1
            return n*_foo(n-1)
        return _foo(n)
    print(foo(6))