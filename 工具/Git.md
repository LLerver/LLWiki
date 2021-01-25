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

```

### Git缓存区stash操作





## GitKraken

### 1. 下载客户端

- [gitkraken windows版本 6.5.1下载地址-1](https://474b.com/file/21384459-442883642)
- [gitkraken windows版本 6.5.1下载地址-2](https://release.axocdn.com/win64/GitKrakenSetup-6.5.0.exe)
- [gitkraken Mac版本 6.5.1下载地址](https://474b.com/file/21384459-442883514)
- [gitkraken linux deb 6.5.1下载地址](https://474b.com/file/21384459-442883174)

### 2. 屏蔽更新host

```css
# gitKraken 更新屏蔽
127.0.0.1 release.gitkraken.com
```

### 3. 打开gitkraken并登陆

### 4. 下载破解脚本

```shell
git clone https://github.com/5cr1pt/GitCracken.git
cd GitCracken/GitCracken
rm yarn.lock
yarn install
yarn build
# windows gitbash
node dist/bin/gitcracken.js patcher --asar ~/AppData/Local/gitkraken/app-7.5.0/resources/app.asar
# mac 
node dist/bin/gitcracken.js patcher --asar 你的gitkraken的目录/resources/app.asar
```

参考：

[插件github](https://github.com/KillWolfVlad/GitKraken-AUR)

[破解插件](https://github.com/5cr1pt/GitCracken/tree/master/GitCracken)

[才发现 gitkraken 现在要给钱才能打开私有库了](https://www.v2ex.com/t/645112)





node安装的错误信息

![image-20210125165844310](d:\user\01404091\Application Data\Typora\typora-user-images\image-20210125165844310.png)

卸载重新安装