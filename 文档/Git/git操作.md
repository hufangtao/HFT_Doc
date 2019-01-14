## git操作

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