# Linux IO模型 #


## 前言 ##

在学习网络知识之前，我们应该对 Linux 网络 IO 模型有足够的认识，再学习建立在这之上的网络协议的数据通信便会变得简单易懂。首先，我们通过一张图来了解应用程序、操作系统和硬件三者之间的数据流转。

![](https://github.com/longtian2/cc3/blob/master/images/io/linux-io-arch.png)

通过上图，我们需要明确三件事：

1、当应用程序的进程/线程(后文将不加区分的只认为是进程)需要某段数据时，它只能在用户空间中属于它自己的内存中访问、修改，这段内存暂且称之为app buffer。

2、如果需要的数据在磁盘上，那么进程首先得发起相关系统调用，通知内核去加载磁盘上的文件。但正常情况下（这里指Linux 2.6 之前的版本未实现零复制机制的情况），数据只能加载到内核的缓冲区，暂且称之为kernel buffer。

3、数据加载到kernel buffer之后，还需将数据复制到app buffer，到了这里，进程就可以对数据进行访问、修改了。

疑惑解答

(1).为什么不能直接将数据加载到app buffer呢？

实际上是可以的，有些程序或者硬件为了提高效率和性能，可以实现内核旁路的功能，避过内核的参与，直接在存储设备和 app buffer 之间进行数据传输，例如，RDMA 技术就实现了内核旁路功能。

但是，最普通也是绝大多数的情况下，为了安全和稳定性，数据必须先拷入内核空间的kernel buffer，再复制到app buffer，目的是防止进程窜进内核空间进行破坏。

(2).上面提到的数据几次拷贝过程，拷贝方式是一样的吗？

不一样。现在的存储设备(包括网卡)基本上都支持 DMA 操作。什么是 DMA (direct memory access，直接内存访问)？简单地说，就是内存和设备之间的数据交互可以直接传输，不再需要计算机的CPU参与，而是通过硬件上的芯片(可以简单地认为是一个小cpu)进行控制。

假设，存储设备不支持 DMA ，那么数据在内存和存储设备之间的传输，必须通过计算机的CPU计算从哪个地址中获取数据、拷入到对方的哪些地址、拷入多少数据(多少个数据块、数据块在哪里)等等，仅仅完成一次数据传输，CPU都要做很多事情。而 DMA 就释放了计算机的 CPU ，让它可以去处理其他任务。

再说kernel buffer和app buffer之间的复制方式，这是两段内存空间的数据传输，只能由CPU来控制。

所以，在加载硬盘数据到kernel buffer的过程是DMA拷贝方式，而从kernel buffer到app buffer的过程是CPU参与的拷贝方式。

## I/O模型 ##

所谓的IO模型，描述的是出现I/O等待时进程的状态以及处理数据的方式。围绕着进程的状态，在数据准备和数据复制两个阶段展开。

1、数据从存储设备复制到 kernel buffer 的过程称为数据准备阶段，此阶段是不需要CPU参与

2、数据从kernel buffer复制到app buffer的过程称为数据复制阶段，此阶段是需要CPU参与

请记住这两个概念，后面描述I/O模型时会一直用这两个概念。

我们先从整体上来了解一下一个请求的数据流处理过程：

![](https://github.com/longtian2/cc3/blob/master/images/io/linux-io-tcp-server.png)

重点分析一下httpd的处理过程：

![](https://github.com/longtian2/cc3/blob/master/images/io/linux-io-tcp-stream-2.png)

## 阻塞 IO 模型 ##

最常用的 IO 模型就是阻塞 IO 模型，缺省情况下，所有文件操作都是阻塞的。应用程序进程从数据准备阶段开始到数据复制阶段结束都是阻塞状态。
 
Blocking IO 模型分析：

(1).当设置为 Blocking IO 模型，httpd从1开始到3结束都是被阻塞的。
(2).只有当数据复制到app buffer完成后，或者发生了错误，httpd才被唤醒处理它app buffer中的数据。
(3).cpu会经过两次上下文切换：用户空间到内核空间再到用户空间。
(4).由于2阶段的拷贝是不需要CPU参与的，所以在2阶段准备数据的过程中，cpu可以去处理其它进程的任务。
(5).3阶段的数据复制需要CPU参与，将httpd阻塞，在某种程度上来说，有助于提升它的拷贝速度。

如下图：

![](https://github.com/longtian2/cc3/blob/master/images/io/linux-io-block.png)

## 非阻塞 IO 模型 ##

应用程序进程在发起系统调用让内核准备数据的时候，如果 kernel buffer 中没有数据或者为完全准备好，就会直接返回一个 EWOULDBLOCK 错误，然后进行轮询检查该状态，直到数据准备好，该阶段是非阻塞的。数据复制阶段是阻塞的。

Non Blocking IO模型分析：

(1).当设置为 Non Blocking IO 模型，httpd第一次发起系统调用(如read())后，立即返回一个错误值EWOULDBLOCK (至于read()读取一个普通文件时能否返回EWOULDBLOCK请无视，毕竟I/O模型主要是针对套接字文件的，就当read()是recv()好了)，而不是让httpd进入睡眠状态。

(2).虽然read()立即返回了，但httpd还要不断地去发送read()检查内核：数据是否已经成功拷贝到kernel buffer了？这称为轮询(polling)。每次轮询时，只要内核没有把数据准备好，read()就返回错误信息EWOULDBLOCK。

(3).直到kernel buffer中数据准备完成，再去轮询时不再返回EWOULDBLOCK，而是将httpd阻塞，以等待数据复制到app buffer。

(4).httpd在1到2阶段不被阻塞，但是会不断去发送read()轮询。在3被阻塞，将cpu交给内核把数据copy到app buffer。

如下图：

![](https://github.com/longtian2/cc3/blob/master/images/io/linux-io-noblock.png)

## IO 多路复用模型 ##

顾名思义，IO 多路复用模型就是能够检查多个 IO 等待的状态。Linux 提供 select 和 poll，经常通过一个或者多个 fd （文件描述符）传递给 select  或 poll 系统调用，阻塞在 select 操作上，这样 select 或 poll 可以实现侦查多个 fd 是否处于就绪状态。select/poll是顺序扫描 fd 是否就绪，而且支持的 fd 数量有限，因此它的使用受到一些制约。Linux 还提供一个 epoll 系统调用，epoll 使用基于事件驱动方式代替顺序扫描，因此性能更高，当有 fd 就绪，立即触发回调函数。

I/O Multiplexing模型分析：

(1).当想要加载某个文件时，假如httpd要发起read()系统调用，如果是阻塞或者非阻塞情形，那么read()会根据数据是否准备好而决定是否返回，是否可以主动去监控这个数据是否准备到了kernel buffer中呢，亦或者是否可以监控send buffer中是否有新数据进入呢？这就是select()/poll()/epoll的作用。

(2).当使用select()时，httpd发起一个select调用，然后httpd进程被select()"阻塞"。由于此处假设只监控了一个请求文件，所以select()会在数据准备到kernel buffer中时直接唤醒httpd进程。之所以阻塞要加上双引号，是因为select()有时间间隔选项可用控制阻塞时长，如果该选项设置为0，则select不阻塞，此时表示立即返回但一直轮询检查是否就绪，还可以设置为永久阻塞。

(3).当select()的监控对象就绪时，将通知(轮询情况)或唤醒(阻塞情况)httpd进程，httpd再发起read()系统调用，此时数据会从kernel buffer复制到app buffer中并read()成功。

(4).httpd发起第二个系统调用(即read())后被阻塞，CPU全部交给内核用来复制数据到app buffer。 

(5).对于httpd只处理一个连接的情况下，IO复用模型还不如blocking I/O模型，因为它前后发起了两个系统调用(即select()和read())，甚至在轮询的情况下会不断消耗CPU。但是IO复用的优势就在于能同时监控多个文件描述符。

如图：

![](https://github.com/longtian2/cc3/blob/master/images/io/linux-io-multiplexing.png)

**select、poll、epoll异同：**

![](https://github.com/longtian2/cc3/blob/master/images/io/linux-poll-epoll-1.png)

![](https://github.com/longtian2/cc3/blob/master/images/io/linux-poll-epoll-2.png)

![](https://github.com/longtian2/cc3/blob/master/images/io/linux-poll-epoll-3.png)

## 信号驱动 IO 模型 ##

当开启信号驱动 IO 模型，并通过系统调用 sigaction 执行一个信号处理函数（此系统调用立即返回，进程继续工作，它是非阻塞的）。当数据准备就绪时，给进程发送 SIGIO 信号，通过信号回调通知应用程序来读取数据。

在发起信号处理的系统调用后，进程不会被阻塞，但是在read()将数据从kernel buffer复制到app buffer时，进程是被阻塞的。

如图：

![](https://github.com/longtian2/cc3/blob/master/images/io/linux-io-signal.png)

## 异步 IO 模型 ##

异步 IO 模型是由内核完成所有操作后再通知应用程序，即数据已复制到 app buffer 中，等待应用程序直接使用。

当设置为异步 IO 模型时，httpd首先发起异步系统调用(如aio_read()，aio_write()等)，并立即返回。这个异步系统调用告诉内核，不仅要准备好数据，还要把数据复制到app buffer中。

httpd从返回开始，直到数据复制到app buffer结束都不会被阻塞。当数据复制到app buffer结束，将发送一个信号通知httpd进程。

![](https://github.com/longtian2/cc3/blob/master/images/io/linux-io-asyn.png)

## 5中模型汇总 ##

阻塞、非阻塞、IO复用、信号驱动都是同步IO模型。因为在发起操作数据的系统调用(如本文的read())过程中是被阻塞的。这里要注意，虽然在加载数据到kernel buffer的数据准备过程中可能阻塞、可能不阻塞，但kernel buffer才是read()函数的操作对象，同步的意思是让kernel buffer和app buffer数据同步。显然，在保持kernel buffer和app buffer同步的过程中，进程必须被阻塞，否则read()就变成异步的read()。

只有异步IO模型才是异步的（即新开启后台线程） read()，因为发起的异步类的系统调用(如aio_read())已经不管kernel buffer何时准备好数据了，aio_read()可以一直等待kernel buffer中的数据，在准备好了之后，aio_read()自然就可以将其复制到app buffer。

如图：

![](https://github.com/longtian2/cc3/blob/master/images/io/linux-io-all-model.png)

最后，强调一点：jdk中实现的 NIO 是采用的 IO 多路复用模型
 

参考文献：

   《Netty权威指南（第二版）》 李林锋

   http://www.cnblogs.com/f-ck-need-u/p/7624733.html

联系方式：

https://github.com/longtian2

**如有用请不吝打赏**

![](https://github.com/longtian2/cc3/blob/master/images/wechat_pay.png)