# Pcap İnceleme

<details>

<summary><strong>AWS hacklemeyi sıfırdan kahramanla öğrenin</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Kırmızı Takım Uzmanı)</strong></a><strong>!</strong></summary>

HackTricks'ı desteklemenin diğer yolları:

* Şirketinizi HackTricks'te **reklamınızı görmek** veya **HackTricks'i PDF olarak indirmek** için [**ABONELİK PLANLARI**](https://github.com/sponsors/carlospolop)'na göz atın!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* [**PEASS Ailesi'ni**](https://opensea.io/collection/the-peass-family) keşfedin, özel [**NFT'lerimizden**](https://opensea.io/collection/the-peass-family) oluşan koleksiyonumuz
* 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) **katılın** veya **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)'ı **takip edin**.
* Hacking hilelerinizi [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github depolarına PR göndererek paylaşın.

</details>

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/), **İspanya**'daki en önemli siber güvenlik etkinliği ve **Avrupa**'daki en önemli etkinliklerden biridir. Teknik bilginin yayılmasını amaçlayan bu kongre, her disiplindeki teknoloji ve siber güvenlik profesyonelleri için kaynayan bir buluşma noktasıdır.

{% embed url="https://www.rootedcon.com/" %}

{% hint style="info" %}
**PCAP** vs **PCAPNG** hakkında bir not: PCAP dosya formatının iki versiyonu vardır; **PCAPNG daha yeni ve tüm araçlar tarafından desteklenmez**. Başka araçlarda çalışmak için bir dosyayı PCAPNG'den PCAP'ye Wireshark veya başka bir uyumlu araç kullanarak dönüştürmeniz gerekebilir.
{% endhint %}

## Pcap'ler için çevrimiçi araçlar

