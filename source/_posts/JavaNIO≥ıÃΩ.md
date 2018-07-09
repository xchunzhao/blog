title: JavaNIO初探
date: 2015-08-30 21:11:23
comments: true
tags: 
        - javaNIO
categories: Java
---
> ## 前言
最近在看公司内部的爬虫框架，获得响应的客户机跟负责分发的master机之间的通信是通过`Netty`。为了能搞懂`Netty`源码，个人觉得很有必要来研究一下`Java NIO`的原理和具体实现。

下面从两个方面分析`Java NIO`

## 目录
1. ##### Java NIO原理及通信模型
2. ##### Java NIO服务端客户端通信的代码分析

## 具体分析
先来看两张图，分别是阻塞IO和非阻塞IO的工作方式。<!-- more -->
![bio](https://raw.githubusercontent.com/su-kaiyao/record/master/others/imgs/bIO.png)
![nio](http://dl.iteye.com/upload/attachment/0066/2123/c17e2880-a712-349f-a818-2c921303f224.jpg)

* IO面向流<->NIO面向通道与缓冲：IO从流中读取数据直至读完，读取的数据没有缓存;NIO从通道读取数据到buffer，灵活性加强，但处理容易出错

* 阻塞IO<->非阻塞IO：阻塞IO中线程请求读写操作时，处于阻塞状态，直至数据读取操作完成前，不能进行其余事件;而NIO的一个线程从一个通道请求读取数据后，处于非阻塞模式，无需等待数据读取完成，而是利用这段空闲时间，线程去进行其余通道的IO操作

 Java NIO的服务端只需启动一个专门的线程来处理所有的 IO 事件，这种通信模型是怎么实现的呢？Java NIO采用了双向通道（channel）进行数据传输，而不是单向的流（stream），在通道上可以注册我们感兴趣的事件。

 服务端和客户端各自维护一个管理通道的对象，我们称之为selector，该对象能检测一个或多个通道 (channel) 上的事件。我们以服务端为例，如果服务端的selector上注册了读事件，某时刻客户端给服务端发送了一些数据，阻塞I/O这时会调用read()方法阻塞地读取数据，而NIO的服务端会在selector中添加一个读事件。服务端的处理线程会轮询地访问selector，如果访问selector时发现有感兴趣的事件到达，则处理这些事件，如果没有感兴趣的事件到达，则处理线程会一直阻塞直到感兴趣的事件到达为止。下面是我理解的java NIO的通信模型示意图：
 ![handler](http://dl.iteye.com/upload/attachment/0066/3190/0184183e-286c-34f1-9742-4adaa28b7003.jpg)

 下面直接上代码，服务端客户端通信代码在[这里](https://github.com/xchunzhao/nettyDemo/tree/master/netty/src/main/java/JavaNio)

**服务端**
```
public class NIOServer {
        // 通道管理器
        private Selector selector;
        public void initServer(int port) throws Exception {
            // 获得一个ServerSocket通道
            ServerSocketChannel serverChannel = ServerSocketChannel.open();
            // 设置通道为 非阻塞
            serverChannel.configureBlocking(false);
            // 将该通道对于的serverSocket绑定到port端口
            serverChannel.socket().bind(new InetSocketAddress(port));
            // 获得一耳光通道管理器
            this.selector = Selector.open();
            // 将通道管理器和该通道绑定，并为该通道注册selectionKey.OP_ACCEPT事件
            // 注册该事件后，当事件到达的时候，selector.select()会返回，
            // 如果事件没有到达selector.select()会一直阻塞

            serverChannel.register(selector, SelectionKey.OP_ACCEPT);
        }

        // 采用轮训的方式监听selector上是否有需要处理的事件，如果有，进行处理
        public void listen() throws Exception {
            System.out.println("start server");
            // 轮询访问selector
            while (true) {
                // 当注册事件到达时，方法返回，否则该方法会一直阻塞
                selector.select();
                // 获得selector中选中的相的迭代器，选中的相为注册的事件
                Iterator ite = this.selector.selectedKeys().iterator();
                while (ite.hasNext()) {
                    SelectionKey key = (SelectionKey) ite.next();
                    // 删除已选的key 以防重负处理
                    ite.remove();
                    // 客户端请求连接事件
                    if (key.isAcceptable()) {
                        ServerSocketChannel server = (ServerSocketChannel) key.channel();
                        // 获得和客户端连接的通道
                        SocketChannel channel = server.accept();
                        // 设置成非阻塞
                        channel.configureBlocking(false);
                        // 在这里可以发送消息给客户端
                        channel.write(ByteBuffer.wrap(("hello clie").getBytes()));
                        // 在客户端 连接成功之后，为了可以接收到客户端的信息，需要给通道设置读的权限
                        channel.register(this.selector, SelectionKey.OP_READ);
                        // 获得了可读的事件
                    } else if (key.isReadable()) {
                        read(key);
                    }
                }
            }
        }

        // 处理 读取客户端发来的信息事件
        private void read(SelectionKey key) throws Exception {
            // 服务器可读消息，得到事件发生的socket通道
            SocketChannel channel = (SocketChannel) key.channel();
            // 穿件读取的缓冲区
            ByteBuffer buffer = ByteBuffer.allocate(10);
            channel.read(buffer);
            byte[] data = buffer.array();
            String msg = new String(data).trim();
            System.out.println(System.currentTimeMillis()+"server receive from client: " + msg);
            ByteBuffer outBuffer = ByteBuffer.wrap("res fr ser".getBytes());
            channel.write(outBuffer);
        }

        public static void main(String[] args) throws Throwable {
            NIOServer server = new NIOServer();
            server.initServer(8989);
            server.listen();
        }
    }

```

**客户端**
```
public class NIOClient {
        // 通道管理器
        private Selector selector;
        /**
         * * // 获得一个Socket通道，并对该通道做一些初始化的工作 * @param ip 连接的服务器的ip // * @param port
         * 连接的服务器的端口号 * @throws IOException
         */
        public void initClient(String ip, int port) throws IOException { // 获得一个Socket通道
            SocketChannel channel = SocketChannel.open(); // 设置通道为非阻塞
            channel.configureBlocking(false); // 获得一个通道管理器
            this.selector = Selector.open(); // 客户端连接服务器,其实方法执行并没有实现连接，需要在listen()方法中调
            // 用channel.finishConnect();才能完成连接
            channel.connect(new InetSocketAddress(ip, port));
            // 将通道管理器和该通道绑定，并为该通道注册SelectionKey.OP_CONNECT事件。
            channel.register(selector, SelectionKey.OP_CONNECT);
        }
        /**
         * * // 采用轮询的方式监听selector上是否有需要处理的事件，如果有，则进行处理 * @throws // IOException
         * @throws Exception
         */
        @SuppressWarnings("unchecked")
        public void listen() throws Exception {// 轮询访问selector
            boolean flag=true;
            //int count=0;
            while (flag) {
                // 选择一组可以进行I/O操作的事件，放在selector中,客户端的该方法不会阻塞，
                // 这里和服务端的方法不一样，查看api注释可以知道，当至少一个通道被选中时，
                // selector的wakeup方法被调用，方法返回，而对于客户端来说，通道一直是被选中的
                selector.select(); // 获得selector中选中的项的迭代器
                Iterator ite = this.selector.selectedKeys().iterator();
                while (ite.hasNext()) {
                    SelectionKey key = (SelectionKey) ite.next(); // 删除已选的key,以防重复处理
                    ite.remove(); // 连接事件发生
                    if (key.isConnectable()) {
                        SocketChannel channel = (SocketChannel) key.channel(); // 如果正在连接，则完成连接
                        if (channel.isConnectionPending()) {
                            channel.finishConnect();
                        } // 设置成非阻塞
                        channel.configureBlocking(false);
                        // 在这里可以给服务端发送信息哦
                        channel.write(ByteBuffer.wrap(("hello serv").getBytes()));
                        // 在和服务端连接成功之后，为了可以接收到服务端的信息，需要给通道设置读的权限。
                        channel.register(this.selector, SelectionKey.OP_READ); // 获得了可读的事件
                    } else if (key.isReadable()) {
                        read(key);
                        //flag=false;
                    }
                }
            }
        }

        private void read(SelectionKey key) throws Exception {
            SocketChannel channel = (SocketChannel) key.channel();
            // 穿件读取的缓冲区
            ByteBuffer buffer = ByteBuffer.allocate(10);
            channel.read(buffer);
            byte[] data = buffer.array();
            String msg = new String(data).trim();
            System.out.println(System.currentTimeMillis()+"client receive msg from server:" + msg);
        }

        /**
         * * // 启动客户端测试 * @throws IOException
         * @throws Exception
         */
        public static void main(String[] args) throws Exception {
            NIOClient client = new NIOClient();
            client.initClient("localhost", 8989);
            client.listen();
        }
    }
```

## 总结
Java NIO主要解决传统的阻塞IO中服务端线程轮询从而导致阻塞的问题(当然也有线程开销的问题)，而NIO中只要在selecter中注册服务端感兴趣的事件，只要一个线程轮询selecter中感兴趣的事件。

但是NIO中坑是比较多的，对于一些OS的细节，作为一个开发人员很难去实现好，从而诞生了从Mina到Netty这样优秀的网络框架。
