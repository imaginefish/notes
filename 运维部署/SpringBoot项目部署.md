# Spring Boot 项目部署

## Linux 部署方案

> 主流的Linux大多使用init.d或systemd来注册服务。下面在Ubuntu 18.04 演示systemd注册服务。
### 基于Linux的Systemd部署
在/etc/systemd/system/目录下新建文件test.service，填入下面内容：
```
[Unit]
Description=test
After=syslog.target
 
[Service]
ExecStart= /usr/bin/java -jar /root/web/test/test-0.0.1-SNAPSHOT.jar
 
[Install]
WantedBy=multi-user.target
```
**注意：** 在实际使用中修改Description和ExecStart后面的内容。

### 日志清除
```shell
# 冲洗日志数据至/var/log/journal
journalctl --flush
# 轮换日志文件
journalctl --rotate
# 清除日志至数据小于500m
journalctl --vacuum-size=500m
# 清除一小时之前的日志
journalctl --vacuum-time=1h
```

**启动服务**
```shell
systemctl start test
```
**停止服务**

```shell
systemctl stop test
```
**重启服务**
```shell
systemctl restart test
```
**开机自启**
```shell
systemctl enable test
```
**服务状态**
```shell
systemctl status test
```
**项目日志**

```shell
journalctl -u test
```
## Windows 部署方案
### AlwaysUp
使用[AlwaysUp](https://www.coretechnologies.com/products/AlwaysUp/)软件实现。
### 注册为 Windows 系统服务
编写 bat 脚本，编写完成后双击运行进行测试
```cmd
java jar xxx.jar
```
sc 命令注册为系统服务，SC 是用来与服务控制管理器和服务进行通信的命令行程序，注意等号后面跟上一个空格。实测power shell无法执行 sc 命令，需以管理员身份运行 cmd 执行以下命令。
```cmd
sc create 服务名 binPath= "xxx.bat" start= "auto" displayname= "服务显示名称"
```
修改服务描述
```cmd
sc description "服务名" "服务描述"
```
启动服务，启动可能出现”服务没有响应控制功能“报错，但此时服务已经正常启动，可能是 bat 脚本代码不完整导致的（未验证）
```cmd
net start "服务名"
```
停止服务
```cmd
net stop "服务名"
```
删除服务
```cmd
sc delete "服务名"
```

参考来源：

[https://blog.csdn.net/ly690226302/article/details/79260875](https://blog.csdn.net/ly690226302/article/details/79260875)

[https://www.junmajinlong.com/linux/systemd/service_1](https://www.junmajinlong.com/linux/systemd/service_1)

[https://www.cnblogs.com/sparkdev/p/8795141.html](https://www.cnblogs.com/sparkdev/p/8795141.html)