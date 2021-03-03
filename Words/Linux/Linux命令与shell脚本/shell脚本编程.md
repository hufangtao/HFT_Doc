# Shell脚本编程
## 显示消息
    echo

## 使用变量
- 变量定义
```
variable=1
${variable}
```

- 命令替换
    从命令输出中提取信息，并将其赋给变量：
    1. 反引号字符
    2. $()格式

## 重定向输入和输出

- 输出重定向
    command > outputfile
    command >> outputfile 在文件后追加

- 输入重定向
    command < inputfile

- 管道
    command1 | command2

## 数学运算

- expr
    expr 1 + 5

- 方括号
    var1=${[1 + 5]}

- 浮点
    bc

## 退出脚本

- 退出状态码
    echo $?     // 输出shell执行的最后一条命令的退出状态码

- 退出命令
    exit    // 类似于return，但是状态码区间只能：0~255

# 结构化命令

## if-then语句
```
if pwd; then
    echo "It worked"
elif ls; then
    echo "ls worked"
else
    echo "It not worked"
fi
```
如果pwd命令执行成功

## test命令
测试退出状态码之外的条件
简化使用[]

- 数值比较
    `[n1 -eq n2]`
    -eq     // 相等
    -ge     // 大于等于
    -gt     // 大于
    -le     // 小于等于
    -lt     // 小于
    -ne     // 不等于

- 字符串比较
    `str1 = str2`
    =       // 相同
    !=      // 不同
    <       // 小于
    `\>`       // 大于，需要反斜杠，避免被认为是重定向
    -n      // 长度是否非0
    -z      // 长度是否为0

- 文件比较
    测试Linux文件系统上文件和目录的状态
    `[-d file]`
    -d      // 是否是目录
    -e      // 是否存在
    -f      // 是否文件
    -r      // 是否可读
    -s      // 是否非空
    -w      // 是否可写
    -x      // 是否可执行
    -O      // 是否owner
    -G      // 是否同Group
    -nt     // 是否更new    `file1 -nt file2`
    -ot     // 是否更old

## 复合条件
    if-then语句允许你使用布尔逻辑来组合测试，两种布尔运算：
    &&      // 且运算
    ||      // 或运算

## if-then高级特性
- 双括号
    `((expression))`
    expression可以是任意的数学赋值或比较表达式
- 双方括号
    `[[expression]]`
    提供模式匹配功能

## case命令
    类似switch-case
    ```
    case $USER in
    fangtao.hu | aaa.xxx)
        echo "Welcome user"
    testing)
        echo "Welcome test account"
    *)
        echo "not allow"
    ```

## for命令
```
for var in member1 member2
do
    commands
    echo "The next state is $test"
done
```
- 从命令读取值
```
for state in $(cat $file)
do
    echo "Visit beauty $state"
done
```

- 更改字段分隔符
    shell脚本默认使用空格分割解析列表
    可以在shell脚本中历史更改`IFS`环境变量来限制被bashshell当作字段分隔符的字符

## C语言风格的for命令
```
for (( i=1; i <= 10; i++))
do
    echo "The next number is $i"
done
```

## while命令
```
while test command
do
    commands
done
```

## until命令
```
until test commands
do
    commands
done
```

## 控制循环
    break
    特殊地，有个break n，可以指定跳出n层循环
    continue

## 循环输出
```
for file in /home/fangtao.hu/*
do
    echo "xxx"
done > output.txt
```
重定向到output

# 处理用户输入

## 命令行参数

- 位置参数
    $0是程序名，$1是第一个参数，$2是第二个参数，以此类推一直到9

## 特殊参数变量
- 参数统计
  - 特殊变量`$#`含有脚本运行时携带的命令行参数的个数

- 抓取所有数据
  - `$*`和`$@`变量可以用来访问所有的参数

- 移动变量
  - 使用`shift`命令，将每个参数变量向左移动一个位置。

## 处理选项

## 获取用户输入

- 基本读取
  - `read`命令从标准输入，`read -p "Enter your name:" first last`

- 超时
  - `read -t 5 -p "Enter your name:" first last ` ，5s超时
- 隐藏方式读取
  - `read -s`读取密码
- 从文件中读取

# 呈现数据

## 理解输入和输出

