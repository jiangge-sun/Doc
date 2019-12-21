# Sencore Git Gerrit
## 1. Introduction 
### Git 的常规初始化:
1. 在新建目录下运行 `git init`;
2. `git config --global user.name  "name" ` && `git config --global user.email "123@126.com"`
3. `git remote add origin "xxx.git: ` 关联项目地址
4. `git fetch origin master` && `git merge origin/master` 抓取远程代码合并到本地
5. 在远程创建一个新的分支 `git push yourgitname.git dev/yournewbranch:remote_new_branch`

## 2. Normal
### 修改文件提交
1. `git add sb.cpp` 添加新修改的代码
2. `git commit` 进行一次提交
3. `git push yourgitname.git dev/yournewbranch:remote_new_branch`<br/>
注 : `git add` 针对的是某次修改，可能一次修改文件add之后再次修改文件，需要再次add；

> 当想对不同的提交进行修改时. 例如 : 一个项目在一个本地分支上分别提交了三次, 现在处于第三次提交之后, 正在修改第三次提交的内容, 此时需要对第二次提交的内容进行修改, 可以按照以下方式来暂存:
1. `git status` 查看此时代码修改的状态
2. `git add` 在这次修改中已经确定可以提交的代码, 然后`git commit --amend`; 这个时候会有一些代码没有添加在这次新的提交中
3. `git stash` 使这些没有检查过的代码进入暂存区.
4. `git log --pretty=oneline` 显示提交日志(如果有过--amend, 建议使用 `git reflog` )
5. `git reset --hard xxx` 回溯到第二次提交
6. 修改并提交之后再回溯到第三次提交 `git reset --hard thirdcommit`
7. `git stash pop` 取出在暂存区中的代码

### 撤销修改
1. 修改了文件还没有add，`git checkout -- sb.cpp` 丢弃工作区修改
2. 已经add了，`git reset HEAD sb.cpp` 从暂存区返回到工作区
3. commit之后，先要 `git reset` 回溯

### 分支合并
1. `git checkout -b dev/sbsbsb`  在所在分支的基础上切换到新建的分支上
2. 修改新建分支上代码并提交，然后切回到工作分支上
3. `git merge dev/sbsbsb` 合并分支 

## 3. Sencore Gerrit
本次项目提交时所遇到的问题是，需要在同一个topic下产生多次提交，但是修改代码 `review -d 110` 时每次都会在本地产生一个review分支，但是如果几次提交的Topic相同的话，新下载的review分支会覆盖掉之前的分支；解决办法有两个：

### 解决办法：
问题： 一个本地分支有过3次 `commit ` `review`，如果当前需要更改第二次的提交，并且又想把更改后的代码作用在当前的状态；
1. 回溯到第二次提交的状态 `git reset --hard xxx`
2. 更改代码然后 `git commit --amend` `git review`
3. 在这次提交之后 `git checkout -b dev/after_commit2` 复制一个新的分支在本地
4. 然后切换回开发分支，在开发分支合并刚复制出来的分支 `git merge dev/after_commit2` 
5. 一般会产生冲突，需要在编辑器内解决合并冲突
6. 然后再添加这次修改、提交 `git add --all` `git commit --amend` 此时会弹出提示需要先提交本次合并
7. `git commit`产生一次新的提交，此时会在 `commit log` 添加了两次提交记录，需要将合并这几次提交记录
8. `git rebase -i commit2` 重新指向第二次提交之后，`git rebase --continue`完成重定向

> 另：在创建了指定Topic的开发分支名称后，可以每一阶段都复制一个分支分别开发，在各个复制分支上进行提交review，然后再gerrit更改Topic

### 关于`scons teamcity`
在每次上传审核代码之前都要对当前代码进行格式上的修复，比如多出空格这种；命令为 `scons fixerrors`；但是有一天我发现明明 `fixerrors success` 了，还是无法使用 `scons teamcity` 编译通过，上传gerrit之后依然有错误；出现这种问题是因为 `scons fixerror` 的使用时机有问题；需要注意以下几点：
1. 使用 `scons fixerrors` 修复问题之前，需要先 `git add` ；
2. `scons fixerros` 之后一定要 `git status` 检查以下当前状态，如果代码有格式问题，会将该问题修改，也就是代码再次发生了更改，需要重新 add
3. 