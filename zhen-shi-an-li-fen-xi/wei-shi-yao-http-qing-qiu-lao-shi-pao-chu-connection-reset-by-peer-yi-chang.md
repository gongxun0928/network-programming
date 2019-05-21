### http请求为啥老是抛出网络异常

问题描述：

> **最近业务监控群里，老是抛出一些java.net.SocketException：Connection reset的异常。        **
>
> 1. **部分请求异常**
> 2. **部分http请求正常返回**

**故障结论：**

> **第一反应，就是全连接或者半连接队列设置不合理，并发太高导致的连接被丢弃，实际去查看机器情况时，的确发现端口对应的tcp连接的全连接队列一直是满的，最后发现原因的确是全连接队列满了，处理不了tcp 连接请求，导致http请求被reset掉，根本原因不是并发高，也不是连接队列太小了，而是程序fullgc导致tomcat卡死了，连接处理不了，所以表现出来的就是连接队列满了。。这又是一个伤心的故事。😓    😓    😓  **



**原因分析:**     

那么为什么http请求会抛出这个网络异常，这个得先从传说中的TCP 三次握手开始，先盗个图，这张图画的很好。

![](/assets/tcp_sync_ack.png)

   这个图详细标出了tcp连接过程中，连接的状态变化。一个正常的连接请求，应该是类似这样的:![](/assets/wireshark_tcp_connect_state.png)

* step1: client发送syn包到server进行握手
* step2: server收到syn后，返回syn+ack给client
* step3: client收到syn+ack之后，回复给server一个ack表示收到了syn+ack，至此client也变成established状态，连接建立成功



现在我们来看看当时粗线故障时，服务器上的抓包情况：![](/assets/tcp_reset_server.png)

对照TCP三次握手的情况分析，从抓包看step1和step2完成，看起来建立连接的过程中step3中client也发出去了ack包了，至此连接应该建立了，为什么会有这么多TCP包发送失败的情况呢？

其实不然，对照TCP三次握手的流程图，里面有两个很重要的队列，对于网络开发经验少的人来说，经常性会忽略，这两个队列影响着TCP连接的成功，现在我完整描述一下TCP连接建立的过程（穿插两个连接队列的作用）：

1. 


