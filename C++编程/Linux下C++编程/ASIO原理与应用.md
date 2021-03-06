## boost相关

### Reactor模式

#### event-driven architecture
事件驱动体系结构是目前比较广泛使用的一种。这种方式会定义一系列的事件处理器来响应事件的发生，并且将服务端接受连接与对事件的处理分离。其中，事件是一种状态的改变。比如，tcp中socket的`new incoming connection、ready for read、ready for write`。

reactor设计模式是event-driven architecture的一种实现方式，处理多个客户端并发的向服务端请求服务的场景。每种服务在服务端可能由多个方法组成。reactor会解耦并发请求的服务并分发给对应的事件处理器来处理。目前，许多流行的开源框架都用到了reactor模式，如：netty、node.js等，包括java的nio。

reactor主要由以下几个角色构成：`handle、Synchronous Event Demultiplexer、Initiation Dispatcher、Event Handler、Concrete Event Handler`

- Handle  
handle在linux中一般称为文件描述符，而在window称为句柄，两者的含义一样。handle是事件的发源地。比如一个网络socket、磁盘文件等。而发生在handle上的事件可以有connection、ready for read、ready for write等。

- Synchronous Event Demultiplexer  
同步事件分离器，本质上是系统调用。比如linux中的select、poll、epoll等。比如，select方法会一直阻塞直到handle上有事件发生时才会返回。

- Event Handler  
事件处理器，其会定义一些回调方法或者称为钩子函数，当handle上有事件发生时，回调方法便会执行，一种事件处理机制。

- Concrete Event Handler  
具体的事件处理器，实现了Event Handler。在回调方法中会实现具体的业务逻辑。

- Initiation Dispatcher  
初始分发器，也是reactor角色，提供了注册、删除与转发event handler的方法。当Synchronous Event Demultiplexer检测到handle上有事件发生时，便会通知initiation dispatcher调用特定的event handler的回调方法。

### ASIO原理

#### ExecutionContext
- execution_context

用于让函数执行的上下文，io_context就是一个例子
```
class execution_context : private noncopyable
{
  public:
    /// 构造
    BOOST_ASIO_DECL execution_context();
    /// 析构
    BOOST_ASIO_DECL ~execution_context();

  public:
    class id;       // service id类用于唯一区分service
    class service;  // 服务类
  
  private:
    // 服务注册器
    boost::asio::detail::service_registry* service_registry_;
}

```
通过下列函数操作execution_context的services
```
template <typename Service> Service& use_service(execution_context&);
template <typename Service> Service& use_service(io_context&);
template <typename Service> void add_service(execution_context&, Service*);
{
  // 先检查Service和execution_context::service类型是否匹配
  (void)static_cast<execution_context::service*>(static_cast<Service*>(0));

  // 调用execution_context的服务注册器，注册服务
  e.service_registry_->template add_service<Service>(svc);
}
template <typename Service> bool has_service(execution_context&);
```
- execution_context::service
```
/// Base class for all io_context services.
class execution_context::service : private noncopyable
{
  public:
    /// Get the context object that owns the service.
    execution_context& context();

  protected:
    /// 构造函数.
    /**
    * @param owner The execution_context object that owns the service.
    */
    BOOST_ASIO_DECL service(execution_context& owner);
  
  private:
    // 用于区分service的key
    struct key
      {
        key() : type_info_(0), id_(0) {}
        const std::type_info* type_info_;
        const execution_context::id* id_;
      } key_;

      execution_context& owner_;  // 所属context
      service* next_;             // 单链表引用
}
```
- service_register

