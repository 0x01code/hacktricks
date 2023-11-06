# Pcap निरीक्षण

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल** हों या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें।**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपने हैकिंग ट्रिक्स साझा करें।**

</details>

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) स्पेन में सबसे महत्वपूर्ण साइबर सुरक्षा इवेंट है और यूरोप में सबसे महत्वपूर्ण माना जाता है। तकनीकी ज्ञान को बढ़ावा देने की मिशन के साथ, यह कांग्रेस प्रौद्योगिकी और साइबर सुरक्षा विशेषज्ञों के लिए एक उबलता हुआ मिलन स्थान है।

{% embed url="https://www.rootedcon.com/" %}

{% hint style="info" %}
**PCAP** vs **PCAPNG** के बारे में एक नोट: PCAP फ़ाइल प्रारूप के दो संस्करण हैं; **PCAPNG नवीनतम है और सभी उपकरणों द्वारा समर्थित नहीं है।** कुछ अन्य उपकरणों में इसके साथ काम करने के लिए आपको एक फ़ाइल को PCAP से PCAPNG में रूपांतरित करने की आवश्यकता हो सकती है, Wireshark या किसी अन्य संगत उपकरण का उपयोग करके।
{% endhint %}

## पीकैप के लिए ऑनलाइन उपकरण

