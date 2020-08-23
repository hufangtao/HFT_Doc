## boost相关

### ASIO应用
#### 定时器
- 声明一个io_context对象，用于和IO函数交流
  ```
  boost::asio::io_context io;
  ```
- 创建一个定时器
    ```
    boost::asio::steady_timer t(io, boost::asio::chrono::seconds(5));
    ```
- 计时

    同步计时：
    ```
    t.wait();
    ```
    异步计时：
    fun函数的回调，会发生在调用io.run的线程中。
    在调用io.run之前，请务必保证io有事情干，不然io.run()会直接返回。
    ```
    t.aync_wait(&fun);
    io.run();
    ```
- 多线程异步
  
  当多个线程同时启动`io.run`时，如果io的工作会访问共享代码，为了避免多线程竞争，asio使用strand保证io工作顺序调度。
  
  ```
  strand_(boost::asio::make_strand(io)),
  ```

#### 网络编程
- 使用io_context声明socket
  
### asio基本原理

#### 常见命名空间
Boost.Asio的所有内容都包含在boost::asio命名空间或者其子命名空间内。
- boost::asio：这是核心类和函数所在的地方。重要的类有io_service和streambuf。类似read, read_at, read_until方法，它们的异步方法，它们的写方法和异步写方法等自由函数也在这里。
- boost::asio::ip：这是网络通信部分所在的地方。重要的类有*address, endpoint, tcp, udp和icmp*，重要的自由函数有connect和async_connect。要注意的是在boost::asio::ip::tcp::socket中间，socket只是boost::asio::ip::tcp类中间的一个typedef关键字。
- boost::asio::error：这个命名空间包含了调用I/O例程时返回的错误码
- boost::asio::ssl：包含了SSL处理类的命名空间
- boost::asio::local：这个命名空间包含了POSIX特性的类
- boost::asio::windows：这个命名空间包含了Windows特性的类

#### IP地址
对于IP地址的处理，Boost.Asio提供了ip::address , ip::address_v4和ip::address_v6类。

它们提供了相当多的函数。下面列出了最重要的几个：
- ip::address(v4_or_v6_address):这个函数把一个v4或者v6的地址转换成ip::address
- ip::address:from_string(str)：这个函数根据一个IPv4地址（用.隔开的）或者一个IPv6地址（十六进制表示）创建一个地址。
- ip::address::to_string() ：这个函数返回这个地址的字符串。
- ip::address_v4::broadcast([addr, mask]):这个函数创建了一个广播地址
- ip::address_v4::any()：这个函数返回一个能表示任意地址的地址。
- ip::address_v4::loopback(), ip_address_v6::loopback()：这个函数返回环路地址（为v4/v6协议）
- ip::host_name()：这个函数用string数据类型返回当前的主机名。

大多数情况你会选择用`ip::address::from_string`：
  ```
  ip::address addr = ip::address::from_string("127.0.0.1");
  ```
如果你想通过一个主机名进行连接，下面的代码片段是无用的：
  ```
  // 抛出异常
  ip::address addr = ip::address::from_string("www.yahoo.com");
  ```
#### 端点

端点是使用某个端口连接到的一个地址。不同类型的socket有它自己的endpoint类，比如ip::tcp::endpoint、ip::udp::endpoint和ip::icmp::endpoint

如果想连接到本机的80端口，你可以这样做：
```
ip::tcp::endpoint ep( ip::address::from_string("127.0.0.1"), 80);
```
有三种方式来建立一个端点：
- endpoint()：这是默认构造函数，某些时候可以用来创建UDP/ICMP socket。
- endpoint(protocol, port)：这个方法通常用来创建可以接受新连接的服务器端socket。
- endpoint(addr, port)：这个方法创建了一个连接到某个地址和端口的端点。

例子如下：
```
ip::tcp::endpoint ep1;
ip::tcp::endpoint ep2(ip::tcp::v4(), 80);
ip::tcp::endpoint ep3(ip::address::from_string("127.0.0.1), 80);
```
如果你想连接到一个主机（不是IP地址），你需要这样做：
```
// 输出 "87.248.122.122"
io_service service;
ip::tcp::resolver resolver(service);
ip::tcp::resolver::query query("www.yahoo.com", "80");
ip::tcp::resolver::iterator iter = resolver.resolve(query);
ip::tcp::endpoint ep = *iter;
std::cout << ep.address().to_string() << std::endl;
```
可以用你需要的socket类型来替换tcp。首先，为你想要查询的名字创建一个查询器，然后用resolve()函数解析它。如果成功，它至少会返回一个入口。你可以利用返回的迭代器，使用第一个入口或者遍历整个列表来拿到全部的入口。

给定一个端点，可以获得他的地址，端口和IP协议（v4或者v6）：
```
std::cout << ep.address().to_string() << ":" << ep.port()
<< "/" << ep.protocol() << std::endl;
```

#### 套接字
`Boost.Asio`有三种类型的套接字类：`ip::tcp`, `ip::udp`和`ip::icmp`。当然它也是可扩展的，你可以创建自己的`socket`类，尽管这相当复杂。如果你选择这样做，参照一下`boost/asio/ip/tcp.hpp`, `boost/asio/ip/udp.hpp`和`boost/asio/ip/icmp.hpp`。它们都是含有内部`typedef`关键字的超小类。

你可以把`ip::tcp`, `ip::udp`, `ip::icmp`类当作占位符；它们可以让你便捷地访问其他类/函数，如下所示：
- `ip::tcp::socket, ip::tcp::acceptor, ip::tcp::endpoint, ip::tcp::resolver, ip::tcp::iostream`
- `ip::udp::socket, ip::udp::endpoint, ip::udp::resolver`
- `ip::icmp::socket, ip::icmp::endpoint, ip::icmp::resolver`

`socket`类创建一个相应的`socket`。而且总是在构造的时候传入io_service实例：
```
io_service service;
ip::udp::socket sock(service)
sock.set_option(ip::udp::socket::reuse_address(true));
```
每一个socket的名字都是一个typedef关键字
```
ip::tcp::socket = basic_stream_socket
ip::udp::socket = basic_datagram_socket
ip::icmp::socket = basic_raw_socket
```
