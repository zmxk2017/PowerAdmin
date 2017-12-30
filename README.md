# PowerAdmin

这是一套Microex正在采用的运维相关shell脚本

## 站点承载机通用脚本

### 初始化脚本

```powershell
# 安装iis承载核心
Add-WindowsFeature web-server
# 导入iis相关命令及iis虚拟驱动器
Import-Module WebAdministration
# 我想大家应该不需要Default Web Site
Remove-Website -Name "Default Web Site"
# 允许网络访问并安装choco
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString
('https://chocolatey.org/install.ps1'))
# choco静默安装
choco feature enable -n allowGlobalConfirmation
# choco安装.net core运行时
choco install dotnetcore-windowshosting
# choco安装webdeploy
choco install webdeploy

# 以下部分脚本可复用

# 导入iis相关命令及iis虚拟驱动器
Import-Module WebAdministration
# 新建web站点"xxxxxx",虚拟路径为"yyyyyy",使用前需确保路径存在: mkdir yyyyyy
New-Website -Name xxxxx -PhysicalPath yyyyyy
# 新建对"xxxxxx"站点的绑定,绑定域名为"yyy.com",可通过[-Port <UInt32> ]进一步指定端口号
New-WebBinding -Name xxxxxx -HostHeader yyy.com
# 之后就可以直接用web deploy进行发布了
```

### 短期运维脚本

```powershell
# 跳转iis驱动器以进行运维
cd IIS:
```

## 数据库承载机通用脚本

### 安装sql express**收费版本sqlserver安装时需要额外键入license信息,这里不涉及,不处理**
```powershell
# 安装包下载地址
$InstallerUrl = 'https://download.microsoft.com/download/6/4/A/64A05A0F-AB28-4583-BD7F-139D0495E473/SQLEXPR_x64_CHS.exe'
# 下载临时目录
$Destination = "$env:temp\SQLEXPR_x64_CHS.exe"
# 执行下载
Invoke-WebRequest -Uri $InstallerUrl -OutFile $Destination -UseBasicParsing
# 执行安装 /q:静默 /action=Install:操作为安装 /FEATURES=SQLEngine:安装数据库引擎 /INSTANCENAME=MSSQLSERVER:数据库实例名称为'MSSQLSERVER' /TCPENABLED=1:允许tcp远程连接 /IACCEPTSQLSERVERLICENSETERMS:同意安装前须知 /SECURITYMODE=SQL:安全认证模式为'sql用户认证' /SAPWD=xxxxxx:sa用户密码为'xxxxxx'
&$Destination /q /ACTION=Install /FEATURES=SQLEngine /INSTANCENAME=MSSQLSERVER /TCPENABLED=1 /IACCEPTSQLSERVERLICENSETERMS  /SECURITYMODE=SQL /SAPWD=xxxxxx
```