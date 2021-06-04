# Docker

## 
## 网络设置
在docker启动的时候可以设置网络配置  
`docker run -i -t --net=host ubuntu:15.10 /bin/bash`  
- brige模式：走docker0网卡
- host：和宿主机一样，直接对外

## 容器操作
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
