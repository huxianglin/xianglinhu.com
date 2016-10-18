# python高级之多线程

## 本节内容
1. 线程与进程定义及区别
2. python全局解释器锁
3. 线程的定义及使用
4. 互斥锁
5. 线程死锁和递归锁
6. 条件变量同步(Condition)
7. 同步条件(Event)
8. 信号量
9. 队列Queue
10. Python中的上下文管理器(contextlib模块)
11. 自定义线程池

## 1.线程与进程定义及区别

### 线程的定义：

线程是操作系统能够进行运算调度的最小单位。它被包含在进程之中，是进程中的实际运作单位。一条线程指的是进程中一个单一顺序的控制流，一个进程中可以并发多个线程，每条线程并行执行不同的任务。

### 进程的定义：

进程（Process）是计算机中的程序关于某数据集合上的一次运行活动，是系统进行资源分配和调度的基本单位，是操作系统结构的基础。在早期面向进程设计的计算机结构中，进程是程序的基本执行实体。程序是指令、数据及其组织形式的描述，进程是程序的实体。

### 进程和线程的区别
- Threads share the address space of the process that created it; processes have their own address space.
- 线程的地址空间共享，每个进程有自己的地址空间。
- Threads have direct access to the data segment of its process; processes have their own copy of the data segment of the parent process.
- 一个进程中的线程直接接入他的进程的数据段，但是每个进程都有他们自己的从父进程拷贝过来的数据段
- Threads can directly communicate with other threads of its process; processes must use interprocess communication to communicate with sibling processes.
- 一个进程内部的线程之间能够直接通信，进程之间必须使用进程间通信实现通信
- New threads are easily created; new processes require duplication of the parent process.
- 新的线程很容易被创建，新的进程需要从父进程复制
- Threads can exercise considerable control over threads of the same process; processes can only exercise control over child processes.
- 一个进程中的线程间能够有相当大的控制力度，进程仅仅只能控制他的子进程
- Changes to the main thread (cancellation, priority change, etc.) may affect the behavior of the other threads of the process; changes to the parent process does not affect child processes.
- 改变主线程（删除，优先级改变等）可能影响这个进程中的其他线程；修改父进程不会影响子进程

## 2.python全局解释器锁

全局解释器锁又叫做GIL

python目前有很多解释器，目前使用最广泛的是CPython，还有PYPY和JPython等解释器，但是使用最广泛的还是CPython解释器，而对于全局解释器锁来说，就是在CPython上面才有的，它的原理是在解释器层面加上一把大锁，保证同一时刻只能有一个python线程在解释器中执行。

对于计算密集型的python多线程来说，无法利用到多线程带来的效果， 在2.7时计算密集型的python多线程执行效率比顺序执行的效率还低的多，在python3.5中对这种情况进行了优化，基本能实现这种多线程执行时间和顺序执行时间差不多的效果。

对于I/O密集型的python多线程来说，GIL的影响不是很大，因为I/O密集型的python多线程进程，每个线程在等待I/O的时候，将会释放GIL资源，供别的线程来抢占。所以对于I/O密集型的python多线程进程来说，还是能比顺序执行的效率要高的。

python的GIL这个东西。。。比较恶心，但是由于CPython解释器中的很多东西都是依赖这个东西开发的，如果改的话，将是一件浩大的工程。。。所以到现在还是存在这个问题，这也是python最为别人所诟病的地方。。。

## 3.线程的定义及使用
### 线程的两种调用方式
线程的调用有两种方式，分为直接调用和继承式调用，示例代码如下：

    #直接调用
    import threading
    import time
     
    def sayhi(num): #定义每个线程要运行的函数
     
        print("running on number:%s" %num)
     
        time.sleep(3)
     
    if __name__ == '__main__':
     
        t1 = threading.Thread(target=sayhi,args=(1,)) #生成一个线程实例
        t2 = threading.Thread(target=sayhi,args=(2,)) #生成另一个线程实例
     
        t1.start() #启动线程
        t2.start() #启动另一个线程
     
        print(t1.getName()) #获取线程名
        print(t2.getName())
    
    #继承式调用
    import threading
    import time
     
     
    class MyThread(threading.Thread):
        def __init__(self,num):
            threading.Thread.__init__(self)
            self.num = num
     
        def run(self):#定义每个线程要运行的函数
     
            print("running on number:%s" %self.num)
     
            time.sleep(3)
     
    if __name__ == '__main__':
     
        t1 = MyThread(1)
        t2 = MyThread(2)
        t1.start()
        t2.start()

