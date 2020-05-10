

大纲：

- http: 特点（请求响应，多个请求一个响应？一个请求多个响应？）、1.1vs2.0
- https: ssl, ca，证书、握手过程
- websocket
- keepalive

#Http协议

HTTP是一种简单可扩展的协议，其Client-Server的结构以及轻松扩展头部信息的能力使得HTTP可以和Web共同发展。

即使HTTP/2为了提高性能将HTTP报文嵌入到帧中这一举措增加了复杂度，但是从Web应用的角度看，报文的基本结构没有变化，从HTTP/1.0发布起就是这样的结构。

默认端口：80



###主要特点

1. 支持客户／服务器模式
2. 简单快速：客户向服务端请求服务时，只需传送请求方式和路径。
3. 灵活：允许传输任意类型的数据对象。由Content-Type加以标记。
4. 无连接：每次响应一个请求，响应完成以后就断开连接（1.0）。
5. 无状态：服务器不保存浏览器的任何信息。每次提交的请求之间没有关联。



### 示例解释

客户端请求

```http
GET / HTTP/1.1
Host: www.google.com
```

发出的请求信息（message request）包括以下几个:

- 请求行。方法、路径、协议版本（例如GET /images/logo.gif HTTP/1.1，表示从/images目录下请求logo.gif这个文件）
- [请求头](https://zh.wikipedia.org/wiki/HTTP头字段#请求字段)（例如Accept-Language: en）
- 空行
- 消息体



服务器应答

```http
HTTP/1.1 200 OK
Content-Length: 3059
Server: GWS/2.0
Date: Sat, 11 Jan 2003 02:44:04 GMT
Content-Type: text/html
Cache-control: private
Set-Cookie: PREF=ID=73d4aef52e57bae9:TM=1042253044:LM=1042253044:S=SMCc_HRPCQiqy
X9j; expires=Sun, 17-Jan-2038 19:14:07 GMT; path=/; domain=.google.com
Connection: keep-alive
```

响应报文包含了下面的元素：

- 响应行。HTTP协议版本号、一个状态码（[status code](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)）、一个状态信息（这个信息是非权威的状态码描述信息，可以由服务端自行设定）
- 响应头。HTTP [headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)，与请求头部类似。
- 空行。
- 消息体。可选项，比起请求报文，响应报文中更常见地包含获取的资源body。



###POST vs GET

| Post一般用于更新或者添加资源信息 | Get一般用于查询操作，而且应该是安全和幂等的 |
| -------------------------------- | :-----------------------------------------: |
| Post更加安全                     |       Get会把请求的信息放到URL的后面        |
| Post传输量一般无大小限制         |               Get不能大于2KB                |
| Post执行效率低                   |               Get执行效率略高               |


####

### HTTP1.0 VS HTTP1.1

在HTTP1.0，单一TCP连接内仅执行一个“客户端发送请求—服务器发送应答”周期，之后释放TCP连接。在HTTP1.1优化支持持续活跃连接：客户端连续多次发送请求、接收应答；批量多请求时，同一TCP连接在活跃（Keep-Live）间期内复用，避免重复TCP初始握手活动，减少网络负荷和响应周期。此外支持应答到达前继续发送请求（通常是两个），称为“流线化”（stream）。

HTTP1.0默认非持续性；HTTP 1.1支持持续性（PersistentConnection）和请求的流水线（Pipelining）处理。

> 持续性和非持续性：前者，浏览器和服务器建立TCP连接后，可以请求多个对象；后者，浏览器和服务器建立TCP连接后，只能请求一个对象
>
> 非流水线和流水线：流水线，不必等到收到服务器的回应就发送下一个报文；非流水线：发出一个报文，等到响应，再发下一个报文。



### HTTP/1.x vs HTTP/2

相比HTTP/1.x，HTTP/2有如下特点：

- **二进制格式**：HTTP/2 采用二进制格式传输数据，而非 HTTP/1.x 的文本格式。基于文本协议的格式解析存在天然缺陷，文本的表现形式有多样性，要做到健壮性考虑的场景必然很多，二进制则不同，只认0和1的组合。基于这种考虑HTTP2.0的协议解析决定采用二进制格式，实现方便且健壮。

  二进制分帧层中， HTTP/2 会将所有传输的信息分割为更小的消息和帧（frame）,并对它们采用二进制格式的编码 ，其中 HTTP1.x 的首部信息会被封装到 HEADER frame，而相应的 Request Body 则封装到 DATA frame 里面。

  http2.0的格式定义更接近tcp层的方式，这张二机制的方式十分高效且精简。length定义了整个frame的开始到结束，type定义frame的类型（一共10种），flags用bit位定义一些重要的参数，stream id用作流控制，剩下的payload就是request的正文了。虽然看上去协议的格式和http1.x完全不同了，实际上http2.0并没有改变http1.x的语义，只是把原来http1.x的header和body部分用frame重新封装了一层而已。调试的时候浏览器甚至会把http2.0的frame自动还原成http1.x的格式。

  <img src="https://pic4.zhimg.com/80/4bc1ad44e91207d56493003bf3805048_720w.jpg" width="320"/>

  

  <img src="https://pic3.zhimg.com/80/05563d500d43202464e1e246e8e69e9a_720w.jpg" width="480"/>

  

- **多路复用**。http2.0要解决的一大难题就是多路复用（MultiPlexing），即连接共享。上面协议解析中提到的stream id就是用作连接共享机制的。一个request对应一个stream并分配一个id，这样一个连接上可以有多个stream，每个stream的frame可以随机的混杂在一起，接收方可以根据stream id将frame再归属到各自不同的request里面。

- **头部压缩：**HTTP/2 对消息头采用 HPACK 进行压缩传输，能够节省消息头占用的网络的流量。而 HTTP/1.x 每次请求，都会携带大量冗余头信息，浪费了很多带宽资源。头压缩能够很好的解决该问题。

- **服务端推送（server push）**

  Server Push的功能前面已经提到过，http2.0能通过push的方式将客户端需要的内容预先推送过去，所以也叫“cache push”。另外有一点值得注意的是，客户端如果退出某个业务场景，出于流量或者其它因素需要取消server push，也可以通过发送RST_STREAM类型的frame来做到。



-----




#Https

**超文本传输安全协议**（英语：**H**yper**T**ext **T**ransfer **P**rotocol **S**ecure，缩写：**HTTPS**；常称为HTTP over TLS、HTTP over SSL或HTTP Secure）是一种通过[计算机网络](https://zh.wikipedia.org/wiki/計算機網絡)进行安全通信的[传输协议](https://zh.wikipedia.org/wiki/網路傳輸協定)。HTTPS经由[HTTP](https://zh.wikipedia.org/wiki/HTTP)进行通信，但利用[SSL/TLS](https://zh.wikipedia.org/wiki/传输层安全)来[加密](https://zh.wikipedia.org/wiki/加密)数据包。HTTPS开发的主要目的，是提供对[网站](https://zh.wikipedia.org/wiki/網站)服务器的[身份认证](https://zh.wikipedia.org/wiki/身份验证)，保护交换数据的隐私与[完整性](https://zh.wikipedia.org/wiki/完整性)。这个协议由[网景](https://zh.wikipedia.org/wiki/網景)公司（Netscape）在1994年首次提出，随后扩展到[互联网](https://zh.wikipedia.org/wiki/網際網路)上。

SSL（Secure Socket Layer，安全套接字层）：1994年为 Netscape 所研发，SSL 协议位于 TCP/IP 协议与各种应用层协议之间，为数据通讯提供安全支持。

TLS（Transport Layer Security，传输层安全）：其前身是 SSL，它最初的几个版本（SSL 1.0、SSL 2.0、SSL 3.0）由网景公司开发，1999年从 3.1 开始被 IETF 标准化并改名，发展至今已经有 TLS 1.0、TLS 1.1、TLS 1.2 三个版本。SSL3.0和TLS1.0由于存在安全漏洞，已经很少被使用到。TLS 1.3 改动会比较大，目前还在草案阶段，目前使用最广泛的是TLS 1.1、TLS 1.2。

端口号是443。



### **作用**

不使用SSL/TLS的HTTP通信，就是不加密的通信。所有信息明文传播，带来了三大风险。

> （1） **窃听风险**（eavesdropping）：第三方可以获知通信内容。
>
> （2） **篡改风险**（tampering）：第三方可以修改通信内容。
>
> （3） **冒充风险**（pretending）：第三方可以冒充他人身份参与通信。

SSL/TLS协议是为了解决这三大风险而设计的，希望达到：

> （1） 所有信息都是**加密传播**，第三方无法窃听。
>
> （2） 具有**校验机制**，一旦被篡改，通信双方会立刻发现。
>
> （3） 配备**身份证书**，防止身份被冒充。



### **握手阶段**

SSL/TLS协议的基本过程是这样的：

> （1） 客户端向服务器端索要并验证公钥。
>
> （2） 双方协商生成"对话密钥"。
>
> （3） 双方采用"对话密钥"进行加密通信。



详细过程

一，客户端发出请求（ClientHello）。客户端（通常是浏览器）先向服务器发出加密通信的请求，这被叫做ClientHello请求。在这一步，客户端主要向服务器提供以下信息。

> （1） 支持的协议版本，比如TLS 1.0版。
>
> （2） 一个客户端生成的随机数，稍后用于生成"对话密钥"。
>
> （3） 支持的加密方法，比如RSA公钥加密。
>
> （4） 支持的压缩方法。

二，服务器回应（SeverHello）。服务器收到客户端请求后，向客户端发出回应，这叫做SeverHello。服务器的回应包含以下内容。

> （1） 确认使用的加密通信协议版本，比如TLS 1.0版本。如果浏览器与服务器支持的版本不一致，服务器关闭加密通信。
>
> （2） 一个服务器生成的随机数，稍后用于生成"对话密钥"。
>
> （3） 确认使用的加密方法，比如RSA公钥加密。
>
> （4） 服务器证书。

三，客户端回应。客户端收到服务器回应以后，首先验证服务器证书。如果证书不是可信机构颁布、或者证书中的域名与实际域名不一致、或者证书已经过期，就会向访问者显示一个警告，由其选择是否还要继续通信。如果证书没有问题，客户端就会从证书中取出服务器的公钥。然后，向服务器发送下面三项信息。

> （1） 一个随机数。该随机数用服务器公钥加密，防止被窃听。
>
> （2） 编码改变通知，表示随后的信息都将用双方商定的加密方法和密钥发送。
>
> （3） 客户端握手结束通知，表示客户端的握手阶段已经结束。这一项同时也是前面发送的所有内容的hash值，用来供服务器校验。

其中，客户端和服务器同时有了三个随机数，接着双方就用事先商定的加密方法，各自生成本次会话所用的同一把"会话密钥"。（三个伪随机就十分接近随机了，每增加一个自由度，随机性增加的可不是一，增强安全性）

四，服务器的最后回应。服务器收到客户端的第三个随机数pre-master key之后，计算生成本次会话所用的"会话密钥"。然后，向客户端最后发送下面信息。

> （1）编码改变通知，表示随后的信息都将用双方商定的加密方法和密钥发送。
>
> （2）服务器握手结束通知，表示服务器的握手阶段已经结束。这一项同时也是前面发送的所有内容的hash值，用来供客户端校验。

至此，整个握手阶段全部结束。接下来，客户端与服务器进入加密通信，就完全是使用普通的HTTP协议，只不过用"会话密钥"加密内容。

示意图：

![img](https://pic2.zhimg.com/80/v2-7ae07af75faf459f780609c9710270ad_1440w.jpg)

总结：

- A机构颁发数字证书给服务端；
- 客户端和服务端进行SSL握手，客户端通过数字证书确定服务端的身份；
- 客户端和服务端传递三个随机数，第三个随机数通过非对称加密算法进行传递；
- 客户端和服务端通过一个对称加密算法生成一个对话密钥，加密接下来的通信内容。



其他：CA、数字证书、数字签名

https://juejin.im/post/5af557a3f265da0b9265a498#heading-35

# 参考

- [MDN-HTTP概述](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Overview)
- [HTTP/2 相比 1.0 有哪些重大改进？](https://www.zhihu.com/question/34074946/answer/108588042)
- [SSL/TLS协议运行机制的概述](https://www.ruanyifeng.com/blog/2014/02/ssl_tls.html)

