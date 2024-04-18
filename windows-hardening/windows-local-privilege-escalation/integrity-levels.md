# Integrity Levels

<details>

<summary><strong>ゼロからヒーローまでAWSハッキングを学ぶ</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS Red Team Expert）</strong></a><strong>！</strong></summary>

HackTricks をサポートする他の方法:

- **HackTricks で企業を宣伝したい**または **HackTricks をPDFでダウンロードしたい**場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
- [**公式PEASS＆HackTricksスワッグ**](https://peass.creator-spring.com)を手に入れる
- [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクションを見つける
- **💬 [Discordグループ](https://discord.gg/hRep4RUj7f)**に参加するか、[telegramグループ](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)をフォローする
- **HackTricks**と**HackTricks Cloud**のGitHubリポジトリにPRを提出して、あなたのハッキングテクニックを共有する

</details>

### [WhiteIntel](https://whiteintel.io)

<figure><img src="/.gitbook/assets/image (1224).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io)は、**ダークウェブ**を活用した検索エンジンで、企業やその顧客が**盗難マルウェア**によって**侵害**されていないかをチェックするための**無料**機能を提供しています。

WhiteIntelの主な目標は、情報窃取マルウェアによるアカウント乗っ取りやランサムウェア攻撃と戦うことです。

彼らのウェブサイトをチェックして、**無料**でエンジンを試すことができます：

{% embed url="https://whiteintel.io" %}

---

## Integrity Levels

Windows Vista以降では、すべての保護されたアイテムには**整合性レベル**タグが付いています。このセットアップでは、ほとんどの場合、ファイルとレジストリキーには「中」の整合性レベルが割り当てられますが、Internet Explorer 7が低い整合性レベルで書き込むことができる特定のフォルダやファイルもあります。標準ユーザーによって開始されたプロセスは通常、中間整合性レベルを持ち、サービスは通常、システム整合性レベルで動作します。高整合性ラベルはルートディレクトリを保護します。

重要なルールの1つは、オブジェクトはオブジェクトのレベルよりも低い整合性レベルのプロセスによって変更されないということです。整合性レベルは次のとおりです：

- **信頼されていない**: このレベルは匿名ログインのプロセス向けです。 %%%例: Chrome%%%
- **低**: 主にインターネットのやり取りに使用され、特にInternet Explorerの保護モードで影響を受ける関連ファイルやプロセス、および**一時インターネットフォルダ**などの特定のフォルダに影響します。低整合性プロセスは、レジストリの書き込みアクセスがないことや、ユーザープロファイルの書き込みアクセスが制限されていることなど、重要な制限に直面します。
- **中**: ほとんどのアクティビティのデフォルトレベルで、標準ユーザーや特定の整合性レベルを持たないオブジェクトに割り当てられます。管理者グループのメンバーでさえ、デフォルトでこのレベルで動作します。
- **高**: 管理者向けに予約されており、高い整合性レベル自体を含む低い整合性レベルのオブジェクトを変更できるようにします。
- **システム**: Windowsカーネルとコアサービスのための最高の操作レベルで、管理者でさえアクセスできないようになっており、重要なシステム機能を保護します。
- **インストーラー**: 他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールできるようにする、他のすべてのオブジェクトをアンインストールでき
```
echo asd >asd.txt
icacls asd.txt
asd.txt BUILTIN\Administrators:(I)(F)
DESKTOP-IDJHTKP\user:(I)(F)
NT AUTHORITY\SYSTEM:(I)(F)
NT AUTHORITY\INTERACTIVE:(I)(M,DC)
NT AUTHORITY\SERVICE:(I)(M,DC)
NT AUTHORITY\BATCH:(I)(M,DC)
```
今、ファイルに最小整合性レベルを**High**に割り当てましょう。これは**管理者として実行されているコンソール**から行う必要があります。**通常のコンソール**は中間整合性レベルで実行されており、オブジェクトに高い整合性レベルを割り当てることが**許可されていません**：
```
icacls asd.txt /setintegritylevel(oi)(ci) High
processed file: asd.txt
Successfully processed 1 files; Failed processing 0 files

C:\Users\Public>icacls asd.txt
asd.txt BUILTIN\Administrators:(I)(F)
DESKTOP-IDJHTKP\user:(I)(F)
NT AUTHORITY\SYSTEM:(I)(F)
NT AUTHORITY\INTERACTIVE:(I)(M,DC)
NT AUTHORITY\SERVICE:(I)(M,DC)
NT AUTHORITY\BATCH:(I)(M,DC)
Mandatory Label\High Mandatory Level:(NW)
```
これが興味深い部分です。ユーザー`DESKTOP-IDJHTKP\user`がファイルに対して**完全な権限**を持っていることがわかります（実際、このユーザーがファイルを作成したユーザーです）、しかし、実装された最小整合性レベルのため、彼はファイルを変更できなくなります（ただし、読むことはできます）。
```
echo 1234 > asd.txt
Access is denied.

del asd.txt
C:\Users\Public\asd.txt
Access is denied.
```
{% hint style="info" %}
**したがって、ファイルが最小整合性レベルを持っている場合、そのファイルを変更するには、少なくともその整合性レベルで実行する必要があります。**
{% endhint %}

### バイナリの整合性レベル

`cmd.exe`のコピーを`C:\Windows\System32\cmd-low.exe`に作成し、**管理者コンソールからその整合性レベルを低に設定しました:**
```
icacls C:\Windows\System32\cmd-low.exe
C:\Windows\System32\cmd-low.exe NT AUTHORITY\SYSTEM:(I)(F)
BUILTIN\Administrators:(I)(F)
BUILTIN\Users:(I)(RX)
APPLICATION PACKAGE AUTHORITY\ALL APPLICATION PACKAGES:(I)(RX)
APPLICATION PACKAGE AUTHORITY\ALL RESTRICTED APP PACKAGES:(I)(RX)
Mandatory Label\Low Mandatory Level:(NW)
```
Now, when I run `cmd-low.exe` it will **run under a low-integrity level** instead of a medium one:

![](<../../.gitbook/assets/image (310).png>)

For curious people, if you assign high integrity level to a binary (`icacls C:\Windows\System32\cmd-high.exe /setintegritylevel high`) it won't run with high integrity level automatically (if you invoke it from a medium integrity level --by default-- it will run under a medium integrity level).

### Integrity Levels in Processes

Not all files and folders have a minimum integrity level, **but all processes are running under an integrity level**. And similar to what happened with the file-system, **if a process wants to write inside another process it must have at least the same integrity level**. This means that a process with low integrity level can’t open a handle with full access to a process with medium integrity level.

Due to the restrictions commented in this and the previous section, from a security point of view, it's always **recommended to run a process in the lower level of integrity possible**.


### [WhiteIntel](https://whiteintel.io)

<figure><img src="/.gitbook/assets/image (1224).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) is a **dark-web** fueled search engine that offers **free** functionalities to check if a company or its customers have been **compromised** by **stealer malwares**.

Their primary goal of WhiteIntel is to combat account takeovers and ransomware attacks resulting from information-stealing malware.

You can check their website and try their engine for **free** at:

{% embed url="https://whiteintel.io" %}


<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Other ways to support HackTricks:

* If you want to see your **company advertised in HackTricks** or **download HackTricks in PDF** Check the [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Get the [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Discover [**The PEASS Family**](https://opensea.io/collection/the-peass-family), our collection of exclusive [**NFTs**](https://opensea.io/collection/the-peass-family)
* **Join the** 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) or the [**telegram group**](https://t.me/peass) or **follow** us on **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Share your hacking tricks by submitting PRs to the** [**HackTricks**](https://github.com/carlospolop/hacktricks) and [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos.

</details>
