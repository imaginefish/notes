# 禁止Windows用户下载服务器数据
## 禁止服务器远程桌面禁止拷贝和复制
Windows服务器一般通过远程桌面连接登录，所以需要禁止服务器远程桌面禁止拷贝和复制，防止服务器数据拷贝至本地。
1. win+r打开运行窗口，输入gpedti.msc进入组策略编辑器
2. 进入目录`计算机配置—>管理模板—>windows组件—>远程桌面服务—>远程桌面会话主机`
3. 找到`设备和资源重定向`，开启以下策略：
  - 不允许剪贴板重定向
  - 不允许COM端口重定向
  - 不允许驱动器重定向
  - 不允许LPT端口重定向
4. 找到`打印机重定向`，开启以下策略：
  - 不允许客服端打印机重定向

## 创建普通用户
进入`计算机管理`，找到本地用户和组，新建普通用户，配置其所属组为`Remote Desktop Users`和`Users`，赋予其远程登录和运行大部分软件的权限。
## 搭建配置sftp服务器
检查服务器是否自带ssh服务，最新的Windows 10 系统自带，Windows Server 2016及以下需要用户自己配置。
1. 下载安装Win32 OpenSSH，在Windows 10的早期版本和Windows Server 2016/2012 R2中，必须从GitHub（https://github.com/PowerShell/Win32-OpenSSH/releases）下载并安装OpenSSH 。
2. 解压后将目标目录路径添加至Path环境变量中。
3. 安装OpenSSH服务器，进入PowerShell，切换至解压路径，执行以下命令：
```PowerShell
.\install-sshd.ps1
```
4. 为SSHD服务启用自动启动，然后使用以下PowerShell服务管理命令启动它：
```PowerShell
Set-Service -Name sshd -StartupType ‘Automatic’
Start-Service sshd
```
5. 配置sftp禁止下载
找到ssh配置文件sshd_config，定位到以下内容位置：
```
Subsystem	sftp	sftp-server.exe
```
修改为：
```
Subsystem	sftp	sftp-server.exe -P read
```
`-P` 表示黑名单请求（blacklisted requests），`-P read`即表示禁止读操作。
## 配置防火墙
### 配置公网ip防火墙
防火墙下行规则开放22端口（ssh/sfpt）、3389（Windows远程桌面），上行规则的所有端口全部禁止，防止数据出网。
### 配置服务器本地防火墙
1. 开启Windows防火墙，不勾选`阻止所有传入连接，包括位于允许应用列表中的应用`。
2. 进入高级安全Windows防火墙，创建相应协议和端口的出站规则，启动规则。
