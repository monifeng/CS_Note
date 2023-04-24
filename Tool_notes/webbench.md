# webbench

1）首先在执行：wget http://home.tiscali.cz/~cz210552/distfiles/webbench-1.5.tar.gz

2）然后解压：tar zxvf webbench-1.5.tar.gz

3）进入webbench-1.5，然后编译webbench。

4）make && make install



安装成功检测：

![image-20230423200057456](C:\Users\OMEN\AppData\Roaming\Typora\typora-user-images\image-20230423200057456.png)

```bash
which webbench
```



## 运行

```bash
 webbench -c 100 -t 10 -2 http://127.0.0.1:8000/     
```

-c clients

-t times

-2 http11

