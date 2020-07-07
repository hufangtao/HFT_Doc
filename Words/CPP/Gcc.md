## gcc相关
调整Ubuntu下gcc版本
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.9 100

## 编译选项
-Wl,-dn

### ar命令
```
库文件操作命令：ar
让我们能产看函数库里的详细情况和用多个对象文件生成一个库文件
```
常用命令：
```

```

### gdb命令
```
gdb attach pid
```
GDB连接已存在的进程

断点：`break code/xxx.cpp:144` 源文件：行
输入c继续运行
触发断点

查看端点处内容：
p 变量名
删除断点
```
delete
用法：delete [breakpoints num] [range…]
delete可删除单个断点，也可删除一个断点的集合，这个集合用连续的断点号来描述。
例如：
delete 5
delete 1-10
clear
用法:clear 
    删除所在行的多有断点。
    clear location
clear 删除所选定的环境中所有的断点
clear location location描述具体的断点。
```
