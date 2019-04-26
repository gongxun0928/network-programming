### **如何实现一个简单的http web框架库**

为什么我们需要http web框架，框架做的事情就是把web开发中经常用到的功能，以plugin功能化的形式提供给web开发者，本质是要求简单易用，功能丰富。

最最基本的http webserver实现的伪代码：

```c
void buildRequest(clientSocket){
    //读取第一行
    lineText=clientSocket.readline()
    parseHTTPLine(lineText)//解析请求行

    while(text=readLine() && text!='\r\n'){
        parseHTTPHeader(Text);//解析http 头部
    }

    //看while条件就知道，http请求头和请求体的间隔，是通过插入一行'\r\n'空行实现的
    httpBody=clientSocket.readall()

    //如何判断http请求结束了，要不就是 contentLength等于读取的长度，要么就是Transfer-Encoding=chunk
    if(contentLength==len(httpboday) or chunkLength==0)){
        http.parseOver()
    }
    return request
}

//循环处理clientSocket
void handle(clientSocket){
    while(true){    
        request=buildRequest(clientSocket)

        //根据request的字段，处理逻辑，回复response
        doRequest(request,response)

        clientSocket.send(response)
    }
}


void main(){
    socket= createSocket();
    while(true){
        clientSocket=accept(socket);
        pthread_create(new_thread,null,handle,clienntSocket)
    }
}
```

在第一节中，我们用不到十行的go代码实现了一个http server，任何请求都返回hello,world。go语言的net/http实现了http webserver的功能，流程上参考上面最简单的webserver代码，go实现了底层tcp协议的所有细节和http协议的解析，request和response的parse和build，并且最终调用了一个通用接口http.Handler来处理用户逻辑。

```
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

所以我们接下来关于web框架的所有代码，都是基于http.Handler的接口展开的。要在用户开始写代码前，能够执行一些我们希望执行的代码，比如在http 头部插入某些字段，又或者再处理用户数据前，统一鉴权；那么这些功能我们应该怎么去实现呢？



### 重载http.Handler

基于Go语言的Duck Type实现相同接口的类型，替换掉http.ListenAndServe\(addr,nil\)时，使用的defaultServerMux对象，使用我们自己的Handler类型。首先我们定义一个数据结构，并且实现http.Handler接口

```
type Mux struct{
}

func(mux *Mux) ServeHTTP(ResponseWriter, *Request){
    //do our logic
}
```



