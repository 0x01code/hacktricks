# Wireshark ट्रिक्स

## Wireshark ट्रिक्स

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें।**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके अपने हैकिंग ट्रिक्स साझा करें।**

</details>

## अपनी Wireshark कौशल को सुधारें

### ट्यूटोरियल

निम्नलिखित ट्यूटोरियल कुछ शानदार मूलभूत ट्रिक्स सीखने के लिए हैं:

* [https://unit42.paloaltonetworks.com/unit42-customizing-wireshark-changing-column-display/](https://unit42.paloaltonetworks.com/unit42-customizing-wireshark-changing-column-display/)
* [https://unit42.paloaltonetworks.com/using-wireshark-display-filter-expressions/](https://unit42.paloaltonetworks.com/using-wireshark-display-filter-expressions/)
* [https://unit42.paloaltonetworks.com/using-wireshark-identifying-hosts-and-users/](https://unit42.paloaltonetworks.com/using-wireshark-identifying-hosts-and-users/)
* [https://unit42.paloaltonetworks.com/using-wireshark-exporting-objects-from-a-pcap/](https://unit42.paloaltonetworks.com/using-wireshark-exporting-objects-from-a-pcap/)

### विश्लेषित जानकारी

**विशेषज्ञ जानकारी**

_**विश्लेषण** --> **विशेषज्ञ जानकारी**_ पर क्लिक करके आपको पैकेट्स में क्या हो रहा है का **अवलोकन** मिलेगा:

![](<../../../.gitbook/assets/image (570).png>)

**संकलित पते**

_**आँकड़े --> संकलित पते**_ के तहत आपको विभिन्न **जानकारी** मिलेगी जो wireshark द्वारा "**संकलित**" की गई है, जैसे पोर्ट/ट्रांसपोर्ट से प्रोटोकॉल, MAC से निर्माता आदि। संचार में क्या शामिल है, इसे जानना दिलचस्प होता है।

![](<../../../.gitbook/assets/image (571).png>)

**प्रोटोकॉल व्यवस्था**

_**आँकड़े --> प्रोटोकॉल व्यवस्था**_ के तहत आपको संचार में शामिल **प्रोटोकॉल** मिलेंगे और उनके बारे में डेटा।

![](<../../../.gitbook/assets/image (572).png>)

**बातचीतें**

_**आँकड़े --> बातचीतें**_ के तहत आपको संचार में हो रही बातचीतों का **सारांश** और उनके बारे में डेटा मिलेगा।

![](<../../../.gitbook/assets/image (573).png>)

**अंत-बिंदु**

_**आँकड़े --> अंत-बिंदु**_ के तहत आपको संचार में **अंत-बिंदु** का सारांश और प्रत्येक बिंदु के बारे में डेटा मिलेगा।

![](<../../../.gitbook/assets/image (575).png>)

**DNS जानकारी**

_**आँकड़े --> DNS**_ के तहत आपको DNS अनुरोध के बारे में आँकड़े मिलेंगे।

![](<../../../.gitbook/assets/image (577).png>)

**I/O ग्राफ**

_**आँकड़े --> I/O ग्राफ**_ के तहत आपको संचार का **ग्राफ** मिलेगा।

![](<../../../.gitbook/assets/image (574).png>)

### फ़िल्टर

यहां आपको प्रोटोकॉल के आधार पर wireshark फ़िल्टर मिलेंगे: [https://www.wireshark.org/docs/dfref/](https://www.wireshark.org/docs/dfref/)\
अन्य दिलचस्प फ़िल्टर:

* `(http.request or ssl.handshake.type == 1) and !(udp.port eq 1900)`
* HTTP और प्रारंभिक HTTPS ट्रैफ़िक
* `(http.request or ssl.handshake.type == 1 or tcp.flags eq 0x0002) and !(udp.port eq 1900)`
* HTTP और प्रारंभिक HTTPS ट्रैफ़िक + TCP SYN
* `(http.request or ssl.handshake.type == 1 or tcp.flags eq 0x0002 or dns) and !(udp.port eq 1900)`
* HTTP और प्रारंभिक HTTPS ट्रैफ़िक + TCP SYN + DNS अनुरोध

### खोज

यदि आप सत्रों के पैकेट्स में **सामग्री** की **खोज** करना चाहते हैं तो _CTRL+f_ दबाएं। आप मुख्य जानकारी पट
## स्थानीय होस्टनेम की पहचान

### DHCP से

वर्तमान Wireshark में `bootp` की बजाय `DHCP` की खोज करनी होगी

![](<../../../.gitbook/assets/image (404).png>)

### NBNS से

![](<../../../.gitbook/assets/image (405).png>)

## TLS की डिक्रिप्शन

### सर्वर निजी कुंजी के साथ https ट्रैफिक की डिक्रिप्शन

_edit>preference>protocol>ssl>_

![](<../../../.gitbook/assets/image (98).png>)

_Edit_ दबाएं और सर्वर और निजी कुंजी के सभी डेटा को जोड़ें (_IP, पोर्ट, प्रोटोकॉल, कुंजी फ़ाइल और पासवर्ड_)

### समानांतर सत्र कुंजियों के साथ https ट्रैफिक की डिक्रिप्शन

यह पता चलता है कि Firefox और Chrome दोनों TLS ट्रैफिक को एन्क्रिप्ट करने के लिए उपयोग की जाने वाली समानांतर सत्र कुंजी को एक फ़ाइल में लॉग करने का समर्थन करते हैं। फिर आप Wireshark को उस फ़ाइल पर इशारा कर सकते हैं और चौंकाने वाली बात! डिक्रिप्ट किया गया TLS ट्रैफिक। अधिक जानकारी के लिए: [https://redflagsecurity.net/2019/03/10/decrypting-tls-wireshark/](https://redflagsecurity.net/2019/03/10/decrypting-tls-wireshark/)\
इसे खोजने के लिए वातावरण में `SSLKEYLOGFILE` चर के लिए खोजें

साझा कुंजियों की एक फ़ाइल इस तरह दिखेगी:

![](<../../../.gitbook/assets/image (99).png>)

Wireshark में इसे आयात करने के लिए \_edit > preference > protocol > ssl > जाएं और इसे (Pre)-Master-Secret log filename में आयात करें:

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

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने की उपलब्धता चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें द्वारा PRs सबमिट करके** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud).

</details>
