## git操作

#### 优秀参考文献
[图解Git](https://marklodato.github.io/visual-git-guide/index-zh-cn.html "图解Git")

#### 常用操作
![blockchain](https://raw.githubusercontent.com/hufangtao/HFT_Doc/master/%E5%9B%BE%E7%89%87/290760-e8491f69473bf200.jpg "图片hover描述")


#### http免密提交
```
cd ~
vim .git-credentials
https://{username}:{password}@github.com
git config --global credential.helper store
```

#### git子模块
cd到相应的目录下，执行：
```
git submodule add https://github.com/submoudle/submoudle.git
Cloning....
```

在其他地方拉取的时候，只能拉取到submoudle文件夹，文件夹中没有东西。
主工程拉取完了之后，cd submoudle文件夹，然后执行：
git submodule init和git submodule update 则从子项目中抓取所有文件

子模块提交？
直接cd到子模块目录下，进行正常git操作既可

#### 解决HEAD游离问题（HEAD DETACHED FROM）
游离问题是指：HEAD指针指向了一个非branch的提交，导致了一个假分支状态。在git branch命令下可以看到
```
$ git branch
* (HEAD detached from 9b7039a)
  master
```
显示当前处于一个未命名的分支，id为9b7039a。
因为9b7039a包含了一次提交，所以解决方案分为两种：
第一种：这些提交不重要，那直接checkout master分支即可。
第二种：这些分支的代码很重要，那创建一个新分支A指向9b7039a。然后checkout主分支，merge A到master上即可。