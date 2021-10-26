+++
title = "C10K Problem - I/O Strategies"
weight = 1
order = 1
date = 2021-10-26
insert_anchor_links = "right"
process = 0.333
[taxonomies]
tags=["I/O","translate"]
+++
> 这是一个web服务器已经能够处理10,000客户端连接的时代
[原文](http://www.kegel.com/c10k.html#strategies)

网络程序的设计者们有很多选择来解决C10K问题，下面就是一些例子:
<!-- TODO concurrency：并发or并行-->
- 是否以及如何从单个线程发出多个I/O调用
  - 不要：直接使用blocking/synchrounous调用，并且可能使用多线程或者进程来达成并发
  - 使用nonblocking调用（例如，将socket中的write（）设置为O_NONBLOCK）来启动一个I/O,并使用就绪通知(例如poll()或/dev/poll)来知道何时可以在该通道上启动下一个I/O。一般只能用于网络I/O，而不能用于磁盘I/O。
  - 使用异步调用（例如 aio_write()）来启动 I/O，并使用完成通知（例如信号或完成端口）来了解 I/O 何时完成。适用于网络和磁盘 I/O。

- 如何控制服务于每个客户端的代码
  - 每个客户端一个进程（经典的 Unix 方法，自 1980 年左右开始使用）
  - 一个os级线程处理多个客户端;每个客户端由:
    - 一个用户级线程(例如GNU状态线程，例如典型的带有绿色线程Java)
    - 状态机（有点深奥，但在某些圈子里很流行；我最喜欢的）
    - [continuation](https://www.zhihu.com/question/61222322)（有点深奥，但在某些圈子中很流行）
  - 每个客户端有一个操作系统级线程（例如具有典型的原生线程的Java）
  - 每个活动客户端都有一个操作系统级线程（例如，带有apache前端的 Tomcat；NT 完成端口；线程池）
- 是使用标准的 O/S 服务，还是将一些代码放入内核（例如在自定义驱动程序、内核模块或 VxD 中）

### 1. 每个线程为多个客户端提供服务，并使用非阻塞 I/O 和电平触发（level triggerd）的就绪通知

...将所有的network handles设置为非阻塞模式，并使用`select()`或`poll()`来判断哪个network handle有数据等待。传统上往往采用这种方法。使用这个方案，内核会告诉你一个文件描述符是否准备好了以及自上次内核回应以后你是否对这个文件描述符做过任何事情。(“电平触发”这个名字来自于计算机硬件设计；它与“[边缘触发(edge triggerd)](http://www.kegel.com/c10k.html#nb.edge)”相反。Jonathon Lemon在他[关于kqueu()的BSDCON 2000论文](http://people.freebsd.org/~jlemon/papers/kqueue.pdf)中介绍了这些术语。)


注意:特别重要的是要记住，来自内核的准备就绪通知只是一个提示;当您试图从文件描述符中读取时，它可能还未准备就绪。这就是为什么在使用就绪通知时使用非阻塞模式（nonblocking mode）很重要。


这种方法的一个重要瓶颈是，如果页面目前不在内核中(发生缺页)，那么从磁盘块`read()`或`sendfile()`，在磁盘文件句柄上设置非阻塞模式没有效果;对于内存映射磁盘文件也是如此。当服务器第一次需要磁盘I/O时，它的进程会阻塞，所有客户端都必须等待，原始的非线程性能就会被浪费。

这就是异步I/O的作用，但在缺乏AIO的系统上，执行磁盘I/O的工作线程或进程也可以绕过这个瓶颈。一种方法是使用内存映射文件，如果mincore()表明需要I/O，请工作人员执行I/O，并继续处理网络流量。jeff Poskanzer提到，Pai、Druschel和Zwaenepoel的1999年Flash网络服务器就使用了这个技巧;他们在Usenix '99上做了一个演讲。看起来mincore()在FreeBSD和Solaris等bsd衍生的Unix中是可用的，但它不是单一Unix规范的一部分。它作为Linux内核2.3.51的一部分可用，感谢Chuck Lever。



对于单个线程来说，有几种方法可以判断一组nonblocking socket中哪一个已经为I/O做好了准备: