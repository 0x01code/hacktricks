# macOS AppleFS

<details>

<summary><strong>ゼロからヒーローまでAWSハッキングを学ぶ</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS Red Team Expert）</strong></a><strong>！</strong></summary>

HackTricks をサポートする他の方法:

* **HackTricks で企業を宣伝したい** または **HackTricks をPDFでダウンロードしたい場合は** [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) をチェックしてください！
* [**公式PEASS＆HackTricksスワッグ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクションを見つける
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f) に参加するか、[**telegramグループ**](https://t.me/peass) に参加するか、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) をフォローする。
* **ハッキングテクニックを共有するためにPRを送信して** [**HackTricks**](https://github.com/carlospolop/hacktricks) と [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) のgithubリポジトリに。

</details>

## Apple Propietary File System (APFS)

**Apple File System (APFS)** は、Hierarchical File System Plus (HFS+) を置き換えるために設計された現代的なファイルシステムです。その開発は、**改善されたパフォーマンス、セキュリティ、効率性**の必要性によって推進されました。

APFSのいくつかの注目すべき機能には次のものがあります:

1. **スペース共有**: APFSは、1つの物理デバイス上で複数のボリュームが**同じ基礎の空きストレージを共有**できるようにします。これにより、ボリュームは手動でのリサイズや再パーティショニングが必要なく、動的に成長および縮小できるため、効率的なスペース利用が可能となります。
1. これは、通常のパーティションがファイルディスク内の異なるパーティション（ボリューム）がすべてディスクスペースを共有するのに対して、**APFSでは異なるパーティション（ボリューム）がすべてのディスクスペースを共有**することを意味します。
2. **スナップショット**: APFSは**スナップショットの作成**をサポートしており、これはファイルシステムの**読み取り専用**の特定時点のインスタンスです。スナップショットにより、追加のストレージを最小限に消費し、迅速に作成または元に戻すことができるため、効率的なバックアップとシステムのロールバックが可能となります。
3. **クローン**: APFSは、元のファイルが変更されるまで、**同じストレージを共有するファイルまたはディレクトリのクローンを作成**できます。この機能により、ストレージスペースを複製せずにファイルやディレクトリのコピーを効率的に作成する方法が提供されます。
4. **暗号化**: APFSは、完全なディスク暗号化だけでなく、ファイルごとやディレクトリごとの暗号化を**ネイティブでサポート**しており、さまざまなユースケースでデータセキュリティを向上させています。
5. **クラッシュ保護**: APFSは、ファイルシステムの一貫性を確保する**コピー時書き込みメタデータスキーム**を使用しており、突然の停電やシステムクラッシュの場合でもデータの破損のリスクを減らし、ファイルシステムの一貫性を確保します。

全体として、APFSは、Appleデバイス向けにより現代的で柔軟かつ効率的なファイルシステムを提供し、パフォーマンス、信頼性、セキュリティの向上に焦点を当てています。
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
左側には**システムボリューム**上のディレクトリパスがあり、右側には**データボリューム**上でのマッピングされたディレクトリパスがあります。つまり、`/library` --> `/system/Volumes/data/library`
