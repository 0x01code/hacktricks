# macOS Kırmızı Takım

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong> ile sıfırdan kahramana kadar AWS hacklemeyi öğrenin!</summary>

HackTricks'ı desteklemenin diğer yolları:

- **Şirketinizi HackTricks'te reklamını görmek** veya **HackTricks'i PDF olarak indirmek** için [**ABONELİK PLANLARI**](https://github.com/sponsors/carlospolop)'na göz atın!
- [**Resmi PEASS & HackTricks ürünleri**](https://peass.creator-spring.com)'ni edinin
- [**PEASS Ailesi'ni**](https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuz
- **Katılın** 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya bizi **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)'da **takip edin**.
- **Hacking püf noktalarınızı göndererek HackTricks** ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına PR gönderin.

</details>

## MDM'leri Kötüye Kullanma

- JAMF Pro: `jamf checkJSSConnection`
- Kandji

Yönetim platformuna erişmek için **yönetici kimlik bilgilerini ele geçirirseniz**, kötü amaçlı yazılımınızı makinelerde dağıtarak **tüm bilgisayarları potansiyel olarak tehlikeye atabilirsiniz**.

MacOS ortamlarında kırmızı takım için MDM'lerin nasıl çalıştığını anlamanız şiddetle tavsiye edilir:

{% content-ref url="macos-mdm/" %}
[macos-mdm](macos-mdm/)
{% endcontent-ref %}

### MDM'yi C2 Olarak Kullanma

Bir MDM, profilleri yüklemeye, sorgulamaya veya kaldırmaya, uygulamaları yüklemeye, yerel yönetici hesapları oluşturmaya, firmware şifresi belirlemeye, FileVault anahtarını değiştirmeye izin verecektir...

Kendi MDM'nizi çalıştırmak için [**https://mdmcert.download/**](https://mdmcert.download/) adresinden alabileceğiniz bir **CSR'nızı bir satıcı tarafından imzalatmanız** gerekmektedir. Apple cihazları için kendi MDM'nizi çalıştırmak için [**MicroMDM**](https://github.com/micromdm/micromdm) kullanabilirsiniz.

Ancak, bir uygulamayı kayıtlı bir cihaza yüklemek için hala bir geliştirici hesabıyla imzalanmış olması gerekmektedir... ancak, MDM kaydı sırasında **cihaz, MDM'nin SSL sertifikasını güvenilir bir CA olarak ekler**, böylece artık herhangi bir şeyi imzalayabilirsiniz.

Cihazı bir MDM'ye kaydetmek için kök olarak bir **`mobileconfig`** dosyası yüklemeniz gerekmektedir, bu dosya bir **pkg** dosyası aracılığıyla teslim edilebilir (zip içine sıkıştırabilir ve safari'den indirildiğinde açılabilir).

**Mythic ajanı Orthrus** bu tekniği kullanır.

### JAMF PRO'yu Kötüye Kullanma

JAMF, **özel betikler** (sistem yöneticisi tarafından geliştirilen betikler), **yerel yükler** (yerel hesap oluşturma, EFI şifresi belirleme, dosya/işlem izleme...) ve **MDM** (cihaz yapılandırmaları, cihaz sertifikaları...) çalıştırabilir.

#### JAMF otomatik kayıt

`https://<şirket-adı>.jamfcloud.com/enroll/` gibi bir sayfaya giderek **otomatik kaydı etkinleştirilip etkinleştirilmediğini** kontrol edebilirsiniz. Etkinleştirilmişse **erişim için kimlik bilgileri isteyebilir**.

[**JamfSniper.py**](https://github.com/WithSecureLabs/Jamf-Attack-Toolkit/blob/master/JamfSniper.py) betiğini kullanarak bir şifre sıçratma saldırısı gerçekleştirebilirsiniz.

Ayrıca, uygun kimlik bilgileri bulduktan sonra diğer kullanıcı adlarını kaba kuvvet saldırısı yapabilirsiniz:

![](<../../.gitbook/assets/image (107).png>)

#### JAMF cihaz Kimlik Doğrulama

<figure><img src="../../.gitbook/assets/image (167).png" alt=""><figcaption></figcaption></figure>

**`jamf`** ikili dosyası, zamanında **herkesle paylaşılan** anahtarı açmak için gizliydi ve o zaman keşfedildiğinde: **`jk23ucnq91jfu9aj`** idi.\
Ayrıca, jamf **`/Library/LaunchAgents/com.jamf.management.agent.plist`** konumunda bir **LaunchDaemon** olarak kalıcıdır.

#### JAMF Cihaz Devralma

**`jamf`**'in kullanacağı **JSS** (Jamf Yazılım Sunucusu) **URL'si**, **`/Library/Preferences/com.jamfsoftware.jamf.plist`** konumundadır.\
Bu dosya temelde URL'yi içerir:

{% code overflow="wrap" %}
```bash
plutil -convert xml1 -o - /Library/Preferences/com.jamfsoftware.jamf.plist

[...]
<key>is_virtual_machine</key>
<false/>
<key>jss_url</key>
<string>https://halbornasd.jamfcloud.com/</string>
<key>last_management_framework_change_id</key>
<integer>4</integer>
[...]
```
{% endcode %}

Bu nedenle, bir saldırgan, **bu dosyayı üzerine yazacak** şekilde kötü niyetli bir paket (`pkg`) bırakabilir ve yüklenirken **URL'yi bir Typhon ajanından Mythic C2 dinleyicisine ayarlayabilir** ve artık JAMF'ı C2 olarak kötüye kullanabilir. 

{% code overflow="wrap" %}
```bash
# After changing the URL you could wait for it to be reloaded or execute:
sudo jamf policy -id 0

# TODO: There is an ID, maybe it's possible to have the real jamf connection and another one to the C2
```
{% endcode %}

#### JAMF Taklit

Bir cihaz ile JMF arasındaki iletişimi **taklit etmek** için şunlara ihtiyacınız vardır:

* Cihazın **UUID**'si: `ioreg -d2 -c IOPlatformExpertDevice | awk -F" '/IOPlatformUUID/{print $(NF-1)}'`
* Cihaz sertifikasını içeren **JAMF anahtarlığı**: `/Library/Application\ Support/Jamf/JAMF.keychain`

Bu bilgilerle, **çalınan** Donanım **UUID'si** ile ve **SIP devre dışı** bırakılmış bir VM **oluşturun**, **JAMF anahtarlığını** bırakın, Jamf **ajansını kancalayın** ve bilgilerini çalın.

#### Sırların Çalınması

<figure><img src="../../.gitbook/assets/image (1025).png" alt=""><figcaption><p>a</p></figcaption></figure>

Ayrıca, **özel betiklerin** yerini izleyebilirsiniz `/Library/Application Support/Jamf/tmp/` yöneticilerin Jamf aracılığıyla çalıştırmak isteyebileceği çünkü buraya yerleştirilir, çalıştırılır ve kaldırılır. Bu betikler **kimlik bilgileri içerebilir**.

Ancak, **kimlik bilgileri** bu betiklere **parametreler** olarak iletilmiş olabilir, bu nedenle `ps aux | grep -i jamf`'yi (root olmadan bile) izlemeniz gerekebilir.

[**JamfExplorer.py**](https://github.com/WithSecureLabs/Jamf-Attack-Toolkit/blob/master/JamfExplorer.py) betiği yeni dosyaların eklenmesini ve yeni işlem argümanlarını dinleyebilir.

### macOS Uzak Erişim

Ve ayrıca **MacOS** "özel" **ağ** **protokolleri** hakkında:

{% content-ref url="../macos-security-and-privilege-escalation/macos-protocols.md" %}
[macos-protocols.md](../macos-security-and-privilege-escalation/macos-protocols.md)
{% endcontent-ref %}

## Active Directory

Bazı durumlarda **MacOS bilgisayarının bir AD'ye bağlı olduğunu** göreceksiniz. Bu senaryoda, genellikle yaptığınız gibi etkin dizini **numaralandırmaya çalışmalısınız**. Aşağıdaki sayfalarda **yardım** bulabilirsiniz:

{% content-ref url="../../network-services-pentesting/pentesting-ldap.md" %}
[pentesting-ldap.md](../../network-services-pentesting/pentesting-ldap.md)
{% endcontent-ref %}

{% content-ref url="../../windows-hardening/active-directory-methodology/" %}
[active-directory-methodology](../../windows-hardening/active-directory-methodology/)
{% endcontent-ref %}

{% content-ref url="../../network-services-pentesting/pentesting-kerberos-88/" %}
[pentesting-kerberos-88](../../network-services-pentesting/pentesting-kerberos-88/)
{% endcontent-ref %}

Size yardımcı olabilecek bazı **yerel MacOS araçları** `dscl`'dir:
```bash
dscl "/Active Directory/[Domain]/All Domains" ls /
```
Ayrıca, MacOS için AD'yi otomatik olarak numaralandırmak ve kerberos ile oynamak için bazı araçlar hazırlanmıştır:

* [**Machound**](https://github.com/XMCyber/MacHound): MacHound, Bloodhound denetleme aracına bir uzantıdır ve MacOS ana bilgisayarlarında Active Directory ilişkilerini toplamayı ve yüklemeyi sağlar.
* [**Bifrost**](https://github.com/its-a-feature/bifrost): Bifrost, macOS'ta Heimdal krb5 API'leri ile etkileşim sağlamak üzere tasarlanmış bir Objective-C projesidir. Projenin amacı, hedef üzerinde başka bir çerçeve veya paket gerektirmeden yerel API'leri kullanarak macOS cihazlarında Kerberos etrafında daha iyi güvenlik testleri yapmayı sağlamaktır.
* [**Orchard**](https://github.com/its-a-feature/Orchard): JavaScript for Automation (JXA) aracı olan Orchard, Active Directory numaralandırması yapmak için kullanılır.

### Alan Bilgileri
```bash
echo show com.apple.opendirectoryd.ActiveDirectory | scutil
```
### Kullanıcılar

MacOS kullanıcılarının üç türü şunlardır:

- **Yerel Kullanıcılar** — Yerel OpenDirectory hizmeti tarafından yönetilir, Active Directory ile herhangi bir şekilde bağlantılı değillerdir.
- **Ağ Kullanıcıları** — Geçici Active Directory kullanıcıları, kimlik doğrulamak için DC sunucusuna bağlantı gerektirirler.
- **Mobil Kullanıcılar** — Yerel yedekleri olan Active Directory kullanıcıları, kimlik bilgileri ve dosyaları için yerel bir yedekleme yaparlar.

Kullanıcılar ve gruplarla ilgili yerel bilgiler, _/var/db/dslocal/nodes/Default_ klasöründe saklanır.\
Örneğin, _mark_ adlı kullanıcıyla ilgili bilgiler _/var/db/dslocal/nodes/Default/users/mark.plist_ dosyasında saklanır ve _admin_ grubuyla ilgili bilgiler _/var/db/dslocal/nodes/Default/groups/admin.plist_ dosyasında bulunur.

HasSession ve AdminTo kenarlarını kullanmanın yanı sıra, **MacHound Bloodhound veritabanına üç yeni kenar ekler**:

- **CanSSH** - ana bilgisayara SSH yapmaya izin verilen varlık
- **CanVNC** - ana bilgisayara VNC yapmaya izin verilen varlık
- **CanAE** - ana bilgisayarda AppleEvent betiklerini çalıştırmaya izin verilen varlık
```bash
#User enumeration
dscl . ls /Users
dscl . read /Users/[username]
dscl "/Active Directory/TEST/All Domains" ls /Users
dscl "/Active Directory/TEST/All Domains" read /Users/[username]
dscacheutil -q user

#Computer enumeration
dscl "/Active Directory/TEST/All Domains" ls /Computers
dscl "/Active Directory/TEST/All Domains" read "/Computers/[compname]$"

#Group enumeration
dscl . ls /Groups
dscl . read "/Groups/[groupname]"
dscl "/Active Directory/TEST/All Domains" ls /Groups
dscl "/Active Directory/TEST/All Domains" read "/Groups/[groupname]"

#Domain Information
dsconfigad -show
```
Daha fazla bilgi için [https://its-a-feature.github.io/posts/2018/01/Active-Directory-Discovery-with-a-Mac/](https://its-a-feature.github.io/posts/2018/01/Active-Directory-Discovery-with-a-Mac/) adresine bakabilirsiniz.

## Anahtarlık Erişimi

Anahtarlık muhtemelen hassas bilgiler içerir ve bir uyarı oluşturmadan erişilirse kırmızı takım egzersizine devam etmede yardımcı olabilir:

{% content-ref url="macos-keychain.md" %}
[macos-keychain.md](macos-keychain.md)
{% endcontent-ref %}

## Harici Hizmetler

MacOS Kırmızı Takımı, genellikle **MacOS'un doğrudan birkaç harici platformla entegre edilmiş olması nedeniyle normal Windows Kırmızı Takımdan farklıdır**. MacOS'un yaygın bir yapılandırması, **OneLogin senkronize kimlik bilgileri kullanarak bilgisayara erişim sağlamak ve OneLogin aracılığıyla birkaç harici hizmete** (github, aws gibi...) erişmektir.

## Çeşitli Kırmızı Takım teknikleri

### Safari

Safari'de bir dosya indirildiğinde, eğer "güvenli" bir dosya ise **otomatik olarak açılacaktır**. Örneğin, bir zip dosyası indirirseniz, otomatik olarak açılacaktır:

<figure><img src="../../.gitbook/assets/image (226).png" alt=""><figcaption></figcaption></figure>

## Referanslar

* [**https://www.youtube.com/watch?v=IiMladUbL6E**](https://www.youtube.com/watch?v=IiMladUbL6E)
* [**https://medium.com/xm-cyber/introducing-machound-a-solution-to-macos-active-directory-based-attacks-2a425f0a22b6**](https://medium.com/xm-cyber/introducing-machound-a-solution-to-macos-active-directory-based-attacks-2a425f0a22b6)
* [**https://gist.github.com/its-a-feature/1a34f597fb30985a2742bb16116e74e0**](https://gist.github.com/its-a-feature/1a34f597fb30985a2742bb16116e74e0)
* [**Come to the Dark Side, We Have Apples: Turning macOS Management Evil**](https://www.youtube.com/watch?v=pOQOh07eMxY)
* [**OBTS v3.0: "An Attackers Perspective on Jamf Configurations" - Luke Roberts / Calum Hall**](https://www.youtube.com/watch?v=ju1IYWUv4ZA)
