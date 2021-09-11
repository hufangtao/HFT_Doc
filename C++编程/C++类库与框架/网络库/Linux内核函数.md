## Linux内核网络函数和一些参数

### sockaddr和sockaddr_in

这两个数据结构体是用来处理网络通信的地址，两者长度一样，都是16个字节，占用的内存大小一致，可以通过`std::static_cast`或`boost::implicit_cast`转化。

`sockaddr`常用于`bind`、`connect`、`recvfrom`、`sendto`等函数的参数，用于指明地址信息，是一种通用的套接字地址。

`sockaddr_in`是`internet`环境下套接字的地址形式，明确包含IP地址和端口等信息字段。

一般我们会对`sockaddr_in`进行操作，填充地址信息，最后使用类型转换为`sockaddr`的统一结构，用于`socket`操作。

1. sockaddr

   该结构的缺陷是`sa_data`把目标地址和端口信息混在一起了，结构定义：

   ```cpp
   struct sockaddr {
        sa_family_t sin_family; //地址族
   　　 char sa_data[14];       //14字节，包含套接字中的目标地址和端口信息  
   };
   ```

2. sockaddr_in

   该结构体解决了sockaddr的缺陷，把port和addr分开存储在两个变量中，结构定义：

   ```cpp
   struct sockaddr_in{
       sa_family_t		sin_family;	// 地址族
       uint16_t		sin_port;   // 16位端口号
       struct in_addr   sin_addr;   // 32位ip地址
       char			sin_zero;	// 不使用
   }
   struct in_addr{
       In_addr_t	s_addr;	// 32位IpV4地址
   }
   ```

### readv和writev

为什么需要readv和writev？

考虑，如果需要将数据读取到两段不连续的内存。一般得两次调用read，或者先存在一个大内存，再分段。这两种方式都会带来系统调用和额外开销。

所以UNIX提供了`readv`和`writev`两个函数，避免两次系统调用和额外拷贝开销。

使用方式：

```cpp
ssize_t readv(int fd, const struct iovec* iov, int iovcnt);
ssize_t writev(int fd, const struct iovec* iov, int iovcnt);
struct iovec{
    void* iov_base;	// 开始地址
    size_t iov_len;	// 长度
};
```

依次填充`iovec`所指向的内存。每块内存用一个`iovec`的元素来表示。

