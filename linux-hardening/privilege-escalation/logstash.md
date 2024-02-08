<details>

<summary><strong>ゼロからヒーローまでAWSハッキングを学ぶ</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS Red Team Expert）</strong></a><strong>！</strong></summary>

HackTricks をサポートする他の方法:

* **HackTricks で企業を宣伝**したい場合や **HackTricks をPDFでダウンロード**したい場合は [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) をチェックしてください！
* [**公式PEASS＆HackTricksスワッグ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)、当社の独占的な [**NFTs**](https://opensea.io/collection/the-peass-family) コレクションを発見する
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f) に参加するか、[**telegramグループ**](https://t.me/peass) に参加するか、**Twitter** 🐦 で私をフォローする [**@carlospolopm**](https://twitter.com/carlospolopm)**。**
* **ハッキングトリックを共有するために PR を提出して** [**HackTricks**](https://github.com/carlospolop/hacktricks) と [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) の github リポジトリに。

</details>


## Logstash

Logstash は **ログを収集、変換、送信** するために使用されます。これは **パイプライン** として知られるシステムを介して行われます。これらのパイプラインは **input**、**filter**、**output** の段階で構成されます。Logstash が侵害されたマシンで動作する場合、興味深い側面が生じます。

### パイプラインの構成

パイプラインは **/etc/logstash/pipelines.yml** ファイルで構成され、パイプライン構成の場所がリストされています。
```yaml
# Define your pipelines here. Multiple pipelines can be defined.
# For details on multiple pipelines, refer to the documentation:
# https://www.elastic.co/guide/en/logstash/current/multiple-pipelines.html

- pipeline.id: main
path.config: "/etc/logstash/conf.d/*.conf"
- pipeline.id: example
path.config: "/usr/share/logstash/pipeline/1*.conf"
pipeline.workers: 6
```
このファイルでは、パイプライン構成を含む **.conf** ファイルがどこにあるかが明らかにされています。**Elasticsearch出力モジュール**を使用する際、**パイプライン** に **Elasticsearchの資格情報** を含めることが一般的であり、これらの資格情報はしばしば Logstash が Elasticsearch にデータを書き込む必要があるため、広範な権限を持っています。構成パス内のワイルドカードを使用することで、Logstash は指定されたディレクトリ内のすべての一致するパイプラインを実行できます。

### 書き込み可能なパイプラインを通じた特権昇格

特権昇格を試みるには、まず Logstash サービスが実行されているユーザーを特定します。通常は **logstash** ユーザーです。次のいずれかの条件を満たしていることを確認してください:

- パイプライン **.conf** ファイルに **書き込みアクセス権** がある **または**
- **/etc/logstash/pipelines.yml** ファイルがワイルドカードを使用しており、対象のフォルダに書き込むことができる

さらに、次のいずれかの条件を満たしている必要があります:

- Logstash サービスを再起動できる権限がある **または**
- **/etc/logstash/logstash.yml** ファイルに **config.reload.automatic: true** が設定されている

構成内にワイルドカードがある場合、このワイルドカードに一致するファイルを作成することでコマンドを実行できます。例:
```bash
input {
exec {
command => "whoami"
interval => 120
}
}

output {
file {
path => "/tmp/output.log"
codec => rubydebug
}
}
```
ここでは、**interval** は実行頻度（秒単位）を決定します。与えられた例では、**whoami** コマンドが120秒ごとに実行され、その出力は **/tmp/output.log** に向けられます。

**/etc/logstash/logstash.yml** に **config.reload.automatic: true** があると、Logstash は新しいまたは変更されたパイプライン構成を自動的に検出して適用し、再起動を必要としません。ワイルドカードがない場合、既存の構成に変更を加えることはできますが、障害を回避するために注意が必要です。


## 参考文献

* [https://insinuator.net/2021/01/pentesting-the-elk-stack/](https://insinuator.net/2021/01/pentesting-the-elk-stack/)


<details>

<summary><strong>htARTE（HackTricks AWS Red Team Expert）でAWSハッキングをゼロからヒーローまで学ぶ</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS Red Team Expert）</strong></a><strong>!</strong></summary>

HackTricks をサポートする他の方法:

* **HackTricks で企業を宣伝したい** または **HackTricks をPDFでダウンロードしたい** 場合は [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) をチェックしてください！
* [**公式PEASS＆HackTricksスウォッグ**](https://peass.creator-spring.com)を手に入れる
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) を発見し、独占的な [**NFTs**](https://opensea.io/collection/the-peass-family) のコレクションを見つける
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f) に参加するか、[**telegramグループ**](https://t.me/peass) に参加するか、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) をフォローする。
* **HackTricks** と [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) のGitHubリポジトリにPRを提出して、あなたのハッキングテクニックを共有してください。

</details>
