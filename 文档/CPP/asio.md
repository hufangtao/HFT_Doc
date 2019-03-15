# boost::asio 异步读写库

### io_context
```
系统io上下文
```
### endpoint
```
终端包括ip和port
```
``` cpp
tcp::endpoint endpoint(tcp::v4(), port);
```

### resolver
```
解析ip/域名
返回通过域名解析出的endpoints列表
```

### 