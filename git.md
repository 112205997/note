## <1> Git 常用命令 ##
**分支命令**
- 
- git branch  查看当前分支
- git branch -v             本分支的最近提交
- git branch [新分支名]        创建分支
- git branch [新分支名] [commit_id]  在某个提交上创建分支
- git branch -m [原分支名] [新分支名]  分支改名
- git branch -d [分支名]    删除分支
- git branch -D [分支]      强行删除

**合并命令**

- git merge [分支名]        合并分支
- git merge -no-ff 禁用fast-forward 多出一个CommitID

**分支切换**
- 
- git checkout [分支]      切换分支
- git checkout -b [新分支名] 创建并切换分支

**文件提交**

- git add     添加到暂存区（追踪和未追踪）解决冲突标记
- git add . 添加所有文件到暂存区（不要使用*，*会忽略ignore的规则）
- 
- git commit      提交到版本库
- git commit -am ""    等同于 add . + commit  只作用于已纳入版本管理的文件，新加时还是要add
- git log --graph 图形化日志
- git rm [文件] 删除
- git rm -f [文件] 强制删除
- git rm --cached [文件] 从暂存区删除
- git mv [] [] 重命名文件


## <2> Git 版本回退 ##
- git reset -hard HEAD^
- git reset --hard  HEAD ~1
- git reset --hard commit_id
- git reflog      查看操作日志，HEAD的变化日志（分支切换和提交）
- git reset HEAD [文件名]    将暂存区的提交撤销，放回工作区
- git checkout -[文件名]   丢弃修改，恢复到暂存区的内容

- git checkout [commit_id]   回到之前的某个历史提交，HEAD处于游离状态，这个状态下的修改需要提交后才能切换分支，一般用在在历史版本开分支 	

- git stash save [说明]      分支切换时，有未提交的时候，不能直接切换。用该命令临时保存（紧急切换，并不想提交时）
- git stash list  查看临时保存列表
- git stash pop   临时状态拿出合并，并删除
- git stash apply 临时状态拿出合并，不删除
- git stash apply stash@{编号} 拿出指定状态
- git stash drop stash@{编号} 删除状态

## <3> Git 标签与diff ##
- git tag [标签名]  创建轻量级标签
- git tag -a [标签名] -m [附注信息] 创建附注标签
- git tag -d [标签名] 删除标签 
- git tag   查看标签
- git tag  -l [*匹配] 查寻标签

- git blame [文件名] 文件修改历史
- ---
- linux 原生diff
	-   diff [源] [目标]
	-   diff -u [源] [目标]
- ---

  
- Git 的diff
- 1）工作区-暂存区比较
   -	git diff  暂存区原始文件 工作区目标文件
- 2）工作区-与本地版本比较  版本库原始文件 工作区目标文件
   -	git diff [commit_id]  某版本
   -	git diff HEAD  最新版本
- 3）暂存区-版本库比较  暂存区原始文件 版本库目标文件
   -	git diff --cached  比较最新
   -	git diff	--cached [commit_id] 某版本
   
## <4> 远程协作 ##
- push 推送 完整写法 git push origin [src,本地分支]:[dest,远程分支]
- pull 拉取，同时merge  =fetch+merge  完整写法 git pull origin [src,远程分支]:[dest,本地分支]

- git remote add [origin远程别名] [远程地址] 配置远程仓库地址
- git push -u [origin] master  首次推送做关联
- git remote show 显示远程仓库别名 （注意local out of date,提示已经过期，远程有新版本）
- git remote show origin 远程仓库详细信息
- git remote rename origin [远程别名]
- git clone [远程地址] [项目名] 克隆远程仓库到本地

- git branch [-av] 查看分支 包括远程 
- git checkout origin/master 此时处于游离分支，此时需要开分支（不要直接修改提交）

## <5> 工具 ##
- gitk  调出gitk
- git gui 调出gui

## <6> git refspec ##