- 标准文件描述符
  - 特殊的文件描述符
    - 0 STDIN 标准输入
    - 1 STDOUT 标准输出
    - 2 STDERR 标准错误
  - 重定向错误
    - `2 > errfile1, 1 > outfile1`
    - 特殊的重定向符号`&>`，将STDERR和STDOUT同时重定向

## 在脚本中重定向输出

- 临时重定向
  - `echo "This is an error message" >& 2`
- 永久重定向
  - `exec 1>testout`

## 在脚本中重定向输入

- `exec 0< testfile`

## 创建自己的重定向

- 创建输出文件描述符
  - 类似的，可以自己创建一个3，`exec 3>test3out`
- 重定向文件描述符

## 列出打开的文件描述符

- `lsof -a -p $$ -d 0,1,2`

## 阻止命令输出

将命令输出重定向到`/dev/null`

## 创建临时文件

- `mktemp testing.XXXXXX`命令会用6个字符码替换这6个X，保证文件名在目录中是唯一的。
- 在`/tmp`目录创建临时文件：`mktemp -t testing.XXXXXX`
- 创建临时目录：`mktemp -d dir.XXXXXX`

## 记录消息

- `tee filename`命令将从`STDIN`过来的数据同时发往两处，一处是`STDOUT`，另一处是命令行所指定的文件名

# 控制脚本

如何在`Linux`系统上运行和控制脚本。控制方法包括：向脚本发送信号、修改脚本优先级以及在脚本运行时切换到运行模式。

## 处理信号

- 生成信号
	- 中断进程：`Ctrl+C`
	- 暂停进程：`Ctrl+Z`
- 捕获信号
  - `trap "echo 'Sorry! I have trapped Ctrl-C'" SIGINT`
  - `trap "echo Goodbye..." EXIT`捕获脚本的退出
- 修改或移除捕获
  - `trap -- SIGINT`

## 后台运行脚本

- 后台运行
  - `./test.sh &`

## 在非控制台下运行脚本

前面的后台运行脚本是终端绑定的，终端退出脚本也会退出。

`nohup`命令运行了另一个命令来阻断所有发给该进程的`SIGUP`信号，这会在退出终端会话时阻止进程退出

- `nohup ./test1.sh > /dev/null 2>&1 &`

## 作业控制

- 重启停止的作业
  - 当作业`Stoped`，以后台模式重启一个作业，可以用`bg`命令加上作业号
  - 以前台模式重启作业，用`fg`命令加上作业号

## 调整调度优先级

调度优先级是个整数，从-20（最高优先）到19（最低优先）。默认情况下shell以优先级0来启动所有进程

- `nice`命令
  - `nice -10 ./test.sh > output.txt`
  - 以指定优先级启动
- `renice`命令
  - `renice -n 10 -p 5055`
  - 指定运行中进程的优先级

## 定时运行作业

### `at`命令

​	在指定时间执行任务

### `cron`时间表

支持每周一次或每月一次这种方式执行任务

- `cron`时间表采用一种特别的格式来指定作业何时运行，格式如下：

  `min hour dayofmonth month dayofweek command`

  时间表允许你用特定值、取值范围（例如1~5）或者是通配符（星号）来指定条目。

- `crontab -l`列出已有的时间表

- `cron`目录在`/etc/cron.*`，有四个基本目录

- `anacron`可以保存Linux在关机期间没能执行的命令

# 创建函数

## 基本的脚本函数

- 定义函数

  - ```shell
    function name {
    	commands
    }
    ```

## 返回值

- 默认情况下，函数的退出状态码是函数中最后一条命令返回的退出状态码。

  在函数执行结束后，可以用标准变量`$?`来确定函数的退出状态码

- 使用`return`命令来退出函数并返回特定状态码

- 使用函数输出`result=$(dbl)`会将`dbl`函数的输出赋值给`result`

## 在函数中使用变量

- 向函数传递参数

  - 将参数和函数放在同一行：

    `func1 $value1 10`

  - 与脚本参数类似，函数中可使用`$# $1 $2`来获取变量

- 在函数中处理变量

  - 全局变量：一般的，直接定义的变量`value=6`都是全局的
  - 局部变量：在定义前加上`local`关键字就行，`local temp`

## 数组变量和函数

## 创建库

类似C++的静态库

- 创建库，在库文件中定义函数
- 使用source，或者快捷方式：`. ./myfuncs`
















