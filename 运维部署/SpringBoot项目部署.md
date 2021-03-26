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

使用[AlwaysUp](https://www.coretechnologies.com/products/AlwaysUp/)软件实现。

参考引用：[https://blog.csdn.net/ly690226302/article/details/79260875](https://blog.csdn.net/ly690226302/article/details/79260875)