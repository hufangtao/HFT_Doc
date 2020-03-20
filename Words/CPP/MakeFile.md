
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


