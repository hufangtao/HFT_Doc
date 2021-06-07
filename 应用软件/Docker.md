# Docker

## Docker安装
docker守护进程启动的时候，会默认赋予名字为docker的用户组读写Unix socket的权限，因此只要创建docker用户组，并将当前用户加入到docker用户组中，那么当前用户就有权限访问Unix socket了，进而也就可以执行docker相关命令  
```
sudo groupadd docker     #添加docker用户组
sudo gpasswd -a $USER docker     #将登陆用户加入到docker用户组中
newgrp docker     #更新用户组
docker ps    #测试docker命令是否可以使用sudo正常使用
```

## 网络设置
在docker启动的时候可以设置网络配置  
`docker run -i -t --net=host ubuntu:15.10 /bin/bash`  
- brige模式：走docker0网卡
- host：和宿主机一样，直接对外

## 容器
容器后台运行：`-d`参数表示在后台运行  
`docker run -itd --name ubuntu-test ubuntu /bin/bash`  
容器列表：`docker ls -a`  
![image](https://user-images.githubusercontent.com/13391912/120773990-68234a00-c554-11eb-90aa-a36f08e403c0.png)
容器启动：
启动一个已经停止容器  
`docker start b750bbbcfd88 `  
容器停止：`docker stop <容器 ID>`  
容器重启：`docker restart <容器 ID>`  
进入容器，共有两种方式进入容器：
- docker attach
- docker exec：推荐使用`docker exec`命令，因为此退出容器终端，不会导致容器的停止。
`docker exec -it 243c32535da7 /bin/bash`  

删除容器：`docker rm -f 1e560fca3906`
清空所有停止的容器：`docker container prune`  

## 镜像
镜像列表：`docker images`  
获取一个新镜像：`docker pull ubuntu:13.10`  
查找镜像：`docker search httpd`  
删除镜像：`docker rmi hello-world`  
更新镜像，在docker run之后，进入镜像更改，然后提交更改  
```
docker commit -m="has update" -a="runoob" e218edb10161 runoob/ubuntu:v2
```
构建镜像，
