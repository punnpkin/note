# HTTP

“HTTP 是一个在计算机世界里专门在两点之间传输文字、图片、音频、视频等超文本数据的约定和规范”。

是一个请求响应协议，运行在TCP之上应用层协议。

在互联网世界里，HTTP 通常跑在 **TCP/IP 协议栈**之上，依靠 **IP 协议**实现**寻址和路由**、**TCP 协议实现可靠数据传输**、**DNS 协议实现域名查找**、**SSL/TLS 协议实现安全通信**。此外，还有一些协议依赖于 HTTP，例如 WebSocket、HTTPDNS 等。这些协议相互交织，构成了一个协议网，而 HTTP 则处于中心地位。

## 1. 方法

常用的请求方法：GET方法，POST方法，HEAD，PUT，DELETE，CONNECT，OPTIONS，TRACE

* GET 用于使用给定的URI从给定服务器中检索信息
* POST 用于将数据发送到服务器以创建或更新资源，请求不会被缓存，长度没有限制
* PUT 用于将数据发送到服务器以创建或更新资源，它可以用上传的内容替换目标资源中的所有当前内容。

区别：PUT方法是幂等的：

* PUT请求：如果两个请求相同，后一个请求会把第一个请求覆盖掉。（所以PUT用来改资源）

* POST请求：后一个请求不会把第一个请求覆盖掉。（所以Post用来增资源）

区别*2：

* GET参数通过URL传递，POST放在Request body中
* GET请求会被浏览器主动cache，而POST不会，除非手动设置
* GET请求参数会被完整保留在浏览器历史记录里，而POST中的参数不会被保留

## 2. URL

HTTP使用统一资源标识符（Uniform Resource Identifiers, URI）来传输数据和建立连接。URL是一种特殊类型的URI，包含了用于查找某个资源的足够的信息，URL,全称是Uniform Resource Locator，中文叫统一资源定位符。

http://www.aspxfans.com:8080/news/index.asp?boardID=5&ID=24618&page=1#name

* 协议部分
* 域名部分
* 端口部分
* 虚拟目录部分
* 文件名部分
* 锚部分
* 参数部分

URI，是uniform resource identifier，统一资源标识符，用来唯一的标识一个资源。

## 3. 请求Request

```
请求 URL: https://www.baidu.com/
请求方法: GET
状态代码: 200 OK
远程地址: 39.156.66.18:443
```

```
Accept: text/plain, */*; q=0.01
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6
Connection: keep-alive
```

**请求行：**URL，方法和HTTP协议

**请求头部：**User-agent浏览器类型，Accept内容类型列表，Cookie：设置cookie,这是最重要的请求头信息之一，Host：初始URL中的主机和端口，Connection：表示是否需要持久连接“Keep-Alive”，请求使用的是HTTP 1.1（HTTP 1.1默认进行持久连接）

**请求数据：**post方法中，会把数据以key value形式发送请求

**空行：**

Get 参数少，大小有限制，地址栏显示内容

```
GET /562f25980001b1b106000338.jpg HTTP/1.1
Host    img.mukewang.com
User-Agent    Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.106 Safari/537.36
Accept    image/webp,image/*,*/*;q=0.8
Referer    http://www.imooc.com/
Accept-Encoding    gzip, deflate, sdch
Accept-Language    zh-CN,zh;q=0.8
```

Post相反

```
POST / HTTP1.1
Host:www.wrox.com
User-Agent:Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 2.0.50727; .NET CLR 3.0.04506.648; .NET CLR 3.5.21022)
Content-Type:application/x-www-form-urlencoded
Content-Length:40
Connection: Keep-Alive

name=Professional%20Ajax&publisher=Wiley
```

## 4. 响应Response

```
Cache-Control: private				缓存控制
Connection: keep-alive
Content-Encoding: gzip
Content-Type: text/html;charset=utf-8
```

**状态行、消息报头、空行和响应正文**

```html
HTTP/1.1 200 OK
Date: Fri, 22 May 2009 06:07:21 GMT
Content-Type: text/html; charset=UTF-8

<html>
      <head></head>
      <body>
            <!--body goes here-->
      </body>
</html>
```

状态行：

| 1**  | 信息，服务器收到请求，需要请求者继续执行操作   |
| ---- | :--------------------------------------------- |
| 2**  | 成功，操作被成功接收并处理                     |
| 3**  | 重定向，需要进一步的操作以完成请求             |
| 4**  | 客户端错误，请求包含语法错误或无法完成请求     |
| 5**  | 服务器错误，服务器在处理请求的过程中发生了错误 |

