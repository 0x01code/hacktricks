# Seccomp

<details>

<summary><strong>AWSハッキングをゼロからヒーローまで学ぶには</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>をご覧ください！</strong></summary>

HackTricksをサポートする他の方法:

* **HackTricksにあなたの会社を広告したい場合**や**HackTricksをPDFでダウンロードしたい場合**は、[**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS & HackTricksグッズ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクションをご覧ください
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)や[**telegramグループ**](https://t.me/peass)に**参加する**か、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)を**フォローしてください。**
* [**HackTricks**](https://github.com/carlospolop/hacktricks)と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のgithubリポジトリにPRを提出して、あなたのハッキングのコツを**共有してください。**

</details>

## 基本情報

**Seccomp**（セキュアコンピューティングモード）は、要約すると、**システムコールフィルター**として機能するLinuxカーネルの機能です。
Seccompには2つのモードがあります。

**seccomp**（セキュアコンピューティングモードの略）は、**Linux** **カーネル**のコンピュータセキュリティ機能です。seccompを使用すると、プロセスは"セキュア"な状態に一方向の遷移を行うことができ、**すでに開かれている**ファイルディスクリプタに対して`exit()`、`sigreturn()`、`read()`、`write()`のシステムコール以外は実行できません。他のシステムコールを試みた場合、**カーネル**はプロセスをSIGKILLまたはSIGSYSで**終了**させます。この意味で、システムのリソースを仮想化するのではなく、プロセスを完全に隔離します。

seccompモードは、`PR_SET_SECCOMP`引数を使用して`prctl(2)`システムコールを介して**有効にされます**、または（Linuxカーネル3.17以降）`seccomp(2)`システムコールを介して有効にされます。以前は、`/proc/self/seccomp`というファイルに書き込むことでseccompモードを有効にしていましたが、`prctl()`を使用する方法に置き換えられました。一部のカーネルバージョンでは、seccompは`RDTSC` x86命令を無効にします。これは、電源オン以降の経過プロセッササイクル数を返すもので、高精度のタイミングに使用されます。

**seccomp-bpf**は、Berkeley Packet Filterルールを使用して実装された設定可能なポリシーを使用してシステムコールをフィルタリングすることを可能にするseccompの拡張です。これは、Chrome OSとLinuxのGoogle Chrome/Chromiumウェブブラウザだけでなく、OpenSSHやvsftpdによって使用されています。（この点で、seccomp-bpfは、Linuxではもはやサポートされていないように見える古いsystraceと同様の機能を達成しますが、より柔軟性があり、パフォーマンスが高いです。）

### **オリジナル/ストリクトモード**

このモードでは、Seccompはシステムコール`exit()`、`sigreturn()`、`read()`、`write()`のみを**許可します**。これらはすでに開かれているファイルディスクリプタに対してのみです。他のシステムコールが行われた場合、プロセスはSIGKILLを使用して終了されます。

{% code title="seccomp_strict.c" %}
```c
#include <fcntl.h>
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <linux/seccomp.h>
#include <sys/prctl.h>

//From https://sysdig.com/blog/selinux-seccomp-falco-technical-discussion/
//gcc seccomp_strict.c -o seccomp_strict

int main(int argc, char **argv)
{
int output = open("output.txt", O_WRONLY);
const char *val = "test";

//enables strict seccomp mode
printf("Calling prctl() to set seccomp strict mode...\n");
prctl(PR_SET_SECCOMP, SECCOMP_MODE_STRICT);

//This is allowed as the file was already opened
printf("Writing to an already open file...\n");
write(output, val, strlen(val)+1);

//This isn't allowed
printf("Trying to open file for reading...\n");
int input = open("output.txt", O_RDONLY);

printf("You will not see this message--the process will be killed first\n");
}
```
{% endcode %}

### Seccomp-bpf

このモードでは、Berkeley Packet Filterルールを使用して実装された設定可能なポリシーを使用して**システムコールのフィルタリング**が可能です。

{% code title="seccomp_bpf.c" %}
```c
#include <seccomp.h>
#include <unistd.h>
#include <stdio.h>
#include <errno.h>

//https://security.stackexchange.com/questions/168452/how-is-sandboxing-implemented/175373
//gcc seccomp_bpf.c -o seccomp_bpf -lseccomp

void main(void) {
/* initialize the libseccomp context */
scmp_filter_ctx ctx = seccomp_init(SCMP_ACT_KILL);

/* allow exiting */
printf("Adding rule : Allow exit_group\n");
seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(exit_group), 0);

/* allow getting the current pid */
//printf("Adding rule : Allow getpid\n");
//seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(getpid), 0);

printf("Adding rule : Deny getpid\n");
seccomp_rule_add(ctx, SCMP_ACT_ERRNO(EBADF), SCMP_SYS(getpid), 0);
/* allow changing data segment size, as required by glibc */
printf("Adding rule : Allow brk\n");
seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(brk), 0);

/* allow writing up to 512 bytes to fd 1 */
printf("Adding rule : Allow write upto 512 bytes to FD 1\n");
seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(write), 2,
SCMP_A0(SCMP_CMP_EQ, 1),
SCMP_A2(SCMP_CMP_LE, 512));

/* if writing to any other fd, return -EBADF */
printf("Adding rule : Deny write to any FD except 1 \n");
seccomp_rule_add(ctx, SCMP_ACT_ERRNO(EBADF), SCMP_SYS(write), 1,
SCMP_A0(SCMP_CMP_NE, 1));

/* load and enforce the filters */
printf("Load rules and enforce \n");
seccomp_load(ctx);
seccomp_release(ctx);
//Get the getpid is denied, a weird number will be returned like
//this process is -9
printf("this process is %d\n", getpid());
}
```
{% endcode %}

## DockerにおけるSeccomp

**Seccomp-bpf**は、コンテナからの**syscalls**を効果的に制限し、攻撃面を減少させるために**Docker**によってサポートされています。**デフォルトでブロックされるsyscalls**は[https://docs.docker.com/engine/security/seccomp/](https://docs.docker.com/engine/security/seccomp/)で確認でき、**デフォルトのseccompプロファイル**はこちらで見つけることができます[https://github.com/moby/moby/blob/master/profiles/seccomp/default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json)。\
異なるseccompポリシーを使用してdockerコンテナを実行するには：
```bash
docker run --rm \
-it \
--security-opt seccomp=/path/to/seccomp/profile.json \
hello-world
```
例えば、あるコンテナが`uname`のような**syscall**を実行するのを**禁止**したい場合、[https://github.com/moby/moby/blob/master/profiles/seccomp/default.json](https://github.com/moby/moby/blob/master/profiles/seccomp/default.json) からデフォルトプロファイルをダウンロードし、リストから`uname`文字列を**削除する**だけです。\
**あるバイナリがdockerコンテナ内で動作しないようにする**ためには、straceを使用してバイナリが使用しているsyscallをリストアップし、それらを禁止することができます。\
以下の例では、`uname`の**syscall**が発見されています：
```bash
docker run -it --security-opt seccomp=default.json modified-ubuntu strace uname
```
{% hint style="info" %}
**Dockerを使用してアプリケーションを起動するだけの場合**、**`strace`** でプロファイリングし、必要なシステムコールのみを許可することができます。
{% endhint %}

### Seccompポリシーの例

Seccomp機能を説明するために、以下のように“chmod”システムコールを無効にするSeccompプロファイルを作成しましょう。
```json
{
"defaultAction": "SCMP_ACT_ALLOW",
"syscalls": [
{
"name": "chmod",
"action": "SCMP_ACT_ERRNO"
}
]
}
```
上記のプロファイルでは、デフォルトアクションを「allow」と設定し、「chmod」を無効にするブラックリストを作成しました。より安全にするために、デフォルトアクションをdropに設定し、システムコールを選択的に有効にするホワイトリストを作成できます。
以下の出力は、seccompプロファイルで無効にされているため、「chmod」コールがエラーを返していることを示しています。
```bash
$ docker run --rm -it --security-opt seccomp:/home/smakam14/seccomp/profile.json busybox chmod 400 /etc/hosts
chmod: /etc/hosts: Operation not permitted
```
以下の出力は、「docker inspect」がプロファイルを表示していることを示しています：
```json
"SecurityOpt": [
"seccomp:{\"defaultAction\":\"SCMP_ACT_ALLOW\",\"syscalls\":[{\"name\":\"chmod\",\"action\":\"SCMP_ACT_ERRNO\"}]}"
],
```
### Dockerで無効にする

フラグを使用してコンテナを起動します: **`--security-opt seccomp=unconfined`**

Kubernetes 1.19から、**すべてのPodに対してseccompがデフォルトで有効になっています**。しかし、Podに適用されるデフォルトのseccompプロファイルは、コンテナランタイム（例えば、Docker、containerd）によって**提供される"RuntimeDefault"プロファイルです**。"RuntimeDefault"プロファイルは、ほとんどのシステムコールを許可しつつ、危険と考えられるものやコンテナに一般的に必要ではないものをいくつかブロックします。

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)で<strong>AWSハッキングをゼロからヒーローまで学ぶ</strong></a><strong>!</strong></summary>

HackTricksをサポートする他の方法:

* **HackTricksにあなたの会社を広告したい**、または**HackTricksをPDFでダウンロードしたい**場合は、[**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS & HackTricksグッズ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクションをチェックする
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)に**参加する**か、[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)で**フォローする**。
* [**HackTricks**](https://github.com/carlospolop/hacktricks)と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のgithubリポジトリにPRを提出して、あなたのハッキングのコツを**共有する**。

</details>
