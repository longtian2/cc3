# Mysql 架构及 Innodb架构 #

## Mysql 系统架构 ##

先来一张架构图（摘自官网），对Mysql 数据库有一个整体的认识。

![](https://github.com/longtian2/cc3/blob/master/images/mysql-arch.png)

- 连接器（connectors）
- 连接池组件（Connction pool ）
- 管理服务和工具组件（Management Serveices & Utilities）
- SQL接口组件（ SQL Interface）
- 查询分析器组件（Parser）
- 优化器组件（Optimizer）
- 缓冲组件（Cache & Buffer）
- 插件式存储引擎（Pluggable Storage Engines）
- 物理文件（Files & Logs）

Mysql数据库区别于其他数据库的最重要一个特点是插件式的表存储引擎。以下是各存储引擎的对比图（摘自《Mysql 技术内幕》）：

![](https://github.com/longtian2/cc3/blob/master/images/mysql-engine.png)

## Innodb 存储引擎架构 ##

![](https://github.com/longtian2/cc3/blob/master/images/Innodb-arch.png)

**Master Thread**

 Master Thread 是一个非常核心的后台线程，主要负责将缓冲池中的数据异步刷新到磁盘，保证数据的一致性，包括脏也的刷新、合并插入缓冲、Undo 页的回收等。

**IO Thread**

  IO Thread 的主要工作是负责这些 IO 请求的回调处理，细分成Write 、 Read 、 Insert Buffer 和 Log IO Thread。

**Purge Thread**

  Purge Thread 是在事物提交后，其所使用的Undo Log 可能不再需要，则由该线程来回收已经使用并分配的Undo 页。

**Page Cleaner Thread**

  Page Cleaner Thread 是在Innodb 1.2x 版本中引入。其作用是将之前版本中 Master Thread 脏页刷新回磁盘交由该线程处理，其目的是为了减轻Master Thread 的工作压力及用户查询线程的阻塞，提高Innodb 存储引擎的性能。

## Innodb 存储引擎工作机制 ##

![](https://github.com/longtian2/cc3/blob/master/images/Innodb-work.png)

**缓冲池**

缓冲池是为解决CPU速度与磁盘速度之前的鸿沟而设计的一块内存区域，通过内存的速度来弥补磁盘速度较慢对数据库性能的影响。数据库读取数据时，先检查缓冲池中是否存在该页，没有再从磁盘读取页，并放入缓冲池；数据库修改数据时，则先修改缓冲池中的页，称为脏页，然后再异步以一定的频率刷新到磁盘。

这里需要注意的是，脏页从缓冲池刷新回磁盘的操作并不是在每次页发生变更就触发，而是通过一种称为 Checkpoint 的机制来完成，其目的是为了提高数据库的性能。

数据库读取数据是从Free List中申请空闲页，申请到空闲页写入数据并放在LRU List中，如果没有则通过LRU List末端回收并重新分配页。在LRU List中的页被修改后，即缓冲池中的页与磁盘上的页的数据就不一致了，这时数据库会通过Checkpoint 机制将脏页刷新回磁盘，Flush List中的页即为脏页列表。需要注意的是，脏页既存在于LRU List中，也存在于Flush List中。LRU List 是用来管理缓冲池中页的可用性，Flush List是用来管理将脏页刷新回磁盘，二者互不影响。

**重做日志缓冲区**

Innodb 存储引擎首先将重做日志信息先写入到该缓冲区，然后再异步按一定频率将其刷新到重做日志文件。

重做日志在以下三种情况下会触发从重做日志缓冲区的页刷新到磁盘的重做日志（Redo Log）文件:

- Master Thread 每一秒触发
- 每个事物提交时会触发
- 重做日志缓冲区剩余空间小二分之一时触发

**Checkpoint 机制**

Checkpoint 机制解决的问题

- 缩短数据库的恢复时间
- 缓冲池不够用时，将脏页刷新回磁盘
- 重做日志不可用时，将脏页刷新回重做日志文件

Checkpoint 机制分类：

Sharp Checkpoint 发生在数据库关闭时将所有的脏页刷新回磁盘，这是默认的工作方式，即参数 innodb_fast_shutdown = 1。

Fuzzy Checkpoint 只刷新一部分脏页，而不是将所有的脏页刷新回磁盘。触发Fuzzy Checkpoint有以下4种情况：

- Master Thread Checkpoint
- FLUSH_LRU_LIST Checkpoint
- Async/Sync Flush Checkpoint
- Dirty Page Too Much Checkpoint




参考文献：

《Mysql技术内幕 Innodb存储引擎（第二版）》 姜承尧

联系方式：

https://github.com/longtian2

**如有用请不吝打赏**

![](https://github.com/longtian2/cc3/blob/master/images/wechat_pay.png)

