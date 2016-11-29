python基础之编译器选择，循环结构，列表

# 本节内容
1. python IDE的选择
2. 字符串的格式化输出
3. 数据类型
4. 循环结构
5. 列表
6. 简单购物车的编写

## 1.python IDE的选择

IDE的全称叫做集成开发环境（IDE，Integrated Development Environment ）

常用的编程语言IDE开发工具有如下一些：


1. VIM #经典的linux下的文本编辑器
2. Emacs #linux 文本编辑器， 比vim更容易使用
3. Eclipse # Java IDE,支持python, c ,c++
4. Visual Studio # 微软开发的 IDE, python,c++,java,C#
5. notepad++ ,#windows下常用的文本编辑器，比原生notepad好用
6. sublime python开发的
7. Pycharm ，是主要用于python开发的ide

pycharm具有代码补全，自动保存，语法检查，多种语言支持（html,js,css等）等特性，使用pycharm能够节省大量编码时间，从而提高生产效率。
pycharm的一些设置这里就不做赘述了，网上百度一大把。。。本人也懒得贴那些图片了。。。

## 2.python字符串的格式化输出

python中字符串格式化输出主要使用的有如下几个占位符：

1. %s占位符	占位字符串
2. %d占位符	占位整形数据
3. %f占位符	占位浮点型数据

举个栗子：

	print("this is %s num %d. the 5/3=%f." %("my",5,5/3))
	this is my num 5. the 5/3=1.666667.
至于更多的字符串格式化输出内容等我有空再继续补充，时间比较仓促。

## 3.数据类型

python中的数据类型主要分为如下几种：

数字：

	整型 python中的整形能表示的数字从2^31-1到-2^31之间，占用4个字节

	长整型 超出整形表示之外的整数将被使用长整型来存储
整型和长整型在python2中有区分，在python3中已经不区分整型和长整型，统一叫做整型了。

布尔类型：
	
	True	真
	False	假

字符串：
	
	用单引号或者双引号包裹起来的内容就是字符串

列表：

	存储一组数据，使用[]来表示

字典：

	存储一组key:value键值对的数据，使用{}表示

集合：

	存储的也是一组数据，但是和列表不同的是，集合中数据没有相同的两个数据，使用set()来定义集合

元祖：
	存储的也是一组数据，只是存储在元祖（tuple）中的数据是不能被改变的，声明该类型的数据使用(,)来实现

## 4.循环结构

其实在上一篇里面已经讲了python中的循环结构，这里照搬过来了：

for循环语句，结构如下所示：

	for 变量 in 可遍历对象：
    	循环体内语句
	else:
    	循环体后语句
 

其中需要注意的是for循环后面还跟了else语句，这个方式我只在python中见到过，目前我知道的一种用法就是当for循环正常执行结束后就会执行else中的语句，但是，当在循环体中使用break或者循环体异常结束时，else中的语句将不会被执行。

while循环语句，结构如下所示：

	while 条件表达式：
    	循环体内语句
	else:
    	循环体后语句
 

while循环中也存在else语句，和for循环中的else语句作用一样，当while之后的条件表达式为真时，才会进入循环体内部执行，注意while循环语句可以一次都不被执行。

下面举几个例子来看看循环体结构的使用：

	_user="abc123"
	_passwd="abc123"
	for i in range(3):
	    username=input("username:").strip()
	    password=input("password:").strip()
	    if username==_user and password==_passwd:
	        print("welcome %s" %username)
	        break
	    else:
	        print("invalid username or password!")
	else:
	    print("拜拜！")
上面例子主要是为了展示for else结构的使用，当然，这种方式也可以通过使用flag的方式来实现：

	_user="abc123"
	_passwd="abc123"
	passed_authentication=False
	for i in range(3):
	    username=input("username:").strip()
	    password=input("password:").strip()
	    if username==_user and password==_passwd:
	        print("welcome %s" %username)
	        passed_authentication=True
	        break
	    else:
	        print("invalid username or password!")
	if not passed_authentication:
	    print("拜拜！")
