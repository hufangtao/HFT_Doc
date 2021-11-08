# LibEvent剖析

```
最近学muduo的时候，了解到evbuffer的设计，觉得非常有意思，所以决定学习一下。辅以《libevent源码深度剖析》。
```

## 基础接口

### 通用的双链表

### EvBuffer

```
Evbuffer使用链表作为基础结构，尽量不拷贝不移动。
```

evBuffer总共包含三个文件：

```c++
buffer.h: 相关函数定义
evbuffer-internal.h: evbuffer具体定义
buffer.c: 相关函数实现
```

#### evbuffer_chain

链表节点基础结构，负责保存数据

```c++
struct evbuffer_chain {
	// 链表
	struct evbuffer_chain *next;
	// 实际内存分配的可用长度
	size_t buffer_len;
	// 前置空位
	ev_misalign_t misalign;
	// 有效buffer长度
	size_t off;
	// 引用计数
	int refcnt;
	// 实际指针
	unsigned char *buffer;
};
```



#### evbuffer_iovec

使用链表buffer，就离不开使用iovec。看看evbuffer是怎么用的？

```c++
evbuffer_add：evbuffer的基础写入函数
evbuffer_add_iovec：写入iovec的数据
evbuffer_copyout：从evbuffer中直接拷贝到buffer，但不删除原内容
evbuffer_drain：删除原内容
```



