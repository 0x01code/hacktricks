<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सदस्यता योजनाएं देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks) github repos में PRs सबमिट करके।

</details>


# खुदाई उपकरण

## Autopsy

जांच में सबसे आम उपकरण जो छवियों से फ़ाइलें निकालने के लिए उपयोग किया जाता है [**Autopsy**](https://www.autopsy.com/download/) है। इसे डाउनलोड करें, स्थापित करें और फ़ाइल को इंजेस्ट करने के लिए इसे बनाएं "छिपी" फ़ाइलें खोजने के लिए। ध्यान दें कि Autopsy को डिस्क छवियों और अन्य प्रकार की छवियों का समर्थन करने के लिए बनाया गया है, लेकिन साधारण फ़ाइलों का नहीं।

## Binwalk <a id="binwalk"></a>

**Binwalk** एक उपकरण है जो छवियों और ऑडियो फ़ाइलों जैसी बाइनरी फ़ाइलों को खोजने के लिए है जिसमें एम्बेडेड फ़ाइलें और डेटा होता है।
इसे `apt` के साथ स्थापित किया जा सकता है हालांकि [स्रोत](https://github.com/ReFirmLabs/binwalk) github पर मिल सकता है।
**उपयोगी कमांड**:
```bash
sudo apt install binwalk #Insllation
binwalk file #Displays the embedded data in the given file
binwalk -e file #Displays and extracts some files from the given file
binwalk --dd ".*" file #Displays and extracts all files from the given file
```
## Foremost

छिपे फ़ाइलें खोजने के लिए एक और सामान्य उपकरण है **foremost**। आप foremost का विन्यास फ़ाइल `/etc/foremost.conf` में पा सकते हैं। यदि आप कुछ विशिष्ट फ़ाइलें खोजना चाहते हैं तो उन्हें uncomment करें। यदि आप कुछ uncomment नहीं करते हैं तो foremost अपने डिफ़ॉल्ट विन्यासित फ़ाइल प्रकारों के लिए खोज करेगा।
```bash
sudo apt-get install foremost
foremost -v -i file.img -o output
#Discovered files will appear inside the folder "output"
```
## **Scalpel**

**Scalpel** एक और उपकरण है जिसका उपयोग **एक फ़ाइल में एम्बेड किए गए फ़ाइलें खोजने और निकालने** के लिए किया जा सकता है। इस मामले में, आपको उसे निकालने के लिए फ़ाइल प्रकारों को कॉन्फ़िगरेशन फ़ाइल \(_/etc/scalpel/scalpel.conf_\) से uncomment करने की आवश्यकता होगी।
```bash
sudo apt-get install scalpel
scalpel file.img -o output
```
## Bulk Extractor

यह टूल काली के अंदर आता है लेकिन आप इसे यहाँ पा सकते हैं: [https://github.com/simsong/bulk\_extractor](https://github.com/simsong/bulk_extractor)

यह टूल एक छवि को स्कैन कर सकता है और उसमें **पीकैप्स को निकाल सकता है**, **नेटवर्क जानकारी\(URLs, डोमेन, IPs, MACs, मेल्स\)** और अधिक **फ़ाइलें**। आपको केवल यह करना है:
```text
bulk_extractor memory.img -o out_folder
```
Navigate through **सभी जानकारी** जो उपकरण ने इकट्ठा की है \(पासवर्ड?\), **डाटा पैकेट** का **विश्लेषण** करें \(पढ़ें [**Pcaps विश्लेषण**](../pcap-inspection/)\), **अजीब डोमेन** की खोज करें \(मैलवेयर से संबंधित या **अस्तित्व में न होने वाले** डोमेन\).

## PhotoRec

आप इसे [https://www.cgsecurity.org/wiki/TestDisk\_Download](https://www.cgsecurity.org/wiki/TestDisk_Download) में पा सकते हैं।

यह GUI और CLI संस्करण के साथ आता है। आप फाइल-प्रकार चुन सकते हैं जिनकी खोज के लिए PhotoRec को।

![](../../../.gitbook/assets/image%20%28524%29.png)

# विशेष डेटा कार्विंग उपकरण

## FindAES

अपने की स्थिति खोजने के लिए AES कुंजियों की खोज करता है उनकी कुंजी अनुसूचियों की खोज करके। 128, 192, और 256 बिट कुंजियों को खोजने में सक्षम है, जैसे कि TrueCrypt और BitLocker द्वारा उपयोग की गई कुंजियाँ।

यहाँ से डाउनलोड करें [here](https://sourceforge.net/projects/findaes/).

# पूरक उपकरण

आप [**viu** ](https://github.com/atanunq/viu)का उपयोग कर सकते हैं छवियों को देखने के लिए टर्मिनल से। आप लिनक्स कमांड लाइन उपकरण **pdftotext** का उपयोग कर सकते हैं एक पीडीएफ को पाठ में बदलने और पढ़ने के लिए।
