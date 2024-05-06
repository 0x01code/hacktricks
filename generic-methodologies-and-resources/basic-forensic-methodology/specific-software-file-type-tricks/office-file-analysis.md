# Office文件分析

<details>

<summary><strong>从零开始学习AWS黑客技术，成为专家</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS Red Team Expert）</strong></a><strong>！</strong></summary>

支持HackTricks的其他方式：

- 如果您想看到您的**公司在HackTricks中做广告**或**下载PDF格式的HackTricks**，请查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
- 获取[**官方PEASS & HackTricks周边产品**](https://peass.creator-spring.com)
- 探索[**PEASS家族**](https://opensea.io/collection/the-peass-family)，我们的独家[NFT](https://opensea.io/collection/the-peass-family)收藏品
- **加入** 💬 [**Discord群组**](https://discord.gg/hRep4RUj7f) 或 [**电报群组**](https://t.me/peass) 或 **关注**我们的**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**。**
- 通过向[**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来分享您的黑客技巧。

</details>

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
使用[**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=office-file-analysis)可以轻松构建和**自动化工作流程**，使用世界上**最先进**的社区工具。\
立即获取访问权限：

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=office-file-analysis" %}

有关更多信息，请查看[https://trailofbits.github.io/ctf/forensics/](https://trailofbits.github.io/ctf/forensics/)。这只是一个摘要：

微软创建了许多办公文档格式，其中两种主要类型是**OLE格式**（如RTF、DOC、XLS、PPT）和**Office Open XML（OOXML）格式**（如DOCX、XLSX、PPTX）。这些格式可以包含宏，使它们成为钓鱼和恶意软件的目标。OOXML文件结构化为zip容器，允许通过解压缩进行检查，揭示文件和文件夹层次结构以及XML文件内容。

为了探索OOXML文件结构，给出了解压缩文档的命令和输出结构。已记录了在这些文件中隐藏数据的技术，表明在CTF挑战中数据隐藏方面的持续创新。

对于分析，**oletools**和**OfficeDissector**提供了用于检查OLE和OOXML文档的全面工具集。这些工具有助于识别和分析嵌入的宏，这些宏通常用作恶意软件传递的向量，通常会下载并执行其他恶意负载。可以使用Libre Office进行VBA宏的分析，而无需Microsoft Office，Libre Office允许使用断点和监视变量进行调试。

**oletools**的安装和使用非常简单，提供了通过pip安装和从文档中提取宏的命令。自动执行宏是通过`AutoOpen`、`AutoExec`或`Document_Open`等函数触发的。
```bash
sudo pip3 install -U oletools
olevba -c /path/to/document #Extract macros
```
<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
使用[**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=office-file-analysis)轻松构建并**自动化**由全球**最先进**的社区工具提供支持的工作流程。\
立即获取访问权限：

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=office-file-analysis" %}

<details>

<summary><strong>从零开始学习AWS黑客技术，成为专家</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS Red Team Expert）</strong></a><strong>！</strong></summary>

支持HackTricks的其他方式：

* 如果您想看到您的**公司在HackTricks中做广告**或**下载PDF格式的HackTricks**，请查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
* 获取[**官方PEASS & HackTricks周边产品**](https://peass.creator-spring.com)
* 发现[**PEASS家族**](https://opensea.io/collection/the-peass-family)，我们的独家[**NFTs**](https://opensea.io/collection/the-peass-family)
* **加入** 💬 [**Discord群组**](https://discord.gg/hRep4RUj7f) 或 [**电报群组**](https://t.me/peass) 或在**Twitter**上关注我们 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**。**
* 通过向[**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来分享您的黑客技巧。

</details>
