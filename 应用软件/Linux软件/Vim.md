## Vim操作

### 多看help参数

### 打开多窗口、多文件切换

#### vim还没有启动的时候：
1. 在终端里输入 vim file1 file2 ... filen便可以打开所有想要打开的文件
2. vim已经启动 输入 :e file 可以再打开一个文件，并且此时vim里会显示出file文件的内容。
3. 同时显示多个文件： :sp //水平切分窗口 :vsplit //垂直切分窗口

#### 在文件之间切换：
1. 文件间切换 Ctrl+6 //两文件间的切换
```
:bn //下一个文件
:bp //上一个文件
:ls //列出打开的文件，带编号
:b1~n //切换至第n个文件 对于用(v)split在多个窗格中打开的文件，这种方法只会在当前窗格中切换不同的文件。
```
2. 在窗格间切换的方法
```
Ctrl+w+方向键——切换到前／下／上／后一个窗格
Ctrl+w+h/j/k/l ——同上
Ctrl+ww——依次向后切换到下一个窗格中
```
### 删除所有内容
gg 这里是跳至文件首行 再执行：dG 

### vim升级8.0

没啥用，对linux版本有要求。

```shell
rpm -Uvh http://mirror.ghettoforge.org/distributions/gf/gf-release-latest.gf.el7.noarch.rpm
rpm --import http://mirror.ghettoforge.org/distributions/gf/RPM-GPG-KEY-gf.el7
yum -y remove vim-minimal vim-common vim-enhanced sudo
yum -y --enablerepo=gf-plus install vim-enhanced sudo
```

