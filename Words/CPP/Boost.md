## boost相关



#### boost与gcc
boost版本和gcc版本有联系
版本releaseNote中有
Compilers Tested这一项
可以查看已经编译通过的gcc版本

#### 查看boost版本
dpkg -S /usr/include/boost/version.hpp

### ASIO

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


## Thread

