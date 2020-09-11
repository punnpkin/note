# NIO

- Channel and Buffers

  不再操作字节流和字符流，而是通过通道和缓存来读取

- Non-blocking IO

- Selectors

  可以监控通道的**事件**

# 1. Overview

三个核心组件:

- Channels
- Buffers
- Selectors

## Channel and Buffers

NIO里所有的IO都开始于Channel

![Java NIO: Channels and Buffers](03_NIO.assets/overview-channels-buffers.png)

Java NIO中Channel的主要实现类：

- FileChannel
- DatagramChannel
- SocketChannel
- ServerSocketChannel

覆盖了TCP+UDP网络IO和文件IO

Buffer的几个重要实现类：

- ByteBuffer
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer

包括byte, short, int, long, float, double and characters.

## Selectors

能够使一个线程管理多个通道，

![Java NIO: Selectors](03_NIO.assets/overview-selectors.png)

使用Selector需要**注册**Channel，select()方法会阻塞，直到有事件发生后返回，线程就可以开始处理这些事件。

# 2. Channel

Channel和stream的区别：

- Channel可以同时用于读写，流只能作用在一个方向
- Channel读写是**同步的**
- Channel总是在读写Buffer

实现类：

- FileChannel：文件
- DatagramChannel：UDP
- SocketChannel：TCP
- ServerSocketChannel：TCP服务端

# 3. Buffer

缓冲区本质上是**一块内存**，这块内存包裹在NIO Buffer对象中（Buffer提供了使用该内存块的方法）

### 3.1 Basic Buffer Usage

使用Buffer进行读写：

1. 将data写入buffer
2. Call `buffer.flip()`
3. 从buffer中读取数据
4. Call `buffer.clear()`or `buffer.compact()`

在buffer中写数据的时候，Buffer会追踪写入的数据。如果需要读取，先要使用`flip()`方法。

`clear()`会清除Buffer中所有的数据，`compact()`只会回收已经读取的数据，未读的数据将会移动到buffer的开头。

### 3.2 Buffer Capacity, Position and Limit

Buffer的三个属性：

- capacity
- position：读写数据时的指针位置，
- limit：写模式limit=capacity，读模式limit指向最后一个写的位置

### 3.3 Buffer Types

- ByteBuffer
- MappedByteBuffer
- CharBuffer
- DoubleBuffer
- FloatBuffer
- IntBuffer
- LongBuffer
- ShortBuffer

### 3.4 Allocating a Buffer

```java
// 字节缓冲
ByteBuffer buf = ByteBuffer.allocate(48);
// 字符缓冲
CharBuffer buf = CharBuffer.allocate(1024);
```

### 3.5 Writing Data to a Buffer

两种方式：

- 通过Channel

  ```java
  int bytesRead = inChannel.read(buf); //read into buffer.
  ```

- 通过Buffer自带的`put()`方法

  ```java
  buf.put(127);
  ```

### 3.6 flip()

读写切换，position指向0，limit指向position

### 3.7 Reading Data from a Buffer

两种方式：

- 通过Channel

  ```java
  //read from buffer into channel.
  int bytesWritten = inChannel.write(buf);
  ```

- 通过Buffer自带的`get()`方法

  ```java
  byte aByte = buf.get();   
  ```

### 3.8 rewind()

重置position到0处，以进行reread

### 3.9 clear() and compact()

If you call `clear()` the `position` is set back to 0 and the `limit` to `capacity`.

`compact()` copies all unread data to the beginning of the `Buffer`. Then it sets `position` to right after the last unread element. The `limit` property is still set to `capacity`, just like `clear()` does.

### 3.10 mark() and reset()

```java
buffer.mark();

//call buffer.get() a couple of times, e.g. during parsing.

buffer.reset();  //set position back to mark.    
```

# 4. Scatter / Gather

提供了Channel到Buffer一对多的能力：

