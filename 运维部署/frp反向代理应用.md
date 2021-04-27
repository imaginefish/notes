# frp 反向代理应用
>frp 是一个专注于内网穿透的高性能的反向代理应用，支持 TCP、UDP、HTTP、HTTPS 等多种协议。可以将内网服务以安全、便捷的方式通过具有公网 IP 节点的中转暴露到公网。

## 为什么使用 frp？ 
通过在具有公网 IP 的节点上部署 frp 服务端，可以轻松地将内网服务穿透到公网，同时提供诸多专业的功能特性，这包括：
- 客户端服务端通信支持 TCP、KCP 以及 Websocket 等多种协议。
- 采用 TCP 连接流式复用，在单个连接间承载更多请求，节省连接建立时间。
- 代理组间的负载均衡。
- 端口复用，多个服务通过同一个服务端端口暴露。
- 多个原生支持的客户端插件（静态文件查看，HTTP、SOCK5 代理等），便于独立使用 frp 客户端完成某些工作。
- 高度扩展性的服务端插件系统，方便结合自身需求进行功能扩展。
- 服务端和客户端 UI 页面。


项目 GitHub 地址：[https://github.com/fatedier/frp](https://github.com/fatedier/frp)

项目中文文档地址：[https://gofrp.org/docs/](https://gofrp.org/docs/)
## frp 配置
frp 提供了全平台的编译文件，你可以在 [https://github.com/fatedier/frp/releases](https://github.com/fatedier/frp/releases) 找到最新发行的版本。frp 分为服务端和客户端，下载解压后，文件中的 frps 即为服务端，frpc 则为客户端。服务端是指具有公网 IP 用于反向代理内网服务的服务器端，客户端是指被代理的内网主机端。
### 服务端配置
编辑 `frps.ini` 文件，修改如下：
```
[common]
bind_port = 7000
dashboard_port = 7500 
dashboard_user = admin 
dashboard_pwd = admin 
```
其中：
- bind_port：公网 frp 服务的端口号， 要同客户端配置的相同
- dashboard_port：frp 服务管理页面端口
- dashboard_user：frp 服务管理页面管理员账号名称
- dashboard_pwd：frp 服务管理页面管理员账号密码
### 客户端配置
编辑 `frpc.ini` 文件，修改如下：
```
[common]
server_addr = xx.xx.xx.xx
server_port = 7000

[frp-rdp]
type = tcp
local_ip = 127.0.0.1
local_port = 3389
remote_port = 3388
```
其中：
- server_addr：公网服务器 ip，可填入域名
- server_port：公网 frp 服务的端口号
- type：要装发的服务连接类型，默认为 tcp，也可省略不写
- local_ip：要转发的内网服务 IP，可以填写其他能在内网访问的服务器 IP
- local_port：要转发的内网服务端口，例如 22（SSH）、6006（tensorboard）等
- remote_port：在 frp 服务器上的访问端口，在外网访问时，使用外网 IP + 这个外网端口即可访问内网对应服务，该端口要求在 frp 服务器上未使用
## frp 运行
### 服务端运行
根据需求，若作为调试，直接执行命令 `./frps -c frps.ini` 即可，若需长期后台运行，可以使用 nohup 命令后台运行，或者参考 [Spring Boot 项目部署](SpringBoot项目部署.md) 注册为系统服务运行。
```bash
nohup ./frps -c frps.ini >/dev/null 2>&1 & 
```
### 客户端运行
以 Windows 系统示例，若作为调试，在解压文件夹下 cmd 执行命令 `frpc.exe -c frpc.ini` 即可，注意，双击 frpc.exe 无法成功运行，必须使用 cmd 执行命令，若需长期后台运行，参考 [Spring Boot 项目部署](SpringBoot项目部署.md) 注册为系统服务运行。
## frp 使用
注意服务端配置中的 `bind_port`、`dashboard_port` 和客户端配置中的 `remote_port` 端口，均需要在公网 IP 防火墙中开放。此时，通过 `server_addr` + `remote_port` 便可实现在外网环境下对内网相关服务的访问了。