## 5. 打开网址的过程

1. DNS服务器请求解析（检查host文件）
2. 建立TCP连接，三次握手
3. 发送HTTP请求，请求行、请求头部、空行和请求数据4部分组
4. 服务器返回，服务器将资源复本写到TCP套接字，由客户端读取。一个响应由状态行、响应头部、空行和响应数据4部分组成
5. 主动释放or延时释放
6. 浏览器进行渲染

# HTTP协议版本对比

### http 1.0

**链接无法复用**，即不支持持久链接：
http 1.0 规定浏览器与服务器保持较短时间的链接，浏览器每次请求都和服务器经过三次握手和慢启动（基本思想是当TCP开始传输数据或发现数据丢失并开始重发时，首先慢慢的对网路实际容量进行试探，避免由于发送了过量的数据而导致阻塞）建立一个TCP链接，服务器完成请求处理后立即断开TCP链接，而且不跟踪每个浏览器的历史请求。一次连接只能获取一个资源。

注意：由于http 1.0每次建立TCP链接对性能的影响实在是太大，http1.1实现持久化链接之后，又反向移植到http 1.0上，只是默认是没有开启持久链接的，通过http的header部分的 Connection: KeepAlive 来启用）

**线头阻塞（Head of Line (HOL) Blocking）**
请求队列的第一个请求因为服务器正忙（或请求格式问题等其他原因），导致后面的请求被阻塞。

### http 1.1

**支持持久连接，支持管道**

通过http pipelining实现。多个http 请求可以复用一个TCP连接，服务器端按照FIFO原则来处理不同的Request。 

HTTP/1.1 中的连接都会默认启用长连接。不需要用什么特殊的头字段指定，只要向服务器发送了第一次请求，后续的请求都会重复利用第一次打开的 TCP 连接，也就是长连接，在这个连接上收发数据。

**缺点**

* 一个请求响应阻塞，就会阻塞后续所有请求
* 并行处理请求时，服务器必须缓冲管道中的响应，从而占用服务器资源，如果有个响应非常大
* 响应失败可能终止 TCP 连接

**http 1.1 增加了请求头和响应头来扩充功能**

### http 2.0

HTTP 2.0把解决性能问题的方案内置在了传输层，通过多路复用来减少延迟，通过压缩 HTTP首部降低开销，同时增加请求优先级和服务器端推送的功能。

**支持多路复用**

多路复用允许同时通过单一的 HTTP 2.0 **连接**发起多重的请求-响应消息，即所有HTTP 2.0 连接都是持久化的，而且客户端与服务器之间也只需要一个连接即可，所有数据流共用同一个连接 ，减少了因http链接多而引起的网络拥塞。

**二进制分帧**

即应用层(HTTP)和传输层(TCP or UDP)之间增加一个二进制分帧层，因此在多向请求和响应时，客户端和服务器可以把HTTP消息分解为互不依赖的帧，然后乱序发送，最后再在另一端把它们重新组合起来，解决了http 1.*的对手阻塞问题。

**首部压缩**
http 2.0支持DEFLATE和HPACK 算法的压缩

**服务端推送**

服务端推送是一种在客户端请求之前发送数据的机制

**请求优先级**
HTTP 2.0 使用一个31比特的优先值,0表示最高优先级, 2(31)-1表示最低优先级，服务器端就可以根据优先级，控制资源分配，优先处理和返回最高优先级的请求帧给客户端。

# websocket与http

## 开始

首先，Websocket是一个持久化的协议，相对于HTTP这种非持久的协议。

HTTP的生命周期通过 `Request` 来界定，也就是一个 `Request` 一个 `Response` ，那么在 `HTTP1.0` 中，这次HTTP请求就结束了。

在HTTP1.1中进行了改进，使得有一个keep-alive，也就是说，在一个HTTP连接中，可以发送多个Request，接收多个Response。但是请记住 `Request = Response`， 在HTTP中永远是这样，也就是说一个request只能有一个response。而且这个response也是被动的，不能主动发起。

Websocket

Websocket是基于HTTP协议的，或者说借用了HTTP的协议来完成一部分握手。

首先我们来看个典型的 `Websocket` 握手（借用Wikipedia的。。）

```
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
Origin: http://example.com
```

这段类似HTTP协议的握手请求中，多了几个东西

```
Upgrade: websocket
Connection: Upgrade
```

表示发起的是Websocket协议

```
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Protocol: chat, superchat
Sec-WebSocket-Version: 13
```

验证信息