下面是展示while else循环结构的代码：

	_user="abc123"
	_passwd="abc123"
	count=0
	while count<3:
	    username=input("username:").strip()
	    password=input("password:").strip()
	    if username==_user and password==_passwd:
	        print("welcome %s" %username)
	        break
	    else:
	        print("invalid username or password!")
	    count+=1
	    if count == 3:
	        flag=input("还想玩吗？[y/n]")
	        if flag=="y":
	            count=0
	else:
	    print("拜拜！")
在循环嵌套中，怎样实现跳出多层嵌套循环呢？可以使用flag标记位来实现：

	exit_flag = False
	for i in range(10):
	    if i<5:
	        continue
	    print(i)
	    for j in range(10):
	        print("lever2:",j)
	        if j == 6:
	            exit_flag = True
	            break
	    if exit_flag:
	        break
好了，关于循环结构语句的补充就到这里了。

##　5.列表

列表是python里面很重要的一种数据类型，学好列表的使用，将有助于写出高质量的代码。

所有序列都可以进行某些特定操作，包括：索引（indexing）、分片（slicing）、加（adding）、乘（multiplying）以及检查某元素是否属于列表成员。

### 迭代：依次对序列中的每个元素重复执行某些操作。

序列的索引：通过元素在列表中的位置可以定位到该元素，这就是列表的索引，使用类似于list[0]对元素进行索引，索引0指向第一个元素。也可使用负数对元素进行索引，使用负数对元素索引时，列表中的最后一个元素由-1表示，例如list[-1]就是该列表的最后一个元素。

序列的分片：序列的分片操作需要提供两个索引作为边界，第一个索引的元素包含在分片内部，第二个索引的元素不包含在分片内部。例如需要取出列表中第三位到第五位的元素可以使用list[2:5],若分片所得部分包含列表的尾部可以将第二个索引置为空。例如list[3:],同样，对于列表首部的元素也可以这样，例如list[:-5]。
对于分片，还有一个步长的参数，该参数可以是隐式设置，隐式设置中，步长是1。分片操作按照该步长遍历序列中元素。例如list[0:5:2]就是以步长为二遍历出列表中从开头到第五位元素。当然，步长也可以为负数，为负数时表示从右到左进行遍历。

列表的加：列表可以进行相加，变成一个更大的列表list=list1+list2,但是限制是相同类型的列表才可以进行相加。虽然字符串也是列表，但是列表和字符串是无法直接相加的。

列表的乘：用一个数字乘以一个列表将会生成一个新的列表，该新列表内容江油老序列重复N次。列表内容为空可以使用None表示，list=[None]*10表示生成一个占用十个空位置的列表。

列表的成员资格：可使用in运算符检查某元素是否是该列表的成员。其返回值是一个布尔类型的值，如果是该列表成员，则返回True，否则返回Flse。例如'a' in list操作。

列表的几个内置函数：len、max和min是列表中的内置函数。len函数返回列表包含元素的数量。max和min函数分别返回列表中最大和最小的元素，列入len(list)、max(list)、min(list)

### 列表的操作

列表可以通过分片来实现增删改查功能：

	a=["wuchao","jinxin","xiaohu","sanpang","ligang"]
	print(a[::1])#从开始到结尾切片输出
	print(a[:-1:])#从开始到倒数第二个切片输出
	print(a[::2])#从开始到结尾步长为2输出
	print(a[::-1])#列表反转输出
	print(a[-1:0:-1])#从倒数第一个到顺数第二个反转输出
	print(a[1:1]=["xxx"])#在索引为1的位置插入数据
	print(a[1:2]=[])#把列表中的索引为1的位置的值删除
	print(a[1:2]=["xxx"])#把列表中索引为1的位置的值改成xxx

列表的方法：

