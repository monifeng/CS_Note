# Git

## 1. 基本操作

### 01 git init

在该文件夹初始化仓库后，才能进行后续操作



### 02 git add

提交到暂存区，然后才能commit；



### 03 git commit

git提交，但是此时并没有提交到远程仓库，而是本地仓库更新；



### 04 git push

推送到远程仓库，这样就可以更新了。



### 05 git rm 

在仓库中删除，同时也会在本地文件夹删除，如果误删记得恢复！

![image-20230422115947463](C:\Users\OMEN\AppData\Roaming\Typora\typora-user-images\image-20230422115947463.png)



### 06 git reset

```bash
git reset HEAD 文件名/目录名

git checkout 文件名/目录名
```

git reset 这个就是进行回退的具体命令，这里先介绍他的几个参数

--soft 、--mixed以及--hard是三个恢复等级。

- 使用--soft就仅仅将头指针恢复，已经add的暂存区以及工作空间的所有东西都不变。
- 如果使用--mixed，就将头恢复掉，已经add的暂存区也会丢失掉，工作空间的代码什么的是不变的。
- 如果使用--hard，那么一切就全都恢复了，头变，aad的暂存区消失，代码什么的也恢复到以前状态.

git reset --head可以回退到未提交版本；



### 07 git push

推送到远程仓库，如果第一次推送，要先绑定到远程仓库`git remote add "YOUR GITHUB"`

第一次推送，需要

```bash
git push -u origin main
```

现在github默认分支为main分支，master还要合并，不如直接操作main分支。

![image-20230422122202869](C:\Users\OMEN\AppData\Roaming\Typora\typora-user-images\image-20230422122202869.png)



### 08 git pull

拉取远程仓库，建议在修改本地仓库之前都拉取一次，避免文件冲突。





## 2. 连接远程仓库的一般步骤

1. 初始化本地仓库，在gitbash界面进入某个文件夹，然后输入`git init`；

2. 确认是否生成SSH并且绑定到自己账号（如果已绑定可以忽略这一步）；

3. 连接远程仓库：

   `git remote add origin "YOUR GITHUB REPOSETORIES ADDRESS"`

   ![image-20230423151947168](C:\Users\OMEN\AppData\Roaming\Typora\typora-user-images\image-20230423151947168.png)

4. 如果默认生成了README.md，一定要重新先gitpull，同步远程仓库，然后再进行修改，否则会出现错误；

   ![image-20230423152101628](C:\Users\OMEN\AppData\Roaming\Typora\typora-user-images\image-20230423152101628.png)

5. `git pull --rebase origin main` 我的仓库默认分支为main，可以是其他分支，但建议为main，保存的会直接修改到main上面。

6. git push -u origin（第一次）；





