# 概述
HTTP/2为HTTP语义提供了更优化的传输.HTTP/2支持HTTP/1.1的所有核心功能.并且在不同的方面做的更高效.

HTTP/2的基本协议单元是帧。每个帧都有不同的用途和类型.例如,报头(HEADERS)和数据(DATA)帧是HTTP请求和响应基础;其他类型帧(如设置(SETTINGS),窗口更新(WINDOW_UPDATE)和推送承诺(PUSH_PROMISE)用于支持其他的HTTP/2特性.

请求多路复用是通过在一个流上分配多个HTTP request response交换来实现的。流在很大程度上是相互独立的，因此一个请求上的阻塞或终止并不会影响其他请求的处理。

流量控制和优先级能确保正确使用复用流。流量控制有助于确保只传递接收者需要使用的数据。优先级能确保有限的资源能优先被更重要的请求使用。

HTTP/2添加了一种新的交互模式，即服务器能推送消息给客户端。服务器推送允许服务端预测客户端需要来发送数据给客户端，交换网络的使用来阻止潜在的延迟增长。服务器通过复用一个以推送承诺(PUSH_PROMISE)帧发送的请求来实现推送，然后服务端可以在一个单独的流里面发送响应给这个合成的请求。

由于连接中使用的http报头字段可能包含大量冗余数据,因此包含它们的帧将被压缩.允许许多请求被压缩到一个分组中,这在公共情况下对请求大小特别有利。

## 文档结构
HTTP/2规范分为四个部分:
- 启动HTTP/2:介绍如何启动HTTP/2连接
- 帧和流层:描述HTTP/2帧被构造和形成为多路复用流的方式
- 真和错误码:定义了HTTP/2中使用的流和错误类型的详细内容
- HTTP寻址和拓展需求:描述了HTTP语义化是如何由帧和流表达的。

一些帧和流层的概念是与HTTP隔离的，因为并不是定义一个完全通用的帧层。这些帧和流层是为了HTTP协议和服务端推送的需求定制的。

## 约定和术语
文档中出现的关键字“必须(must)”,“绝对不能(must not)”，“要求(REQUIRED)”，“应(SHALL)”，“不应(SHALL NOT)”，“应该(SHOULD)”，“不应该(SHOULD NOT)”，“建议(RECOMMENDED)”，“可以(MAY)”及“可选(OPTIONAL)”可通过在 [RFC 2119](http://http2.github.io/http2-spec/index.html#RFC2119) 的解释进行理解。

所有的数值都是按网络字节顺序。除非有另外说明，数值是无符号的。按情况提供十进制或十六进制的文本值。十六进制用前缀0x来区分。

文中术语包括:
- client(客户顿): 发起HTTP/2请求的端点
- connection(连接): 在连个端点之间的传输层级别的连接
- connection error(连接错误): 整个HTTP/2连接过程中发生的错误
- endpoint(端点): 连接的客户端或者服务端
- frame(帧): HTTP/2.0通信连接中的最小单元，由报头和根据帧类型构造的可变长度的八位字节序列组成
- peer(对等端): 一个端点。当讨论特定的端点时，“对等端”指的是讨论的主题的远程端点
- receiver(接收端): 正在接收帧的端点
- sender(发送端): 正在传输帧的端点
- server(服务器): 接受HTTP/2连接的端点。 服务器接收HTTP request并发送HTTP response
- stream(流): 一个双向字节帧流穿过HTTP/2连接中的虚拟通道
- stream error(流错误)：一个HTTP/2流中的错误

最后，术语“gateway(网关)”，“intermediary(媒介)”，“proxy(代理)”和“tunnel(隧道)”在[RFC7230](http://http2.github.io/http2-spec/index.html#RFC7230)的[第2.3节](https://svn.tools.ietf.org/svn/wg/httpbis/specs/rfc7230.html#intermediaries)中定义。 媒介在不同时间充当客户端和服务器。

术语“(payload body)有效载荷主体”在[RFC7230](http://http2.github.io/http2-spec/index.html#RFC7230)的[第3.3节](https://svn.tools.ietf.org/svn/wg/httpbis/specs/rfc7230.html#message.body)中定义。


