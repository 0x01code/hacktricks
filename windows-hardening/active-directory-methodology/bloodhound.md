# BloodHound & 他のAD Enumツール

<details>

<summary><strong>htARTE（HackTricks AWS Red Team Expert）</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>でAWSハッキングをゼロからヒーローまで学ぶ</strong></a><strong>！</strong></summary>

* **サイバーセキュリティ企業**で働いていますか？ **HackTricksで会社を宣伝**してみたいですか？または、**PEASSの最新バージョンにアクセスしたり、HackTricksをPDFでダウンロード**したいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[NFTs](https://opensea.io/collection/the-peass-family)のコレクションを見つけます
* [**公式PEASS＆HackTricksスワッグ**](https://peass.creator-spring.com)を手に入れます
* **[💬](https://emojipedia.org/speech-balloon/) Discordグループ**に**参加**するか、[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter**で私をフォローする🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**。**
* **ハッキングトリックを共有するために、[hacktricksリポジトリ](https://github.com/carlospolop/hacktricks)と[hacktricks-cloudリポジトリ](https://github.com/carlospolop/hacktricks-cloud)にPRを提出してください。**

</details>

## AD Explorer

[AD Explorer](https://docs.microsoft.com/en-us/sysinternals/downloads/adexplorer) はSysinternal Suiteから：

> 高度なActive Directory（AD）ビューアーおよびエディター。AD Explorerを使用して、ADデータベースを簡単にナビゲートし、お気に入りの場所を定義し、ダイアログボックスを開かずにオブジェクトのプロパティと属性を表示し、アクセス許可を編集し、オブジェクトのスキーマを表示し、保存および再実行できる複雑な検索を実行できます。

### スナップショット

AD ExplorerはADのスナップショットを作成できるため、オフラインで確認できます。\
オフラインで脆弱性を発見したり、ADデータベースの異なる状態を比較したりするために使用できます。

ADのスナップショットを取るには、接続するためのユーザー名、パスワード、および方向が必要です。

ADのスナップショットを取るには、`File` --> `Create Snapshot`に移動し、スナップショットの名前を入力します。

## ADRecon

[**ADRecon**](https://github.com/adrecon/ADRecon) は、AD環境からさまざまなアーティファクトを抽出して組み合わせるツールです。情報は、**特別にフォーマットされた**Microsoft Excel **レポート**に表示され、分析を容易にし、対象のAD環境の現在の状態の包括的なイメージを提供するためのメトリクス付きのサマリービューが含まれています。
```bash
# Run it
.\ADRecon.ps1
```
## BloodHound

From [https://github.com/BloodHoundAD/BloodHound](https://github.com/BloodHoundAD/BloodHound)

> BloodHoundは、[Linkurious](http://linkurio.us/)をベースに構築された、[Electron](http://electron.atom.io/)でコンパイルされた、C#データコレクターによって供給されるNeo4jデータベースを使用する、シングルページのJavascript Webアプリケーションです。

BloodHoundは、グラフ理論を使用して、Active DirectoryやAzure環境内の隠れた関係や意図しない関係を明らかにします。攻撃者はBloodHoundを使用して、通常素早く特定することが不可能である高度に複雑な攻撃経路を簡単に特定できます。防御者はBloodHoundを使用して、同じ攻撃経路を特定し排除することができます。青チームと赤チームの両方が、Active DirectoryやAzure環境内の特権関係をより深く理解するためにBloodHoundを使用できます。

したがって、[Bloodhound](https://github.com/BloodHoundAD/BloodHound)は、ドメインを自動的に列挙し、すべての情報を保存し、特権昇格経路を見つけ、グラフを使用してすべての情報を表示できる素晴らしいツールです。

Bloodhoundは、**インジェスタ**と**可視化アプリケーション**の2つの主要な部分で構成されています。

**インジェスタ**は、**ドメインを列挙し、すべての情報を抽出**するために使用されます。

**可視化アプリケーションはneo4jを使用**して、情報がどのように関連しているかを示し、ドメイン内で特権を昇格させるさまざまな方法を示します。

### インストール
BloodHound CEの作成後、プロジェクト全体がDockerを使用した利便性のために更新されました。始める最も簡単な方法は、事前に構成されたDocker Compose構成を使用することです。

1. Docker Composeをインストールします。これは[Docker Desktop](https://www.docker.com/products/docker-desktop/)のインストールに含まれるはずです。
2. 実行:
```
curl -L https://ghst.ly/getbhce | docker compose -f - up
```
3. Docker Composeのターミナル出力からランダムに生成されたパスワードを見つけます。
4. ブラウザで、http://localhost:8080/ui/login に移動します。ユーザー名をadmin、ログからランダムに生成されたパスワードでログインします。

その後、ランダムに生成されたパスワードを変更する必要があり、新しいインターフェースが準備され、そこから直接インジェスタをダウンロードできます。

### SharpHound

いくつかのオプションがありますが、ドメインに参加しているPCからSharpHoundを実行し、現在のユーザーを使用してすべての情報を抽出したい場合は、次のようにします:
```
./SharpHound.exe --CollectionMethods All
Invoke-BloodHound -CollectionMethod All
```
> **CollectionMethod** について詳しくは、[こちら](https://support.bloodhoundenterprise.io/hc/en-us/articles/17481375424795-All-SharpHound-Community-Edition-Flags-Explained) を参照してください。

異なる資格情報を使用して SharpHound を実行したい場合は、CMD netonly セッションを作成し、そこから SharpHound を実行できます。
```
runas /netonly /user:domain\user "powershell.exe -exec bypass"
```
[**Bloodhoundについて詳しくはired.teamをご覧ください。**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-with-bloodhound-on-kali-linux)


## Group3r

[**Group3r**](https://github.com/Group3r/Group3r)は、Active Directoryに関連する**グループポリシー**の**脆弱性**を見つけるためのツールです。\
**ドメイン内のホスト**から**任意のドメインユーザ**を使用して**group3rを実行する**必要があります。
```bash
group3r.exe -f <filepath-name.log>
# -s sends results to stdin
# -f send results to file
```
## PingCastle

[**PingCastle**](https://www.pingcastle.com/documentation/) **はAD環境のセキュリティポストを評価**し、グラフ付きの**レポート**を提供します。

実行するには、バイナリ`PingCastle.exe`を実行し、**インタラクティブセッション**を開始してオプションのメニューを表示します。使用するデフォルトオプションは**`healthcheck`**で、これにより**ドメイン**の基本的な**概要**が確立され、**設定ミス**や**脆弱性**が見つかります。&#x20;
