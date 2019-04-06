### HTTP协议及其应用

HTTP协议应该是互联网领域运用最广泛的一种协议，它的重要性无需多言，作为web开发人员，这是我们最应该掌握的技能，首先我们看看HTTP请求和响应长什么样子。

首先我们用Go首先一个web server,然后编译运行。

```go
package main

import (
    "log"
    "net/http"
)

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("hello wolrd"))
    })
    log.Fatal(http.ListenAndServe(":19028", nil))
}
```

然后使用curl -v 请求webserver,查看打印值

```
1. ➜  ~ curl -v http://localhost:19028
2. * Rebuilt URL to: http://localhost:19028/
3. *   Trying ::1...
4. * TCP_NODELAY set
5. * Connected to localhost (::1) port 19028 (#0)
6. > GET / HTTP/1.1
7. > Host: localhost:19028
8. > User-Agent: curl/7.54.0
9. > Accept: */*
10. >
11. < HTTP/1.1 200 OK
12. < Date: Sun, 31 Mar 2019 13:41:50 GMT
13. < Content-Length: 11
14. < Content-Type: text/plain; charset=utf-8
15. <
16. * Connection #0 to host localhost left intact
17. hello wolrd%
```

现在我们来逐行解释上面的打印是什么意思

第1-5行http协议没什么关系，暂时不做解释。

第6行代表 HTTP的请求行，由三个元素组成，HTTP的Method（GET），请求的Url （/）和协议版本（HTTP/1.1）

第7-9行代表HTTP的消息头，每一行代表一个头部选项

第11行代表HTTP的状态行，也有3个元素组成，HTTP协议，与请求行一致，状态码和状态码的文本描述

第12-14行代表HTTP响应头部

第17行代表响应正文



HTTP协议的基本结构就是上面那样，主要使用的HTTP方法主要就是GET,PUT,POST,DELETE,OPTIONS等方法

http请求可以通过Query携带具体的查询参数，不同的查询参数使用&链接，特殊字符在url中一般会通过转义\(%十六进制\)

特殊说明一下，OPTIONS方法主要用于获取服务端支持的Method，常用的还有获取后端的跨域支持\(Access-Control-Allow-Origin: \*\)

其他的普通方法，暂时不做说明。

