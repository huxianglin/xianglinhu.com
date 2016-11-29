# python高级之函数

## 本节内容
1. 函数的介绍
2. 函数的创建
3. 函数参数及返回值
4. LEGB作用域
5. 特殊函数
6. 函数式编程

## 1.函数的介绍
为什么要有函数？因为在平时写代码时，如果没有函数的话，那么将会出现很多重复的代码，这样代码重用率就比较低。。。并且这样的代码维护起来也是很有难度的，为了解决这些问题，就出现了函数，用来将一些经常出现的代码进行封装，这样就可以在任何需要调用这段代码的地方调用这个函数就行了。

函数的定义：函数是指将一组语句的集合通过一个名字(函数名)封装起来，要想执行这个函数，只需调用其函数名即可

特性：

1. 代码重用
2. 保持一致性
3. 可扩展性

## 2.函数的创建
在python中函数定义的格式如下：

	def 函数名(形参):
		函数体内部代码块

函数的调用使用  函数名(实参) 就可以调用函数了。

函数名的命名规则和变量的命名规则一样：

- 函数名必须以下划线或字母开头，可以包含任意字母、数字或下划线的组合。不能使用任何的标点符号；
- 函数名是区分大小写的。
- 函数名不能是保留字。

形参和实参的区别：

函数在定义的时候，函数名后面的括号中可以添加参数，这些参数就叫做形参，形参：顾名思义就是形式参数，只是一个代号。

实参是在调用函数的时候函数名后面的括号中的参数，形参和实参需要一一对应起来，否则调用函数会报错。

## 3.函数参数及返回值
前面提到函数的形参和实参要一一对应，那么参数对应有如下几种：

1. 必须参数
2. 关键字参数
3. 默认参数
4. 不定长参数 *args
5. 不定长参数 **kwargs

### 1.必须参数：
必须参数必须以对应的关系一个一个传递进入函数，函数调用时传递的实参必须和函数定义时的形参一一对应，不能多也不能少，顺序也得一致。

举个栗子：

	def f(name,age):
		print(name,age)
	f("小明",18)

### 2.关键字参数
关键字参数是实参里面的概念，在调用函数的时候声明某个参数是属于某个关键字的。使用关键字参数允许函数调用时参数的顺序与声明时不一致，因为 Python 解释器能够用参数名匹配参数值。

举个栗子：

	def f(name,age):
		print(name,age)
	f(name="小明",18)

### 3.默认参数
默认参数是在函数声明的时候，可以给某个参数指定默认值，这样的参数叫做默认值参数。如果在调用函数的时候，默认参数没有接收到对应的实参，那么就会将默认值赋值给这个参数。

举个栗子：

	def f(name,age,sex="male"):
		print(name,age,sex)
	f(name="小明",18)

这样，就会把默认参数male赋值给sex了。

### 4.不定长参数 *args
在python里面，函数在声明的时候，参数中可以使用(\*变量名)的方式来接受不确定长度的参数，但是在python里面大家约定俗成使用\*args接受不定长参数，这样在调用函数的时候传递的参数就可以是不定长度的了。args接受了不定长参数之后，将这些参数放到一个tuple里面，可以通过访问args来获取这些不定长参数。

举个栗子：

	def f(*args):
		print(args)
	f("小明",18,"male")

打印出来的是一个tuple，里面存放了("小明",18,"male")这三个元素。

### 不定长参数 **kwargs
但是上面的args只能接收未命名的参数，那假如有类似于关键字参数的不定长参数该怎么办呢？python里面使用(**变量名)来接收不定长的命名变量参数。同样，python里面也约定俗成使用\*\*kwargs接收不定长命名参数。kwargs接收了不定长参数之后，将这些参数放到一个字典里面，可以通过key获取到相应的参数值。

举个栗子：

	def f(**kwargs):
		print(kwargs)
	f(name="小明",age=18,sex="male")

介绍完了这些参数之后，接下来要介绍的是关于这些参数混合使用的情况：

