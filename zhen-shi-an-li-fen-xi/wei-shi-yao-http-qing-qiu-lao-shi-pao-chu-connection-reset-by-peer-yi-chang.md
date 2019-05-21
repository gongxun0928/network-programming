### http请求为啥老是抛出网络异常

**问题描述：**

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

其实不然，对照TCP三次握手的流程图，里面有两个很重要的队列，对于网络开发经验少的人来说，经常性会忽略，这两个队列影响着TCP连接的成功，现在我完整描述一下TCP连接建立的过程（穿插两个连接队列的作用）

* client端发起SYN请求，请求建立连接,client进入SYN\_SEND状态，server端收到SYN请求，

* 连接进入SYN\_RECV状态，此时会创建一个连接信息存储在**syns queue**队列里，然后回复syn+ack到client

* client收到server 回复的syn+ack后，回复ack给server端，client进去established状态，**此时客户端可能会进行数据发送** ,server端收到client的ack后，server会删除syns queue队列中的连接信息，然后进入到一个叫做**accept queue**队列

流程中，如果syns queue和accept queue满，都可能导致连接异常，具体情况取决于系统参数。

case 1：如果半连接队列满了，在阶段2就会出问题，导致正常请求的SYN包就没有办法正常处理，不是我们遇到的情况

case2： 如果连接队列满了，服务端会按照网络参数net.ipv4.tcp\_abort\_on\_overflow的值进行处理，如果tcp\_abort\_on\_overflow=0，则丢弃掉client的ack包，不作处理，服务端过一段时间后会重新发送syn+ack的包，按照重传机制，执行net.ipv4.tcp\_synack\_retries次重传，每次重传间隔时间double，最终失败后，执行reset；当然这里如果client超时时间较短的话，就会抛出连接超时connection timeout之类的异常。

OK，通过上面的分析，对照着我们的抓包，发现情况，就是case2的情况，去服务器上执行sysctl -a查看系统的tcp参数。

```
net.ipv4.tcp_abort_on_overflow = 0
net.ipv4.tcp_max_syn_backlog = 2048
net.ipv4.tcp_syn_retries = 3
net.ipv4.tcp_synack_retries = 5
net.ipv4.tcp_syncookies = 1
```

发现果然tcp\_abort\_on\_overflow=0，tcp\_synack\_retries 重传次数是5次，经过63秒后，连接被reset。

> **notice:**
>
> 1. **半连接队列的大小，取决于系统net.ipv4.tcp\_max\_syn\_backlog变量**
> 2. **全连接队列的大小，取决于定义socket时，Listen传递的backlog的大小，和内核参数net.core.somaxconn的较小值，即min\(backlog,somaxconn\)的值**
>
> **我们写代码的时候，应该有很多人从来没有想过这个backlog，很容易直接忽视了它**

上面是我最喜欢的抓包分析法，对于tcpdump和wireshark不熟悉的同事，也有其他简单的方法来定位这个问题，但是没有抓包分析法这么准确。

**快速定位法：**

> **使用更常用的工具定位，netstat和ss命令定位问题。**

```
[root@localhost]$ netstat -s |egrep "listen|LISTEN"
    19800 times the listen queue of a socket overflowed
    19800 SYNs to LISTEN sockets dropped
```

19800 表示全连接队列溢出的次数，隔几秒钟执行下，如果这个数字一直在增加的话, 肯定全连接队列偶尔满了。

ss命令

```
[root@localhost]$ ss -lnt
Recv-Q Send-Q Local Address:Port Peer Address:Port
   101    100              :8080            *:*
```

上面看到的第二列Send-Q 值是100，表示第三列的listen端口上的全连接队列最大为100，第一列Recv-Q为全连接队列当前使用了多少,联系上面全连接队列大小公式，我们查到java进程 tomcat6的默认backlog是100。远远小于somaxconn=65535的值。

Recv-Q全连接队列使用了101个，肯定是满了，所以我们通过这个就可以看到，8080端口的连接，全连接队列爆了。

