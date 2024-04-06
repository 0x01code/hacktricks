# macOS MIG - Mach Interface Generator

<details>

<summary><strong>ゼロからヒーローまでAWSハッキングを学ぶ</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS Red Team Expert）</strong></a><strong>！</strong></summary>

HackTricks をサポートする他の方法:

* **HackTricks で企業を宣伝したい** または **HackTricks をPDFでダウンロードしたい** 場合は [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) をチェックしてください！
* [**公式PEASS＆HackTricksグッズ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な [**NFTs**](https://opensea.io/collection/the-peass-family) のコレクションを見つける
* **💬** [**Discordグループ**](https://discord.gg/hRep4RUj7f) **または** [**telegramグループ**](https://t.me/peass) **に参加するか、Twitter 🐦** [**@carlospolopm**](https://twitter.com/hacktricks\_live) **をフォローする**
* **ハッキングテクニックを共有するために、** [**HackTricks**](https://github.com/carlospolop/hacktricks) と [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) のGitHubリポジトリにPRを提出する

</details>

MIG は **Mach IPC のコード作成プロセスを簡素化** するために作成されました。基本的には、サーバーとクライアントが指定された定義と通信するために必要なコードを **生成** します。生成されたコードが醜い場合でも、開発者はそれをインポートするだけで、彼のコードは以前よりもはるかにシンプルになります。

### 例

定義ファイルを作成し、この場合は非常に単純な関数を持つもの:

{% code title="myipc.defs" %}
```cpp
subsystem myipc 500; // Arbitrary name and id

userprefix USERPREF;        // Prefix for created functions in the client
serverprefix SERVERPREF;    // Prefix for created functions in the server

#include <mach/mach_types.defs>
#include <mach/std_types.defs>

simpleroutine Subtract(
server_port :  mach_port_t;
n1          :  uint32_t;
n2          :  uint32_t);
```
{% endcode %}

今、migを使用して、互いに通信し合い、Subtract関数を呼び出すためのサーバーおよびクライアントコードを生成します:

```bash
mig -header myipcUser.h -sheader myipcServer.h myipc.defs
```

現在のディレクトリにいくつかの新しいファイルが作成されます。

**`myipcServer.c`** と **`myipcServer.h`** のファイルには、**`SERVERPREFmyipc_subsystem`** 構造体の宣言と定義が含まれており、受信したメッセージIDに基づいて呼び出す関数が基本的に定義されています（開始番号は500と指定されています）:

```c
/* Description of this subsystem, for use in direct RPC */
const struct SERVERPREFmyipc_subsystem SERVERPREFmyipc_subsystem = {
myipc_server_routine,
500, // start ID
501, // end ID
(mach_msg_size_t)sizeof(union __ReplyUnion__SERVERPREFmyipc_subsystem),
(vm_address_t)0,
{
{ (mig_impl_routine_t) 0,
// Function to call
(mig_stub_routine_t) _XSubtract, 3, 0, (routine_arg_descriptor_t)0, (mach_msg_size_t)sizeof(__Reply__Subtract_t)},
}
};
```

次の手順で、MIG インターフェースを生成します。

1. MIG 定義ファイル (myipc.defs) を作成します。
2. MIG コンパイラを使用して、myipc.defs ファイルから myipcServer.c ファイルを生成します。
3. myipcServer.c ファイルをサーバーアプリケーションに組み込みます。
4. サーバーアプリケーションをビルドして実行します。

```c
/* Description of this subsystem, for use in direct RPC */
extern const struct SERVERPREFmyipc_subsystem {
mig_server_routine_t	server;	/* Server routine */
mach_msg_id_t	start;	/* Min routine number */
mach_msg_id_t	end;	/* Max routine number + 1 */
unsigned int	maxsize;	/* Max msg size */
vm_address_t	reserved;	/* Reserved */
struct routine_descriptor	/* Array of routine descriptors */
routine[1];
} SERVERPREFmyipc_subsystem;
```

前の構造に基づいて、**`myipc_server_routine`** 関数は **メッセージ ID** を取得し、適切な呼び出すべき関数を返します:

```c
mig_external mig_routine_t myipc_server_routine
(mach_msg_header_t *InHeadP)
{
int msgh_id;

msgh_id = InHeadP->msgh_id - 500;

if ((msgh_id > 0) || (msgh_id < 0))
return 0;

return SERVERPREFmyipc_subsystem.routine[msgh_id].stub_routine;
}
```

この例では、定義で関数を1つだけ定義しましたが、複数の関数を定義した場合、それらは\*\*`SERVERPREFmyipc_subsystem`\*\*の配列内にあり、最初の関数はID **500**に割り当てられ、2番目の関数はID **501**に割り当てられます...

実際には、この関係を\*\*`myipcServer.h`**の**`subsystem_to_name_map_myipc`\*\*構造体で特定することができます:

```c
#ifndef subsystem_to_name_map_myipc
#define subsystem_to_name_map_myipc \
{ "Subtract", 500 }
#endif
```

```c
mig_external boolean_t myipc_server
(mach_msg_header_t *InHeadP, mach_msg_header_t *OutHeadP)
{
/*
* typedef struct {
* 	mach_msg_header_t Head;
* 	NDR_record_t NDR;
* 	kern_return_t RetCode;
* } mig_reply_error_t;
*/

mig_routine_t routine;

OutHeadP->msgh_bits = MACH_MSGH_BITS(MACH_MSGH_BITS_REPLY(InHeadP->msgh_bits), 0);
OutHeadP->msgh_remote_port = InHeadP->msgh_reply_port;
/* Minimal size: routine() will update it if different */
OutHeadP->msgh_size = (mach_msg_size_t)sizeof(mig_reply_error_t);
OutHeadP->msgh_local_port = MACH_PORT_NULL;
OutHeadP->msgh_id = InHeadP->msgh_id + 100;
OutHeadP->msgh_reserved = 0;

if ((InHeadP->msgh_id > 500) || (InHeadP->msgh_id < 500) ||
	    ((routine = SERVERPREFmyipc_subsystem.routine[InHeadP->msgh_id - 500].stub_routine) == 0)) {
		((mig_reply_error_t *)OutHeadP)->NDR = NDR_record;
((mig_reply_error_t *)OutHeadP)->RetCode = MIG_BAD_ID;
return FALSE;
}
	(*routine) (InHeadP, OutHeadP);
	return TRUE;
}
```

```c
// gcc myipc_server.c myipcServer.c -o myipc_server

#include <stdio.h>
#include <mach/mach.h>
#include <servers/bootstrap.h>
#include "myipcServer.h"

kern_return_t SERVERPREFSubtract(mach_port_t server_port, uint32_t n1, uint32_t n2)
{
printf("Received: %d - %d = %d\n", n1, n2, n1 - n2);
return KERN_SUCCESS;
}

int main() {

mach_port_t port;
kern_return_t kr;

// Register the mach service
kr = bootstrap_check_in(bootstrap_port, "xyz.hacktricks.mig", &port);
if (kr != KERN_SUCCESS) {
printf("bootstrap_check_in() failed with code 0x%x\n", kr);
return 1;
}

// myipc_server is the function that handles incoming messages (check previous exlpanation)
mach_msg_server(myipc_server, sizeof(union __RequestUnion__SERVERPREFmyipc_subsystem), port, MACH_MSG_TIMEOUT_NONE);
}
```



```c
// gcc myipc_client.c myipcUser.c -o myipc_client

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

#include <mach/mach.h>
#include <servers/bootstrap.h>
#include "myipcUser.h"

int main() {

// Lookup the receiver port using the bootstrap server.
mach_port_t port;
kern_return_t kr = bootstrap_look_up(bootstrap_port, "xyz.hacktricks.mig", &port);
if (kr != KERN_SUCCESS) {
printf("bootstrap_look_up() failed with code 0x%x\n", kr);
return 1;
}
printf("Port right name %d\n", port);
USERPREFSubtract(port, 40, 2);
}
```

### バイナリ解析

多くのバイナリが今やMIGを使用してmachポートを公開しているため、**MIGが使用されたことを特定**し、各メッセージIDでMIGが実行する**関数を特定**する方法を知ることが興味深いです。

[**jtool2**](../../macos-apps-inspecting-debugging-and-fuzzing/#jtool2)は、Mach-OバイナリからMIG情報を解析し、メッセージIDを示し、実行する関数を特定できます。

```bash
jtool2 -d __DATA.__const myipc_server | grep MIG
```

**`myipc_server`** 関数は、受信したメッセージ ID に応じて正しい関数を呼び出す機能を担当することが以前に言及されました。ただし、通常はバイナリのシンボル（関数名なし）を持っていないため、**デコンパイルしたものがどのように見えるかを確認する**ことが興味深いです（この関数のコードは公開された関数に独立しています）：

{% tabs %}
{% tab title="myipc_server decompiled 1" %}
<pre class="language-c"><code class="lang-c">int _myipc_server(int arg0, int arg1) {
var_10 = arg0;
var_18 = arg1;
// 適切な関数ポインタを見つけるための初期命令
*(int32_t *)var_18 = *(int32_t *)var_10 &#x26; 0x1f;
*(int32_t *)(var_18 + 0x8) = *(int32_t *)(var_10 + 0x8);
*(int32_t *)(var_18 + 0x4) = 0x24;
*(int32_t *)(var_18 + 0xc) = 0x0;
*(int32_t *)(var_18 + 0x14) = *(int32_t *)(var_10 + 0x14) + 0x64;
*(int32_t *)(var_18 + 0x10) = 0x0;
if (*(int32_t *)(var_10 + 0x14) &#x3C;= 0x1f4 &#x26;&#x26; *(int32_t *)(var_10 + 0x14) >= 0x1f4) {
rax = *(int32_t *)(var_10 + 0x14);
// この関数を識別するのに役立つ sign_extend_64 の呼び出し
// これにより、呼び出す必要のある呼び出しのポインタが rax に格納されます
// アドレス 0x100004040（関数アドレス配列の使用）を確認します
// 0x1f4 = 500（開始 ID）
<strong>            rax = *(sign_extend_64(rax - 0x1f4) * 0x28 + 0x100004040);
</strong>            var_20 = rax;
// もし - そうでなければ、if は false を返し、else は正しい関数を呼び出して true を返します
<strong>            if (rax == 0x0) {
</strong>                    *(var_18 + 0x18) = **_NDR_record;
*(int32_t *)(var_18 + 0x20) = 0xfffffffffffffed1;
var_4 = 0x0;
}
else {
// 2 つの引数を持つ適切な関数を呼び出す計算されたアドレス
<strong>                    (var_20)(var_10, var_18);
</strong>                    var_4 = 0x1;
}
}
else {
*(var_18 + 0x18) = **_NDR_record;
*(int32_t *)(var_18 + 0x20) = 0xfffffffffffffed1;
var_4 = 0x0;
}
rax = var_4;
return rax;
}
</code></pre>
{% endtab %}

{% tab title="myipc_server decompiled 2" %}
これは、異なる Hopper free バージョンでデコンパイルされた同じ関数です：

<pre class="language-c"><code class="lang-c">int _myipc_server(int arg0, int arg1) {
r31 = r31 - 0x40;
saved_fp = r29;
stack[-8] = r30;
var_10 = arg0;
var_18 = arg1;
// 適切な関数ポインタを見つけるための初期命令
*(int32_t *)var_18 = *(int32_t *)var_10 &#x26; 0x1f | 0x0;
*(int32_t *)(var_18 + 0x8) = *(int32_t *)(var_10 + 0x8);
*(int32_t *)(var_18 + 0x4) = 0x24;
*(int32_t *)(var_18 + 0xc) = 0x0;
*(int32_t *)(var_18 + 0x14) = *(int32_t *)(var_10 + 0x14) + 0x64;
*(int32_t *)(var_18 + 0x10) = 0x0;
r8 = *(int32_t *)(var_10 + 0x14);
r8 = r8 - 0x1f4;
if (r8 > 0x0) {
if (CPU_FLAGS &#x26; G) {
r8 = 0x1;
}
}
if ((r8 &#x26; 0x1) == 0x0) {
r8 = *(int32_t *)(var_10 + 0x14);
r8 = r8 - 0x1f4;
if (r8 &#x3C; 0x0) {
if (CPU_FLAGS &#x26; L) {
r8 = 0x1;
}
}
if ((r8 &#x26; 0x1) == 0x0) {
r8 = *(int32_t *)(var_10 + 0x14);
// 0x1f4 = 500（開始 ID）
<strong>                    r8 = r8 - 0x1f4;
</strong>                    asm { smaddl     x8, w8, w9, x10 };
r8 = *(r8 + 0x8);
var_20 = r8;
r8 = r8 - 0x0;
if (r8 != 0x0) {
if (CPU_FLAGS &#x26; NE) {
r8 = 0x1;
}
}
// 前のバージョンと同じ if else
// アドレス 0x100004040（関数アドレス配列の使用）を確認します
<strong>                    if ((r8 &#x26; 0x1) == 0x0) {
</strong><strong>                            *(var_18 + 0x18) = **0x100004000;
</strong>                            *(int32_t *)(var_18 + 0x20) = 0xfffffed1;
var_4 = 0x0;
}
else {
// 関数があるべき計算されたアドレスに呼び出し
<strong>                            (var_20)(var_10, var_18);
</strong>                            var_4 = 0x1;
}
}
else {
*(var_18 + 0x18) = **0x100004000;
*(int32_t *)(var_18 + 0x20) = 0xfffffed1;
var_4 = 0x0;
}
}
else {
*(var_18 + 0x18) = **0x100004000;
*(int32_t *)(var_18 + 0x20) = 0xfffffed1;
var_4 = 0x0;
}
r0 = var_4;
return r0;
}

</code></pre>
{% endtab %}
{% endtabs %}

実際には、**`0x100004000`** 関数に移動すると、**`routine_descriptor`** 構造体の配列が見つかります。構造体の最初の要素は、**関数が実装されているアドレス**であり、**構造体は 0x28 バイト**を取るため、0 バイトから始まる 0x28 バイトごとに 8 バイトを取得し、それが**呼び出される関数のアドレス**になります。

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

このデータは、[**この Hopper スクリプト**](https://github.com/knightsc/hopper/blob/master/scripts/MIG%20Detect.py)を使用して抽出できます。

<details>

<summary><strong>htARTE（HackTricks AWS Red Team Expert）</strong>でゼロからヒーローまでの AWS ハッキングを学びましょう！</summary>

HackTricks をサポートする他の方法：

* HackTricks で **会社を宣伝**したり、**HackTricks を PDF でダウンロード**したりするには、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) をチェックしてください！
* [**公式 PEASS & HackTricks スワッグ**](https://peass.creator-spring.com)を手に入れる
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な [**NFT**](https://opensea.io/collection/the-peass-family)コレクションを見つける

<!---->

* [**Discordグループ**](https://discord.gg/hRep4RUj7f)**に参加するか、**[**telegramグループ**](https://t.me/peass)**に参加するか、Twitterで@carlospolopmをフォローしてください🐦** [**@carlospolopm**](https://twitter.com/hacktricks\_live)。\*\*
* \*\*ハッキングトリックを共有するには、[**HackTricks**](https://github.com/carlospolop/hacktricks)\*\*と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)**のGitHubリポジトリにPRを提出してください。**

</details>