假如一个函数使用了上面所有种类的参数，那该怎么办？为了不产生歧义，python里面规定了假如有多种参数混合的情况下，遵循如下的顺序使用规则：

	def f(必须参数,默认参数,*args,**kwargs)：
		pass

如果同时存在*args和**kwargs的话，*args在左边

默认参数在必须参数的右边，在*args的左边

关键字参数的位置不固定(ps:关键字参数也不在函数定义的时候确定)

那么，假如有一个列表想要传递进入一个不定长的未命名参数的函数中去，可以在该列表前面加上\*实现，同理如果想传递一个字典进入不定长命名参数的函数中去，可以在该字典前面加上\*\*

举个栗子：

	def f(*args,**kwargs):
		print(args)
		for i in kwargs:
			print("%s:%s"%(i,kwargs[i]))

	f(*[1,2,3],**{"a":1,"b":2})

### 函数的返回值
要想获取函数的执行结果，就可以用return语句把结果返回

注意:

函数在执行过程中只要遇到return语句，就会停止执行并返回结果，也可以理解为 return 语句代表着函数的结束
如果未在函数中指定return,那这个函数的返回值为None  
return多个对象，解释器会把这多个对象组装成一个元组作为一个一个整体结果输出。

## 4.LEGB作用域
python中的作用域分4种情况：

L：local，局部作用域，即函数中定义的变量；

E：enclosing，嵌套的父级函数的局部作用域，即包含此函数的上级函数的局部作用域，但不是全局的；

G：globa，全局变量，就是模块级别定义的变量；

B：built-in，系统固定模块里面的变量，比如int, bytearray等。 搜索变量的优先级顺序依次是：作用域局部>外层作用域>当前模块中的全局>python内置作用域，也就是LEGB。

local和enclosing是相对的，enclosing变量相对上层来说也是local。

在Python中，只有模块（module），类（class）以及函数（def、lambda）才会引入新的作用域，其它的代码块（如if、try、for等）不会引入新的作用域。

### 变量的修改（错误修改，面试题里经常出）：

	x=6
	def f2():
	    print(x)
	    x=5
	f2()
	  
	# 错误的原因在于print(x)时,解释器会在局部作用域找,会找到x=5(函数已经加载到内存),但x使用在声明前了,所以报错:
	# local variable 'x' referenced before assignment.如何证明找到了x=5呢?简单:注释掉x=5,x=6
	# 报错为:name 'x' is not defined
	#同理
	x=6
	def f2():
	    x+=1 #local variable 'x' referenced before assignment.
	f2()

### global关键字 
当内部作用域想修改外部作用域的变量时，就要用到global和nonlocal关键字了，当修改的变量是在全局作用域（global作用域）上的，就要使用global先声明一下，代码如下：

	count = 10
	def outer():
	    global count
	    print(count) 
	    count = 100
	    print(count)
	outer()

### nonlocal关键字 
global关键字声明的变量必须在全局作用域上，不能嵌套作用域上，当要修改嵌套作用域（enclosing作用域，外层非全局作用域）中的变量怎么办呢，这时就需要nonlocal关键字了

	def outer():
	    count = 10
	    def inner():
	        nonlocal count
	        count = 20
	        print(count)
	    inner()
	    print(count)
	outer()

### 小结
1. 变量查找顺序：LEGB，作用域局部>外层作用域>当前模块中的全局>python内置作用域；

2. 只有模块、类、及函数才能引入新作用域；

3. 对于一个变量，内部作用域先声明就会覆盖外部变量，不声明直接使用，就会使用外部作用域的变量；

4. 内部作用域要修改外部作用域变量的值时，全局变量要使用global关键字，嵌套作用域变量要使用nonlocal关键字。nonlocal是python3新增的关键字，有了这个 关键字，就能完美的实现闭包了。 

## 5.特殊函数
递归函数定义：递归函数就是在函数内部调用自己

