# macOS捆绑包

{% hint style="success" %}
学习并练习AWS Hacking：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks培训AWS红队专家（ARTE）**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习并练习GCP Hacking：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks培训GCP红队专家（GRTE）**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持HackTricks</summary>

* 查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord群组**](https://discord.gg/hRep4RUj7f) 或 [**电报群组**](https://t.me/peass) 或 **关注**我们的 **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* 通过向[**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来分享黑客技巧。

</details>
{% endhint %}

## 基本信息

macOS中的捆绑包用作各种资源（包括应用程序、库和其他必要文件）的容器，使它们在Finder中显示为单个对象，例如熟悉的`*.app`文件。最常见的捆绑包是`.app`捆绑包，但其他类型如`.framework`、`.systemextension`和`.kext`也很常见。

### 捆绑包的基本组件

在捆绑包中，特别是在`<application>.app/Contents/`目录中，存储着各种重要资源：

* **\_CodeSignature**：此目录存储了验证应用程序完整性所必需的代码签名详细信息。您可以使用命令检查代码签名信息，例如： %%%bash openssl dgst -binary -sha1 /Applications/Safari.app/Contents/Resources/Assets.car | openssl base64 %%%
* **MacOS**：包含应用程序的可执行二进制文件，用户交互时运行。
* **Resources**：存放应用程序的用户界面组件，包括图像、文档和界面描述（nib/xib文件）。
* **Info.plist**：作为应用程序的主要配置文件，对于系统识别和与应用程序交互至关重要。

#### Info.plist中的重要键

`Info.plist`文件是应用程序配置的基石，包含诸如以下键：

* **CFBundleExecutable**：指定位于`Contents/MacOS`目录中的主可执行文件的名称。
* **CFBundleIdentifier**：为应用程序提供全局标识符，macOS广泛用于应用程序管理。
* **LSMinimumSystemVersion**：指示应用程序运行所需的macOS最低版本。

### 探索捆绑包

要探索捆绑包的内容，例如`Safari.app`，可以使用以下命令：`bash ls -lR /Applications/Safari.app/Contents`

此探索将显示诸如`_CodeSignature`、`MacOS`、`Resources`等目录，以及`Info.plist`等文件，每个都具有从保护应用程序到定义其用户界面和操作参数的独特目的。

#### 其他捆绑包目录

除了常见目录外，捆绑包还可能包括：

* **Frameworks**：包含应用程序使用的捆绑框架。框架类似于带有额外资源的dylibs。
* **PlugIns**：用于增强应用程序功能的插件和扩展的目录。
* **XPCServices**：保存应用程序用于进程间通信的XPC服务。

这种结构确保了所有必要组件都封装在捆绑包中，促进了模块化和安全的应用程序环境。

有关`Info.plist`键及其含义的更详细信息，苹果开发者文档提供了广泛的资源：[Apple Info.plist键参考](https://developer.apple.com/library/archive/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html)。

{% hint style="success" %}
学习并练习AWS Hacking：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks培训AWS红队专家（ARTE）**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习并练习GCP Hacking：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks培训GCP红队专家（GRTE）**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持HackTricks</summary>

* 查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord群组**](https://discord.gg/hRep4RUj7f) 或 [**电报群组**](https://t.me/peass) 或 **关注**我们的 **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* 通过向[**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来分享黑客技巧。

</details>
{% endhint %}
