# Linux系统调用

主要记录一些不太熟悉的linux系统调用；



## 1、timerfd

### 01 初始化

```c++
int timerfd_create(int clockid, int flags);
```

第一个参数是指定用哪种时钟来计时：

主要有：

- `CLOCK_REALTIME`：绝对时间 **从1970年1月1日0点0分到目前的时间** ；
- `CLOCK_MONOTONIC`：相对时间 从系统启动到现在的时间；

其余省略，可以在man手册中查看

第二个参数flags：

- TFD_NONBLOCK：非阻塞调用；
- TFD_CLOSEXEC：fork后关闭共享的文件描述符，一般都会这样指定；



### 02 settime

函数原型：

```c++
int timerfd_settime(int fd, int flags,
                    const struct itimerspec *new_value,
                    struct itimerspec *old_value);
```

`fd`：前面初始化的timerfd；

`new_val`: 用于指定定时器第一次到期的时间和和后续的定时周期；

数据结构` itimerspec` ：

```c++
struct itimerspec {
    struct timespec it_interval;  /* Interval for periodic timer */
    struct timespec it_value;     /* Initial expiration */
};

struct timespec {
    time_t tv_sec;                /* Seconds */
    long   tv_nsec;               /* Nanoseconds */
};

```

由数据结构可以看出timerfd的精确度可以达到纳秒级；

> 1s = 1e3(ms) = 1e6 (us) = 1e9 (ns)

第一个 `it_interval `代表时间间隔：如果为0，就只启动一次，否则会启动多次；

第二个 `it_value` 代表第一次超时时间：从启动settime到第一次超时，就会中止定时器；



`flags` ：可以为0，即使用相对定时器；

> TFD_TIMER_ABSTIME：使用绝对定时器，new_value里的it_value是相对于选择的时钟的绝对时间（假如调用timerfd_settime()时clock的时间是8:00，此时flag为0，使用相对定时器，传递的new_value.it_value是10分钟，那么定时器就会在8:10触发；如果flag使用了TFD_TIMER_ABSTIME标志，使用绝对定时器，那么new_value.it_value就需要传递8:10，使用绝对时间）
> TFD_TIMER_CANCEL_ON_SET：只有在timerfd_create()里选择的时钟是CLOCK_REALTIME或CLOCK_REALTIME_ALARM才有意义，并且必须与标志TFD_TIMER_ABSTIME一起使用。当时钟经常发生不连续的变化时，这个定时器是可以取消的。前面也提到过，CLOCK_REALTIME对应的时钟是可变的。
> ————————————————
> 版权声明：本文为CSDN博主「爱就是恒久忍耐」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
> 原文链接：https://blog.csdn.net/whahu1989/article/details/103722557

`old_value` ：一般设为NULL足矣，否则设定为旧的timerfd的设定值（并不常用）；



### 03 gettime

```c++
int timerfd_gettime(int fd, struct itimerspec *curr_value);
```

真正的返回值在curr_value中，返回的是调用该函数的时间**离第一次到期的剩余时间**



### 04 实例

```c++
#include <sys/timerfd.h>
#include <sys/time.h>
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <unistd.h>


#define handle_error(msg) \
       do { perror(msg); exit(EXIT_FAILURE); } while (0)

void print_elapsed_time(void);

int main(void)
{
    int timerfd = timerfd_create(CLOCK_MONOTONIC, TFD_NONBLOCK);
    if (timerfd == -1)
    {
        handle_error("timerfd_create");
    }

    struct itimerspec new_value = {};
    new_value.it_value.tv_sec  = 10; // 10s
    new_value.it_value.tv_nsec = 0;
    
    new_value.it_interval.tv_sec  = 0; // one shot
    new_value.it_interval.tv_nsec = 0;

    if (timerfd_settime(timerfd, 0, &new_value, NULL) == -1)
    {
        handle_error("timerfd_settime");
    }

    print_elapsed_time();
    printf("timer started\n");


    uint64_t exp = 0;
    while (1)
    {
        int ret = read(timerfd, &exp, sizeof(uint64_t));
        
        if (ret == sizeof(uint64_t)) // 第一次定时到期
        {
            printf("ret: %d\n", ret);
            printf("expired times: %llu\n", exp);
            break;
        }

        // struct itimerspec curr;
        // if (timerfd_gettime(timerfd, &curr) == -1)
        // {
        //     handle_error("timerfd_gettime");
        // }
        // printf("remained time: %lds\n", curr.it_value.tv_sec);

        print_elapsed_time();
    }

    return 0;

}



void print_elapsed_time(void)
{
    static struct timeval start = {};
    static int first_call = 1;

    if (first_call == 1)
    {
        first_call = 0;
        if (gettimeofday(&start, NULL) == -1)
        {
            handle_error("gettimeofday");
        }
    }

    struct timeval current = {};
    if (gettimeofday(&current, NULL) == -1)
    {
        handle_error("gettimeofday");
    }

    static int old_secs = 0, old_usecs = 0;

    int secs  = current.tv_sec - start.tv_sec;
    int usecs = current.tv_usec - start.tv_usec;
    if (usecs < 0)
    {
        --secs;
        usecs += 1000000;
    }

    usecs = (usecs + 500)/1000; // 四舍五入

    if (secs != old_secs || usecs != old_usecs)
    {
    	printf("%d.%03d\n", secs, usecs);
    	old_secs = secs;
    	old_usecs = usecs;
    }
}


```



