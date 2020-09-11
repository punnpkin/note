# 服务器单机最大TCP连接数是多少？

在没接触过这个问题之前，自然会想到服务器端连接数是由**服务器端口号**限制的。但这其实是一个很严重的误解，要解决这个问题，必须理解socket的连接过程。

以python为例，tcp服务端socket需要经过如下的初始化步骤：

```python
import socket

# 建立socket对象
s = socket.socket()
# 初始化地址，注意这里用的是一个元组
addr = ("127.0.0.1", 43211)
# 绑定
s.bind(addr)
# 监听
s.listen(5)					
```

这时服务端的socket就准备好了，进到下一步等待connect的阶段：accept。看一下accept的说明：

> Accept a connection. The socket must be bound to an address and listening for connections. The return value is a pair `(conn, address)` where *conn* is a *new* socket object usable to send and receive data on the connection, and *address* is the address bound to the socket on the other end of the connection.

accept会返回一个元组，包含一个新的**socket对象**和客户端的地址。这个新的对象用于发送或接收数据。

就是说，对于TCP服务端来说，返回的**新的socket对象**，和最开始用于listen的socket对象是不一样的。也就是说，最开始绑定了端口和主机的socket对象是专门用于接收连接请求的。

我们实际使用一下这个函数：

```python
# 客户端运行
import socket

s = socket.socket()
s.connect(("127.0.0.1", 43211))
```

```python
# 服务端
s.accept()
# (<socket.socket fd=568, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 43211), raddr=('127.0.0.1', 2588)>, ('127.0.0.1', 2588))
```

再来一遍

```python
# 客户端运行
import socket

s = socket.socket()
s.connect(("127.0.0.1", 43211))
```

```python
# 服务端
s.accept()
# (<socket.socket fd=468, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 43211), raddr=('127.0.0.1', 2823)>, ('127.0.0.1', 2823))
```

可以看到，每次返回的socket的包含了协议，服务端地址+端口号和客户端地址+端口号。这个五元组唯一标识了这个连接，也就是说限制单机TCP最大连接数的因素，从理论上来说是ip地址范围*端口范围，连接数最多是2^32\*2^16=2^48。但实际情况中这个和设备的内存，一条tcp连接占用的内存有关，端口号的范围65535并不是单机服务器处理的连接数上限。

















