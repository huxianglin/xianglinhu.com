# python基础之dict、set及字符串处理

## 本节内容
1. 字典介绍及内置方法
2. 集合介绍
3. 字符串处理

## 1.字典介绍及内置方法

字典是python中唯一的映射类型，采用键值对（key-value）的形式存储数据。python对key进行哈希函数运算，根据计算的结果决定value的存储地址，所以字典是无序存储的，且key必须是可哈希的。可哈希表示key必须是不可变类型，如：数字、字符串、元组。

字典(dictionary)是除列表以外python之中最灵活的内置数据结构类型。列表是有序的对象结合，字典是无序的对象集合。两者之间的区别在于：字典当中的元素是通过键来存取的，而不是通过偏移存取。

### 字典创建方法

第一种创建方法

    dic1={"name":"xxx","age":22}
第二种创建方法，使用dict()类（工厂函数）创建，dict里面传递的参数可以是字典，列表或者其他可迭代的对象

    dic1=dict((("name","xxx"),))
    dic1=dict([["name","xxx"]])
    a=[1,2,3]
    b=["a","b","c"]
    dic1=dict(zip(a,b))

### 字典的操作
- 增

	    dic1={"name":"xxx","age":22}
	    dic1["hobby"]="girl"
	    dic2=dic1.setdefault("age",18) #使用setdefault方法假如字典中有该key，则返回原来的value并不修改value
	    dic2=dic1.setdefault("hobby","girl")#假如字典中没有该key，则添加该key,value对并返回新增加的value
- 查

	    dic2={"name":"xxx","age":22}
	    dic2["name"] #通过key获取value
	    print(dic2.keys())#返回所有的key
	    print(dic2.values())#返回所有的value
	    print(dic2.items())#返回所有的item
		print(dic2.get("name"))#通过get方法获取某个key对应的value
- 改

	    dic3={"name":"xxx","age":22}
	    dic4={"age":33,"hobby":"girl"}
	    dic3.update(dic4)#假如dic4中的key在dic3中存在，那么覆盖dit3中的value，否则添加这个键值对到dic3中
	    dic3["name"]="alex2"#修改字典中对应key的value
	    dic3["name"]="abc"#通过key对字典进行修改
- 删

		dic5={"name":"xxx","age":22}
		dic5.clear()#清空该字典内容
		del dic5['name']#通过key删除某个键值对
		print(dic5.popitem())#popitem将随机删除字典中的一个item并将该item返回
		print(dic5.pop("name"))#删除指定key对应的键值对，并返回该键对应的value
		del dic5#删除该字典
- 其他操作及方法
	
	dict.fromkeys方法也可以创建字典，创建字典方式如下：
	
		dic6=dict.fromkeys(['host1','host2','host3'],'test')
		print(dic6)#{'host3': 'test', 'host1': 'test', 'host2': 'test'}
		dic6['host2']='abc'
		print(dic6)
	但是通过这种方式创建的字典的value初始都是同一个值，假如下面这种情况就会出现异常：

		dic6=dict.fromkeys(['host1','host2','host3'],['test1','tets2'])
		print(dic6)
		{'host2': ['test1', 'tets2'], 'host3': ['test1', 'tets2'], 'host1': ['test1', 'tets2']}
		dic6['host2'][1]='test3'
		print(dic6)
		{'host3': ['test1', 'test3'], 'host2': ['test1', 'test3'], 'host1': ['test1', 'test3']}
	说明这里面的value类似于浅拷贝一样的效果，修改value中的列表的内容的时候，所有value都修改了。
	
	dict.copy()方法是对字典的浅拷贝,返回一个和之前字典有相同内容的新字典
	
	dict没有内置sort方法，要对字典进行排序需要使用sorted(dict)方式，默认是根据key来进行排序，也可以根据value或者自定义方式来排序。如sorted(dict.values())就是根据value进行排序。

## 2.集合介绍

集合定义：把不同的可hash的元素放在一起就是一个集合。

集合的特点：没有重复值，值必须是可hash的。

集合的声明：a=set(“序列”) 里面的序列可以是一个字符串，可以是一个列表，也可以是一个字典（是字典的话，只能存储字典的key）

