<details>

<summary><strong>从零到英雄学习AWS黑客攻击</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>！</strong></summary>

支持HackTricks的其他方式：

* 如果您想在**HackTricks中看到您的公司广告**或**下载HackTricks的PDF**，请查看[**订阅计划**](https://github.com/sponsors/carlospolop)！
* 获取[**官方PEASS & HackTricks商品**](https://peass.creator-spring.com)
* 发现[**PEASS家族**](https://opensea.io/collection/the-peass-family)，我们独家的[**NFTs系列**](https://opensea.io/collection/the-peass-family)
* **加入** 💬 [**Discord群组**](https://discord.gg/hRep4RUj7f) 或 [**telegram群组**](https://t.me/peass) 或在 **Twitter** 🐦 上**关注**我 [**@carlospolopm**](https://twitter.com/carlospolopm)**。**
* **通过向** [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来分享您的黑客技巧。

</details>


# 识别打包的二进制文件

* **缺少字符串**：通常发现打包的二进制文件几乎没有任何字符串
* 许多**未使用的字符串**：同样，当恶意软件使用某种商业打包器时，常常会发现许多没有交叉引用的字符串。即使这些字符串存在，也并不意味着二进制文件没有被打包。
* 您还可以使用一些工具尝试找出用于打包二进制文件的打包器：
* [PEiD](http://www.softpedia.com/get/Programming/Packers-Crypters-Protectors/PEiD-updated.shtml)
* [Exeinfo PE](http://www.softpedia.com/get/Programming/Packers-Crypters-Protectors/ExEinfo-PE.shtml)
* [Language 2000](http://farrokhi.net/language/)

# 基本建议

* **从IDA底部开始**分析打包的二进制文件**并向上移动**。解包器一旦解包代码退出就会退出，因此解包器不太可能在开始时将执行权传递给解包代码。
* 寻找**跳转**或**调用**到**寄存器**或**内存区域**的**JMP's**或**CALLs**。还要寻找**函数推送参数和一个地址方向然后调用`retn`**，因为在这种情况下函数的返回可能会调用刚刚推送到堆栈上的地址。
* 在`VirtualAlloc`上设置**断点**，因为这会在内存中分配空间，程序可以在其中写入解包代码。使用"运行到用户代码"或使用F8**获取EAX内的值**执行函数后，**在转储中跟踪该地址**。你永远不知道那是否是将要保存解包代码的区域。
* **`VirtualAlloc`**的参数值为"**40**"意味着读+写+执行（一些需要执行的代码将被复制到这里）。
* 在解包代码时，通常会发现对**算术操作**和像**`memcopy`**或**`Virtual`**`Alloc`这样的函数的**多次调用**。如果你发现自己处于一个看似只执行算术操作和可能一些`memcopy`的函数中，建议尝试**找到函数的末尾**（可能是跳转或调用某个寄存器）**或**至少是**调用最后一个函数**并运行到那里，因为代码不是很有趣。
* 在解包代码时**注意**每当你**改变内存区域**，因为内存区域的改变可能表明**解包代码的开始**。你可以使用Process Hacker轻松转储内存区域（进程 --> 属性 --> 内存）。
* 在尝试解包代码时，判断你是否已经在处理解包代码的一个好方法（这样你就可以直接转储它）是**检查二进制文件的字符串**。如果在某个时刻你执行了一个跳转（可能改变了内存区域），并且你注意到**添加了更多的字符串**，那么你就可以知道**你正在处理解包代码**。\
然而，如果打包器已经包含了很多字符串，你可以看看有多少字符串包含单词"http"，并看看这个数字是否增加了。
* 当你从内存区域转储可执行文件时，你可以使用[PE-bear](https://github.com/hasherezade/pe-bear-releases/releases)修复一些头部。


<details>

<summary><strong>从零到英雄学习AWS黑客攻击</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>！</strong></summary>

支持HackTricks的其他方式：

* 如果您想在**HackTricks中看到您的公司广告**或**下载HackTricks的PDF**，请查看[**订阅计划**](https://github.com/sponsors/carlospolop)！
* 获取[**官方PEASS & HackTricks商品**](https://peass.creator-spring.com)
* 发现[**PEASS家族**](https://opensea.io/collection/the-peass-family)，我们独家的[**NFTs系列**](https://opensea.io/collection/the-peass-family)
* **加入** 💬 [**Discord群组**](https://discord.gg/hRep4RUj7f) 或 [**telegram群组**](https://t.me/peass) 或在 **Twitter** 🐦 上**关注**我 [**@carlospolopm**](https://twitter.com/carlospolopm)**。**
* **通过向** [**HackTricks**](https://github.com/carlospolop/hacktricks) 和 [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来分享您的黑客技巧。

</details>
