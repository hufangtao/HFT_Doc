## 记录使用中出现的问题

#### 编码问题
再不同平台的通信的时候，用于socket传输的数据类型可能不同。
例如：c++中使用的char[]发送数据，而c#中使用byte[ ]。表面上会造成双方使用数据类型不统一的情况。而protobuf并没有其他序列化的api。
c#只提供了ToByteArray的方法，而c++只有SerializeToString的方案。
不禁让我们有疑问，c#序列化后的bytearray是否能被c++的ParseFromString解析出来。
其实很简单的一个知识点，protobuf的序列化是将对象序列化成二进制的，无论是bytearray还是string，只是表现形式不同而已。所使用的二进制其实是同一个。