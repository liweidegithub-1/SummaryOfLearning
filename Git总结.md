# Git总结

## 一、创建或克隆仓库

### 1.1、初始化新的仓库

```shell
① 首先创建一个空的文件夹
mkdir gitrepo
② 执行git初始化命令（执行命令之后会在文件夹中生成一个.git的目录）
git init
③ 查看git状态
git status
```

### 1.2、克隆一个远程仓库

```shell
执行git克隆命令，会在当前操作路径下生成一个与远程仓库同名的文件夹，这个文件夹就作为本地仓库（与1.1中创建的文件夹同理）
git clone git@github.com:michaelliao/gitskills.git
```

## 二、把文件添加到版本库并提交到远程仓库

### 2.1、将文件添加到本地仓库

```shell
① 首先创建需要提交到仓库的编辑文件（以a.txt为例）
touch a.txt
② 将文件从工作区添加至暂存区（可多次添加或一次多添加）
git add a.txt（git add a.txt b.txt）
③ 将暂存区的文件提交到本地仓库
git commit -m "为本次提交内容添加注释"
```

### 2.2、将本地仓库提交至远程仓库

```shell
① 将本地仓库与远程仓库绑定
git remote add origin git@github.com:michaelliao/learngit.git
解释：origin是给远程仓库自定义的名字（自己想怎么起就怎么起，不过见名知义最好），git@github.com:michaelliao/learngit.git为远程仓库的ssh地址
② 查看远程仓库绑定信息
git remote -v
③ 将本地修改内容提交至远程仓库
git push origin master:master
解释：origin是第一步中我们给远程仓库起的别名，master:master中第一个master是我们本地分支的名，第二个master是远程仓库分支的名。这条命令的意思就是将我们本地仓库的master分支提交至远程仓库的master分支
如果本地分支名和远程仓库分支名一样，那么就可以简写
git push origin master
```

## 三、分支管理

### 3.1、创建分支

```shell
① 创建一个dev分支
git branch dev
② 创建并切换到dev分支
方式一：git checkout -b dev
方式二：git switch -c dev
③ 分支改名（将分支dev修改为dev1）
git branch -m dev dev1
```

### 3.2、切换分支

```shell
方式一：git checkout dev
方式二：git switch dev
```

### 3.3、查看分支

```shell
① 查看本地分支
git branch
② 查看远程分支
git branch -r
③ 查看本地和远程
git branch -a
④ 查看已经合并到当前分支的分支
git branch --merged
⑤ 查看未合并到当前分支的分支
git branch --no-merged
```

### 3.4、合并分支

```shell
① 切换至主分支（假设有两条分支，master和dev，现在想把dev分支的内容合并至master分支，那么master就作合并的主分支）
git switch master
② 合并dev分支
git merge dev
```

### 3.5、删除分支

```shell
① 删除本地已合并分支
git branch -d dev
② 强制删除本地未合并分支
git branch -D dev
③ 删除远程分支
git push origin --delete dev
```

## 四、解决冲突与版本回退

### 4.1、本地合并分支产生冲突

```shell
① 执行合并分支操作
git merge dev
② git会合并两个分支中相同的文件，如有内容冲突则git会返回合并失败的错误，并且会返回合并错误文件；查看错误文件（以a.txt为例）
cat a.txt
$ cat a.txt
<<<<<<< HEAD
hello
hi
=======
my name is mary
>>>>>>> dev
解释：以“=======”作为分割符，上面是主分支的内容，下面是dev的内容
③ git会为展示出文件的冲突内容，我们需要手动修改冲突文件，然后重新执行提交流程
```

### 4.2、提交至远程仓库产生冲突

```shell
① 首先拉去远程最新版本
	git pull是git fetch（拉取远程仓库内容）和git merge（合并）的组合（拉取并合并）
	语法：git pull <远程仓库别名> <远程分支名:本地分支名>
	现在我们本地只添加了一个远程仓库，因此仓库别名可省略
git pull（将远程所有分支内容与本地仓库进行对比）
② 如果内容有冲突，则git会和本地合并一样打为我们打印出冲突提示，同样，我们需要手动修改这些冲突内容，然后重新执行提交流程
```

### 4.3、如果远程仓库与本地仓库的分支信息不一致（他有我无或他无我有）

```shell
需要将本地分支与远程分支进行链接
git branch --set-upstream-to=origin/dev dev
```

### 4.4、对比版本

```shell
① 对比工作区与暂存区的差异
git diff <filename>
② 对比工作区与本地版本库的差异
git diff HEAD <filename>
③ 对比暂存区与本地版本库的差异
git diff --cached <filename>
```

### 4.5、版本回退

```shell
① 查看我们操作过的命令
git log
显示简要的操作信息
git log --pretty=oneline
② 撤销工作区的修改
git checkout -- <filename>
解释：如果文件自修改后还没被放到暂存区（修改后还没有git add），现在撤销修改，回到和版本库一致（上次git commit的状态）;如果文件已经添加到了暂存区（git add），之后又对工作区的文件进行了修改，现在撤销修改，回到和暂存区一致。总之就是这个文件的内容会回到最近一次git add或git commit的状态。
③ 撤销暂存区的修改（使暂存区回到本地版本库最新一次的状态）
git reset HEAD <filename>
④ 回退本地版本库版本（撤销本地仓库的修改）
首先使用<git log --pretty=oneline>查看历史信息，每次提交的记录前都会有版本号
git reset --mixed 版本号
解释：撤销提交和暂存区的更改，保留工作区修改（版本库和暂存区回退至上个版本）
git reset --soft 版本号
解释：撤销提交，保留暂存区和工作区的修改（版本库回退至上个版本）
git reset --hard 版本号
解释：撤销提交和暂存区的更改，并且删除工作区修改（版本库、暂存区和工作区都回退至上个版本）
```