可以看到直接调用是导入threading模块并定义一个函数，之后实例化threading.Thread类的时候，将刚定义的函数名通过target参数传递进去，然后调用实例的start()方法启动一个线程。

而继承式调用是创建一个类继承自threading.Thread类，并在构造方法中调用父类的构造方法，之后重写run方法，run方法中就是每个线程起来之后执行的内容，就类似于前面通过target参数传递进去的函数。之后以这个继承的类来创建对象，并执行对象的start()方法启动一个线程。

从这里可以看出，其实。。。直接调用通过使用target参数把函数带进类里面之后应该是用这个函数替代了run方法。

### join和setDaemon

join()方法在该线程对象启动了之后调用线程的join()方法之后，那么主线程将会阻塞在当前位置直到子线程执行完成才继续往下走，如果所有子线程对象都调用了join()方法，那么主线程将会在等待所有子线程都执行完之后再往下执行。

setDaemon(True)方法在子线程对象调用start()方法（启动该线程）之前就调用的话，将会将该子线程设置成守护模式启动，这是什么意思呢？当子线程还在运行的时候，父线程已经执行完了，如果这个子线程设置是以守护模式启动的，那么随着主线程执行完成退出时，子线程立马也退出，如果没有设置守护启动子线程（也就是正常情况下）的话，主线程执行完成之后，进程会等待所有子线程执行完成之后才退出。

示例代码如下：
    
    import threading
    from time import ctime,sleep
    import time
    
    def music(func):
        for i in range(2):
            print ("Begin listening to %s. %s" %(func,ctime()))
            sleep(4)
            print("end listening %s"%ctime())
    
    def move(func):
        for i in range(2):
            print ("Begin watching at the %s! %s" %(func,ctime()))
            sleep(5)
            print('end watching %s'%ctime())
    
    threads = []
    t1 = threading.Thread(target=music,args=('七里香',))
    threads.append(t1)
    t2 = threading.Thread(target=move,args=('阿甘正传',))
    threads.append(t2)
    
    if __name__ == '__main__':
    
        for t in threads:
            # t.setDaemon(True)
            t.start()
            # t.join()
        # t1.join()
        t2.join()########考虑这三种join位置下的结果？
        print ("all over %s" %ctime())

## 4.互斥锁
互斥锁的产生是因为前面提到过多线程之间是共享同一块内存地址的，也就是说多个不同的线程能够访问同一个变量中的数据，那么，当多个线程要修改这个变量，会产生什么情况呢？当多个线程修改同一个数据的时候，如果操作的时间够短的话，能得到我们想要的结果，但是，如果修改数据不是原子性的（这中间的时间太长）的话。。。很有可能造成数据的错误覆盖，从而得到我们不想要的结果。例子如下：

    import time
    import threading
    
    def addNum():
        global num #在每个线程中都获取这个全局变量
        # num-=1  # 如果是这种情况，那么操作时间足够短，类似于原子操作了，所以，能够得到我们想要的结果
    
        temp=num
        print('--get num:',num )  # 因为print会调用终端输出，终端是一个设备，相当于要等待终端I/O就绪之后才能输出打印内容，在等待终端I/O的过程中，该线程已经挂起。。。这时其他线程获取到的是没被改变之前的num值，之后该线程I/O就绪之后切换回来，对num-1了，其他线程在I/O就绪之后也在没被改变之前的num基础上减一，这样。。。就得到了我们不想看到的结果。。。
        #time.sleep(0.1)  # sleep也能达到相同的效果，执行到sleep时，该线程直接进入休眠状态，释放了GIL直到sleep时间过去。
        num =temp-1 #对此公共变量进行-1操作
    
    
    num = 100  #设定一个共享变量
    thread_list = []
    for i in range(100):
        t = threading.Thread(target=addNum)
        t.start()
        thread_list.append(t)
    
    for t in thread_list: #等待所有线程执行完毕
        t.join()
    
    print('final num:', num )

