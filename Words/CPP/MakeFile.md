## MakeFile笔记

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

### MakeFile编写

#### 最基本版本
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
#### 自动推导
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
#### 自动推导
### 语法
#### 执行命令
- 直接用make，make会自动搜索当前目录下的Makefile文件
- 使用make的`'-f'`或`'-file'`参数，如`make -f server.mk`
#### 文件搜索

#### foreach循环
names := a b c d
files := $(foreach n,$(names),$(n).o)
foreach var,$(files),(echo $(var))

#### include
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

#### .PHONY
### 问题
添加一个Makefile的target的时，总会出现“make: Nothing to be done for `xxxxx’”的提示，而书写语法表面正确。
### 原因
Makefile的target和目录或文件名字冲突。
```
A phony target is one that is not really the name of a file;
rather it is just a name for a recipe to be executed when you make an explicit request.
There are two reasons to use a phony target:
to avoid a conflict with a file of the same name, and to improve performance.
If you write a rule whose recipe will not create the target file, the recipe will be executed every time the target comes up for remaking.
```
### 描述
```phony的意思是“赝品”，在这里可以形象的理解成“不是文件”```
### 问题
GNU默认Makefile的taget是一个文件（或目录）
他会检测同级目录下是否已存在这个文件，如果存在，则会abort掉make进程，并提示
```
make: Nothing to be done for `xxxx`
```
### 解决
将target文件添加到phony中

