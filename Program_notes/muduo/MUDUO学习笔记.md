# MUDUO学习笔记

## 前言

> 木铎之心，素履之往。



## 0、需求分析

网络库的需求：

高级语言没有对Socket提供更高级的封装，开发一个网络库能够降低开发难度。

同时能够更方便的处理并发连接。（C10k）



## 1、设计模式：Reactor

### 1.1 网络设计模式

#### 01 阻塞IO

线程一直等待数据准备完毕，在这个过程中进程阻塞，直到数据准备完毕。

用户的大部分线程都在等待线程，浪费资源。



#### 02 非阻塞IO

轮询IO事件是否准备完毕，无论是否准备完毕，都会立即返回（即非阻塞）。也就是通过一个线程来监听所有网络连接的IO事件是否准备就绪，这也就是 **IO多路复用**。

Reactor模型就是基于IO多路复用构建起来的。



#### 03 事件驱动

事件驱动的核心：每当IO事件准备就绪时，以事件的方式通知IO读写线程进行数据的读写工作，然后业务线程直接对数据进行处理。

这种模式的好处是：读写进程和业务线程每次工作一定会有数据进行处理，免去了等待IO的过程。

Reactor模型的核心就是事件驱动。

Reactor正是通过IO多路复用来达到事件驱动的目的。



### 1.2 Reactor模型

- 单线程模型
- 多线程模型
- 主从多线程模型

通过两Reactor和Handler两大角色组成：

- Reactor：主要负责连接的建立，监听IO事件，IO事件读写以及将IO事件分发到Handlers进行处理
- Handler：主要负责业务逻辑的处理；



#### 01 单线程模型

单线程中，所有的IO操作，业务处理，都由一个线程完成；

优点：逻辑简单。

缺点：

- 处理的连接数有限，CPU很容易满，性能方面有屏障；
- 多个事件同时触发，前面的没处理完，后面就无法进行，会造成消息的积压以及请求超时；
- 处理IO事件时，无法同时处理业务的分发，连接的建立；
- IO线程如果一直满，就会造成服务端节点不可用；



#### 02 多线程模型

Handler不再进行业务的处理，而是交给线程池中的线程， 然后将执行结果回调到Handler，Handler再返回给客户端。

缺点：

- 线程的竞争会导致各种并发问题；
- 瞬间高并发场景会让单Reactor性能遇到瓶颈；



#### 03 主从多线程模型

主线程负责新连接的建立，从线程负责IO读写，事务分发。

解决了高并发问题，并且更好的利用了CPU多核的性能；



## 2、 muduo的主要函数回调机制

在本类中用一个函数，将需要进行的操作进行包裹，如read，该函数是在channel_中注册一个回调函数，channel是loop的一个成员类，根据one loop per thread的原则该类并不需要用互斥锁来保护，但这里就需要一个函数来检查所在循环是否是原本的IO线程。

