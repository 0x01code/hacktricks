# 其他网络技巧

{% hint style="success" %}
学习并练习 AWS 黑客技能：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks 培训 AWS 红队专家 (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习并练习 GCP 黑客技能：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks 培训 GCP 红队专家 (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持 HackTricks</summary>

* 查看 [**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**电报群组**](https://t.me/peass) 或 **关注** 我们的 **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* 通过向 [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github 仓库提交 PR 来分享黑客技巧。

</details>
{% endhint %}

### 主机头部

有时后端会信任 **主机头部** 来执行某些操作。例如，它可以使用其值作为 **发送密码重置的域**。因此，当您收到一封包含重置密码链接的电子邮件时，使用的域是您放在主机头部中的域。然后，您可以请求其他用户的密码重置并将域更改为您控制的域以窃取其密码重置代码。[WriteUp](https://medium.com/nassec-cybersecurity-writeups/how-i-was-able-to-take-over-any-users-account-with-host-header-injection-546fff6d0f2)。

{% hint style="warning" %}
请注意，您甚至可能无需等待用户单击重置密码链接以获取令牌，因为甚至 **垃圾邮件过滤器或其他中间设备/机器人可能会单击以分析它**。
{% endhint %}

### 会话布尔值

有时，当您正确完成某些验证时，后端将 **只向您的会话的安全属性添加一个值为“True”的布尔值**。然后，不同的端点将知道您是否成功通过了该检查。\
但是，如果您 **通过了检查** 并且您的会话在安全属性中被授予了该“True”值，您可以尝试 **访问其他资源**，这些资源 **依赖于相同的属性**，但您 **不应该有权限** 访问。[WriteUp](https://medium.com/@ozguralp/a-less-known-attack-vector-second-order-idor-attacks-14468009781a)。

### 注册功能

尝试以已存在的用户注册。还尝试使用等效字符（点、大量空格和 Unicode）。

### 接管电子邮件

注册一个电子邮件，在确认之前更改电子邮件，然后，如果新的确认电子邮件发送到第一个注册的电子邮件，您可以接管任何电子邮件。或者如果您可以启用第二个电子邮件确认第一个电子邮件，您也可以接管任何帐户。

### 访问使用 Atlassian 的公司内部服务台

{% embed url="https://yourcompanyname.atlassian.net/servicedesk/customer/user/login" %}

### TRACE 方法

开发人员可能会忘记在生产环境中禁用各种调试选项。例如，HTTP `TRACE` 方法是为诊断目的而设计的。如果启用，Web 服务器将通过在响应中回显接收到的确切请求来响应使用 `TRACE` 方法的请求。这种行为通常是无害的，但有时会导致信息泄露，例如可能由反向代理附加到请求的内部身份验证标头的名称。![Image for post](https://miro.medium.com/max/60/1\*wDFRADTOd9Tj63xucenvAA.png?q=20)

![Image for post](https://miro.medium.com/max/1330/1\*wDFRADTOd9Tj63xucenvAA.png)


{% hint style="success" %}
学习并练习 AWS 黑客技能：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks 培训 AWS 红队专家 (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习并练习 GCP 黑客技能：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks 培训 GCP 红队专家 (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持 HackTricks</summary>

* 查看 [**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**电报群组**](https://t.me/peass) 或 **关注** 我们的 **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* 通过向 [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github 仓库提交 PR 来分享黑客技巧。

</details>
{% endhint %}
