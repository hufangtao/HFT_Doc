## git操作

#### 常用操作
![blockchain](https://raw.githubusercontent.com/hufangtao/HFT_Doc/master/%E5%9B%BE%E7%89%87/290760-e8491f69473bf200.jpg "图片hover描述")


#### http免密提交
cd ~
vim .git-credentials
https://{username}:{password}@github.com
git config --global credential.helper store

#### git子模块
cd到相应的目录下，执行：
git submodule add https://github.com/submoudle/submoudle.git
Cloning....

在其他地方拉取的时候，只能拉取到submoudle文件夹，文件夹中没有东西。
主工程拉取完了之后，cd submoudle文件夹，然后执行：
git submodule init和git submodule update 则从子项目中抓取所有文件