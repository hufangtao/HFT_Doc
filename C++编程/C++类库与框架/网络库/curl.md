# 安装

1. 去libcurl官网下载稳定版压缩包：`curl-x.x.x.tar.gz`
2. 解压
3. 配置安装位置`./configure --prefix=/xxx/xxx/`，指定安装位置
4. `make && make install`编译安装
5. 一般地，工程中只需要`include`和`lib`文件夹，分别包含编程所需头文件和库文件



# 包含入工程
```
ifndef CURL_INC
	CURL_INC = true
	ifdef EXT_DIR
		CUR_DIR = ${EXT_DIR}/curl
		INC_DIR += ${CUR_DIR}/include
		LIB_DIR += ${CUR_DIR}/lib
	endif
endif
```

# 编程手册

## 编程流程（easy）

`LibCurl`的程序中，主要采用回调函数的形式完成传输任务，用户在启动传输前设置好各类参数和回调函数，当满足条件时`libcurl`将调用用户的回调函数完成特定功能。`libcurl`完成传输任务的流程：

1. 调用`curl_global_init()`初始化`libcurl`
2. 调用`curl_easy_init()`函数得到`easy interface`型指针
3. 调用`curl_easy_setopt`设置传输选项
4. 根据`curl_easy_setopt`设置的传输选项，实现回调函数以完成用户特定任务
5. 调用`curl_easy_perform()`函数完成传输任务
6. 调用`curl_easy_cleanup()`释放内存

整个过程中设置`curl_easy_setopt`参数是最关键的，几乎所有的`libcurl`程序都要使用它

## 相关函数（easy）

### `curl_global_init(long flags)`

#### 描述：

这个函数初始化`libcurl`库

如果这个函数在`curl_easy_init`函数调用时还没调用，它将由libcurl库自动完成。

因为函数不是线程安全的，所以多线程环境下，最好手动加锁初始化。

### `curl_easy_init()`

#### 描述：

用来初始化一个`CURL`指针（类似FILE类型的指针）。相应的在调用结束是需要对应的`curl_easy_cleanup`函数清理。

一般`curl_easy_init`意味着一个会话开始

### `curl_easy_setopt(CURL* handle, CURLoption option, parameter)`

#### 描述：

定义curl的行为

#### curl行为：

1. `CURLOPT_URL`

   设置访问URL

2. `CURLOPT_WRITEFUNCTION`

   设置回调，输出接收到的数据

   ```c++
   size_t write_callback(char* ptr, size_t size, size_t nmemb, void* userData)
   CURLcode curl_easy_setopt(CURL* handle, CURLOPT_WRITEFUNCTION, write_callback)
   ```

   函数将在`libcurl`接收到数据后被调用。

   一般的，callback会被多次调用，每次输出一部分数据。每次都会按照最大上限输出数据。

   `ptr`指向输出的数据，`nmemb`表示数据大小，`size`总是1。数据总大小为：`size * nmemb`

   `userData`由`CURLOPT_WRITEDATA`来设置

3. `CURLOPT_WRITEDATA`

   ```c++
   CURLcode curl_easy_setopt(CURL* handle, CURLOPT_WRITEDATA, void* pointer)
   ```

   设置指针，传递给`write_callback`。

   如果没有自定义`write_callback`函数，需要传递给该指针一个`FILE*`指针，curl会把数据直接写入文件，否则会默认输出到`stdout`。

### `curl_easy_perform(CURL* handle)`

#### 描述：

执行curl

## 编程流程（multi）

1. 调用`curl_multi_init`初始化一个`multi curl`对象
2. 调用`curl_easy_init`初始化多个`easy curl`对象
3. 使用`curl_easy_setopt`进行相关设置
4. 调用`curl_multi_add_handle`把`easy curl`对象添加到`multi curl`对象中
5. 执行`curl_multi_perform`方法进行并发的访问
6. 让问结束后`curl_multi_remove_handle`移除相关`easy curl`对象
7. 使用`curl_easy_cleanup`清除`easy curl`对象
8. 最后`curl_multi_cleanup`清除`multi curl`对象

## 相关函数（multi）

`easy`接口是阻塞的，一般要等到上一个`curl`请求执行完后，下一个请求才能继续执行。

当程序需要进行多次`curl`并发请求的时候，`easy`接口就无能为力了，这个时候`curl`提供的`multi`接口就派上用场了。

`multi`接口是依赖`easy`接口的。

``













