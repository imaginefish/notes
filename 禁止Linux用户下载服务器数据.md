# 禁止Linux用户下载服务器数据

本文用于向用户开放服务器的使用权限，如连接Hadoop集群的一台client节点，但是需要保证服务器上的数据安全，禁止用户下载，允许用户上传和部署作业。本文仅提供用户SSH秘钥登录，SFTP上传文件。

## 创建Linux用户

root或者sudo组用户登录服务器（最好禁用root或sudo用户密码登录，使用SSH私钥登录，保证安全），创建普通用户。

```shell
# 交互式创建用户
adduser user
```

以下提供其他相关命令。

```shell
# 静态创建用户，之后需自行设定密码
useradd user
passwd user
# 删除用户，并删除用户home目录和mail spool
userdel -r user
# 添加组
groupadd group
# 删除组
groupdel group
# 赋予用户sudo权限，末行添加user ALL=(ALL:ALL) ALL
visudo /etc/sudoers
```

## 创建SSH秘钥

配置SSH支持密码登录，以确保ssh-copy-id命令可以执行，因为该命令基于SSH将本地公钥上传至服务器，在尚未能使用秘钥登录前，需通过用户密码登录。完成公钥上传后，再禁止密码登录。

```shell
# 编辑ssh配置文件，找到PasswordAuthentication no，将no修改为yes，公钥上传完后，再改回no
vim /etc/ssh/sshd_config
```

切换至新建用户，并转到home目录，方便操作。

```bash
# 切换用户
su user
# 切换至home目录
cd ~
```

创建SSH秘钥对，并把公钥加入授权文件，私钥交付用户登录。

```shell
# -t 加密类型
ssh-keygen -t rsa
# 修改私钥文件权限，权限过高ssh会拒绝使用该私钥登录
chmod 400 ~/.ssh/id_rsa
# 公钥加入主机（localhost）授权文件，-i 公钥文件路径
ssh-copy-id -i ~/.ssh/id_rsa.pub user@localhost
```

测试秘钥登录

```shell
ssh -i ~/.ssh/id_rsa user@localhost
```

## SFTP

SFTP是SSH的子系统，在服务器端，通过ssh配置文件中Subsystem sftp指定。目前，有两种sftp可选，一种是internal-sftp，一种是位于/usr/lib下的sftp-server，该文件实际为一个软链，真实路径为/usr/lib/openssh/sftp-server。即ssh配置文件有以下两种写法：

```
# internal-sftp
Subsystem sftp internal-sftp
# sftp-server
Subsystem sftp /usr/lib/sftp-server
```

参考sftp-server（man sftp-server）手册可知，sftp-server可以添加sftp请求的黑白名单，已达到控制用户操作权限的目的。注意，sftp-server不能直接执行，只能在ssh配置文件中指定命令参数，供ssh调用。以下为例：

```
# 设置sftp服务器为只读模式，任何改变文件系统的操作都将被拒绝，比如修改文件等
Subsystem sftp /usr/lib/sftp-server -R
# 即将read操作加入黑名单，以实现禁止通过sftp下载文件的目的
Subsystem sftp /usr/lib/sftp-server -P read
```
## FTP

用户通过SFTP进行数据上传，不提供FTP服务，不开放相应端口。

## SCP

通过修改scp执行文件名，达到禁止使用的目的。

```shell
# 找到scp路径
whereis scp
# 修改文件名，需切换至root或者sudo用户
mv /usr/bin/scp /usr/bin/scp_
```

## 防火墙

由于SSH远程登录和SFTP上传文件已经满足用户操作需求，为保障安全，防火墙下行规则仅开放22端口，上行规则的所有端口全部禁止。

## Hadoop配置
在hdfs上创建用户作业目录，并修改目录所属为用户，以便用户向集群提交和运行作业。
```shell
hdfs dfs -mkdir -p /user/user
hdfs dfs -chown -R user:user /user/user
```
root用户初始化hive，创建用户数据库，并修改数据库目录所属为用户。
```shell
# 命令行进入并初始化hive
hive
# 创建用户数据库
hive> create database user;
# 修改数据库目录所属为用户
hdfs dfs -chown -R user:user /user/hive/warehouse/user.db
```
若遇spark-sql运行报错。
- 检查${SPARK_HOME}/jars目录下，是否缺失相关依赖包。若缺失，考虑重新编译，在编译时加入相关命令。
- 检查${SPARK_HOME}/conf目录下，是否缺失hive-site.mxl配置文件。若缺失，将hive的该配置文件拷贝至该目录下。
