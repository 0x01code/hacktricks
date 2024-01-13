<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> से!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग प्राप्त करें**](https://peass.creator-spring.com)
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**.
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>


# कार्विंग टूल्स

## Autopsy

फोरेंसिक्स में इमेजेज से फाइलें निकालने के लिए सबसे आम टूल [**Autopsy**](https://www.autopsy.com/download/) है। इसे डाउनलोड करें, इंस्टॉल करें और फाइल को "छिपी" फाइलों को खोजने के लिए इंजेस्ट करें। ध्यान दें कि Autopsy डिस्क इमेजेज और अन्य प्रकार की इमेजेज का समर्थन करता है, लेकिन साधारण फाइलों का नहीं।

## Binwalk <a id="binwalk"></a>

**Binwalk** एक टूल है जो बाइनरी फाइलों जैसे इमेजेज और ऑडियो फाइलों में एम्बेडेड फाइलों और डेटा की खोज करता है।
इसे `apt` के साथ इंस्टॉल किया जा सकता है हालांकि [सोर्स](https://github.com/ReFirmLabs/binwalk) github पर मिल सकता है।
**उपयोगी कमांड्स**:
```bash
sudo apt install binwalk #Insllation
binwalk file #Displays the embedded data in the given file
binwalk -e file #Displays and extracts some files from the given file
binwalk --dd ".*" file #Displays and extracts all files from the given file
```
## Foremost

छिपी हुई फाइलों को ढूंढने के लिए एक और आम उपकरण है **foremost**। आप foremost की कॉन्फ़िगरेशन फाइल `/etc/foremost.conf` में पा सकते हैं। यदि आप केवल कुछ विशिष्ट फाइलों की खोज करना चाहते हैं तो उन्हें uncomment करें। यदि आप कुछ भी uncomment नहीं करते हैं तो foremost अपने डिफ़ॉल्ट कॉन्फ़िगर किए गए फाइल प्रकारों की खोज करेगा।
```bash
sudo apt-get install foremost
foremost -v -i file.img -o output
#Discovered files will appear inside the folder "output"
```
## **Scalpel**

**Scalpel** एक और उपकरण है जिसका उपयोग **फाइल में एम्बेडेड फाइल्स** को खोजने और निकालने के लिए किया जा सकता है। इस मामले में आपको कॉन्फ़िगरेशन फाइल से (_/etc/scalpel/scalpel.conf_) उन फाइल प्रकारों को अनकमेंट करना होगा जिन्हें आप इससे निकालना चाहते हैं।
```bash
sudo apt-get install scalpel
scalpel file.img -o output
```
## Bulk Extractor

यह टूल काली में आता है लेकिन आप इसे यहाँ पा सकते हैं: [https://github.com/simsong/bulk\_extractor](https://github.com/simsong/bulk_extractor)

यह टूल एक इमेज को स्कैन कर सकता है और उसमें **pcaps निकालेगा**, **नेटवर्क जानकारी\(URLs, डोमेन्स, IPs, MACs, मेल्स\)** और अधिक **फाइलें**. आपको केवल करना होगा:
```text
bulk_extractor memory.img -o out_folder
```
उपकरण द्वारा एकत्रित की गई **सभी जानकारी** के माध्यम से नेविगेट करें \(पासवर्ड?\), **पैकेट्स** का **विश्लेषण** करें \(पढ़ें[ **Pcaps विश्लेषण**](../pcap-inspection/)\), **अजीब डोमेन्स** की खोज करें \(मैलवेयर से संबंधित डोमेन्स या **अस्तित्वहीन**\).

## PhotoRec

आप इसे [https://www.cgsecurity.org/wiki/TestDisk\_Download](https://www.cgsecurity.org/wiki/TestDisk_Download) पर पा सकते हैं।

इसमें GUI और CLI संस्करण शामिल हैं। आप उन **फाइल-प्रकारों** का चयन कर सकते हैं जिनकी खोज PhotoRec करेगा।

![](../../../.gitbook/assets/image%20%28524%29.png)

# विशिष्ट डेटा कार्विंग उपकरण

## FindAES

AES कुंजियों की खोज करता है उनके की शेड्यूल्स की खोज करके। 128, 192, और 256 बिट कुंजियों को खोजने में सक्षम, जैसे कि TrueCrypt और BitLocker द्वारा प्रयुक्त।

डाउनलोड [यहाँ](https://sourceforge.net/projects/findaes/).

# पूरक उपकरण

आप [**viu** ](https://github.com/atanunq/viu)का उपयोग करके टर्मिनल से चित्र देख सकते हैं।
आप लिनक्स कमांड लाइन उपकरण **pdftotext** का उपयोग करके एक पीडीएफ को टेक्स्ट में बदल सकते हैं और इसे पढ़ सकते हैं।



<details>

<summary><strong>Learn AWS hacking from zero to hero with</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में शामिल हों या मुझे **Twitter** 🐦 पर **फॉलो** करें [**@carlospolopm**](https://twitter.com/carlospolopm)**.**
* **HackTricks** के [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें।

</details>