<img src="03_NIO.assets/scatter.png" alt="scatter" style="zoom: 80%;" /><img src="03_NIO.assets/gather.png" alt="gather" style="zoom:80%;" />





# 🌰 

基本概念，就不多说了。Nio编程中客户端和服务端都会使用Selector轮询Channel的事件，不同的是客户端使用的是SocketChannel，服务端使用的是ServerSocketChannel，服务端要监听端口，而客户端进行连接。

## SocketChannel

```java
public class TimeClientHandle implements Runnable {

    private String host;
    private int port;

    private Selector selector;
    private SocketChannel socketChannel;

    private volatile boolean stop;

    public TimeClientHandle(String host, int port) {
        this.host = host == null ? "127.0.0.1" : host;
        this.port = port;
        try {
            // 打开SocketChannel
            selector = Selector.open();
            socketChannel = SocketChannel.open();
            // 配置伪非阻塞模式
            socketChannel.configureBlocking(false);
        } catch (IOException e) {
            e.printStackTrace();
            System.exit(1);
        }
    }

    /*
     * (non-Javadoc)
     *
     * @see java.lang.Runnable#run()
     */
    @Override
    public void run() {
        try {
            doConnect();
        } catch (IOException e) {
            e.printStackTrace();
            System.exit(1);
        }
        while (!stop) {
            try {
                // 轮询Key
                selector.select(1000);
                Set<SelectionKey> selectedKeys = selector.selectedKeys();
                Iterator<SelectionKey> it = selectedKeys.iterator();
                SelectionKey key = null;
                while (it.hasNext()) {
                    key = it.next();
                    it.remove();
                    try {
                        handleInput(key);
                    } catch (Exception e) {
                        if (key != null) {
                            key.cancel();
                            if (key.channel() != null)
                                key.channel().close();
                        }
                    }
                }
            } catch (Exception e) {
                e.printStackTrace();
                System.exit(1);
            }
        }

        // 多路复用器关闭后，所有注册在上面的Channel和Pipe等资源都会被自动去注册并关闭，所以不需要重复释放资源
        if (selector != null)
            try {
                selector.close();
            } catch (IOException e) {
                e.printStackTrace();
            }

    }

    private void handleInput(SelectionKey key) throws IOException {

        if (key.isValid()) {
            // 判断是否连接成功
            SocketChannel sc = (SocketChannel) key.channel();
            if (key.isConnectable()) {
                if (sc.finishConnect()) {
                    sc.register(selector, SelectionKey.OP_READ);
                    doWrite(sc);
                } else
                    System.exit(1);// 连接失败，进程退出
            }
            if (key.isReadable()) {
                ByteBuffer readBuffer = ByteBuffer.allocate(1024);
                int readBytes = sc.read(readBuffer);
                if (readBytes > 0) {
                    readBuffer.flip();
                    byte[] bytes = new byte[readBuffer.remaining()];
                    readBuffer.get(bytes);
                    String body = new String(bytes, "UTF-8");
                    System.out.println("Now is : " + body);
                    this.stop = true;
                } else if (readBytes < 0) {
                    // 对端链路关闭
                    key.cancel();
                    sc.close();
                } else
                    ; // 读到0字节，忽略
            }
        }

    }

    private void doConnect() throws IOException {
        // 如果直接连接成功，则注册到多路复用器上，发送请求消息，读应答
        if (socketChannel.connect(new InetSocketAddress(host, port))) {
            socketChannel.register(selector, SelectionKey.OP_READ);
            doWrite(socketChannel);
        } else
            socketChannel.register(selector, SelectionKey.OP_CONNECT);
    }

    private void doWrite(SocketChannel sc) throws IOException {
        byte[] req = "QUERY TIME ORDER".getBytes();
        ByteBuffer writeBuffer = ByteBuffer.allocate(req.length);
        writeBuffer.put(req);
        writeBuffer.flip();
        sc.write(writeBuffer);
        if (!writeBuffer.hasRemaining())
            System.out.println("Send order 2 server succeed.");
    }

}
```