**然后服务器**会返回下列东西，表示已经接受到请求， 成功建立Websocket啦！

```
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
Sec-WebSocket-Protocol: chat
```

这里开始就是HTTP最后负责的区域了，告诉客户，我已经成功**切换协议**啦~

```
Upgrade: websocket
Connection: Upgrade
```

后面两个也是验证

至此，HTTP已经完成它所有工作了，接下来就是完全按照Websocket协议进行了。具体的协议就不在这阐述了。

## Websocket的作用

HTTP 协议有一个缺陷：通信只能由客户端发起，这种单向请求的特点，注定了如果服务器有连续的状态变化，客户端要获知就非常麻烦。我们只能使用["轮询"](https://www.pubnub.com/blog/2014-12-01-http-long-polling/)：每隔一段时候，就发出一个询问，了解服务器有没有新的信息。最典型的场景就是聊天室。

轮询的效率低，非常浪费资源（因为必须不停连接，或者 HTTP 连接始终打开）

### ajax

ajax轮询的原理非常简单，让浏览器隔个几秒就发送一次请求，询问服务器是否有新信息。

### long poll

`long poll` 其实原理跟 `ajax`轮询差不多，都是采用轮询的方式，不过采取的是阻塞模型（一直打电话，没收到就不挂电话），也就是说，客户端发起连接后，如果没消息，就一直不返回Response给客户端。直到有消息才返回，返回完之后，客户端再次建立连接，周而复始。

场景：

客户端：啦啦啦，有没有新信息，没有的话就等有了才返回给我吧（Request）

服务端：额。。 等待到有消息的时候。。来 给你（Response）

客户端：啦啦啦，有没有新信息，没有的话就等有了才返回给我吧（Request） -loop

从上面可以看出其实这两种方式，都是在不断地建立HTTP连接，然后等待服务端处理，可以体现HTTP协议的另外一个特点，被动性。

何为被动性呢，其实就是，服务端不能主动联系客户端，只能有客户端发起。

**ajax轮询**需要服务器有很快的处理速度和资源。（速度）**long poll** 需要有很高的并发，也就是说同时接待客户的能力。（场地大小）

### WebSocket

通过上面这个例子，我们可以看出，这两种方式都不是最好的方式，需要很多资源。

此外，HTTP还是一个状态协议。

通俗的说就是，服务器因为每天要接待太多客户了，是个健忘鬼，你一挂电话，他就把你的东西全忘光了，把你的东西全丢掉了。你第二次还得再告诉服务器一遍。

所以在这种情况下出现了，Websocket出现了。他解决了HTTP的这几个难题。首先：

**被动性**，当服务器完成协议升级后（HTTP->Websocket），服务端就可以主动推送信息给客户端。

**减少消耗，**Websocket只需要一次HTTP握手，所以说整个通讯过程是建立在一次连接/状态中，也就避免了HTTP的非状态性，服务端会一直知道你的信息，直到你关闭请求



# HTTPS

端口：443

连接使用了**SSL**进行加密

HTTPS传输流程：

1. 首先客户端通过URL访问服务器建立SSL连接。
2. 服务端发送证书（公钥）到客户端
3. 客户端解析证书，生成随机值（用于**对称加密**），使用公钥对随机值**非对称加密**
4. 服务器使用私钥解密，把内容通过随机值加密
5. 对称加密后的信息传输给客户端
6. 客户端解密 

# 代理

代理有很多的种类，常见的有：

1. 匿名代理：完全“隐匿”了被代理的机器，外界看到的只是代理服务器；
2. 透明代理：顾名思义，它在传输过程中是“透明开放”的，外界既知道代理，也知道客户端；
3. 正向代理：靠近客户端，代表客户端向服务器发送请求；
4. 反向代理：靠近服务器端，代表服务器响应客户端的请求；



# 特点

## 无状态

“状态”其实就是客户端或者服务器里保存的一些数据或者标志，记录了**通信过程**中的一些变化信息。

你一定知道，TCP 协议是有状态的，一开始处于 CLOSED 状态，连接成功后是 ESTABLISHED 状态，断开连接后是 FIN-WAIT 状态，最后又是 CLOSED 状态。

这些**“状态”**就需要 TCP 在内部用一些数据结构去维护，可以简单地想象成是个标志量，标记当前所处的状态，例如 0 是 CLOSED，2 是 ESTABLISHED 等等。



1. HTTP 是灵活可扩展的，可以任意添加头字段实现任意功能；
2. HTTP 是可靠传输协议，基于 TCP/IP 协议“尽量”保证数据的送达；
3. HTTP 是应用层协议，比 FTP、SSH 等更通用功能更多，能够传输任意数据；
4. HTTP 使用了请求 - 应答模式，客户端主动发起请求，服务器被动回复请求；
5. HTTP 本质上是无状态的，每个请求都是互相独立、毫无关联的，协议不要求客户端或服务器记录请求相关的信息。

# Body

## 数据类型与编码

客户端：Accept, Accept-Encoding, Accept-Language: zh-CN, zh, en, Accept-Charset: gbk, utf-8

服务端：Content-Type, Content-Encoding, Content-Language: zh-CN

**MIME type**指明了类型：

1. text：即文本格式的可读数据，我们最熟悉的应该就是 text/html 了，表示超文本文档，此外还有纯文本 text/plain、样式表 text/css 等。
2. image：即图像文件，有 image/gif、image/jpeg、image/png 等。
3. audio/video：音频和视频数据，例如 audio/mpeg、video/mp4 等。
4. application：数据格式不固定，可能是文本也可能是二进制，必须由上层应用程序来解释。常见的有 application/json，application/javascript、application/pdf 等，另外，如果实在是不知道数据是什么类型，像刚才说的“黑盒”，就会是 application/octet-stream，即不透明的二进制数据。

**Encoding type**就少了很多，常用的只有下面三种：

1. gzip：GNU zip 压缩格式，也是互联网上最流行的压缩格式；
2. deflate：zlib（deflate）压缩格式，流行程度仅次于 gzip；
3. br：一种专门为 HTTP 优化的新压缩算法（Brotli）。

## 内容协商的质量值

在 HTTP 协议里用 Accept、Accept-Encoding、Accept-Language 等请求头字段进行内容协商的时候，还可以用一种特殊的“q”参数表示权重来设定优先级，这里的“q”是“quality factor”的意思。

```
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
```

权重的最大值是 1，最小值是 0.01，默认值是 1，如果值是 0 就表示拒绝。具体的形式是在数据类型或语言代码后面加一个“;”，然后是“q=value”。

在HTTP的内容协商里却恰好反了过来，“;”的**断句语气是小于**“,”的。

# 如何传输大文件

## 数据压缩

文本数据(`text/html`)可以使用gzip进行压缩

## 分块传输

chunked分块编码传输可以用于**大文件和流式数据传输**

```html
分块长度：length CRLF
分块数据：chunked data CRLF
分块长度：length CRLF
分块数据：chunked data CRLF
...

结束分块：0 CRLF
空行：CRLF
```

## 范围请求

服务器支持范围请求要发送`Accept-Ranges: bytes`

请求头**Range**是 HTTP 范围请求的专用字段，格式是“**bytes=x-y**”，其中的 x 和 y 是以字节为单位的数据范围。

有了范围请求之后，HTTP 处理大文件就更加轻松了，看视频时可以根据时间点计算出文件的 Range，不用下载整个文件，直接精确获取片段所在的数据内容。

### 断点续传

不仅看视频的拖拽进度需要范围请求，常用的下载工具里的多段下载、断点续传也是基于它实现的，要点是：

- 先发个 HEAD，看服务器是否支持**范围请求**，同时获取文件的大小；
- 开 N 个线程，每个线程使用 **Range 字段划分出各自负责下载的片段**，发请求传输数据；
- 下载意外中断也不怕，不必重头再来一遍，只要根据上次的下载记录，用 Range 请求剩下的那一部分就可以了。

### 多段数据

Range 头里使用多个“x-y”，一次性获取多个片段数据。

这种情况需要使用一种特殊的 MIME 类型：“**multipart/byteranges**”，表示报文的 body 是由多段字节序列组成的，并且还要用一个参数“**boundary=xxx**”给出段之间的分隔标记。



# 重定向

“**Location**”字段属于响应字段，必须出现在响应报文里。但只有配合 301/302 状态码才有意义，它**标记了服务器要求重定向的 URI**，浏览器会根据Location从字段里提取出URI发起新的HTTP请求。

**301**俗称“永久重定向”（Moved Permanently），意思是原 URI 已经“永久”性地不存在了，今后的所有请求都必须改用新的 URI。

**302**俗称“临时重定向”（“Moved Temporarily”），意思是原 URI 处于“临时维护”状态，新的 URI 是起“顶包”作用的“临时工”。

什么时候使用重定向？

* 资源不可用
* 重定向到主站

问题：

- 性能损耗，多使用了请求应答
- 循环跳转：自动跳出



