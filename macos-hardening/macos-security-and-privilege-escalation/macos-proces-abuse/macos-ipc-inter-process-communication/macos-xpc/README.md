# macOS XPC

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* サイバーセキュリティ会社で働いていますか？ HackTricksであなたの会社を宣伝したいですか？または、最新バージョンのPEASSにアクセスしたり、HackTricksをPDFでダウンロードしたりしたいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を見つけてください、私たちの独占的な[**NFT**](https://opensea.io/collection/the-peass-family)のコレクション
* [**公式のPEASS＆HackTricks swag**](https://peass.creator-spring.com)を手に入れましょう
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter**で[**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**をフォローしてください。**
* **ハッキングのトリックを共有するには、PRを** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **と** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **に提出してください。**

</details>

## 基本情報

XPCは、macOSおよびiOSで使用されるカーネルであるXNUの間の**プロセス間通信**を行うためのフレームワークです。XPCは、システム上の異なるプロセス間で**安全な非同期メソッド呼び出し**を行うためのメカニズムを提供します。これはAppleのセキュリティパラダイムの一部であり、**特権を分離したアプリケーションの作成**を可能にし、各**コンポーネント**が**必要な権限のみ**を持って動作することで、侵害されたプロセスからの潜在的な被害を制限します。

XPCは、同じシステム上で実行される異なるプログラム間でデータを送受信するための一連の方法である**プロセス間通信（IPC）**の形式を使用します。

XPCの主な利点は次のとおりです：

1. **セキュリティ**：作業を異なるプロセスに分割することで、各プロセスに必要な権限のみを付与することができます。これにより、プロセスが侵害された場合でも、被害を最小限に抑えることができます。
2. **安定性**：XPCは、クラッシュを発生したコンポーネントに限定して分離するのに役立ちます。プロセスがクラッシュした場合、システムの他の部分に影響を与えることなく再起動することができます。
3. **パフォーマンス**：XPCにより、異なるタスクを異なるプロセスで同時に実行することが容易になります。

唯一の**欠点**は、アプリケーションを複数のプロセスに分割してXPCを介して通信することは**効率が低下する**ことです。しかし、現在のシステムではほとんど気づかれず、利点の方が優れています。

## アプリケーション固有のXPCサービス

アプリケーションのXPCコンポーネントは、**アプリケーション自体の中にあります**。たとえば、Safariでは、**`/Applications/Safari.app/Contents/XPCServices`**にそれらを見つけることができます。拡張子は**`.xpc`**（例：**`com.apple.Safari.SandboxBroker.xpc`**）であり、**メインバイナリ**もそれと一緒に**バンドル**されています：`/Applications/Safari.app/Contents/XPCServices/com.apple.Safari.SandboxBroker.xpc/Contents/MacOS/com.apple.Safari.SandboxBroker`および`Info.plist: /Applications/Safari.app/Contents/XPCServices/com.apple.Safari.SandboxBroker.xpc/Contents/Info.plist`

XPCコンポーネントは、他のXPCコンポーネントやメインのアプリケーションバイナリとは異なる**エンタイトルメントと特権**を持つ場合があります。ただし、XPCサービスが**Info.plist**ファイルで[**JoinExistingSession**](https://developer.apple.com/documentation/bundleresources/information\_property\_list/xpcservice/joinexistingsession)を「True」に設定されている場合は除きます。この場合、XPCサービスは、それを呼び出したアプリケーションと**同じセキュリティセッションで実行**されます。

XPCサービスは、必要に応じて**launchd**によって**起動**され、すべてのタスクが**完了**した後に**シャットダウン**され、システムリソースを解放します。**アプリケーション固有のXPCコンポーネントは、アプリケーションのみが利用**できるため、潜在的な脆弱性に関連するリスクを低減します。

## システム全体のXPCサービス

システム全体のXPCサービスは、すべてのユーザーがアクセスできます。これらのサービスは、launchdまたはMachタイプであり、**`/System/Library/LaunchDaemons`**、**`/Library/LaunchDaemons`**、**`/System/Library/LaunchAgents`**、または**`/Library/LaunchAgents`**などの指定されたディレクトリにあるplistファイルで**定義する必要があります**。

これらのplistファイルには、サービスの名前を持つ**`MachServices`**キーと、バイナリへのパスを持つ**`Program`**キーがあります：
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
**`LaunchDameons`**内のものはrootで実行されます。したがって、特権を持たないプロセスがこれらのいずれかと通信できれば、特権をエスカレーションすることができます。

## XPCイベントメッセージ

アプリケーションは、異なるイベントメッセージに**サブスクライブ**することができ、そのようなイベントが発生したときに**オンデマンドで起動**することができます。これらのサービスのセットアップは、**`LaunchEvent`**キーを含む**launchd plistファイル**によって行われます。これらのファイルは、前述のものと同じディレクトリにあります。

### XPC接続プロセスのチェック

プロセスがXPC接続を介してメソッドを呼び出そうとするとき、**XPCサービスはそのプロセスが接続を許可されているかどうかをチェックする必要があります**。以下は、そのチェック方法と一般的な落とし穴です:

{% content-ref url="macos-xpc-connecting-process-check/" %}
[macos-xpc-connecting-process-check](macos-xpc-connecting-process-check/)
{% endcontent-ref %}

## XPC認証

Appleはまた、アプリが**いくつかの権限とその取得方法を設定**することも許可しています。したがって、呼び出し元のプロセスがこれらの権限を持っている場合、XPCサービスからのメソッドの呼び出しが**許可されます**:

{% content-ref url="macos-xpc-authorization.md" %}
[macos-xpc-authorization.md](macos-xpc-authorization.md)
{% endcontent-ref %}

## XPCスニッファ

XPCメッセージをスニッフするには、[**xpcspy**](https://github.com/hot3eed/xpcspy)を使用することができます。これは**Frida**を使用しています。
```bash
# Install
pip3 install xpcspy
pip3 install xpcspy --no-deps # To not make xpcspy install Frida 15 and downgrade your Frida installation

# Start sniffing
xpcspy -U -r -W <bundle-id>
## Using filters (i: for input, o: for output)
xpcspy -U <prog-name> -t 'i:com.apple.*' -t 'o:com.apple.*' -r
```
## Cコードの例

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
```c
#include <stdio.h>
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
{% endtab %}

{% tab title="xpc_server.c" %}
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
{% tab title="xyz.hacktricks.service.plist" %}xyz.hacktricks.service.plistファイルは、macOSでXPCサービスを作成するために使用されるプロパティリストファイルです。このファイルには、サービスの設定と実行に関する情報が含まれています。

以下は、xyz.hacktricks.service.plistファイルの例です。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>xyz.hacktricks.service</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/bin/python</string>
        <string>/path/to/xyz.hacktricks.service.py</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/var/log/xyz.hacktricks.service.log</string>
    <key>StandardErrorPath</key>
    <string>/var/log/xyz.hacktricks.service.log</string>
</dict>
</plist>
```

この例では、`xyz.hacktricks.service`というラベルのXPCサービスが定義されています。サービスは`/usr/bin/python`を実行し、`/path/to/xyz.hacktricks.service.py`スクリプトを引数として渡します。`RunAtLoad`と`KeepAlive`キーは、サービスが起動時に実行されることと、異常終了時に再起動されることを示しています。`StandardOutPath`と`StandardErrorPath`キーは、ログファイルのパスを指定します。

このプロパティリストファイルを使用して、macOSでXPCサービスを作成および管理することができます。{% endtab %}
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
## Objective-Cのコード例

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
{% tab title="xyz.hacktricks.svcoc.plist" %}xyz.hacktricks.svcoc.plistファイルは、macOSでXPCサービスを作成するために使用されるプロパティリスト（plist）ファイルです。XPCサービスは、異なるプロセス間での通信を可能にするために使用されます。このファイルには、XPCサービスの設定とプロパティが含まれています。

XPCサービスは、セキュリティと特権エスカレーションの観点から重要です。悪意のあるユーザーがXPCサービスを悪用することで、システムに深刻な影響を与える可能性があります。

xyz.hacktricks.svcoc.plistファイルを分析することで、XPCサービスの設定とプロパティを理解し、セキュリティ上の脆弱性を特定することができます。これにより、システムのハードニングとプライバシーの向上が可能になります。

このファイルを分析する際には、潜在的な脆弱性やセキュリティ上の問題を特定するために、XPCサービスの動作と関連する情報を詳細に調査する必要があります。{% endtab %}
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
## Dylbコード内のクライアント

The client code inside the Dylb is responsible for establishing a connection with the server and sending requests. It is an essential component of the inter-process communication (IPC) mechanism used in macOS.

Dylbコード内のクライアントは、サーバーとの接続を確立し、リクエストを送信する責任を持ちます。これは、macOSで使用されるプロセス間通信（IPC）メカニズムの重要なコンポーネントです。

To interact with the server, the client uses the XPC (eXtensible Procedure Call) framework provided by macOS. XPC allows processes to communicate with each other securely and efficiently.

サーバーとのやり取りには、macOSが提供するXPC（eXtensible Procedure Call）フレームワークをクライアントが使用します。XPCを使用することで、プロセス間で安全かつ効率的に通信することができます。

The client code typically includes the following steps:

クライアントコードには通常、次の手順が含まれます。

1. Importing the necessary XPC headers and frameworks.

   必要なXPCヘッダーとフレームワークをインポートする。

2. Creating an XPC connection using `xpc_connection_create()`.

   `xpc_connection_create()`を使用してXPC接続を作成する。

3. Setting up event handlers for the connection using `xpc_connection_set_event_handler()`.

   `xpc_connection_set_event_handler()`を使用して接続のイベントハンドラを設定する。

4. Sending requests to the server using `xpc_connection_send_message()`.

   `xpc_connection_send_message()`を使用してサーバーにリクエストを送信する。

5. Handling responses from the server in the event handler.

   イベントハンドラでサーバーからのレスポンスを処理する。

6. Releasing the XPC connection using `xpc_release()` when done.

   終了時に`xpc_release()`を使用してXPC接続を解放する。

By understanding the client code inside the Dylb, you can gain insights into how the communication between processes is established and how requests are sent and received. This knowledge can be valuable for analyzing and securing inter-process communication in macOS.

Dylb内のクライアントコードを理解することで、プロセス間の通信がどのように確立され、リクエストがどのように送受信されるかについての洞察を得ることができます。この知識は、macOSにおけるプロセス間通信の分析とセキュリティ確保に役立ちます。
```objectivec
// gcc -dynamiclib -framework Foundation oc_xpc_client.m -o oc_xpc_client.dylib
// gcc injection example:
// DYLD_INSERT_LIBRARIES=oc_xpc_client.dylib /path/to/vuln/bin

#import <Foundation/Foundation.h>

@protocol MyXPCProtocol
- (void)sayHello:(NSString *)some_string withReply:(void (^)(NSString *))reply;
@end

__attribute__((constructor))
static void customConstructor(int argc, const char **argv)
{
NSString*  _serviceName = @"xyz.hacktricks.svcoc";

NSXPCConnection* _agentConnection = [[NSXPCConnection alloc] initWithMachServiceName:_serviceName options:4096];

[_agentConnection setRemoteObjectInterface:[NSXPCInterface interfaceWithProtocol:@protocol(MyXPCProtocol)]];

[_agentConnection resume];

[[_agentConnection remoteObjectProxyWithErrorHandler:^(NSError* error) {
(void)error;
NSLog(@"Connection Failure");
}] sayHello:@"Hello, Server!" withReply:^(NSString *response) {
NSLog(@"Received response: %@", response);
}    ];
NSLog(@"Done!");

return;
}
```
<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* **サイバーセキュリティ企業**で働いていますか？ **HackTricksで会社を宣伝**したいですか？または、**最新バージョンのPEASSにアクセスしたり、HackTricksをPDFでダウンロード**したいですか？[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を見つけてください。独占的な[**NFT**](https://opensea.io/collection/the-peass-family)のコレクションです。
* [**公式のPEASS＆HackTricksのグッズ**](https://peass.creator-spring.com)を手に入れましょう。
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter**で**フォロー**してください[**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **ハッキングのトリックを共有するには、PRを** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **と** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **に提出してください。**

</details>
