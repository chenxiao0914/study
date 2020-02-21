SVN：属于集中式版本控制

GIT：属于分布式版本控制

# 一、GIT的功能

协同修改：

​	多人并行不悖的修改i同一个文件

数据备份：

​	能够每一个提交过的历史状态

版本管理：

​	SVN才用增量式广利的方式，GIT才用文件系统快照的方式

权限控制：

​	对参与开发的人员进行权限控制，对别的开发者贡献的飞马进行审核

历史记录：

​	查看修改人、修改时间、修改内容等信息；将文件恢复到某一个历史状态

分支管理：

# 二、GIT和代码托管中心

代码托管中心：维护远程库

![托管中心](D:%5Cstudy%5Cnote%5Cgit%5Cimages%5C%E6%89%98%E7%AE%A1%E4%B8%AD%E5%BF%83.PNG)

局域网环境：

​	GitLab服务器

外网环境：

​	GitHub

​	码云

# 三、本地库操作

## 1、本地初始化

​	git init

## 2、设置签名

形式：

​	用户名：user

​	Email地址：xxx@xxx.com

辨析：这里设置的签名和登录远程仓库（代码托管中心）的账号密码无关

命令：

​	项目级别/仓库级别：仅在当前本地库范围内有效

​		git config user.name tom_pro

​		git config user.email  good@xxx.com

​	系统用户级别：登录当前操作系统的用户

​		git config --global user.name tom_glb

​		git config --global user.email  good@xxx.com

## 	3、常用命令

git add xxx		===>	添加（提交）文件到暂存区

git rm --cached xxx		===>	将暂存区的文件恢复到未提交状态

git commit -m "备注内容" xxx		===>	提交到本地库	

git log		===>	查看历史提交记录

​	git log --pretty=oneline

​	git log --oneline

​	git reflog

前进后退版本

​	git reset --hard 88e6276(索引值)		===》	前进后退版本

​	git reset --hard HEAD^^		===>	只能后退，后退两步，一个^表示一步

​	git reset --hard HEAD~3		===》	只能后退，后退三步

比较文件差异

​	git diff  xxx		===>	比较暂存区和工作区的文件

# 四、分支

git branch xxx		===》	创建分支：

git branch -v（-a）===》	查看分支

git checkout xxx		===》	切换分支

合并分支：（必须先切换到被合并的分支（比如：master）

​	git merge sit		===》	将sit分支的修改合并到master上

==解决合并分支时出现冲突：修改完文件后，add后、commit时不需要加写文件名==

# 五、git原理

才用哈希算法来验证文件，从根本上保证数据的完整性。

哈希算法共同点：

​	不管输入的数据多大，同一个哈希算法，得到的加密结果长度固定；

​	哈希算法不可逆；

​	哈希算法确定，输入数据确定，哈希加密结果就固定；

# 六、远程库

## 1、本地创建远程库别名

​	git remote -v

​	git remote add origin https://github.com/chenxiao0914/huashan.git

## 2、拉取

​	pull = fetch + merge

​	fetch  origin master		先拉取，看有无问题

​	merge origin/master		再同步到本地

==发生冲突解决方式==：

​	先拉取pull下来，查看冲突文件，修改好，按合并分支发生冲突方式合并（==修改完文件后，add后、commit时不需要加写文件名==）

# 七、问题

1、git add 时有中文会乱码（[\256\346\200\273\347\273\223](https://www.cnblogs.com/EasonJim/p/8403587.html)），设置方法：

```git
git config --global core.quotepath false
```



 git remote rm origin 