服务器注册器：注册`execution_context::service`，以`service_key`为主键用单链表的方式保存`service`
```
class service_registry : private noncopyable
{
  public:
    // Constructor.
    BOOST_ASIO_DECL service_registry(execution_context& owner);
  
  private:
  template <typename Service>
  Service& use_service(io_context& owner);

  // Add a service object. Throws on error, in which case ownership of the object is retained by the caller.
  template <typename Service>
  void add_service(Service* new_service);

  // Check whether a service object of the specified type already exists.
  template <typename Service>
  bool has_service() const;

  // Mutex to protect access to internal data.
  mutable boost::asio::detail::mutex mutex_;

  // The owner of this service registry and the services it contains.
  execution_context& owner_;

  // services服务链表头
  execution_context::service* first_service_;
}
```
#### io_context

继承自`execution_context`，提供核心io功能和允许自定义异步服务

源码：
```
class io_context : public execution_context
  {
    io_context()
    : impl_(add_impl(new impl_type(*this,
          BOOST_ASIO_CONCURRENCY_HINT_DEFAULT, false)))
    {
    }
    io_context::impl_type& add_impl(io_context::impl_type* impl)
    {
      boost::asio::detail::scoped_ptr<impl_type> scoped_impl(impl);
      boost::asio::add_service<impl_type>(*this, scoped_impl.get());
      return *scoped_impl.release();
    }

    // 直接调用impl的run函数
    io_context::count_type run(boost::system::error_code& ec)
    {
      return impl_.run(ec);
    }

    private:
      // io_context的真实实现类
      io_context_impl& io_context_impl_;
  }
```
`io_context`的具体实现是`io_context_impl`，`window`下使用iocp完成端口设计，linux下使用scheduler调度器实现：
```
#if defined(BOOST_ASIO_HAS_IOCP)
  typedef class win_iocp_io_context io_context_impl;
  class win_iocp_overlapped_ptr;
#else
  typedef class scheduler io_context_impl;
#endif
```
#### io_context_impl_初始化scheduler
- scheduler调度器
```
class scheduler
  : public execution_context_service_base<scheduler>,
    public thread_context
{
    public:
      // Constructor. Specifies the number of concurrent threads that are likely to
      // run the scheduler. If set to 1 certain optimisation are performed.
      BOOST_ASIO_DECL scheduler(boost::asio::execution_context& ctx, int concurrency_hint = 0, bool own_thread = true);
      
      // io_context的真实实现
      // Run the event loop until interrupted or no more work.
      BOOST_ASIO_DECL std::size_t run(boost::system::error_code& ec);
    
    private:
      // 时间管理大师 (线程之间通信管理)
      conditionally_enabled_event wakeup_event_;

      // 任务的真正执行者
      reactor* task_;

      // handler队列
      op_queue<operation> op_queue_;

      // The thread that is running the scheduler.
      boost::asio::detail::thread* thread_;
```
- conditionally_enabled_event

封装了一层基于条件变量event
```
// Mutex adapter used to conditionally enable or disable locking.
class conditionally_enabled_event
  : private noncopyable
{
  public:
    // Signal the event. (Retained for backward compatibility.)
    void signal(conditionally_enabled_mutex::scoped_lock& lock)
    {
      if (lock.mutex_.enabled_)
        event_.signal(lock);
    }
  private:
    boost::asio::detail::event event_;
};
```
不同情况下event的定义：
```
#if !defined(BOOST_ASIO_HAS_THREADS)
typedef null_event event;
#elif defined(BOOST_ASIO_WINDOWS)
typedef win_event event;
#elif defined(BOOST_ASIO_HAS_PTHREADS)
typedef posix_event event;
#elif defined(BOOST_ASIO_HAS_STD_MUTEX_AND_CONDVAR)
typedef std_event event;
#endif
```
- posix_event

定义在scheduler类中，当scheduler是多线程时，用于多线程之间的唤醒，底层使用条件变量pthread_cond_t相关函数实现

