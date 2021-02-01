# Mac软件

## maven的安装

```shell
# 配置环境变量
# 编辑bash_profile文件
vim ~/.bash_profile
# 配置maven文件地址
export M2_HOME=/Users/xxx/Documents/maven/apache-maven-3.6.1
export PATH=$PATH:$M2_HOME/bin
# 保存文件，执行如下命令使配置生效
source ~/.bash_profile
# 验证
mvn -v
```

## Mac升级node.js和npm

```shell
# 第一步，先查看本机node.js版本：
node -v

# 第二步，清除node.js的cache：
sudo npm cache clean -f

# 第三步，安装 n 工具，这个工具是专门用来管理node.js版本的，别怀疑这个工具的名字，是他是他就是他，他的名字就是 "n"
sudo npm install -g n

# 第四步，安装最新版本的node.js
sudo n stable

# 第五步，再次查看本机的node.js版本：
node -v

# 第六步，更新npm到最新版：
$ sudo npm install npm@latest -g

# 第七步，验证
node -v
npm -v
```

### Mac安装yarn

```shell
sudo -s
# 输入密码
npm i -g yarn

npm WARN engine yarn@1.22.10: wanted: {"node":">=4.0.0"} (current: {"node":"v0.10.29","npm":"1.4.14"})
 
> yarn@1.22.10 preinstall /usr/local/lib/node_modules/yarn
> :; (node ./preinstall.js > /dev/null 2>&1 || true)

/usr/local/bin/yarn -> /usr/local/lib/node_modules/yarn/bin/yarn.js
/usr/local/bin/yarnpkg -> /usr/local/lib/node_modules/yarn/bin/yarn.js
yarn@1.22.10 /usr/local/lib/node_modules/yarn
# 安装成功
```



## Mac安装GitKraken

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
node dist/bin/gitcracken.js patcher --asar /Applications/GitKraken.app/Contents/Resources/app.asar
```

参考：

[插件github](https://github.com/KillWolfVlad/GitKraken-AUR)

[破解插件](https://github.com/5cr1pt/GitCracken/tree/master/GitCracken)

[才发现 gitkraken 现在要给钱才能打开私有库了](https://www.v2ex.com/t/645112)



