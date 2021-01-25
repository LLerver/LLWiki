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
# 
```







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