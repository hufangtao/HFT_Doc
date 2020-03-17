## 游戏服务器gateway设计
```
游戏服务器中gateway负责监听端口，接受连接请求，生成与客户端游戏通信的类
```
#### 几个概念
概念|描述|例|备注
--|:--:|--:|--
链接类型|客户端连接服务器的方式|ssl, tcp|为了通用性设计，也可以只支持一种连接方式
acceptor|监听端口，接受链接|tcpacceptor、 sslacceptor|一个链接类型对应一个acceptor，不断异步accept，异步成功返回后启动client
client|与客户端交换信息||负责与客户端通信


#### gate


#### 参考
[为什么有监听socket和连接socket，为什么产生两个socket？](https://www.cnblogs.com/liangjf/p/9900928.html "博客园")