源码如下：
```
class posix_event : private noncopyable
{
  public:
    // ConsPtructor.
    BOOST_ASIO_DECL posix_event();

    // If there's a waiter, unlock the mutex and signal it.
    template <typename Lock>
    bool maybe_unlock_and_signal_one(Lock& lock)
    {
      BOOST_ASIO_ASSERT(lock.locked());
      state_ |= 1;
      if (state_ > 1)
      {
        lock.unlock();
        ::pthread_cond_signal(&cond_); // Ignore EINVAL.
        return true;
      }
      return false;
    }

    // Wait for the event to become signalled.
    template <typename Lock>
    void wait(Lock& lock)
    {
      BOOST_ASIO_ASSERT(lock.locked());
      while ((state_ & 1) == 0)
      {
        state_ += 2;
        ::pthread_cond_wait(&cond_, &lock.mutex().mutex_); // Ignore EINVAL.
        state_ -= 2;
      }
    }
  private:
    ::pthread_cond_t cond_;
    std::size_t state_;
};
```
- scheduler_operation

用于表示所有操作的基类
```
class scheduler_operation BOOST_ASIO_INHERIT_TRACKED_HANDLER
{
  public:
    // 一个操作完成时调用的函数，实际就是调用构造函数中传入的函数指针func
    void complete(void* owner, const boost::system::error_code& ec, std::size_t bytes_transferred)
    {
      func_(owner, this, ec, bytes_transferred);
    }
  protected:
    typedef void (*func_type)(void*, scheduler_operation*, const boost::system::error_code&, std::size_t);
    scheduler_operation(func_type func)
      : next_(0),
        func_(func),
        task_result_(0)
      {
      }
  
  private:
    friend class op_queue_access;
    scheduler_operation* next_;
    func_type func_;
  protected:
    friend class scheduler;
    unsigned int task_result_; // Passed into bytes transferred.
};
```

#### reactor
- epoll_reactor  
事件分离器，linux下使用epoll实现。
```
class epoll_reactor
  : public execution_context_service_base<epoll_reactor>
{
  // descriptor_state定义
  class descriptor_state;

  // Constructor.
  BOOST_ASIO_DECL epoll_reactor(boost::asio::execution_context& ctx);
  // Destructor.
  BOOST_ASIO_DECL ~epoll_reactor();

  // 注册一个槽位 新建一个descriptor_data并注册到epoll
  BOOST_ASIO_DECL int register_descriptor(socket_type descriptor,
      per_descriptor_data& descriptor_data);
  
  // Post a reactor operation for immediate completion.
  void post_immediate_completion(reactor_op* op, bool is_continuation)
  {
    scheduler_.post_immediate_completion(op, is_continuation);
  }

  // 执行descriptor_data
  BOOST_ASIO_DECL void start_op(int op_type, socket_type descriptor,
      per_descriptor_data& descriptor_data, reactor_op* op,
      bool is_continuation, bool allow_speculative);
  
  // Run epoll，对应io_context的run调用
  BOOST_ASIO_DECL void run(long usec, op_queue<operation>& ops);

  // epoll fd
  int epoll_fd_;

  // 时间，主要用于定时run任务
  int timer_fd_;
};
```
1. 各实现类：socket等在调用async_wait函数时，调用start_op开始执行epoll操作
2. 在epoll操作结束之后，调用post_immediate_completion告知scheduler调度器，任务已经完成
3. 调度器中，。
- descriptor
负责描述
```
// Per-descriptor queues.
class descriptor_state : operation
{
  friend class epoll_reactor;
  friend class object_pool_access;

  descriptor_state* next_;
  descriptor_state* prev_;

  mutex mutex_;
  epoll_reactor* reactor_;
  int descriptor_;
  uint32_t registered_events_;
  op_queue<reactor_op> op_queue_[max_ops];
  bool try_speculative_[max_ops];
  bool shutdown_;

  BOOST_ASIO_DECL descriptor_state(bool locking);
  void set_ready_events(uint32_t events) { task_result_ = events; }
  void add_ready_events(uint32_t events) { task_result_ |= events; }
  BOOST_ASIO_DECL operation* perform_io(uint32_t events);
  BOOST_ASIO_DECL static void do_complete(
      void* owner, operation* base,
      const boost::system::error_code& ec, std::size_t bytes_transferred);
};
```

-----------------

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
