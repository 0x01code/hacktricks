# macOS カーネル & システム拡張機能

{% hint style="success" %}
AWSハッキングの学習と実践:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングの学習と実践: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksのサポート</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェック！
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)に参加するか、[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォロー**してください。
* **HackTricks**と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出して、ハッキングテクニックを共有してください。

</details>
{% endhint %}

## XNU カーネル

**macOSのコアはXNU**であり、「X is Not Unix」の略です。このカーネルは基本的に**Machマイクロカーネル**（後述します）と**Berkeley Software Distribution（BSD）**の要素から構成されています。XNUはまた、**I/O Kitというシステムを介してカーネルドライバを提供**します。XNUカーネルはDarwinオープンソースプロジェクトの一部であり、**そのソースコードは自由にアクセスできます**。

セキュリティ研究者やUnix開発者の観点から見ると、**macOS**は、エレガントなGUIと多くのカスタムアプリケーションを備えた**FreeBSD**システムにかなり**似て**いると感じるかもしれません。BSD向けに開発されたほとんどのアプリケーションは、Unixユーザにとって馴染みのあるコマンドラインツールがmacOSにすべて備わっているため、修正を加えることなくmacOSでコンパイルおよび実行できます。ただし、XNUカーネルにはMachが組み込まれているため、従来のUnixライクなシステムとmacOSの間にはいくつかの重要な違いがあり、これらの違いは潜在的な問題を引き起こすか、独自の利点を提供する可能性があります。

XNUのオープンソースバージョン: [https://opensource.apple.com/source/xnu/](https://opensource.apple.com/source/xnu/)

### Mach

Machは**UNIX互換のマイクロカーネル**であり、その主要な設計原則の1つは、**カーネルスペースで実行されるコードの量を最小限に抑え、ファイルシステム、ネットワーキング、I/Oなどの多くの典型的なカーネル機能をユーザレベルのタスクとして実行**することです。

XNUでは、Machは**プロセッサスケジューリング、マルチタスキング、および仮想メモリ管理**など、通常カーネルが処理する多くの重要な低レベル操作を担当しています。

### BSD

XNU **カーネル**はまた、**FreeBSD**プロジェクトから派生したコードのかなりの量を**組み込んで**います。このコードは、**Machと同じアドレス空間内でカーネルの一部として実行**されます。ただし、XNU内のFreeBSDコードは、Machとの互換性を確保するために変更が必要であるため、元のFreeBSDコードとは大きく異なる場合があります。 FreeBSDは、次のような多くのカーネル操作に貢献しています：

* プロセス管理
* シグナル処理
* ユーザーおよびグループ管理を含む基本的なセキュリティメカニズム
* システムコールインフラストラクチャ
* TCP/IPスタックおよびソケット
* ファイアウォールおよびパケットフィルタリング

BSDとMachの相互作用を理解することは複雑であり、それぞれ異なる概念フレームワークを持っているためです。たとえば、BSDはプロセスを基本的な実行単位として使用しますが、Machはスレッドに基づいて動作します。この相違は、XNUにおいて、**各BSDプロセスを1つのMachスレッドを含むMachタスクに関連付ける**ことで調整されます。BSDのfork()システムコールが使用されると、カーネル内のBSDコードは、タスクとスレッド構造を作成するためにMach関数を使用します。

さらに、**MachとBSDはそれぞれ異なるセキュリティモデルを維持**しています：**Mach**のセキュリティモデルは**ポート権限**に基づいており、一方、BSDのセキュリティモデルは**プロセス所有権**に基づいて動作します。これら2つのモデルの相違は、場合によってはローカル特権昇格の脆弱性を引き起こすことがあります。典型的なシステムコールに加えて、**ユーザースペースプログラムがカーネルとやり取りするためのMachトラップ**も存在します。これらの異なる要素が組み合わさり、macOSカーネルの多面的でハイブリッドなアーキテクチャを形成しています。

### I/O Kit - ドライバー

I/O KitはXNUカーネル内のオープンソースのオブジェクト指向**デバイスドライバーフレームワーク**であり、**動的にロードされるデバイスドライバ**を処理します。これにより、多様なハードウェアをサポートするために、カーネルにモジュール化されたコードを即座に追加できます。

{% content-ref url="macos-iokit.md" %}
[macos-iokit.md](macos-iokit.md)
{% endcontent-ref %}

### IPC - プロセス間通信

{% content-ref url="../macos-proces-abuse/macos-ipc-inter-process-communication/" %}
[macos-ipc-inter-process-communication](../macos-proces-abuse/macos-ipc-inter-process-communication/)
{% endcontent-ref %}

### カーネルキャッシュ

**カーネルキャッシュ**は、XNUカーネルの**事前にコンパイルおよびリンクされたバージョン**と、必須のデバイス**ドライバー**および**カーネル拡張機能**を含んでいます。これは**圧縮**形式で保存され、起動プロセス中にメモリに展開されます。カーネルキャッシュにより、カーネルと重要なドライバーの準備完了バージョンが利用可能になるため、起動時にこれらのコンポーネントを動的にロードおよびリンクするために費やされる時間とリソースが削減され、**より高速な起動時間**が実現されます。

iOSでは、**`/System/Library/Caches/com.apple.kernelcaches/kernelcache`**にあり、macOSでは**`find / -name kernelcache 2>/dev/null`**または**`mdfind kernelcache | grep kernelcache`**で見つけることができます。

**`kextstat`**を実行して、ロードされたカーネル拡張機能を確認することができます。

#### IMG4

IMG4ファイル形式は、AppleがiOSおよびmacOSデバイスで使用する**ファームウェアコンポーネント（カーネルキャッシュなど）を安全に**保存および検証するために使用されるコンテナ形式です。IMG4形式には、ヘッダーと複数のタグが含まれており、実際のペイロード（カーネルやブートローダーなど）、署名、および一連のマニフェストプロパティをカプセル化しています。この形式は暗号的な検証をサポートしており、デバイスがファームウェアコンポーネントの真正性と整合性を実行する前に確認できます。

通常、以下のコンポーネントで構成されています：

* **ペイロード（IM4P）**：
* しばしば圧縮されています（LZFSE4、LZSSなど）
* オプションで暗号化されています
* **マニフェスト（IM4M）**：
* 署名を含む
* 追加のキー/値の辞書
* **リストア情報（IM4R）**：
* APNonceとしても知られています
* 一部の更新の再生を防止します
* オプション：通常、これは見つかりません

カーネルキャッシュを展開する:
```bash
# pyimg4 (https://github.com/m1stadev/PyIMG4)
pyimg4 im4p extract -i kernelcache.release.iphone14 -o kernelcache.release.iphone14.e

# img4tool (https://github.com/tihmstar/img4tool
img4tool -e kernelcache.release.iphone14 -o kernelcache.release.iphone14.e
```
#### カーネルキャッシュシンボル

時々、Appleは**シンボル**付きの**カーネルキャッシュ**をリリースします。[https://theapplewiki.com](https://theapplewiki.com/)のリンクをたどることで、いくつかのファームウェアにシンボルが付いたものをダウンロードできます。

### IPSW

これらはAppleの**ファームウェア**で、[**https://ipsw.me/**](https://ipsw.me/)からダウンロードできます。他のファイルの中には**カーネルキャッシュ**が含まれています。\
ファイルを**抽出**するには、単に**解凍**するだけです。

ファームウェアを抽出した後、次のようなファイルが得られます: **`kernelcache.release.iphone14`**。これは**IMG4**形式で、次のコマンドで興味深い情報を抽出できます:

* [**pyimg4**](https://github.com/m1stadev/PyIMG4)

{% code overflow="wrap" %}
```bash
pyimg4 im4p extract -i kernelcache.release.iphone14 -o kernelcache.release.iphone14.e
```
{% endcode %}

* [**img4tool**](https://github.com/tihmstar/img4tool)
```bash
img4tool -e kernelcache.release.iphone14 -o kernelcache.release.iphone14.e
```
以下のコマンドを使用して、抽出されたkernelcacheのシンボルを確認できます: **`nm -a kernelcache.release.iphone14.e | wc -l`**

これにより、**すべての拡張機能**または**興味を持っている1つ**を抽出できます:
```bash
# List all extensions
kextex -l kernelcache.release.iphone14.e
## Extract com.apple.security.sandbox
kextex -e com.apple.security.sandbox kernelcache.release.iphone14.e

# Extract all
kextex_all kernelcache.release.iphone14.e

# Check the extension for symbols
nm -a binaries/com.apple.security.sandbox | wc -l
```
## macOSカーネル拡張機能

macOSは**カーネル拡張機能（.kext）をロードすることに非常に制限的**であり、そのコードが実行される高い特権のためです。実際、デフォルトではほぼ不可能です（バイパスが見つかる限り）。

{% content-ref url="macos-kernel-extensions.md" %}
[macos-kernel-extensions.md](macos-kernel-extensions.md)
{% endcontent-ref %}

### macOSシステム拡張機能

macOSはカーネル拡張機能の代わりにシステム拡張機能を作成しました。これにより、開発者はカーネル拡張機能を使用する必要がなくなり、ユーザーレベルのAPIを介してカーネルとやり取りできます。

{% content-ref url="macos-system-extensions.md" %}
[macos-system-extensions.md](macos-system-extensions.md)
{% endcontent-ref %}

## 参考文献

* [**The Mac Hacker's Handbook**](https://www.amazon.com/-/es/Charlie-Miller-ebook-dp-B004U7MUMU/dp/B004U7MUMU/ref=mt\_other?\_encoding=UTF8\&me=\&qid=)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)

{% hint style="success" %}
AWSハッキングの学習と実践：<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングの学習と実践：<img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksのサポート</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェック！
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)に参加するか、[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)をフォローする。
* ハッキングトリックを共有するために、[**HackTricks**](https://github.com/carlospolop/hacktricks)と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出する。

</details>
{% endhint %}
