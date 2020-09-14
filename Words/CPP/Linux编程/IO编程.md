## I/O编程

### 文件I/O

文件IO经常被称为`不带缓存的IO（unbuffered I/O)`。不带缓存指的是每个read，write都调用内核中的一个系统调用。也就是一般所说的低级I/O——操作系统提供的基本IO服务，与os绑定，特定于linix或unix平台。

#### 常用IO函数
常用的系统调用IO函数有5个：open、close、read、write、ioctl，此外还有个非系统调用IO函数lseek，系统调用IO都是不带缓冲的IO。
- open
    
    open用于创建一个新文件或打开一个已有文件，返回一个非负的文件描述符fd。0、1、2为系统预定义的文件描述符，分别代表STDIN_FILENO、STDOUT_FILENO、STDERR_FILENO。
    ```
    //成功返回文件描述符，失败返回-1
    int open(const char *pathname, int flags, ... /* mode_t mode */);
    ```
- close

    close用于关闭一个已打开文件。
    ```
    //成功返回0，失败返回-1
    int close(int fd);
    ```

- read

    read用于从打开文件中读数据。
    ```
    //成功返回读到的字节数；若读到文件尾则返回0；失败返回-1
    ssize_t read(int fd, void *buf, size_t count);
    ```

- write

    write用于向文件写入数据。
    ```
    //成功返回写入的字节数，失败返回-1
    ssize_t write(int fd, const void *buf, size_t count);
    ```
    write的返回值通常与参数count相同，否则表示出错。
    对于普通文件，write操作从文件的当前偏移量处开始
    若指定了O_APPEND选项，则每次写之前先将文件偏移量设置到文件尾
    成功写入之后，文件偏移量增加实际写的字节数。

- lseek

    lseek用于设置打开文件的偏移量。
    ```
    //成功返回新的文件偏移量，失败返回-1
    off_t lseek(int fd, off_t offset, int whence);
    ```
    对offset的解释取决于whence的值：

        若whence == SEEK_SET，则将文件偏移量设为距文件开头offset个字节，此时offset必须为非负
        若whence == SEEK_CUR，则将文件偏移量设为当前值 + offset，此时offset可正可负
        若whence == SEEK_END，则将文件偏移量设为文件长度 + offset，此时offset可正可负

    注意：

        lseek仅将新的文件偏移量记录在内核中，它并不引起任何IO操作，因此它不是系统调用IO，但该偏移量会用于下一次read/write操作
        管道、FIFO和套接字不支持设置文件偏移量，不能对其调用lseek

- ioctl

    ioctl提供了一个用于控制设备及其描述符行为和配置底层服务的接口。
    ```
    //出错返回-1，成功返回其他值
    int ioctl(int fd, int cmd, ...);
    ```
    ioctl对描述符fd引用的对象执行由cmd参数指定的操作
    每个设备驱动程序都可以定义它自己专用的一组ioctl命令

### 标准I/O

标准I/O是ANSI C建立的一个标bai准I/O模型，是一个标准函数包和stdio.h头文件中的du定义，具有一定的可移植性。标准IO库处理很多细节。例如缓存分配，以优化长度执行IO等。

标准的IO提供了三种类型的缓存：

    1. 全缓存：当填满标准IO缓存后才进行实际的IO操作。
    2. 行缓存：当输入或输出中遇到新行符时，标准IO库执行IO操作。
    3. 不带缓存：stderr就是了。

#### 常用标准IO函数

- 打开流
    ```
    //成功返回文件指针，失败返回NULL
    FILE *fopen(const char *pathname, const char *type);
    ```
    type指定了对该IO流的读写方式。
- 二进制IO

    二进制IO就是fread和fwrite。
    ```
    //返回读或写的对象数
    size_t fread(void *ptr, size_t size, size_t nobj, FILE *fp);
    size_t fwrite(const void *ptr, size_t size, size_t nobj, FILE *fp);
    ```
- 格式化IO

    格式化IO包括输入函数族和输出函数族，包括常见的“sprintf、fprintf”等都属于格式化IO

### 高级IO

#### 同步与异步
```
同步是指一个任务的完成需要依赖另外一个任务时，只有等待被依赖的任务完成后，依赖的任务才能算完成。
```
```
异步是指不需要等待被依赖的任务完成，只是通知被依赖的任务要完成什么工作。然后继续执行下面代码逻辑，只要自己完成了整个任务就算完成了（异步一般使用状态、通知和回调）
```
```
区别：读写数据的方式不同。
同步：主动拷贝IO数据到用户进程
异步：只要拷贝IO数据完成的通知，拷贝过程由内核完成
```

#### 阻塞与非阻塞
```
阻塞是指调用结果返回之前，当前线程会被挂起，一直处于等待消息通知，不能够执行其他业务
```
```
非阻塞是指在不能立刻得到结果之前，该函数不会阻塞当前线程，而会立刻返回
```
```
区别：需要访问的IO数据未就绪的情况下，是否需要等待
```

#### IO模型
```
一个输入操作通常包括两个不同阶段：
1. 等待数据准备好；例如：等待数据报分组到达
2. 从内核向进程复制数据
```

由于存在这两个阶段，Linux产生了下面五种IO模型（以socket为例）
1. 阻塞式IO

   - 当用户进程调用了recvfrom等方法时，内核进入IO的第1个阶段：准备数据（内核需要等待足够的数据再拷贝）这个过程需要等待，用户进程阻塞等待内核将数据准备好。然后用户进程同步拷贝到用户地址空间。

2. 非阻塞IO
   - 用户进程发起`read`请求，如果内核种的数据还没有准备好，那么它不会阻塞等待，而是立即返回一个`error`：EWOULDBLOCK、EAGAIN
   - 用户进程判断是一个`error`时，进程知道数据还没准备好，于是它可以再次发送`read`请求
   - 在某次`read`请求时，发现数据已经准备好，那么同步将数据拷贝到用户内存
   - 非阻塞IO模式下，用户进程需要轮询查看Linux内核是否准备好了
      
      ![blockchain](https://github.com/hufangtao/HFT_Doc/blob/master/Pictures/1127869-20181210212858009-948984805.png?raw=true "图片hover描述")
3. IO多路复用
   - 通过一种机制，一个进程同时监视多个文件描述符。一旦某个文件描述符就绪，能够通知程序进行响应的读写操作，而不需要程序轮询。
   - 常用的IO多路复用：`select`、`poll`和`epoll`
      
      ![blockchain](https://github.com/hufangtao/HFT_Doc/blob/master/Pictures/1127869-20181210212908314-1267377747.png?raw=true "图片hover描述")

4. 信号驱动IO
   - 内核文件描述符就绪之后，通过信号通知用户进程，用户进程通过系统调用读取数据
   - 此方法属于同步IO，因为从内核到用户空间的读取是由用户进程负责的，一般用信号量实现。
      
      ![blockchain](https://github.com/hufangtao/HFT_Doc/blob/master/Pictures/1127869-20181210212934040-13536334.png?raw=true "图片hover描述")
5. 异步IO（POSIX的aio_系列函数）
   - 用户进程发起read操作之后，立即就开始去做其他事情。内核收到一个异步`IO read`之后，不会阻塞用户进程
   - 内核等待数据准备完成，然后将数据拷贝到用户内存。当一切都完成之后，内核向用户进程发送一个`signal`，通知用户进程`read`操作完成
      
      ![blockchain](https://github.com/hufangtao/HFT_Doc/blob/master/Pictures/1127869-20181210212944334-1184572641.png?raw=true "图片hover描述")
