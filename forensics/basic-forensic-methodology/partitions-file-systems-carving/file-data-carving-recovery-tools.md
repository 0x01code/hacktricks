<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> से!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**.
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें.

</details>

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

उन कमजोरियों को खोजें जो सबसे महत्वपूर्ण हैं ताकि आप उन्हें तेजी से ठीक कर सकें। Intruder आपकी अटैक सरफेस को ट्रैक करता है, प्रोएक्टिव थ्रेट स्कैन चलाता है, और आपके पूरे टेक स्टैक में मुद्दों को खोजता है, APIs से लेकर वेब ऐप्स और क्लाउड सिस्टम्स तक। आज ही [**मुफ्त में इसे आजमाएं**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks)।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

# Carving & Recovery tools

अधिक टूल्स [https://github.com/Claudio-C/awesome-datarecovery](https://github.com/Claudio-C/awesome-datarecovery) पर

## Autopsy

फोरेंसिक्स में इमेजेस से फाइलें निकालने के लिए सबसे आम टूल [**Autopsy**](https://www.autopsy.com/download/) है। इसे डाउनलोड करें, इंस्टॉल करें और फाइल को "छिपी" फाइलें खोजने के लिए इंजेस्ट करें। ध्यान दें कि Autopsy डिस्क इमेजेस और अन्य प्रकार की इमेजेस को सपोर्ट करने के लिए बनाया गया है, लेकिन सिंपल फाइल्स को नहीं।

## Binwalk <a href="#binwalk" id="binwalk"></a>

**Binwalk** एक टूल है जो बाइनरी फाइल्स जैसे कि इमेजेस और ऑडियो फाइल्स में एम्बेडेड फाइल्स और डेटा की खोज करता है।\
इसे `apt` के साथ इंस्टॉल किया जा सकता है हालांकि [सोर्स](https://github.com/ReFirmLabs/binwalk) github पर मिल सकता है।\
**उपयोगी कमांड्स**:
```bash
sudo apt install binwalk #Insllation
binwalk file #Displays the embedded data in the given file
binwalk -e file #Displays and extracts some files from the given file
binwalk --dd ".*" file #Displays and extracts all files from the given file
```
## Foremost

छिपी हुई फाइलों को ढूंढने के लिए एक और सामान्य उपकरण है **foremost**। आप foremost की कॉन्फ़िगरेशन फाइल `/etc/foremost.conf` में पा सकते हैं। यदि आप केवल कुछ विशिष्ट फाइलों की खोज करना चाहते हैं तो उन्हें अनकमेंट करें। यदि आप कुछ भी अनकमेंट नहीं करते हैं तो foremost अपने डिफ़ॉल्ट कॉन्फ़िगर किए गए फाइल प्रकारों की खोज करेगा।
```bash
sudo apt-get install foremost
foremost -v -i file.img -o output
#Discovered files will appear inside the folder "output"
```
## **Scalpel**

**Scalpel** एक और उपकरण है जिसका उपयोग **फाइल में एम्बेडेड फाइलों** को खोजने और निकालने के लिए किया जा सकता है। इस मामले में, आपको कॉन्फ़िगरेशन फाइल (_/etc/scalpel/scalpel.conf_) से उन फाइल प्रकारों को अनकमेंट करना होगा जिन्हें आप इससे निकालना चाहते हैं।
```bash
sudo apt-get install scalpel
scalpel file.img -o output
```
## Bulk Extractor

यह टूल काली में आता है लेकिन आप इसे यहाँ पा सकते हैं: [https://github.com/simsong/bulk\_extractor](https://github.com/simsong/bulk\_extractor)

यह टूल एक इमेज को स्कैन कर सकता है और उसमें **pcaps निकालेगा**, **नेटवर्क जानकारी (URLs, डोमेन्स, IPs, MACs, मेल्स)** और अधिक **फाइलें**. आपको केवल करना होगा:
```
bulk_extractor memory.img -o out_folder
```
उपकरण द्वारा एकत्रित की गई **सभी जानकारी** के माध्यम से नेविगेट करें (पासवर्ड?), **विश्लेषण** करें **पैकेट्स** का (पढ़ें[ **Pcaps विश्लेषण**](../pcap-inspection/)), खोजें **अजीब डोमेन्स** (मैलवेयर या **अस्तित्वहीन** से संबंधित डोमेन्स).

## PhotoRec

आप इसे [https://www.cgsecurity.org/wiki/TestDisk\_Download](https://www.cgsecurity.org/wiki/TestDisk\_Download) पर पा सकते हैं।

इसमें GUI और CLI संस्करण शामिल हैं। आप उन **फाइल-प्रकारों** का चयन कर सकते हैं जिनकी खोज PhotoRec करेगा।

![](<../../../.gitbook/assets/image (524).png>)

## binvis

कोड की जांच करें [code](https://code.google.com/archive/p/binvis/) और [वेब पेज उपकरण](https://binvis.io/#/).

### BinVis की विशेषताएं

* दृश्य और सक्रिय **संरचना दर्शक**
* विभिन्न फोकस बिंदुओं के लिए एकाधिक प्लॉट्स
* नमूने के हिस्सों पर ध्यान केंद्रित करना
* PE या ELF निष्पादन योग्य फाइलों में **स्ट्रिंग्स और संसाधनों** को **देखना**
* फाइलों पर क्रिप्टोएनालिसिस के लिए **पैटर्न्स** प्राप्त करना
* पैकर या एनकोडर एल्गोरिदम की **पहचान करना**
* पैटर्न्स द्वारा स्टेगानोग्राफी की **पहचान करना**
* **दृश्य** बाइनरी-डिफिंग

BinVis एक अज्ञात लक्ष्य के साथ परिचित होने के लिए एक शानदार **प्रारंभिक बिंदु** है एक ब्लैक-बॉक्सिंग परिदृश्य में।

# विशिष्ट डेटा कार्विंग उपकरण

## FindAES

AES कुंजियों की खोज करता है उनके की शेड्यूल की खोज करके। 128, 192, और 256 बिट कुंजियों को खोजने में सक्षम, जैसे कि TrueCrypt और BitLocker द्वारा प्रयुक्त।

यहां डाउनलोड करें [here](https://sourceforge.net/projects/findaes/).

# पूरक उपकरण

आप [**viu** ](https://github.com/atanunq/viu)का उपयोग करके टर्मिनल से चित्र देख सकते हैं।\
आप लिनक्स कमांड लाइन उपकरण **pdftotext** का उपयोग करके एक पीडीएफ को टेक्स्ट में बदल सकते हैं और इसे पढ़ सकते हैं।


<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

सबसे महत्वपूर्ण कमजोरियों का पता लगाएं ताकि आप उन्हें तेजी से ठीक कर सकें। Intruder आपकी अटैक सरफेस की निगरानी करता है, सक्रिय खतरा स्कैन चलाता है, आपके पूरे टेक स्टैक में मुद्दों का पता लगाता है, APIs से लेकर वेब ऐप्स और क्लाउड सिस्टम्स तक। आज ही [**मुफ्त में आजमाएं**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks)।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का अन्य तरीकों से समर्थन करें:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** का अनुसरण करें।**
* **HackTricks** के लिए अपनी हैकिंग ट्रिक्स साझा करें PRs जमा करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>
