
## 语法
foreach循环
names := a b c d
files := $(foreach n,$(names),$(n).o)
foreach var,$(files),(echo $(var))

## include
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

## .PHONY
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

