# BloodHound & 其他 AD 枚举工具

<details>

<summary><strong>从零开始学习 AWS 黑客技术，成为专家</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS 红队专家）</strong></a><strong>！</strong></summary>

* 您在 **网络安全公司** 工作吗？ 想要看到您的 **公司在 HackTricks 中被宣传** 吗？ 或者想要访问 **PEASS 的最新版本或下载 HackTricks 的 PDF** 吗？ 请查看 [**订阅计划**](https://github.com/sponsors/carlospolop)!
* 发现 [**PEASS 家族**](https://opensea.io/collection/the-peass-family)，我们的独家 [**NFTs**](https://opensea.io/collection/the-peass-family) 收藏品
* 获取 [**官方 PEASS & HackTricks 商品**](https://peass.creator-spring.com)
* **加入** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**电报群组**](https://t.me/peass) 或 **关注** 我的 **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**。**
* **通过向 [hacktricks 仓库](https://github.com/carlospolop/hacktricks) 和 [hacktricks-cloud 仓库](https://github.com/carlospolop/hacktricks-cloud) 提交 PR 来分享您的黑客技巧**。

</details>

## AD Explorer

[AD Explorer](https://docs.microsoft.com/en-us/sysinternals/downloads/adexplorer) 来自 Sysinternal Suite：

> 一个高级的 Active Directory (AD) 查看器和编辑器。 您可以使用 AD Explorer 轻松浏览 AD 数据库，定义喜爱的位置，查看对象属性和属性而无需打开对话框，编辑权限，查看对象的模式，并执行可以保存和重新执行的复杂搜索。

### 快照

AD Explorer 可以创建 AD 的快照，以便您可以离线检查它。\
它可用于离线发现漏洞，或比较 AD 数据库在不同时间点的不同状态。

您将需要用户名、密码和连接方向（需要任何 AD 用户）。

要对 AD 进行快照，转到 `文件` --> `创建快照` 并输入快照的名称。

## ADRecon

[**ADRecon**](https://github.com/adrecon/ADRecon) 是一个从 AD 环境中提取和组合各种工件的工具。 信息可以呈现在一个 **特别格式化** 的 Microsoft Excel **报告** 中，其中包括摘要视图和指标，以便于分析并提供目标 AD 环境当前状态的整体图片。
```bash
# Run it
.\ADRecon.ps1
```
## BloodHound

From [https://github.com/BloodHoundAD/BloodHound](https://github.com/BloodHoundAD/BloodHound)

> BloodHound是一个单页Javascript Web应用程序，构建在[Linkurious](http://linkurio.us/)之上，使用[Electron](http://electron.atom.io/)编译，使用由C#数据收集器提供数据的[Neo4j](https://neo4j.com/)数据库。

BloodHound使用图论来揭示Active Directory或Azure环境中隐藏且通常是意外的关系。攻击者可以使用BloodHound轻松识别高度复杂的攻击路径，否则将无法快速识别。防御者可以使用BloodHound识别并消除相同的攻击路径。蓝队和红队都可以使用BloodHound轻松获得对Active Directory或Azure环境中特权关系的更深入了解。

因此，[Bloodhound](https://github.com/BloodHoundAD/BloodHound)是一个令人惊叹的工具，可以自动枚举域，保存所有信息，找到可能的特权升级路径，并使用图形显示所有信息。

Booldhound由2个主要部分组成：**摄取器**和**可视化应用程序**。

**摄取器**用于**枚举域并提取所有信息**，以一种可被可视化应用程序理解的格式。

**可视化应用程序使用neo4j**来展示所有信息之间的关系，并展示在域中升级特权的不同方式。

### 安装
在创建BloodHound CE之后，整个项目已更新以便与Docker更轻松地使用。开始的最简单方法是使用其预配置的Docker Compose配置。

1. 安装Docker Compose。这应该包含在[Docker Desktop](https://www.docker.com/products/docker-desktop/)安装中。
2. 运行：
```
curl -L https://ghst.ly/getbhce | docker compose -f - up
```
3. 在Docker Compose的终端输出中找到随机生成的密码。
4. 在浏览器中，导航至 http://localhost:8080/ui/login。使用用户名admin和日志中随机生成的密码登录。

完成后，您需要更改随机生成的密码，然后您将准备好新界面，可以直接下载摄取器。

### SharpHound

它们有几个选项，但如果您想要从加入域的个人电脑上运行SharpHound，使用您当前的用户并提取所有信息，您可以执行：
```
./SharpHound.exe --CollectionMethods All
Invoke-BloodHound -CollectionMethod All
```
> 您可以在[此处](https://support.bloodhoundenterprise.io/hc/en-us/articles/17481375424795-All-SharpHound-Community-Edition-Flags-Explained)详细了解**CollectionMethod**和循环会话。

如果您希望使用不同的凭据执行SharpHound，可以创建一个CMD netonly会话，并从那里运行SharpHound：
```
runas /netonly /user:domain\user "powershell.exe -exec bypass"
```
[**在 ired.team 了解更多关于 Bloodhound 的信息。**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-with-bloodhound-on-kali-linux)


## Group3r

[**Group3r**](https://github.com/Group3r/Group3r) 是一个用于查找 Active Directory 关联的 **Group Policy** 中的 **漏洞** 的工具。\
您需要使用 **任何域用户** 从域内主机上 **运行 group3r**。
```bash
group3r.exe -f <filepath-name.log>
# -s sends results to stdin
# -f send results to file
```
## PingCastle

[**PingCastle**](https://www.pingcastle.com/documentation/) **评估AD环境的安全状况**，并提供带有图表的**报告**。

要运行它，可以执行二进制文件 `PingCastle.exe`，它将启动一个**交互式会话**，呈现选项菜单。要使用的默认选项是**`healthcheck`**，它将建立**域**的基线**概述**，并查找**配置错误**和**漏洞**。&#x20;