然后在poller中进行注册监听，如果有事件，循环就执行，在循环中调用所有注册的回调函数。![在这里插入图片描述](https://img-blog.csdnimg.cn/fb494c8461004306b74f6c4f6480e6c2.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dhbmdnYW9fMTk5MA==,size_16,color_FFFFFF,t_70)





## 3、 遇到的问题

### 01 TcpConnection::send()中的race condition

在这里调用send，一般来说TcpConnectionPtr会一直持有到结束整个过程，但是如果TcpConnection释放了ptr，而runInLoop此时才调用send()的话，可能会导致指向空指针的异常。
（因为this指针会在析构TcpConnectionPtr时也析构）；

一个较好的调用方案是：shared_from_this()，这样就会持有这个this指针，达到了延续变量寿命的目的；

```c++
void TcpConnection::send(const StringPiece& message)
{
  if (state_ == kConnected)
  {
    if (loop_->isInLoopThread())
    {
      sendInLoop(message);
    }
    else
    {
      void (TcpConnection::*fp)(const StringPiece& message) = &TcpConnection::sendInLoop;
      loop_->runInLoop(
          std::bind(fp,
                    this,     // FIXME
                    message.as_string()));
                    //std::forward<string>(message)));
    }   
  }
}
```



### 02 TcpConnection不提供close

muduo 在 TCP 这一层面解决了“当你打算关闭网络连接的时候，如何得知对方有没有发了一些数据而你还没有收到？”

解决办法是，主动关闭写端，被动关闭读端；

关闭写端：shutdown();

关闭读端：muduo 在 read() 返回 0 的时候回调connection callback，这样客户代码就知道对方断开连接了。

完整流程：

我们发完了数据，于是 shutdownWrite，发送 TCP FIN 分节，对方会读到 0 字节，然后对方通常会关闭连接，这样 muduo 会读到 0 字节，然后 muduo 关闭连接。

这样就实现了对方发送的数据服务端完整收到。

真正的close socket会在析构TcpConnection的时候实现，析构函数会`close(sockfd_)`



### 03 回调函数机制

1. 注册可读可写的回调函数；
2. 等待可读可写的函数触发；
3. 触发后集中处理可读可写的回调函数；

### 04 accept返回EMFILE错误

文件描述符用尽：

在高并发的情况发，无论如何提升上限都可能会出错，所以一个比较优雅的办法是：

始终预留一个空闲的fd，遇到错误，立刻关闭空闲，接收socket，然后立刻关闭，打开空闲，这样就断开了客户端的连接。



## 4、 用法

### 4.1 创建服务端（TcpServer）

将一个TcpServer封装为自定义类的私有成员，并封装回调函数，在构造函数中接收loop和listenAddr，也就是循环和一个IP地址，InetAddress参数为一个端口就监听所有来源IP地址；构造函数中还使用server的set，set各种回调函数；

主函数中，首先初始化一个EventLoop；初始化一个InetAddress，然后用EventLoop的指针和InetAddress初始化EchoServer，调用start()，再调用loop()。

### 4.2 content，request，response



#### 01 content-type

用于定义网络文件的类型和网页的编码，常见类型有：

| content-type | description |
| :----------- | ----------- |
| text/html    | 超文本      |
| text/plain   | 纯文本      |
| image/png    | png文件     |



## 5. Makefile

设置变量：

```makefile
OBJ=xxx.o xx.o x.o
CFLAGS=-lm
CC=g++
```

运行httpserver的时候，需要在加上http库文件 `-lmuduo_http`;

muduo通用的Makefile模板如下：

```makefile
CC=g++
CFLAGS= -lmuduo_net -lmuduo_base -lpthread -lmuduo_base_http -std=c++11
FILE=server

%.o: %.cc
	$(CC) -c -o $@ $< $(CFLAGS)
# $(CC) $(CFLAGS) -c $^ -o $@

server: HttpServer_test.o
	$(CC) -o $@ $^ $(CFLAGS)
# $(CC) $(CFLAGS) $^ -o $@ 

.PHONY: clean

clean: 
	rm -rf *.o
```



## 6. benchmark

### 01 webbench的原理

Webbench 首先 fork 出多个子进程，每个子进程都循环做 web 访问测试。子进程把访问的结果通过pipe 告诉父进程，父进程做最终的统计结果。

根据多线程性能的分析（原文章链接：https://blog.csdn.net/pangzhaowen/article/details/106141365?spm=1001.2014.3001.5506）





> **线程等待时间所占比例越高，需要越多线程；线程CPU时间所占比例越高，需要越少线程。**

那么webbench仅仅是访问，并不是IO密集型计算，所以多线程更能发挥性能。

### 02 webbench测试

性能测试，使用webbench工具，对简单的静态http服务器进行访问；

webbench 10000 个并发连接，对单线程服务器muduo进行30s的测试

```bash
webbench -c 10000 -t 30 -2 http://127.0.0.1:8000/
```

![image-20230423214024832](C:\Users\OMEN\AppData\Roaming\Typora\typora-user-images\image-20230423214024832.png)

有一个错误，说明单线程的负载勉强达到10000，性能不太如意，也许是CPU的问题，所以此时可以选择多线程再进行测试。

```bash
 webbench -c 20000 -t 10 -2 http://127.0.0.1:8000/
```

在开启8个线程以后，明显感觉性能得到提升；

![image-20230423214603797](C:\Users\OMEN\AppData\Roaming\Typora\typora-user-images\image-20230423214603797.png)

即使是20000并发连接，也不会出现问题了



在开启4个线程时，性能得到了进一步提升

![image-20230423214901262](C:\Users\OMEN\AppData\Roaming\Typora\typora-user-images\image-20230423214901262.png)

测试链接数为30000也仅有少量错误，但是速度明显提升很大。

原因猜测是因为该linux系统内核设置为4，所以速度更快，但在处理并发连接数时，可能没有那么的有优势。



### 03 性能分析

单线程：

![image-20230423215945585](C:\Users\OMEN\AppData\Roaming\Typora\typora-user-images\image-20230423215945585.png)

多线程（4个）：

![image-20230423220001356](C:\Users\OMEN\AppData\Roaming\Typora\typora-user-images\image-20230423220001356.png)



对比两图，可以看出，单线程的速度和QPS几乎只是多线程情况下的一半，但是可能是因为测试时间过短地原因，单线程并没有出现错误；

此时将时间调制30s：

![image-20230423220636923](C:\Users\OMEN\AppData\Roaming\Typora\typora-user-images\image-20230423220636923.png)

单线程果然出现了错误，根据前面的分析，对于webbench，线程越多理应更快。

那么对多线程（4）进行测试：

![image-20230423221026630](C:\Users\OMEN\AppData\Roaming\Typora\typora-user-images\image-20230423221026630.png)

结果是，虽然QPS提升了非常多，两倍有余，但是错误数也非常明显的有提升，也许是因为CPU核数的问题，我的WSL是4核的linux系统，如果换成8核也许会不一样？

再对多线程（12）进行测试：

![image-20230423221451768](C:\Users\OMEN\AppData\Roaming\Typora\typora-user-images\image-20230423221451768.png)

此时发现，QPS提高更多，但是很神奇的是连接失败数居然减少了！继续增加线程数来试试：

24线程：

![image-20230423221821167](C:\Users\OMEN\AppData\Roaming\Typora\typora-user-images\image-20230423221821167.png)

### 疑问（尚未解决，希望各位大佬能提供思路或者更好的测试方法）



为什么线程越多，会让QPS提高很多，但并不能提高并发连接数呢（30000连接无论如何都跑不通）？

webbench到底是IO密集型还是CPU密集型的请求呢？

4核CPU情况开多少线程最能发挥出CPU的性能呢？（引申为n核CPU开多少线程最能发挥CPU的性能）。
