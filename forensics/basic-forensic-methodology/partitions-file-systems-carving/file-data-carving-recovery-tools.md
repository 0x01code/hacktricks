<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें** द्वारा **PRs सबमिट करके** [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

वह सुरक्षितता खोजें जो सबसे महत्वपूर्ण है ताकि आप उन्हें तेजी से ठीक कर सकें। Intruder आपकी हमले की सतह का ट्रैक करता है, प्रोएक्टिव धारणा स्कैन चलाता है, आपकी पूरी तकनीकी स्टैक, API से वेब ऐप्स और क्लाउड सिस्टम तक मुद्दे खोजता है। [**आज ही मुफ्त में इसका प्रयास करें**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks)।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

# कार्विंग और रिकवरी टूल

अधिक टूल [https://github.com/Claudio-C/awesome-datarecovery](https://github.com/Claudio-C/awesome-datarecovery)

## Autopsy

छवियों से फ़ाइलें निकालने के लिए ज्यादा प्रयोग किया जाने वाला टूल है [**Autopsy**](https://www.autopsy.com/download/)। इसे डाउनलोड करें, इंस्टॉल करें और फ़ाइल को इंजेस्ट करने के लिए इसे खोजें "छिपी" फ़ाइलें। ध्यान दें कि Autopsy डिस्क छवियों और अन्य प्रकार की छवियों का समर्थन करने के लिए बनाया गया है, लेकिन साधे फ़ाइलों का नहीं।

## Binwalk <a href="#binwalk" id="binwalk"></a>

**Binwalk** एक टूल है जो इमेजेस और ऑडियो फ़ाइल्स जैसे बाइनरी फ़ाइलों को खोजने और डेटा के लिए एम्बेडेड फ़ाइलों के लिए है।\
इसे `apt` के साथ इंस्टॉल किया जा सकता है हालांकि [स्रोत](https://github.com/ReFirmLabs/binwalk) github पर मिल सकता है।\
**उपयोगी कमांड्स**:
```bash
sudo apt install binwalk #Insllation
binwalk file #Displays the embedded data in the given file
binwalk -e file #Displays and extracts some files from the given file
binwalk --dd ".*" file #Displays and extracts all files from the given file
```
## Foremost

छिपी हुई फ़ाइलें खोजने के लिए एक और सामान्य उपकरण है **foremost**। आप foremost का विन्यास फ़ाइल `/etc/foremost.conf` में पा सकते हैं। यदि आप कुछ विशिष्ट फ़ाइलों की खोज करना चाहते हैं तो उन्हें uncomment करें। यदि आप कुछ uncomment नहीं करते हैं तो foremost अपने डिफ़ॉल्ट विन्यसित फ़ाइल प्रकारों के लिए खोज करेगा।
```bash
sudo apt-get install foremost
foremost -v -i file.img -o output
#Discovered files will appear inside the folder "output"
```
## **Scalpel**

**Scalpel** एक औजार है जिसका उपयोग **एक फ़ाइल में एम्बेडेड फ़ाइलें** खोजने और निकालने के लिए किया जा सकता है। इस मामले में, आपको वह फ़ाइल प्रकार अनकमेंट करने की आवश्यकता होगी जिसे आप निकालना चाहते हैं। (/etc/scalpel/scalpel.conf_)
```bash
sudo apt-get install scalpel
scalpel file.img -o output
```
## Bulk Extractor

यह टूल काली के अंदर आता है लेकिन आप इसे यहाँ पा सकते हैं: [https://github.com/simsong/bulk\_extractor](https://github.com/simsong/bulk\_extractor)

यह टूल एक छवि को स्कैन कर सकता है और उसमें **पीकैप्स को निकाल सकता है**, **नेटवर्क सूचना (URLs, डोमेन, IPs, MACs, मेल्स)** और अधिक **फ़ाइलें**। आपको केवल यह करना है:
```
bulk_extractor memory.img -o out_folder
```
**उस सभी जानकारी** में नेविगेट करें जो टूल ने इकट्ठा की है (पासवर्ड?), **पैकेट्स** का **विश्लेषण** करें (पढ़ें[ **Pcaps विश्लेषण**](../pcap-inspection/)), **अजीब डोमेन** (मैलवेयर से संबंधित या **अस्तित्वहीन**) की खोज करें।

## PhotoRec

आप इसे [https://www.cgsecurity.org/wiki/TestDisk\_Download](https://www.cgsecurity.org/wiki/TestDisk\_Download) में पा सकते हैं।

यह GUI और CLI संस्करणों के साथ आता है। आप चुन सकते हैं **फ़ाइल-प्रकार** जिनकी खोज के लिए PhotoRec को।

![](<../../../.gitbook/assets/image (524).png>)

## binvis

कोड की जाँच करें [यहाँ](https://code.google.com/archive/p/binvis/) और [वेब पेज टूल](https://binvis.io/#/)।

### BinVis की विशेषताएँ

* दृश्यात्मक और सक्रिय **संरचना दर्शक**
* विभिन्न फोकस बिंदुओं के लिए एकाधिक चित्र
* एक नमूने के भागों पर ध्यान केंद्रित करना
* **स्ट्रिंग्स और संसाधनों** को देखना, PE या ELF कार्यक्रमों में जैसे
* फ़ाइलों पर क्रिप्टानालिसिस के लिए **पैटर्न** प्राप्त करना
* पैकर या इन्कोडर एल्गोरिदम को **स्पॉट** करना
* पैटर्न द्वारा स्टेगेनोग्राफी की **पहचान** करना
* **दृश्यात्मक** बाइनरी-डिफिंग

BinVis एक महान **शुरुआती बिंदु है** जिससे आप एक अज्ञात लक्ष्य के साथ परिचित हो सकते हैं एक ब्लैक-बॉक्सिंग स्थिति में।

# विशेष डेटा कार्विंग टूल

## FindAES

AES कुंजियों की खोज करने के लिए उनके कुंजी अनुसूचियों की खोज करके FindAES खोजता है। 128, 192, और 256 बिट कुंजियों को खोजने में सक्षम है, जैसे कि TrueCrypt और BitLocker द्वारा उपयोग की गई कुंजियाँ।

यहाँ से डाउनलोड करें (https://sourceforge.net/projects/findaes/).

# पूरक टूल

आप [**viu** ](https://github.com/atanunq/viu)का उपयोग कर सकते हैं ताकि आप छवियों को टर्मिनल से देख सकें।\
आप लिनक्स कमांड लाइन टूल **pdftotext** का उपयोग कर सकते हैं ताकि आप एक पीडीएफ को पाठ में परिवर्तित कर सकें और इसे पढ़ सकें।


<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

उन वंशावलियताओं को खोजें जो सबसे अधिक मायने रखती हैं ताकि आप उन्हें तेजी से ठीक कर सकें। Intruder आपकी हमले की सतह का ट्रैक करता है, प्रोएक्टिव धारणा स्कैन चलाता है, आपकी पूरे तकनीकी स्टैक, एपीआई से वेब ऐप्स और क्लाउड सिस्टम तक के मुद्दे खोजता है। [**इसे मुफ्त में प्रयास करें**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) आज।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें** हैकट्रिक्स और हैकट्रिक्स क्लाउड गिटहब रेपो में पीआर जमा करके।

</details>
