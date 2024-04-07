<details>

<summary><strong>Sıfırdan kahraman olmaya kadar AWS hackleme öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Kırmızı Takım Uzmanı)</strong></a><strong>!</strong></summary>

HackTricks'ı desteklemenin diğer yolları:

* **Şirketinizi HackTricks'te reklamını görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARI**]'na (https://github.com/sponsors/carlospolop) göz atın!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* [**PEASS Ailesi'ni**](https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuz
* **Katılın** 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) veya bizi **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** takip edin.**
* **Hacking püf noktalarınızı paylaşarak PR'lar göndererek** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına katkıda bulunun.

</details>


## Temel Kavramlar

- **Akıllı Sözleşmeler**, belirli koşullar karşılandığında bir blok zincirinde yürütülen programlar olarak tanımlanır, aracısız anlaşma yürütümlerini otomatikleştirir.
- **Merkeziyetsiz Uygulamalar (dApps)**, akıllı sözleşmelere dayanarak, kullanıcı dostu bir ön uç ve şeffaf, denetlenebilir bir arka uç sunar.
- **Token'lar ve Coin'ler**, coin'lerin dijital para olarak hizmet verirken, token'lar belirli bağlamlarda değer veya sahipliği temsil eder.
- **Fayda Token'ları**, hizmetlere erişim sağlarken, **Güvenlik Token'ları** varlık sahipliğini belirtir.
- **DeFi**, merkezi otoriteler olmadan finansal hizmetler sunan Decentralized Finance'ı temsil eder.
- **DEX** ve **DAO'lar**, sırasıyla Merkeziyetsiz Borsa Platformları ve Merkeziyetsiz Otonom Organizasyonları ifade eder.

## Konsensüs Mekanizmaları

Konsensüs mekanizmaları, blok zincirinde güvenli ve kabul edilmiş işlem doğrulamalarını sağlar:
- **Proof of Work (PoW)**, işlem doğrulaması için hesaplama gücüne dayanır.
- **Proof of Stake (PoS)**, doğrulayıcıların belirli miktarda token tutmasını gerektirir ve PoW'a göre enerji tüketimini azaltır.

## Bitcoin Temelleri

### İşlemler

Bitcoin işlemleri, adresler arasında fon transferini içerir. İşlemler dijital imzalar aracılığıyla doğrulanır, yalnızca özel anahtar sahibinin transferleri başlatabileceğini garanti eder.

#### Ana Bileşenler:

- **Çoklu İmza İşlemleri**, bir işlemi yetkilendirmek için birden fazla imza gerektirir.
- İşlemler, **girişler** (fon kaynağı), **çıkışlar** (hedef), **ücretler** (madencilere ödenen) ve **scriptler** (işlem kuralları) içerir.

### Lightning Network

Birden fazla işlemi bir kanal içinde gerçekleştirerek Bitcoin'in ölçeklenebilirliğini artırmayı amaçlar, sadece son durumu blok zincirine yayınlar.

## Bitcoin Gizlilik Endişeleri

**Ortak Giriş Sahipliği** ve **UTXO Değişim Adresi Tespiti** gibi gizlilik saldırıları, işlem desenlerini sömürür. **Karıştırıcılar** ve **CoinJoin** gibi stratejiler, kullanıcılar arasındaki işlem bağlantılarını belirsizleştirerek gizliliği artırır.

## Anonim Olarak Bitcoin Edinme

Yöntemler arasında nakit işlemler, madencilik ve karıştırıcı kullanımı bulunur. **CoinJoin**, izlenebilirliği karmaşıklaştırmak için birden fazla işlemi karıştırırken, **PayJoin**, gizliliği artırmak için CoinJoin'leri normal işlemlere benzetir.

# Bitcoin Gizlilik Saldırıları

# Bitcoin Gizlilik Saldırılarının Özeti

Bitcoin dünyasında işlemlerin gizliliği ve kullanıcıların anonimliği genellikle endişe konularıdır. İşte saldırganların Bitcoin gizliliğini tehlikeye atabileceği birkaç yaygın yöntemin basitleştirilmiş bir genel bakışı.

## **Ortak Giriş Sahipliği Varsayımı**

