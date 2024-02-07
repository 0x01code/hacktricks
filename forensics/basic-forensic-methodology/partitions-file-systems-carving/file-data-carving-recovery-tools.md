<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, HackTricks** को PRs जमा करके [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में।

</details>

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

वह सुरक्षितता खोजें जो सबसे महत्वपूर्ण है ताकि आप उन्हें तेजी से ठीक कर सकें। Intruder आपकी हमले की सतह का ट्रैक करता है, प्रोएक्टिव धारणा स्कैन चलाता है, आपकी पूरी तकनीकी स्टैक, API से वेब ऐप्स और क्लाउड सिस्टम तक मुद्दे खोजता है। [**आज ही मुफ्त में इसका प्रयास करें**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks)।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

# कार्विंग और रिकवरी टूल

अधिक टूल [https://github.com/Claudio-C/awesome-datarecovery](https://github.com/Claudio-C/awesome-datarecovery)

## Autopsy

छवियों से फ़ाइलें निकालने के लिए ज्यादा प्रयोग किया जाने वाला टूल है [**Autopsy**](https://www.autopsy.com/download/)। इसे डाउनलोड करें, इंस्टॉल करें और फ़ाइल को इंजेस्ट करने के लिए इसे बनाएं "छिपी" फ़ाइलें खोजने के लिए। ध्यान दें कि Autopsy को डिस्क छवियों और अन्य प्रकार की छवियों का समर्थन करने के लिए बनाया गया है, लेकिन साधारण फ़ाइलों का नहीं।

## Binwalk <a href="#binwalk" id="binwalk"></a>

**Binwalk** एक टूल है जो बाइनरी फ़ाइलों का विश्लेषण करने के लिए है जिसमें एम्बेडेड सामग्री खोजने के लिए है। इसे `apt` के माध्यम से स्थापित किया जा सकता है और इसका स्रोत [GitHub](https://github.com/ReFirmLabs/binwalk) पर है।

**उपयोगी कमांड**:
```bash
sudo apt install binwalk #Insllation
binwalk file #Displays the embedded data in the given file
binwalk -e file #Displays and extracts some files from the given file
binwalk --dd ".*" file #Displays and extracts all files from the given file
```
## Foremost

छिपे हुए फ़ाइलें खोजने के लिए एक और सामान्य उपकरण है **foremost**। आप foremost की विन्यास फ़ाइल को `/etc/foremost.conf` में पा सकते हैं। यदि आप कुछ विशिष्ट फ़ाइलों के लिए खोजना चाहते हैं तो उन्हें uncomment करें। यदि आप कुछ uncomment नहीं करते हैं तो foremost अपने डिफ़ॉल्ट विन्यस्त फ़ाइल प्रकारों के लिए खोजेगा।
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
**उस सारी जानकारी** में नेविगेट करें जो टूल ने इकट्ठा की है (पासवर्ड?), **पैकेट** का **विश्लेषण** करें (पढ़ें[ **Pcaps विश्लेषण**](../pcap-inspection/)), **अजीब डोमेन** (मैलवेयर से संबंधित या **अस्तित्वहीन**) की खोज करें।

## PhotoRec

आप इसे [https://www.cgsecurity.org/wiki/TestDisk\_Download](https://www.cgsecurity.org/wiki/TestDisk\_Download) में पा सकते हैं।

यह GUI और CLI संस्करणों के साथ आता है। आप फाइल-प्रकार चुन सकते हैं जिनकी खोज के लिए PhotoRec को।

![](<../../../.gitbook/assets/image (524).png>)

## binvis

कोड की [जाँच](https://code.google.com/archive/p/binvis/) और [वेब पेज टूल](https://binvis.io/#/) करें।

### BinVis की विशेषताएँ

* दृश्यात्मक और सक्रिय **संरचना दर्शक**
* विभिन्न ध्यान स्थानों के लिए एकाधिक चित्र
* एक नमूने के भागों पर ध्यान केंद्रित करना
* **PE या ELF एक्जीक्यूटेबल्स** में स्ट्रिंग और संसाधन देखना, जैसे कि
* फ़ाइलों पर क्रिप्टोएनालिसिस के लिए **पैटर्न** प्राप्त करना
* पैकर या इन्कोडर एल्गोरिदम की **पहचान** करना
* पैटर्न द्वारा स्टेगेनोग्राफी की **पहचान** करना
* **दृश्यात्मक** बाइनरी-डिफिंग

BinVis एक अज्ञात लक्ष्य के साथ परिचित होने के लिए एक महान **आरंभ-बिंदु** है एक काल-बॉक्सिंग स्थिति में।

# विशेष डेटा कार्विंग टूल

## FindAES

अपने की स्केज्यूल्स की खोज करके AES कुंजियों की खोज करता है। 128, 192, और 256 बिट कुंजियों को खोजने में सक्षम है, जैसे कि जो TrueCrypt और BitLocker द्वारा उपयोग की जाती हैं।

यहाँ से डाउनलोड करें (https://sourceforge.net/projects/findaes/).

# पूरक टूल

आप [**viu** ](https://github.com/atanunq/viu)का उपयोग करके छवियों को देखने के लिए कर सकते हैं।\
आप लिनक्स कमांड लाइन टूल **pdftotext** का उपयोग करके एक पीडीएफ को पाठ में परिवर्तित करने और इसे पढ़ने के लिए कर सकते हैं।


<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

जो सबसे अधिक महत्वपूर्ण संकटों को खोजें ताकि आप उन्हें जल्दी ठीक कर सकें। Intruder आपकी हमले की सतह का ट्रैक करता है, प्रोएक्टिव धमकी स्कैन चलाता है, आपकी पूरे तकनीकी स्टैक, एपीआई से वेब ऐप्स और क्लाउड सिस्टम्स तक के मुद्दे खोजता है। [**इसे मुफ्त में प्रयास करें**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks) आज।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप अपनी कंपनी का विज्ञापन **HackTricks** में देखना चाहते हैं या **HackTricks** को **PDF** में डाउनलोड करना चाहते हैं तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **जुड़ें** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और हमें **ट्विटर** 🐦 [**@hacktricks_live**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **अपने हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github रेपो में पीआर जमा करके।

</details>