## 2、epoll

### 01 基本函数

初始化，监听，阻塞。

```C++
#include <sys/epoll.h>

int epoll_create(int size); // 旧版，size取值必须大于0
int epoll_create1(int flags); // 新版，flags可以是0，或者EPOLL_CLOEXEC

int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);

int epoll_wait(int epfd, struct epoll_event *events,
                      int maxevents, int timeout);

```



关键结构体

```c++
struct epoll_event {
    uint32_t     events;    /* Epoll events */
    epoll_data_t data;      /* User data variable */
};

typedef union epoll_data {
    void    *ptr;
    int      fd;
    uint32_t u32;
    uint64_t u64;
} epoll_data_t;

```



### 02 LT和ET的区别

水平触发就是只要在一个状态就会一直触发，而边缘触发是当环境发生改变的时候才会触发；

epoll默认是水平触发；



### 03 实例

```c++
/*
 * @Author: monifeng 1098264843@qq.com
 * @Date: 2023-05-05 16:40:18
 * @LastEditors: monifeng 1098264843@qq.com
 * @LastEditTime: 2023-05-05 16:59:38
 * @FilePath: /cpp-6.824/test/epoll_timerfd.cpp
 * @Description: 
 * 
 * Copyright (c) 2023 by monifeng, All Rights Reserved. 
 */
#include <sys/timerfd.h>
#include <sys/time.h>
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <unistd.h>
#include <sys/epoll.h>


#define handle_error(msg) \
       do { perror(msg); exit(EXIT_FAILURE); } while (0)

void print_elapsed_time(void)
{
    static struct timeval start = {};
    static int first_call = 1;

    if (first_call == 1)
    {
        first_call = 0;
        if (gettimeofday(&start, NULL) == -1)
        {
            handle_error("gettimeofday");
        }
    }

    struct timeval current = {};
    if (gettimeofday(&current, NULL) == -1)
    {
        handle_error("gettimeofday");
    }

    static int old_secs = 0, old_usecs = 0;

    int secs  = current.tv_sec - start.tv_sec;
    int usecs = current.tv_usec - start.tv_usec;
    if (usecs < 0)
    {
        --secs;
        usecs += 1000000;
    }

    usecs = (usecs + 500)/1000; // 四舍五入

    if (secs != old_secs || usecs != old_usecs)
    {
    	printf("%d.%03d\n", secs, usecs);
    	old_secs = secs;
    	old_usecs = usecs;
    }
}

int main(void)
{
    int timerfd = timerfd_create(CLOCK_MONOTONIC, TFD_NONBLOCK);
    if (timerfd == -1)
    {
        handle_error("timerfd_create");
    }

    struct itimerspec new_value = {};
    new_value.it_value.tv_sec  = 3; // 第一次1s到期
    new_value.it_value.tv_nsec = 0;

    new_value.it_interval.tv_sec  = 5; // 后续周期是5s cycle
    new_value.it_interval.tv_nsec = 0;

    if (timerfd_settime(timerfd, 0, &new_value, NULL) == -1)
    {
        handle_error("timerfd_settime");
    }

    print_elapsed_time();
    printf("timer started\n");

    int epollfd = epoll_create1(EPOLL_CLOEXEC);
    if (epollfd == -1) 
    {
        handle_error("epoll_create1");
    }

    struct epoll_event ev;
    ev.events = EPOLLIN;
    ev.data.fd = timerfd;
    
    epoll_ctl(epollfd, EPOLL_CTL_ADD, timerfd, &ev);

    const int maxEvents = 1; // 也可以设置为1
    struct epoll_event events[maxEvents]; 

    while(1)
    {
        int nfd = epoll_wait(epollfd, events, maxEvents, -1);

        if (nfd > 0)
        {
            for (int i = 0; i < maxEvents; ++i)
            {
                if (events[i].data.fd == timerfd)
                {
                    uint64_t exp = 0;
                    int ret = read(timerfd, &exp, sizeof(uint64_t));
                    if (ret != sizeof(uint64_t))
                    {
                        handle_error("read timerfd");
                    }
                    print_elapsed_time();   // 除第一次1s外，每5s打印一次
                }
            }
        }
    }

    return 0;
}
```

调试结果如下图：

![image-20230505170244981](https://gitee.com/moni_world/pic_bed/raw/master/img/image-20230505170244981.png)



