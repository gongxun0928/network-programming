# 网络基础篇

任何网络相关的书籍，普及网络知识总是从TCP/IP协议分层开始的，本书也不例外，网络基础的普及也是会采用TCP/IP分层的方式，但是本书侧重于实际应用中，网络的基础知识和工具的知识普及，不会涉及太多理论知识。

本章会从TCP/IP协议分层方式，去介绍各层中，实际运用较多的网络协议和知识。

第一节主要介绍应用层协议的基本知识和协议的应用场景。主要讲解应用最广泛的HTTP协议，websocket协议，会简单介绍DNS协议的相关知识

第二节主要介绍传输层协议，主要讲解TCP协议的相关知识、UDP协议的知识和运用场景，简单介绍现代编程中的可靠UDP协议的原理。

第三节介绍IP层的相关知识，会讲解网络路由的原理，介绍日常使用的traceroute命令背后的原理

第四节介绍数据链路层的相关知识，包含Mac地址，ARP请求，MTU等等

第五节介绍一些额外的网络知识，一般开发接触不到，但是日常生活中经常遇到的网络知识，如连接wifi时，手机是如何获取一个可用IP的；家用网络是如何通过路由器组成的局域网给多台手机的相同应用提供网络服务的；我们说的科学上网，背后的sock5，shadowsocks代理的原理是什么，如何自己实现一个sock5代理。

