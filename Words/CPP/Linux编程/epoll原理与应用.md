## 应用

### 函数
#### 头文件
```
#include <sys/epoll.h>
```
#### epoll创建
```
int epoll_create(int size);
```
创建一个`epoll`实例。从`Linux 2.6.8`之后，`size`参数可以被忽略，但必须大于0。  
创建成功，该函数会返回一个非负数文件描述符`epfd`。  
创建失败，该函数返回`-1`,并会设置errno标识错误  
#### epoll控制函数
```
int epoll_ctl(int epfd, int op, int fd, struct epoll_event* event);
```
该函数通过`epfd`对`epoll`实例执行控制操作`op`。所有控制操作都是针对目标文件描述符`fd`的。  
`op`参数的有效值包括：
```
增：EPOLL_CTL_ADD
改：EPOLL_CTL_MOD
删：EPOLL_CTL_DEL
```
`event`参数 与目标文件描述符`fd`关联的对象，结构定义：
```
typedef union epoll_data {
    void        *ptr;
    int          fd;
    uint32_t     u32;
    uint64_t     u64;
} epoll_data_t;

struct epoll_event {
    uint32_t     events;      /* Epoll events */
    epoll_data_t data;        /* User data variable */
};
```
`epoll_event`的events参数是一个位掩码，用下列事件类型：
```
EPOLLIN
EPOLLOUT
EPOLLRDHUP
EPOLLPRI
EPOLLERR
EPOLLHUP
EPOLLET
EPOLLONESHOT
EPOLLWAKEUP
```
#### epoll等待
```
int epoll_wait(int epfd, struct epoll_event* events, int maxevents, int timeout);
```
该函数会等待`epfd`的可用`events`。`events`指针指向的内存会包含所有可用的`event`。最多可回传`maxevents`个`events`。  
可用`events`中包含的所有`epoll_event`结构数据，和用户用`epoll_ctl`设置的数据相同。而每个`epoll_event.events`字段中包含的是可用事件类型。  
## 原理

