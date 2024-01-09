<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **अपनी हैकिंग ट्रिक्स साझा करें PRs जमा करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में.

</details>


पिंग प्रतिक्रिया में TTL:\
127 = Windows\
254 = Cisco\
बाकी, कोई लिनक्स

$1$- md5\
$2$ या $2a$ - Blowfish\
$5$- sha256\
$6$- sha512

यदि आपको नहीं पता कि किसी सेवा के पीछे क्या है, तो HTTP GET अनुरोध करने का प्रयास करें।

**UDP Scans**\
nc -nv -u -z -w 1 \<IP> 160-16

एक विशिष्ट पोर्ट पर एक खाली UDP पैकेट भेजा जाता है। यदि UDP पोर्ट खुला है, तो लक्ष्य मशीन से कोई प्रतिक्रिया वापस नहीं भेजी जाती है। यदि UDP पोर्ट बंद है, तो लक्ष्य मशीन से एक ICMP पोर्ट अनुपलब्ध पैकेट वापस भेजा जाना चाहिए।

UDP पोर्ट स्कैनिंग अक्सर अविश्वसनीय होती है, क्योंकि फ़ायरवॉल और राउटर ICMP पैकेट्स को ड्रॉप कर सकते हैं। इससे आपके स्कैन में गलत सकारात्मक परिणाम आ सकते हैं, और आप नियमित रूप से देखेंगे कि UDP पोर्ट स्कैन स्कैन की गई मशीन पर सभी UDP पोर्ट्स खुले हुए दिखा रहे हैं।\
o अधिकांश पोर्ट स्कैनर सभी उपलब्ध पोर्ट्स को स्कैन नहीं करते हैं, और आमतौर पर एक पूर्व निर्धारित सूची होती है\
"रोचक पोर्ट्स" की जो स्कैन की जाती हैं।

# CTF - ट्रिक्स

**Windows** में फाइलों की खोज के लिए **Winzip** का उपयोग करें।\
**Alternate data Streams**: _dir /r | find ":$DATA"_\
```
binwalk --dd=".*" <file> #Extract everything
binwalk -M -e -d=10000 suspicious.pdf #Extract, look inside extracted files and continue extracing (depth of 10000)
```
## क्रिप्टो

**featherduster**\

**Basae64**(6—>8) —> 0...9, a...z, A…Z,+,/\
**Base32**(5 —>8) —> A…Z, 2…7\
**Base85** (Ascii85, 7—>8) —> 0...9, a...z, A...Z, ., -, :, +, =, ^, !, /, \*, ?, &, <, >, (, ), \[, ], {, }, @, %, $, #\
**Uuencode** --> "_begin \<mode> \<filename>_" से शुरू होता है और अजीब चरित्र होते हैं\
**Xxencoding** --> "_begin \<mode> \<filename>_" से शुरू होता है और B64 होता है\
\
**Vigenere** (आवृत्ति विश्लेषण) —> [https://www.guballa.de/vigenere-solver](https://www.guballa.de/vigenere-solver)\
**Scytale** (अक्षरों का ऑफसेट) —> [https://www.dcode.fr/scytale-cipher](https://www.dcode.fr/scytale-cipher)

**25x25 = QR**

factordb.com\
rsatool

Snow --> स्पेस और टैब का उपयोग करके संदेश छिपाएं

# चरित्र

%E2%80%AE => RTL चरित्र (पेलोड्स को उल्टा लिखता है)


<details>

<summary><strong>AWS हैकिंग सीखें शून्य से नायक तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** 🐦 पर मुझे **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>
