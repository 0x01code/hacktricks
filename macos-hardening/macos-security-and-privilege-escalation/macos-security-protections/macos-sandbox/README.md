# macOS Sandbox

{% hint style="success" %}
Learn & practice AWS Hacking:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
Learn & practice GCP Hacking: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>Support HackTricks</summary>

* Check the [**subscription plans**](https://github.com/sponsors/carlospolop)!
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **Share hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
{% endhint %}

## Basic Information

MacOS Sandbox（最初称为 Seatbelt）**限制应用程序**在沙箱内运行时的**允许操作，这些操作在应用程序运行的沙箱配置文件中指定**。这有助于确保**应用程序仅访问预期的资源**。

任何具有**权限** **`com.apple.security.app-sandbox`**的应用程序都将在沙箱内执行。**Apple 二进制文件**通常在沙箱内执行，并且为了在**App Store**中发布，**此权限是强制性的**。因此，大多数应用程序将在沙箱内执行。

为了控制进程可以或不能做什么，**沙箱在内核的所有**syscalls**中都有钩子**。**根据**应用程序的**权限**，沙箱将**允许**某些操作。

沙箱的一些重要组件包括：

* **内核扩展** `/System/Library/Extensions/Sandbox.kext`
* **私有框架** `/System/Library/PrivateFrameworks/AppSandbox.framework`
* 在用户空间运行的**守护进程** `/usr/libexec/sandboxd`
* **容器** `~/Library/Containers`

在容器文件夹中，您可以找到**每个以沙箱方式执行的应用程序的文件夹**，其名称为包 ID：
```bash
ls -l ~/Library/Containers
total 0
drwx------@ 4 username  staff  128 May 23 20:20 com.apple.AMPArtworkAgent
drwx------@ 4 username  staff  128 May 23 20:13 com.apple.AMPDeviceDiscoveryAgent
drwx------@ 4 username  staff  128 Mar 24 18:03 com.apple.AVConference.Diagnostic
drwx------@ 4 username  staff  128 Mar 25 14:14 com.apple.Accessibility-Settings.extension
drwx------@ 4 username  staff  128 Mar 25 14:10 com.apple.ActionKit.BundledIntentHandler
[...]
```
在每个 bundle id 文件夹内，您可以找到应用的 **plist** 和 **数据目录**：
```bash
cd /Users/username/Library/Containers/com.apple.Safari
ls -la
total 104
drwx------@   4 username  staff    128 Mar 24 18:08 .
drwx------  348 username  staff  11136 May 23 20:57 ..
-rw-r--r--    1 username  staff  50214 Mar 24 18:08 .com.apple.containermanagerd.metadata.plist
drwx------   13 username  staff    416 Mar 24 18:05 Data

ls -l Data
total 0
drwxr-xr-x@  8 username  staff   256 Mar 24 18:08 CloudKit
lrwxr-xr-x   1 username  staff    19 Mar 24 18:02 Desktop -> ../../../../Desktop
drwx------   2 username  staff    64 Mar 24 18:02 Documents
lrwxr-xr-x   1 username  staff    21 Mar 24 18:02 Downloads -> ../../../../Downloads
drwx------  35 username  staff  1120 Mar 24 18:08 Library
lrwxr-xr-x   1 username  staff    18 Mar 24 18:02 Movies -> ../../../../Movies
lrwxr-xr-x   1 username  staff    17 Mar 24 18:02 Music -> ../../../../Music
lrwxr-xr-x   1 username  staff    20 Mar 24 18:02 Pictures -> ../../../../Pictures
drwx------   2 username  staff    64 Mar 24 18:02 SystemData
drwx------   2 username  staff    64 Mar 24 18:02 tmp
```
{% hint style="danger" %}
请注意，即使符号链接存在以“逃离”沙箱并访问其他文件夹，应用程序仍然需要**拥有权限**才能访问它们。这些权限在**`.plist`**中。
{% endhint %}
```bash
# Get permissions
plutil -convert xml1 .com.apple.containermanagerd.metadata.plist -o -

# Binary sandbox profile
<key>SandboxProfileData</key>
<data>
AAAhAboBAAAAAAgAAABZAO4B5AHjBMkEQAUPBSsGPwsgASABHgEgASABHwEf...

# In this file you can find the entitlements:
<key>Entitlements</key>
<dict>
<key>com.apple.MobileAsset.PhishingImageClassifier2</key>
<true/>
<key>com.apple.accounts.appleaccount.fullaccess</key>
<true/>
<key>com.apple.appattest.spi</key>
<true/>
<key>keychain-access-groups</key>
<array>
<string>6N38VWS5BX.ru.keepcoder.Telegram</string>
<string>6N38VWS5BX.ru.keepcoder.TelegramShare</string>
</array>
[...]

# Some parameters
<key>Parameters</key>
<dict>
<key>_HOME</key>
<string>/Users/username</string>
<key>_UID</key>
<string>501</string>
<key>_USER</key>
<string>username</string>
[...]

# The paths it can access
<key>RedirectablePaths</key>
<array>
<string>/Users/username/Downloads</string>
<string>/Users/username/Documents</string>
<string>/Users/username/Library/Calendars</string>
<string>/Users/username/Desktop</string>
<key>RedirectedPaths</key>
<array/>
[...]
```
{% hint style="warning" %}
由沙盒应用程序创建/修改的所有内容将获得**隔离属性**。这将通过触发Gatekeeper来防止沙盒空间，如果沙盒应用程序尝试使用**`open`**执行某些操作。
{% endhint %}

### 沙盒配置文件

沙盒配置文件是指示在该**沙盒**中将被**允许/禁止**的配置文件。它使用**沙盒配置文件语言（SBPL）**，该语言使用[**Scheme**](https://en.wikipedia.org/wiki/Scheme\_\(programming\_language\))编程语言。

在这里您可以找到一个示例：
```scheme
(version 1) ; First you get the version

(deny default) ; Then you shuold indicate the default action when no rule applies

(allow network*) ; You can use wildcards and allow everything

(allow file-read* ; You can specify where to apply the rule
(subpath "/Users/username/")
(literal "/tmp/afile")
(regex #"^/private/etc/.*")
)

(allow mach-lookup
(global-name "com.apple.analyticsd")
)
```
{% hint style="success" %}
查看这个 [**研究**](https://reverse.put.as/2011/09/14/apple-sandbox-guide-v1-0/) **以检查更多可以被允许或拒绝的操作。**
{% endhint %}

重要的 **系统服务** 也在它们自己的自定义 **沙箱** 中运行，例如 `mdnsresponder` 服务。您可以在以下位置查看这些自定义 **沙箱配置文件**：

* **`/usr/share/sandbox`**
* **`/System/Library/Sandbox/Profiles`**&#x20;
* 其他沙箱配置文件可以在 [https://github.com/s7ephen/OSX-Sandbox--Seatbelt--Profiles](https://github.com/s7ephen/OSX-Sandbox--Seatbelt--Profiles) 中查看。

**App Store** 应用使用 **配置文件** **`/System/Library/Sandbox/Profiles/application.sb`**。您可以在此配置文件中检查诸如 **`com.apple.security.network.server`** 的权限如何允许进程使用网络。

SIP 是一个名为 platform\_profile 的沙箱配置文件，位于 /System/Library/Sandbox/rootless.conf

### 沙箱配置文件示例

要使用 **特定沙箱配置文件** 启动应用程序，您可以使用：
```bash
sandbox-exec -f example.sb /Path/To/The/Application
```
{% tabs %}
{% tab title="touch" %}
{% code title="touch.sb" %}
```scheme
(version 1)
(deny default)
(allow file* (literal "/tmp/hacktricks.txt"))
```
{% endcode %}
```bash
# This will fail because default is denied, so it cannot execute touch
sandbox-exec -f touch.sb touch /tmp/hacktricks.txt
# Check logs
log show --style syslog --predicate 'eventMessage contains[c] "sandbox"' --last 30s
[...]
2023-05-26 13:42:44.136082+0200  localhost kernel[0]: (Sandbox) Sandbox: sandbox-exec(41398) deny(1) process-exec* /usr/bin/touch
2023-05-26 13:42:44.136100+0200  localhost kernel[0]: (Sandbox) Sandbox: sandbox-exec(41398) deny(1) file-read-metadata /usr/bin/touch
2023-05-26 13:42:44.136321+0200  localhost kernel[0]: (Sandbox) Sandbox: sandbox-exec(41398) deny(1) file-read-metadata /var
2023-05-26 13:42:52.701382+0200  localhost kernel[0]: (Sandbox) 5 duplicate reports for Sandbox: sandbox-exec(41398) deny(1) file-read-metadata /var
[...]
```
{% code title="touch2.sb" %}
```scheme
(version 1)
(deny default)
(allow file* (literal "/tmp/hacktricks.txt"))
(allow process* (literal "/usr/bin/touch"))
; This will also fail because:
; 2023-05-26 13:44:59.840002+0200  localhost kernel[0]: (Sandbox) Sandbox: touch(41575) deny(1) file-read-metadata /usr/bin/touch
; 2023-05-26 13:44:59.840016+0200  localhost kernel[0]: (Sandbox) Sandbox: touch(41575) deny(1) file-read-data /usr/bin/touch
; 2023-05-26 13:44:59.840028+0200  localhost kernel[0]: (Sandbox) Sandbox: touch(41575) deny(1) file-read-data /usr/bin
; 2023-05-26 13:44:59.840034+0200  localhost kernel[0]: (Sandbox) Sandbox: touch(41575) deny(1) file-read-metadata /usr/lib/dyld
; 2023-05-26 13:44:59.840050+0200  localhost kernel[0]: (Sandbox) Sandbox: touch(41575) deny(1) sysctl-read kern.bootargs
; 2023-05-26 13:44:59.840061+0200  localhost kernel[0]: (Sandbox) Sandbox: touch(41575) deny(1) file-read-data /
```
{% endcode %}

{% code title="touch3.sb" %}
```scheme
(version 1)
(deny default)
(allow file* (literal "/private/tmp/hacktricks.txt"))
(allow process* (literal "/usr/bin/touch"))
(allow file-read-data (literal "/"))
; This one will work
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% hint style="info" %}
请注意，**苹果编写的** **软件** 在 **Windows** 上 **没有额外的安全措施**，例如应用程序沙箱。
{% endhint %}

绕过示例：

* [https://lapcatsoftware.com/articles/sandbox-escape.html](https://lapcatsoftware.com/articles/sandbox-escape.html)
* [https://desi-jarvis.medium.com/office365-macos-sandbox-escape-fcce4fa4123c](https://desi-jarvis.medium.com/office365-macos-sandbox-escape-fcce4fa4123c)（他们能够写入以 `~$` 开头的沙箱外文件）。

### MacOS 沙箱配置文件

macOS 将系统沙箱配置文件存储在两个位置：**/usr/share/sandbox/** 和 **/System/Library/Sandbox/Profiles**。

如果第三方应用程序携带 _**com.apple.security.app-sandbox**_ 权限，系统会将 **/System/Library/Sandbox/Profiles/application.sb** 配置文件应用于该进程。

### **iOS 沙箱配置文件**

默认配置文件称为 **container**，我们没有 SBPL 文本表示。在内存中，这个沙箱被表示为每个权限的允许/拒绝二叉树。

### 调试与绕过沙箱

在 macOS 上，与 iOS 不同，iOS 的进程从一开始就由内核进行沙箱化，**进程必须自行选择进入沙箱**。这意味着在 macOS 上，进程在主动决定进入沙箱之前不会受到沙箱的限制。

如果进程具有权限 `com.apple.security.app-sandbox`，则它们在启动时会自动从用户空间沙箱化。有关此过程的详细说明，请查看：

{% content-ref url="macos-sandbox-debug-and-bypass/" %}
[macos-sandbox-debug-and-bypass](macos-sandbox-debug-and-bypass/)
{% endcontent-ref %}

### **检查 PID 权限**

[**根据这个**](https://www.youtube.com/watch?v=mG715HcDgO8\&t=3011s)，**`sandbox_check`**（它是一个 `__mac_syscall`）可以检查 **某个 PID 是否允许某个操作**。

[**工具 sbtool**](http://newosxbook.com/src.jl?tree=listings\&file=sbtool.c) 可以检查某个 PID 是否可以执行某个操作：
```bash
sbtool <pid> mach #Check mac-ports (got from launchd with an api)
sbtool <pid> file /tmp #Check file access
sbtool <pid> inspect #Gives you an explaination of the sandbox profile
sbtool <pid> all
```
### App Store 应用中的自定义 SBPL

公司可能会使他们的应用程序 **使用自定义沙盒配置文件**（而不是默认配置文件）。他们需要使用权限 **`com.apple.security.temporary-exception.sbpl`**，该权限需要得到 Apple 的授权。

可以在 **`/System/Library/Sandbox/Profiles/application.sb:`** 中检查此权限的定义。
```scheme
(sandbox-array-entitlement
"com.apple.security.temporary-exception.sbpl"
(lambda (string)
(let* ((port (open-input-string string)) (sbpl (read port)))
(with-transparent-redirection (eval sbpl)))))
```
这将**在此权限后评估字符串**作为沙盒配置文件。

{% hint style="success" %}
学习与实践 AWS 黑客技术：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks 培训 AWS 红队专家 (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习与实践 GCP 黑客技术：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks 培训 GCP 红队专家 (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持 HackTricks</summary>

* 查看 [**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**Telegram 群组**](https://t.me/peass) 或 **关注** 我们的 **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* **通过向** [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) GitHub 仓库提交 PR 来分享黑客技巧。

</details>
{% endhint %}
