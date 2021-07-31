# MakeFile笔记

## MakeFile编写
### 最基本版本
```
  1 GCC := gcc
  2 GXX := g++
  3
  4 OBJS = main.o player_session.o player.o
  5
  6 server: $(OBJS)
  7     $(GXX) -o server $(OBJS)
  8
  9 main.o: main.cpp player.h
 10     $(GXX) -c main.cpp
 11
 12 player_session.o: player_session.h player_session.cpp
 13     $(GXX) -c player_session.cpp
 14
 15 player.o: player.cpp player_session.h player.h
 16     $(GXX) -c player.cpp
 17
 18 clean:
 19     -rm server $(OBJS)
```

### 自动推导
Make可以自动推导文件以及文件依赖关系后面的命令

推导规则如下：
1. 只要make看到一个[.o]文件，自动把[.c]文件加在依赖关系中。
2. 并且`cc -c whatever.c`也会被推导出来
```
  1 GCC := gcc
  2 GXX := g++
  3
  4 OBJS = main.o player_session.o player.o
  5
  6 server: $(OBJS)
  7     $(GXX) -o server $(OBJS)
  8
  9 main.o: player.h
 10
 11 player_session.o: player_session.h
 12
 13 player.o: player_session.h player.h
 14
 15 clean:
 16     -rm server $(OBJS)
```

### 文件搜索
```
SRC_FILES = $(wildcard SRC_DIR/*.cpp)

```

## 语法

### 基本规则
```
target ... : prerequisites
	command
	...
	...
```
- target表示目标文件，可以是obj文件也可以是可执行文件，可以是单个或多个目标。
- prerequisites是生成target所需要的文件或目标
- command是make需要执行的命令（任意的shell命令）

这是文件的依赖关系，target依赖于prerequisites中的文件，按照command命令生成。

### 执行命令
- 直接用make，make会自动搜索当前目录下的Makefile文件
- 使用make的`'-f'`或`'-file'`参数，如`make -f server.mk`

### 文件搜索

### foreach循环
names := a b c d
files := $(foreach n,$(names),$(n).o)
foreach var,$(files),(echo $(var))

### include
包含库文件，在库文件外层增加MakeFile.inc文件，包含CPP头文件和库文件
在引用的MakeFile种include这个MakeFile.inc即可
```
# MakeFile Boost Include
ifndef BOOST_1_56_0_INC
	BOOST_1_56_0_INC = true
	ifdef EXT_DIR
		CUR_DIR := $(EXT_DIR)/boost_1_56_0
		INC_DIR += $(CUR_DIR)/
		LIB_DIR += $(CUR_DIR)/stage/lib/
	endif
endif
```

### .PHONY
添加一个Makefile的target的时，总会出现“make: Nothing to be done for `xxxxx’”的提示，而书写语法表面正确。
因为GNU默认Makefile的taget是一个文件（或目录）
他会检测同级目录下是否已存在这个文件，如果存在，则会abort掉make进程，并提示
```
make: Nothing to be done for `xxxx`
```
Makefile的target和目录或文件名字冲突。
```
A phony target is one that is not really the name of a file;
rather it is just a name for a recipe to be executed when you make an explicit request.
There are two reasons to use a phony target:
to avoid a conflict with a file of the same name, and to improve performance.
If you write a rule whose recipe will not create the target file, the recipe will be executed every time the target comes up for remaking.
```
```phony的意思是“赝品”，在这里可以形象的理解成“不是文件”```

### 多目标

### define

定义变量
```
define two-lines
echo foo
echo $(bar)
endef
```
变量名：two-lines
变量值：可以是函数、命令、文字或是其它变量。命令需要以[Tab]键开头

### 条件判断
```
ifeq ($(CC), gcc)
  $(CC) -o foo $(libs_for_gcc)
else
  $(CC) -o foo $(normal_libs)
endif
```

判断函数：
ifeq：两值相等
ifneq：两值不等
ifdef：测试一个变量是否有值
ifndef：与ifdef相反

### 函数
调用方法：
```
$(<function>; <arguments>;)
```

## 应用

### 自动生成依赖关系

**参考：**https://blog.51cto.com/u_11134889/2108280

一般C++的Makefiel中，目标的编译语法都是：

```shell
$(OUT_DIR)/%.o: %.cpp
	$(XX) $(C_FLAGS) -c $< -o $@
```

目标`.o`文件只依赖`.cpp`文件，只有`.cpp`文件变动时，才会重新编译。如果我们修改了头文件，并不会重新编译，会导致错误。

为了弥补错误。我们在Makefile要手动创造这种依赖。这就是`dep`的作用。

**实现前提1：**GCC通过`-MM`参数，可以获取本文件依赖的文件。因为这个地方出来的是`haclient_mgr.o`不带路径，一般用`sed`来为他加上路径，例如`out/haclient_mgr.o`。`sed`命令一般为`sed 's,\(.*\)\.o[:],'$(OUT_DIR)'\/\1.o: ,g'`

```bash
[fangtao.hu@bh3dev4 code]$ gcc -MM hnet/asio_client/haclient_mgr.h -I./
haclient_mgr.o: hnet/asio_client/haclient_mgr.h ext_head.h \
 tools/thread_group.h hlog/hlogger.h hlog/hlog_stream.h \
 tools/noncopyable.h tools/fixed_buffer.h hlog/log_file_mgr.h \
 tools/file_utils.h tools/noncopyable.h
```

**实现前提2：**对于一个目标，可以声明多次依赖。最终目标是多次依赖的合集

```makefile
test: a b

test: b c
# test最终结果是 a、b、c
```

**实现前提3：** `Makefile`里的`include`关键字，是直接替换内容。如果被include的文件不存在，会根据对应规则生成。

**最终实现：**通过`gcc -MM`获取`cpp`文件的依赖，并用include包含进Makefile。
