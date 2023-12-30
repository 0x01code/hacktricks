# Wireshark ट्रिक्स

## Wireshark ट्रिक्स

<details>

<summary><strong>AWS हैकिंग सीखें शुरुआत से लेकर एक्सपर्ट तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **अपनी हैकिंग ट्रिक्स साझा करें** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके.

</details>

## अपने Wireshark कौशल में सुधार करें

### ट्यूटोरियल्स

निम्नलिखित ट्यूटोरियल्स कुछ बेसिक ट्रिक्स सीखने के लिए अद्भुत हैं:

* [https://unit42.paloaltonetworks.com/unit42-customizing-wireshark-changing-column-display/](https://unit42.paloaltonetworks.com/unit42-customizing-wireshark-changing-column-display/)
* [https://unit42.paloaltonetworks.com/using-wireshark-display-filter-expressions/](https://unit42.paloaltonetworks.com/using-wireshark-display-filter-expressions/)
* [https://unit42.paloaltonetworks.com/using-wireshark-identifying-hosts-and-users/](https://unit42.paloaltonetworks.com/using-wireshark-identifying-hosts-and-users/)
* [https://unit42.paloaltonetworks.com/using-wireshark-exporting-objects-from-a-pcap/](https://unit42.paloaltonetworks.com/using-wireshark-exporting-objects-from-a-pcap/)

### विश्लेषित जानकारी

**एक्सपर्ट जानकारी**

_**Analyze** --> **Expert Information**_ पर क्लिक करने से आपको पैकेट्स में हो रही गतिविधियों का **अवलोकन** मिलेगा:

![](<../../../.gitbook/assets/image (570).png>)

**Resolved Addresses**

_**Statistics --> Resolved Addresses**_ के अंतर्गत आपको Wireshark द्वारा "**resolved**" की गई कई **जानकारियां** मिलेंगी जैसे कि पोर्ट/ट्रांसपोर्ट से प्रोटोकॉल, MAC से निर्माता आदि। यह जानना दिलचस्प है कि संचार में क्या शामिल है।

![](<../../../.gitbook/assets/image (571).png>)

**Protocol Hierarchy**

_**Statistics --> Protocol Hierarchy**_ के अंतर्गत आपको संचार में शामिल **प्रोटोकॉल्स** और उनके बारे में डेटा मिलेगा।

![](<../../../.gitbook/assets/image (572).png>)

**Conversations**

_**Statistics --> Conversations**_ के अंतर्गत आपको संचार में होने वाली **बातचीत का सारांश** और उनके बारे में डेटा मिलेगा।

![](<../../../.gitbook/assets/image (573).png>)

**Endpoints**

_**Statistics --> Endpoints**_ के अंतर्गत आपको संचार में शामिल **एंडपॉइंट्स का सारांश** और प्रत्येक के बारे में डेटा मिलेगा।

![](<../../../.gitbook/assets/image (575).png>)

**DNS जानकारी**

_**Statistics --> DNS**_ के अंतर्गत आपको कैप्चर किए गए DNS अनुरोधों के बारे में सांख्यिकी मिलेगी।

![](<../../../.gitbook/assets/image (577).png>)

**I/O Graph**

_**Statistics --> I/O Graph**_ के अंतर्गत आपको संचार का **ग्राफ** मिलेगा।

![](<../../../.gitbook/assets/image (574).png>)

### फिल्टर्स

यहां आपको प्रोटोकॉल के आधार पर Wireshark फिल्टर मिलेंगे: [https://www.wireshark.org/docs/dfref/](https://www.wireshark.org/docs/dfref/)\
अन्य रोचक फिल्टर्स:

* `(http.request or ssl.handshake.type == 1) and !(udp.port eq 1900)`
* HTTP और प्रारंभिक HTTPS ट्रैफिक
* `(http.request or ssl.handshake.type == 1 or tcp.flags eq 0x0002) and !(udp.port eq 1900)`
* HTTP और प्रारंभिक HTTPS ट्रैफिक + TCP SYN
* `(http.request or ssl.handshake.type == 1 or tcp.flags eq 0x0002 or dns) and !(udp.port eq 1900)`
* HTTP और प्रारंभिक HTTPS ट्रैफिक + TCP SYN + DNS अनुरोध

### खोज

यदि आप सत्रों के **पैकेट्स** में **सामग्री** की **खोज** करना चाहते हैं तो _CTRL+f_ दबाएं। आप मुख्य जानकारी बार (No., Time, Source, आदि) में नई परतें जोड़ सकते हैं, दाएं बटन को दबाकर और फिर कॉलम संपादित करें।

अभ्यास: [https://www.malware-traffic-analysis.net/](https://www.malware-traffic-analysis.net)

## डोमेन्स की पहचान करना

आप एक कॉलम जोड़ सकते हैं जो HTTP हेडर के Host को दिखाता है:

![](<../../../.gitbook/assets/image (403).png>)

और एक कॉलम जो एक प्रारंभिक HTTPS कनेक्शन से Server नाम जोड़ता है (**ssl.handshake.type == 1**):

![](<../../../.gitbook/assets/image (408) (1).png>)

## स्थानीय होस्टनेम्स की पहचान करना

### DHCP से

वर्तमान Wireshark में `bootp` के बजाय आपको `DHCP` के लिए खोज करनी होगी

![](<../../../.gitbook/assets/image (404).png>)

### NBNS से

![](<../../../.gitbook/assets/image (405).png>)

## TLS डिक्रिप्ट करना

### सर्वर प्राइवेट की के साथ https ट्रैफिक डिक्रिप्ट करना

_edit>preference>protocol>ssl>_

![](<../../../.gitbook/assets/image (98).png>)

_Edit_ पर क्लिक करें और सर्वर और प्राइवेट की की सभी डेटा जोड़ें (_IP, Port, Protocol, Key file और password_)

### सिमेट्रिक सेशन कीज़ के साथ https ट्रैफिक डिक्रिप्ट करना

यह पता चला है कि Firefox और Chrome दोनों TLS ट्रैफिक को एन्क्रिप्ट करने के लिए इस्तेमाल किए गए सिमेट्रिक सेशन की को एक फाइल में लॉग करने का समर्थन करते हैं। फिर आप Wireshark को उस फाइल की ओर इशारा कर सकते हैं और तत्काल! डिक्रिप्टेड TLS ट्रैफिक। अधिक जानकारी के लिए: [https://redflagsecurity.net/2019/03/10/decrypting-tls-wireshark/](https://redflagsecurity.net/2019/03/10/decrypting-tls-wireshark/)\
इसका पता लगाने के लिए पर्यावरण के अंदर `SSLKEYLOGFILE` वेरिएबल के लिए खोज करें

शेयर्ड कीज़ की एक फाइल इस तरह दिखेगी:

![](<../../../.gitbook/assets/image (99).png>)

Wireshark में इसे आयात करने के लिए \_edit > preference > protocol > ssl > में जाएं और इसे (Pre)-Master-Secret log filename में आयात करें:

![](<../../../.gitbook/assets/image (100).png>)

## ADB संचार

ADB संचार से एक APK निकालें जहां APK भेजा गया था:
```python
from scapy.all import *

pcap = rdpcap("final2.pcapng")

def rm_data(data):
splitted = data.split(b"DATA")
if len(splitted) == 1:
return data
else:
return splitted[0]+splitted[1][4:]

all_bytes = b""
for pkt in pcap:
if Raw in pkt:
a = pkt[Raw]
if b"WRTE" == bytes(a)[:4]:
all_bytes += rm_data(bytes(a)[24:])
else:
all_bytes += rm_data(bytes(a))
print(all_bytes)

f = open('all_bytes.data', 'w+b')
f.write(all_bytes)
f.close()
```
<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो** करें**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>
