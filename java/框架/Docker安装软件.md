# Docker安装软件

## Docker安装MySQL5.7

```powershell
# 查询当前docker容器是否存在MySQL进程
[root@iZ2zej1nogjvovnfeobqe8Z /]# ps -ef|grep mysql
root     21590 10559  0 17:49 pts/0    00:00:00 grep --color=auto mysql
# 查询当前docker容器是否存在MySQL镜像文件
[root@iZ2zej1nogjvovnfeobqe8Z /]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
hello-world   latest    bf756fb1ae65   13 months ago   13.3kB
# 查询docker hub商店中的MySQL
[root@iZ2zej1nogjvovnfeobqe8Z /]# docker search mysql
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   10425     [OK]       
mariadb                           MariaDB is a community-developed fork of MyS…   3868      [OK]       
mysql/mysql-server                Optimized MySQL Server Docker images. Create…   763                  [OK]
# 开始拉取MySQL5.7镜像到docker容器中
[root@iZ2zej1nogjvovnfeobqe8Z /]# docker pull mysql:5.7
5.7: Pulling from library/mysql
a076a628af6f: Pull complete 
f6c208f3f991: Pull complete 
88a9455a9165: Pull complete 
406c9b8427c6: Pull complete 
7c88599c0b25: Pull complete 
25b5c6debdaf: Pull complete 
43a5816f1617: Pull complete 
1831ac1245f4: Pull complete 
37677b8c1f79: Pull complete 
27e4ac3b0f6e: Pull complete 
7227baa8c445: Pull complete 
Digest: sha256:b3d1eff023f698cd433695c9506171f0d08a8f92a0c8063c1a4d9db9a55808df
Status: Downloaded newer image for mysql:5.7
docker.io/library/mysql:5.7
# 检查一下是否拉取MySQL镜像成功
[root@iZ2zej1nogjvovnfeobqe8Z /]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
mysql         5.7       a70d36bc331a   8 days ago      449MB
hello-world   latest    bf756fb1ae65   13 months ago   13.3kB
# 创建mysql安装目录
[root@iZ2zej1nogjvovnfeobqe8Z docker_app]# mkdir docker_mysql5.7
[root@iZ2zej1nogjvovnfeobqe8Z docker_app]pwd
/opt/docker_app/docker_mysql5.7
# 开始配置环境变量PWD
[root@iZ2zej1nogjvovnfeobqe8Z docker_mysql5.7]# echo $PWD
/opt/docker_app/docker_mysql5.7
# 查看MySQL镜像的下载地址
[root@iZ2zej1nogjvovnfeobqe8Z docker_mysql5.7]# cd /var/lib/docker/containers/
[root@iZ2zej1nogjvovnfeobqe8Z containers]# ll
total 4
drwx------ 4 root root 4096 Jan 15 17:00 f723497ae95cedc3b4a2e1611109450918d80812a3a9a2a974d81ffb31a62a90
# 运行MySQL容器
[root@iZ2zej1nogjvovnfeobqe8Z containers]# docker run -d -p 3306:3306 --privileged=true -v $PWD/conf/my.cnf:/etc/my.cnf -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql mysql:5.7 --character-set-server=utf8mb4 --collation-server=utf8mb4_general_ci
# 控制台会展示一个新的镜像文件名称
e55f902a094dc0eacf76774306c4204976ecc7a0b75fe979bc1d46204feab9ab
# 再次查看/var/lib/docker/containers/下的镜像文件情况
[root@iZ2zej1nogjvovnfeobqe8Z containers]# cd /var/lib/docker/containers/
[root@iZ2zej1nogjvovnfeobqe8Z containers]# ll
total 20
drwxr-xr-x 3 root    root 4096 Jan 27 18:37 conf
drwxr-xr-x 5 polkitd root 4096 Jan 27 18:37 data
drwx------ 4 root    root 4096 Jan 27 18:37 e55f902a094dc0eacf76774306c4204976ecc7a0b75fe979bc1d46204feab9ab
drwx------ 4 root    root 4096 Jan 15 17:00 f723497ae95cedc3b4a2e1611109450918d80812a3a9a2a974d81ffb31a62a90
drwxr-xr-x 2 root    root 4096 Jan 27 18:37 logs
# 会发现执行完docker run 命令之后,控制台日志会显示的新的镜像名称与docker下载镜像目录下的镜像名称一致.同时原来最初从hub上面下载的镜像文件也还存在.
# 查看MySQL进程
[root@iZ2zej1nogjvovnfeobqe8Z containers]# docker ps -a
CONTAINER ID   IMAGE         COMMAND                  CREATED         STATUS                   PORTS                               NAMES
e55f902a094d   mysql:5.7     "docker-entrypoint.s…"   4 minutes ago   Up 4 minutes             0.0.0.0:3306->3306/tcp, 33060/tcp   mysql
f723497ae95c   hello-world   "/hello"                 12 days ago     Exited (0) 12 days ago                                       cool_hofstadter
# 进入到MySQL容器内
[root@iZ2zej1nogjvovnfeobqe8Z containers]# docker exec -it mysql bash
# 登录MySQL服务器
root@e55f902a094d:/# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.33 MySQL Community Server (GPL)

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql> 
# 开启远程访问权限
mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

# 查询用户
mysql> select host,user from user;
+-----------+---------------+
| host      | user          |
+-----------+---------------+
| %         | root          |
| localhost | mysql.session |
| localhost | mysql.sys     |
| localhost | root          |
+-----------+---------------+
4 rows in set (0.00 sec)

# 设置用户密码 镜像里面 root用户已经有远程连接权限在里面，所以不需要去设置，只是模式不一样才导致无法连接，把root用户的密码改成 mysql_native_password 模式，即可远程连接
mysql> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
Query OK, 0 rows affected (0.01 sec)

# 刷新
mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
# 使用navicat根据宿主机IP以及暴露端口进行远程访问即可

[root@iZ2zej1nogjvovnfeobqe8Z ~]# systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: man:firewalld(1)
# 
[root@iZ2zej1nogjvovnfeobqe8Z ~]# systemctl mask firewalld
Created symlink from /etc/systemd/system/firewalld.service to /dev/null.
# 
[root@iZ2zej1nogjvovnfeobqe8Z ~]# systemctl disable firewalld
[root@iZ2zej1nogjvovnfeobqe8Z ~]# systemctl status firewalld
● firewalld.service
   Loaded: masked (/dev/null; bad)
   Active: inactive (dead)

```



## Docker安装JDK8

```shell
# 安装JDK8
yum install java-1.8.0-openjdk* -y

# 检查Java环境
[root@iZ2zej1nogjvovnfeobqe8Z overlay2]# java -version
openjdk version "1.8.0_282"
OpenJDK Runtime Environment (build 1.8.0_282-b08)
OpenJDK 64-Bit Server VM (build 25.282-b08, mixed mode)

# jdk自动安装目录为：/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.212.b04-0.el7_6.x86_64
# 配置环境变量：

# 在 /etc/profile 文件中加入如下内容：
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.212.b04-0.el7_6.x86_64
export JRE_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.212.b04-0.el7_6.x86_64
export CLASSPATH=.:JAVAHOME/jre/lib:JAVA_HOME/lib
export PATH=PATH:JAVA_HOME/bin

# 执行下面命令使设置生效
[root@iZ2zej1nogjvovnfeobqe8Z etc]# source /etc/profile

# 创建dockerFile文件

```

