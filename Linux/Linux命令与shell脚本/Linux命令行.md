### 检测程序
- 探测进程
    常用命令：ps -elf
- 实时监控
### 磁盘检查
- 磁盘容量
    df -h
- 文件容量
    du -sh

### 处理文件数据
- 排序
    sort file_name
    -n 按数字排序
    -r 按降序输出
    -M 按时间戳排序
- 搜索
    grep pattern file_name
    -v 反向搜索
    -n 行数
    -c count
- 压缩
    gzip myprog
- 归档tar
    -z 表示将输出重定向给gzip亚索
    tar -zxvf data.tar
    tar -zcvf test.tar data/

### scp命令

```shell
scp -P 58422 data_202104221650.tar.gz fangtao.hu@10.9.30.115:/home/fangtao.hu/serverCode/app/bhxy3/trunk/gameserver
```

### 软链

```shell
ln -s ../gameserver/data data
```





## 网络检测

### netstate

### ss

- `ss`网络连接基本状态
- `ss -s` 显示套接字摘要
- `ss -l`显示监听状态的套接字
- `ss -a`显示所有套接字


# Shell

## shell的内建命令
用type -a 命令，可以区分命令定义位置
- 外部命令是存在于shell之外的程序，一般在/bin、/usr/bin，包括python脚本之类
    会建立子进程来执行命令
- 内建命令，例如cd、ls之类

- 历史命令
    history
    一般用ctrl + R就可以了
- 命令别名
    alias

## 环境变量
### 全局变量
- print env
- env
全局变量使用
echo $HOME

### 局部变量
- set 显示所有变量，包括全局变量、局部变量

## 用户定义变量
### 设置用户定义变量
```
my_variable=HELLO
echo $my_veriable
```

### 设置全局环境变量
```
my_variable="i am Global now"
export my_variable
```

### 删除环境变量
unset my_variable

## 系统环境变量
1. 当你登录linux系统时，bash shell会作为登录shell启动。登录shell会从5个不同的启动文件里读取命令：
    - /etc/profile          // 系统默认主启动文件，还引用profile.d文件夹下的命令
    - $HOME/.bash_profile
    - $HOME/.bashrc         // 一般个人用户修改这个文件实现，永久变量
    - $HOME/.bash_login
    - $HOME/.profile

## 数组变量
```
myarray=(one two three)
echo ${myarray[0]}
echo ${myarray[*]}
```

# 7. 文件权限
## 7.1 Linux安全
- /etc/passwd 文件
    保存用户信息

- /etc/shadow
    真正保存密码的文件

- 添加新用户
    useradd -m fangtao.hu -p 123456   // 创建一个带用户目录的用户，目录名为fangtao.hu，带默认密码

- 删除用户
    userdel -r fangtao.hu

- 修改用户
    - usermod 用来修改/etc/passwd文件中的大部分字段
        -l 修改登录名
        -L 锁定用户
        -p 修改密码
    
    - passwd和chpasswd  修改用户密码
        passpwd fangtao.hu
## 7.2 Linux组
- /etc/group
    包含系统上用到的每个组的信息。

- 创建新组
    groupadd groupname 
    usermod -G groupname username

- 修改组
    groupmod -n newgroupname groupname

## 7.3 文件权限

- 修改文件权限
    chmod 760 newfile
    chmod [u/g/o/a + w/r/x]

- 改变所属关系
    ```
    chown fangtaohu[.group] file
    ```

## 共享文件

# 程序
## 包管理

## Debian的apt
## Red Hat的yum

# 编辑器
## vim编辑器

