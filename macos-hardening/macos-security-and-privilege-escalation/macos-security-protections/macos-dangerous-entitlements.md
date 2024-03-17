# macOS危险权限和TCC权限

<details>

<summary><strong>从零开始学习AWS黑客技术，成为专家</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS红队专家）</strong></a><strong>！</strong></summary>

支持HackTricks的其他方式：

* 如果您想看到您的**公司在HackTricks中做广告**或**下载PDF格式的HackTricks**，请查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
* 获取[**官方PEASS和HackTricks周边产品**](https://peass.creator-spring.com)
* 探索[**PEASS家族**](https://opensea.io/collection/the-peass-family)，我们的独家[NFT收藏品](https://opensea.io/collection/the-peass-family)
* **加入** 💬 [**Discord群**](https://discord.gg/hRep4RUj7f) 或 [**电报群**](https://t.me/peass) 或在**Twitter**上关注我们 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**。**
* 通过向[**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来分享您的黑客技巧。

</details>

{% hint style="warning" %}
请注意，以**`com.apple`**开头的权限仅供苹果公司授予，第三方无法使用。
{% endhint %}

## 高级

### `com.apple.rootless.install.heritable`

权限**`com.apple.rootless.install.heritable`**允许**绕过SIP**。查看[此处了解更多信息](macos-sip.md#com.apple.rootless.install.heritable)。

### **`com.apple.rootless.install`**

权限**`com.apple.rootless.install`**允许**绕过SIP**。查看[此处了解更多信息](macos-sip.md#com.apple.rootless.install)。

### **`com.apple.system-task-ports`（之前称为`task_for_pid-allow`）**

此权限允许获取除内核外的任何进程的**任务端口**。查看[**此处了解更多信息**](../mac-os-architecture/macos-ipc-inter-process-communication/)。

### `com.apple.security.get-task-allow`

此权限允许具有**`com.apple.security.cs.debugger`**权限的其他进程获取具有此权限的二进制运行的进程的任务端口，并对其进行**代码注入**。查看[**此处了解更多信息**](../mac-os-architecture/macos-ipc-inter-process-communication/)。

### `com.apple.security.cs.debugger`

具有调试工具权限的应用程序可以调用`task_for_pid()`来检索未签名和第三方应用程序的有效任务端口，其中`Get Task Allow`权限设置为`true`。然而，即使具有调试工具权限，调试器**无法获取**没有`Get Task Allow`权限的进程的任务端口，因此受到系统完整性保护。查看[**此处了解更多信息**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_debugger)。

### `com.apple.security.cs.disable-library-validation`

此权限允许**加载未由苹果签名或与主可执行文件具有相同团队ID签名的框架、插件或库**，因此攻击者可能滥用某些任意库加载来注入代码。查看[**此处了解更多信息**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_disable-library-validation)。

### `com.apple.private.security.clear-library-validation`

此权限与**`com.apple.security.cs.disable-library-validation`**非常相似，但**不是直接禁用**库验证，而是允许进程**调用`csops`系统调用来禁用它**。\
查看[**此处了解更多信息**](https://theevilbit.github.io/posts/com.apple.private.security.clear-library-validation/)。

### `com.apple.security.cs.allow-dyld-environment-variables`

此权限允许**使用DYLD环境变量**，可用于注入库和代码。查看[**此处了解更多信息**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-dyld-environment-variables)。

### `com.apple.private.tcc.manager`或`com.apple.rootless.storage`.`TCC`

根据[**此博客**](https://objective-see.org/blog/blog\_0x4C.html)和[**此博客**](https://wojciechregula.blog/post/play-the-music-and-bypass-tcc-aka-cve-2020-29621/)，这些权限允许**修改** **TCC** 数据库。

### **`system.install.apple-software`**和**`system.install.apple-software.standar-user`**

这些权限允许**在不请求用户权限的情况下安装软件**，这对于**特权升级**可能有帮助。

### `com.apple.private.security.kext-management`

需要的权限，用于请求内核加载内核扩展。

### **`com.apple.private.icloud-account-access`**

权限**`com.apple.private.icloud-account-access`**可以与**`com.apple.iCloudHelper`** XPC服务通信，该服务将**提供iCloud令牌**。

**iMovie**和**Garageband**拥有此权限。

有关从该权限获取iCloud令牌的漏洞的更多**信息**，请查看演讲：[**#OBTS v5.0：“您的Mac上发生的事情，留在苹果的iCloud中？！” - Wojciech Regula**](https://www.youtube.com/watch?v=\_6e2LhmxVc0)

### `com.apple.private.tcc.manager.check-by-audit-token`

待办事项：我不知道这允许做什么

### `com.apple.private.apfs.revert-to-snapshot`

待办事项：在[**此报告**](https://jhftss.github.io/The-Nightmare-of-Apple-OTA-Update/)中提到，此权限可用于在重启后更新SSV保护的内容。如果您知道如何操作，请提交PR！

### `com.apple.private.apfs.create-sealed-snapshot`

待办事项：在[**此报告**](https://jhftss.github.io/The-Nightmare-of-Apple-OTA-Update/)中提到，此权限可用于在重启后更新SSV保护的内容。如果您知道如何操作，请提交PR！

### `keychain-access-groups`

此权限列出应用程序可以访问的**钥匙串**组。
```xml
<key>keychain-access-groups</key>
<array>
<string>ichat</string>
<string>apple</string>
<string>appleaccount</string>
<string>InternetAccounts</string>
<string>IMCore</string>
</array>
```
### **`kTCCServiceSystemPolicyAllFiles`**

提供**完全磁盘访问**权限，这是您可以拥有的TCC最高权限之一。

### **`kTCCServiceAppleEvents`**

允许应用程序向其他常用于**自动化任务**的应用程序发送事件。控制其他应用程序，可以滥用授予这些其他应用程序的权限。

比如让它们要求用户输入密码：
```bash
osascript -e 'tell app "App Store" to activate' -e 'tell app "App Store" to activate' -e 'tell app "App Store" to display dialog "App Store requires your password to continue." & return & return default answer "" with icon 1 with hidden answer with title "App Store Alert"'
```
{% endcode %}

或让它们执行**任意操作**。

### **`kTCCServiceEndpointSecurityClient`**

允许，除其他权限外，**写入用户的 TCC 数据库**。

### **`kTCCServiceSystemPolicySysAdminFiles`**

允许**更改**用户的 **`NFSHomeDirectory`** 属性，从而更改其主文件夹路径，因此可以**绕过 TCC**。

### **`kTCCServiceSystemPolicyAppBundles`**

允许修改应用程序包内的文件（在 app.app 内），这是**默认禁止的**。

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

可以在 _系统偏好设置_ > _隐私与安全性_ > _应用程序管理_ 中检查谁拥有此访问权限。

### `kTCCServiceAccessibility`

该进程将能够**滥用 macOS 辅助功能**，这意味着例如他将能够按下按键。因此，他可以请求访问控制应用程序，如 Finder，并使用此权限批准对话框。

## 中等

### `com.apple.security.cs.allow-jit`

此授权允许**创建可写和可执行的内存**，通过将 `MAP_JIT` 标志传递给 `mmap()` 系统函数。查看[**此处获取更多信息**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-jit)。

### `com.apple.security.cs.allow-unsigned-executable-memory`

此授权允许**覆盖或修补 C 代码**，使用长期不推荐使用的 **`NSCreateObjectFileImageFromMemory`**（基本上是不安全的），或使用 **DVDPlayback** 框架。查看[**此处获取更多信息**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_allow-unsigned-executable-memory)。

{% hint style="danger" %}
包含此授权会使您的应用程序暴露于内存不安全代码语言中的常见漏洞。请仔细考虑您的应用程序是否需要此例外。
{% endhint %}

### `com.apple.security.cs.disable-executable-page-protection`

此授权允许**修改其磁盘上自己的可执行文件的部分**以强制退出。查看[**此处获取更多信息**](https://developer.apple.com/documentation/bundleresources/entitlements/com\_apple\_security\_cs\_disable-executable-page-protection)。

{% hint style="danger" %}
禁用可执行页面保护授权是一项极端授权，它会从您的应用程序中删除一项基本安全保护，使攻击者有可能在不被察觉的情况下重写您的应用程序的可执行代码。如果可能的话，请优先选择更狭窄的授权。
{% endhint %}

### `com.apple.security.cs.allow-relative-library-loads`

待办事项

### `com.apple.private.nullfs_allow`

此授权允许挂载 nullfs 文件系统（默认情况下被禁止）。工具：[**mount\_nullfs**](https://github.com/JamaicanMoose/mount\_nullfs/tree/master)。

### `kTCCServiceAll`

根据这篇博文，此 TCC 权限通常以以下形式找到：
```
[Key] com.apple.private.tcc.allow-prompting
[Value]
[Array]
[String] kTCCServiceAll
```
允许进程**请求所有TCC权限**。

### **`kTCCServicePostEvent`**

<details>

<summary><strong>从零开始学习AWS黑客技术，成为专家</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS Red Team Expert）</strong></a><strong>！</strong></summary>

其他支持HackTricks的方式：

* 如果您想看到您的**公司在HackTricks中做广告**或**下载PDF格式的HackTricks**，请查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
* 获取[**官方PEASS & HackTricks周边产品**](https://peass.creator-spring.com)
* 探索[**PEASS家族**](https://opensea.io/collection/the-peass-family)，我们的独家[**NFTs**](https://opensea.io/collection/the-peass-family)
* **加入** 💬 [**Discord群**](https://discord.gg/hRep4RUj7f) 或 [**电报群**](https://t.me/peass) 或 **关注**我们的**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**。**
* 通过向[**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来分享您的黑客技巧。 

</details>
