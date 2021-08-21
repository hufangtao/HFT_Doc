## Muduo设计

跟着muduo的设计，自己手撸一遍代码

### EventLoop

一个啥都干，但是啥都不干的类，类似`asio`的`io_service`。

主要负责循环调用Poller

1. muduo用了一个threadlocal来控制EventLoop是否在多个线程声明

   这点和asio不一样，会导致一些问题，比如我同时生命多个event_loop，只不过跑loop的线程不同。

   而且析构函数中清除了threadlocal对象，会不会有创建和销毁不在同一线程（不知道会不会有问题）

   既然它啥都不干，为啥又要跟线程绑定呢？

2. 持有一个Poller

### Poller

负责IO多路复用，监听fd对应的事件。向Channel填充事件

### Channel

负责管理fd，设置和处理对应的事件回调。

### TimerQueue

基于EventLoop实现一个定时器功能。定时器的实现其实多种多样，TimerWheel形式之类的。
