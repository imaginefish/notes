# Ubuntu 实现多用户同时登录远程桌面
> Xrdp 是 Microsoft 远程桌面协议（RDP）的开源实现，可让您以图形方式控制远程系统。 使用 RDP，您可以登录到远程计算机并创建真实的桌面会话，就像登录到本地计算机一样。

本教程介绍如何在 Ubuntu 20.04 上安装和配置 Xrdp 服务器，实现多用户同时登录远程桌面。

## 脚本安装
通过[http://www.c-nergy.be/products.html](http://www.c-nergy.be/products.html)下载对应脚本，一键快捷安装。
## 手动安装
### 安装桌面环境
Ubuntu 服务器是从命令行管理的，默认情况下未安装桌面环境。 如果您运行桌面版的 Ubuntu，请跳过此步骤。

您可以选择 Ubuntu 存储库中的各种桌面环境。 一种选择是安装 Gnome，这是 Ubuntu 20.04 中的默认桌面环境。 另一种选择是安装 Xfce 。 它是一种快速，稳定且轻巧的桌面环境，非常适合在远程服务器上使用。

运行以下命令之一来安装您选择的桌面环境。

安装 Gnome：
```shell
sudo apt update
sudo apt install ubuntu-desktop
```
安装 Xfce：
```shell
sudo apt update
sudo apt install xubuntu-desktop
```
取决于您的系统，下载和安装 GUI 软件包将需要一些时间。

### 安装 Xrdp
Xrdp 包含在默认的 Ubuntu 存储库中。 要安装它，请运行：
```shell
sudo apt install xrdp
```
安装完成后，Xrdp 服务将自动启动。 您可以输入以下内容进行验证：
```shell
sudo systemctl status xrdp
```
输出将如下所示：
```shell
● xrdp.service - xrdp daemon
     Loaded: loaded (/lib/systemd/system/xrdp.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2020-05-22 17:36:16 UTC; 4min 41s ago
  ...
```
默认情况下，Xrdp 使用 /etc/ssl/private/ssl-cert-snakeoil.key 文件，该文件仅由``ssl-cert``组的成员读取。 运行以下命令以将 xrdp 用户添加到组：

sudo adduser xrdp ssl-cert  
重新启动 Xrdp 服务，以使更改生效：

sudo systemctl restart xrdp
Xrdp 已安装在您的 Ubuntu 服务器上，您可以开始使用它。
### Xrdp配置
Xrdp 配置文件位于 /etc/xrdp 目录中。 对于基本的Xrdp连接，您无需对配置文件进行任何更改。

Xrdp 使用默认的 X Window 桌面环境（Gnome 或 XFCE）。

主要配置文件名为 xrdp.ini 。 该文件分为几个部分，可让您设置全局配置设置（例如安全性和侦听地址）并创建不同的 xrdp 登录会话。

无论何时对配置文件进行任何更改，都需要重新启动 Xrdp 服务。

Xrdp 使用 startwm.sh 文件启动X会话。 如果要使用另一个 X Window 桌面，请编辑此文件。

### 配置防火墙
Xrdp 守护程序在所有接口上的端口3389上进行侦听。 如果您在 Ubuntu 服务器上运行防火墙，则需要打开 Xrdp 端口。

要允许从特定的IP地址或IP范围（例如192.168.33.0/24）访问 Xrdp 服务器，请运行以下命令：
```shell
sudo ufw allow from 192.168.33.0/24 to any port 3389
```
如果要允许从任何地方访问（出于安全考虑，强烈建议不要这样做），请运行：
```shell
sudo ufw allow 3389
```
为了提高安全性，您可以考虑将 Xrdp 设置为仅在 localhost 上侦听，并创建 SSH 隧道，该隧道将流量从本地计算机在端口3389上安全地转发到同一端口上的服务器。

### 连接到 Xrdp 服务器
现在，您已经设置了 Xrdp 服务器，是时候打开 Xrdp 客户端并连接到服务器了。

如果您有 Windows PC，则可以使用默认的 RDP 客户端。 在 Windows 搜索栏中键入``remote``，然后单击``Remote Desktop Connection``。 这将打开 RDP 客户端。 在``计算机``字段中，输入远程服务器的 IP 地址，然后单击``连接``。

在登录屏幕上，输入您的用户名和密码，然后单击``确定``。

登录后，您应该看到默认的 Gnome 或 Xfce 桌面。

您现在可以使用键盘和鼠标从本地计算机开始与远程桌面进行交互。

如果您正在运行 macOS，则可以从 Mac App Store 安装 Microsoft 远程桌面应用程序。 Linux 用户可以使用RDP客户端，例如 Remmina 或 Vinagre。

参考来源：[https://www.myfreax.com/how-to-install-xrdp-on-ubuntu-20-04/](https://www.myfreax.com/how-to-install-xrdp-on-ubuntu-20-04/)