## protobuf C++ Api 翻译
总共包含4个包：
bao|desc
|:-|:-|
google::protobuf|核心包
google::protobuf::io|使用protobuf的I/O辅助类
google::protobuf::util|工具类
google::protobuf::compiler|编译类

#### google::protobuf 核心

###### arena
定义一个arena内存分配器
###### descriptor
在运行状态下，可以获取Message的所有字段，一般用于对Message类型未知的情况下。比如服务器传递
包括EnumDescriptor、EnumValueDescriptor、DesciriptorPool（用于构建）

###### message
定义Message类型
proto中定义：
```
message Foo {
  optional string text = 1;
  repeated int32 numbers = 2;
}
```
生成的Foo类中，继承自Message类
对于text字段包含如下对应函数
```
inline void clear_text();
inline const ::std::string& text();
inline void set_text(const ::std::string& value);
inline void set_text(const char* value);
```
还有静态函数：
```
static const ::google::protobuf::Descriptor* descriptor();
```
在程序中声明Message：
方式一：
直接通过类型操作
```
{
  // Create a message and serialize it.
  Foo foo;
  foo.set_text("Hello World!");
  foo.add_numbers(1);
  foo.add_numbers(5);
  foo.add_numbers(42);

  foo.SerializeToString(&data);
}

{
  // Parse the serialized message and check that it contains the
  // correct data.
  Foo foo;
  foo.ParseFromString(data);

  assert(foo.text() == "Hello World!");
  assert(foo.numbers_size() == 3);
  assert(foo.numbers(0) == 1);
  assert(foo.numbers(1) == 5);
  assert(foo.numbers(2) == 42);
}
```

方式二：
通过descriptor获取修改Message类型的定义字段等等
reflection获取和修改Message内容
```
{
  // Same as the last block, but do it dynamically via the Message
  // reflection interface.
  Message* foo = new Foo;
  const Descriptor* descriptor = foo->GetDescriptor();

  // Get the descriptors for the fields we're interested in and verify
  // their types.
  const FieldDescriptor* text_field = descriptor->FindFieldByName("text");
  assert(text_field != NULL);
  assert(text_field->type() == FieldDescriptor::TYPE_STRING);
  assert(text_field->label() == FieldDescriptor::LABEL_OPTIONAL);
  const FieldDescriptor* numbers_field = descriptor->
                                         FindFieldByName("numbers");
  assert(numbers_field != NULL);
  assert(numbers_field->type() == FieldDescriptor::TYPE_INT32);
  assert(numbers_field->label() == FieldDescriptor::LABEL_REPEATED);

  // Parse the message.
  foo->ParseFromString(data);

  // Use the reflection interface to examine the contents.
  const Reflection* reflection = foo->GetReflection();
  assert(reflection->GetString(*foo, text_field) == "Hello World!");
  assert(reflection->FieldSize(*foo, numbers_field) == 3);
  assert(reflection->GetRepeatedInt32(*foo, numbers_field, 0) == 1);
  assert(reflection->GetRepeatedInt32(*foo, numbers_field, 1) == 5);
  assert(reflection->GetRepeatedInt32(*foo, numbers_field, 2) == 42);

  delete foo;
}
```
