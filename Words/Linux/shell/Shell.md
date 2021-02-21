
### 符号

### 循环
for循环基本语法:
```
# !/bin/sh
for var in "test1" "test2"
do
    echo "&var"
done
```
通配符拓展：
```
for var in $(ls ./)
do 
    echo "$var"
done
```

while循环
read pwd
while ["$pwd" != "test"]
do 
    echo "try agin"
    read pwd
done

until循环
read pwd
until [ "$pwd" = "test2" ]
do
echo "try again"
read pwd
done

语法：
```
$(foreach <var>;,<list>;,<text>;)
```

### 常用命令

#### 压缩
tar -czvf mypb.tar.gz pb