这时候，就需要互斥锁出场了，前面出现的num可以被称作临界资源（会被多个线程同时访问），为了让临界资源能够实现按照我们控制访问，需要使用互斥锁来锁住临界资源，当一个线程需要访问临界资源时先检查这个资源有没有被锁住，如果没有被锁住，那么访问这个资源并同时给这个资源加上锁，这样别的线程就无法访问该临界资源了，直到这个线程访问完了这个临界资源之后，释放这把锁，其他线程才能够抢占该临界资源。这个，就是互斥锁的概念。
示例代码：

    import time
    import threading
    
    def addNum():
        global num #在每个线程中都获取这个全局变量
        # num-=1
        lock.acquire()  # 检查互斥锁，如果没锁住，则锁住并往下执行，如果检查到锁住了，则挂起等待锁被释放时再抢占。
        temp=num
        print('--get num:',num )
        #time.sleep(0.1)
        num =temp-1 #对此公共变量进行-1操作
        lock.release()  # 释放该锁
    
    num = 100  #设定一个共享变量
    thread_list = []
    lock=threading.Lock()  # 定义互斥锁
    
    for i in range(100):
        t = threading.Thread(target=addNum)
        t.start()
        thread_list.append(t)
    
    for t in thread_list: #等待所有线程执行完毕
        t.join()
    
    print('final num:', num )

互斥锁与GIL的关系？

Python的线程在GIL的控制之下，线程之间，对整个python解释器，对python提供的C API的访问都是互斥的，这可以看作是Python内核级的互斥机制。但是这种互斥是我们不能控制的，我们还需要另外一种可控的互斥机制———用户级互斥。内核级通过互斥保护了内核的共享资源，同样，用户级互斥保护了用户程序中的共享资源。

GIL 的作用是：对于一个解释器，只能有一个thread在执行bytecode。所以每时每刻只有一条bytecode在被执行一个thread。GIL保证了bytecode 这层面上是thread safe的。
但是如果你有个操作比如 x += 1，这个操作需要多个bytecodes操作，在执行这个操作的多条bytecodes期间的时候可能中途就换thread了，这样就出现了data races的情况了。

## 5.线程死锁和递归锁
如果公共的临界资源比较多，并且线程间都使用互斥锁去访问临界资源，那么将有可能出现一个情况：

- 线程1拿到了资源A，接着需要资源B才能继续执行下去
- 线程2拿到了资源B，接着需要资源A才能继续执行下去

这样，线程1和线程2互不相让。。。结果就都卡死在这了，这就是线程死锁的由来。。。

示例代码如下：
    
    import threading,time
    
    class myThread(threading.Thread):
        def doA(self):
            lockA.acquire()  # 锁住A资源
            print(self.name,"gotlockA",time.ctime())
            time.sleep(3)
            lockB.acquire()  # 锁住B资源
            print(self.name,"gotlockB",time.ctime())
            lockB.release()  # 解锁B资源
            lockA.release()  # 解锁A资源
    
        def doB(self):
            lockB.acquire()
            print(self.name,"gotlockB",time.ctime())
            time.sleep(2)
            lockA.acquire()
            print(self.name,"gotlockA",time.ctime())
            lockA.release()
            lockB.release()
        def run(self):
            self.doA()
            self.doB()
    if __name__=="__main__":
    
        lockA=threading.Lock()
        lockB=threading.Lock()
        threads=[]
        for i in range(5):
            threads.append(myThread())
        for t in threads:
            t.start()
        for t in threads:
            t.join()  # 等待线程结束

那么，怎么解决这个问题呢？python中提供了一个方法（不止python中，基本上所有的语言中都支持这个方法）那就是递归锁。递归锁的创建是使用threading.RLock()，它里面其实维护了两个东西，一个是Lock，另一个是counter，counter记录了加锁的次数，每加一把锁，counter就会+1，释放一次锁counter就会减一，直到所有加的锁都被释放掉了之后其他线程才能够访问这把锁获取资源。当然这个限制是对于线程之间的，同一个线程中，只要这个线程抢到了这把锁，那么这个线程就可以对这把锁加多个锁，而不会阻塞自己的执行。这就是递归锁的原理。

