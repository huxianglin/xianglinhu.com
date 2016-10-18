# python高级之网络编程

## 本节内容
1. 网络通信概念
2. socket编程
3. socket模块一些方法
4. 聊天socket实现
5. 远程执行命令及上传文件
6. socketserver及其源码分析

## 1.网络通信概念
说到网络通信，那就不得不说TCP/IP协议簇的OSI七层模型了，这个东西当初在学校都学烂了。。。（PS:毕竟本人是网络工程专业出身。。。）
简单介绍下七层模型从底层到上层的顺序：物理层（定义物理设备的各项标准），数据链路层（mac地址等其他东西的封装），网络层（IP包头的的封装），传输层（TCP/UDP数据报头的封装），会话层（这一层涉及的东西不多，但是我觉得SSL(安全套接字)应该封装在这一层，对于应用是透明的），表示层（数据的压缩解压缩放在这一层），应用层（这一层就是用户使用的应用了）

OSI的七层模型从上到下是一层一层封装的过程，一个数据从一台计算机到另一台计算机是先从上到下封装，之后传输到达另一台计算机之后是从下到上一层一层解封装的过程。

好了，说了这么多，那么我们开发里面涉及到的网络通信包含一些什么东西呢？首先我们程序开发中大部分所使用的协议是TCP协议（传输层）UDP协议涉及的比较少。网络层是IPV4协议，当然IPV6可能是未来的主流，还有一些其他网络协议这里并不涉及。。。SSL（安全套接字）可能在web应用中为了提供数据安全性用得比较多，但是服务端应用程序和程序之间的数据传输不会使用到ssl进行数据安全性的保护。所以，我们平常用的基于网络编程所使用的网络协议一般就是使用传输层的TCP协议，网络层的IPV4协议，更底层的协议。。。并不需要我们考虑，那些是网络设备之间该关心的事情。。。哈哈哈

那么，为什么要用TCP协议呢？因为TCP协议提供了一个面向连接的，稳定的通道，数据是按照顺序到达的，TCP协议的机制保证了TCP协议传输数据是一个可靠的传输。但是TCP协议也有他的缺点，那就是保证可靠传输所牺牲的代价就是速度比较慢（当然也不是很离谱，除非在某些极端情况下：比如传输大量的小数据）TCP之所以是面向连接的是因为它的三次握手以及四次挥手（太出名了，这里不用介绍了。。。），保障数据包按照顺序到达是通过分片编号实现的，每一个数据包都有一个编号，接收端根据这个编号去将接收的数据按原来数据顺序组装起来。当然，TCP协议之所以是一个可靠的传输和他的重传机制也是分不开的，当对方没有收到你发送的一个数据包的时候，TCP协议会重新传输这个数据包到对方，直到对方确认收到了这个数据包为止。。。。当然TCP协议里面还有很多优秀的东西。这里就不一一赘述了。哈哈，不然这里就成了网络专场了。。。

好了，现在确定了我们使用TCP/IP来进行网络通信，那么要实现通信需要知道对方机器的IP地址和端口号（当然，这里指的端口号一般是指TCP的端口号，从1-65535，其中，1024及之前的端口被定义为知名端口，最好不要使用）

## 2.socket编程
前面的第一节只是前戏，为了引出我们今天介绍的socket编程，这前戏做的也是挺累的，哈哈哈。

python中的socket是一个模块，使用这个内置模块能够让我们实现网络通信。其实在unix/linux一切皆文件的思想下，socket也可以看作是一个文件。。。python进行网络数据通信也可以理解为一个打开文件读写数据的操作，只不过这是一个特殊的操作罢了。接下来是一个关于socket通信的流程图：

