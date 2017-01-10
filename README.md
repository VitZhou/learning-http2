# HTTP 2.0学习笔记（翻译）

## 介绍

超文本传输协议(http)是一个非常成功的协议.但是,http 1.1使用的底层传输方式([[RFC7230]](http://http2.github.io/http2-spec/index.html#RFC7230),[ Section 6](https://svn.tools.ietf.org/svn/wg/httpbis/specs/rfc7230.html#connection.management))有几个特性,对当前应用程序性能有负面影响。

特别是，HTTP/1.0只允许在一个连接上建立一个当前未完成的请求。HTTP/1.1管道只部分处理了请求并发和报头阻塞的问题。因此客户端需要发起多次请求通过数次连接服务器来减少延迟。

此外，HTTP/1.1的报头字段经常重复和冗长。在产生更多或更大的网络数据包时，可能导致小的初始[TCP](http://http2.github.io/http2-spec/index.html#TCP)堵塞窗口被快速填充。这可能在多个请求建立在一个新的TCP连接时导致过度的延迟。

HTTP/2.0通过定义一个优化的基础连接的HTTP语义映射来解决这些问题。 具体地，它允许在同一连接上交错地建立请求和响应消息，并使用高效率编码的HTTP报头字段。 它还允许请求的优先级，让更多的重要的请求更快速的完成，进一步提升了性能。

协议的设计是对网络更加友好的,因为与HTTP/1.X相比可以减少TCP连接.这意味着与其他流更少的竞争以及更长时间的连接,从而更有效的利用可用的网络容量.

最后，HTTP / 2还能够通过使用二进制消息成帧来更有效地处理消息。


## 导航
- [官网](http://http2.github.io/http2-spec/index.html#intro)

