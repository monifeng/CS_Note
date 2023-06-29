# 记录使用AddressSanitilzer真实debug的过程

## 前言

近期由于在学习MIT6.824，并自己在写一个C++版本，因为涉及多线程，遇到各种千奇百怪的bug，之前大部分都是靠肉眼debug来解决问题，在这个项目里出现的问题却迟迟没有解决，结果正确，但是master线程会因为segmentation fault直接退出，虽然并不影响，但始终让我心里有根刺，往后学习的时候，突然看到一篇介绍linux环境debug的文章，我就一路找到AddressSanitilzer介绍去，一经使用，**惊为天人！** 太好用了！一个小时解决了我之前两三天没能解决的问题！贴一个我看的博客的[链接](https://www.jianshu.com/p/3a2df9b7c353)。

贴一个我的项目地址（只有笔记）：[mit6.824](https://github.com/monifeng/6.824-Cpp.git)

贴一个我的源码地址：[mit6.824](https://gitee.com/moni_world/mit6.824-cpp.git)

喜欢就帮忙点个star，谢谢！

## 1、使用方法

编译器自带，直接在编译的时候加上即可

推荐使用Makefile，手打还是挺痛苦的。

```bash
g++ -o master.o -c -g master.cpp -std=c++11 -lzmq -pthread -g -fsanitize=thread -O2 -fno-omit-frame-pointer 
g++ -o master master.o -std=c++11 -lzmq -pthread -g -fsanitize=thread -O2 -fno-omit-frame-pointer
```



## 2、 真实过程回顾

然后执行可执行文件：

```bash
./master test.txt
#另一个终端
./worker
```

![image-20230506233221042](https://gitee.com/moni_world/pic_bed/raw/master/img/image-20230506233221042.png)

在master中报错，红色字体：

**SEGV on unknown address 0x0000000000e0 (pc 0x7f952a77cf74 bp 0x7f9526ebcac0 sp 0x7f9526ebc118 T11675)**

这个就是个空指针问题，推荐在vscode的终端里使用，这样ctrl+单击就可以直接跳转过去。

注意到#3这里：问题出现在 `mapWait()` 这个函数，仔细一看发现该函数在线程中运行，如果主函数退出了，master也就注销了，map_成为一个**空指针**！

解决办法：判断map_如果为null，直接退出即可。

```c++
if (map_ == NULL) return NULL;
```

![image-20230506233551464](https://gitee.com/moni_world/pic_bed/raw/master/img/image-20230506233551464.png)



## 3、注意事项

这些参数仅用于debug，如果运行程序，请去掉之前的参数（-fsanitize=thread等）；否则有些库函数的问题也会报错，导致程序无法正常运行。

![image-20230506234038623](https://gitee.com/moni_world/pic_bed/raw/master/img/image-20230506234038623.png)



## 4、总结

直接配合编译器使用，在编译时加上即可，debug完记得去掉sanitizer的参数。

本人太菜，如果发现错误，烦请指证，有什么问题也可以联系我。

**最后祝大家愉快都能愉快codeing~**

