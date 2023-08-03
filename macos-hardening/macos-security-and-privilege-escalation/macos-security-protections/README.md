# macOS安全保护

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks云 ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 推特 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* 你在一家**网络安全公司**工作吗？你想在HackTricks中看到你的**公司广告**吗？或者你想获得**PEASS的最新版本或下载PDF格式的HackTricks**吗？请查看[**订阅计划**](https://github.com/sponsors/carlospolop)！
* 发现我们的独家[**NFTs**](https://opensea.io/collection/the-peass-family)收藏品[**The PEASS Family**](https://opensea.io/collection/the-peass-family)
* 获取[**官方PEASS和HackTricks周边产品**](https://peass.creator-spring.com)
* **加入**[**💬**](https://emojipedia.org/speech-balloon/) [**Discord群组**](https://discord.gg/hRep4RUj7f)或[**电报群组**](https://t.me/peass)，或者**关注**我在**Twitter**上的[**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**。**
* **通过向**[**hacktricks repo**](https://github.com/carlospolop/hacktricks) **和**[**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **提交PR来分享你的黑客技巧。**

</details>

## Gatekeeper

**Gatekeeper**是为Mac操作系统开发的安全功能，旨在确保用户在其系统上**只运行可信任的软件**。它通过**验证用户从App Store之外的来源下载并尝试打开的软件**（如应用程序、插件或安装程序包）来实现。

Gatekeeper的关键机制在于其**验证**过程。它检查下载的软件是否由**已知开发者签名**，确保软件的真实性。此外，它还确定软件是否经过了**Apple的公证**，以确认其不包含已知的恶意内容，并且在公证后没有被篡改。

此外，Gatekeeper通过**提示用户批准首次打开**下载的软件来加强用户控制和安全性。这个保护措施有助于防止用户无意中运行可能有害的可执行代码，而他们可能将其误认为是无害的数据文件。
```bash
# Check the status
spctl --status
# Enable Gatekeeper
sudo spctl --master-enable
# Disable Gatekeeper
sudo spctl --master-disable
```
### 应用程序签名

应用程序签名，也称为代码签名，是苹果安全基础设施的关键组成部分。它们用于验证软件作者（开发者）的身份，并确保代码自上次签名以来没有被篡改。

以下是其工作原理：

1. **签署应用程序：** 当开发者准备分发他们的应用程序时，他们使用私钥对应用程序进行签名。这个私钥与苹果开发者计划中开发者注册时苹果颁发给他们的证书相关联。签名过程涉及对应用程序的所有部分创建一个加密哈希，并使用开发者的私钥对该哈希进行加密。
2. **分发应用程序：** 签名的应用程序随后与开发者的证书一起分发给用户，该证书包含相应的公钥。
3. **验证应用程序：** 当用户下载并尝试运行应用程序时，他们的Mac操作系统使用开发者证书中的公钥来解密哈希。然后，它根据应用程序的当前状态重新计算哈希，并将其与解密的哈希进行比较。如果它们匹配，这意味着应用程序自开发者签名以来**没有被修改**，系统允许应用程序运行。

应用程序签名是苹果Gatekeeper技术的重要组成部分。当用户尝试**打开从互联网下载的应用程序**时，Gatekeeper会验证应用程序的签名。如果它使用苹果颁发给已知开发者的证书进行签名，并且代码没有被篡改，Gatekeeper允许应用程序运行。否则，它会阻止应用程序并向用户发出警报。

从macOS Catalina开始，Gatekeeper还会检查应用程序是否已被苹果**公证**，增加了额外的安全层。公证过程会检查应用程序是否存在已知的安全问题和恶意代码，如果这些检查通过，苹果会向应用程序添加一个Gatekeeper可以验证的凭证。

#### 检查签名

在检查一些**恶意软件样本**时，您应该始终**检查二进制文件的签名**，因为签名它的**开发者**可能已经与**恶意软件**有关。
```bash
# Get signer
codesign -vv -d /bin/ls 2>&1 | grep -E "Authority|TeamIdentifier"

# Check if the app’s contents have been modified
codesign --verify --verbose /Applications/Safari.app

# Get entitlements from the binary
codesign -d --entitlements :- /System/Applications/Automator.app # Check the TCC perms

# Check if the signature is valid
spctl --assess --verbose /Applications/Safari.app

# Sign a binary
codesign -s <cert-name-keychain> toolsdemo
```
### 验证

苹果的验证流程是为了保护用户免受潜在有害软件的额外保障。它涉及开发者将他们的应用程序提交给苹果的验证服务进行审查，这与应用审核不同。这个服务是一个自动化系统，会对提交的软件进行检查，以查找恶意内容和代码签名可能存在的问题。

如果软件通过了这个检查而没有引起任何关注，验证服务会生成一个验证票据。然后，开发者需要将这个票据附加到他们的软件上，这个过程被称为“装订”。此外，验证票据也会在网上发布，苹果的安全技术Gatekeeper可以访问它。

当用户首次安装或执行软件时，验证票据的存在（无论是附加到可执行文件还是在线找到的）会通知Gatekeeper该软件已由苹果进行了验证。因此，Gatekeeper在初始启动对话框中显示一个描述性消息，指示该软件已经经过苹果的恶意内容检查。这个过程增强了用户对他们在系统上安装或运行的软件的安全信心。

### 隔离文件

在下载应用程序或文件时，特定的 macOS 应用程序（如网络浏览器或电子邮件客户端）会将一个称为“隔离标志”的扩展文件属性附加到下载的文件上。这个属性作为一个安全措施，将文件标记为来自不受信任的来源（互联网），并可能带有风险。然而，并不是所有的应用程序都会附加这个属性，例如，常见的 BitTorrent 客户端软件通常会绕过这个过程。

当用户尝试执行文件时，隔离标志的存在会向 macOS 的 Gatekeeper 安全功能发出信号。

在没有隔离标志的情况下（例如通过某些 BitTorrent 客户端下载的文件），Gatekeeper 的检查可能不会执行。因此，用户在打开从不太安全或未知来源下载的文件时应谨慎。

{% hint style="info" %}
验证代码签名的有效性是一个资源密集型的过程，它包括生成代码及其所有捆绑资源的加密哈希。此外，检查证书的有效性还涉及在线检查苹果的服务器，以查看证书是否在发放后被吊销。出于这些原因，每次启动应用程序时运行完整的代码签名和验证检查是不切实际的。

因此，这些检查仅在执行带有隔离属性的应用程序时运行。
{% endhint %}

{% hint style="warning" %}
请注意，Safari 和其他网络浏览器和应用程序需要标记下载的文件。

此外，由沙箱进程创建的文件也会附加此属性，以防止沙箱逃逸。
{% endhint %}

可以使用以下命令（需要 root 权限）来检查其状态和启用/禁用：
```bash
spctl --status
assessments enabled

spctl --enable
spctl --disable
#You can also allow nee identifies to execute code using the binary "spctl"
```
您还可以使用以下命令**查找文件是否具有扩展属性**：
```bash
xattr portada.png
com.apple.macl
com.apple.quarantine
```
使用以下命令检查扩展属性的值：
```bash
xattr -l portada.png
com.apple.macl:
00000000  03 00 53 DA 55 1B AE 4C 4E 88 9D CA B7 5C 50 F3  |..S.U..LN.....P.|
00000010  16 94 03 00 27 63 64 97 98 FB 4F 02 84 F3 D0 DB  |....'cd...O.....|
00000020  89 53 C3 FC 03 00 27 63 64 97 98 FB 4F 02 84 F3  |.S....'cd...O...|
00000030  D0 DB 89 53 C3 FC 00 00 00 00 00 00 00 00 00 00  |...S............|
00000040  00 00 00 00 00 00 00 00                          |........|
00000048
com.apple.quarantine: 0081;607842eb;Brave;F643CD5F-6071-46AB-83AB-390BA944DEC5
```
然后使用以下命令**删除**该属性：
```bash
xattr -d com.apple.quarantine portada.png
#You can also remove this attribute from every file with
find . -iname '*' -print0 | xargs -0 xattr -d com.apple.quarantine
```
使用以下命令查找所有被隔离的文件：

{% code overflow="wrap" %}
```bash
find / -exec ls -ld {} \; 2>/dev/null | grep -E "[x\-]@ " | awk '{printf $9; printf "\n"}' | xargs -I {} xattr -lv {} | grep "com.apple.quarantine"
```
{% endcode %}

## XProtect

XProtect是macOS中内置的反恶意软件功能。它是苹果的安全系统的一部分，静默地在后台工作，保护您的Mac免受已知恶意软件和恶意插件的侵害。

XProtect通过将任何下载的文件与其已知恶意软件和不安全文件类型的数据库进行检查来发挥作用。当您通过某些应用程序（如Safari、Mail或Messages）下载文件时，XProtect会自动扫描该文件。如果它在其数据库中与任何已知恶意软件匹配，XProtect将阻止该文件运行并向您发出警报。

XProtect数据库由苹果定期更新，包含新的恶意软件定义，并且这些更新会自动下载并安装到您的Mac上。这确保了XProtect始终与最新的已知威胁保持同步。

然而，值得注意的是，XProtect并不是一个功能齐全的防病毒解决方案。它只检查特定的已知威胁列表，并且不像大多数防病毒软件那样执行实时扫描。因此，虽然XProtect提供了对已知恶意软件的一层保护，但仍建议在从互联网下载文件或打开电子邮件附件时要谨慎。

您可以获取有关最新XProtect更新的信息：

{% code overflow="wrap" %}
```bash
system_profiler SPInstallHistoryDataType 2>/dev/null | grep -A 4 "XProtectPlistConfigData" | tail -n 5
```
{% endcode %}

## MRT - 恶意软件移除工具

恶意软件移除工具（MRT）是 macOS 安全基础设施的另一部分。顾名思义，MRT 的主要功能是**从受感染的系统中移除已知的恶意软件**。

一旦在 Mac 上检测到恶意软件（无论是通过 XProtect 还是其他方式），就可以使用 MRT 自动**移除恶意软件**。MRT 在后台静默运行，通常在系统更新或下载新的恶意软件定义时运行（看起来 MRT 用于检测恶意软件的规则嵌入在二进制文件中）。

虽然 XProtect 和 MRT 都是 macOS 的安全措施的一部分，但它们具有不同的功能：

* **XProtect** 是一种预防工具。它会在文件下载时（通过某些应用程序）**检查文件**，如果检测到任何已知类型的恶意软件，它会**阻止文件打开**，从而防止恶意软件首次感染系统。
* 另一方面，**MRT 是一种响应工具**。它在检测到系统上的恶意软件后运行，目的是移除有问题的软件以清理系统。

## 进程限制

### SIP - 系统完整性保护

{% content-ref url="macos-sip.md" %}
[macos-sip.md](macos-sip.md)
{% endcontent-ref %}

### 沙盒

macOS 沙盒将在沙盒内运行的应用程序**限制为沙盒配置文件中指定的允许操作**。这有助于确保**应用程序只能访问预期的资源**。

{% content-ref url="macos-sandbox/" %}
[macos-sandbox](macos-sandbox/)
{% endcontent-ref %}

### TCC - 透明度、同意和控制

**TCC（透明度、同意和控制）**是 macOS 中的一种机制，用于**限制和控制应用程序对某些功能的访问**，通常从隐私角度考虑。这可能包括位置服务、联系人、照片、麦克风、摄像头、辅助功能、完全磁盘访问等等。

{% content-ref url="macos-tcc/" %}
[macos-tcc](macos-tcc/)
{% endcontent-ref %}

## 信任缓存

苹果 macOS 信任缓存，有时也称为 AMFI（Apple Mobile File Integrity）缓存，是 macOS 中的一种安全机制，旨在**防止未经授权或恶意软件运行**。实质上，它是操作系统用于**验证软件的完整性和真实性的加密哈希列表**。

当应用程序或可执行文件尝试在 macOS 上运行时，操作系统会检查 AMFI 信任缓存。如果在信任缓存中找到文件的哈希值，则系统会**允许**该程序运行，因为它被识别为可信任的。

## 启动限制

它控制从何处以及什么可以启动 Apple 签名的二进制文件：

* 如果应该由 launchd 运行，则无法直接启动应用程序。
* 无法在受信任位置之外运行应用程序（如 /System/）。

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* 您在**网络安全公司**工作吗？您想在 HackTricks 中**宣传您的公司**吗？或者您想获得最新版本的 PEASS 或下载 PDF 格式的 HackTricks 吗？请查看[**订阅计划**](https://github.com/sponsors/carlospolop)！
* 发现我们的独家 [**NFTs**](https://opensea.io/collection/the-peass-family) 集合 - [**The PEASS Family**](https://opensea.io/collection/the-peass-family)
* 获取[**官方 PEASS 和 HackTricks 商品**](https://peass.creator-spring.com)
* **加入** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**Telegram 群组**](https://t.me/peass)，或在 **Twitter** 上**关注**我 [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**。**
* **通过向** [**hacktricks 仓库**](https://github.com/carlospolop/hacktricks) **和** [**hacktricks-cloud 仓库**](https://github.com/carlospolop/hacktricks-cloud) **提交 PR 来分享您的黑客技巧。**

</details>
