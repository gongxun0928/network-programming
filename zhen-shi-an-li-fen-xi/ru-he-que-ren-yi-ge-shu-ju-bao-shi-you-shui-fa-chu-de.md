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



