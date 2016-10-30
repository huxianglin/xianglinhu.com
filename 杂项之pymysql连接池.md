# 杂项之pymysql连接池

## 本节内容
1. 本文的诞生
2. 连接池及单例模式
3. 多线程提升
4. 协程提升
5. 后记

## 1.本文的诞生
由于前几天接触了pymysql，在测试数据过程中，使用普通的pymysql插入100W条数据，消耗时间很漫长，实测990s也就是16.5分钟左右才能插完，于是，脑海中诞生了一个想法，能不能造出一个连接池出来，提升数据呢？就像一根管道太小，那就多加几根管道看效果如何呢？于是。。。前前后后折腾了将近一天时间，就有了本文的诞生。。。

## 2.连接池及单例模式
先说单例模式吧，为什么要在这使用单例模式呢？使用单例模式能够节省资源。

其实单例模式没有什么神秘的，简单的单例模式实现其实就是在类里面定义一个变量，再定义一个类方法，这个类方法用来为调用者提供这个类的实例化对象。（ps:个人对单例模式的一点浅薄理解...）

那么连接池是怎么回事呢？原来使用pymysql创建一个conn对象的时候，就已经和mysql之间创建了一个tcp的长连接，只要不调用这个对象的close方法，这个长连接就不会断开，这样，我们创建了一组conn对象，并将这些conn对象放到队列里面去，这个队列现在就是一个连接池了。

现在，我们先用一个连接，往数据库中插入100W条数据，下面是源码：
    
    import pymysql
    import time
    start=time.time()
    conn = pymysql.connect(host="192.168.10.103",port=3306,user="root",passwd="123456",db="sql_example",charset="utf8")
    conn.autocommit(True)  # 设置自动commit
    cursor = conn.cursor(cursor=pymysql.cursors.DictCursor)  # 设置返回的结果集用字典来表示，默认是元祖
    data=(("男",i,"张小凡%s" %i) for i in range(1000000))  # 伪造数据，data是个生成器
    cursor.executemany("insert into tb1(gender,class_id,sname) values(%s,%s,%s)",data)  # 可以使用executemany执行多条sql
    # conn.commit()
    cursor.close()
    conn.close()
    print("totol time:",time.time()-start)

执行结果为：

totol time: 978.7649309635162

## 3.多线程提升
使用多线程，在启动时创建一组线程，每个线程去连接池里面获取一个连接，然后插入数据，这样将会大大提升执行sql的速度，下面是使用多线程实现的连接池源码：
    
    from gevent import monkey
    monkey.patch_all()
    
    import threading
    
    import pymysql
    from queue import Queue
    import time
    
    class Exec_db:
    
        __v=None
    
        def __init__(self,host=None,port=None,user=None,passwd=None,db=None,charset=None,maxconn=5):
            self.host,self.port,self.user,self.passwd,self.db,self.charset=host,port,user,passwd,db,charset
            self.maxconn=maxconn
            self.pool=Queue(maxconn)
            for i in range(maxconn):
                try:
                    conn=pymysql.connect(host=self.host,port=self.port,user=self.user,passwd=self.passwd,db=self.db,charset=self.charset)
                    conn.autocommit(True)
                    # self.cursor=self.conn.cursor(cursor=pymysql.cursors.DictCursor)
                    self.pool.put(conn)
                except Exception as e:
                    raise IOError(e)
    
        @classmethod
        def get_instance(cls,*args,**kwargs):
            if cls.__v:
                return cls.__v
            else:
                cls.__v=Exec_db(*args,**kwargs)
                return cls.__v
    
        def exec_sql(self,sql,operation=None):
            """
                执行无返回结果集的sql，主要有insert update delete
            """
            try:
                conn=self.pool.get()
                cursor=conn.cursor(cursor=pymysql.cursors.DictCursor)
                response=cursor.execute(sql,operation) if operation else cursor.execute(sql)
            except Exception as e:
                print(e)
                cursor.close()
                self.pool.put(conn)
                return None
            else:
                cursor.close()
                self.pool.put(conn)
                return response
    
    
        def exec_sql_feach(self,sql,operation=None):
            """
                执行有返回结果集的sql,主要是select
            """
            try:
                conn=self.pool.get()
                cursor=conn.cursor(cursor=pymysql.cursors.DictCursor)
                response=cursor.execute(sql,operation) if operation else cursor.execute(sql)
            except Exception as e:
                print(e)
                cursor.close()
                self.pool.put(conn)
                return None,None
            else:
                data=cursor.fetchall()
                cursor.close()
                self.pool.put(conn)
                return response,data
    
        def exec_sql_many(self,sql,operation=None):
            """
                执行多个sql，主要是insert into 多条数据的时候
            """
            try:
                conn=self.pool.get()
                cursor=conn.cursor(cursor=pymysql.cursors.DictCursor)
                response=cursor.executemany(sql,operation) if operation else cursor.executemany(sql)
            except Exception as e:
                print(e)
                cursor.close()
                self.pool.put(conn)
            else:
                cursor.close()
                self.pool.put(conn)
                return response
    
        def close_conn(self):
            for i in range(self.maxconn):
                self.pool.get().close()
    
    obj=Exec_db.get_instance(host="192.168.10.103",port=3306,user="root",passwd="012615",db="sql_example",charset="utf8",maxconn=10)
    
    def test_func(num):
        data=(("男",i,"张小凡%s" %i) for i in range(num))
        sql="insert into tb1(gender,class_id,sname) values(%s,%s,%s)"
        print(obj.exec_sql_many(sql,data))
    
    job_list=[]
    for i in range(10):
        t=threading.Thread(target=test_func,args=(100000,))
        t.start()
        job_list.append(t)
    for j in job_list:
        j.join()
    obj.close_conn()
    print("totol time:",time.time()-start)

