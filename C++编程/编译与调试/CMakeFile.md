## CMakeFile

```markdown
Clion使用CMakeFile管理工程，所以学习一下，用来代码提示
```

### 编译流程

```markdown
1. 编写CmakeLists.txt
2. 执行命令"cmake PATH"或者"ccmake PATH"生成Makefile（PATH是CmakeLists.txt所在目录）
3. 使用make命令进行编译
```

### 输出

```cmake
message(WARNING "Hello World = ${VER}")
```

### 从第一个文件开始

 现在只有一个main.cpp，在同目录下编写CmakeLists

```cmake
PROJECT(main)
CMAKE_MINIMUM_REQUIRED(VERSION 2.6)
AUX_SOURCE_DIRECTORY(. DIR_SRCS)
ADD_EXECUTABLE(main ${DIR_SRCS})
```

AUX_SOURCE_DIRECTORY：用于搜索当前目录`.`下的所有源文件，**不包含子目录**

### 多文件多目录



