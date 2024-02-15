# チェックリスト - ローカルWindows特権昇格

<details>

<summary><strong>htARTE（HackTricks AWS Red Team Expert）</strong>で**ゼロからヒーローまでのAWSハッキングを学ぶ**</summary>

HackTricksをサポートする他の方法：

- **HackTricksで企業を宣伝**したい場合や**HackTricksをPDFでダウンロード**したい場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
- [**公式PEASS＆HackTricksスワッグ**](https://peass.creator-spring.com)を入手する
- [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクションを見つける
- **Discordグループ**に**参加**する💬（https://discord.gg/hRep4RUj7f）または[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter**で**フォロー**する🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)。
- **HackTricks**（https://github.com/carlospolop/hacktricks）および**HackTricks Cloud**（https://github.com/carlospolop/hacktricks-cloud）のGitHubリポジトリにPRを提出して、あなたのハッキングテクニックを共有してください。

</details>

### **Windowsローカル特権昇格ベクターを探すための最適なツール:** [**WinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)

### [システム情報](windows-local-privilege-escalation/#system-info)

- [ ] [**システム情報**](windows-local-privilege-escalation/#system-info)を取得する
- [ ] **スクリプトを使用して** **カーネル**の[**脆弱性を検索**](windows-local-privilege-escalation/#version-exploits)
- [ ] Googleを使用してカーネルの**脆弱性を検索**
- [ ] searchsploitを使用してカーネルの**脆弱性を検索**
- [ ] [**環境変数**](windows-local-privilege-escalation/#environment)に興味深い情報はありますか？
- [ ] PowerShell履歴にパスワードはありますか？[**PowerShell履歴**](windows-local-privilege-escalation/#powershell-history)
- [ ] [**インターネット設定**](windows-local-privilege-escalation/#internet-settings)に興味深い情報はありますか？
- [ ] [**ドライブ**](windows-local-privilege-escalation/#drives)はありますか？
- [ ] [**WSUSの脆弱性**](windows-local-privilege-escalation/#wsus)はありますか？
- [**AlwaysInstallElevated**](windows-local-privilege-escalation/#alwaysinstallelevated)はありますか？

### [ログ/AV列挙](windows-local-privilege-escalation/#enumeration)

- [ ] [**監査**](windows-local-privilege-escalation/#audit-settings)および[**WEF**](windows-local-privilege-escalation/#wef)設定を確認する
- [ ] [**LAPS**](windows-local-privilege-escalation/#laps)を確認する
- [ ] [**WDigest**](windows-local-privilege-escalation/#wdigest)がアクティブかどうかを確認する
- [ ] [**LSA保護**](windows-local-privilege-escalation/#lsa-protection)はありますか？
- [ ] [**資格情報ガード**](windows-local-privilege-escalation/#credentials-guard)はありますか？[**キャッシュされた資格情報**](windows-local-privilege-escalation/#cached-credentials)はありますか？
- [ ] いずれかの[**AV**](windows-av-bypass)があるかどうかを確認する
- [ ] [**AppLockerポリシー**](authentication-credentials-uac-and-efs#applocker-policy)はありますか？
- [ ] [**UAC**](authentication-credentials-uac-and-efs/uac-user-account-control)はありますか？
- [ ] [**ユーザー特権**](windows-local-privilege-escalation/#users-and-groups)はありますか？
- [ ] [**現在の**ユーザーの**特権**](windows-local-privilege-escalation/#users-and-groups)を確認する
- [ ] 特権のあるグループのメンバーですか？[**特権グループ**](windows-local-privilege-escalation/#privileged-groups)を確認する
- [ ] これらのトークンのいずれかが有効になっているかどうかを確認する：**SeImpersonatePrivilege、SeAssignPrimaryPrivilege、SeTcbPrivilege、SeBackupPrivilege、SeRestorePrivilege、SeCreateTokenPrivilege、SeLoadDriverPrivilege、SeTakeOwnershipPrivilege、SeDebugPrivilege**[**トークン操作**](windows-local-privilege-escalation/#token-manipulation)はありますか？
- [**ユーザーセッション**](windows-local-privilege-escalation/#logged-users-sessions)はありますか？
- [ ] [**ユーザーのホーム**](windows-local-privilege-escalation/#home-folders)を確認する（アクセスは？）
- [ ] [**パスワードポリシー**](windows-local-privilege-escalation/#password-policy)を確認する
- [ ] [**クリップボード**](windows-local-privilege-escalation/#get-the-content-of-the-clipboard)には何が入っていますか？

### [ネットワーク](windows-local-privilege-escalation/#network)

- [ ] **現在の**[**ネットワーク情報**](windows-local-privilege-escalation/#network)を確認する
- [ ] 外部に制限された**非表示のローカルサービス**を確認する

### [実行中のプロセス](windows-local-privilege-escalation/#running-processes)

- [ ] プロセスのバイナリ[**ファイルおよびフォルダの権限**](windows-local-privilege-escalation/#file-and-folder-permissions)を確認する
- [ ] [**メモリパスワードの採掘**](windows-local-privilege-escalation/#memory-password-mining)を行う
- [ ] [**セキュリティの脆弱なGUIアプリ**](windows-local-privilege-escalation/#insecure-gui-apps)を確認する
- [ ] `ProcDump.exe`を使用して**興味深いプロセス**から資格情報を盗みますか？（firefox、chromeなど...）

### [サービス](windows-local-privilege-escalation/#services)

- [ ] 任意のサービスを**変更**できますか？[**権限**](windows-local-privilege-escalation#permissions)を確認する
- [ ] 任意のサービスが実行する**バイナリ**を**変更**できますか？[**サービスバイナリのパスを変更**](windows-local-privilege-escalation/#modify-service-binary-path)できますか？
- [ ] 任意のサービスの**レジストリ**を**変更**できますか？[**サービスレジストリの変更権限**](windows-local-privilege-escalation/#services-registry-modify-permissions)を確認する
- [ ] 任意の**引用符のないサービス**バイナリ**パス**を利用できますか？[**引用符のないサービスパス**](windows-local-privilege-escalation/#unquoted-service-paths)を確認する

### [**アプリケーション**](windows-local-privilege-escalation/#applications)

- インストールされたアプリケーションの[**書き込み権限**](windows-local-privilege-escalation/#write-permissions)を確認する
- [**起動アプリケーション**](windows-local-privilege-escalation/#run-at-startup)を確認する
- **脆弱な**[**ドライバー**](windows-local-privilege-escalation/#drivers)を確認する

### [DLLハイジャック](windows-local-privilege-escalation/#path-dll-hijacking)

- **PATH内の任意のフォルダに書き込めますか**？
- **存在しないDLLを読み込もうとする既知のサービスバイナリ**がありますか？
- 任意の**バイナリフォルダ**に**書き込めますか**？
### [ネットワーク](windows-local-privilege-escalation/#network)

* [ ] ネットワークを列挙する（共有、インターフェイス、ルート、隣接者、...）
* [ ] ローカルホスト（127.0.0.1）でリスニングしているネットワークサービスに特に注意する

### [Windowsの資格情報](windows-local-privilege-escalation/#windows-credentials)

* [ ] [**Winlogon** ](windows-local-privilege-escalation/#winlogon-credentials)の資格情報
* [ ] [**Windows Vault**](windows-local-privilege-escalation/#credentials-manager-windows-vault)で使用できる資格情報は？
* [ ] 興味深い[**DPAPIの資格情報**](windows-local-privilege-escalation/#dpapi)は？
* [ ] 保存された[**Wifiネットワーク**](windows-local-privilege-escalation/#wifi)のパスワードは？
* [ ] 興味深い情報は[**保存されたRDP接続**](windows-local-privilege-escalation/#saved-rdp-connections)にありますか？
* [ ] [**最近実行されたコマンド**](windows-local-privilege-escalation/#recently-run-commands)のパスワードは？
* [ ] [**リモートデスクトップ資格情報マネージャー**](windows-local-privilege-escalation/#remote-desktop-credential-manager)のパスワードは？
* [ ] [**AppCmd.exe**が存在](windows-local-privilege-escalation/#appcmd-exe)するか？資格情報は？
* [ ] [**SCClient.exe**](windows-local-privilege-escalation/#scclient-sccm)は？DLLサイドローディングは？

### [ファイルとレジストリ（資格情報）](windows-local-privilege-escalation/#files-and-registry-credentials)

* [ ] **Putty:** [**資格情報**](windows-local-privilege-escalation/#putty-creds) **および** [**SSHホストキー**](windows-local-privilege-escalation/#putty-ssh-host-keys)
* [ ] レジストリにある[**SSHキー**](windows-local-privilege-escalation/#ssh-keys-in-registry)は？
* [ ] [**無人ファイル**](windows-local-privilege-escalation/#unattended-files)にパスワードは？
* [ ] いずれかの[**SAM & SYSTEM**](windows-local-privilege-escalation/#sam-and-system-backups)のバックアップはありますか？
* [ ] [**クラウドの資格情報**](windows-local-privilege-escalation/#cloud-credentials)は？
* [ ] [**McAfee SiteList.xml**](windows-local-privilege-escalation/#mcafee-sitelist.xml)ファイルは？
* [ ] [**キャッシュされたGPPパスワード**](windows-local-privilege-escalation/#cached-gpp-pasword)は？
* [ ] [**IIS Web構成ファイル**](windows-local-privilege-escalation/#iis-web-config)にパスワードは？
* [ ] 興味深い情報は[**ウェブログ**](windows-local-privilege-escalation/#logs)にありますか？
* [ ] ユーザーに[**資格情報を要求**](windows-local-privilege-escalation/#ask-for-credentials)したいですか？
* [ ] リサイクルビン内の[**興味深いファイル**](windows-local-privilege-escalation/#credentials-in-the-recyclebin)はありますか？
* [ ] 他の[**資格情報を含むレジストリ**](windows-local-privilege-escalation/#inside-the-registry)は？
* [ ] ブラウザデータ（データベース、履歴、ブックマーク、...）内に[**資格情報**](windows-local-privilege-escalation/#browsers-history)はありますか？
* [**ファイルとレジストリ内の一般的なパスワード検索**](windows-local-privilege-escalation/#generic-password-search-in-files-and-registry)は？
* パスワードを自動的に検索する[**ツール**](windows-local-privilege-escalation/#tools-that-search-for-passwords)は？

### [漏洩したハンドラ](windows-local-privilege-escalation/#leaked-handlers)

* [ ] 管理者によって実行されたプロセスのハンドラにアクセスできますか？

### [パイプクライアントの擬似化](windows-local-privilege-escalation/#named-pipe-client-impersonation)

* [ ] それを悪用できるかどうかを確認する