示例代码：

    import time
    import threading
    class Account:
        def __init__(self, _id, balance):
            self.id = _id
            self.balance = balance
            self.lock = threading.RLock()
    
        def withdraw(self, amount):
            with self.lock:  # 会将包裹的这块代码用锁保护起来，直到这块代码执行完成之后，这把锁就会被释放掉
                self.balance -= amount
    
        def deposit(self, amount):
            with self.lock:
                self.balance += amount
    
    
        def drawcash(self, amount):#lock.acquire中嵌套lock.acquire的场景
    
            with self.lock:
                interest=0.05
                count=amount+amount*interest
    
                self.withdraw(count)
    
    
    def transfer(_from, to, amount):
    
        #锁不可以加在这里 因为其他的线程执行的其它方法在不加锁的情况下数据同样是不安全的
         _from.withdraw(amount)
         to.deposit(amount)

    alex = Account('alex',1000)
    yuan = Account('yuan',1000)
    
    t1=threading.Thread(target = transfer, args = (alex,yuan, 100))
    t1.start()
    
    t2=threading.Thread(target = transfer, args = (yuan,alex, 200))
    t2.start()
    
    t1.join()
    t2.join()
    
    print('>>>',alex.balance)
    print('>>>',yuan.balance)

## 6.条件变量同步(Condition)

有一类线程需要满足条件之后才能够继续执行，Python提供了threading.Condition 对象用于条件变量线程的支持，它除了能提供RLock()或Lock()的方法外，还提供了 wait()、notify()、notifyAll()方法。

lock_con=threading.Condition([Lock/Rlock])： 锁是可选选项，不传人锁，对象自动创建一个RLock()。


1. wait()：条件不满足时调用，线程会释放锁并进入等待阻塞；
2. notify()：条件创造后调用，通知等待池激活一个线程；
3. notifyAll()：条件创造后调用，通知等待池激活所有线程。

示例代码：

    import threading,time
    from random import randint
    class Producer(threading.Thread):
        def run(self):
            global L
            while True:
                val=randint(0,100)
                print('生产者',self.name,":Append"+str(val),L)
                if lock_con.acquire():
                    L.append(val)
                    lock_con.notify()
                    lock_con.release()
                time.sleep(3)
    class Consumer(threading.Thread):
        def run(self):
            global L
            while True:
                    lock_con.acquire()
                    if len(L)==0:
                        lock_con.wait()
                    print('消费者',self.name,":Delete"+str(L[0]),L)
                    del L[0]
                    lock_con.release()
                    time.sleep(0.25)
    
    if __name__=="__main__":
    
        L=[]
        lock_con=threading.Condition()
        threads=[]
        for i in range(5):
            threads.append(Producer())
        threads.append(Consumer())
        for t in threads:
            t.start()
        for t in threads:
            t.join()

## 7.同步条件(Event)

 条件同步和条件变量同步差不多意思，只是少了锁功能，因为条件同步设计于不访问共享资源的条件环境。event=threading.Event()：条件环境对象，初始值 为False；

1. event.isSet()：返回event的状态值；
1. event.wait()：如果 event.isSet()==False将阻塞线程；
1. event.set()： 设置event的状态值为True，所有阻塞池的线程激活进入就绪状态， 等待操作系统调度；
1. event.clear()：恢复event的状态值为False。

例子1：

    import threading,time
    class Boss(threading.Thread):
        def run(self):
            print("BOSS：今晚大家都要加班到22:00。")
            event.isSet() or event.set()
            time.sleep(5)
            print("BOSS：<22:00>可以下班了。")
            event.isSet() or event.set()
    class Worker(threading.Thread):
        def run(self):
            event.wait()
            print("Worker：哎……命苦啊！")
            time.sleep(0.25)
            event.clear()
            event.wait()
            print("Worker：OhYeah!")
    if __name__=="__main__":
        event=threading.Event()
        threads=[]
        for i in range(5):
            threads.append(Worker())
        threads.append(Boss())
        for t in threads:
            t.start()
        for t in threads:
            t.join()

