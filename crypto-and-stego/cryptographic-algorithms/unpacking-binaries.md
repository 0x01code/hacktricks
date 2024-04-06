<details>

<summary><strong>从零开始学习AWS黑客技术，成为专家</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS红队专家）</strong></a><strong>！</strong></summary>

支持HackTricks的其他方式：

* 如果您想在HackTricks中看到您的**公司广告**或**下载PDF格式的HackTricks**，请查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
* 获取[**官方PEASS & HackTricks周边产品**](https://peass.creator-spring.com)
* 探索[**PEASS家族**](https://opensea.io/collection/the-peass-family)，我们独家的[**NFTs**](https://opensea.io/collection/the-peass-family)收藏品
* **加入** 💬 [**Discord群**](https://discord.gg/hRep4RUj7f) 或 [**电报群**](https://t.me/peass) 或在**Twitter**上**关注**我们 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**。**
* 通过向[**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来**分享您的黑客技巧**。

</details>


# 识别打包的二进制文件

* **缺少字符串**：发现打包的二进制文件几乎没有任何字符串是很常见的
* 大量**未使用的字符串**：当恶意软件使用某种商业打包工具时，通常会发现大量没有交叉引用的字符串。即使存在这些字符串，也不意味着二进制文件没有被打包。
* 您还可以使用一些工具来尝试找出用于打包二进制文件的打包工具：
* [PEiD](http://www.softpedia.com/get/Programming/Packers-Crypters-Protectors/PEiD-updated.shtml)
* [Exeinfo PE](http://www.softpedia.com/get/Programming/Packers-Crypters-Protectors/ExEinfo-PE.shtml)
* [Language 2000](http://farrokhi.net/language/)

# 基本建议

* **从底部开始**在IDA中分析打包的二进制文件，然后向上移动。解包器在解包的代码退出时退出，因此解包器不太可能在开始时将执行权传递给解包的代码。
* 搜索**JMP**或**CALL**到**寄存器**或**内存区域**。还要搜索**将参数推送到地址方向然后调用`retn`**的函数，因为在这种情况下函数的返回可能会调用刚刚推送到堆栈上的地址。
* 在`VirtualAlloc`上设置**断点**，因为这会在程序可以写入解包的代码的内存空间中分配空间。"运行到用户代码"或使用F8在执行函数后**进入EAX内部的值**，然后**在转储中跟随该地址**。您永远不知道解包的代码将保存在哪个区域。
* **`VirtualAlloc`**的值为“**40**”作为参数表示Read+Write+Execute（需要执行的某些代码将被复制到这里）。
* **解包**代码时，通常会发现**多次调用**算术运算和函数，如**`memcopy`**或**`Virtual`**`Alloc`。如果发现自己在一个似乎只执行算术运算和可能一些`memcopy`的函数中，建议尝试**找到函数的结尾**（也许是JMP或调用某个寄存器）**或**至少是**最后一个函数的调用**，然后运行代码，因为这部分代码不重要。
* 在解包代码时，**注意**每当**更改内存区域**时，内存区域的更改可能表示**解包代码的开始**。您可以使用Process Hacker轻松转储内存区域（进程 --> 属性 --> 内存）。
* 尝试解包代码时，了解**您是否已经在处理解包的代码**（因此可以直接转储它）的一个好方法是**检查二进制文件的字符串**。如果在某个时刻执行了跳转（也许更改了内存区域）并且注意到**添加了更多字符串**，那么您就可以知道**您正在处理解包的代码**。\
但是，如果打包工具已经包含了许多字符串，您可以查看包含单词“http”的字符串数量是否增加。
* 当您从内存区域转储一个可执行文件时，您可以使用[PE-bear](https://github.com/hasherezade/pe-bear-releases/releases)修复一些标头。

<details>

<summary><strong>从零开始学习AWS黑客技术，成为专家</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS红队专家）</strong></a><strong>！</strong></summary>

支持HackTricks的其他方式：

* 如果您想在HackTricks中看到您的**公司广告**或**下载PDF格式的HackTricks**，请查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
* 获取[**官方PEASS & HackTricks周边产品**](https://peass.creator-spring.com)
* 探索[**PEASS家族**](https://opensea.io/collection/the-peass-family)，我们独家的[**NFTs**](https://opensea.io/collection/the-peass-family)收藏品
* **加入** 💬 [**Discord群**](https://discord.gg/hRep4RUj7f) 或 [**电报群**](https://t.me/peass) 或在**Twitter**上**关注**我们 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**。**
* 通过向[**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来**分享您的黑客技巧**。

</details>
