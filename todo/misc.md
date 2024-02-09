<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

हैकट्रिक्स का समर्थन करने के अन्य तरीके:

* अगर आप चाहते हैं कि आपकी **कंपनी हैकट्रिक्स में विज्ञापित हो** या **हैकट्रिक्स को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live) पर **फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>


पिंग प्रतिक्रिया में TTL:\
127 = Windows\
254 = Cisco\
Lo demás,algunlinux

$1$- md5\
$2$or $2a$ - Blowfish\
$5$- sha256\
$6$- sha512

अगर आपको पता नहीं है कि किसी सेवा के पीछे क्या है, तो एक HTTP GET अनुरोध बनाने का प्रयास करें।

**UDP स्कैन**\
nc -nv -u -z -w 1 \<IP> 160-16

एक खाली UDP पैकेट एक विशिष्ट पोर्ट पर भेजा जाता है। यदि UDP पोर्ट खुला है, तो लक्षित मशीन से कोई प्रतिक्रिया नहीं भेजी जाती है। यदि UDP पोर्ट बंद है, तो लक्षित मशीन से एक ICMP पोर्ट अनुपलब्ध पैकेट भेजा जाना चाहिए।

UDP पोर्ट स्कैनिंग अक्सर अविश्वसनीय होती है, क्योंकि फ़ायरवॉल और राउटर ICMP पैकेट्स को छोड़ सकते हैं। यह आपके स्कैन में गलत सकारात्मक परिणाम देने की संभावना है, और आप नियमित रूप से देखेंगे कि UDP पोर्ट स्कैन में स्कैन किए गए मशीन पर सभी UDP पोर्ट खुले हुए दिखाई देते हैं।\
o अधिकांश पोर्ट स्कैनर सभी उपलब्ध पोर्टों को स्कैन नहीं करते हैं, और आम तौर पर "रोचक पोर्ट" की पूर्वनिर्धारित सूची होती है जो स्कैन की जाती है।

# CTF - ट्रिक्स

**Windows** में फ़ाइलों की खोज के लिए **Winzip** का उपयोग करें।\
**वैकल्पिक डेटा स्ट्रीम्स**: _dir /r | find ":$DATA"_\
```
binwalk --dd=".*" <file> #Extract everything
binwalk -M -e -d=10000 suspicious.pdf #Extract, look inside extracted files and continue extracing (depth of 10000)
```
## गुप्त

**featherduster**\

**Basae64**(6—>8) —> 0...9, a...z, A…Z,+,/\
**Base32**(5 —>8) —> A…Z, 2…7\
**Base85** (Ascii85, 7—>8) —> 0...9, a...z, A...Z, ., -, :, +, =, ^, !, /, \*, ?, &, <, >, (, ), \[, ], {, }, @, %, $, #\
**Uuencode** --> "_begin \<mode> \<filename>_" से शुरू होता है और अजीब वर्ण\
**Xxencoding** --> "_begin \<mode> \<filename>_" से शुरू होता है और B64\
\
**Vigenere** (आवृत्ति विश्लेषण) —> [https://www.guballa.de/vigenere-solver](https://www.guballa.de/vigenere-solver)\
**Scytale** (वर्णों का ओफसेट) —> [https://www.dcode.fr/scytale-cipher](https://www.dcode.fr/scytale-cipher)

**25x25 = QR**

factordb.com\
rsatool

Snow --> अंतरिक्ष और टैब का उपयोग करके संदेश छुपाना

# वर्ण

%E2%80%AE => RTL वर्ण (पेयलोड को पीछे से लिखता है)


<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **जुड़ें** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** हैकट्रिक्स और हैकट्रिक्स क्लाउड github रेपो में PR जमा करके।

</details>