* यदि आपके pcap का हैडर **टूटा हुआ** है, तो आपको इसे ठीक करने के लिए इस्तेमाल करना चाहिए: [http://f00l.de/hacking/**pcapfix.php**](http://f00l.de/hacking/pcapfix.php)
* [**PacketTotal**](https://packettotal.com) में pcap में **जानकारी** और **मैलवेयर** खोजें
* [**www.virustotal.com**](https://www.virustotal.com) और [**www.hybrid-analysis.com**](https://www.hybrid-analysis.com) का उपयोग करके **खतरनाक गतिविधि** खोजें

## जानकारी निकालें

निम्नलिखित उपकरण विशेषज्ञता, फ़ाइलें, आदि निकालने के लिए उपयोगी हैं।

### Wireshark

{% hint style="info" %}
**यदि आप PCAP का विश्लेषण करने जा रहे हैं, तो आपको मूल रूप से Wireshark का उपयोग करना चाहिए**
{% endhint %}

आप Wireshark ट्रिक्स कुछ यहां पा सकते हैं:

{% content-ref url="wireshark-tricks.md" %}
[wireshark-tricks.md](wireshark-tricks.md)
{% endcontent-ref %}

### Xplico Framework

[**Xplico** ](https://github.com/xplico/xplico)_(केवल लिनक्स)_ एक pcap का विश्लेषण कर सकता है और इससे जानकारी निकाल सकता है। उदाहरण के लिए, pcap फ़ाइल Xplico, प्रत्येक ईमेल (POP, IMAP, और SMTP प्रोटोकॉल), सभी HTTP सामग्री, प्रत्येक VoIP कॉल (SIP), FTP, TFTP, आदि निकालता है।

**स्थापित करें**
```bash
sudo bash -c 'echo "deb http://repo.xplico.org/ $(lsb_release -s -c) main" /etc/apt/sources.list'
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 791C25CE
sudo apt-get update
sudo apt-get install xplico
```
**चलाएँ**
```
/etc/init.d/apache2 restart
/etc/init.d/xplico start
```
_**127.0.0.1:9876**_ क्रेडेंशियल्स के साथ एक्सेस करें _**xplico:xplico**_

फिर से **नया मामला बनाएं**, मामले के अंदर एक **नई सत्र** बनाएं और **पीकैप** फ़ाइल अपलोड करें।

### NetworkMiner

Xplico की तरह यह एक टूल है **पीकैप से विश्लेषण और वस्तुओं को निकालने** के लिए। इसका एक मुफ्त संस्करण है जिसे आप यहां से **डाउनलोड** कर सकते हैं [**यहां**](https://www.netresec.com/?page=NetworkMiner)। यह **Windows** के साथ काम करता है।\
यह टूल भी उपयोगी है **पैकेट में से अन्य जानकारी विश्लेषित** करने के लिए ताकि आप एक **तेज़ तरीके से पता लगा सकें** कि क्या हो रहा था।

### NetWitness Investigator

आप यहां से [**NetWitness Investigator डाउनलोड कर सकते हैं**](https://www.rsa.com/en-us/contact-us/netwitness-investigator-freeware) **(यह Windows में काम करता है)**।\
यह एक और उपयोगी टूल है जो **पैकेटों का विश्लेषण करता है** और जानकारी को एक उपयोगी तरीके से **पता लगाने के लिए व्यवस्थित करता है**।

![](<../../../.gitbook/assets/image (567) (1).png>)

### [BruteShark](https://github.com/odedshimon/BruteShark)

* उपयोगकर्ता नाम और पासवर्ड (HTTP, FTP, Telnet, IMAP, SMTP...) को निकालना और एनकोड करना
* प्रमाणीकरण हैश निकालना और Hashcat का उपयोग करके उन्हें क्रैक करना (Kerberos, NTLM, CRAM-MD5, HTTP-Digest...)
* एक विज़ुअल नेटवर्क आरेख (नेटवर्क नोड और उपयोगकर्ता) बनाना
* DNS क्वेरी निकालना
* सभी TCP और UDP सत्रों का पुनर्निर्माण करना
* फ़ाइल कार्विंग

### Capinfos
```
capinfos capture.pcap
```
### Ngrep

यदि आप pcap में कुछ ढूंढ़ रहे हैं तो आप **ngrep** का उपयोग कर सकते हैं। यहां मुख्य फ़िल्टर का उपयोग करके एक उदाहरण दिया गया है:
```bash
ngrep -I packets.pcap "^GET" "port 80 and tcp and host 192.168 and dst host 192.168 and src host 192.168"
```
### कार्विंग

पीकैप से फ़ाइलें और जानकारी निकालने के लिए सामान्य कार्विंग तकनीकों का उपयोग करना उपयोगी हो सकता है:

{% content-ref url="../partitions-file-systems-carving/file-data-carving-recovery-tools.md" %}
[file-data-carving-recovery-tools.md](../partitions-file-systems-carving/file-data-carving-recovery-tools.md)
{% endcontent-ref %}

### क्रेडेंशियल्स को दर्ज करना

आप [https://github.com/lgandx/PCredz](https://github.com/lgandx/PCredz) जैसे उपकरण का उपयोग कर सकते हैं जो पीकैप या लाइव इंटरफ़ेस से क्रेडेंशियल्स को पार्स करते हैं।

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) स्पेन में सबसे महत्वपूर्ण साइबर सुरक्षा इवेंट है और यूरोप में सबसे महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने** की मिशन के साथ, यह कांग्रेस प्रौद्योगिकी और साइबर सुरक्षा विशेषज्ञों के लिए हर विषय में एक उबलता हुआ मिलने का समय है।

{% embed url="https://www.rootedcon.com/" %}

## एक्सप्लॉइट्स/मैलवेयर की जांच करें

### सुरिकाटा

**स्थापित करें और सेटअप करें**
```
apt-get install suricata
apt-get install oinkmaster
echo "url = http://rules.emergingthreats.net/open/suricata/emerging.rules.tar.gz" >> /etc/oinkmaster.conf
oinkmaster -C /etc/oinkmaster.conf -o /etc/suricata/rules
```
**पीकैप जांचें**
```
suricata -r packets.pcap -c /etc/suricata/suricata.yaml -k none -v -l log
```
### YaraPcap

[**YaraPCAP**](https://github.com/kevthehermit/YaraPcap) एक ऐसा टूल है जो

* एक PCAP फ़ाइल को पढ़ता है और Http स्ट्रीम्स को निकालता है।
* किसी भी संपीड़ित स्ट्रीम को gzip द्वारा विफलित करता है
* हर फ़ाइल को yara के साथ स्कैन करता है
* एक रिपोर्ट.txt लिखता है
* वैकल्पिक रूप से मिलने वाली फ़ाइलों को एक डिरेक्टरी में सहेजता है

### Malware Analysis

जांचें कि क्या आप किसी ज्ञात मैलवेयर के उंगली प्रिंट को खोज सकते हैं:

{% content-ref url="../malware-analysis.md" %}
[malware-analysis.md](../malware-analysis.md)
{% endcontent-ref %}

## Zeek

> Zeek एक पैशिव, ओपन-सोर्स नेटवर्क ट्रैफ़िक विश्लेषक है। कई ऑपरेटर शंकाप्रद या दुष्ट गतिविधि की जांच का समर्थन करने के लिए Zeek का उपयोग नेटवर्क सुरक्षा मॉनिटर (NSM) के रूप में करते हैं। Zeek नेटवर्क सुरक्षा डोमेन के अलावा प्रदर्शन मापन और समस्या निवारण सहित विभिन्न ट्रैफ़िक विश्लेषण कार्यों का समर्थन भी करता है।

मूल रूप से, `zeek` द्वारा बनाए गए लॉग्स **pcaps** नहीं होते हैं। इसलिए आपको लॉग्स का विश्लेषण करने के लिए **अन्य उपकरण** का उपयोग करना होगा जहां **pcaps** के बारे में **जानकारी** होती है।

### Connections Info
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
### DNS जानकारी

DNS (Domain Name System) एक प्रोटोकॉल है जो इंटरनेट पर नेटवर्क उपकरणों को डोमेन नामों को इंटरनेट पतों में बदलने के लिए उपयोग किया जाता है। DNS जानकारी का अध्ययन करने के लिए, हम पीकैप (PCAP) फ़ाइलों का उपयोग कर सकते हैं।

जब हम किसी पीकैप फ़ाइल को जांचते हैं, तो हम DNS पैकेट्स को खोज सकते हैं जो नेटवर्क ट्रैफ़िक में डोमेन नामों के साथ संबंधित होते हैं। इन पैकेट्स का अध्ययन करके, हम निम्नलिखित जानकारी प्राप्त कर सकते हैं:

- DNS प्रश्न (DNS queries): यह पैकेट्स डोमेन नामों के लिए DNS सर्वरों को प्रश्न करते हैं।
- DNS उत्तर (DNS responses): यह पैकेट्स DNS प्रश्नों का उत्तर देते हैं और डोमेन नामों को इंटरनेट पतों में बदलते हैं।
- DNS अवरोध (DNS blocking): यह पैकेट्स डोमेन नामों को अवरोधित करते हैं ताकि उपयोगकर्ता उन डोमेनों तक पहुंच नहीं पाते हैं।

DNS जानकारी का अध्ययन करने के लिए, हम Wireshark जैसे टूल का उपयोग कर सकते हैं जो पीकैप फ़ाइलों को खोलने और उन्हें विश्लेषित करने की अनुमति देता है। इसके अलावा, कई ऑनलाइन सेवाएं भी उपलब्ध हैं जो पीकैप फ़ाइलों का विश्लेषण करने में मदद कर सकती हैं।
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
## अन्य pcap विश्लेषण ट्रिक्स

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

[**RootedCON**](https://www.rootedcon.com/) स्पेन में सबसे महत्वपूर्ण साइबर सुरक्षा कार्यक्रम है और यूरोप में सबसे महत्वपूर्ण माना जाता है। तकनीकी ज्ञान को बढ़ावा देने की मिशन के साथ, यह सम्मेलन प्रौद्योगिकी और साइबर सुरक्षा विशेषज्ञों के लिए एक उत्कंठित मिलन स्थल है।

{% embed url="https://www.rootedcon.com/" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित** की जाए? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का एक्सेस** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का** **अनुसरण** करें।**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके अपने हैकिंग ट्रिक्स साझा करें।**

</details>