## ServerSocketChannel

```java
public class MultiplexerTimeServer implements Runnable {

    private Selector selector;

    private ServerSocketChannel servChannel;

    private volatile boolean stop;

    /**
     * 初始化多路复用器、绑定监听端口
     *
     * @param port
     */
    public MultiplexerTimeServer(int port) {
        try {
            // 创建多路复用器Selector、ServerSocketChannel
            selector = Selector.open();
            servChannel = ServerSocketChannel.open();
            // 配置为非阻塞模式
            servChannel.configureBlocking(false);
            // 绑定端口，将backlog设置为1024
            servChannel.socket().bind(new InetSocketAddress(port), 1024);
            // 系统资源初始化成功后，把serverSocketChannel注册到Selector，监听SelectionKey.OP_ACCEPT操作位
            servChannel.register(selector, SelectionKey.OP_ACCEPT);
            System.out.println("The time server is start in port : " + port);
        } catch (IOException e) {
            e.printStackTrace();
            System.exit(1);
        }
    }

    public void stop() {
        this.stop = true;
    }

    /*
     * (non-Javadoc)
     *
     * @see java.lang.Runnable#run()
     */
    @Override
    public void run() {
        while (!stop) {
            try {
                //循环体重遍历selector，超时时间为1s。
                selector.select(1000);
                Set<SelectionKey> selectedKeys = selector.selectedKeys();
                Iterator<SelectionKey> it = selectedKeys.iterator();
                SelectionKey key = null;
                while (it.hasNext()) {
                    key = it.next();
                    it.remove();
                    try {
                        handleInput(key);
                    } catch (Exception e) {
                        if (key != null) {
                            key.cancel();
                            if (key.channel() != null)
                                key.channel().close();
                        }
                    }
                }
            } catch (Throwable t) {
                t.printStackTrace();
            }
        }

        // 多路复用器关闭后，所有注册在上面的Channel和Pipe等资源都会被自动去注册并关闭，所以不需要重复释放资源
        if (selector != null)
            try {
                selector.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
    }

    private void handleInput(SelectionKey key) throws IOException {

        if (key.isValid()) {
            // 处理新接入的请求消息
            // 根据操作位判断网络类型。
            if (key.isAcceptable()) {
                // Accept the new connection
                ServerSocketChannel ssc = (ServerSocketChannel) key.channel();
                SocketChannel sc = ssc.accept();
                sc.configureBlocking(false);
                // Add the new connection to the selector
                sc.register(selector, SelectionKey.OP_READ);
            }
            if (key.isReadable()) {
                // 读取客户端的消息            
                SocketChannel sc = (SocketChannel) key.channel();
                ByteBuffer readBuffer = ByteBuffer.allocate(1024);
                int readBytes = sc.read(readBuffer);
                if (readBytes > 0) {
                    readBuffer.flip();
                    byte[] bytes = new byte[readBuffer.remaining()];
                    readBuffer.get(bytes);
                    String body = new String(bytes, "UTF-8");
                    System.out.println("The time server receive order : "
                            + body);
                    String currentTime = "QUERY TIME ORDER"
                            .equalsIgnoreCase(body) ? new java.util.Date(
                            System.currentTimeMillis()).toString()
                            : "BAD ORDER";
                    doWrite(sc, currentTime);
                } else if (readBytes < 0) {
                    // 对端链路关闭
                    key.cancel();
                    sc.close();
                } else
                    ; // 读到0字节，忽略
            }
        }
    }

    // 发送消息异步发给客户端。
    private void doWrite(SocketChannel channel, String response)
            throws IOException {
        if (response != null && response.trim().length() > 0) {
            byte[] bytes = response.getBytes();
            ByteBuffer writeBuffer = ByteBuffer.allocate(bytes.length);
            writeBuffer.put(bytes);
            writeBuffer.flip();
            channel.write(writeBuffer);
        }
    }
}
```











