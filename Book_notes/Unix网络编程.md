# Unix网络编程

> 仅记录一些我认为非常优雅或非常重要的内容。

## 1、TCP服务器

### 1.1 慢系统调用中的信号处理

#### 慢系统调用

慢系统调用：指非常长的，可能会一直阻塞的系统调用，多数网络支持函数都是这一类，例如 **accept** 函数，如果服务器一直没有客户来链接，就会一直阻塞。

（1）读写‘慢’设备（包括pipe，终端设备，网络连接等）。读时，数据不存在，需要等待；写时，缓冲区满或其他原因，需要等待。

（2）当打开某些特殊文件时，需要等待某些条件，才能打开。例如：打开中断设备时，需要等到连接设备的modem响应才能完成。

（3）pause和wait函数。pause函数使调用进程睡眠，直到捕获到一个信号。wait等待子进程终止。



#### EINTER错误

EINTER错误通常指的是慢系统调用在阻塞时收到中断信号，从阻塞状态变为中断状态。

![img](https://pic2.zhimg.com/v2-3d28d43bf79323f8517bb548bc123369_r.jpg)

此时的处理方式，较为常见的是：重新调用该系统函数。



#### 重新调用系统函数

在UNP中，接收到的错误讯号，如果是EINTER，就会continue重新开始循坏，也就是重新从accept开始，如果信号为其他，就退出程序，并返回错误信息。

```c++
while(1)
{
	clilen = sizeof(cliaddr);
    
    if ( connfd = accept(listenfd, (SA *) &cliaddr, &clilen) < 0)
    {
        if (errno == EINTER)
            continue;
        else 
            err_sys("accept error")
    }
    if ((childpid = (fork()) == 0)
        {
            Close(listenfd); // 子进程仅处理，不建立连接
            str_echo(connfd);
            exit(0);
        }
    Close(connfd); // 父进程关闭连接，仅进行监听
    exit(0);
}	
```

