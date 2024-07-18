# Android 取证

{% hint style="success" %}
学习并练习 AWS 黑客：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks 培训 AWS 红队专家 (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习并练习 GCP 黑客：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks 培训 GCP 红队专家 (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持 HackTricks</summary>

* 查看 [**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**电报群组**](https://t.me/peass) 或 **关注** 我们的 **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* 通过向 [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github 仓库提交 PR 来分享黑客技巧。

</details>
{% endhint %}

## 锁定设备

要开始从 Android 设备中提取数据，必须先解锁设备。如果设备已锁定，您可以：

* 检查设备是否启用了通过 USB 调试。
* 检查可能的 [指纹攻击](https://www.usenix.org/legacy/event/woot10/tech/full\_papers/Aviv.pdf)
* 尝试使用 [暴力破解](https://www.cultofmac.com/316532/this-brute-force-device-can-crack-any-iphones-pin-code/)

## 数据获取

使用 adb 创建一个 [android 备份](mobile-pentesting/android-app-pentesting/adb-commands.md#backup)，然后使用 [Android Backup Extractor](https://sourceforge.net/projects/adbextractor/) 提取它：`java -jar abe.jar unpack file.backup file.tar`

### 如果有 root 访问权限或物理连接到 JTAG 接口

* `cat /proc/partitions`（查找闪存的路径，通常第一个条目是 _mmcblk0_，对应整个闪存）。
* `df /data`（发现系统的块大小）。
* dd if=/dev/block/mmcblk0 of=/sdcard/blk0.img bs=4096（使用从块大小获取的信息执行）。

### 内存

使用 Linux Memory Extractor (LiME) 提取 RAM 信息。这是一个应通过 adb 加载的内核扩展。

{% hint style="success" %}
学习并练习 AWS 黑客：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks 培训 AWS 红队专家 (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
学习并练习 GCP 黑客：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks 培训 GCP 红队专家 (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>支持 HackTricks</summary>

* 查看 [**订阅计划**](https://github.com/sponsors/carlospolop)!
* **加入** 💬 [**Discord 群组**](https://discord.gg/hRep4RUj7f) 或 [**电报群组**](https://t.me/peass) 或 **关注** 我们的 **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**.**
* 通过向 [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github 仓库提交 PR 来分享黑客技巧。

</details>
{% endhint %}
