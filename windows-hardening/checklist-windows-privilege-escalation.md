# チェックリスト - ローカルWindows特権昇格

{% hint style="success" %}
AWSハッキングを学び、実践する:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングを学び、実践する: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksをサポートする</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)を確認してください！
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**Telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォローしてください。**
* **[**HackTricks**](https://github.com/carlospolop/hacktricks)および[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してハッキングトリックを共有してください。**

</details>
{% endhint %}

**Try Hard Security Group**

<figure><img src="../.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

***

### **Windowsローカル特権昇格ベクトルを探すための最良のツール:** [**WinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)

### [システム情報](windows-local-privilege-escalation/#system-info)

* [ ] [**システム情報**](windows-local-privilege-escalation/#system-info)を取得する
* [ ] **カーネル**の[**エクスプロイトをスクリプトで検索**](windows-local-privilege-escalation/#version-exploits)
* [ ] **Googleでカーネルのエクスプロイトを検索する**
* [ ] **searchsploitでカーネルのエクスプロイトを検索する**
* [ ] [**環境変数**](windows-local-privilege-escalation/#environment)に興味深い情報はあるか？
* [ ] [**PowerShellの履歴**](windows-local-privilege-escalation/#powershell-history)にパスワードはあるか？
* [ ] [**インターネット設定**](windows-local-privilege-escalation/#internet-settings)に興味深い情報はあるか？
* [ ] [**ドライブ**](windows-local-privilege-escalation/#drives)は？
* [ ] [**WSUSエクスプロイト**](windows-local-privilege-escalation/#wsus)は？
* [ ] [**AlwaysInstallElevated**](windows-local-privilege-escalation/#alwaysinstallelevated)は？

### [ログ/AV列挙](windows-local-privilege-escalation/#enumeration)

* [ ] [**監査**](windows-local-privilege-escalation/#audit-settings)および[**WEF**](windows-local-privilege-escalation/#wef)設定を確認する
* [ ] [**LAPS**](windows-local-privilege-escalation/#laps)を確認する
* [ ] [**WDigest**](windows-local-privilege-escalation/#wdigest)がアクティブか確認する
* [ ] [**LSA保護**](windows-local-privilege-escalation/#lsa-protection)は？
* [ ] [**Credentials Guard**](windows-local-privilege-escalation/#credentials-guard)[?](windows-local-privilege-escalation/#cached-credentials)
* [ ] [**キャッシュされた資格情報**](windows-local-privilege-escalation/#cached-credentials)は？
* [ ] 何らかの[**AV**](https://github.com/carlospolop/hacktricks/blob/master/windows-hardening/windows-av-bypass/README.md)があるか？
* [ ] [**AppLockerポリシー**](https://github.com/carlospolop/hacktricks/blob/master/windows-hardening/authentication-credentials-uac-and-efs/README.md#applocker-policy)は？
* [ ] [**UAC**](https://github.com/carlospolop/hacktricks/blob/master/windows-hardening/authentication-credentials-uac-and-efs/uac-user-account-control/README.md)は？
* [ ] [**ユーザー特権**](windows-local-privilege-escalation/#users-and-groups)
* [ ] [**現在の**ユーザーの**特権**](windows-local-privilege-escalation/#users-and-groups)を確認する
* [ ] [**特権グループのメンバー**](windows-local-privilege-escalation/#privileged-groups)か？
* [ ] [これらのトークンのいずれかが有効か確認する](windows-local-privilege-escalation/#token-manipulation): **SeImpersonatePrivilege, SeAssignPrimaryPrivilege, SeTcbPrivilege, SeBackupPrivilege, SeRestorePrivilege, SeCreateTokenPrivilege, SeLoadDriverPrivilege, SeTakeOwnershipPrivilege, SeDebugPrivilege** ?
* [ ] [**ユーザーセッション**](windows-local-privilege-escalation/#logged-users-sessions)は？
* [ ] [**ユーザーホーム**](windows-local-privilege-escalation/#home-folders)を確認する（アクセス？）
* [ ] [**パスワードポリシー**](windows-local-privilege-escalation/#password-policy)を確認する
* [ ] [**クリップボードの中身**](windows-local-privilege-escalation/#get-the-content-of-the-clipboard)は？

### [ネットワーク](windows-local-privilege-escalation/#network)

* [ ] [**現在の**ネットワーク**情報**](windows-local-privilege-escalation/#network)を確認する
* [ ] 外部に制限された**隠れたローカルサービス**を確認する

### [実行中のプロセス](windows-local-privilege-escalation/#running-processes)

* [ ] プロセスバイナリの[**ファイルとフォルダの権限**](windows-local-privilege-escalation/#file-and-folder-permissions)
* [ ] [**メモリパスワードマイニング**](windows-local-privilege-escalation/#memory-password-mining)
* [ ] [**安全でないGUIアプリ**](windows-local-privilege-escalation/#insecure-gui-apps)
* [ ] `ProcDump.exe`を介して**興味深いプロセス**で資格情報を盗む？（firefox, chromeなど...）

### [サービス](windows-local-privilege-escalation/#services)

* [ ] [**サービスを変更できるか**？](windows-local-privilege-escalation/#permissions)
* [ ] [**サービスによって実行される**バイナリを**変更できるか**？](windows-local-privilege-escalation/#modify-service-binary-path)
* [ ] [**サービスの**レジストリを**変更できるか**？](windows-local-privilege-escalation/#services-registry-modify-permissions)
* [ ] [**引用符なしのサービス**バイナリ**パスを利用できるか**？](windows-local-privilege-escalation/#unquoted-service-paths)

### [**アプリケーション**](windows-local-privilege-escalation/#applications)

* [ ] **インストールされたアプリケーションの**[**書き込み権限**](windows-local-privilege-escalation/#write-permissions)
* [ ] [**スタートアップアプリケーション**](windows-local-privilege-escalation/#run-at-startup)
* [ ] **脆弱な**[**ドライバー**](windows-local-privilege-escalation/#drivers)

### [DLLハイジャック](windows-local-privilege-escalation/#path-dll-hijacking)

* [ ] **PATH内の任意のフォルダに書き込めるか**？
* [ ] **存在しないDLLを読み込もうとする**既知のサービスバイナリはあるか？
* [ ] **任意のバイナリフォルダに書き込めるか**？

### [ネットワーク](windows-local-privilege-escalation/#network)

* [ ] ネットワークを列挙する（共有、インターフェース、ルート、隣接、...）
* [ ] localhost (127.0.0.1)でリッスンしているネットワークサービスに特に注意を払う

### [Windows資格情報](windows-local-privilege-escalation/#windows-credentials)

* [ ] [**Winlogon**](windows-local-privilege-escalation/#winlogon-credentials)資格情報
* [ ] [**Windows Vault**](windows-local-privilege-escalation/#credentials-manager-windows-vault)の資格情報は使用できるか？
* [ ] 興味深い[**DPAPI資格情報**](windows-local-privilege-escalation/#dpapi)は？
* [ ] 保存された[**Wifiネットワーク**](windows-local-privilege-escalation/#wifi)のパスワードは？
* [ ] [**保存されたRDP接続**](windows-local-privilege-escalation/#saved-rdp-connections)に興味深い情報はあるか？
* [ ] [**最近実行されたコマンド**](windows-local-privilege-escalation/#recently-run-commands)のパスワードは？
* [ ] [**リモートデスクトップ資格情報マネージャー**](windows-local-privilege-escalation/#remote-desktop-credential-manager)のパスワードは？
* [ ] [**AppCmd.exe**が存在する](windows-local-privilege-escalation/#appcmd-exe)？資格情報は？
* [ ] [**SCClient.exe**](windows-local-privilege-escalation/#scclient-sccm)？DLLサイドローディング？

### [ファイルとレジストリ（資格情報）](windows-local-privilege-escalation/#files-and-registry-credentials)

* [ ] **Putty:** [**資格情報**](windows-local-privilege-escalation/#putty-creds) **と** [**SSHホストキー**](windows-local-privilege-escalation/#putty-ssh-host-keys)
* [ ] [**レジストリ内のSSHキー**](windows-local-privilege-escalation/#ssh-keys-in-registry)は？
* [ ] [**無人ファイル**](windows-local-privilege-escalation/#unattended-files)にパスワードは？
* [ ] [**SAM & SYSTEM**](windows-local-privilege-escalation/#sam-and-system-backups)のバックアップは？
* [ ] [**クラウド資格情報**](windows-local-privilege-escalation/#cloud-credentials)は？
* [ ] [**McAfee SiteList.xml**](windows-local-privilege-escalation/#mcafee-sitelist.xml)ファイルは？
* [ ] [**キャッシュされたGPPパスワード**](windows-local-privilege-escalation/#cached-gpp-pasword)は？
* [ ] [**IIS Web構成ファイル**](windows-local-privilege-escalation/#iis-web-config)にパスワードは？
* [ ] [**ウェブログ**](windows-local-privilege-escalation/#logs)に興味深い情報はあるか？
* [ ] ユーザーに[**資格情報を要求する**](windows-local-privilege-escalation/#ask-for-credentials)か？
* [ ] [**ごみ箱内の興味深いファイル**](windows-local-privilege-escalation/#credentials-in-the-recyclebin)は？
* [ ] 他の[**資格情報を含むレジストリ**](windows-local-privilege-escalation/#inside-the-registry)は？
* [ ] [**ブラウザデータ**](windows-local-privilege-escalation/#browsers-history)内（dbs、履歴、ブックマーク、...）は？
* [ ] [**ファイルとレジストリ内の一般的なパスワード検索**](windows-local-privilege-escalation/#generic-password-search-in-files-and-registry)は？
* [ ] パスワードを自動的に検索するための[**ツール**](windows-local-privilege-escalation/#tools-that-search-for-passwords)

### [漏洩したハンドラー](windows-local-privilege-escalation/#leaked-handlers)

* [ ] 管理者によって実行されるプロセスのハンドラーにアクセスできるか？

### [パイプクライアントの偽装](windows-local-privilege-escalation/#named-pipe-client-impersonation)

* [ ] 悪用できるか確認する

**Try Hard Security Group**

<figure><img src="../.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

{% hint style="success" %}
AWSハッキングを学び、実践する:<img src="/.gitbook/assets/arte.png" alt="" data-size="line">[**HackTricks Training AWS Red Team Expert (ARTE)**](https://training.hacktricks.xyz/courses/arte)<img src="/.gitbook/assets/arte.png" alt="" data-size="line">\
GCPハッキングを学び、実践する: <img src="/.gitbook/assets/grte.png" alt="" data-size="line">[**HackTricks Training GCP Red Team Expert (GRTE)**<img src="/.gitbook/assets/grte.png" alt="" data-size="line">](https://training.hacktricks.xyz/courses/grte)

<details>

<summary>HackTricksをサポートする</summary>

* [**サブスクリプションプラン**](https://github.com/sponsors/carlospolop)を確認してください！
* **💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**Telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**をフォローしてください。**
* **[**HackTricks**](https://github.com/carlospolop/hacktricks)および[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出してハッキングトリックを共有してください。**

</details>
{% endhint %}
