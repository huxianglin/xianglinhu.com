# Python笔记 #
## 第二章 字典 ##
### １.字典的创建和使用 ###
字典的创建：dict_a={'a':1,'b':2,'c':3}字典是由多个键以及对应的值组成的键值对组成。键和值之间使用冒号隔开，键值对之间用逗号隔开，整个字典用大括号括起来。

dict函数：dict函数可以通过其他映射生成字典，例如dict([('a',1),('b',2)])，dict函数也可使用关键字参数创建字典，例如dict(name='a',age=1),如果不传递任何参数，dict函数将会返回一个空的字典。

字典的基本操作：
1. len(dict):返回dict中所有的键值对数量
2. dict[key]:返回dict中key所对应的value
3. dict[key]=value:将value赋值给dict的key
4. del dict[key]:删除该key以及对应的value
5. key in dict:判断该key是否存在于该dict中
6. 