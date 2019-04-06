### 常用的HTTP头部说明

上一节，通过curl -v展示了最基本的HTTP请求，介绍了HTTP请求和返回的主要组成部分

HTTP头部请求包含太广泛，一一介绍并不合适，我们主要从访问控制，权限，缓存，压缩，格式

访问控制:

```
    我们常说的跨域资源请求\(CORS\)就是属于访问控制的范畴，一般资源是否允许跨域访问，都是通过Options访问，查看资源返回的头部里Access-Control-Allow-Origin字段包含的domain值来确定域名，Access-Control-Allow-Methods用来确认支持的方法等等。
```

权限:

```
    HTTP协议的权限检验，一般是通过包含Authorization头部字段来进行校验，生产环境一般是通过自定义的权限校验
```

缓存:

```
    HTTP用来控制缓存的字段Cache-Control,用来指导浏览器如何处理资源的缓存，常用的不缓存组合:no-cache,no-store; 静态资源的缓存组合一般是:public,max-age=31536000或者private,max-age=31536000,public和private的区别在于是否可以被公共缓存给缓存起来

    请求时是否使用缓存，可以通过ETag和If-Motified-Since来处理缓存访问问题
```

压缩：

```
    请求使用Accept-Encoding协商客户端支持的压缩算法，Transfer-Encoding表明网络中内容传输使用的什么压缩算法，使用Content-Encoding返回实际压缩算法
```

格式：

```
    Accept表明可接受的MIME类型
```

分片支持:

```
    Range头支持分段请求，常用于静态文件的断点续传
```



