# C++多线程系统编程

## 第1章 线程安全的对象生命期管理

借助`shared_ptr`和`weak_ptr`完美解决

### 1.1 当析构函数遇到多线程

常见的对象销毁的竞态条件：

1. 在析构一个对象是，从何而得知是否有别的线程正在执行该对象的成员函数？
2. 如何保证在执行成员函数期间，对象不会在另一个线程被析构
3. 在对象成员韩式执行前，如何得知这个对象还活着？它会不会正在析构？

#### 线程安全的定义

一个线程安全的类，应当满足：

- 多线程同时访问，其表现出正确的行为
- 无论操作系统如何调度这些线程，无论这些线程的执行顺序如何交织
- 调用段代码无需额外的同步或其它协调动作

#### `MutexLock`与`MutextLockGuard`

- `MuextLock`就是`std::mutex`的功能
- `MutexLockGurad`就是`std::lock_guard`的功能，用于加锁和解锁临界区。一般是个栈上对象。

#### 一个线程安全的`Counter`示例

```c++
int Counter::getAndIncrease()
{
	MutextLockGuard lock(mutex);
	int ret = value++;
	return ret;
}
```

一般情况下，只有一个`value`值需要保护，只要用`atomic`原子操作就行

如果`mutex`是`static`也是可以的，但是这个保护将是针对所有`Counter`对象的，可以用来保护`static`的变量

### 1.2 对象的创建

线程安全的对象构造，要求：

- 不要在构造函数中注册任何回调
- 不要在构造函数中把`this`指针传出去
- 即便在构造函数的最后一行也不行

因为构造函数执行期间，对象还没有完全初始化，如果`this`被泄露了，后果难料

一般通过二段式构造——构造函数+`init`函数

### 本章总结

就一句话，用智能指针，严肃认真谨慎地使用

## 第2章 线程同步精要

线程同步的原则：

1. 尽量最低限度地共享对象，减少需要同步的场合。一个对象能不暴露给别的线程就不要暴露。非要暴露，优先考虑`immutable`对象；实在不行才暴露可修改对象，并用同步措施来充分保护
2. 其次是使用高级的并发编程构件，如`TaskQueue`、`Producer-Consumer Queue`等
3. 最后不得已必须使用底层同步原语时，使用非递归的互斥器和条件变量，慎用读写锁，不要用信号量
4. 除了使用`atomic`整数之外，不自己写`lock-free`代码，也不要用“内核级”同步原语。不凭空猜测“那种做法性能会更好”。

### 2.1互斥器（mutex）

`mutex`应该是使用最多的同步原语，它保护了临界区，任何一个时刻最多只能有一个线程在此`mutex`划出的临界区内活动。一些使用原则：

- 用`RAII`手法封装`mutex`的创建、销毁、加锁和解锁这四个操作。
- 只用非递归的`mutex`（即不可重入的`mutex`）
- 不手动调用`lock`和`unlock`函数，一切交给栈上的`Guard`对象的构造和析构函数负责。
- 在每次构造`Guard`对象的时候，思考调用栈上已经持有的锁，防止因加锁顺序不同而导致死锁。

```c++
void function()
{
    std::lock_guard<std::mutext> lock(mutext_);
}
```

### 2.2条件变量

### 2.3不要用读写锁和信号量

### 2.4封装`MutextLock`、`MutexLockGuard`、`Condition`

本节作者主要讲解如何封装，以`muduo`为例

### 2.5线程安全的`singleton`实现

一般地，我们使用双重检查锁的方式来保证单例的线程安全

```c++
if (!Singleton::init)
{
    std::lock_guard<std::mutex> lock(mutex_);
    if (!Singleton::init)
    {
        init();
    }
}
```

`Java`的双重检查锁参考：https://www.cnblogs.com/xz816111/p/8470048.html

单例的`init`一般需要初始化对象：

1. 分配内存
2. 初始化对象
3. 将对象指向刚分配的内存空间

但有的编译器为了性能，可能会将2和3进行`重排序`，顺序就变成：

1. 分配内存
2. 将对象指向刚分配的内存空间
3. 初始化对象

现在考虑排序后，两个线程发生以下调用：

| Time | Thread A            | Thread B                                          |
| ---- | ------------------- | ------------------------------------------------- |
| T1   | 检查到未初始化      |                                                   |
| T2   | 获取锁              |                                                   |
| T3   | 再次检查到未初始化  |                                                   |
| T4   | 为singleton分配内存 |                                                   |
| T5   | 将singleton         |                                                   |
| T6   |                     | 检查到`singleton`不为空                           |
| T7   |                     | 访问`singleton`<br />（但此时对象还未完成初始化） |
| T8   | 初始化singleton     |                                                   |

这种情况下，`T7`时刻线程`B`对单例的访问，访问的是一个`初始化未完成`的对象。

