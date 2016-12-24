# python高级之生成器&迭代器

## 本机内容
1. 概念梳理
2. 容器
3. 可迭代对象
4. 迭代器
5. for循环内部实现
6. 生成器

## 1.概念梳理
1. 容器(container)：多个元素组织在一起的数据结构
1. 可迭代对象(iterable)：对象中含有\_\_iter\_\_()方法
1. 迭代器(iterator)：对象含有\_\_next\_\_()方法，并且迭代器也有__iter__()方法
1. 生成器(generator)：生成器其实是一种特殊的迭代器，不过这种迭代器更加优雅
1. 列表/集合/字典推导式(list,set,dict comprehension)

下面这张图解释了这些概念之间的关系：

1. 生成器表达式或者生成器函数一定是一个生成器对象
2. 一个生成器对象一定是一个迭代器
3. 一个迭代器具有\_\_next\_\_()方法（或者可以使用next(obj)获取迭代器数据），并且迭代器也有__iter__()方法
4. 一个对象如果具有\_\_iter\_\_()方法那么这个对象就是一个可迭代对象
5. 迭代器总是一个可迭代对象
6. list,set,dict等就是典型的可迭代对象，因为他们都具有\_\_iter\_\_()方法

![](http://7xsn7l.com2.z0.glb.clouddn.com/迭代器生成器等转换.png)

## 2.容器
前面已经简单说明了什么叫容器，下面再深入解释什么是容器：

容器是一种把多个元素组织在一起的数据结构，容器中的元素可以逐个地迭代获取，可以用 in , not in 关键字判断元素是否包含在容器中。通常这类数据结构把所有的元素存储在内存中（也有一些特列并不是所有的元素都放在内存）在Python中，常见的容器对象有：

- list, deque, ....
- set（可变集合）, frozensets（不可变集合）, ....
- dict, defaultdict（字典的一种特殊方式,带默认值）, OrderedDict（有序字典）, Counter（统计列表中元素出现次数）, ....
- tuple, namedtuple（带名字的tuple）, …
- str

这里需要引出一个概念。。。（强行引出）。。。

python中有个模块叫做collections模块

Python有很多常用的内置数据类型，如int,str,list,tuple,dict,set等，但是他们有一些功能的不完善，比如字典的”键值对”是无序的。

Collections模块在这些内置数据类型的基础实现了几个额外的数据类型

1. namedtuple: 生成可以使用名字来访问元素内容的tuple子类
2. deque: 双端队列，可以快速的从另外一侧追加和推出对象
3. Counter: 计数器，主要用来计数
4. OrderedDict: 有序字典
5. defaultdict: 带有默认值的字典

### 1.namedtuple
namedtuple是tuple的派生类，我们知道访问tuple的元素只能通过索引(下标)访问，在某些情况，我们需要通过名字访问元素，比如对于表示坐标的tuple_coordinate (1,  2, 3)，我们希望通过tuple\_coordinate.x tuple\_coordinate.y, tuple\_coordinate.z来访问元素，这时就可以用到namedtuple。

例子：
```python
from collections import namedtuple

CoordinatesClass = namedtuple('CoordinatesClass',['x', 'y', 'z'])
coordinates = CoordinatesClass(1, 2, 3)
print(type(coordinates))  # <class '__main__.CoordinatesClass'>
print(coordinates.x)  # 1
print(coordinates.y)  # 2
print(coordinates.z)  # 3
```
### 2.deque
deque是一个双向队列，可以从队列的两端添加或弹出元素，且时间复杂度都是O(1)，而从list头部插入或移除元素时，列表的时间复杂度为O(N)(python中的列表内部实现方式是以c中的数组来实现的)，python的queue也提供了一个单向队列queue.Queue，但是需要使用到双向队列的时候可以通过这个方式。

以下是deque的常用方法：

- append()：append从队列的右边添加一个元素
- appendleft()：appendleft从队列的左边添加一个元素
- pop()：pop从队列的右边弹出一个元素并返回
- popleft()：popleft从队列的左边弹出一个元素并返回
- extend()：extend从队列的右边添加多个元素
- extendleft():extendleft从队列的左边添加多个元素
- rotate()，当参数n为正数时，rotate(n)将整个队列元素往右移动N位，左边空出来的位置由右边移出去的元素补充，n为负数时，把整个队列的元素往左移动N位
- remove()：remove删除队列的一个元素
- clear()：clear清空队列

### 3.Counter
假如我们有个列表，list_number = [1, 3, 5, 2, 6, 2, 5, 1, 7, 3, 2, 8, 9, 9]

现在需要找出list中出现次数最多的元素，那么Counter类就正好可以解决这类问题，它提供的most_common()方法可以直接给出答案：
```python
from collections import Counter

list_number = [1, 3, 5, 2, 6, 2, 5, 1, 7, 3, 2, 8, 9, 9]
number_counter = Counter(list_number)
top_two = number_counter.most_common(2)  # 出现次数最多的两个元素及出现次数
print(number_counter, type(number_counter))
print(top_two, type(top_two))
#结果：
Counter({2: 3, 1: 2, 3: 2, 5: 2, 9: 2, 6: 1, 7: 1, 8: 1}) <class 'collections.Counter'>
[(2, 3), (1, 2)] <class 'list'>
#表示出现最多的是元素2，共出现3次，然后是1，出现2次，其他3，5，9也是出现2次，但是1是第一个出现的，所以先输出

Counter的elements()方法输出所有的元素：

from collections import Counter

list_number = [1, 3, 5, 2, 6, 2, 5, 1, 7, 3, 2, 8, 9, 9]
number_counter = Counter(list_number)
top_two = number_counter.most_common(2)
for element in number_counter.elements():
    print(element)
```
我们查看Counter类的定义可以看到Counter是继承dict，  class Counter(dict): ， 所以dict有的方法，Counter也有，比如keys(), values(),items()

### 4.OrderedDict
有时我们需要让字典保持有序，特别是想构建一个映射结构以便稍后对其做序列化或编码成另一种格式时，OrderedDict就显得特别有用，例如我们在进行JSON编码时希望精确控制各字段的顺序，这时就可以用OrderedDict实现。

OrderedDict继承dict，并且内部维护了一个双向链表，它会根据元素加入的顺序来排列键的位置，新加入的元素被放置在链表的末尾。

需要注意的是OrderedDict大小是普通dict的两倍多，这是由于他额外创建的链表所致，当涉及大量OrderedDict实例时，需要考虑所带来的好处能否超越额外的内存开销。
```python
from collections import OrderedDict

dic1 = OrderedDict()
dic1['k1'] = 'v1'
dic1['k2'] = 'v2'
dic1['k3'] = 'v3'
print(dic1, type(dic1))
#执行结果：
OrderedDict([('k1', 'v1'), ('k2', 'v2'), ('k3', 'v3')]) <class 'collections.OrderedDict'>
```
OrderedDict的常用方法：

- pop(key)  # 删除key:value对，并返回这个key对应的value
- popitem()  # 从最尾端删除一个key:value对，并返回该key:value对组成的元祖
- move_to_end(key) # 把某个key移动到最后位置

例子如下：
```python
from collections import OrderedDict

dic1 = OrderedDict()
dic1['k1'] = 'v1'
dic1['k2'] = 'v2'
dic1['k3'] = 'v3'
ret = dic1.pop('k2')
print(ret)
ret = dic1.popitem()
print(ret)
#执行结果
v2
('k3', 'v3')
#######################################
from collections import OrderedDict

dic1 = OrderedDict()
dic1['k1'] = 'v1'
dic1['k2'] = 'v2'
dic1['k3'] = 'v3'

print(dic1)
dic1.move_to_end('k1')
print(dic1)
#执行结果
OrderedDict([('k1', 'v1'), ('k2', 'v2'), ('k3', 'v3')])
OrderedDict([('k2', 'v2'), ('k3', 'v3'), ('k1', 'v1')])
```
将一个无序的字典变成一个有序的字典。。。

举例：
```python
d = {‘banana’:3,’apple’:4,’pear’:1,’orange’:2}
collections.OrderedDict(sorted(d.items(),key = lambda t:t[0]))
或者
collections.OrderedDict(sorted(d.items(),key = lambda t:t[1]))
或者
collections.OrderedDict(sorted(d.items(),key = lambda t:len(t[0])))
```
### 5.defaultdict
defaultdict类在创建对象时可以接收一个参数，该参数指定字典“键值对“里值的默认类型(下面的代码指定dict的值为空列表)：
```python
from collections import defaultdict

dic = defaultdict(list)
print(dic)
#执行结果：
defaultdict(<class 'list'>, {})
```
这样在学习dict时的一个问题就可以得到简化：

需求：将列表list_number = [11, 22, 33, 44, 55, 66, 77, 88, 99] 中元素大于等于66的放入字典的k1中，其余的放在k2中：

使用dict的代码如下：
```python
list_number = [11, 22, 33, 44, 55, 66, 77, 88, 99]
dic = {}
for item in list_number:
    if item >= 66:
        # if 'k1' in dic.keys():
            # dic['k1'].append(item)
        # else:
            # dic['k1'] = [item,]
        dic.setdefault(k1,[]).append(item)  # setdefault当key存在时返回对应的value，否则创建key，并将[]当做value，并返回value
    else:
        # if 'k2' in dic.keys():
            # dic['k2'].append(item)
        # else:
            # dic['k2'] = [item, ]
        dic.setdefault(k2,[]).append(item)

print(dic)
```

如果使用defaultdict，代码得以简化：
```python
from collections import  defaultdict

dic = defaultdict(list)
list_number = [11, 22, 33, 44, 55, 66, 77, 88, 99]
for item in list_number:
    if item >= 66:
        dic['k1'].append(item)
    else:
        dic['k2'].append(item)
print(dic)
```

这里的dic['k1'].append(item)其实就是替换了上面的dic.setdefault(k1,[]).append(item)这个操作，但是使用defaultdict要更快。

强行填坑终于填完了。。。。。。下面我们继续：

容器比较容易理解，因为你就可以把它看作是一个盒子、一栋房子、一个柜子，里面可以塞任何东西。从技术角度来说，当它可以用来询问某个元素是否包含在其中时，那么这个对象就可以认为是一个容器，比如 list，set，tuples都是容器对象

尽管绝大多数容器都提供了某种方式来获取其中的每一个元素，但这并不是容器本身提供的能力，而是 可迭代对象 赋予了容器这种能力，当然并不是所有的容器都是可迭代的。

## 3.可迭代对象
之前第一节中概念解释里面解释了什么是可迭代对象，可迭代对象就是当该对象含有一个\_\_iter\_\_()方法时，该对象就是一个可迭代对象（Iterable）。
刚才说过，很多容器都是可迭代对象，此外还有更多的对象同样也是可迭代对象，比如处于打开状态的files，sockets等等。但凡是可以返回一个 迭代器 的对象都可称之为可迭代对象，听起来可能有点困惑，没关系，可迭代对象与迭代器有一个非常重要的区别:那就是迭代器对象一定有\_\_next\_\_()方法，可迭代对象没有。迭代器和可迭代对象都有\_\_iter\_\_()方法。

例子：
```python
a=[1,2,3]
print(type(a))
print(a.__iter__())
#执行结果：
<class 'list'>
<list_iterator object at 0x0000018D41DA74A8>
#说明a的__iter__方法返回的是一个迭代器，但是a本身是一个列表，也是一个可迭代对象。
#如果执行a.__next__()则会报错
b=iter([1,2,3])
print(type(b))
print(b.__next__())
print(b.__iter__())
#执行结果：
<class 'list_iterator'>
1
<list_iterator object at 0x0000028FF21573C8>
#说明b是一个迭代器对象，其有__next__()方法，也有__iter__()方法
```

可迭代对象和容器一样是一种通俗的叫法，并不是指某种具体的数据类型，list是可迭代对象，dict是可迭代对象，set也是可迭代对象。b是一个独立的迭代器，迭代器内部持有一个状态，该状态用于记录当前迭代所在的位置，以方便下次迭代的时候获取正确的元素。迭代器有一种具体的迭代器类型，比如 list_iterator ， set_iterator 。可迭代对象实现了 __iter__ 和 __next__ 方法（python2中是 next 方法，python3是 __next__ 方法），这两个方法对应内置函数 iter() 和 next() 。 __iter__ 方法返回可迭代对象本身，这使得他既是一个可迭代对象同时也是一个迭代器。

## 4.迭代器
那么什么迭代器呢？它是一个带状态的对象，他能在你调用 next() 方法的时候返回容器中的下一个值，任何实现了 __next__() （python2中实现 next() ）方法的对象都是迭代器，至于它是如何实现的这并不重要。

现在我们就以斐波那契数列()为例，学习为何创建以及如何创建一个迭代器：

著名的斐波拉契数列（Fibonacci），除第一个和第二个数外，任意一个数都可由前两个数相加得到：
```python
def fab(max): 
    L = []
    n, a, b = 0, 0, 1
    while n < max:
        L.append(b)
        a, b = b, a + b
        n = n + 1
    return L
```

这是使用普通方法实现的斐波那契数列列表。下面我们自己创建一个迭代器去实现斐波那契数列：
```python
class Fab(object):
    def __init__(self, max):
        self.max = max
        self.n, self.a, self.b = 0, 0, 1

    def __iter__(self):  # __iter__方法返回的就是这个对象本身
        return self

    def __next__(self):  # 拥有__next__方法说明这个对象是一个迭代器对象
        if self.n < self.max:
            r = self.b
            self.a, self.b = self.b, self.a + self.b
            self.n = self.n + 1
            return r
        raise StopIteration()  # 当读取完了最后一个元素时，强行抛出异常

for key in Fab(5):
    print(key)
```

Fab类通过 \_\_next\_\_() 不断返回数列的下一个数，内存占用始终为常数

Fab既是一个可迭代对象（因为它实现了 \_\_iter\_\_ 方法），又是一个迭代器（因为实现了 \_\_next\_\_ 方法）。实例变量 self .a 和 self.b 用户维护迭代器内部的状态。每次调用 next() 方法的时候做两件事：

1. 为下一次调用 next() 方法修改状态
1. 为当前这次调用生成返回结果

迭代器就像一个懒加载的工厂，等到有人需要的时候才给它生成值返回，没调用的时候就处于休眠状态等待下一次调用。

## 5.for循环内部实现
上一个例子用for循环去遍历一个迭代器，那用for循环也可以遍历一个列表，那么for循环的内部实现机制是什么呢？

for i in obj内部实现机制如下：

- 如果in后面的对象是一个迭代器，那么for循环内部先执行这个对象的\_\_iter\_\_()方法，获取该对象本身。。。调用该迭代器的\_\_next\_\_()方法直到最后捕获异常完成迭代器的遍历。
- 如果in后面的对象是一个可迭代对象，那么for循环内部就首先执行这个对象的\_\_iter\_\_()方法返回一个迭代器iter(obj)，再用迭代器的方式去处理。

## 6.生成器
生成器算得上是Python语言中最吸引人的特性之一，生成器其实是一种特殊的迭代器，不过这种迭代器更加优雅。代码2远没有代码1简洁，生成器（yield）既可以保持代码1的简洁性，又可以保持代码2的效果。它不需要再像上面的类一样写 \_\_iter\_\_() 和 \_\_next\_\_() 方法了，只需要一个 yiled 关键字。 生成器有如下特征是它一定也是迭代器（反之不成立），因此任何生成器也是以一种懒加载的模式生成值。用生成器来实现斐波那契数列的例子是：
```python
    def fab(max):
        n, a, b = 0, 0, 1
        while n < max:
            yield b
            a, b = b, a + b
            n = n + 1
    for n in fab(5):
        print n
```
fab 就是一个普通的python函数，它特殊的地方在于函数体中没有 return 关键字，函数的返回值是一个生成器对象。当执行 f=fab(5) 返回的是一个生成器对象，此时函数体中的代码并不会执行，只有显示或隐示地调用next的时候才会真正执行里面的代码。

yield 的作用就是把一个函数变成一个 generator，带有 yield 的函数不再是一个普通函数，Python 解释器会将其视为一个 generator，在 for 循环执行时，每次循环都会执行 fab 函数内部的代码，执行到 yield b 时，fab 函数就返回一个迭代值，下次迭代时，代码从 yield b 的下一条语句继续执行，而函数的本地变量看起来和上次中断执行前是完全一样的，于是函数继续执行，直到再次遇到 yield。看起来就好像一个函数在正常执行的过程中被 yield 中断了数次，每次中断都会通过 yield 返回当前的迭代值。

也可以手动调用 fab(5) 的 \_\_next\_\_() 方法（因为 fab(5) 是一个 generator 对象，该对象具有 \_\_next\_\_() 方法），这样我们就可以更清楚地看到 fab 的执行流程：
```python
>>> f = fab(3)
>>> f.__next__()
>>> f.__next__()
>>> f.__next__()
>>> f.__next__()

Traceback (most recent call last):
  File "<pyshell#62>", line 1, in <module>
    f.next()
StopIteration
```
需要明确的就是生成器也是iterator迭代器，因为它遵循了迭代器协议.

### 生成器的两种创建方式

1. 包含yield的函数
2. 生成器表达式

生成器函数跟普通函数只有一点不一样，就是把 return 换成yield,其中yield是一个语法糖，内部实现了迭代器协议，同时保持状态可以挂起。如下:

### return

在一个生成器中，如果没有return，则默认执行到函数完毕；如果遇到return,如果在执行过程中 return，则直接抛出 StopIteration 终止迭代.
```python
def f():

    yield 5
    print("ooo")
    return
    yield 6
    print("ppp")
        # if str(tem)=='None':
        #     print("ok")

f=f()
# print(f.__next__())
# print(f.__next__())
for i in f:
    print(i)

'''
return即迭代结束
for不报错的原因是内部处理了迭代结束的这种情况
'''

return

### 文件读取

def read_file(fpath):
    BLOCK_SIZE = 1024 
    with open(fpath, 'rb') as f:
        while True:
            block = f.read(BLOCK_SIZE)
            if block:
                yield block
            else:
                return
```

如果直接对文件对象调用 read() 方法，会导致不可预测的内存占用。好的方法是利用固定长度的缓冲区来不断读取文件内容。通过 yield，我们不再需要编写读文件的迭代类，就可以轻松实现文件读取。

生成器对象就是一种特殊的迭代器对象，满足迭代器协议，可以调用next；对生成器对象for 循环时，调用iter方法返回了生成器对象，然后再不断next迭代，而iter和next都是在yield内部实现的。

使用文件读取找出文件中最长的行长度：

max(len(x.strip()) for x in open('/hello/abc','r'))

### 生成器表达式

前面我们介绍过列表推导试快速生成列表，例如a=[i for i in range(10)],现在只需要把中括号改成小括号：a=（i for i in range(10)）之后a就是一个生成器了。

### 代码分析
```python
def add(s, x):
 return s + x

def gen():
 for i in range(4):
  yield i

base = gen()
for n in [1, 10]:
    base = (add(i, n) for i in base)

print(list(base))
# 执行结果：
[20, 21, 22, 23]
```

咦，奇怪吧，执行结果并不是我们想象的。。。
因为执行for循环里面的代码的时候，base = (add(i, n) for i in base)这块代码并没有执行下去，也就是说这时候base并不像我们平时想的那样生成一个类似于列表[1,2,3,4]一样的东西，而是两次循环都执行完成之后，这时候，n的值变为了10，在最后一行print(list(base))中的list(base)才将生成器转换成了一个列表，这时候将n=10带入进去计算，得到的结果就是[20, 21, 22, 23]。。。。。。代码很具有迷惑性。。。比较难理解。

### 生成器的扩展

生成器对象支持几个方法，如gen.__next__() ,gen.send() ,gen.throw()等。

gen.__next:__()方法我们在前面就使用过了，这里不举例子了。

send的工作方式：
```python
def f():
    print("ok")
    s=yield 7
    print(s)
    yield 8

f=f()
print(f.send(None))
print(next(f))

#print(f.send(None))等同于print(next(f)),执行流程:打印ok,yield7,当再next进来时:将None赋值给s,然后返回8,可以通过断点来观察 

gen.throw()方式示例代码如下：

def mygen():
    try:
        yield "something"
    except ValueError:
        yield "value error"
    finally:
        print("clean")  # 一定会被执行
gg=mygen()
print(gg.__next__()) # something
print(gg.throw(ValueError)) # value error clean
# 执行结果：
something
value error
clean
```
从现象来看，throw将会执行except里面对应的error类型里面的yield并打印返回的结果。

### 协程应用：

所谓协同程序也就是是可以挂起，恢复，有多个进入点。其实说白了，也就是说多个函数可以同时进行，可以相互之间发送消息等。
```python
import queue
def tt():
    for x in range(4):
        print ('tt'+str(x) )
        yield

def gg():
    for x in range(4):
        print ('xx'+str(x) )
        yield

class Task():
    def __init__(self):
        self._queue = queue.Queue()

    def add(self,gen):
        self._queue.put(gen)

    def run(self):
        while not self._queue.empty():
            for i in range(self._queue.qsize()):
                try:
                    gen= self._queue.get()
                    gen.send(None)
                except StopIteration:
                    pass
                else:
                    self._queue.put(gen)

t=Task()
t.add(tt())
t.add(gg())
t.run()

# tt0
# xx0
# tt1
# xx1
# tt2
# xx2
# tt3
# xx3
```