![](http://7xsn7l.com2.z0.glb.clouddn.com/socket通信流程.png)

流程描述：
 
1. 服务器根据地址类型（ipv4,ipv6）、socket类型、协议创建socket
 
2. 服务器为socket绑定ip地址和端口号
 
3. 服务器socket监听端口号请求，随时准备接收客户端发来的连接，这时候服务器的socket并没有被打开
 
4. 客户端创建socket
 
5. 客户端打开socket，根据服务器ip地址和端口号试图连接服务器socket
 
6. 服务器socket接收到客户端socket请求，被动打开，开始接收客户端请求，直到客户端返回连接信息。这时候socket进入阻塞状态，所谓阻塞即accept()方法一直等到客户端返回连接信息后才返回，开始接收下一个客户端连接请求
 
7. 客户端连接成功，向服务器发送连接状态信息
 
8. 服务器accept方法返回，连接成功

9. 客户端向socket写入信息(或服务端向socket写入信息)

10. 服务器读取信息(客户端读取信息)

11. 客户端关闭

12. 服务器端关闭

## 3.socket模块一些方法
下面是socket模块里面提供的一些方法，让我们实现socke通信。

sk=socket.socket()

创建socket对象，这个时候可以指定两个参数，一个是family，另一个是type

family:AF_INET(代表IPV4，默认参数)，AF_INET6（代表使用IPV6），AF_UNIX(UNIX文件系统通信)

type:SOCK_STREAM(TCP协议通信，默认参数)，SOCK_DGRAM（UDP协议通信）
默认不写的话，family=AF_INET type=SOCK_STREAM

sk.bind(address)

sk.bind(address) 将套接字绑定到地址。address地址的格式取决于地址族。在AF_INET下，以元组（host,port）的形式表示地址。

sk.listen(backlog)

开始监听传入连接。backlog指定在拒绝连接之前，可以挂起的最大连接数量。
backlog等于5，表示内核已经接到了连接请求，但服务器还没有调用accept进行处理的连接个数最大为5
这个值不能无限大，因为要在内核中维护连接队列

sk.setblocking(bool)

是否阻塞（默认True），如果设置False，那么accept和recv时一旦无数据，则报错。

sk.accept()

接受连接并返回（conn,address）,其中conn是新的套接字对象，可以用来接收和发送数据。address是连接客户端的地址。

接收TCP 客户的连接（阻塞式）等待连接的到来

sk.connect(address)

连接到address处的套接字。一般，address的格式为元组（hostname,port）,如果连接出错，返回socket.error错误。

sk.connect_ex(address)

同上，只不过会有返回值，连接成功时返回 0 ，连接失败时候返回编码，例如：10061

sk.close()

关闭套接字

sk.recv(bufsize[,flag])

接受套接字的数据。数据以字符串形式返回，bufsize指定最多可以接收的数量。flag提供有关消息的其他信息，通常可以忽略。

sk.recvfrom(bufsize[.flag])

与recv()类似，但返回值是（data,address）。其中data是包含接收数据的字符串，address是发送数据的套接字地址。

sk.send(string[,flag])

将string中的数据发送到连接的套接字。返回值是要发送的字节数量，该数量可能小于string的字节大小。即：可能未将指定内容全部发送。

sk.sendall(string[,flag])

将string中的数据发送到连接的套接字，但在返回之前会尝试发送所有数据。成功返回None，失败则抛出异常。

内部通过递归调用send，将所有内容发送出去。

sk.sendto(string[,flag],address)

将数据发送到套接字，address是形式为（ipaddr，port）的元组，指定远程地址。返回值是发送的字节数。该函数主要用于UDP协议。

sk.settimeout(timeout)

设置套接字操作的超时期，timeout是一个浮点数，单位是秒。值为None表示没有超时期。一般，超时期应该在刚创建套接字时设置，因为它们可能用于连接的操作（如 client 连接最多等待5s ）

sk.getpeername()

返回连接套接字的远程地址。返回值通常是元组（ipaddr,port）。

sk.getsockname()

返回套接字自己的地址。通常是一个元组(ipaddr,port)

sk.fileno()

套接字的文件描述符

## 4.聊天socket实现
好了，前戏做了那么多，到现在都还没写一行代码呢，接下来到了实战的步骤了。

    #server端实现:
    import socket  # 导入socket模块
    sk=socket.socket()  # 实例化socket对象
    address=("0.0.0.0",8000)  # 定义好监听的IP地址和端口，可以绑定网卡上的IP地址，但是生产环境中一般绑定0.0.0.0这样所有网卡上都能够接收并处理这个端口的数据了
    sk.bind(address)  # 绑定IP地址和端口
    sk.listen(5)  # 设定侦听开始并设定侦听队列里面等待的连接数最大为5
    
    while True:  # 循环，为了建立客户端连接
        conn,addr=sk.accept()  # conn拿到的是接入的客户端的对象，之后服务端和客户端通信都是基于这个conn对象进行通信，add获取的是连接的客户端的IP和端口
        while True:  # 这个循环来实现相互之间聊天的逻辑
            try:  # 为什么要使用try包裹这一块呢？因为在通信过程中客户端突然退出了的话，服务端阻塞在recv状态时将会抛出异常并终止程序，为了解决这个异常，需要我们自己捕获这个异常，这是在windows上，linux上不会抛出异常
                data=conn.recv(1024)  # 定义接收的数据大小为1024个字节并接受客户端传递过来的数据，注意这里的数据在python3中是bytes类型的，在python3中网络通信发送和接收的数据必须是bytes类型的，否则会报错
                if data:  # 假如客户端传递数据过来时
                    print("-->",str(data,"utf-8"))  # 打印客户端传递过来的数据，需要从bytes类型数据解码成unicode类型的数据
                    data=bytes(input(">>>"),"utf-8")  # 接收输入的数据并转换成bytes类型的数据
                    conn.send(data)  # 将bytes类型的数据发送给客户端
                else:  # 否则关闭这个客户端连接的对象，当客户端正常退出，执行了sk.close()时将不会发送数据到服务端，也就是说recv获取到的数据是空的
                    conn.close()  # 这时关闭这个conn对象并退出当前循环等待下一个客户端对象来连接
                    break
            except ConnectionResetError as e:  # 捕获到异常之后，打印异常出来并退出循环等待下一个客户端连接
                print(e)
                break
    
    #client端实现
    import socket  # 导入socket模块
    sk=socket.socket()  # 实例化客户端对象
    address=("127.0.0.1",8000)  # 设置客户端需要连接的服务端IP地址以及端口
    sk.connect(address)  # 连接服务端的IP地址以及端口
    while True:  # 循环实现对话
        data=input(">>>").strip()  # 获取用户输入的数据
        if data=="exit":  # 如果输入的是exit 关闭该对象并退出程序
            sk.close()  # 关闭对象
            break  # 退出循环
        sk.send(bytes(data,"utf-8"))  # 发送刚输入的数据，要先转换成bytes类型的数据
        data=str(sk.recv(1024),"utf-8")  # 接收服务端发送的数据，并将其转换成unicode数据类型
        print("-->",data)  # 打印服务端传输过来的数据

上面是一个简单聊天服务器的实现。。。下面实现的逻辑比这个更复杂一点。

##  5.远程执行命令及上传文件
远程执行命令需要服务端添加一个subprocess模块，将执行命令并返回执行结果，下面是执行代码：
    #server端实现
    import socket  # 导入socket模块
    import subprocess  # 导入subprocess模块
    sk=socket.socket()  # 实例化socket对象
    address=("0.0.0.0",8000)  # 设定绑定IP地址以及端口
    sk.bind(address)  # 绑定IP地址及端口
    sk.listen(5)  # 进入侦听状态，并设置等待队列长度为5
    
    while True:  # 循环接收客户端的连接
        conn,addr=sk.accept()  # 接收客户端的连接
        while True:  # 循环处理客户端发送过来的命令
            try:  # 捕捉客户端不正常退出异常
                cmd=str(conn.recv(1024),"utf-8")  # 接收客户端传递过来的命令
                if cmd:  # 命令不为空，则执行下面代码
                    print("-->",cmd)  # 打印客户端传过来的命令
                    obj=subprocess.Popen(cmd,shell=True,stdout=subprocess.PIPE,stderr=subprocess.PIPE)
                    """
                    调用subprocess的Popen类，执行客户端传过来的命令，shell设置为true，否则可能执行异常，把标准正确输出和标准错误输出都接收，这样执行结果就不会显示在server端了
                    """
                    if not obj.wait():  # obj对象有一个wait方法，等待子进程执行完成并返回执行退出状态码，0为正常，否则不正常
                        cmd_result=obj.stdout.read()  # 正常情况下，接收执行结果
                    else:
                       print("error")  # 否则打印出error
                       cmd_result=obj.stderr.read()  # 接收错误信息
                       print(str(cmd_result,"gbk"))  # 在server中打印出错误信息
                    result_len=bytes(str(len(cmd_result)),"utf-8")  # 获取返回结果的长度
                    conn.send(result_len)  # 将返回结果的长度发送给客户端
                    conn.recv(1024)  # 为了防止数据包黏连，用来在两个send中间插入一个recv,否则，可能两次send的时候前面发送的数据和后面的数据之间会发生数据黏连，造成客户端收到的不是一个纯数字的字符串而引发报错
                    conn.sendall(cmd_result)  # 将执行结果发送给客户端
                else:
                    conn.close()  # 如果客户端退出，则关闭这个链接并退出循环，等待下一个链接
                    break
            except ConnectionResetError as e:  # 如果客户端异常退出，打印异常并退出当前循环，等待下一个链接
                print(e)
                break
    #客户端实现
    import socket  # 导入socket模块
    sk=socket.socket()  # 创建socket对象
    address=("127.0.0.1",8000)  # 设置服务端的IP和端口
    sk.connect(address)  # 和服务端建立链接
    while True:  # 循环用来输入命令并接受返回结果
        cmd=input(">>>").strip()  # 接受命令
        if cmd=="exit":  # 如果客户端输入exit则关闭该链接并退出
            sk.close()
            break
        sk.send(bytes(cmd,"utf-8"))  # 发送命令到服务端
        result_len=int(str(sk.recv(1024),"utf-8"))  # 接受服务端发送的执行结果的长度
        sk.send(bytes("ok","utf8"))  # 防止数据接受黏连设置的一个send，没有实际意义
        print(result_len)  # 打印接受到的数据长度
        data=bytes()  # 定义一个bytes类型的变量用来接受执行结果
        while len(data) != result_len:  # 循环接收执行结果，直到接收完成为止
            data+=sk.recv(1024)
        print(str(data,"gbk"))  # 打印执行结果

上面是远程执行命令的实现，下面要实现远程上传文件的代码。

    #server端实现
    import socket  # 导入socket模块
    import os  @ 导入os模块
    sk=socket.socket()  # 创建socket对象
    address=("0.0.0.0",8000)  # 设置绑定地址
    sk.bind(address)  # 绑定地址
    sk.listen()  # 侦听地址及端口
    BASE_DIR=os.path.dirname(os.path.abspath(__file__))  # 获取当前文件所在目录
    while True:  # 循环，连接客户端
        conn,addr=sk.accept()  # 获取连接的客户端对象
        while True:  # 循环实现文件上传
            try:  # 捕捉客户端异常退出
                data=str(conn.recv(1024),"utf8")  # 接收客户端发送过来的数据
                file_name=data.split("|")[0]  # 获取数据的第一个值为文件名
                file_size=int(data.split("|")[1])  # 第二个值为文件大小
                path=os.path.join(BASE_DIR,"post_server_dir",file_name)  # 路径拼接，获取到该文件名的绝对路径
                writ_size=0  # 设置接收的数据大小初始值
                with open(path,"wb") as f:  # 以wb的方式打开文件，因为传输过来的数据都是bytes类型的
                    while writ_size < file_size:  # 当接收的数据小于文件的大小时，循环接收，一直到接受的数据大小等于文件大小结束
                        data=conn.recv(1024)  # 每次接收1024字节的数据
                        f.write(data)  # 将接受的数据写入到文件中
                        writ_size+=len(data)  # 接收的文件数据大小自加
                print("reveive %s successful!"%file_name)  # 循环结束时证明文件接收完成，打印接收成功信息
                conn.sendall(bytes("true","utf8"))  # 将接受成功信息发送给客户端
            except Exception as e:  # 捕捉到异常后打印异常并退出当前循环，等待下一次连接
                print(e)
                break
    #客户端实现
    import socket  # 导入socket模块
    import os  # 导入os模块
    sk=socket.socket()  # 创建socket对象
    address=("127.0.0.1",8000)  # 设置服务端连接的IP及地址
    sk.connect(address)  # 连接服务端
    BASE_DIR=os.path.dirname(os.path.abspath(__file__))  # 获取当前文件所在的目录路径
    while True:  # 用来循环上传文件
        cmd=input(">>>").strip()  # 获取用户输入命令
        if cmd.split("|")[0]=="post":  # 如果输入的第一个参数是post
            path=os.path.join(BASE_DIR,cmd.split("|")[1])  # 获取第二个参数文件名，并将其添加到路径中，获取文件绝对路径
            file_name=os.path.basename(path)  # 获取文件名
            file_size=os.stat(path).st_size  # 获取文件大小
            sk.sendall(bytes("|".join((file_name,str(file_size))),"utf8"))  # 把文件名和文件大小发送给服务端
            with open(path,"rb") as f:  # 打开文件，以rb的方式读取，可直接发送给服务端
                while True:  # 循环读取文件直到读取完成
                    data=f.read(1024)  # 读取文件的1024字节
                    if not data:  # 如果文件读取完成，读取的内容就为空
                        break  # 这时，退出循环
                    sk.sendall(data)  # 把读取的数据发送给服务端
            flag=str(sk.recv(1024),"utf8")  # 读取服务端发送的消息
            if flag=="true":  # 如果服务端发送过来接收成功的消息，说明文件传输成功，打印传输成功消息
                print("send file %s successful!"%file_name)
            else:  # 否则打印文件传输失败的消息
                print("upload file %s failed!"%file_name)

上面是实现文件传输的socket编程，因为每次都读取文件的1024字节并传递给服务端，这样不会把整个文件加载进内存，从而可以实现大文件的上传。

## 6.socketserver及其源码分析

前面写的socket都是每次只能实现和一个客户端通信的，那么能不能实现同时和多个客户端通信，也就是我们所说的并发呢？当然是可以的，python中内置了一个模块叫做socketserver，这个模块对python的socket模块进行更高级的封装，通过使用多进程或者多线程的方式实现并发通信，也就是说每个客户端来连接的话，新开一个进程或者线程去和它通信，虽然这种方式对资源消耗很大，但是它的源码其实并不复杂，阅读他的源码能够对我们的提升有比较好的帮助，所以，建议有空的可以去读读socketserver的源码。加上注释一共才700行左右的源码。

我们先来实现一个支持多客户端的socketserver示例，再来分析这个示例在python中是如何实现的。

    #服务端实现
    import socketserver  # 导入socketserver模块
    class Myserver(socketserver.BaseRequestHandler):  # 定义一个类名继承自socketserver.BaseRequestHandler
        def handle(self):  # 自定义一个函数名handle（注意这个名字不能命名成别的名字，父类中有这个方法，相当于重写了这个方法）里面实现的是socket通信的逻辑代码块
            while True:
                conn=self.request
                while True:
                    try:
                        data=str(conn.recv(1024),"utf8")
                        print("<<<",data)
                        data=bytes(input(">>>"),"utf8")
                        conn.sendall(data)
                    except Exception as e:
                        print(e)
                        conn.close()
                        break
    
    if __name__=="__main__":
        server=socketserver.ThreadingTCPServer(("0.0.0.0",8081),Myserver)  # 创建socketserver.ThreadingTCPServer实例，将地址及端口以及我们刚刚创建的类传递进去，这个类将会在客户端请求过来时threading一个线程来处理这个请求
        server.serve_forever()  # 调用实例的serve_forever方法
    #客户端实现
    import socket  # 这块代码和上面的基本一致，可以参照上面的注释解释
    sk=socket.socket()
    address=("127.0.0.1",8081)
    sk.connect(address)
    while True:
        data=bytes(input(">>>"),"utf8")
        if str(data,"utf8")=="exit":
            sk.close()
            break
        sk.sendall(data)
        data=str(sk.recv(1024),"utf8")
        print("<<<",data)

可以发现使用socketserver模块，比直接使用socket模块方便了很多，能够省去很多代码，并且能够通过多线程的使用来实现真正的多客户端的通信。

socketserver源码分析:
首先，socketserver提供了五个类供我们使用，他们之间的关系如下所示：

    +------------+
    | BaseServer |  # 基类
    +------------+
          |
          v
    +-----------+        +------------------+
    | TCPServer |------->| UnixStreamServer |
    +-----------+        +------------------+
          |
          v
    +-----------+        +--------------------+
    | UDPServer |------->| UnixDatagramServer |
    +-----------+        +--------------------+
而我们使用的类ThreadingTCPServer在源码中的定义方式如下所示：
class ThreadingTCPServer(ThreadingMixIn, TCPServer): pass
是通过继承ThreadingMixIn, TCPServer两个类实现的，那么，这里面帮我们干了些什么呢？首先，我们来分析server=socketserver.ThreadingTCPServer(("0.0.0.0",8081),Myserver)这条语句帮我们做了些什么，这条语句是创建一个类的对象，创建对象的时候会执行类的\_\_init\_\_方法，首先，找的是父类ThreadingMixIn的\_\_init\_\_方法，但是ThreadingMixIn中并没有，于是去找TCPServer中的。在TCPServer中的\_\_init\_\_方法帮我们做了一些事情，帮我们创建socket连接对象，并调用了父类BaseServer的\_\_init\_\_方法做了一些初始化工作。

然后server.serve_forever()这条语句帮我们做了什么事呢？

调用对象的serve_forever方法，首先去父类ThreadingMixIn中没找到,去TCPServer中也没找到，到父类BaseServer中找到了，代码如下：

    def serve_forever(self, poll_interval=0.5):  # poll_interval设置的是超时时间，以秒为单位，作为select的第四个可选参数传递进入
            self.__is_shut_down.clear()  # 设置event中的flag为False
            try:
                with _ServerSelector() as selector:
                    selector.register(self, selectors.EVENT_READ)  # 设置读事件
    
                    while not self.__shutdown_request:
                        ready = selector.select(poll_interval)  # 实例化select对象
                        if ready:
                            self._handle_request_noblock()  #  如果ready是真，获取客户端的请求
                        self.service_actions()
            finally:
                self.__shutdown_request = False
                self.__is_shut_down.set()

这个函数做的事情是调用了selecter实现非阻塞的I/O多路复用。并调用_handle_request_noblock-->process_request-->finish_request-->RequestHandlerClass(这个也就是我们创建对象时传进去的Myserver)这里面在初始化时-->setup-->handle(这个函数在原来的父类中是什么都没写的，我们给重写了这个函数，所以，调用到这的时候就是调用我们自己写的子类的函数执行我们的代码)


整个socketserver源码如下：

    #!/usr/bin/env python
    # encoding:utf-8
    # __author__: socket_server_resource_code
    # date: 2016/9/29 15:30
    # blog: http://huxianglin.cnblogs.com/ http://xianglinhu.blog.51cto.com/
    
    __version__ = "0.4"
    
    import socket  # 导入socket模块
    import selectors  # 导入selectors模块
    import os  # 导入os模块
    import twist
    try:
        import threading  # 导入threading模块
    except ImportError:  # 如果没有这个模块则导入dummy_threading模块并重命名为threading模块
        import dummy_threading as threading
    from time import monotonic as time  # 从time模块导入monotonic模块并重命名为time模块
    
    __all__ = ["BaseServer", "TCPServer", "UDPServer", "ForkingUDPServer",
               "ForkingTCPServer", "ThreadingUDPServer", "ThreadingTCPServer",
               "BaseRequestHandler", "StreamRequestHandler",
               "DatagramRequestHandler", "ThreadingMixIn", "ForkingMixIn"]
    """
    内置一个全局变量存储的是这个模块中的所有类名称
    """
    if hasattr(socket, "AF_UNIX"):  # 如果socket对象中有“AF_UNIX”，那么在__all__列表中添加下面几个方法
    
        __all__.extend(["UnixStreamServer","UnixDatagramServer",
                        "ThreadingUnixStreamServer",
                        "ThreadingUnixDatagramServer"])
    
    if hasattr(selectors, 'PollSelector'):  # 判断selectors对象中是否有PollSelector
        _ServerSelector = selectors.PollSelector
    else:
        _ServerSelector = selectors.SelectSelector
    
    class BaseServer:  # 基类
    
        timeout = None  # 超时时间，设置为空
    
        def __init__(self, server_address, RequestHandlerClass):
            """Constructor.  May be extended, do not override."""
            self.server_address = server_address  # 设置server的地址
            self.RequestHandlerClass = RequestHandlerClass  # 设置处理逻辑类
            self.__is_shut_down = threading.Event()  # 线程会阻塞在threading,Event()的地方，直到主线程执行完成后才会执行子线程
            self.__shutdown_request = False  # 设置一个bool
    
        def server_activate(self):
            pass
    
        def serve_forever(self, poll_interval=0.5):  # poll_interval设置的是超时时间，以秒为单位，作为select的第四个可选参数传递进入
            self.__is_shut_down.clear()  # 设置event中的flag为False
            try:
                with _ServerSelector() as selector:
                    selector.register(self, selectors.EVENT_READ)  # 设置读事件
    
                    while not self.__shutdown_request:
                        ready = selector.select(poll_interval)  # 实例化select对象
                        if ready:
                            self._handle_request_noblock()  #  如果ready是真，获取客户端的请求
                        self.service_actions()
            finally:
                self.__shutdown_request = False
                self.__is_shut_down.set()
    
        def shutdown(self):
            self.__shutdown_request = True
            self.__is_shut_down.wait()
    
        def service_actions(self):
            pass
    
        def handle_request(self):
            timeout = self.socket.gettimeout()
            if timeout is None:
                timeout = self.timeout
            elif self.timeout is not None:
                timeout = min(timeout, self.timeout)
            if timeout is not None:
                deadline = time() + timeout
            with _ServerSelector() as selector:
                selector.register(self, selectors.EVENT_READ)
    
                while True:
                    ready = selector.select(timeout)
                    if ready:
                        return self._handle_request_noblock()
                    else:
                        if timeout is not None:
                            timeout = deadline - time()
                            if timeout < 0:
                                return self.handle_timeout()
    
        def _handle_request_noblock(self):
            try:
                request, client_address = self.get_request()
            except OSError:
                return
            if self.verify_request(request, client_address):  # 永远返回true
                try:
                    self.process_request(request, client_address)
                except:
                    self.handle_error(request, client_address)
                    self.shutdown_request(request)
            else:
                self.shutdown_request(request)
    
        def handle_timeout(self):
            pass
    
        def verify_request(self, request, client_address):
            return True
    
        def process_request(self, request, client_address):
            self.finish_request(request, client_address)  # 调用处理逻辑类
            self.shutdown_request(request)  # 调用处理关闭逻辑类。。。其实什么都没干
    
        def server_close(self):
            pass
    
        def finish_request(self, request, client_address):
            self.RequestHandlerClass(request, client_address, self)  # 调用逻辑类
    
        def shutdown_request(self, request):
            self.close_request(request)
    
        def close_request(self, request):
            pass
    
        def handle_error(self, request, client_address):
            print('-'*40)
            print('Exception happened during processing of request from', end=' ')
            print(client_address)
            import traceback
            traceback.print_exc() # XXX But this goes to stderr!
            print('-'*40)
    
    
    class TCPServer(BaseServer):
    
        address_family = socket.AF_INET
    
        socket_type = socket.SOCK_STREAM
    
        request_queue_size = 5
    
        allow_reuse_address = False
    
        def __init__(self, server_address, RequestHandlerClass, bind_and_activate=True):  # 将传入的参数进行初始化
            """Constructor.  May be extended, do not override."""
            BaseServer.__init__(self, server_address, RequestHandlerClass)  # 调用父类构造方法，将传入参数赋值到对象上
            self.socket = socket.socket(self.address_family,
                                        self.socket_type)  # 设置socket对象，ip tcp模式
            if bind_and_activate:
                try:
                    self.server_bind()  # 绑定IP和端口
                    self.server_activate()  # 设置侦听
                except:
                    self.server_close()  # 关闭对象并引发异常
                    raise
    
        def server_bind(self):
            if self.allow_reuse_address:
                self.socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)  # 设置tcp的状态一般不会立即关闭而经历TIME_WAIT的过程。后想继续重用该socket
            self.socket.bind(self.server_address)  # 绑定IP和端口
            self.server_address = self.socket.getsockname()  # getsockname返回自己套接字的地址
    
        def server_activate(self):  # 设置侦听
            self.socket.listen(self.request_queue_size)
    
        def server_close(self):  # 关闭socket连接
            self.socket.close()
    
        def fileno(self):  # 返回socket的fileno信息
            return self.socket.fileno()
    
        def get_request(self):  # socket server设置准备就绪状态，等待接收客户端发来的信息
            return self.socket.accept()
    
        def shutdown_request(self, request):  # 关闭客户端socket的连接
            try:
                request.shutdown(socket.SHUT_WR)
            except OSError:
                pass
            self.close_request(request)
    
        def close_request(self, request):
            request.close()
    
    
    class UDPServer(TCPServer):
    
        allow_reuse_address = False
    
        socket_type = socket.SOCK_DGRAM
    
        max_packet_size = 8192
    
        def get_request(self):
            data, client_addr = self.socket.recvfrom(self.max_packet_size)
            return (data, self.socket), client_addr
    
        def server_activate(self):
            pass
    
        def shutdown_request(self, request):
            self.close_request(request)
    
        def close_request(self, request):
            pass
    
    class ForkingMixIn:
    
        timeout = 300
        active_children = None
        max_children = 40
    
        def collect_children(self):
            if self.active_children is None:
                return
            while len(self.active_children) >= self.max_children:
                try:
                    pid, _ = os.waitpid(-1, 0)
                    self.active_children.discard(pid)
                except ChildProcessError:
                    self.active_children.clear()
                except OSError:
                    break
            for pid in self.active_children.copy():
                try:
                    pid, _ = os.waitpid(pid, os.WNOHANG)
                    self.active_children.discard(pid)
                except ChildProcessError:
                    self.active_children.discard(pid)
                except OSError:
                    pass
    
        def handle_timeout(self):
            self.collect_children()
    
        def service_actions(self):
            self.collect_children()
    
        def process_request(self, request, client_address):
            pid = os.fork()
            if pid:
                if self.active_children is None:
                    self.active_children = set()
                self.active_children.add(pid)
                self.close_request(request)
                return
            else:
                try:
                    self.finish_request(request, client_address)
                    self.shutdown_request(request)
                    os._exit(0)
                except:
                    try:
                        self.handle_error(request, client_address)
                        self.shutdown_request(request)
                    finally:
                        os._exit(1)
    
    
    class ThreadingMixIn:
        daemon_threads = False
    
        def process_request_thread(self, request, client_address):
            try:
                self.finish_request(request, client_address)
                self.shutdown_request(request)
            except:
                self.handle_error(request, client_address)
                self.shutdown_request(request)
    
        def process_request(self, request, client_address):
            t = threading.Thread(target = self.process_request_thread,
                                 args = (request, client_address))
            t.daemon = self.daemon_threads
            t.start()
    
    
    class ForkingUDPServer(ForkingMixIn, UDPServer): pass
    class ForkingTCPServer(ForkingMixIn, TCPServer): pass
    
    class ThreadingUDPServer(ThreadingMixIn, UDPServer): pass
    class ThreadingTCPServer(ThreadingMixIn, TCPServer): pass
    
    if hasattr(socket, 'AF_UNIX'):
    
        class UnixStreamServer(TCPServer):
            address_family = socket.AF_UNIX
    
        class UnixDatagramServer(UDPServer):
            address_family = socket.AF_UNIX
    
        class ThreadingUnixStreamServer(ThreadingMixIn, UnixStreamServer): pass
    
        class ThreadingUnixDatagramServer(ThreadingMixIn, UnixDatagramServer): pass
    
    class BaseRequestHandler:
    
        def __init__(self, request, client_address, server):
            self.request = request
            self.client_address = client_address
            self.server = server
            self.setup()
            try:
                self.handle()
            finally:
                self.finish()
    
        def setup(self):
            pass
    
        def handle(self):
            pass
    
        def finish(self):
            pass
    
    class StreamRequestHandler(BaseRequestHandler):
    
        rbufsize = -1
        wbufsize = 0
        disable_nagle_algorithm = False
    
        def setup(self):
            self.connection = self.request
            if self.timeout is not None:
                self.connection.settimeout(self.timeout)
            if self.disable_nagle_algorithm:
                self.connection.setsockopt(socket.IPPROTO_TCP,
                                           socket.TCP_NODELAY, True)
            self.rfile = self.connection.makefile('rb', self.rbufsize)
            self.wfile = self.connection.makefile('wb', self.wbufsize)
    
        def finish(self):
            if not self.wfile.closed:
                try:
                    self.wfile.flush()
                except socket.error:
                    pass
            self.wfile.close()
            self.rfile.close()
    
    
    class DatagramRequestHandler(BaseRequestHandler):
    
        """Define self.rfile and self.wfile for datagram sockets."""
    
        def setup(self):
            from io import BytesIO
            self.packet, self.socket = self.request
            self.rfile = BytesIO(self.packet)
            self.wfile = BytesIO()
    
        def finish(self):
            self.socket.sendto(self.wfile.getvalue(), self.client_address)
