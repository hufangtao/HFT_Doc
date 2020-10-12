### getaddrinfo和getnameinfo
#### getnameinfo
`getnameinfo`是以一个套接口地址为参数，返回一个描述主机的字符串和一个描述服务的字符串。其函数原型如下：
```
#include <netdb.h>
int getnameinfo(const struct sockaddr *sockaddr, socklen_t addrlen, char *host, size_t hostlen, char *serv, size_t servlen, int flags);
```
成功返回0，错误返回-1
#### getaddrinfo


### sockaddr和sockaddr_in
`struct sockaddr`和`struct sockaddr_in`这两个结构体用来处理网络通信的地址。一般的编程中并不直接对`struct sockaddr`进行操作，而使用`sockaddr_in`。
#### sockaddr
`sockaddr`在头文件`#include <sys/socket.h>`中定义，`sockaddr`的缺陷是：`sa_data`把目标地址和端口信息混在一起了，如下：
```
struct sockaddr {  
    sa_family_t sin_family; //地址族
    char sa_data[14];       //14字节，包含套接字中的目标地址和端口信息               
};
```
#### sockaddr_in
`sockaddr_in`在头文件`#include<netinet/in.h>`或`#include <arpa/inet.h>`中定义，该结构体解决了`sockaddr`的缺陷，把`port`和`addr`分开储存在两个变量中，如下： 
```
struct sockaddr_in
{
    sa_family_t sin_family;     // 地址族
    uint16_t sin_port;			// 端口
    struct in_addr sin_addr;	        // IP地址
    char     sin_zero[8];       // 对齐sockaddr
};

struct in_addr
{
    uint32_t s_addr;
};
```
sin_port和sin_addr都必须是网络字节序（NBO），一般可视化的数字都是主机字节序（HBO）。
