<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT संग्रह**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>


पिंग प्रतिक्रिया में TTL:\
127 = Windows\
254 = Cisco\
अन्य, कुछ लिनक्स

$1$- md5\
$2$ या $2a$ - Blowfish\
$5$- sha256\
$6$- sha512

यदि आपको पता नहीं है कि किसी सेवा के पीछे क्या है, तो एक HTTP GET अनुरोध बनाने का प्रयास करें।

**UDP स्कैन**\
nc -nv -u -z -w 1 \<IP> 160-16

एक खाली UDP पैकेट एक विशिष्ट पोर्ट पर भेजा जाता है। यदि UDP पोर्ट खुला है, तो लक्षित मशीन से कोई प्रतिक्रिया नहीं भेजी जाती है। यदि UDP पोर्ट बंद है, तो लक्षित मशीन से ICMP पोर्ट अनुपलब्ध पैकेट भेजा जाना चाहिए।

UDP पोर्ट स्कैनिंग अक्सर अविश्वसनीय होती है, क्योंकि फ़ायरवॉल और राउटर ICMP पैकेट छोड़ सकते हैं। इससे आपके स्कैन में गलत सकारात्मक परिणाम हो सकते हैं, और आप नियमित रूप से एक स्कैन के दौरान सभी UDP पोर्ट खुले दिखाई देंगे।
o अधिकांश पोर्ट स्कैनर सभी उपलब्ध पोर्टों को स्कैन नहीं करते हैं, और आमतौर पर एक पूर्व-निर्धारित सूची होती है "दिलचस्प पोर्ट" जो स्कैन की जाती है।

# CTF - ट्रिक्स

**Windows** में फ़ाइलों की खोज करने के लिए **Winzip** का उपयोग करें।\
**वैकल्पिक डेटा स्ट्रीम**: _dir /r | find ":$DATA"_\
```
binwalk --dd=".*" <file> #Extract everything
binwalk -M -e -d=10000 suspicious.pdf #Extract, look inside extracted files and continue extracing (depth of 10000)
```
## क्रिप्टो

**featherduster**\


**बेस64**(6—>8) —> 0...9, a...z, A…Z,+,/\
**बेस32**(5 —>8) —> A…Z, 2…7\
**बेस85** (Ascii85, 7—>8) —> 0...9, a...z, A...Z, ., -, :, +, =, ^, !, /, \*, ?, &, <, >, (, ), \[, ], {, }, @, %, $, #\
**Uuencode** --> "_शुरू करें \<मोड> \<फ़ाइलनाम>_" और अजीब वर्ण\
**Xxencoding** --> "_शुरू करें \<मोड> \<फ़ाइलनाम>_" और B64\
\
**विजेनेर** (आवृत्ति विश्लेषण) —> [https://www.guballa.de/vigenere-solver](https://www.guballa.de/vigenere-solver)\
**साइटेल** (वर्णों का ऑफ़सेट) —> [https://www.dcode.fr/scytale-cipher](https://www.dcode.fr/scytale-cipher)

**25x25 = QR**

factordb.com\
rsatool

Snow --> अंतरिक्ष और टैब का उपयोग करके संदेश छिपाएं

# वर्ण

%E2%80%AE => RTL वर्ण (पेलोड उल्टा लिखता है)


<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>
