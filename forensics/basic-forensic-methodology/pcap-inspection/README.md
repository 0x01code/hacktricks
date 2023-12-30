# Pcap निरीक्षण

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **अपनी हैकिंग ट्रिक्स साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके.

</details>

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) **स्पेन** में साइबर सुरक्षा की सबसे प्रासंगिक घटना है और **यूरोप** में सबसे महत्वपूर्ण में से एक है। तकनीकी ज्ञान को बढ़ावा देने के मिशन के साथ, यह कांग्रेस प्रौद्योगिकी और साइबर सुरक्षा पेशेवरों के लिए एक उबलता मिलन बिंदु है।

{% embed url="https://www.rootedcon.com/" %}

{% hint style="info" %}
**PCAP** बनाम **PCAPNG** के बारे में एक नोट: PCAP फाइल प्रारूप के दो संस्करण हैं; **PCAPNG नया है और सभी टूल्स द्वारा समर्थित नहीं है**। कुछ अन्य टूल्स में काम करने के लिए आपको PCAPNG से PCAP में फाइल को परिवर्तित करने की आवश्यकता हो सकती है, इसके लिए Wireshark या कोई अन्य संगत टूल का उपयोग करें।
{% endhint %}

## पीकैप्स के लिए ऑनलाइन टूल्स