有时候解决某些问题的时候，逻辑比较复杂，这时候可以考虑使用递归，因为使用递归函数的话，逻辑比较清晰，可以解决一些比较复杂的问题。但是递归函数存在一个问题就是假如递归调用自己的次数比较多的话，将会使得计算速度变得很慢，而且在python中默认的递归调用深度是1000层，超过这个层数将会导致“爆栈”。。。所以，在可以不用递归的时候建议尽量不要使用递归。

举个栗子：
	
	def factorial(n):  # 使用循环实现求和
	    Sum=1
	    for i in range(2,n+1):
	        Sum*=i
	    return Sum
	print(factorial(7))
	
	def recursive_factorial(n):  # 使用递归实现求和
	    return (2 if n==2 else n*recursive_factorial(n-1))
	
	print(recursive_factorial(7))
	
	def feibo(n):  # 使用递归实现菲波那切数列
	    if n==0 or n==1:return n
	    else:return feibo(n-1)+feibo(n-2)
	print(feibo(8))
	
	def feibo2(n):  # 使用循环实现菲波那切数列
	    before,after=0,1
	    for i in range(n):
	        before,after=after,before+after
	    return before
	print(feibo2(300))

递归函数的优点:定义简单，逻辑清晰。理论上，所有的递归函数都可以写成循环的方式，但循环的逻辑不如递归清晰。

递归特性:

1. 必须有一个明确的结束条件

2. 每次进入更深一层递归时，问题规模相比上次递归都应有所减少

3. 递归效率不高，递归层次过多会导致栈溢出(在计算机中，函数调用是通过栈（stack）这种数据结构实现的，每当进入一个函数调用，栈就会加一层栈帧，每当函数返     回，栈就会减一层栈帧。由于栈的大小不是无限的，所以，递归调用的次数过多，会导致栈溢出。)

## 6.函数式编程


关于函数式编程，我理解的也不是很深，但是python中有4个比较重要的内置函数，组合起来使用有时候能大大提高编程效率。

1 filter(function, sequence)


	str = ['a', 'b','c', 'd']
	def fun1(s):
	    if s != 'a':
	        return s
	ret = filter(fun1, str)
 
print(list(ret))# ret是一个迭代器对象       
对sequence中的item依次执行function(item)，将执行结果为True的item做成一个filter object的迭代器返回。可以看作是过滤函数。

2 map(function, sequence) 


	str = [1, 2,'a', 'b']
	def fun2(s):
	    return s + "alvin"
	ret = map(fun2, str)
	print(ret)      #  map object的迭代器
	print(list(ret))#  ['aalvin', 'balvin', 'calvin', 'dalvin']

对sequence中的item依次执行function(item)，将执行结果组成一个map object迭代器返回.
map也支持多个sequence，这就要求function也支持相应数量的参数输入：

	def add(x,y):
	    return x+y
	print (list(map(add, range(10), range(10))))##[0, 2, 4, 6, 8, 10, 12, 14, 16, 18]

3 reduce(function, sequence, starting_value)　　
	
	from functools import reduce
	def add1(x,y):
	    return x + y
	 
	print (reduce(add1, range(1, 101)))## 4950 （注：1+2+...+99）
	 
	print (reduce(add1, range(1, 101), 20))## 4970 （注：1+2+...+99+20）

对sequence中的item顺序迭代调用function，如果有starting_value，还可以作为初始值调用.

4 lambda

普通函数与匿名函数的对比：


	#普通函数
	def add(a,b):
	    return a + b
	 
	print add(2,3)
	 
	  
	#匿名函数
	add = lambda a,b : a + b
	print add(2,3)
	 
	 
	#========输出===========
	5
	5

匿名函数的命名规则，用lamdba 关键字标识，冒号（：）左侧表示函数接收的参数（a,b） ,冒号（：）右侧表示函数的返回值（a+b）。

因为lamdba在创建时不需要命名，所以，叫匿名函数　