开启10个连接池插入100W数据的时间：

totol time: 242.81142950057983

开启50个连接池插入100W数据的时间：

totol time: 192.49499201774597

开启100个线程池插入100W数据的时间：

totol time: 191.73923873901367

## 4.协程提升
使用协程的话，在I/O阻塞时，将会切换到其他任务去执行，这样理论上来说消耗的资源应该会比多线程要少。下面是协程实现的连接池源代码：

    from gevent import monkey
    monkey.patch_all()
    import gevent
    
    import pymysql
    from queue import Queue
    import time
    
    class Exec_db:
    
        __v=None
    
        def __init__(self,host=None,port=None,user=None,passwd=None,db=None,charset=None,maxconn=5):
            self.host,self.port,self.user,self.passwd,self.db,self.charset=host,port,user,passwd,db,charset
            self.maxconn=maxconn
            self.pool=Queue(maxconn)
            for i in range(maxconn):
                try:
                    conn=pymysql.connect(host=self.host,port=self.port,user=self.user,passwd=self.passwd,db=self.db,charset=self.charset)
                    conn.autocommit(True)
                    # self.cursor=self.conn.cursor(cursor=pymysql.cursors.DictCursor)
                    self.pool.put(conn)
                except Exception as e:
                    raise IOError(e)
    
        @classmethod
        def get_instance(cls,*args,**kwargs):
            if cls.__v:
                return cls.__v
            else:
                cls.__v=Exec_db(*args,**kwargs)
                return cls.__v
    
        def exec_sql(self,sql,operation=None):
            """
                执行无返回结果集的sql，主要有insert update delete
            """
            try:
                conn=self.pool.get()
                cursor=conn.cursor(cursor=pymysql.cursors.DictCursor)
                response=cursor.execute(sql,operation) if operation else cursor.execute(sql)
            except Exception as e:
                print(e)
                cursor.close()
                self.pool.put(conn)
                return None
            else:
                cursor.close()
                self.pool.put(conn)
                return response
    
    
        def exec_sql_feach(self,sql,operation=None):
            """
                执行有返回结果集的sql,主要是select
            """
            try:
                conn=self.pool.get()
                cursor=conn.cursor(cursor=pymysql.cursors.DictCursor)
                response=cursor.execute(sql,operation) if operation else cursor.execute(sql)
            except Exception as e:
                print(e)
                cursor.close()
                self.pool.put(conn)
                return None,None
            else:
                data=cursor.fetchall()
                cursor.close()
                self.pool.put(conn)
                return response,data
    
        def exec_sql_many(self,sql,operation=None):
            """
                执行多个sql，主要是insert into 多条数据的时候
            """
            try:
                conn=self.pool.get()
                cursor=conn.cursor(cursor=pymysql.cursors.DictCursor)
                response=cursor.executemany(sql,operation) if operation else cursor.executemany(sql)
            except Exception as e:
                print(e)
                cursor.close()
                self.pool.put(conn)
            else:
                cursor.close()
                self.pool.put(conn)
                return response
    
        def close_conn(self):
            for i in range(self.maxconn):
                self.pool.get().close()
    
    obj=Exec_db.get_instance(host="192.168.10.103",port=3306,user="root",passwd="123456",db="sql_example",charset="utf8",maxconn=10)
    
    def test_func(num):
        data=(("男",i,"张小凡%s" %i) for i in range(num))
        sql="insert into tb1(gender,class_id,sname) values(%s,%s,%s)"
        print(obj.exec_sql_many(sql,data))
    
    start=time.time()
    job_list=[]
    for i in range(10):
        job_list.append(gevent.spawn(test_func,100000))
    
    gevent.joinall(job_list)
    
    obj.close_conn()
    
    print("totol time:",time.time()-start)

开启10个连接池插入100W数据的时间：

totol time: 240.16892313957214

开启50个连接池插入100W数据的时间：

totol time: 202.82087111473083

开启100个线程池插入100W数据的时间：

totol time: 196.1710569858551

## 5.后记

统计结果如下：

单线程一个连接使用时间：978.76s

<table border="1" cellspacing="0">
    <tr>
        <td></td>
        <td>10个连接池</td>
        <td>50个连接池</td>
        <td>100个连接池</td>
    </tr>
    <tr>
        <td>多线程版</td>
        <td>242.81s</td>
        <td>192.49s</td>
        <td>191.74s</td>
    </tr>
    <tr>
        <td>协程版</td>
        <td>240.17s</td>
        <td>202.82s</td>
        <td>196.17s</td>
    </tr>
</table>

通过统计结果显示,通过协程和多线程操作连接池插入相同数据，相对一个连接提升速度明显，但是在将连接池开到50以及100时，性能提升并没有想象中那么大，这时候，瓶颈已经不在网络I/O上了，而在数据库中，mysql在大量连接写入数据时，也会有锁的产生，这时候就需要优化数据库的相关设置了。

在对比中显示多线程利用线程池和协程利用线程池的性能差不多，但是多线程的开销比协程要大。

和大神讨论过，在项目开发中需要考虑到不同情况使用不同的技术，多线程适合使用在连接量较大，但每个连接处理时间很短的情况下，而协程适用于处理大量连接，但同时活跃的链接比较少，并且每个连接的时间量比较大的情况下。

在实际生产应用中，创建连接池可以按需分配，当连接不够用时，在连接池没达到上限的情况下，在连接池里面加入新的连接，在连接池比较空闲的情况下，关闭一些连接，实现这一个操作的原理是通过queue里面的超时时间来控制，当等待时间超过了超时时间时，说明连接不够用了，需要加入新的连接。