### 到底是谁给我发的数据包

这个问题的来源是，最近公司机房迁移，收到了某个业务方发来的反馈，说我们的某台机器\(xxx.web02\)还在请求即将下线的服务A\(100.69.238.11:8000 \)，收到同事反馈时，本来以为是一个很容易的事情，到业务方给的对应IP的机器上，执行一下netstat 或者lsof命令就可以得到具体的进程号了，结果发现事情没有想象的那么简单。

首先，我登录了xxx.web02的机器，执行了`netstat -anp|grep '8000'`，结果并没有发现对应的进程号；由此猜想这个是http请求，可能是不定期请求，所以我写了一个shell脚本.内容大致如下:

```bash
while (( 1 ))
do
        port=`netstat -anp|grep 100.69.238.11:8000|awk '{print $4}'|cut -d ':' -f 2`
        if [ ! -z "$port" ]
        then
                set -x
                lsof -i "TCP:$port"
                set +x
        fi
        sleep 1
done
```

开始我期望通过定时1s调用 `netstat -anp|grep 100.69.238.11:8000` 命令获取到占用8000端口的进程号，结果并不如意，实际由于http请求，连接很快断开了，当我使用netstat 命令去查看端口占用时，此时连接已经处于TIME\_WAIT状态，进程号查询不出来，此时我就想，是否可以通过lsof -i去查看本地端口占用情况；结果也是一样。

![](/assets/watch_shell.png)

如上图所示，lsof -i 去查看连接占用的本地端口，没有任何输出，获取不到本地进程号。

此证明，短连接情况下，查看端口的进程占用情况，netstat 和 lsof 作用不大；这时，我想是否能够通过tcpdump抓包去分析这个http包里的数据，去猜测到底是哪个进程占用端口，于是使用如下命令抓包：

```
tcpdump -i eth0 dst host 100.69.238.11 and port 8000 -s 0 -w dquality.pcap
```

tcpdump的抓包情况大致类似下图:![](/assets/tcpdump_dquality.png)备注：tcpdump并不适合分析数据包，此时可以祭出图形化分析工具WirkShark. 使用方式参见[常用工具篇](/chang-yong-gong-ju-pian.md)

通过wireshark的分析，发现http头部的User-Agent里包含:Apach-HttpClient之类的字段，由此可见，是一个java进程发送的。

jps查看java进程，发现有十几个java进程；这里要简单解释一下这个服务器的历史，由于这台服务器是其他项目组交给我的，但是上面跑了特别多历史项目，且没人知道到底有没有在用，所以这台服务器的环境很复杂，十几个java进程，只有少数知道是做什么的，其他都不清楚，此时有一个办法，就是ps -ef 去查看每个java进程的目录，根据目录去看到底什么项目，然后再去gitlab上一个个项目去索引，谁调用了服务A。

聪明的程序员，当然不能使用上面的办法，且为了今天是http请求，可以通过user-agent请求去投机取巧，然后遇到DNS或者ICMP这类请求，我们又该如何去定位呢，至此一个好用的工具[ptrap](https://github.com/mvitale1989/ptrap) 出现了。

先秀一下用ptrap分析的结果,然后我们再来讲解一下ptrap的原理![](/assets/ptrap_shell.png)

