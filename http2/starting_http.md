# 3.启动HTTP 2.0
一个HTTP/2连接是运行在TCP连接上的应用层协议.客户端是TCP连接的发起者.

HTTP/2使用与HTTP/1.1相同的"http"和"https"资源标识符(URI).使用相同的默认端口:"http"的80端口和"https"的443端口.因此,实现对例如`http://example.org/foo`或`https://example.com/bar`目标资源的URI请求处理需要首先确定上游服务端(当前客户端希望建立连接的对等端)是否支持HTTP/2

确定对HTTP/2的支持的手段对于“http”和“https”URI是不同的,检测"http"URIs在章节3.2中描述。检测"https"URIs 在章节3.3中描述。

## 3.1 HTTP/2版本标识
在本文档中定义的协议有两个标识符。
- 字符"h2"表示HTTP/2协议使用 [Transport Layer Security[TLS]](http://http2.github.io/http2-spec/index.html#TLS12)。这种方式用在HTTP/1.1的升级字段、TLS 应用层协议协商扩展字段以及其他需要定义协议的地方。当在定义[ALPN协议](http://http2.github.io/http2-spec/index.html#TLS-ALPN)(序列化的字节)中序列化时。 "h2"字符序列化到 ALPN 协议中变成两个字节序列：0x68，0x32。
- 字符"h2c" 表示HTTP/2协议运行在明文TCP上。这个标识用在HTTP/1.1 升级报头字段以及任何TCP是确定的地方。“h2c”字符从ALPN标识符空间保留，但描述不使用TLS的协议。

用到"h2" 或者 "h2c" 表明使用文档中定义的传输、安全、帧及语义化消息。

## 3.2 为“http”URI启动HTTP/2
客户端无法预知服务端是否支持HTTP/2.0 的情况下使用HTTP升级机制发起“http” URI请求([RFC7230](http://http2.github.io/http2-spec/index.html#RFC7230) [章节6.7](https://svn.tools.ietf.org/svn/wg/httpbis/specs/rfc7230.html#header.upgrade))。客户端通过做出包括具有“h2c”令牌的升级(Upgrade)报头字段的HTTP/1.1请求来做升级。HTTP/1.1必须包含一个确切的[HTTP2-Settings](http://http2.github.io/http2-spec/index.html#Http2SettingsHeader)报头字段。
例如:
```json
    GET / HTTP/1.1
    Host: server.example.com
    Connection: Upgrade, HTTP2-Settings
    Upgrade: h2c
    HTTP2-Settings: <base64url encoding of HTTP/2 SETTINGS payload>
```
包含payload body的请求必须在客户端能发送HTTP/2帧前全部发送。这意味着一个大的请求实体能阻塞连接的使用直到其全部被发送。

如果一个请求的并发后续请求是重要的，那么可以使用一个小的请求来执行升级到HTTP/2的操作,这样仅消耗一个额外的往返成本。

不支持HTTP/2的服务端对请求返回一个不包含升级(Upgrade)的报头字段的响应：
```json
    HTTP/1.1 200 OK
    Content-Length: 243
    Content-Type: text/html

    ...
```
服务端必须忽略升级报头字段中的“h2” token。“h2” token基于TLS实现的HTTP/2,协商方法在章节3.3中定义。

支持HTTP/2的服务端可以返回一个101(转换协议)响应来接受升级请求。在101空内容响应终止后，服务端可以开始发送HTTP/2帧。这些帧必须包含一个发起升级的请求的响应。
例如:
```json
    HTTP/1.1 101 Switching Protocols
    Connection: Upgrade
    Upgrade: h2c

    [ HTTP/2 connection ...
```
服务器发送的第一个HTTP/2帧必须是由设置(SETTING)帧组成的服务器连接序言.收到101响应后,客户端必须发送一个包含设置(SETTING)帧的连接序言。
在升级之前发送的HTTP/1.1请求被分配具有默认优先级最高的流标识符1。 流1从客户端到服务器是隐式“half-closed”，因为请求作为HTTP/1.1请求完成了。 开启HTTP/2连接后，流1用于响应。

### 3.2.1 HTTP2-Settings 报头属性
从HTTP/1.1升级到HTTP/2的请求必须包含一个确切的HTTP2-Settings报头字段。HTTP2-Settings 的报头字段是逐跳报头字段，它包含管理HTTP/2连接参数。这是从对于服务端接受升级请求的预测中所获取的。

HTTP2-Settings = token68

服务端未检测到此报头字段必须拒绝客户端的升级尝试。服务端绝对不能发送此报头字段。

HTTP2-Settings报头字段的内容是设置(SETTINGS)帧的有效载体，使用base64url字符编码(URL及文件名安全的Base64编码，编码描述在[RFC4648](http://http2.github.io/http2-spec/index.html#RFC4648) [章节5](https://tools.ietf.org/html/rfc4648#section-5)中，忽略任何“=”字符。) [ABNF](http://http2.github.io/http2-spec/index.html#RFC5234)[RFC5234]产品中对token68的定义在[RFC7235](http://http2.github.io/http2-spec/index.html#RFC7235) [章节2.1](https://svn.tools.ietf.org/svn/wg/httpbis/specs/rfc7235.html#challenge.and.response)中。

由于升级仅适用于即时连接，因此发送报头字段HTTP2-Settings的客户端还必须在报头字段Connection中发送HTTP2-Settings作为连接选项，以防止它被转发（参见[RFC7230](http://http2.github.io/http2-spec/index.html#RFC7230)）。

服务端就像对任何其他设置(SETTINGS)帧一样对这些值进行解码和解释。因为101响应的隐式声明，对这些设置参数的确认不是必须的。这些升级请求中的值使得协议不需要上述设置参数的默认值，同时使客户端有机会在从服务端接受任何帧之前提供其他参数。

## 3.3 针对“https”启动HTTP/2
客户端在不了解服务端是否支持HTTP/2的时候，会使用[TSL](http://http2.github.io/http2-spec/index.html#TLS12)[TLS12] 于其应用层协议协商扩展 [TLSALPN](http://http2.github.io/http2-spec/index.html#TLS-ALPN)。

使用TLS的HTTP/2使用"h2"协议标识符。“h2c”token绝对不能由客户端发送或者由服务端选择;“h2c”协议标识符描述不使用TLS的协议。

一旦TLS协商完成，客户端和服务器必须发送连接序言

## 3.4 Starting HTTP/2 with Prior Knowledge(大多数翻译是:先验下启动HTTP/2)
客户端可以通过其他方式判断服务端是否支持HTTP/2。例如，[ALT-SVC](http://http2.github.io/http2-spec/index.html#ALT-SVC)定义一种机制让HTTP头字段进行广播。

客户端可以对支持HTTP/2的服务端在连接序言(章节3.5)之后立即发送HTTP/2帧。服务端可以通过连接序言中的存在来识别这些连接。这只对基于明文TCP的HTTP/2连接建立有影响；支持HTTP / 2 over TLS的实现必须在[TLS](http://http2.github.io/http2-spec/index.html#TLS-ALPN) [TLS-ALPN]中使用协议协商。

同样，服务器必须发送连接序言（第3.5节）。

没有其他信息，对HTTP/2的预先支持不是很强求的信号，即给定服务器将支持HTTP/2用于将来的连接。例如，服务器配置可能更改，配置在群集服务器中的实例之间不同，或网络条件更改。

