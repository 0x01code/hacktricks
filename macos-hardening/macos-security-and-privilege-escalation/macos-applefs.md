# macOS AppleFS

<details>

<summary><strong>AWSハッキングをゼロからヒーローまで学ぶには</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>！</strong></summary>

HackTricksをサポートする他の方法:

* **HackTricksにあなたの会社を広告したい**、または**HackTricksをPDFでダウンロードしたい**場合は、[**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS & HackTricksグッズ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクションをチェックする
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)に**参加する**か、[**テレグラムグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)を**フォローする**。
* [**HackTricks**](https://github.com/carlospolop/hacktricks)と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のgithubリポジトリにPRを提出して、あなたのハッキングのコツを共有する。

</details>

## Apple独自のファイルシステム (APFS)

APFS、またはApple File Systemは、Apple Inc.によって開発された現代的なファイルシステムで、**パフォーマンス、セキュリティ、効率の向上**に重点を置いて、古い階層型ファイルシステムプラス(HFS+)を置き換えるために設計されました。

APFSの注目すべき特徴には以下のものがあります:

1. **スペース共有**: APFSは、単一の物理デバイス上の**同じ基盤となる空き容量を複数のボリュームで共有する**ことを可能にします。これにより、手動でのリサイズや再パーティショニングなしにボリュームが動的に拡大・縮小できるため、より効率的なスペース利用が可能になります。
1. これは、従来のファイルディスクのパーティションと比較して、**APFSでは異なるパーティション（ボリューム）が全てのディスクスペースを共有する**ことを意味し、通常のパーティションは固定サイズを持っていました。
2. **スナップショット**: APFSは**スナップショットの作成**をサポートしており、これは**読み取り専用**の、ファイルシステムの時点のインスタンスです。スナップショットは、追加のストレージをほとんど消費せず、迅速に作成または復元できるため、効率的なバックアップと簡単なシステムロールバックを可能にします。
3. **クローン**: APFSは、元のファイルが変更されるまで、**同じストレージを共有するファイルまたはディレクトリのクローンを作成**できます。この機能は、ストレージスペースを複製することなく、ファイルやディレクトリのコピーを効率的に作成する方法を提供します。
4. **暗号化**: APFSは**フルディスク暗号化**をネイティブにサポートするとともに、ファイルごと、ディレクトリごとの暗号化もサポートし、さまざまな使用状況におけるデータセキュリティを強化します。
5. **クラッシュ保護**: APFSは、突然の電源喪失やシステムクラッシュの場合でもファイルシステムの整合性を保証する**コピー・オン・ライトのメタデータスキーム**を使用しており、データの破損リスクを減らします。

全体として、APFSはAppleデバイスにとってより現代的で柔軟で効率的なファイルシステムを提供し、パフォーマンス、信頼性、セキュリティの向上に焦点を当てています。
```bash
diskutil list # Get overview of the APFS volumes
```
## ファームリンク

`Data` ボリュームは **`/System/Volumes/Data`** にマウントされています（`diskutil apfs list` で確認できます）。

ファームリンクのリストは **`/usr/share/firmlinks`** ファイルで見つけることができます。
```bash
cat /usr/share/firmlinks
/AppleInternal	AppleInternal
/Applications	Applications
/Library	Library
[...]
```
**左**には**System volume**上のディレクトリパスがあり、**右**にはそれが**Data volume**上でマッピングされるディレクトリパスがあります。したがって、`/library` --> `/system/Volumes/data/library`

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)でゼロからヒーローまでのAWSハッキングを学ぶ</strong></summary>

HackTricksをサポートする他の方法:

* **HackTricksにあなたの会社を広告したい**、または**HackTricksをPDFでダウンロードしたい**場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS & HackTricksグッズ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクションをチェックする
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)に**参加する**か、[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)を**フォローする**。
* [**HackTricks**](https://github.com/carlospolop/hacktricks) および [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) githubリポジトリにPRを提出して、あなたのハッキングのコツを**共有する**。

</details>