* यदि आपके pcap का हेडर **टूटा हुआ** है तो आपको इसे **ठीक** करने की कोशिश करनी चाहिए: [http://f00l.de/hacking/**pcapfix.php**](http://f00l.de/hacking/pcapfix.php)
* पीकैप के अंदर **जानकारी** निकालें और **मैलवेयर** के लिए खोज करें [**PacketTotal**](https://packettotal.com) में
* [**www.virustotal.com**](https://www.virustotal.com) और [**www.hybrid-analysis.com**](https://www.hybrid-analysis.com) का उपयोग करके **दुर्भावनापूर्ण गतिविधि** के लिए खोज करें

## जानकारी निकालें

निम्नलिखित टूल्स आंकड़े, फाइलें, आदि निकालने के लिए उपयोगी हैं।

### Wireshark

{% hint style="info" %}
**यदि आप PCAP का विश्लेषण करने जा रहे हैं तो आपको बुनियादी रूप से Wireshark का उपयोग करना आना चाहिए**
{% endhint %}

आप Wireshark ट्रिक्स पा सकते हैं:

{% content-ref url="wireshark-tricks.md" %}
[wireshark-tricks.md](wireshark-tricks.md)
{% endcontent-ref %}

### Xplico Framework

[**Xplico**](https://github.com/xplico/xplico) _(केवल linux)_ एक **pcap** का **विश्लेषण** कर सकता है और उससे जानकारी निकाल सकता है। उदाहरण के लिए, Xplico एक pcap फाइल से प्रत्येक ईमेल (POP, IMAP, और SMTP प्रोटोकॉल), सभी HTTP सामग्री, प्रत्येक VoIP कॉल (SIP), FTP, TFTP, आदि निकालता है।

**इंस्टॉल करें**
```bash
sudo bash -c 'echo "deb http://repo.xplico.org/ $(lsb_release -s -c) main" /etc/apt/sources.list'
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 791C25CE
sudo apt-get update
sudo apt-get install xplico
```
**रन**
```
/etc/init.d/apache2 restart
/etc/init.d/xplico start
```
प्रवेश करें _**127.0.0.1:9876**_ प्रमाणपत्रों के साथ _**xplico:xplico**_

फिर एक **नया मामला** बनाएं, मामले के अंदर एक **नया सत्र** बनाएं और **pcap फ़ाइल अपलोड करें**।

### NetworkMiner

Xplico की तरह यह भी एक उपकरण है जो **pcaps से वस्तुओं का विश्लेषण और निष्कर्षण करता है**। इसका एक मुफ्त संस्करण है जिसे आप **डाउनलोड कर सकते हैं** [**यहाँ**](https://www.netresec.com/?page=NetworkMiner)। यह **Windows** के साथ काम करता है।\
यह उपकरण पैकेट्स से **अन्य जानकारी का विश्लेषण करने** में भी उपयोगी है ताकि यह जान सकें कि **तेजी से** क्या हो रहा था।

### NetWitness Investigator

आप [**NetWitness Investigator यहाँ से डाउनलोड कर सकते हैं**](https://www.rsa.com/en-us/contact-us/netwitness-investigator-freeware) **(यह Windows में काम करता है)**।\
यह एक और उपयोगी उपकरण है जो **पैकेट्स का विश्लेषण करता है** और जानकारी को इस तरह से व्यवस्थित करता है ताकि **अंदर क्या हो रहा है यह जान सकें**।

![](<../../../.gitbook/assets/image (567) (1).png>)

### [BruteShark](https://github.com/odedshimon/BruteShark)

* उपयोगकर्ता नाम और पासवर्ड निकालना और कोडिंग करना (HTTP, FTP, Telnet, IMAP, SMTP...)
* प्रमाणीकरण हैश निकालना और Hashcat का उपयोग करके उन्हें क्रैक करना (Kerberos, NTLM, CRAM-MD5, HTTP-Digest...)
* एक दृश्य नेटवर्क आरेख बनाना (नेटवर्क नोड्स और उपयोगकर्ता)
* DNS प्रश्न निकालना
* सभी TCP और UDP सत्रों का पुनर्निर्माण
* फ़ाइल कार्विंग

### Capinfos
```
capinfos capture.pcap
```
### Ngrep

यदि आप pcap के अंदर **कुछ** खोज रहे हैं, तो आप **ngrep** का उपयोग कर सकते हैं। यहाँ मुख्य फिल्टर्स का उपयोग करते हुए एक उदाहरण है:
```bash
ngrep -I packets.pcap "^GET" "port 80 and tcp and host 192.168 and dst host 192.168 and src host 192.168"
```
### कार्विंग

पीसीएपी से फाइलों और जानकारी निकालने के लिए सामान्य कार्विंग तकनीकों का उपयोग करना उपयोगी हो सकता है:

{% content-ref url="../partitions-file-systems-carving/file-data-carving-recovery-tools.md" %}
[file-data-carving-recovery-tools.md](../partitions-file-systems-carving/file-data-carving-recovery-tools.md)
{% endcontent-ref %}

### क्रेडेंशियल्स कैप्चरिंग

आप [https://github.com/lgandx/PCredz](https://github.com/lgandx/PCredz) जैसे टूल्स का उपयोग करके पीसीएपी या लाइव इंटरफेस से क्रेडेंशियल्स पार्स कर सकते हैं।

<figure><img src="https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-L_2uGJGU7AVNRcqRvEi%2Fuploads%2FelPCTwoecVdnsfjxCZtN%2Fimage.png?alt=media&#x26;token=9ee4ff3e-92dc-471c-abfe-1c25e446a6ed" alt=""><figcaption></figcaption></figure>

[**RootedCON**](https://www.rootedcon.com/) **स्पेन** में सबसे महत्वपूर्ण साइबरसिक्योरिटी इवेंट है और **यूरोप** में सबसे अधिक महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने के मिशन** के साथ, यह कांग्रेस हर अनुशासन में प्रौद्योगिकी और साइबरसिक्योरिटी पेशेवरों के लिए एक उबलता मिलन बिंदु है।

{% embed url="https://www.rootedcon.com/" %}

## एक्सप्लॉइट्स/मैलवेयर की जांच करें

### सुरिकाटा

**इंस्टॉल और सेटअप**
```
apt-get install suricata
apt-get install oinkmaster
echo "url = http://rules.emergingthreats.net/open/suricata/emerging.rules.tar.gz" >> /etc/oinkmaster.conf
oinkmaster -C /etc/oinkmaster.conf -o /etc/suricata/rules
```
**pcap की जांच करें**
```
suricata -r packets.pcap -c /etc/suricata/suricata.yaml -k none -v -l log
```
### YaraPcap

[**YaraPCAP**](https://github.com/kevthehermit/YaraPcap) एक उपकरण है जो

* PCAP फ़ाइल पढ़ता है और Http स्ट्रीम्स निकालता है।
* किसी भी संपीड़ित स्ट्रीम्स को gzip deflate करता है
* हर फ़ाइल को yara से स्कैन करता है
* report.txt लिखता है
* वैकल्पिक रूप से मिलान करने वाली फ़ाइलों को Dir में सहेजता है

### Malware Analysis

जांचें कि क्या आप किसी ज्ञात malware के फिंगरप्रिंट का पता लगा सकते हैं:

{% content-ref url="../malware-analysis.md" %}
[malware-analysis.md](../malware-analysis.md)
{% endcontent-ref %}

## Zeek

> Zeek एक पैसिव, ओपन-सोर्स नेटवर्क ट्रैफिक एनालाइज़र है। कई ऑपरेटर्स Zeek का उपयोग नेटवर्क सिक्योरिटी मॉनिटर (NSM) के रूप में संदिग्ध या मैलिशियस गतिविधि की जांच के समर्थन में करते हैं। Zeek सुरक्षा डोमेन से परे विभिन्न प्रकार के ट्रैफिक विश्लेषण कार्यों का भी समर्थन करता है, जिसमें प्रदर्शन मापन और समस्या निवारण शामिल हैं।

मूल रूप से, `zeek` द्वारा बनाए गए लॉग **pcaps** नहीं होते हैं। इसलिए आपको लॉग्स का विश्लेषण करने के लिए **अन्य उपकरणों** का उपयोग करना होगा जहां pcaps के बारे में **जानकारी** होती है।

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
## अन्य pcap विश्लेषण तरकीबें

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

[**RootedCON**](https://www.rootedcon.com/) **स्पेन** में साइबर सुरक्षा की सबसे महत्वपूर्ण घटना है और **यूरोप** में सबसे अधिक महत्वपूर्ण में से एक है। **तकनीकी ज्ञान को बढ़ावा देने के मिशन** के साथ, यह कांग्रेस हर अनुशासन में प्रौद्योगिकी और साइबर सुरक्षा पेशेवरों के लिए एक उबलता मिलन बिंदु है।

{% embed url="https://www.rootedcon.com/" %}

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से हीरो तक AWS हैकिंग सीखें</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>!</strong></a></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**।
* **HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग तरकीबें साझा करें।

</details>
