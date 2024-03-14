# FZ - Sub-GHz

<details>

<summary><strong>从零开始学习AWS黑客技术，成为专家</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS红队专家）</strong></a><strong>！</strong></summary>

支持HackTricks的其他方式：

* 如果您想看到您的**公司在HackTricks中做广告**或**下载PDF格式的HackTricks**，请查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
* 获取[**官方PEASS和HackTricks周边产品**](https://peass.creator-spring.com)
* 探索[**PEASS家族**](https://opensea.io/collection/the-peass-family)，我们的独家[NFTs](https://opensea.io/collection/the-peass-family)收藏品
* **加入** 💬 [**Discord群**](https://discord.gg/hRep4RUj7f) 或 [**电报群**](https://t.me/peass) 或在**Twitter**上关注我们 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**。**
* 通过向[**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来分享您的黑客技巧。

</details>

**Try Hard Security Group**

<figure><img src="../.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

***

## 简介 <a href="#kfpn7" id="kfpn7"></a>

Flipper Zero可以在300-928 MHz范围内**接收和发送无线电频率**，其内置模块可以读取、保存和模拟遥控器。这些遥控器用于与门、栅栏、无线电锁、遥控开关、无线门铃、智能灯等进行交互。Flipper Zero可以帮助您了解您的安全是否受到威胁。

<figure><img src="../../../.gitbook/assets/image (3) (2) (1).png" alt=""><figcaption></figcaption></figure>

## Sub-GHz硬件 <a href="#kfpn7" id="kfpn7"></a>

Flipper Zero具有内置的基于[CC1101芯片](https://www.ti.com/lit/ds/symlink/cc1101.pdf)和无线电天线的次1 GHz模块（最大范围为50米）。CC1101芯片和天线均设计用于在300-348 MHz、387-464 MHz和779-928 MHz频段工作。

<figure><img src="../../../.gitbook/assets/image (1) (8) (1).png" alt=""><figcaption></figcaption></figure>

## 操作

### 频率分析仪

{% hint style="info" %}
如何找到遥控器使用的频率
{% endhint %}

在分析时，Flipper Zero会在频率配置中的所有可用频率上扫描信号强度（RSSI）。Flipper Zero会显示具有最高RSSI值的频率，信号强度高于-90 [dBm](https://en.wikipedia.org/wiki/DBm)。

要确定遥控器的频率，请执行以下操作：

1. 将遥控器放在Flipper Zero的左侧非常近的位置。
2. 转到**主菜单** **→ Sub-GHz**。
3. 选择**频率分析仪**，然后按住要分析的遥控器上的按钮。
4. 在屏幕上查看频率值。

### 读取

{% hint style="info" %}
查找使用的频率信息（也是找到使用的频率的另一种方法）
{% endhint %}

**读取**选项会**监听配置频率**上指定的调制：默认为433.92 AM。如果在读取时**发现了什么**，屏幕上会显示信息。此信息可用于将来复制信号。

在使用读取时，可以按**左侧按钮**并**进行配置**。\
此时有**4种调制**（AM270、AM650、FM328和FM476），以及**存储的几个相关频率**：

<figure><img src="../../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

您可以设置**您感兴趣的任何频率**，但是，如果您**不确定遥控器使用的是哪个频率**，请将Hopping设置为ON（默认为Off），然后多次按按钮，直到Flipper捕获并提供您需要设置频率的信息。

{% hint style="danger" %}
在频率之间切换需要一些时间，因此在切换时传输的信号可能会被错过。为了获得更好的信号接收，请设置由频率分析仪确定的固定频率。
{% endhint %}

### **读取原始数据**

{% hint style="info" %}
在配置的频率上窃取（和重放）信号
{% endhint %}

**读取原始数据**选项会记录在监听频率上发送的信号。这可以用于**窃取**信号并**重放**它。

默认情况下，**读取原始数据也是在433.92的AM650中**，但如果使用读取选项发现您感兴趣的信号在**不同的频率/调制**上，您也可以通过按左键（在读取原始数据选项内）进行修改。

### 暴力破解

如果您知道例如车库门使用的协议，可以**生成所有代码并使用Flipper Zero发送它们**。这是一个支持通用常见类型车库的示例：[**https://github.com/tobiabocchi/flipperzero-bruteforce**](https://github.com/tobiabocchi/flipperzero-bruteforce)

### 手动添加

{% hint style="info" %}
从配置的协议列表中添加信号
{% endhint %}

#### [支持的协议列表](https://docs.flipperzero.one/sub-ghz/add-new-remote) <a href="#id-3iglu" id="id-3iglu"></a>

| Princeton\_433（适用于大多数静态代码系统） | 433.92 | 静态  |
| ---------------------------------------- | ------ | ----- |
| Nice Flo 12bit\_433                      | 433.92 | 静态  |
| Nice Flo 24bit\_433                      | 433.92 | 静态  |
| CAME 12bit\_433                          | 433.92 | 静态  |
| CAME 24bit\_433                          | 433.92 | 静态  |
| Linear\_300                              | 300.00 | 静态  |
| CAME TWEE                                | 433.92 | 静态  |
| Gate TX\_433                             | 433.92 | 静态  |
| DoorHan\_315                             | 315.00 | 动态  |
| DoorHan\_433                             | 433.92 | 动态  |
| LiftMaster\_315                          | 315.00 | 动态  |
| LiftMaster\_390                          | 390.00 | 动态  |
| Security+2.0\_310                        | 310.00 | 动态  |
| Security+2.0\_315                        | 315.00 | 动态  |
| Security+2.0\_390                        | 390.00 | 动态  |
### 支持的Sub-GHz供应商

查看列表[https://docs.flipperzero.one/sub-ghz/supported-vendors](https://docs.flipperzero.one/sub-ghz/supported-vendors)

### 按地区支持的频率

查看列表[https://docs.flipperzero.one/sub-ghz/frequencies](https://docs.flipperzero.one/sub-ghz/frequencies)

### 测试

{% hint style="info" %}
获取保存频率的dBm
{% endhint %}

## 参考

* [https://docs.flipperzero.one/sub-ghz](https://docs.flipperzero.one/sub-ghz)

**Try Hard Security Group**

<figure><img src="../.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

<details>

<summary><strong>从零开始学习AWS黑客技术，成为专家</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

支持HackTricks的其他方式:

* 如果您想在HackTricks中看到您的**公司广告**或**下载PDF格式的HackTricks**，请查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
* 获取[**官方PEASS & HackTricks周边产品**](https://peass.creator-spring.com)
* 发现[**PEASS家族**](https://opensea.io/collection/the-peass-family)，我们的独家[**NFTs**](https://opensea.io/collection/the-peass-family)收藏品
* **加入** 💬 [**Discord群**](https://discord.gg/hRep4RUj7f) 或 [**电报群**](https://t.me/peass) 或 **关注**我们的**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* 通过向[**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来分享您的黑客技巧。

</details>
