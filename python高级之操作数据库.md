# python高级之操作数据库

## 本节内容
1. pymysql介绍及安装
2. 使用pymysql执行sql
3. 获取新建数据自增ID
4. fetch数据类型设置

## 1.pymysql介绍及安装
在python2中连接数据库使可以使用mysqldb模块，为什么在python3中使用pymysql模块呢？因为在python2中mysqldb和pymysql都可以操作数据库，但是当python升级到3以后，pymysql模块支持python3了，但是mysqldb模块目前还不支持python3，所以用python3的小伙伴们还是先用pymysql吧。pymysql和mysqldb的使用方法基本相同。

安装pymysql可以使用pip安装：

    pip3 install pymysql

安装完成之后在python解释器中执行import pymysql如果不报错说明安装成功了。

## 2.使用pymysql执行sql

    import pymysql  # 导入模块
    
    conn=pymysql.connect(host="192.168.12.120",port=3306,user="root",passwd="123456",db="sql_example",charset="utf8")
    # 注意，这里传入的参数必须是关键字参数，不能省略关键字，否则将连接不上数据库
    # conn.autocommit()  # 如果设置了这个属性，则会自动commit sql语句，而不需要手动commit
    cursor=conn.cursor()  # 创建游标
    # cursor.execute("insert into class(caption) values('全栈二班')")
    
    r=cursor.execute("select * from class")  # 执行查询语句并返回查询的数据行数
    print(r)  # 打印行数
    # print(cursor.fetchall())  # 打印返回的所有数据，数据结构是元祖套元祖((),()),里面的一个元祖就是一行数据
    # print(cursor.fetchmany(num))  # 打印返回的num条数据，数据结构是元祖套元祖((),()),里面的一个元祖就是一行数据
    print(cursor.fetchone())   # 打印返回的数据中第一行数据，结构是只有一个元祖
    cursor.scrroll()  # 修改游标的位置
    
    conn.commit()  # 手动提交要执行的sql
    cursor.close()  # 关闭游标
    conn.close()  # 关闭连接

注：在fetch数据时按照顺序进行，可以使用cursor.scroll(num,mode)来移动游标位置，如：

- cursor.scroll(1,mode='relative')  # 相对当前位置移动
- cursor.scroll(2,mode='absolute') # 相对绝对位置移动

## 3.获取新建数据自增ID

    conn = pymysql.connect(host="192.168.12.120",port=3306,user="root",passwd="123456",db="sql_example",charset="utf8")
    cursor = conn.cursor()
    cursor.executemany("insert into hosts(host,color_id)values(%s,%s)", [("1.1.1.11",1),("1.1.1.11",2)])
    ## executemany可以执行多条插入语句，后面接的数据需要是一个可迭代的对象，一般是列表里面套元祖或列表
    conn.commit()
    cursor.close()
    conn.close()
    
    # 获取最新自增ID
    new_id = cursor.lastrowid  # 通过cursor.lastrowid可以获取获取最新行的自增ID

表里面只能有一个字段设置为自增ID。。。

## 4.fetch数据类型设置

关于默认获取的数据是元祖类型，如果想要获得字典类型的数据，即：

    conn = pymysql.connect(host="192.168.12.120",port=3306,user="root",passwd="123456",db="sql_example",charset="utf8")
      
    # 游标设置为字典类型
    cursor = conn.cursor(cursor=pymysql.cursors.DictCursor)
    r = cursor.execute("call p1()")
      
    result = cursor.fetchone()
      
    conn.commit()
    cursor.close()
    conn.close()