例子2：

    import threading,time
    import random
    def light():
        if not event.isSet():
            event.set() #wait就不阻塞 #绿灯状态
        count = 0
        while True:
            if count < 10:
                print('\033[42;1m--green light on---\033[0m')
            elif count <13:
                print('\033[43;1m--yellow light on---\033[0m')
            elif count <20:
                if event.isSet():
                    event.clear()
                print('\033[41;1m--red light on---\033[0m')
            else:
                count = 0
                event.set() #打开绿灯
            time.sleep(1)
            count +=1
    def car(n):
        while 1:
            time.sleep(random.randrange(10))
            if  event.isSet(): #绿灯
                print("car [%s] is running.." % n)
            else:
                print("car [%s] is waiting for the red light.." %n)
    if __name__ == '__main__':
        event = threading.Event()
        Light = threading.Thread(target=light)
        Light.start()
        for i in range(3):
            t = threading.Thread(target=car,args=(i,))
            t.start()

## 8.信号量

信号量用来控制线程并发数的，BoundedSemaphore或Semaphore管理一个内置的计数 器，每当调用acquire()时-1，调用release()时+1。

计数器不能小于0，当计数器为 0时，acquire()将阻塞线程至同步锁定状态，直到其他线程调用release()。(类似于停车位的概念)

BoundedSemaphore与Semaphore的唯一区别在于前者将在调用release()时检查计数 器的值是否超过了计数器的初始值，如果超过了将抛出一个异常。

例子：

    import threading,time
    class myThread(threading.Thread):
        def run(self):
            if semaphore.acquire():
                print(self.name)
                time.sleep(5)
                semaphore.release()
    if __name__=="__main__":
        semaphore=threading.Semaphore(5)
        thrs=[]
        for i in range(100):
            thrs.append(myThread())
        for t in thrs:
            t.start()


## 9.队列Queue

使用队列方法：

    创建一个“队列”对象
    import Queue
    q = Queue.Queue(maxsize = 10)
    Queue.Queue类即是一个队列的同步实现。队列长度可为无限或者有限。可通过Queue的构造函数的可选参数maxsize来设定队列长度。如果maxsize小于1就表示队列长度无限。
    
    将一个值放入队列中
    q.put(10)
    调用队列对象的put()方法在队尾插入一个项目。put()有两个参数，第一个item为必需的，为插入项目的值；第二个block为可选参数，默认为
    1。如果队列当前为空且block为1，put()方法就使调用线程暂停,直到空出一个数据单元。如果block为0，put方法将引发Full异常。
    
    将一个值从队列中取出
    q.get()
    调用队列对象的get()方法从队头删除并返回一个项目。可选参数为block，默认为True。如果队列为空且block为True，get()就使调用线程暂停，直至有项目可用。如果队列为空且block为False，队列将引发Empty异常。
    
    Python Queue模块有三种队列及构造函数:
    1、Python Queue模块的FIFO队列先进先出。  class queue.Queue(maxsize)
    2、LIFO类似于堆，即先进后出。             class queue.LifoQueue(maxsize)
    3、还有一种是优先级队列级别越低越先出来。   class queue.PriorityQueue(maxsize)
    
    此包中的常用方法(q = Queue.Queue()):
    q.qsize() 返回队列的大小
    q.empty() 如果队列为空，返回True,反之False
    q.full() 如果队列满了，返回True,反之False
    q.full 与 maxsize 大小对应
    q.get([block[, timeout]]) 获取队列，timeout等待时间
    q.get_nowait() 相当q.get(False)
    非阻塞 q.put(item) 写入队列，timeout等待时间
    q.put_nowait(item) 相当q.put(item, False)
    q.task_done() 在完成一项工作之后，q.task_done() 函数向任务已经完成的队列发送一个信号
    q.join() 实际上意味着等到队列为空，再执行别的操作