* Pcap başlığı **bozuk** ise, [http://f00l.de/hacking/**pcapfix.php**](http://f00l.de/hacking/pcapfix.php) adresinden düzeltebilirsiniz.
* Bir pcap içinde **bilgi** çıkarmak ve **kötü amaçlı yazılım** aramak için [**PacketTotal**](https://packettotal.com) kullanabilirsiniz.
* **Kötü amaçlı faaliyetleri** aramak için [**www.virustotal.com**](https://www.virustotal.com) ve [**www.hybrid-analysis.com**](https://www.hybrid-analysis.com) adreslerini kullanabilirsiniz.

## Bilgi Çıkarma

Aşağıdaki araçlar, istatistikler, dosyalar vb. çıkarmak için kullanışlıdır.

### Wireshark

{% hint style="info" %}
**Bir PCAP'ı analiz edecekseniz, temel olarak Wireshark'ı nasıl kullanacağınızı bilmelisiniz**
{% endhint %}

Wireshark hakkında bazı ipuçlarına şuradan ulaşabilirsiniz:

{% content-ref url="wireshark-tricks.md" %}
[wireshark-tricks.md](wireshark-tricks.md)
{% endcontent-ref %}

### Xplico Framework

[**Xplico** ](https://github.com/xplico/xplico)_(yalnızca linux)_ bir pcap'ı analiz edebilir ve içinden bilgi çıkarabilir. Örneğin, Xplico bir pcap dosyasından her e-postayı (POP, IMAP ve SMTP protokolleri), tüm HTTP içeriklerini, her VoIP aramasını (SIP), FTP, TFTP vb. çıkarır.

**Kurulum**
```bash
sudo bash -c 'echo "deb http://repo.xplico.org/ $(lsb_release -s -c) main" /etc/apt/sources.list'
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 791C25CE
sudo apt-get update
sudo apt-get install xplico
```
**Çalıştır**
```
/etc/init.d/apache2 restart
/etc/init.d/xplico start
```
_**127.0.0.1:9876**_ kimlik bilgileriyle erişin _**xplico:xplico**_

Ardından **yeni bir durum** oluşturun, durumun içinde **yeni bir oturum** oluşturun ve **pcap** dosyasını **yükleyin**.

### NetworkMiner

Xplico gibi, pcaplardan nesneleri **analiz etmek ve çıkarmak** için bir araçtır. Ücretsiz bir sürümü vardır ve [**buradan indirebilirsiniz**](https://www.netresec.com/?page=NetworkMiner). **Windows** ile çalışır.\
Bu araç, paketlerden **diğer bilgileri analiz etmek** için de kullanışlıdır, böylece ne olduğunu **daha hızlı bir şekilde** öğrenebilirsiniz.

### NetWitness Investigator

[**NetWitness Investigator'ı buradan indirebilirsiniz**](https://www.rsa.com/en-us/contact-us/netwitness-investigator-freeware) **(Windows'ta çalışır)**.\
Bu, paketleri **analiz eden** ve bilgileri **içeride ne olduğunu bilmek için kullanışlı bir şekilde sıralayan** başka bir kullanışlı araçtır.

### [BruteShark](https://github.com/odedshimon/BruteShark)

* Kullanıcı adlarını ve şifreleri çıkarma ve kodlama (HTTP, FTP, Telnet, IMAP, SMTP...)
* Kimlik doğrulama karma değerlerini çıkarma ve Hashcat kullanarak kırmak (Kerberos, NTLM, CRAM-MD5, HTTP-Digest...)
* Görsel bir ağ diyagramı oluşturma (Ağ düğümleri ve kullanıcılar)
* DNS sorgularını çıkarma
* Tüm TCP ve UDP Oturumlarını yeniden oluşturma
* Dosya Kesme

### Capinfos
```
capinfos capture.pcap
```
### Ngrep

Eğer pcap içinde bir şey arıyorsanız, **ngrep** kullanabilirsiniz. İşte ana filtreleri kullanarak bir örnek:
```bash
ngrep -I packets.pcap "^GET" "port 80 and tcp and host 192.168 and dst host 192.168 and src host 192.168"
```
### Oyma

Ortak oyma tekniklerini kullanarak, pcap'ten dosyaları ve bilgileri çıkarmak faydalı olabilir:

{% content-ref url="../partitions-file-systems-carving/file-data-carving-recovery-tools.md" %}
[file-data-carving-recovery-tools.md](../partitions-file-systems-carving/file-data-carving-recovery-tools.md)
{% endcontent-ref %}

### Kimlik bilgilerini yakalama

[https://github.com/lgandx/PCredz](https://github.com/lgandx/PCredz) gibi araçları kullanarak, bir pcap veya canlı bir arayüzden kimlik bilgilerini ayrıştırabilirsiniz.

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/), **İspanya**'daki en ilgili siber güvenlik etkinliği ve **Avrupa**'nın en önemlilerinden biridir. **Teknik bilginin teşvik edilmesi misyonuyla**, bu kongre, her disiplindeki teknoloji ve siber güvenlik profesyonelleri için kaynayan bir buluşma noktasıdır.

{% embed url="https://www.rootedcon.com/" %}

## Sızma Testi/Malware Kontrolü

### Suricata

**Kurulum ve yapılandırma**
```
apt-get install suricata
apt-get install oinkmaster
echo "url = http://rules.emergingthreats.net/open/suricata/emerging.rules.tar.gz" >> /etc/oinkmaster.conf
oinkmaster -C /etc/oinkmaster.conf -o /etc/suricata/rules
```
**Pcap Kontrolü**

Pcap dosyası, ağ trafiğini kaydetmek için kullanılan bir dosya formatıdır. Pcap dosyaları, ağ üzerinde gerçekleşen iletişimi analiz etmek ve sorunları tespit etmek için kullanılır. Pcap dosyalarını kontrol etmek, ağ trafiğini incelemek ve potansiyel güvenlik tehditlerini belirlemek için önemli bir adımdır.

Pcap dosyasını kontrol etmek için aşağıdaki adımları izleyebilirsiniz:

1. **Wireshark** veya benzeri bir ağ analiz aracını kullanarak pcap dosyasını açın.
2. Dosyayı açtıktan sonra, ağ trafiğini incelemek için filtreler kullanabilirsiniz. Örneğin, belirli bir IP adresine veya port numarasına sahip paketleri filtreleyebilirsiniz.
3. Ağ trafiğini analiz ederken, dikkatlice paketlerin içeriğini inceleyin. İletilen verileri, protokol bilgilerini ve diğer önemli ayrıntıları kontrol edin.
4. Potansiyel güvenlik tehditlerini belirlemek için ağ trafiğinde anormallikleri arayın. Örneğin, şüpheli IP adresleri veya anormal protokol davranışları gibi durumlar dikkatinizi çekebilir.
5. Analiz sonuçlarını kaydedin ve gerektiğinde raporlayın. Bulduğunuz güvenlik açıklarını veya sorunları ilgili kişilere bildirin.

Pcap dosyalarını kontrol etmek, ağ güvenliği ve sorun giderme süreçlerinde önemli bir rol oynar. Bu yöntem, ağ trafiğini analiz etmek ve potansiyel tehditleri belirlemek için kullanılan temel bir adımdır.
```
suricata -r packets.pcap -c /etc/suricata/suricata.yaml -k none -v -l log
```
### YaraPcap

[**YaraPCAP**](https://github.com/kevthehermit/YaraPcap), bir PCAP dosyasını okur ve HTTP akışlarını çıkarır. Sıkıştırılmış akışları gzip ile açar, her dosyayı yara ile tarar, bir rapor.txt dosyası yazar ve eşleşen dosyaları isteğe bağlı olarak bir dizine kaydeder.

### Zararlı Yazılım Analizi

Bilinen bir zararlı yazılımın herhangi bir parmak izini bulup bulamadığını kontrol edin:

{% content-ref url="../malware-analysis.md" %}
[malware-analysis.md](../malware-analysis.md)
{% endcontent-ref %}

## Zeek

> [Zeek](https://docs.zeek.org/en/master/about.html), pasif, açık kaynaklı bir ağ trafiği analizörüdür. Birçok operatör, şüpheli veya kötü amaçlı faaliyetlerin araştırılmasını desteklemek için Zeek'i Bir Ağ Güvenlik Monitörü (NSM) olarak kullanır. Zeek, ayrıca performans ölçümü ve sorun giderme de dahil olmak üzere güvenlik alanının ötesinde bir dizi trafik analizi görevini destekler.

Temel olarak, `zeek` tarafından oluşturulan günlükler **pcap** dosyaları değildir. Bu nedenle, pcaplara ilişkin bilgilerin bulunduğu günlükleri analiz etmek için **diğer araçları** kullanmanız gerekecektir.

### Bağlantı Bilgileri
```bash
#Get info about longest connections (add "grep udp" to see only udp traffic)
#The longest connection might be of malware (constant reverse shell?)
cat conn.log | zeek-cut id.orig_h id.orig_p id.resp_h id.resp_p proto service duration | sort -nrk 7 | head -n 10

10.55.100.100   49778   65.52.108.225   443     tcp     -       86222.365445
10.55.100.107   56099   111.221.29.113  443     tcp     -       86220.126151
10.55.100.110   60168   40.77.229.82    443     tcp     -       86160.119664


#Improve the metrics by summing up the total duration time for connections that have the same destination IP and Port.
cat conn.log | zeek-cut id.orig_h id.resp_h id.resp_p proto duration | awk 'BEGIN{ FS="\t" } { arr[$1 FS $2 FS $3 FS $4] += $5 } END{ for (key in arr) printf "%s%s%s\n", key, FS, arr[key] }' | sort -nrk 5 | head -n 10

10.55.100.100   65.52.108.225   443     tcp     86222.4
10.55.100.107   111.221.29.113  443     tcp     86220.1
10.55.100.110   40.77.229.82    443     tcp     86160.1

#Get the number of connections summed up per each line
cat conn.log | zeek-cut id.orig_h id.resp_h duration | awk 'BEGIN{ FS="\t" } { arr[$1 FS $2] += $3; count[$1 FS $2] += 1 } END{ for (key in arr) printf "%s%s%s%s%s\n", key, FS, count[key], FS, arr[key] }' | sort -nrk 4 | head -n 10

10.55.100.100   65.52.108.225   1       86222.4
10.55.100.107   111.221.29.113  1       86220.1
10.55.100.110   40.77.229.82    134       86160.1

#Check if any IP is connecting to 1.1.1.1
cat conn.log | zeek-cut id.orig_h id.resp_h id.resp_p proto service | grep '1.1.1.1' | sort | uniq -c

#Get number of connections per source IP, dest IP and dest Port
cat conn.log | zeek-cut id.orig_h id.resp_h id.resp_p proto | awk 'BEGIN{ FS="\t" } { arr[$1 FS $2 FS $3 FS $4] += 1 } END{ for (key in arr) printf "%s%s%s\n", key, FS, arr[key] }' | sort -nrk 5 | head -n 10


# RITA
#Something similar can be done with the tool rita
rita show-long-connections -H --limit 10 zeek_logs

+---------------+----------------+--------------------------+----------------+
|   SOURCE IP   | DESTINATION IP | DSTPORT:PROTOCOL:SERVICE |    DURATION    |
+---------------+----------------+--------------------------+----------------+
| 10.55.100.100 | 65.52.108.225  | 443:tcp:-                | 23h57m2.3655s  |
| 10.55.100.107 | 111.221.29.113 | 443:tcp:-                | 23h57m0.1262s  |
| 10.55.100.110 | 40.77.229.82   | 443:tcp:-                | 23h56m0.1197s  |

#Get connections info from rita
rita show-beacons zeek_logs | head -n 10
Score,Source IP,Destination IP,Connections,Avg Bytes,Intvl Range,Size Range,Top Intvl,Top Size,Top Intvl Count,Top Size Count,Intvl Skew,Size Skew,Intvl Dispersion,Size Dispersion
1,192.168.88.2,165.227.88.15,108858,197,860,182,1,89,53341,108319,0,0,0,0
1,10.55.100.111,165.227.216.194,20054,92,29,52,1,52,7774,20053,0,0,0,0
0.838,10.55.200.10,205.251.194.64,210,69,29398,4,300,70,109,205,0,0,0,0
```
### DNS bilgileri

DNS (Domain Name System), internet üzerindeki alan adlarını IP adreslerine çeviren bir sistemdir. Bir PCAP dosyasını incelemek için DNS bilgilerini kullanabilirsiniz.

#### DNS Sorgularını Bulma

DNS sorgularını bulmak için aşağıdaki komutu kullanabilirsiniz:

```bash
tshark -r file.pcap -Y "dns"
```

Bu komut, PCAP dosyasındaki tüm DNS trafiğini filtreleyecektir.

#### DNS Sorgularını Ayrıştırma

DNS sorgularını ayrıştırmak için aşağıdaki komutu kullanabilirsiniz:

```bash
tshark -r file.pcap -Y "dns.flags.response == 0" -T fields -e dns.qry.name
```

Bu komut, PCAP dosyasındaki tüm DNS sorgularını ayrıştıracaktır.

#### DNS Yanıtlarını Ayrıştırma

DNS yanıtlarını ayrıştırmak için aşağıdaki komutu kullanabilirsiniz:

```bash
tshark -r file.pcap -Y "dns.flags.response == 1" -T fields -e dns.resp.name -e dns.a
```

Bu komut, PCAP dosyasındaki tüm DNS yanıtlarını ayrıştıracaktır.

#### DNS Sorgularını ve Yanıtlarını Ayrıştırma

DNS sorgularını ve yanıtlarını ayrıştırmak için aşağıdaki komutu kullanabilirsiniz:

```bash
tshark -r file.pcap -Y "dns" -T fields -e dns.qry.name -e dns.a
```

Bu komut, PCAP dosyasındaki tüm DNS sorgularını ve yanıtlarını ayrıştıracaktır.
```bash
#Get info about each DNS request performed
cat dns.log | zeek-cut -c id.orig_h query qtype_name answers

#Get the number of times each domain was requested and get the top 10
cat dns.log | zeek-cut query | sort | uniq | rev | cut -d '.' -f 1-2 | rev | sort | uniq -c | sort -nr | head -n 10

#Get all the IPs
cat dns.log | zeek-cut id.orig_h query | grep 'example\.com' | cut -f 1 | sort | uniq -c

#Sort the most common DNS record request (should be A)
cat dns.log | zeek-cut qtype_name | sort | uniq -c | sort -nr

#See top DNS domain requested with rita
rita show-exploded-dns -H --limit 10 zeek_logs
```
## Diğer pcap analizi hileleri

{% content-ref url="dnscat-exfiltration.md" %}
[dnscat-exfiltration.md](dnscat-exfiltration.md)
{% endcontent-ref %}

{% content-ref url="wifi-pcap-analysis.md" %}
[wifi-pcap-analysis.md](wifi-pcap-analysis.md)
{% endcontent-ref %}

{% content-ref url="usb-keystrokes.md" %}
[usb-keystrokes.md](usb-keystrokes.md)
{% endcontent-ref %}

​

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) İspanya'daki en ilgili siber güvenlik etkinliği ve Avrupa'daki en önemli etkinliklerden biridir. Teknik bilginin yayılmasını amaçlayan bu kongre, her disiplindeki teknoloji ve siber güvenlik profesyonelleri için kaynayan bir buluşma noktasıdır.

{% embed url="https://www.rootedcon.com/" %}

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert)</strong> ile sıfırdan kahramana kadar AWS hackleme öğrenin<strong>!</strong></summary>

HackTricks'i desteklemenin diğer yolları:

* Şirketinizi HackTricks'te **reklamınızı görmek** veya HackTricks'i **PDF olarak indirmek** için [**ABONELİK PLANLARINI**](https://github.com/sponsors/carlospolop) kontrol edin!
* [**Resmi PEASS & HackTricks ürünlerini**](https://peass.creator-spring.com) edinin
* Özel [**NFT'lerden**](https://opensea.io/collection/the-peass-family) oluşan koleksiyonumuz [**The PEASS Family**](https://opensea.io/collection/the-peass-family)'yi keşfedin
* 💬 [**Discord grubuna**](https://discord.gg/hRep4RUj7f) veya [**telegram grubuna**](https://t.me/peass) **katılın** veya bizi **Twitter** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)**'da takip edin.**
* **Hacking hilelerinizi** [**HackTricks**](https://github.com/carlospolop/hacktricks) ve [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github reposuna PR göndererek paylaşın.

</details>
