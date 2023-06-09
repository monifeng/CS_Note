## Lab2B 日志复制

### 01 流程概述

该流程基于论文，实现时可能有些许不同，但绝大部分应该符合论文的流程。

leader一旦被选举出来，就会开始处理客户端请求，客户端的每一条请求都会被复制状态机执行。leader首先将该指令添加到自己的日志中，然后并行地发起appendEntries给其他所有状态机。如果日志条目已经被大多数服务器复制，leader开始执行该指令，然后返回结果给客户端。（丢包也会持续发送）

leader决定了 `committed` 这个概念，也就是能安全应用到大部分状态机的就是 committed 的。我们需要保证所有 committed 的日志都是可持久化并且最终会被所有可用状态机执行的。一旦过半服务器接受了某个日志条目，立刻就变为 `committed` 状态，并且该条目之前的所有条目都会被提交，所以leader需要记录 `committed`的最高索引，然后在心跳AE中发送给所有状态机，然后follower执行已提交的日志条目（按顺序）。

raft需要维护的**日志匹配**机制：如果两个日志包含具有相同索引和任期的条目，则在给定索引之前的所有条目的内容都是相同的。§5.3

- 如果两个在不同日志的条目有相同的索引和任期，那么他们储存同样的指令
- 如果两个在不同日志中的条目有相同的索引和任期，这两个日志中该索引之前的条目都是相同的

follower如果在收到AE（包含最新committed最大索引和任期）后，找不到相同的日志索引和任期，拒绝AE请求。

raft中冲突的日志会被leader强制覆盖，然后来解决不一致问题，解决步骤如下：

1. 找到两者最新的相同日志，删除follower中所有的内容，然后将leader该条目后所有的内容发给follower。
2. leader为每个follower维护一个nextIndex，在leader一经选举以后，将所有的nextIndex初始化为该日志中最大索引之后的索引；
3. 具体实现：每次AE如果返回失败，就减少nextIndex，并重试AE，直到相同，然后删除follower该index之后日志，然后追加leader后面的所有日志，所以一旦AE返回成功，leader和follower的日志就会一致。

通过以上操作，就可以让leader自动与其他follower保持一致而不需要用特殊的处理，永不覆盖leader的日志，只会通过AE操作follower的日志。

Raft 永远不会通过计算副本数目的方式来提交之前任期内的日志条目，只有当前任期内的日志条目才会通过计算副本数目来提交。



### 02 理解测试

一开始不知道从何下手，浏览诸多资料，官方建议是从测试用例开始了解整个Lab2B需要做什么东西。

![img](https://img2022.cnblogs.com/blog/1729513/202209/1729513-20220913101403346-1209966514.png)





### 02 AE日志结构体设计

直接参考论文就行。

![image-20230530203151450](C:\Users\OMEN\AppData\Roaming\Typora\typora-user-images\image-20230530203151450.png)