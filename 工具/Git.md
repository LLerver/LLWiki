# Git

## Git命令

### Git拉取代码

```shell
# 拉取master分支
$ git clone master分支git地址
# 拉取对应分支代码
$ git clone -b 分支全名 master分支git地址
```

### GitHub创建新分支

```shell

```

### 查看当前分支

```shell
$ git branch
* dayStudy
  main
```

### 切换当前分支

```shell
$ git checkout main/dayStudy
Switched to branch 'main'
Your branch is up to date with 'origin/main'.

```

### Git提交代码

```shell
# 先查询文件状态,红色的基本都是新增的为主
uang:LLWiki maguagua$ git status
On branch dayStudy
Your branch is up to date with 'origin/dayStudy'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   "\345\267\245\345\205\267/Git.md"

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	"java/\346\241\206\346\236\266/kafka.md"
	"java/\346\241\206\346\236\266/redis\344\270\262\350\256\262.md"
	"java/\346\241\206\346\236\266/spring\345\220\257\345\212\250IOC\350\277\207\347\250\213.md"
	"java/\346\241\206\346\236\266/\344\270\200.Docker\346\246\202\350\277\260.md"
	"java/\346\241\206\346\236\266/\346\225\260\346\215\256\345\272\223\350\260\203\344\274\230\344\270\262\350\256\262.md"
	
# 将需要提交的文件提交到本地仓库缓存中
$ git add .

# 写提交备注
$ git commit -m "文件提交"

# push到远程仓库
huang:LLWiki maguagua$ git commit -m "同步mac文件"
[dayStudy 8667053] 同步mac文件
 6 files changed, 1916 insertions(+), 1 deletion(-)
 create mode 100644 "java/\346\241\206\346\236\266/kafka.md"
 create mode 100644 "java/\346\241\206\346\236\266/redis\344\270\262\350\256\262.md"
 create mode 100644 "java/\346\241\206\346\236\266/spring\345\220\257\345\212\250IOC\350\277\207\347\250\213.md"
 create mode 100644 "java/\346\241\206\346\236\266/\344\270\200.Docker\346\246\202\350\277\260.md"
 create mode 100644 "java/\346\241\206\346\236\266/\346\225\260\346\215\256\345\272\223\350\260\203\344\274\230\344\270\262\350\256\262.md"
huang:LLWiki maguagua$ git push
Username for 'https://github.com': liangliewei1@163.com
Password for 'https://liangliewei1@163.com@github.com':
Enumerating objects: 16, done.
Counting objects: 100% (16/16), done.
Delta compression using up to 4 threads
Compressing objects: 100% (10/10), done.
Writing objects: 100% (11/11), 36.19 KiB | 5.17 MiB/s, done.
Total 11 (delta 2), reused 0 (delta 0)
remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
To https://github.com/LLerver/LLWiki.git
   b619847..8667053  dayStudy -> dayStudy
```

Git分支代码合并

```shell
# A,B两个分支,想将A分支的代码合并到B分支上.先将当前分支切换到B分支上,然后进行git rebase A即可
# 现在将dayStudy分支的文件,合并到master分支上
$ git checkout dayStudy

# 检查是否当前所在分支
huang:LLWiki maguagua$ git branch
* dayStudy
  main
# 进行rebase操作

huang:LLWiki maguagua$ git rebase main
error: cannot rebase: You have unstaged changes.
error: Please commit or stash them.

# 嘤嘤嘤 报错了?检查一下本地是否有修改但未提交的代码哟
huang:LLWiki maguagua$ git status
On branch dayStudy
Your branch is up to date with 'origin/dayStudy'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   "\345\267\245\345\205\267/Git.md"

no changes added to commit (use "git add" and/or "git commit -a")

# 果然有修改但是没有提交的代码,怎么办呢.要么直接提交相关文件到远程仓库,要么进行stash操作,先进行提交操作吧
$ git add -A
$ git commit -m "测试rebase的提交"
$ git push

# 终于合并成功了
huang:LLWiki maguagua$ git rebase main
First, rewinding head to replay your work on top of it...
Applying: 同步mac文件
Applying: 二次同步mac端文件内容
Applying: 测试rebase的提交操作
Applying: tmp提交

# 这就完事了?这只是将dayStudy的代码合并到master分支上了,但是这都是本地操作,远程仓库的代码并没有变化,所以需要将本地的master分支代码push到远程仓库中
$ git checkout main
error: Your local changes to the following files would be overwritten by checkout:
	工具/Git.md
Please commit your changes or stash them before you switch branches.
Aborting

# 本地还是有修改的文件,所以还是需要先进行提交,才能进行切换分支操作
# 一顿add commit push操作
huang:LLWiki maguagua$ git push
To https://github.com/LLerver/LLWiki.git
 ! [rejected]        dayStudy -> dayStudy (non-fast-forward)
error: failed to push some refs to 'https://github.com/LLerver/LLWiki.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.

# 哦豁~又报错了.妙啊.就是说远程仓库与本地仓库不一致,需要先进行
```

### Git缓存区stash操作