1. append:在列表末尾追加新的对象，例如list.append(4)将4追加到list的末尾。
2. count:统计某个元素在列表中出现的次数（只能统计当前层中List中的元素）例如list.count('a')
3. extend:可以在列表的末尾追加另一个列表，和列表连接操作相似，但是列表连接操作返回一个新的列表，extend返回的是原来列表扩展后的列表，效率比列表连接操作高。例如list1.extend(list2)
4. index:从列表中找到某个值第一个匹配项的索引位置例如list.index('a')
5. insert:将对象插入到列表的指定位置，例如list.insert(3,'a')插入的位置在insert的参数中定义的位置，原先在该位置的值将依次往后移一位
6. pop:移除列表中的一个元素（默认最后一个），并返回该元素的值。list.pop(3)就是将第四个位置的值删除并返回该值。python中没有入栈的方法，但是可以结合使用append和pop方法实现栈的功能（先进先出，后进先出）
7. remove:移除列表中某个元素的第一个匹配项，list.remove('a')会移除掉第一个匹配的a，但是不管有没有匹配到该元素都不会有返回值。
8. reverse:将列表中的元素反向存放，list.reverse(),同样没有返回值。
9. sort:在原位置的列表基础上对列表进行排序，而不是返回一个排序好的副本。list.sort()该方法也没有返回值。
10. sorted:该方法对源列表进行排序，并返回一个新的排序好的列表副本，该操作将不会修改源列表的顺序，list2=list1.sorted()
11. 高级排序：如果想要自己定义排序规则可以定义一个函数，compare(x,y)当x<y时返回负数，x>y时返回正数，x=y则返回0定义好函数后可将该函数作为sort的参数传递进sort里面进行排序，另，sort还有两个可选参数key和reverse，若需要使用则需通过关键字参数指定，key的值为在排序过程中使用的函数，例如list.sort(key=len)则排序方式为按照元素的长度进行排序。reverse的值为一个bool类型的值，如果为True则进行反向排序，默认是False例如list.sort(reverse=True)就是进行反向排序。

## 6.简单购物车的编写

需求如下：

	购物车程序
        salary = 5000
        1.  iphone6s  5800
        2.  mac book    9000
        3.  coffee      32
        4.  python book    80
        5.  bicyle         1500
      >>>:1
         余额不足，-3000
      >>>:5
      已加入bicyle 到你的购物车， 当前余额:3500
      >>>:quit
      您已购买一下商品
      bicyle    1500
      coffee    30
      您的余额为:2970
      欢迎下次光临
代码实现：

	#!/usr/bin/env python
	# encoding:utf-8
	# __author__: huxianglin
	# date: 2016-08-22
	# blog: http://huxianglin.cnblogs.com/ http://xianglinhu.blog.51cto.com/
	
	def main():
	    Commodity_list=[["iphone6s",5800],["mac book",9000],["coffee",32],["python book",80],["bicycle",1500]]
	    shopping_list=[]
	    welcome_msg="""
	    ********************************
	    ***    欢迎来到本商店购物     ***
	    ********************************"""
	    print(welcome_msg)
	    salary=None
	    while True:
	        salary=input("请输入你的工资:")
	        if salary.isdigit():
	            salary=int(salary)
	            break
	        else:
	            print("工资只能为数字。。。")
	            continue
	    for i in range(0,len(Commodity_list)):
	        print("\t%s. %s\t\t%s"%(i+1,Commodity_list[i][1],Commodity_list[i][0]))
	    while True:
	        customer_choose=input("请输入要购买的商品编号，退出请输入quit: ").strip()
	        if customer_choose=="quit":
	            print("您的购物清单为：")
	            for i in range(0,len(shopping_list)):
	                print("\t%s. %s\t\t%s"%(i+1,shopping_list[i][1],shopping_list[i][0]))
	            print("您的余额为：%s"%salary)
	            exit()
	        elif customer_choose.isdigit():
	            if int(customer_choose) > len(Commodity_list):
	                print("您输入的商品编号不在本系统中，请重新输入。。。")
	                continue
	            if salary >= Commodity_list[int(customer_choose)-1][1]:
	                shopping_list.append(Commodity_list[int(customer_choose)-1])
	                salary -= Commodity_list[int(customer_choose)-1][1]
	                print("%s已加入您的购物车，当前余额：%s"%(Commodity_list[int(customer_choose)-1][0],salary))
	            else:
	                print("余额不足,%s"%(salary-Commodity_list[int(customer_choose)-1][1]))
	                continue
	        else:
	            print("您输入有误，请重新输入。。。")
	
	if __name__=="__main__":
	    main()