例子集合：

    #例子1：    
    import threading,queue
    from time import sleep
    from random import randint
    class Production(threading.Thread):
        def run(self):
            while True:
                r=randint(0,100)
                q.put(r)
                print("生产出来%s号包子"%r)
                sleep(1)
    class Proces(threading.Thread):
        def run(self):
            while True:
                re=q.get()
                print("吃掉%s号包子"%re)
    if __name__=="__main__":
        q=queue.Queue(10)
        threads=[Production(),Production(),Production(),Proces()]
        for t in threads:
            t.start()
    
    #例子2
    import time,random
    import queue,threading
    q = queue.Queue()
    def Producer(name):
      count = 0
      while count <20:
        time.sleep(random.randrange(3))
        q.put(count)
        print('Producer %s has produced %s baozi..' %(name, count))
        count +=1
    def Consumer(name):
      count = 0
      while count <20:
        time.sleep(random.randrange(4))
        if not q.empty():
            data = q.get()
            print(data)
            print('\033[32;1mConsumer %s has eat %s baozi...\033[0m' %(name, data))
        else:
            print("-----no baozi anymore----")
        count +=1
    p1 = threading.Thread(target=Producer, args=('A',))
    c1 = threading.Thread(target=Consumer, args=('B',))
    p1.start()
    c1.start()
    
    #例子3
    #实现一个线程不断生成一个随机数到一个队列中(考虑使用Queue这个模块)
    # 实现一个线程从上面的队列里面不断的取出奇数
    # 实现另外一个线程从上面的队列里面不断取出偶数
    
    import random,threading,time
    from queue import Queue
    #Producer thread
    class Producer(threading.Thread):
      def __init__(self, t_name, queue):
        threading.Thread.__init__(self,name=t_name)
        self.data=queue
      def run(self):
        for i in range(10):  #随机产生10个数字 ，可以修改为任意大小
          randomnum=random.randint(1,99)
          print ("%s: %s is producing %d to the queue!" % (time.ctime(), self.getName(), randomnum))
          self.data.put(randomnum) #将数据依次存入队列
          time.sleep(1)
        print ("%s: %s finished!" %(time.ctime(), self.getName()))
    
    #Consumer thread
    class Consumer_even(threading.Thread):
      def __init__(self,t_name,queue):
        threading.Thread.__init__(self,name=t_name)
        self.data=queue
      def run(self):
        while 1:
          try:
            val_even = self.data.get(1,5) #get(self, block=True, timeout=None) ,1就是阻塞等待,5是超时5秒
            if val_even%2==0:
              print ("%s: %s is consuming. %d in the queue is consumed!" % (time.ctime(),self.getName(),val_even))
              time.sleep(2)
            else:
              self.data.put(val_even)
              time.sleep(2)
          except:   #等待输入，超过5秒 就报异常
            print ("%s: %s finished!" %(time.ctime(),self.getName()))
            break
    class Consumer_odd(threading.Thread):
      def __init__(self,t_name,queue):
        threading.Thread.__init__(self, name=t_name)
        self.data=queue
      def run(self):
        while 1:
          try:
            val_odd = self.data.get(1,5)
            if val_odd%2!=0:
              print ("%s: %s is consuming. %d in the queue is consumed!" % (time.ctime(), self.getName(), val_odd))
              time.sleep(2)
            else:
              self.data.put(val_odd)
              time.sleep(2)
          except:
            print ("%s: %s finished!" % (time.ctime(), self.getName()))
            break
    #Main thread
    def main():
      queue = Queue()
      producer = Producer('Pro.', queue)
      consumer_even = Consumer_even('Con_even.', queue)
      consumer_odd = Consumer_odd('Con_odd.',queue)
      producer.start()
      consumer_even.start()
      consumer_odd.start()
      producer.join()
      consumer_even.join()
      consumer_odd.join()
      print ('All threads terminate!')
    
    if __name__ == '__main__':
      main()
    
    #注意：列表是线程不安全的
    #例子4
    import threading,time
    
    li=[1,2,3,4,5]
    
    def pri():
        while li:
            a=li[-1]
            print(a)
            time.sleep(1)
            try:
                li.remove(a)
            except:
                print('----',a)
    
    t1=threading.Thread(target=pri,args=())
    t1.start()
    t2=threading.Thread(target=pri,args=())
    t2.start()

