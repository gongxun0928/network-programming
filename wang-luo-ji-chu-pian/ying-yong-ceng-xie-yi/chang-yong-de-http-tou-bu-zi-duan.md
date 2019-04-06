### 常用的HTTP头部说明

上一节，通过curl -v展示了最基本的HTTP请求，介绍了HTTP请求和返回的主要组成部分

HTTP头部请求包含太广泛，一一介绍并不合适，我们主要从访问控制，权限，缓存，压缩，格式

访问控制:

```
    我们常说的跨域资源请求\(CORS\)就是属于访问控制的范畴，一般资源是否允许跨域访问，都是通过Options访问，查看资源返回的头部里
    Access-Control-Allow-Origin字段包含的domain值来确定域名，Access-Control-Allow-Methods用来确认支持的方法等等。
```

权限:

```
    HTTP协议的权限检验，一般是通过包含Authorization头部字段来进行校验，生产环境一般是通过自定义的权限校验
```

缓存:

```
    HTTP用来控制缓存的字段Cache-Control,用来指导浏览器如何处理资源的缓存，常用的不缓存组合:no-cache,no-store;

    静态资源的缓存组合一般是:public,max-age=31536000或者private,max-age=31536000,public和private的区别在于是否可以被公共缓存给缓存起来

    请求时是否使用缓存，可以通过ETag和If-Motified-Since来处理缓存访问问题
```

压缩：

```
    请求使用Accept-Encoding协商客户端支持的压缩算法，Transfer-Encoding表明网络中内容传输使用的什么压缩算法，使用
Content-Encoding返回实际压缩算法
```

格式：

```
    Accept表明可接受的MIME类型，Content-Type返回内容的MIME类型
```

分片支持:

```
    Range头支持分段请求，常用于静态文件的断点续传
```

其他字段

```
    HOST字段用于指明服务器的域名

    Location字段，用于3xx的跳转

    Cookie字段，以kv形式存储信息，cookie的值，是通过set-cookie头部设置的

    X-Forwarded-For在客户端访问服务器的过程中如果需要经过HTTP代理或者负载均衡服务器，可以被用来获取最初发起请求的客户端的IP地址
```

常用HTTP状态码

100 Continue 与Expect:100-continue有关

101 Switching Protocol 与Upgrade:websocket有关

200 OK 正常请求

204 no content 一般表明不需要改变页面内容

206   Partial Content 一般返回Range请求，用于返回部分内容

301 永久性重定向

302 临时性重定向 对SEO不友好

304 Not Modified

307

400 Bad Request 一般参数错误

401 Unauthorized 需要权限校验

403 Forbidden 一般指授权问题

404 Not Found 请求不存在

405 Method Not Allow 方法不允许

500 Internal Error 内部错误

502 Bad Gateway 网关获取到后端服务错误，一般服务器启动，端口挂了

503 Service Unavailable 一般指服务器挂了

504 Gateway Timeout 网关超时