不可变集合：通过frozenset(“序列”)可以创建不可变集合，不可变集合和集合之间的关系和list和tuple之间的关系差不多

### 集合方法：
a = set([1, 2, 3, 4, 5])
a.add(6)  # 没有返回值，添加一个元素，如果添加的元素在set里面的话，set的内容将不会被改变

a.update("hello")  # Update a set with the union of itself and others将一个序列转换成set并取并集给原来的set

a.clear()  # 清空集合的内容

a.remove(4)  # 删除集合里面指定的key

a.pop()  # 随机删除集合中的一个key并返回这个key


### 集合间的关系：

a=set([1,2,3,4,5])

b=set([4,5,6,7,8])

求交集：intersection

print(a.intersection(b))

print(a&b)

求并集：union

print(a.union(b))

print(a|b)

求差集：difference

print(a.difference(b))

print(b-a)

求对称差集 反向交集：symmetric_difference

print(a.symmetric_difference(b))

print(a^b)

print((a | b)-(a&b))  # 使用并集减去交集也能得到相同结果  集合之间可以有-运算，但是没有+运算

求父集 超集：issuperset父集 issubset子集

print(a.issuperset(b))  # a>=b

print(a>=b)

print(a.issubset(b))  # a<=b

print(a<=b)

print(a=>b)  # a包含b

print(a<=b)  # b包含a

print(a==b)  # a和b等价

## 3.字符串处理

字符串处理方法比较多，下面通过分类来进行解释字符串的一些方法：

字母处理方法：

    全部大写：str.upper()
    全部小写：str.lower()
    大小写互换：str.swapcase()
    整个字符串首字母大写，其余小写：str.capitalize()
    每个单词的首字母大写：str.title()
格式化相关

	获取固定长度，右对齐，左边不够用默认用空格补齐，可自行在第二个参数中设置补充字符：str.rjust(width，"*")
	获取固定长度，左对齐，右边不够用默认用空格补齐，可自行在第二个参数中设置补充字符：str.ljust(width，"*")
	获取固定长度，中间对齐，两边不够用默认空格补齐，可自行在第二个参数中设置补充字符：str.center(width，"*")
	获取固定长度，右对齐，左边不足用0补齐：str.zfill(20)
	使用format格式化输出，用{}包裹变量，（如果字符串中含有{}的话，左括号弄成{{，右括号弄成}}实现大括号转义）:str.format(变量名=值)
	使用format_map格式化输出，用{}包裹变量，（如果字符串中含有{}的话，左括号弄成{{，右括号弄成}}实现大括号转义）和format区别是传递参数形式不一样:str.format_map(dict)
字符串搜索相关

	搜索指定字符串，没有返回-1：str.find('t')
	指定起始位置搜索：str.find('t',start)
	指定起始及结束位置搜索：str.find('t',start,end)
	从右边开始查找：str.rfind('t')
	上面所有方法都可用index代替，不同的是使用index查找不到会抛异常，而find返回-1
	搜索到多少个指定字符串：str.count('t')
字符串替换相关

	替换old为new：str.replace('old','new')
	替换指定次数的old为new：str.replace('old','new',maxReplaceTimes)	
字符串去空格及去指定字符

	去两边空格，换行符，制表符等：str.strip()
	去左空格，换行符，制表符等：str.lstrip()
	去右空格，换行符，制表符等：str.rstrip()
	去两边字符串：str.strip('d')，相应的也有lstrip，rstrip
按指定字符分割字符串为数组：

	默认按空格分隔，也可按照自定义分隔符分隔，第二个参数可以指定最大分隔次数：str.split(' '，maxnum)
	从右往左默认按空格分隔，也可按照自定义分隔符分隔，第二个参数可以指定最大分隔次数：str.rsplit(' '，maxnum)
字符串判断相关

	是否以start开头：str.startswith('start')
	是否以end结尾：str.endswith('end')
	是否全为字母或数字：str.isalnum()
	是否全字母：str.isalpha()
	是否全数字：str.isdigit()
	是否全小写：str.islower()
	是否全大写：str.isupper()
	