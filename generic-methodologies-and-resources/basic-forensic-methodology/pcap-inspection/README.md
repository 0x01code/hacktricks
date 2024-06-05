# Pcap Inspection

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS & HackTricks swag प्राप्त करें**](https://peass.creator-spring.com)
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) स्पेन में सबसे महत्वपूर्ण साइबर सुरक्षा घटना है और यूरोप में सबसे महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने** के मिशन के साथ, यह कांग्रेस प्रौद्योगिकी और साइबर सुरक्षा विशेषज्ञों के लिए एक उबाऊ मिलन स्थल है।

{% embed url="https://www.rootedcon.com/" %}

{% hint style="info" %}
**PCAP** vs **PCAPNG** के बारे में एक नोट: PCAP फ़ाइल प्रारूप के दो संस्करण हैं; **PCAPNG नया है और सभी उपकरणों द्वारा समर्थित नहीं है**। कुछ अन्य उपकरणों में इसके साथ काम करने के लिए आपको एक फ़ाइल को PCAP से PCAPNG में रूपांतरित करने की आवश्यकता हो सकती है, Wireshark या किसी अन्य संगत उपकरण का उपयोग करके।
{% endhint %}

## Pcap के लिए ऑनलाइन उपकरण

* यदि आपके pcap का हेडर **टूटा हुआ** है तो आपको इसे ठीक करने की कोशिश करनी चाहिए: [http://f00l.de/hacking/**pcapfix.php**](http://f00l.de/hacking/pcapfix.php)
* [**PacketTotal**](https://packettotal.com) में pcap में **जानकारी** और **मैलवेयर** खोजें
* [**www.virustotal.com**](https://www.virustotal.com) और [**www.hybrid-analysis.com**](https://www.hybrid-analysis.com) का उपयोग करके **दुष्ट गतिविधि** खोजें
* [**https://apackets.com/**](https://apackets.com/) में **ब्राउज़र से पूरी pcap विश्लेषण**

## जानकारी निकालें

निम्नलिखित उपकरण सांख्यिकी, फ़ाइलें, आदि निकालने के लिए उपयोगी हैं।

### Wireshark

{% hint style="info" %}
**यदि आप PCAP का विश्लेषण करने जा रहे हैं तो आपको मूल रूप से Wireshark का उपयोग कैसे करना है यह जानना चाहिए**
{% endhint %}

आप Wireshark ट्रिक्स में कुछ जानकारी यहाँ पा सकते हैं:

{% content-ref url="wireshark-tricks.md" %}
[wireshark-tricks.md](wireshark-tricks.md)
{% endcontent-ref %}

### [**https://apackets.com/**](https://apackets.com/)

ब्राउज़र से Pcap विश्लेषण।

### Xplico Framework

[**Xplico** ](https://github.com/xplico/xplico)_(केवल लिनक्स)_ एक **pcap** का विश्लेषण कर सकता है और इससे जानकारी निकाल सकता है। उदाहरण के लिए, Xplico, एक pcap फ़ाइल से प्रत्येक ईमेल (POP, IMAP, और SMTP प्रोटोकॉल), सभी HTTP सामग्री, प्रत्येक VoIP कॉल (SIP), FTP, TFTP, आदि निकालता है।

**स्थापित करें**
```bash
sudo bash -c 'echo "deb http://repo.xplico.org/ $(lsb_release -s -c) main" /etc/apt/sources.list'
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 791C25CE
sudo apt-get update
sudo apt-get install xplico
```
**चलाएं**
```
/etc/init.d/apache2 restart
/etc/init.d/xplico start
```
उपयोगकर्ता नाम _**127.0.0.1:9876**_ के साथ पहुंचें उपयोगकर्ता नाम _**xplico:xplico**_

फिर **नया मामला** बनाएं, मामले के अंदर **नई सत्र** बनाएं और **pcap** फ़ाइल **अपलोड** करें।

### NetworkMiner

Xplico की तरह यह एक उपकरण है **pcaps से वस्तुओं का विश्लेषण और निकालने** के लिए। इसका एक मुफ्त संस्करण है जिसे आप यहाँ **डाउनलोड** कर सकते हैं (https://www.netresec.com/?page=NetworkMiner)। यह **Windows** के साथ काम करता है।\
यह उपकरण भी उपयोगी है **पैकेट्स में से अन्य जानकारी** प्राप्त करने के लिए ताकि एक **तेज़ तरीके से** पता चल सके कि क्या हो रहा था।

### NetWitness Investigator

आप यहाँ से [**NetWitness Investigator डाउनलोड कर सकते हैं**](https://www.rsa.com/en-us/contact-us/netwitness-investigator-freeware) **(यह Windows में काम करता है)**।\
यह एक और उपयोगी उपकरण है जो **पैकेट्स का विश्लेषण** करता है और जानकारी को एक उपयोगी तरीके से **व्यवस्थित करता है** ताकि पता चल सके कि **अंदर क्या हो रहा है**।

### [BruteShark](https://github.com/odedshimon/BruteShark)

* उपयोगकर्ता नाम और पासवर्ड (HTTP, FTP, Telnet, IMAP, SMTP...) का निकालना और एन्कोडिंग
* प्रमाणीकरण हैश का निकालना और Hashcat का उपयोग करके उन्हें क्रैक करना (Kerberos, NTLM, CRAM-MD5, HTTP-Digest...)
* एक विजुअल नेटवर्क आरेखित करना (नेटवर्क नोड्स और उपयोगकर्ता)
* DNS क्वेरी का निकालना
* सभी TCP और UDP सत्रों का पुनर्निर्माण
* फ़ाइल कार्विंग

### Capinfos
```
capinfos capture.pcap
```
### Ngrep

यदि आप pcap के अंदर कुछ ढूंढ़ रहे हैं तो आप **ngrep** का उपयोग कर सकते हैं। यहाँ मुख्य फ़िल्टर का उपयोग करके एक उदाहरण है:
```bash
ngrep -I packets.pcap "^GET" "port 80 and tcp and host 192.168 and dst host 192.168 and src host 192.168"
```
### कार्विंग

सामान्य कार्विंग तकनीकों का उपयोग करके पीकैप से फ़ाइलें और जानकारी निकालना उपयोगी हो सकता है:

{% content-ref url="../partitions-file-systems-carving/file-data-carving-recovery-tools.md" %}
[file-data-carving-recovery-tools.md](../partitions-file-systems-carving/file-data-carving-recovery-tools.md)
{% endcontent-ref %}

### क्रेडेंशियल कैप्चरिंग

आप [https://github.com/lgandx/PCredz](https://github.com/lgandx/PCredz) जैसे उपकरणों का उपयोग कर सकते हैं ताकि आप pcap या लाइव इंटरफेस से क्रेडेंशियल को पार्स कर सकें।

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) **स्पेन** में सबसे महत्वपूर्ण साइबर सुरक्षा घटना है और **यूरोप** में सबसे महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने** के मिशन के साथ, यह कांग्रेस प्रौद्योगिकी और साइबर सुरक्षा विशेषज्ञों के लिए एक उबाऊ मिलन स्थल है।

{% embed url="https://www.rootedcon.com/" %}

## एक्सप्लॉइट्स/मैलवेयर की जाँच करें

### सुरिकाटा

**स्थापित करें और सेटअप करें**
```
apt-get install suricata
apt-get install oinkmaster
echo "url = http://rules.emergingthreats.net/open/suricata/emerging.rules.tar.gz" >> /etc/oinkmaster.conf
oinkmaster -C /etc/oinkmaster.conf -o /etc/suricata/rules
```
**पीकैप जाँचें**
```
suricata -r packets.pcap -c /etc/suricata/suricata.yaml -k none -v -l log
```
### यारापीसीएपी

[**YaraPCAP**](https://github.com/kevthehermit/YaraPcap) एक टूल है जो

* एक PCAP फ़ाइल पढ़ता है और Http स्ट्रीम निकालता है।
* किसी भी संपीडित स्ट्रीम को gzip डिफ़्लेट करता है
* हर फ़ाइल को यारा के साथ स्कैन करता है
* रिपोर्ट.txt लिखता है
* वैकल्पिक रूप से मिलने वाली फ़ाइलों को एक डिर में सहेजता है

### मैलवेयर विश्लेषण

जांच करें कि क्या आप किसी जाने-माने मैलवेयर का कोई आंकड़ा खोज सकते हैं:

{% content-ref url="../malware-analysis.md" %}
[malware-analysis.md](../malware-analysis.md)
{% endcontent-ref %}

## Zeek

> [Zeek](https://docs.zeek.org/en/master/about.html) एक पैसिव, ओपन-सोर्स नेटवर्क ट्रैफ़िक विश्लेषक है। कई ऑपरेटर्स संदेहात्मक या दुष्ट गतिविधि की जांच का समर्थन करने के लिए Zeek का उपयोग नेटवर्क सुरक्षा मॉनिटर (NSM) के रूप में करते हैं। Zeek इसके अलावा सुरक्षा डोमेन के पार एक व्यापक ट्रैफ़िक विश्लेषण कार्यों का समर्थन भी करता है, जिसमें प्रदर्शन मापन और ट्रबलशूटिंग शामिल है।

मूल रूप से, `zeek` द्वारा बनाई गई लॉग **पीकैप्स** नहीं हैं। इसलिए आपको लॉग विश्लेषण के लिए **अन्य उपकरण** का उपयोग करना होगा जहां **जानकारी** पीकैप्स के बारे में है।

### कनेक्शन जानकारी
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

[**RootedCON**](https://www.rootedcon.com/) स्पेन में सबसे महत्वपूर्ण साइबर सुरक्षा घटना है और यूरोप में सबसे महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने** के मिशन के साथ, यह कांग्रेस प्रौद्योगिकी और साइबर सुरक्षा विशेषज्ञों के लिए एक उफान मिलने का समारोह है।

{% embed url="https://www.rootedcon.com/" %}

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का **विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **जुड़ें** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) से या हमें **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
