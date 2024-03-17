# Tarayıcı Kalıntıları

<details>

<summary><strong>Sıfırdan kahraman olmak için AWS hackleme öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Kırmızı Takım Uzmanı)</strong></a><strong>!</strong></summary>

HackTricks'ı desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamını görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARI**](https://github.com/sponsors/carlospolop)'na göz atın!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* [**PEASS Ailesi'ni**](https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuz
* **Katılın** 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya bizi **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'da takip edin.**
* **Hacking püf noktalarınızı paylaşarak PR'lar göndererek HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına katkıda bulunun.

</details>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
Dünyanın **en gelişmiş topluluk araçları** tarafından desteklenen **iş akışlarını kolayca oluşturmak ve otomatikleştirmek** için [**Trickest'i**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) kullanın.\
Bugün Erişim Alın:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## Tarayıcı Kalıntıları <a href="#id-3def" id="id-3def"></a>

Tarayıcı kalıntıları, web tarayıcıları tarafından depolanan çeşitli veri türlerini içerir; gezinme geçmişi, yer imleri ve önbellek verileri gibi. Bu kalıntılar işletim sistemi içinde belirli klasörlerde saklanır, tarayıcılara göre konum ve ad farklı olabilir, ancak genellikle benzer veri türlerini depolarlar.

İşte en yaygın tarayıcı kalıntılarının özeti:

* **Gezinme Geçmişi**: Kullanıcının web sitelerini ziyaretlerini takip eder, kötü amaçlı sitelere ziyaretleri tanımlamak için faydalıdır.
* **Otomatik Tamamlama Verileri**: Sık aramalara dayalı öneriler, gezinme geçmişi ile birleştirildiğinde içgörüler sunar.
* **Yer İmleri**: Kullanıcı tarafından hızlı erişim için kaydedilen siteler.
* **Uzantılar ve Eklentiler**: Kullanıcı tarafından yüklenen tarayıcı uzantıları veya eklentileri.
* **Önbellek**: Web içeriğini (örneğin, resimler, JavaScript dosyaları) saklar, web sitesi yükleme sürelerini iyileştirmek için değerli bir araştırma analizi aracıdır.
* **Girişler**: Saklanan giriş kimlik bilgileri.
* **Favikonlar**: Sitelerle ilişkilendirilen simgeler, sekmelerde ve yer imlerinde görünür, kullanıcı ziyaretleri hakkında ek bilgiler için faydalıdır.
* **Tarayıcı Oturumları**: Açık tarayıcı oturumlarıyla ilgili veriler.
* **İndirmeler**: Tarayıcı aracılığıyla indirilen dosyaların kayıtları.
* **Form Verileri**: Web formlarına girilen bilgiler, gelecekte otomatik doldurma önerileri için kaydedilir.
* **Küçük Resimler**: Web sitelerinin önizleme görüntüleri.
* **Özel Sözlük.txt**: Kullanıcının tarayıcının sözlüğüne eklediği kelimeler.

## Firefox

Firefox, kullanıcı verilerini işletim sistemine bağlı olarak belirli konumlarda depolanan profiller içinde düzenler:

* **Linux**: `~/.mozilla/firefox/`
* **MacOS**: `/Users/$USER/Library/Application Support/Firefox/Profiles/`
* **Windows**: `%userprofile%\AppData\Roaming\Mozilla\Firefox\Profiles\`

Bu dizinlerdeki `profiles.ini` dosyası kullanıcı profillerini listeler. Her profilin verileri, `profiles.ini` dosyasının bulunduğu dizindeki `Path` değişkeninde adlandırılan bir klasörde saklanır. Bir profil klasörü eksikse, silinmiş olabilir.

Her profil klasöründe, birkaç önemli dosya bulabilirsiniz:

* **places.sqlite**: Geçmişi, yer imlerini ve indirmeleri saklar. Windows'ta [BrowsingHistoryView](https://www.nirsoft.net/utils/browsing\_history\_view.html) gibi araçlar geçmiş verilerine erişebilir.
* Geçmiş ve indirme bilgilerini çıkarmak için belirli SQL sorgularını kullanın.
* **bookmarkbackups**: Yer imlerinin yedeklerini içerir.
* **formhistory.sqlite**: Web formu verilerini saklar.
* **handlers.json**: Protokol işleyicilerini yönetir.
* **persdict.dat**: Özel sözlük kelimeleri.
* **addons.json** ve **extensions.sqlite**: Yüklenen eklenti ve uzantı bilgileri.
* **cookies.sqlite**: Çerez depolama, Windows'ta inceleme için [MZCookiesView](https://www.nirsoft.net/utils/mzcv.html) kullanılabilir.
* **cache2/entries** veya **startupCache**: Önbellek verileri, [MozillaCacheView](https://www.nirsoft.net/utils/mozilla\_cache\_viewer.html) gibi araçlar aracılığıyla erişilebilir.
* **favicons.sqlite**: Favikonları saklar.
* **prefs.js**: Kullanıcı ayarları ve tercihleri.
* **downloads.sqlite**: Eski indirme veritabanı, şimdi places.sqlite'a entegre edilmiştir.
* **thumbnails**: Web sitesi küçük resimleri.
* **logins.json**: Şifrelenmiş giriş bilgileri.
* **key4.db** veya **key3.db**: Hassas bilgileri güvence altına alan şifreleme anahtarlarını saklar.

Ayrıca, tarayıcının anti-phishing ayarlarını kontrol etmek için `prefs.js` dosyasında `browser.safebrowsing` girişlerini arayarak güvenli gezinme özelliklerinin etkin veya devre dışı bırakılıp bırakılmadığını kontrol edebilirsiniz.

Ana şifreyi çözmek için şu adresten [https://github.com/unode/firefox\_decrypt](https://github.com/unode/firefox\_decrypt) yararlanabilirsiniz\
Aşağıdaki betik ve çağrı ile kaba kuvvetle bir şifre dosyası belirleyebilirsiniz:

{% code title="brute.sh" %}
```bash
#!/bin/bash

#./brute.sh top-passwords.txt 2>/dev/null | grep -A2 -B2 "chrome:"
passfile=$1
while read pass; do
echo "Trying $pass"
echo "$pass" | python firefox_decrypt.py
done < $passfile
```
## Google Chrome

Google Chrome, kullanıcı profillerini işletim sistemine bağlı olarak belirli konumlarda saklar:

- **Linux**: `~/.config/google-chrome/`
- **Windows**: `C:\Users\XXX\AppData\Local\Google\Chrome\User Data\`
- **MacOS**: `/Users/$USER/Library/Application Support/Google/Chrome/`

Bu dizinlerde, kullanıcı verilerinin çoğu **Default/** veya **ChromeDefaultData/** klasörlerinde bulunabilir. Önemli verileri içeren aşağıdaki dosyalar bulunmaktadır:

- **History**: URL'leri, indirmeleri ve arama anahtar kelimelerini içerir. Windows'ta, geçmişi okumak için [ChromeHistoryView](https://www.nirsoft.net/utils/chrome\_history\_view.html) kullanılabilir. "Transition Type" sütunu, kullanıcıların bağlantılara tıklamalarını, yazılan URL'leri, form gönderimlerini ve sayfa yenilemelerini içerir.
- **Cookies**: Çerezleri saklar. İncelemek için [ChromeCookiesView](https://www.nirsoft.net/utils/chrome\_cookies\_view.html) kullanılabilir.
- **Cache**: Önbelleğe alınmış verileri saklar. İncelemek için Windows kullanıcıları [ChromeCacheView](https://www.nirsoft.net/utils/chrome\_cache\_view.html) kullanabilir.
- **Bookmarks**: Kullanıcı yer imleri.
- **Web Data**: Form geçmişini içerir.
- **Favicons**: Web sitesi faviconlarını saklar.
- **Login Data**: Kullanıcı adları ve şifreler gibi giriş kimlik bilgilerini içerir.
- **Current Session**/**Current Tabs**: Geçerli gezinme oturumu ve açık sekmeler hakkında veriler.
- **Last Session**/**Last Tabs**: Chrome kapatılmadan önceki son oturum sırasında aktif olan siteler hakkında bilgi.
- **Extensions**: Tarayıcı uzantıları ve eklentileri için dizinler.
- **Thumbnails**: Web sitesi minik resimlerini saklar.
- **Preferences**: Eklentiler, uzantılar, açılır pencereler, bildirimler ve daha fazlası için ayarları içeren zengin bir dosya.
- **Tarayıcının yerleşik anti-phishing'i**: Anti-phishing ve kötü amaçlı yazılım korumasının etkin olup olmadığını kontrol etmek için `grep 'safebrowsing' ~/Library/Application Support/Google/Chrome/Default/Preferences` komutunu çalıştırın. Çıktıda `{"enabled: true,"}` arayın.

## **SQLite DB Veri Kurtarma**

Önceki bölümlerde gözlemleyebileceğiniz gibi, hem Chrome hem de Firefox verileri saklamak için **SQLite** veritabanlarını kullanır. Silinmiş girişleri kurtarmak için [**sqlparse**](https://github.com/padfoot999/sqlparse) veya [**sqlparse\_gui**](https://github.com/mdegrazia/SQLite-Deleted-Records-Parser/releases) gibi araçlar kullanılabilir.

## **Internet Explorer 11**

Internet Explorer 11, depolanan bilgileri ve ilgili ayrıntıları kolay erişim ve yönetim için farklı konumlarda yönetir.

### Metadata Depolama

Internet Explorer için metadata `%userprofile%\Appdata\Local\Microsoft\Windows\WebCache\WebcacheVX.data` (VX V01, V16 veya V24 olabilir) içinde saklanır. Buna ek olarak, `V01.log` dosyası, `WebcacheVX.data` ile değişiklik zamanı uyumsuzluklarını gösterebilir, bu da `esentutl /r V01 /d` kullanarak onarım gerektiğini gösterebilir. Bu ESE veritabanında bulunan metadata, photorec ve [ESEDatabaseView](https://www.nirsoft.net/utils/ese\_database\_view.html) gibi araçlar kullanılarak kurtarılabilir ve incelenebilir. **Containers** tablosu içinde, her veri segmentinin depolandığı belirli tabloları veya konteynerleri ayırt edebilirsiniz, Skype gibi diğer Microsoft araçları için önbellek ayrıntılarını içerir.

### Önbellek İnceleme

[IECacheView](https://www.nirsoft.net/utils/ie\_cache\_viewer.html) aracı, önbellek incelemesine olanak tanır, önbellek veri çıkarma klasör konumunu gerektirir. Önbellek için metadata, dosya adı, dizin, erişim sayısı, URL kökeni ve önbellek oluşturma, erişim, değiştirme ve sona erme zamanlarını gösteren zaman damgalarını içerir.

### Çerez Yönetimi

Çerezler, [IECookiesView](https://www.nirsoft.net/utils/iecookies.html) kullanılarak incelenebilir, metadata isimleri, URL'ler, erişim sayıları ve çeşitli zamanla ilgili ayrıntıları içerir. Kalıcı çerezler `%userprofile%\Appdata\Roaming\Microsoft\Windows\Cookies` içinde saklanırken, oturum çerezleri bellekte saklanır.

### İndirme Ayrıntıları

İndirme metadata'sı [ESEDatabaseView](https://www.nirsoft.net/utils/ese\_database\_view.html) ile erişilebilir, belirli konteynerler URL, dosya türü ve indirme konumu gibi verileri içerir. Fiziksel dosyalar `%userprofile%\Appdata\Roaming\Microsoft\Windows\IEDownloadHistory` altında bulunabilir.

### Gezinti Geçmişi

Gezinti geçmişini incelemek için [BrowsingHistoryView](https://www.nirsoft.net/utils/browsing\_history\_view.html) kullanılabilir, çıkarılan geçmiş dosyalarının konumunu ve Internet Explorer için yapılandırmayı gerektirir. Buradaki metadata, değiştirme ve erişim zamanları ile erişim sayılarını içerir. Geçmiş dosyaları `%userprofile%\Appdata\Local\Microsoft\Windows\History` içinde bulunur.

### Yazılan URL'ler

Yazılan URL'ler ve kullanım zamanları, kullanıcı tarafından girilen son 50 URL'yi ve son giriş zamanlarını takip eden `NTUSER.DAT` altında `Software\Microsoft\InternetExplorer\TypedURLs` ve `Software\Microsoft\InternetExplorer\TypedURLsTime` içinde saklanır.

## Microsoft Edge

Microsoft Edge, kullanıcı verilerini `%userprofile%\Appdata\Local\Packages` içinde saklar. Farklı veri türleri için yollar şunlardır:

- **Profil Yolu**: `C:\Users\XX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC`
- **Geçmiş, Çerezler ve İndirmeler**: `C:\Users\XX\AppData\Local\Microsoft\Windows\WebCache\WebCacheV01.dat`
- **Ayarlar, Yer İmleri ve Okuma Listesi**: `C:\Users\XX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC\MicrosoftEdge\User\Default\DataStore\Data\nouser1\XXX\DBStore\spartan.edb`
- **Önbellek**: `C:\Users\XXX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC#!XXX\MicrosoftEdge\Cache`
- **Son Etkin Oturumlar**: `C:\Users\XX\AppData\Local\Packages\Microsoft.MicrosoftEdge_XXX\AC\MicrosoftEdge\User\Default\Recovery\Active`

## Safari

Safari verileri `/Users/$User/Library/Safari` içinde saklanır. Ana dosyalar şunları içerir:

- **History.db**: URL'leri ve ziyaret zamanlarını içeren `history_visits` ve `history_items` tablolarını içerir. Sorgulamak için `sqlite3` kullanın.
- **Downloads.plist**: İndirilen dosyalar hakkında bilgi.
- **Bookmarks.plist**: Yer imlerini saklar.
- **TopSites.plist**: En sık ziyaret edilen siteler.
- **Extensions.plist**: Safari tarayıcı uzantılarının listesi. Almak için `plutil` veya `pluginkit` kullanın.
- **UserNotificationPermissions.plist**: Bildirim göndermeye izin verilen alanlar. Ayrıştırmak için `plutil` kullanın.
- **LastSession.plist**: Son oturumdan sekmeler. Ayrıştırmak için `plutil` kullanın.
- **Tarayıcının yerleşik anti-phishing'i**: `defaults read com.apple.Safari WarnAboutFraudulentWebsites` kullanarak kontrol edin. 1 yanıtı özelliğin etkin olduğunu gösterir.

## Opera

Opera'nın verileri `/Users/$USER/Library/Application Support/com.operasoftware.Opera` içinde saklanır ve Chrome'un geçmiş ve indirme formatını paylaşır.

- **Tarayıcının yerleşik anti-phishing'i**: `fraud_protection_enabled`in `true` olarak ayarlandığını kontrol ederek doğrulayın, bunu `grep` kullanarak yapabilirsiniz.

Bu yollar ve komutlar, farklı web tarayıcıları tarafından depolanan gezinme verilerine erişmek ve anlamak için önemlidir.
* Eğer **şirketinizin HackTricks'te reklamını görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARI**](https://github.com/sponsors/carlospolop)'na göz atın!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* [**PEASS Ailesi'ni**](https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuz
* **Katılın** 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya bizi **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)**'da takip edin.**
* **Hacking püf noktalarınızı paylaşın, PR'larınızı göndererek** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına.
