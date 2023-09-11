# macOS AppleFS

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ企業**で働いていますか？ **HackTricksで会社を宣伝**したいですか？または、**最新バージョンのPEASSにアクセスしたり、HackTricksをPDFでダウンロード**したいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見しましょう、私たちの独占的な[**NFT**](https://opensea.io/collection/the-peass-family)のコレクション
* [**公式のPEASS＆HackTricksのグッズ**](https://peass.creator-spring.com)を手に入れましょう
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter**で**フォロー**してください[**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **ハッキングのトリックを共有するには、PRを** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **と** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **に提出してください。**

</details>

## Apple Propietary File System (APFS)

APFS、またはApple File Systemは、Apple Inc.によって開発された現代的なファイルシステムで、古いHierarchical File System Plus (HFS+)を置き換えるために設計されています。APFSは**パフォーマンス、セキュリティ、効率の向上**を重視しています。

APFSのいくつかの注目すべき特徴は次のとおりです：

1. **スペース共有**：APFSは、複数のボリュームが単一の物理デバイス上の**同じ基礎の空きストレージを共有**できるようにします。これにより、ボリュームは手動でのリサイズや再パーティショニングの必要なく、動的に成長および縮小することができます。
1. これは、APFSでは異なるパーティション（ボリューム）がディスクスペースを共有することを意味しますが、通常のパーティションは固定サイズを持っていました。
2. **スナップショット**：APFSは、ファイルシステムの**読み取り専用**の特定時点のインスタンスであるスナップショットの作成をサポートしています。スナップショットは、追加のストレージを最小限に消費し、迅速に作成または元に戻すことができるため、効率的なバックアップとシステムのロールバックを可能にします。
3. **クローン**：APFSは、オリジナルと同じストレージを共有するファイルまたはディレクトリのクローンを作成できます。この機能により、ストレージスペースを複製せずにファイルやディレクトリのコピーを効率的に作成することができます。
4. **暗号化**：APFSは、フルディスク暗号化だけでなく、ファイルごとやディレクトリごとの暗号化も**ネイティブでサポート**しており、さまざまなユースケースでデータのセキュリティを向上させます。
5. **クラッシュ保護**：APFSは、コピー・オン・ライトのメタデータスキームを使用して、突然の停電やシステムクラッシュの場合でもファイルシステムの整合性を確保します。これにより、データの破損のリスクを減らします。

全体的に、APFSはAppleデバイスに対してより現代的で柔軟かつ効率的なファイルシステムを提供し、パフォーマンス、信頼性、セキュリティの向上に重点を置いています。
```bash
diskutil list # Get overview of the APFS volumes
```
## Firmlinks

`Data`ボリュームは**`/System/Volumes/Data`**にマウントされています（`diskutil apfs list`で確認できます）。

ファームリンクのリストは**`/usr/share/firmlinks`**ファイルにあります。
```bash
cat /usr/share/firmlinks
/AppleInternal	AppleInternal
/Applications	Applications
/Library	Library
[...]
```
**左側**には**システムボリューム**上のディレクトリパスがあり、**右側**には**データボリューム**上でのマッピングされたディレクトリパスがあります。したがって、`/library` --> `/system/Volumes/data/library`
