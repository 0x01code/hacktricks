# Yerel Bulut Depolama

<details>

<summary><strong>Sıfırdan kahraman olmak için AWS hackleme öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Kırmızı Takım Uzmanı)</strong></a><strong>!</strong></summary>

HackTricks'ı desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamını görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARI**]'na (https://github.com/sponsors/carlospolop) göz atın!
* [**Resmi PEASS & HackTricks ürünlerine göz atın**](https://peass.creator-spring.com)
* [**PEASS Ailesi'ni**](https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuz
* **Katılın** 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya bizi **Twitter** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** takip edin.**
* **Hacking püf noktalarınızı paylaşarak PR'lar göndererek HackTricks** (https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına katkıda bulunun.

</details>

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=local-cloud-storage) kullanarak dünyanın **en gelişmiş topluluk araçları** tarafından desteklenen **iş akışlarını kolayca oluşturun ve otomatikleştirin**.\
Bugün Erişim Alın:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=local-cloud-storage" %}

## OneDrive

Windows'ta, OneDrive klasörünü `\Users\<kullanıcıadı>\AppData\Local\Microsoft\OneDrive` içinde bulabilirsiniz. Ve içinde `logs\Personal` klasöründe senkronize edilen dosyalarla ilgili bazı ilginç veriler içeren `SyncDiagnostics.log` dosyasını bulabilirsiniz:

* Bayt cinsinden boyut
* Oluşturma tarihi
* Değiştirme tarihi
* Bulutta bulunan dosya sayısı
* Klasörde bulunan dosya sayısı
* **CID**: OneDrive kullanıcısının benzersiz kimliği
* Rapor oluşturma zamanı
* İşletim sisteminin HD boyutu

CID'yi bulduktan sonra **bu kimliği içeren dosyaları aramanız önerilir**. _**\<CID>.ini**_ ve _**\<CID>.dat**_ adında dosyalar bulabilir ve bu dosyaların OneDrive ile senkronize edilen dosyaların adlarını içerebileceği ilginç bilgiler içerebilir.

## Google Drive

Windows'ta, ana Google Drive klasörünü `\Users\<kullanıcıadı>\AppData\Local\Google\Drive\user_default` içinde bulabilirsiniz.\
Bu klasör, hesabın e-posta adresi, dosya adları, zaman damgaları, dosyaların MD5 karma değerleri vb. gibi bilgiler içeren Sync\_log.log adlı bir dosya içerir. Silinmiş dosyalar bile, ilgili MD5 değeriyle birlikte o log dosyasında görünür.

**`Cloud_graph\Cloud_graph.db`** dosyası, **`cloud_graph_entry`** tablosunu içeren bir sqlite veritabanıdır. Bu tabloda **senkronize edilen dosyaların adı**, değiştirilme zamanı, boyut ve dosyaların MD5 toplamı bulunabilir.

Veritabanı **`Sync_config.db`** tablo verileri hesabın e-posta adresini, paylaşılan klasörlerin yolunu ve Google Drive sürümünü içerir.

## Dropbox

Dropbox dosyaları yönetmek için **SQLite veritabanlarını** kullanır. Bu\
Veritabanlarını aşağıdaki klasörlerde bulabilirsiniz:

* `\Users\<kullanıcıadı>\AppData\Local\Dropbox`
* `\Users\<kullanıcıadı>\AppData\Local\Dropbox\Instance1`
* `\Users\<kullanıcıadı>\AppData\Roaming\Dropbox`

Ve ana veritabanları şunlardır:

* Sigstore.dbx
* Filecache.dbx
* Deleted.dbx
* Config.dbx

".dbx" uzantısı, veritabanlarının **şifreli olduğu** anlamına gelir. Dropbox, **DPAPI**'yi kullanır ([https://docs.microsoft.com/en-us/previous-versions/ms995355(v=msdn.10)?redirectedfrom=MSDN](https://docs.microsoft.com/en-us/previous-versions/ms995355\(v=msdn.10\)?redirectedfrom=MSDN))

Dropbox'un kullandığı şifrelemeyi daha iyi anlamak için [https://blog.digital-forensics.it/2017/04/brush-up-on-dropbox-dbx-decryption.html](https://blog.digital-forensics.it/2017/04/brush-up-on-dropbox-dbx-decryption.html) adresini okuyabilirsiniz.

Ancak, ana bilgiler şunlardır:

* **Entropi**: d114a55212655f74bd772e37e64aee9b
* **Tuz**: 0D638C092E8B82FC452883F95F355B8E
* **Algoritma**: PBKDF2
* **İterasyonlar**: 1066

Bu bilgilerin yanı sıra, veritabanlarını şifrelemek için hala gerekenler:

* **Şifreli DPAPI anahtarı**: Bu veriyi `NTUSER.DAT\Software\Dropbox\ks\client` içinde kayıt defterinde bulabilirsiniz (bu veriyi ikili olarak dışa aktarın)
* **`SYSTEM`** ve **`SECURITY`** hive'ları
* **DPAPI anahtarları**: Bunlar `\Users\<kullanıcıadı>\AppData\Roaming\Microsoft\Protect` içinde bulunabilir
* Windows kullanıcısının **kullanıcı adı** ve **şifresi**

Ardından [**DataProtectionDecryptor**](https://nirsoft.net/utils/dpapi\_data\_decryptor.html)** aracını kullanabilirsiniz:**

![](<../../../.gitbook/assets/image (443).png>)

Her şey beklediğiniz gibi giderse, araç **kullanmanız gereken birincil anahtarı** belirtecektir. Orijinal anahtarı kurtarmak için, bu [cyber\_chef reçetesini](https://gchq.github.io/CyberChef/#recipe=Derive\_PBKDF2\_key\(%7B'option':'Hex','string':'98FD6A76ECB87DE8DAB4623123402167'%7D,128,1066,'SHA1',%7B'option':'Hex','string':'0D638C092E8B82FC452883F95F355B8E'%7D\)) alıntılanan anahtarı "parola" olarak reçeteye yerleştirin.

Sonuçta elde edilen onaltılık, veritabanlarını şifrelemek için kullanılan nihai anahtardır ve şununla şifrelenmiş veritabanlar şifresi çözülebilir:
```bash
sqlite -k <Obtained Key> config.dbx ".backup config.db" #This decompress the config.dbx and creates a clear text backup in config.db
```
**`config.dbx`** veritabanı şunları içerir:

- **Email**: Kullanıcının e-posta adresi
- **usernamedisplayname**: Kullanıcının adı
- **dropbox\_path**: Dropbox klasörünün bulunduğu yol
- **Host\_id**: Buluta kimlik doğrulamak için kullanılan hash. Bu sadece web üzerinden iptal edilebilir.
- **Root\_ns**: Kullanıcı kimliği

**`filecache.db`** veritabanı, Dropbox ile senkronize edilen tüm dosya ve klasörler hakkında bilgi içerir. En fazla kullanışlı bilgiye sahip olan tablo `File_journal` şudur:

- **Server\_path**: Sunucu içinde dosyanın bulunduğu yol (bu yol, istemcinin `host_id`'si tarafından önce gelir).
- **local\_sjid**: Dosyanın sürümü
- **local\_mtime**: Değiştirme tarihi
- **local\_ctime**: Oluşturma tarihi

Bu veritabanındaki diğer tablolar daha ilginç bilgiler içerir:

- **block\_cache**: Dropbox'un tüm dosya ve klasörlerinin hash'leri
- **block\_ref**: `block_cache` tablosundaki hash kimliğini `file_journal` tablosundaki dosya kimliği ile ilişkilendirir
- **mount\_table**: Dropbox'un paylaşılan klasörleri
- **deleted\_fields**: Silinen Dropbox dosyaları
- **date\_added**

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm_source=hacktricks&utm_medium=text&utm_campaign=ppc&utm_content=local-cloud-storage)'i kullanarak dünyanın en gelişmiş topluluk araçlarıyla desteklenen iş akışlarını kolayca oluşturun ve **otomatikleştirin**.\
Bugün Erişim Alın:

{% embed url="https://trickest.com/?utm_source=hacktricks&utm_medium=banner&utm_campaign=ppc&utm_content=local-cloud-storage" %}

<details>

<summary><strong>Sıfırdan kahraman olmak için AWS hackleme öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks'ı desteklemenin diğer yolları:

- **Şirketinizi HackTricks'te reklamını görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARI**](https://github.com/sponsors/carlospolop)'na göz atın!
- [**Resmi PEASS & HackTricks ürünlerine göz atın**](https://peass.creator-spring.com)
- [**The PEASS Family'yi keşfedin**](https://opensea.io/collection/the-peass-family), özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuzu keşfedin
- 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) katılın veya bizi Twitter'da 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)'ı takip edin.
- **Hacking püf noktalarınızı göndererek HackTricks ve HackTricks Cloud** github depolarına PR göndererek paylaşın.

</details>
