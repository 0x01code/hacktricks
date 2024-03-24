# チェックリスト - ローカルWindows特権昇格

<details>

<summary><strong>htARTE（HackTricks AWS Red Team Expert）</strong>を通じて、ゼロからヒーローまでAWSハッキングを学ぶ</summary>

HackTricksをサポートする他の方法：

- **HackTricksで企業を宣伝**したい場合や**HackTricksをPDFでダウンロード**したい場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
- [**公式PEASS＆HackTricksスワッグ**](https://peass.creator-spring.com)を入手する
- [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクションを見つける
- **Discordグループ**に**参加**する💬（https://discord.gg/hRep4RUj7f）または[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter**🐦で**フォロー**する：[**@carlospolopm**](https://twitter.com/hacktricks_live)。
- 自分のハッキングテクニックを共有するために、[**HackTricks**](https://github.com/carlospolop/hacktricks)と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のGitHubリポジトリにPRを提出する

</details>

**Try Hard Security Group**

<figure><img src="../.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

---

### **Windowsローカル特権昇格ベクターを探すための最適なツール:** [**WinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)

### [システム情報](windows-local-privilege-escalation/#system-info)

- [ ] [**システム情報**](windows-local-privilege-escalation/#system-info)を取得する
- [ ] **スクリプトを使用して** **カーネル**の[**脆弱性を検索**](windows-local-privilege-escalation/#version-exploits)
- [ ] Googleを使用してカーネルの脆弱性を検索する
- [ ] searchsploitを使用してカーネルの脆弱性を検索する
- [ ] [**環境変数**](windows-local-privilege-escalation/#environment)に興味深い情報はありますか？
- [ ] [**PowerShell履歴**](windows-local-privilege-escalation/#powershell-history)にパスワードはありますか？
- [ ] [**インターネット設定**](windows-local-privilege-escalation/#internet-settings)に興味深い情報はありますか？
- [**ドライブ**](windows-local-privilege-escalation/#drives)は？
- [**WSUSの脆弱性**](windows-local-privilege-escalation/#wsus)は？
- [**AlwaysInstallElevated**](windows-local-privilege-escalation/#alwaysinstallelevated)は？

### [ログ/AV列挙](windows-local-privilege-escalation/#enumeration)

- [ ] [**監査**](windows-local-privilege-escalation/#audit-settings)と[**WEF**](windows-local-privilege-escalation/#wef)の設定を確認する
- [ ] [**LAPS**](windows-local-privilege-escalation/#laps)を確認する
- [ ] [**WDigest**](windows-local-privilege-escalation/#wdigest)がアクティブかどうかを確認する
- [ ] [**LSA保護**](windows-local-privilege-escalation/#lsa-protection)は？
- [ ] [**Credentials Guard**](windows-local-privilege-escalation/#credentials-guard)は？
- [ ] [**キャッシュされた資格情報**](windows-local-privilege-escalation/#cached-credentials)は？
- [ ] いずれかの[**AV**](windows-av-bypass)があるかどうかを確認する
- [**AppLockerポリシー**](authentication-credentials-uac-and-efs#applocker-policy)は？
- [**UAC**](authentication-credentials-uac-and-efs/uac-user-account-control)は？
- [**ユーザ権限**](windows-local-privilege-escalation/#users-and-groups)は？
- 現在のユーザの[**権限**](windows-local-privilege-escalation/#users-and-groups)を確認する
- 特権のあるグループのメンバーですか？[**有効化されているトークン**](windows-local-privilege-escalation/#token-manipulation)はこれらのいずれかを持っていますか？ **SeImpersonatePrivilege、SeAssignPrimaryPrivilege、SeTcbPrivilege、SeBackupPrivilege、SeRestorePrivilege、SeCreateTokenPrivilege、SeLoadDriverPrivilege、SeTakeOwnershipPrivilege、SeDebugPrivilege**？
- [**ユーザセッション**](windows-local-privilege-escalation/#logged-users-sessions)は？
- [**ユーザのホーム**](windows-local-privilege-escalation/#home-folders)をチェックする（アクセスは？）
- [**パスワードポリシー**](windows-local-privilege-escalation/#password-policy)は？
- [**クリップボードの中身**](windows-local-privilege-escalation/#get-the-content-of-the-clipboard)は？

### [ネットワーク](windows-local-privilege-escalation/#network)

- 現在の[**ネットワーク情報**](windows-local-privilege-escalation/#network)をチェックする
- 外部に制限された**非表示のローカルサービス**をチェックする

### [実行中のプロセス](windows-local-privilege-escalation/#running-processes)

- プロセスのバイナリ[**ファイルとフォルダの権限**](windows-local-privilege-escalation/#file-and-folder-permissions)をチェックする
- [**メモリパスワードマイニング**](windows-local-privilege-escalation/#memory-password-mining)を行う
- [**セキュリティの脆弱なGUIアプリ**](windows-local-privilege-escalation/#insecure-gui-apps)をチェックする
- `ProcDump.exe`を使用して、興味深いプロセスから資格情報を盗むことはできますか？（firefox、chromeなど...）

### [サービス](windows-local-privilege-escalation/#services)

- 任意のサービスを**変更**できますか？（[**権限**](windows-local-privilege-escalation#permissions)）
- 任意のサービスが実行する**バイナリ**を**変更**できますか？（[**サービスバイナリパスの変更**](windows-local-privilege-escalation/#modify-service-binary-path)）
- 任意のサービスの**レジストリ**を**変更**できますか？（[**サービスレジストリの変更権限**](windows-local-privilege-escalation/#services-registry-modify-permissions)）
- いずれかの**未引用サービス**バイナリ**パス**を利用できますか？（[**未引用サービスパス**](windows-local-privilege-escalation/#unquoted-service-paths)）

### [**アプリケーション**](windows-local-privilege-escalation/#applications)

- インストールされたアプリケーションの[**書き込み権限**](windows-local-privilege-escalation/#write-permissions)をチェックする
- [**起動アプリケーション**](windows-local-privilege-escalation/#run-at-startup)をチェックする
- **脆弱な**[**ドライバ**](windows-local-privilege-escalation/#drivers)をチェックする
### [DLL Hijacking](windows-local-privilege-escalation/#path-dll-hijacking)

* [ ] **PATH**内の**任意のフォルダに書き込めます**か？
* [ ] **存在しないDLLを読み込もうとする**既知のサービスバイナリがありますか？
* [ ] **バイナリフォルダに書き込めます**か？

### [ネットワーク](windows-local-privilege-escalation/#network)

* [ ] ネットワークを列挙します（共有、インターフェース、ルート、隣接者など）
* [ ] ローカルホスト（127.0.0.1）でリスニングしているネットワークサービスに特に注意します

### [Windowsの資格情報](windows-local-privilege-escalation/#windows-credentials)

* [ ] [**Winlogon**](windows-local-privilege-escalation/#winlogon-credentials)の資格情報
* [ ] 使用できる[**Windows Vault**](windows-local-privilege-escalation/#credentials-manager-windows-vault)の資格情報はありますか？
* [ ] 興味深い[**DPAPIの資格情報**](windows-local-privilege-escalation/#dpapi)はありますか？
* [ ] 保存された[**Wifiネットワークのパスワード**](windows-local-privilege-escalation/#wifi)はありますか？
* [ ] 保存されたRDP接続に関する興味深い情報はありますか？
* [ ] 最近実行されたコマンドのパスワードはありますか？
* [ ] [**リモートデスクトップ資格情報マネージャー**](windows-local-privilege-escalation/#remote-desktop-credential-manager)のパスワードはありますか？
* [ ] [**AppCmd.exe**が存在しますか](windows-local-privilege-escalation/#appcmd-exe)？資格情報は？
* [ ] [**SCClient.exe**](windows-local-privilege-escalation/#scclient-sccm)はありますか？DLLサイドローディングは？

### [ファイルとレジストリ（資格情報）](windows-local-privilege-escalation/#files-and-registry-credentials)

* [ ] **Putty:** [**資格情報**](windows-local-privilege-escalation/#putty-creds) **および** [**SSHホストキー**](windows-local-privilege-escalation/#putty-ssh-host-keys)
* [ ] レジストリにある[**SSHキー**](windows-local-privilege-escalation/#ssh-keys-in-registry)はありますか？
* [ ] [**無人ファイル**](windows-local-privilege-escalation/#unattended-files)にパスワードはありますか？
* [ ] [**SAM & SYSTEM**](windows-local-privilege-escalation/#sam-and-system-backups)のバックアップはありますか？
* [ ] [**クラウドの資格情報**](windows-local-privilege-escalation/#cloud-credentials)はありますか？
* [ ] [**McAfee SiteList.xml**](windows-local-privilege-escalation/#mcafee-sitelist.xml)ファイルはありますか？
* [ ] [**キャッシュされたGPPパスワード**](windows-local-privilege-escalation/#cached-gpp-pasword)はありますか？
* [ ] [**IIS Web構成ファイル**](windows-local-privilege-escalation/#iis-web-config)にパスワードはありますか？
* [ ] [**Webログ**](windows-local-privilege-escalation/#logs)に興味深い情報はありますか？
* [ ] ユーザーに[**資格情報を要求**](windows-local-privilege-escalation/#ask-for-credentials)したいですか？
* [ ] リサイクルビン内の[**興味深いファイル**](windows-local-privilege-escalation/#credentials-in-the-recyclebin)はありますか？
* [ ] 他の[**資格情報を含むレジストリ**](windows-local-privilege-escalation/#inside-the-registry)はありますか？
* [ ] [**ブラウザデータ**](windows-local-privilege-escalation/#browsers-history)内（データベース、履歴、ブックマークなど）には？
* [**ファイルとレジストリ内の一般的なパスワード検索**](windows-local-privilege-escalation/#generic-password-search-in-files-and-registry)は？
* パスワードを自動的に検索する[**ツール**](windows-local-privilege-escalation/#tools-that-search-for-passwords)は？

### [漏洩したハンドラ](windows-local-privilege-escalation/#leaked-handlers)

* [ ] 管理者によって実行されたプロセスのハンドラにアクセスできますか？

### [パイプクライアントの擬似化](windows-local-privilege-escalation/#named-pipe-client-impersonation)

* [ ] それを悪用できるかどうかを確認します

**Try Hard Security Group**

<figure><img src="../.gitbook/assets/telegram-cloud-document-1-5159108904864449420.jpg" alt=""><figcaption></figcaption></figure>

{% embed url="https://discord.gg/tryhardsecurity" %}

<details>

<summary><strong>htARTE（HackTricks AWS Red Team Expert）</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>でゼロからヒーローまでAWSハッキングを学びましょう</strong></a><strong>！</strong></summary>

HackTricksをサポートする他の方法：

* **HackTricksをPDFでダウンロード**したい場合や**HackTricksで企業を宣伝**したい場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS＆HackTricksスウォッグ**](https://peass.creator-spring.com)を手に入れましょう
* 独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)を収集できる、[**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)や[**telegramグループ**](https://t.me/peass)に**参加**したり、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)で**フォロー**したりしてください。
* **HackTricks**と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks)のGitHubリポジトリにPRを提出して、自分のハッキングトリックを共有してください。

</details>
