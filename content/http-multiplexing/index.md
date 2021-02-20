+++
title = "谈谈http/2的多路复用"
weight = 1
order = 1
date = 2021-02-11
insert_anchor_links = "right"
[taxonomies]
tags=["网络","http"]
+++
http协议的multiplexing首次在google的SPDY协议中提出，随后SPDY协议得到了Chrome、Firefox和opera的支持，而2015年正式批准的HTTP/2草案也正是基于SPDY。

HTTP/2与HTTP1.x的不同主要在于：
1. **降低延迟**，针对HTTP高延迟的问题，HTTP/2采取了多路复用（multiplexing）。多路复用通过多个请求stream共享一个tcp连接的方式，解决了HOL blocking的问题，降低了延迟同时提高了带宽的利用率。
2. **请求优先级（request prioritization）**。多路复用带来一个新的问题是，在连接共享的基础之上有可能会导致关键请求被阻塞。SPDY允许给每个request设置优先级，这样重要的请求就会优先得到响应。比如浏览器加载首页，首页的html内容应该优先展示，之后才是各种静态资源文件，脚本文件等加载，这样可以保证用户能第一时间看到网页内容。
3. **header压缩**。前面提到HTTP1.x的header很多时候都是重复多余的。选择合适的压缩算法可以减小包的大小和数量。
基于HTTPS的加密协议传输，大大提高了传输数据的可靠性。
4. **服务端推送**:采用了SPDY的网页，例如我的网页有一个sytle.css的请求，在客户端收到sytle.css数据的同时，服务端会将sytle.js的文件推送给客户端，当客户端再次尝试获取sytle.js时就可以直接从缓存中获取到，不用再发请求了。

## 队头阻塞
我们知道如果在HTTP1.x中，想要同时发送多个并行请求以提升性能，则必须使用多个[TCP链接](https://hpbn.co/http1x/#using-multiple-tcp-connections)。HTTP/1.x支持流水线模型(pipeling)，这种交付模型可以保证每次链接只交付一个响应(响应排队)，但是这种模型会导致队头阻塞问题(Head-of-line blocking,HOL blocking)。HOL问题的原因是一列的第一个数据包（队头）受阻而导致整列数据包受阻。HOL问题在HTTP中表现为：HTTP管道化要求服务端必须按照请求发送的顺序返回响应，那如果一个响应返回延迟了，那么其后续的响应都会被延迟，直到队头的响应送达。

而HTTP/2采用了多路复用来解决HOL问题，同时还实现了单一的TCP链接实现并行传输多个响应。

## 分帧
为什么HTTP/1.x无法使用多路复用而HTTP/2可以呢?因为HTTP/1.x的解析基于文本，而HTTP/2是基于二进制格式。HTTP/2增加了新的二进制分帧层，它定义了如何封装 HTTP 消息并在客户端与服务器之间传输。
![binary framing layer](./binary_framing_layer01.svg)
这里所谓的“层”，指的是位于套接字接口与应用可见的高级 HTTP API 之间一个经过优化的新编码机制：HTTP 的语义（包括各种动词、方法、标头）都不受影响，不同的是传输期间对它们的编码方式变了。 HTTP/1.x 协议以换行符作为纯文本的分隔符，而 HTTP/2 将所有传输的信息分割为更小的消息和帧，并采用二进制格式对它们编码。

一个帧由9字节的帧头和帧体组成,帧头的格式如下
```
  +-----------------------------------------------+
    |                 Length (24)                   |
    +---------------+---------------+---------------+
    |   Type (8)    |   Flags (8)   |
    +-+-------------+---------------+-------------------------------+
    |R|                 Stream Identifier (31)                      |
    +=+=============================================================+
    |                   Frame Payload (0...)                      ...
    +---------------------------------------------------------------+
```
帧头字段的定义如下：
1. **Length**:24bit，用于表示帧有效负载长度,即帧体(frame body)的长度。除非接收方为 SETTINGS_MAX_FRAME_SIZE 设置了较大的值(详情见[这里](https://github.com/halfrost/Halfrost-Field/blob/master/contents/Protocol/HTTP:2-HTTP-Frames-Definitions.md#2-defined-settings-parameters))，否则不得发送大于2 ^ 14（16,384）的值。
2. **Type**:8bit,用于表示帧类型。帧类型确定帧的格式和语义。实现方必须忽略并丢弃任何类型未知的帧。
3. **Flags**:8bit，是为特定于帧类型的布尔标志保留的 8 位字段，为标志分配特定于指示帧类型的语义。常用的标志位有`END_HEADERS`表示头数据结束，相当于HTTP/1中的空行("\r\n")，`END_STREAM`表示单方向数据发送结束(EOS,End of Stream)，相当于HTTP/1中的Chuncked分块结束标志("o\r\n\r\n")，没有定义语义类型的标志被忽略,并且在发送时保持未设置(0x0)
4. **R**:保留的1位字段，该位的语义未定义，发送保持未设置(0x0)，接收被忽略
5. **Stream Identifier**:31bit，为流标识符(参见[下文](#多路复用与流的优先级))

一个文件在HTTP1.x和HTTP2.0下可能分别是这样:
![数据分帧](./multyplexing.png)

## 数据流、消息和帧
首先需要明白HTTP/2中的三个概念:
- 数据流：已建立的连接内的双向字节流，可以承载一条或多条消息。
- 消息：与逻辑请求或响应消息对应的完整的一系列帧。（例如请求或响应）
- 帧：HTTP/2 通信的最小单位，每个帧都包含帧头，至少也会标识出当前帧所属的数据流。

它们的关系如下：一个http消息被分割组装成一个或多个二进制编码帧，再通过流进行运输。如下图所示：
![流消息帧](./streams_messages_frames01.svg)

## 多路复用与流的优先级



>参考
>1. [什么是队头阻塞以及如何解决](https://juejin.cn/post/6844903853985366023)
>2. [队头阻塞 wiki](https://zh.wikipedia.org/wiki/%E9%98%9F%E5%A4%B4%E9%98%BB%E5%A1%9E)
> 3. [HTTP/2简介](https://developers.google.com/web/fundamentals/performance/http2)
> 4. [HTTP/1,HTTP/1.1和HTTP2.0的区别](https://juejin.cn/post/6844903489596833800)
> 5. [MDN HTTP消息](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Messages)
>6. [HTTP/2 中的 HTTP 帧和流的多路复用](https://halfrost.com/http2-http-frames/)