- git config --global alias.[别名] [命令] 为一些命令设置简写别名
- git push --set-upstream origin [分支名] 将本地分支推送的远程 等同git push -u origin [分支名]，远程/本地分支名不同时使用src:dest方式
- 远程分支拉取
- 1）
	- pull只能拉取远程分支索引
	- git checkout -b [分支名] origin/[分支名] 本地创建分支并和远程同步

- 2）
	- git checkout --track origin/[分支名] 创建同名本地分支

- 远程分支删除
	- git push origin :[远程分支名] 推送空本地分支到远程
	- git push origin --delete [远程分支名] 删除远程分支
- 远程标签
	- git tag -a [标签] -m [说明] 本地打标签
	- 	git show [标签] 查看标签
	- 	git tag -l ”查询“
	- 	git push origin [标签名] [多个标签]
	- 	git push origin --tags 推送本地的所有标签

## <7> submodule ##
裸库 没有工作区的仓库 作为远程仓库
git init --bare 创建裸库

submodule 项目互相依赖的情况
1）原始的方式是jar包依赖或手工拷贝，不适用与公共库频繁修改的情况
2）多仓库的依赖方式，类似源码依赖

步骤 现存2个库 parent child
1） 在parent中
    git submodule add [child的远程地址] [放置child的目录（module），不能存在]
	这样将child的代码拉入到目录中
2）推送到远程
    git push -am ''
3）进入到module目录
    git pull 获取依赖的更新
3）或者在根目录
    git submodule foreach git pull 拉取所有依赖的更新
4）更新后再提交
   git push
   
5）此时clone后，submodule是空的
   git clone [远程parent地址] [项目名]
   git submodule init     子模块初始化
   git submodule update --recursive  子模块更新
   git checkout master    子模块切换到主分支
5）或者执行
   git clone [远程parent地址] [项目名] --recursive
   

- 移除submodule   
   - 1）缓存区移除   
       - git rm --cached [子模块目录]  
   - 2）工作区删除    
       - rm -rf [子模块目录]
       - 删除.gitmodule目录   
   - 3）提交   
       - commit
       - push
   
## <8> git subtree  ##
提升submodule的功能，针对在父工程中修改子工程的代码，submodule在这种情况下会出现问题

- 创建步骤 现存2个库 parent child
    - 1）给子模块远程库设置别名
         - git remote add subtree-origin [子模块远程地址]
    - 2）添加子模块远程库到父库,clone子模块到subtree （三种方式 --prefix=subtree , --prefix subtree, -P subtree ）
         - git subtree add --prefix=subtree [远程地址subtree-origin] [分支名master] [--squash 去除commit日志多提交合并一个提交，如果使用，之后pull/push全部都使用]
         - [防止主仓库污染,慎用会影响三方合并]
	- 3）提交
   		- git push
	- 4）子模块更新
   		- git subtree pull --prefix=subtree subtree-origin master 
  
- 在父工程中改子模块的东西
	- 1）普通的push 是修改parent的项目中的内容
	- 2）更新子模块的内容
   		- git subtree push --prefix=subtree subtree-origin master 

## <9> cherry-pick 使用在本地上 ##
将某分支的修改应用到另一个分支，应用场景是开发过程中切换错了分支，并且提交

切换到正确的分支
git cherry-pick commit_id 
切换到错误修改分支
reset或checkout 还原
  可以checkout 某个提交掉创建分支，然后合并

## <10> rebase 变基 衍合 ##
与merge类似
merge会产生新的提交，指向合并的两个分支
rebase会把另分支的提交应用到当前分支，修改提交历史，不要在master上使用rebase,而且一般都是本地分支，尚未推送到远程
注意：不要在共享的分支上操作

git rebase [另一分支]

冲突： 
解决冲突后继续rebase
       git rebase -- continue
恢复到rebase前的状态
       git rebase -- abort
冲突时忽略掉本分支的修改，以另一个分支的变更为准，每次忽略一次本分支的冲突
	   git rebase --skip
	   