Farklı kullanıcılara ait girişlerin genellikle aynı işlemde birleştirilmesi nadirdir, bu nedenle **aynı işlemdeki iki giriş adresinin genellikle aynı sahibe ait olduğu varsayılır**.

## **UTXO Değişim Adresi Tespiti**

Bir UTXO veya **Harcanmamış İşlem Çıktısı**, bir işlemde tamamen harcanmalıdır. Eğer sadece bir kısmı başka bir adrese gönderilirse, kalan kısım yeni bir değişim adresine gider. Gözlemciler, bu yeni adresin gönderene ait olduğunu varsayabilir, gizliliği tehlikeye atar.

### Örnek
Bunu önlemek için karıştırma hizmetleri veya birden fazla adres kullanımı sahipliği belirsizleştirmeye yardımcı olabilir.

## **Sosyal Ağlar ve Forumlar Maruziyeti**

Kullanıcılar bazen Bitcoin adreslerini çevrimiçi paylaşırlar, bu da adresi sahibiyle **kolayca ilişkilendirmeyi** sağlar.

## **İşlem Grafiği Analizi**

İşlemler grafikler halinde görselleştirilebilir, fon akışına dayanarak kullanıcılar arasındaki potansiyel bağlantıları ortaya çıkarabilir.

## **Gereksiz Giriş Heuristiği (Optimal Değişim Heuristiği)**

Bu heuristik, birden fazla giriş ve çıkış içeren işlemleri analiz ederek, değişim olarak gönderilen çıktının gönderene geri dönen değişim olduğunu tahmin etmeye dayanır.

### Örnek
```bash
2 btc --> 4 btc
3 btc     1 btc
```
## **Zorunlu Adres Tekrarı**

Saldırganlar, önceki kullanılmış adreslere küçük miktarlar gönderebilir ve umut eder ki alıcı, gelecekteki işlemlerde bu miktarları diğer girdilerle birleştirerek adresleri birbirine bağlar.

### Doğru Cüzdan Davranışı
Bu gizlilik sızıntısını önlemek için cüzdanlar, zaten kullanılmış boş adreslere gelen paraları kullanmaktan kaçınmalıdır.

## **Diğer Blockchain Analiz Teknikleri**

- **Tam Ödeme Miktarları:** Değişiklik olmadan yapılan işlemler muhtemelen aynı kullanıcıya ait iki adres arasında gerçekleşir.
- **Yuvarlanmış Sayılar:** Bir işlemdeki yuvarlanmış bir sayı, muhtemelen bir ödeme olduğunu gösterir, yuvarlanmamış çıktının muhtemelen değişiklik olduğu anlamına gelir.
- **Cüzdan Parmak İzi:** Farklı cüzdanlar benzersiz işlem oluşturma desenlerine sahiptir, analistlerin kullanılan yazılımı ve muhtemelen değişiklik adresini belirlemesine olanak tanır.
- **Miktar ve Zaman Korelasyonları:** İşlem zamanlarını veya miktarlarını açıklamak, işlemlerin izlenebilir olmasına neden olabilir.

## **Trafik Analizi**

Ağ trafiğini izleyerek, saldırganlar potansiyel olarak işlemleri veya blokları IP adresleriyle ilişkilendirebilir ve kullanıcı gizliliğini tehlikeye atabilir. Bu özellikle bir kurumun birçok Bitcoin düğümünü işletiyorsa ve işlemleri izleme yeteneklerini artırıyorsa geçerlidir.

