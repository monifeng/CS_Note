# Lab1

## 1、概要

完成三个函数的实现：

- master，worker，rpc

master调用worker，他们之间通过rpc来通信。输入一个文件，map输出中间文件，worker输出文件

![](https://gitee.com/moni_world/pic_bed/raw/master/img/image-20230428163702025.png)



## 2、Master设计

### 01 分析

Master的任务：

- 控制worker执行map和reduce操作；
- 当worker失效时将他的任务拷贝并分配给其他任务；



在paper中提到：

- 存储每个Map和Reduce的状态；
- Worker机器的标识；
- 存储了Map任务产生的R个中间文件的存储位置和大小；





