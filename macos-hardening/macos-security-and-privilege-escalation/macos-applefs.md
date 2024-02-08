# macOS AppleFS

<details>

<summary><strong>从零开始学习AWS黑客技术，成为专家</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS Red Team Expert）</strong></a><strong>！</strong></summary>

支持HackTricks的其他方式：

* 如果您想看到您的**公司在HackTricks中做广告**或**下载PDF格式的HackTricks**，请查看[**订阅计划**](https://github.com/sponsors/carlospolop)!
* 获取[**官方PEASS & HackTricks周边产品**](https://peass.creator-spring.com)
* 探索[**PEASS家族**](https://opensea.io/collection/the-peass-family)，我们独家的[**NFTs**](https://opensea.io/collection/the-peass-family)收藏品
* **加入** 💬 [**Discord群**](https://discord.gg/hRep4RUj7f) 或 [**电报群**](https://t.me/peass) 或在**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**上关注**我们。
* 通过向[**HackTricks**](https://github.com/carlospolop/hacktricks)和[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github仓库提交PR来分享您的黑客技巧。

</details>

## Apple专有文件系统（APFS）

**Apple文件系统（APFS）**是一种现代文件系统，旨在取代分层文件系统加强版（HFS+）。其开发是为了满足**改进性能、安全性和效率**的需求。

APFS的一些显著特点包括：

1. **空间共享**：APFS允许多个卷**共享单个物理设备上的相同底层空闲存储空间**。这使得空间利用更加高效，因为卷可以动态增长和收缩，无需手动调整大小或重新分区。
1. 这意味着，与文件磁盘中的传统分区相比，**在APFS中，不同分区（卷）共享所有磁盘空间**，而常规分区通常具有固定大小。
2. **快照**：APFS支持**创建快照**，这些快照是**只读**的，是文件系统的特定时间点实例。快照可以实现高效备份和轻松系统回滚，因为它们消耗的额外存储空间很少，可以快速创建或还原。
3. **克隆**：APFS可以**创建共享与原始文件相同存储空间的文件或目录克隆**，直到克隆或原始文件被修改为止。此功能提供了一种有效的方式来创建文件或目录的副本，而无需复制存储空间。
4. **加密**：APFS**原生支持全盘加密**以及按文件和按目录的加密，增强了不同用例下的数据安全性。
5. **崩溃保护**：APFS使用**写时复制元数据方案**，即使在突然断电或系统崩溃的情况下，也能确保文件系统的一致性，降低数据损坏的风险。

总的来说，APFS为Apple设备提供了一个更现代、灵活和高效的文件系统，专注于改进性能、可靠性和安全性。
```bash
diskutil list # Get overview of the APFS volumes
```
## Firmlinks

`Data` 卷被挂载在 **`/System/Volumes/Data`**（您可以使用 `diskutil apfs list` 命令来检查）。

可以在 **`/usr/share/firmlinks`** 文件中找到 firmlinks 的列表。
```bash
cat /usr/share/firmlinks
/AppleInternal	AppleInternal
/Applications	Applications
/Library	Library
[...]
```
在**左侧**是**系统卷**上的目录路径，在**右侧**是它在**数据卷**上映射的目录路径。因此，`/library` --> `/system/Volumes/data/library`