## Daha Fazla
Gizlilik saldırıları ve savunmaları için kapsamlı bir liste için [Bitcoin Wiki'deki Bitcoin Gizliliği](https://en.bitcoin.it/wiki/Privacy) sayfasını ziyaret edin.


# Anonim Bitcoin İşlemleri

## Bitcoins'i Anonim Bir Şekilde Nasıl Alabilirsiniz

- **Nakit İşlemler**: Bitcoin edinme yoluyla nakit kullanımı.
- **Nakit Alternatifleri**: Hediye kartları satın almak ve bunları çevrimiçi olarak bitcoin'e dönüştürmek.
- **Madencilik**: Bitcoin kazanmanın en gizli yolu madencilik yapmaktır, özellikle yalnız yapıldığında, çünkü madencilik havuzları madencinin IP adresini bilebilir. [Madencilik Havuzları Bilgisi](https://en.bitcoin.it/wiki/Pooled_mining)
- **Hırsızlık**: Teorik olarak, bitcoin çalmak başka bir anonim şekilde edinme yöntemi olabilir, ancak yasa dışıdır ve önerilmez.

## Karıştırma Hizmetleri

Karıştırma hizmeti kullanarak bir kullanıcı **bitcoin gönderebilir** ve **farklı bitcoinler alabilir**, bu da orijinal sahibini izlemeyi zorlaştırır. Ancak, bu, hizmetin günlük tutmamasına ve gerçekten bitcoinleri geri vermesine güven gerektirir. Alternatif karıştırma seçenekleri arasında Bitcoin casinoları bulunmaktadır.

## CoinJoin

**CoinJoin**, farklı kullanıcılardan gelen birden fazla işlemi birleştirerek, girdileri çıktılarla eşleştirmeye çalışan herkes için işlemi karmaşık hale getirir. Etkili olmasına rağmen, benzersiz girdi ve çıktı boyutlarına sahip işlemler potansiyel olarak hala izlenebilir olabilir.

CoinJoin'i kullandığı düşünülen örnek işlemler şunları içerebilir: `402d3e1df685d1fdf82f36b220079c1bf44db227df2d676625ebcbee3f6cb22a` ve `85378815f6ee170aa8c26694ee2df42b99cff7fa9357f073c1192fff1f540238`.

Daha fazla bilgi için [CoinJoin](https://coinjoin.io/en) sayfasını ziyaret edin. Ethereum'da benzer bir hizmet için [Tornado Cash](https://tornado.cash) sayfasına göz atın, bu hizmet madencilerden gelen fonlarla işlemleri anonimleştirir.

## PayJoin

CoinJoin'in bir türevi olan **PayJoin** (veya P2EP), işlemi iki taraf arasında (örneğin, bir müşteri ve bir satıcı) normal bir işlem gibi gizler, CoinJoin'in karakteristik eşit çıktılarını içermez. Bu, tespit etmeyi son derece zorlaştırır ve işlem gözetleme kuruluşları tarafından kullanılan ortak-girdi-sahipliği heuristiğini geçersiz kılabilir.
```plaintext
2 btc --> 3 btc
5 btc     4 btc
```
Yukarıdaki gibi işlemler PayJoin olabilir, standart bitcoin işlemlerinden ayırt edilemeyen gizliliği artırır.

**PayJoin'un kullanımı geleneksel gözetim yöntemlerini önemli ölçüde bozabilir**, bu da işlem gizliliğinin peşinde umut verici bir gelişmedir.


# Kripto Paralarda Gizlilik İçin En İyi Uygulamalar

## **Cüzdan Senkronizasyon Teknikleri**

Gizliliği ve güvenliği korumak için cüzdanları blok zinciri ile senkronize etmek önemlidir. İki yöntem öne çıkıyor:

- **Tam düğüm**: Tüm blok zincirini indirerek tam bir düğüm maksimum gizliliği sağlar. Yapılan tüm işlemler yerel olarak depolanır, bu da düşmanların kullanıcının hangi işlemlere veya adreslere ilgi duyduğunu belirlemesini imkansız hale getirir.
- **İstemci tarafı blok filtreleme**: Bu yöntem, blok zincirinde her blok için filtreler oluşturmayı içerir, bu da cüzdanların belirli ilgi alanlarını ağ gözlemcilerine açığa çıkarmadan ilgili işlemleri tanımlamasına olanak tanır. Hafif cüzdanlar bu filtreleri indirir, kullanıcının adresleriyle eşleşme bulunduğunda yalnızca tam blokları alır.

## **Anonimlik İçin Tor Kullanımı**

Bitcoin'in eşler arası ağ üzerinde çalıştığı göz önüne alındığında, ağla etkileşimde bulunurken gizliliği artırmak için IP adresinizi gizlemek için Tor kullanılması önerilir.

## **Adres Tekrarını Önleme**

Gizliliği korumak için her işlem için yeni bir adres kullanmak hayati önem taşır. Adreslerin tekrar kullanılması, işlemleri aynı varlığa bağlayarak gizliliği tehlikeye atabilir. Modern cüzdanlar tasarımları aracılığıyla adres tekrarını önlemeye teşvik eder.

## **İşlem Gizliliği İçin Stratejiler**

- **Birden fazla işlem**: Bir ödemeyi birkaç işleme bölmek, işlem miktarını belirsizleştirerek gizlilik saldırılarını engelleyebilir.
- **Değişiklikten kaçınma**: Değişiklik çıktıları gerektirmeyen işlemleri tercih etmek, değişiklik tespit yöntemlerini bozarak gizliliği artırır.
- **Birden fazla değişiklik çıktısı**: Değişiklikten kaçınmak mümkün değilse, birden fazla değişiklik çıktısı oluşturmak yine de gizliliği artırabilir.

# **Monero: Anonimliğin Işığı**

Monero, dijital işlemlerde mutlak anonimliğe olan ihtiyacı ele alır ve gizlilik için yüksek bir standart belirler.

# **Ethereum: Gaz ve İşlemler**

## **Gazın Anlaşılması**

Gaz, Ethereum'da işlemleri gerçekleştirmek için gereken hesaplama çabasını ölçer ve **gwei** cinsinden fiyatlandırılır. Örneğin, 2.310.000 gwei (veya 0.00231 ETH) maliyeti olan bir işlem, bir gaz limiti ve bir taban ücret içerir, madencileri teşvik etmek için bir bahşiş içerir. Kullanıcılar fazla ödeme yapmamak için maksimum ücreti ayarlayabilir, fazlası geri ödenir.

## **İşlemlerin Yürütülmesi**

Ethereum'daki işlemler bir gönderici ve bir alıcıyı içerir, bunlar kullanıcı veya akıllı kontrat adresleri olabilir. Bir ücret gerektirir ve madenciliği yapılmalıdır. Bir işlemin temel bilgileri alıcı, göndericinin imzası, değer, isteğe bağlı veriler, gaz limiti ve ücretleri içerir. Önemli bir nokta olarak, göndericinin adresi imzadan çıkarılarak işlem verilerinde gerekli olmaz.

Bu uygulamalar ve mekanizmalar, gizlilik ve güvenliği önceliklendiren herkes için kripto paralarla etkileşimde bulunmak isteyenler için temel niteliktedir.


## Referanslar

* [https://en.wikipedia.org/wiki/Proof\_of\_stake](https://en.wikipedia.org/wiki/Proof\_of\_stake)
* [https://www.mycryptopedia.com/public-key-private-key-explained/](https://www.mycryptopedia.com/public-key-private-key-explained/)
* [https://bitcoin.stackexchange.com/questions/3718/what-are-multi-signature-transactions](https://bitcoin.stackexchange.com/questions/3718/what-are-multi-signature-transactions)
* [https://ethereum.org/en/developers/docs/transactions/](https://ethereum.org/en/developers/docs/transactions/)
* [https://ethereum.org/en/developers/docs/gas/](https://ethereum.org/en/developers/docs/gas/)
* [https://en.bitcoin.it/wiki/Privacy](https://en.bitcoin.it/wiki/Privacy#Forced\_address\_reuse)


<details>

<summary><strong>Sıfırdan kahraman olmak için AWS hackleme öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks'ı desteklemenin diğer yolları:

* **HackTricks'te şirketinizi reklamını görmek istiyorsanız** veya **HackTricks'i PDF olarak indirmek istiyorsanız** [**ABONELİK PLANLARI**](https://github.com/sponsors/carlospolop)'na göz atın!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family)'yi keşfedin, özel [**NFT'lerimiz**](https://opensea.io/collection/the-peass-family) koleksiyonumuz
* **💬 [Discord grubuna](https://discord.gg/hRep4RUj7f) veya [telegram grubuna](https://t.me/peass) katılın veya** bizi **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)'da takip edin.
* **Hacking püf noktalarınızı göndererek HackTricks ve HackTricks Cloud** github depolarına PR'lar göndererek **paylaşın**.

</details>
