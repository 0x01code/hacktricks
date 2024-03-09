# फ़ाइल/डेटा कार्विंग और रिकवरी टूल्स

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और **हमें** **ट्विटर** 🐦 [**@hacktricks\_live**](https://twitter.com/hacktricks\_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>

## कार्विंग और रिकवरी टूल्स

अधिक टूल [https://github.com/Claudio-C/awesome-datarecovery](https://github.com/Claudio-C/awesome-datarecovery) में हैं।

### Autopsy

जांच में सबसे आम टूल जो छवियों से फ़ाइलें निकालने के लिए उपयोग किया जाता है [**Autopsy**](https://www.autopsy.com/download/) है। इसे डाउनलोड करें, इंस्टॉल करें और फ़ाइल को इंजेस्ट करने के लिए इसे इंगेस्ट करें ताकि "छिपी" फ़ाइलें मिलें। ध्यान दें कि Autopsy को डिस्क छवियों और अन्य प्रकार की छवियों का समर्थन करने के लिए बनाया गया है, लेकिन साधारण फ़ाइलों का नहीं।

### Binwalk <a href="#binwalk" id="binwalk"></a>

**Binwalk** एक टूल है जो बाइनरी फ़ाइलों का विश्लेषण करने के लिए उपयुक्त है ताकि एम्बेडेड सामग्री को खोजा जा सके। इसे `apt` के माध्यम से स्थापित किया जा सकता है और इसका स्रोत [GitHub](https://github.com/ReFirmLabs/binwalk) पर है।

**उपयोगी कमांड**:
```bash
sudo apt install binwalk #Insllation
binwalk file #Displays the embedded data in the given file
binwalk -e file #Displays and extracts some files from the given file
binwalk --dd ".*" file #Displays and extracts all files from the given file
```
### Foremost

छिपे हुए फ़ाइलें खोजने के लिए एक और सामान्य उपकरण है **foremost**। आप foremost का विन्यास फ़ाइल `/etc/foremost.conf` में पा सकते हैं। यदि आप कुछ विशिष्ट फ़ाइलों की खोज करना चाहते हैं तो उन्हें uncomment करें। यदि आप कुछ uncomment नहीं करते हैं तो foremost अपने डिफ़ॉल्ट विन्यसित फ़ाइल प्रकारों के लिए खोज करेगा।
```bash
sudo apt-get install foremost
foremost -v -i file.img -o output
#Discovered files will appear inside the folder "output"
```
### **Scalpel**

**Scalpel** एक औजार है जिसका उपयोग **एक फ़ाइल में एम्बेड किए गए फ़ाइलें** खोजने और निकालने के लिए किया जा सकता है। इस मामले में, आपको उसे निकालना चाहिए जिसे आप निकालना चाहते हैं, इसके लिए कॉन्फ़िगरेशन फ़ाइल (_/etc/scalpel/scalpel.conf_) से टिप्पणी हटानी होगी।
```bash
sudo apt-get install scalpel
scalpel file.img -o output
```
### Bulk Extractor

यह टूल काली के अंदर आता है लेकिन आप इसे यहाँ पा सकते हैं: [https://github.com/simsong/bulk\_extractor](https://github.com/simsong/bulk\_extractor)

यह टूल एक इमेज स्कैन कर सकता है और उसमें **pcaps को निकाल सकता है**, **नेटवर्क सूचना (URLs, डोमेन, IPs, MACs, मेल्स)** और अधिक **फ़ाइलें**। आपको केवल यह करना है:
```
bulk_extractor memory.img -o out_folder
```
### PhotoRec

आप इसे [यहाँ](https://www.cgsecurity.org/wiki/TestDisk\_Download) पा सकते हैं।

यह GUI और CLI संस्करणों के साथ आता है। आप फाइल प्रकार चुन सकते हैं जिन्हें PhotoRec खोजने के लिए।

![](<../../../.gitbook/assets/image (524).png>)

### binvis

कोड की [जाँच](https://code.google.com/archive/p/binvis/) और [वेब पेज टूल](https://binvis.io/#/) करें।

#### BinVis की विशेषताएँ

* दृश्यात्मक और सक्रिय **संरचना दर्शक**
* विभिन्न ध्यान केंद्रों के लिए एकाधिक चित्र
* नमूने के भागों पर ध्यान केंद्रित करना
* PE या ELF कार्यक्रमों में **स्ट्रिंग और संसाधन** देखना जैसे
* फ़ाइलों पर क्रिप्टोएनालिसिस के लिए **पैटर्न**
* पैकर या इन्कोडर एल्गोरिदम की **पहचान**
* पैटर्न द्वारा स्टेगेनोग्राफी की **पहचान**
* **दृश्यात्मक** बाइनरी-डिफिंग

BinVis एक महान **शुरुआती बिंदु है** एक अज्ञात लक्ष्य के साथ एक काला-बॉक्सिंग स्थिति में परिचित होने के लिए।

## विशेष डेटा कार्विंग उपकरण

### FindAES

AES कुंजियों की खोज करने के लिए उनकी कुंजी अनुसूचियों की खोज करके FindAES खोजता है। 128, 192, और 256 बिट कुंजियों को खोजने में सक्षम है, जैसे कि TrueCrypt और BitLocker द्वारा उपयोग की गई कुंजियाँ।

यहाँ से [डाउनलोड करें](https://sourceforge.net/projects/findaes/).

## पूरक उपकरण

आप [**viu** ](https://github.com/atanunq/viu) का उपयोग करके छवियों को देखने के लिए कर सकते हैं।\
आप लिनक्स कमांड लाइन उपकरण **pdftotext** का उपयोग करके एक पीडीएफ को पाठ में परिवर्तित करने और पढ़ने के लिए कर सकते हैं।