`java`中可以使用`volatile`关键字定义单例变量。使用`volatile`关键字后，重排序被禁止，所有写操作都将发生在读操作之前。

`GCC4.0`以前可以使用`pthread_once`来保证初始化

```c++
template<typname T>
class Singleton : boost::noncopyable
{
    public:
        static T& instance()
        {
            pthread_once(&ponce_, &Singleton::init);
            return *value_;
        }

    private:
        Singleton() {}
        ~Singleton() {}

    private:
        static pthread_once_t ponce_;
        static T* value_;
};

template<typename T>
pthread_once_t Singleton<T>::ponce_ = PTHREAD_ONCE_INIT;

template<typename T>
T* Singleton<T>::value_ = NULL;
```

`GCC4.0`以后，可以用局部静态变量来实现。坏处就是创建和析构的不确定性，只要访问`getInstance`就可能导致其生命周期发生变化

```c++
template <typename T>
class SingleTon
{
    public:
    	static T& getInstance()
        {
            static T s;
            return s;
        }
    private:
    	Singleton() {}
       	~Singleton() {}
}
```



### 2.6`sleep`不是同步原语

作者表达了对`sleep`的深恶痛绝，主要是觉得总是得给CPU点活干，不要让它被随意浪费

可以用条件变量或者select来等待（有待考证）

### 2.7总结

1. 提倡正确加锁，不要自己编写lock-free算法，不要开发一些同步设置
2. 在没有实测数据支持下，不要乱优化
3. 分布式系统，多机的伸缩性比单机的性能优化更值得投入精力

## 第3章 多线程服务器的使用场合与常用编程模型

### 3.1进程与线程

有意思的奇妙比喻，把进程比喻成人，内存是人的记忆。人与人交谈形成进程间通信

- 容错	万一有人死了
- 扩容    新人中途加入
- 负载均衡 把甲的活挪给乙做
- 退休    甲要修复bug，先别分派新任务，等他做完手上的事情就把他重启

### 3.2单线程服务器的常用编程模型

最常见的就是`Reactor`模型，采用`non-blocking IO + IO multiplexing`模型，程序的基本结构是一个事件循环，以事件驱动和事件回调的方式实现业务逻辑：

```c++
while (!done)
{
    int timeout_ms = max(1000, getNextTimedCallback());
    int reval = ::poll(fds, nfds, timeout_ms);
    if (reval < 0)
    {
        // 处理错误，回调用户的error handler
    }
    else
    {
        // 处理到期的timers，回调用户的timer handler
        if (retval > 0)
        {
            // 处理IO时间，回调用户的IO event handler
        }
    }
}
```

### 3.3多线程服务器的常用编程模型

#### 3.3.1 one loop per thread

相当于每个线程启动一个事件循环，把数据处理任务分摊到另几个计算线程中

#### 3.3.2线程池

对于没有IO而只有计算任务的线程，使用`event loop`有点浪费，可以用一种补充方案——`blocking queue`实现的任务队列

```c++
void workerThread()
{
    while(running)
    {
        Func task = taskQueue.take();
        task();
    }
}
```

实现可以参考`Java`的`BlockingQueue`，生产者和消费者会更简单

**项目中使用非阻塞的安全队列 + `event loop`实现**

#### 3.3.3推荐模式

总结起来，作者推荐的`C++`多线程服务器编程模式为：`one(event) loop per thread + thread pool`

- `event loop`（也叫`IO loop`）用作`IO多路复用`，配合非阻塞IO和定时器
- `thread pool`用来做计算，具体可以是任务队列或生产者消费者队列

### 3.4进程间通信只用TCP

进程间通信的方式数不胜数，匿名管道、命名管道、消息队列等等

- `TCP sockets`是最自然，也是最需要理解学习的IPC

- `TCP port`由一个进程独占，且操作系统会自动回收

- 两个进程通过`TCP`通信，如果一个崩溃了，操作系统会关闭连接，另一个进程几乎立刻就能感知，可以快速`failover`。当然应用层的心跳也是必不可少的
- 使用`TCP`的字节流通信，要求选用合适的消息格式。主要用`Google PB`

**分布式系统中使用TCP长连接通信**

### 3.5多线程服务器的使用场合

#### 多线程应用场景

- `cpu`是多核的，废话
- 服务不是单一的，比如说除了工作线程、还有

## 第4章C++多线程系统编程精要

### 4.1基本线程原语的选用

- 线程的创建。`std::thread`
- `mutex`的创建、加锁。`std::mutex`、`std::lock_guard`
- 条件变量。`std::condition_variable`

### 4.2C/C++系统库的线程安全性

对标准而言，关键的是规定内存模型。特别是规定一个线程对某个共享变量的修改何时能被其他线程看到，这成为内存序或者内存能见度。

从理论上讲，如果没有合适的内存模型，编写正确的多线程程序属于撞大运行为。





















