# BloodHound & 他のAD Enumツール

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> - <a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ企業**で働いていますか？ **HackTricksで企業を宣伝**してみたいですか？または、**PEASSの最新バージョンにアクセス**したいですか、または**HackTricksをPDFでダウンロード**したいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[NFTs](https://opensea.io/collection/the-peass-family)のコレクションを見つけてください
* [**公式PEASS＆HackTricks swag**](https://peass.creator-spring.com)を手に入れましょう
* **[💬](https://emojipedia.org/speech-balloon/) [Discordグループ](https://discord.gg/hRep4RUj7f)**に参加するか、[telegramグループ](https://t.me/peass)に参加するか、**Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)**をフォロー**してください。
* **ハッキングトリックを共有するには、[hacktricksリポジトリ](https://github.com/carlospolop/hacktricks)と[hacktricks-cloudリポジトリ](https://github.com/carlospolop/hacktricks-cloud)**にPRを提出してください。

</details>

## AD Explorer

[AD Explorer](https://docs.microsoft.com/en-us/sysinternals/downloads/adexplorer) はSysinternal Suiteから：

> 高度なActive Directory（AD）ビューアーおよびエディター。AD Explorerを使用して、ADデータベースを簡単にナビゲートしたり、お気に入りの場所を定義したり、ダイアログボックスを開かずにオブジェクトのプロパティや属性を表示したり、アクセス許可を編集したり、オブジェクトのスキーマを表示したり、保存および再実行できる複雑な検索を実行したりできます。

### スナップショット

AD ExplorerはADのスナップショットを作成してオフラインで確認できます。\
オフラインで脆弱性を発見したり、ADデータベースの異なる状態を比較したりするために使用できます。

ADのスナップショットを取るには、ユーザー名、パスワード、および接続先が必要です。

ADのスナップショットを取るには、`File` --> `Create Snapshot`に移動し、スナップショットの名前を入力します。

## ADRecon

[**ADRecon**](https://github.com/adrecon/ADRecon) は、AD環境からさまざまなアーティファクトを抽出して組み合わせるツールです。情報は、**特別にフォーマットされた**Microsoft Excel **レポート**に表示され、分析を容易にし、対象のAD環境の現在の状態の包括的な画像を提供するためのメトリクス付きのサマリービューが含まれています。
```bash
# Run it
.\ADRecon.ps1
```
## BloodHound

From [https://github.com/BloodHoundAD/BloodHound](https://github.com/BloodHoundAD/BloodHound)

> BloodHoundは、[Linkurious](http://linkurio.us/)をベースに構築された、[Electron](http://electron.atom.io/)でコンパイルされた単一ページのJavascript Webアプリケーションで、C#データコレクターによって供給されるNeo4jデータベースを使用しています。

BloodHoundは、グラフ理論を使用して、Active DirectoryやAzure環境内の隠れている場合が多い関係を明らかにします。攻撃者はBloodHoundを使用して、通常素早く特定することが不可能である高度に複雑な攻撃経路を簡単に特定できます。防御者はBloodHoundを使用して、同じ攻撃経路を特定し排除することができます。青チームと赤チームの両方が、Active DirectoryやAzure環境内の特権関係をより深く理解するためにBloodHoundを使用できます。

したがって、[Bloodhound](https://github.com/BloodHoundAD/BloodHound)は、ドメインを自動的に列挙し、すべての情報を保存し、特権昇格経路を見つけ、グラフを使用してすべての情報を表示できる素晴らしいツールです。

Bloodhoundは、**インジェスタ**と**可視化アプリケーション**の2つの主要な部分で構成されています。

**インジェスタ**は、**ドメインを列挙し、すべての情報を抽出**するために使用されます。

**可視化アプリケーションはneo4jを使用**して、情報がどのように関連しているかを示し、ドメイン内で特権を昇格させるさまざまな方法を示します。

### インストール
BloodHound CEの作成後、プロジェクト全体がDockerを使用した簡単な使用のために更新されました。始める最も簡単な方法は、事前に構成されたDocker Compose構成を使用することです。

1. Docker Composeをインストールします。これは[Docker Desktop](https://www.docker.com/products/docker-desktop/)のインストールに含まれるはずです。
2. 実行：
```
curl -L https://ghst.ly/getbhce | docker compose -f - up
```
3. Docker Composeのターミナル出力からランダムに生成されたパスワードを見つけます。
4. ブラウザで、http://localhost:8080/ui/login に移動します。ユーザー名をadmin、ログからランダムに生成されたパスワードでログインします。

その後、ランダムに生成されたパスワードを変更する必要があり、新しいインターフェースが準備され、そこから直接インジェスタをダウンロードできます。

### SharpHound

いくつかのオプションがありますが、ドメインに参加したPCからSharpHoundを実行し、現在のユーザーを使用してすべての情報を抽出したい場合は、次のようにします:
```
./SharpHound.exe --CollectionMethods All
Invoke-BloodHound -CollectionMethod All
```
> **CollectionMethod**についてやループセッションについては、[こちら](https://support.bloodhoundenterprise.io/hc/en-us/articles/17481375424795-All-SharpHound-Community-Edition-Flags-Explained)で詳細を読むことができます。

異なる資格情報を使用してSharpHoundを実行したい場合は、CMD netonlyセッションを作成し、そこからSharpHoundを実行できます。
```
runas /netonly /user:domain\user "powershell.exe -exec bypass"
```
[**ired.team で Bloodhound について詳しく学ぶ。**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-with-bloodhound-on-kali-linux)


## Group3r

[**Group3r**](https://github.com/Group3r/Group3r) は、Active Directory 関連の **グループポリシー** に存在する **脆弱性** を見つけるためのツールです。\
**任意のドメインユーザ** を使用して、ドメイン内のホストから **group3r を実行する必要があります**。
```bash
group3r.exe -f <filepath-name.log>
# -s sends results to stdin
# -f send results to file
```
## PingCastle

[**PingCastle**](https://www.pingcastle.com/documentation/) **はAD環境のセキュリティポストを評価**し、グラフ付きの**レポート**を提供します。

実行するには、バイナリ`PingCastle.exe`を実行し、**対話型セッション**を開始すると、オプションのメニューが表示されます。使用するデフォルトオプションは**`healthcheck`**で、これにより**ドメイン**の基本的な**概要**が確立され、**設定ミス**や**脆弱性**が見つかります。&#x20;
