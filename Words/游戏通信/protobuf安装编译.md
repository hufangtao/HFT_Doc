## protobuf相关使用

#### 安装
在GitHub上下载最新的protobuf.x.x.x.tar.gz，按照要求安装即可

#### 编译选项
添加g++编译选项
```
-lprotobuf
```

#### proto版本问题
优先使用proto3版本

#### 编写proto
proto文件定义类型主要包括：
头文件
``` c++
syntax = "proto3";  // 定义proto版本
package ErrorCode;  // 包名
import "test.proto" // 引入其他proto文件
```
message：类似于class的定义
``` c++
message P_Player
{
    int32 id = 1;
    string name = 2;
    int32 sex = 3;
    string head_img = 4;
}
```
enum：枚举类型定义，**下标必须从0开始**
``` c++
enum Error
{
    E_OK            = 0;    // 成功
    E_UNKNOWN       = 1;    // 未知错误
}
```

#### 编译proto生成.h和.cc文件
``` shell
protoc -I=$(PROTO_DIR) --cpp_out=$(PROTO_DEST) $(PROTO_FILE)
```
