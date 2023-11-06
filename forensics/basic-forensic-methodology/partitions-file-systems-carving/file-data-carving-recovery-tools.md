<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने की अनुमति चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud)** को PR जमा करके।

</details>

<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

वे संगठनों को खोजें जो सबसे महत्वपूर्ण हैं ताकि आप उन्हें जल्दी ठीक कर सकें। Intruder आपकी हमले की सतह का ट्रैक करता है, प्रोएक्टिव धमकी स्कैन चलाता है, आपकी पूरी टेक स्टैक, एपीआई से वेब ऐप्स और क्लाउड सिस्टम तक, में समस्याएं खोजता है। [**इसे नि: शुल्क परीक्षण के लिए आज़माएं**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks)।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

***

# Carving & Recovery tools

अधिक उपकरण [https://github.com/Claudio-C/awesome-datarecovery](https://github.com/Claudio-C/awesome-datarecovery) में मिलेंगे

## Autopsy

छवियों से फ़ाइलें निकालने के लिए जो सबसे आम उपकरण है, वह है [**Autopsy**](https://www.autopsy.com/download/)। इसे डाउनलोड करें, इंस्टॉल करें और इसे फ़ाइल को "छिपी हुई" फ़ाइलें खोजने के लिए इंजेस्ट करें। ध्यान दें कि Autopsy को डिस्क छवियों और अन्य प्रकार की छवियों का समर्थन करने के लिए बनाया गया है, लेकिन साधारित फ़ाइलें नहीं।

## Binwalk <a href="#binwalk" id="binwalk"></a>

**Binwalk** एक उपकरण है जो छवियों और ऑडियो फ़ाइलों जैसे बाइनरी फ़ाइलों में एम्बेडेड फ़ाइलें और डेटा की खोज के लिए उपयोग होता है।\
इसे `apt` के साथ स्थापित किया जा सकता है हालांकि [स्रोत](https://github.com/ReFirmLabs/binwalk) github पर मिल सकता है।\
**उपयोगी कमांडें**:
```bash
sudo apt install binwalk #Insllation
binwalk file #Displays the embedded data in the given file
binwalk -e file #Displays and extracts some files from the given file
binwalk --dd ".*" file #Displays and extracts all files from the given file
```
## Foremost

छिपे हुए फ़ाइलों को खोजने के लिए एक और सामान्य उपकरण है **foremost**। आप foremost के कॉन्फ़िगरेशन फ़ाइल को `/etc/foremost.conf` में ढूंढ सकते हैं। यदि आप केवल कुछ विशिष्ट फ़ाइलों की खोज करना चाहते हैं, तो उन्हें uncomment करें। यदि आप कुछ uncomment नहीं करते हैं, तो foremost अपने डिफ़ॉल्ट कॉन्फ़िगर किए गए फ़ाइल प्रकारों की खोज करेगा।
```bash
sudo apt-get install foremost
foremost -v -i file.img -o output
#Discovered files will appear inside the folder "output"
```
## **Scalpel**

**Scalpel** एक औजार है जिसका उपयोग किया जा सकता है **एक फ़ाइल में समाहित फ़ाइलें खोजने और निकालने** के लिए। इस मामले में, आपको कॉन्फ़िगरेशन फ़ाइल (_/etc/scalpel/scalpel.conf_) से उन फ़ाइल प्रकारों को uncomment करना होगा जिन्हें आप निकालना चाहते हैं।
```bash
sudo apt-get install scalpel
scalpel file.img -o output
```
## Bulk Extractor

यह टूल काली में शामिल है, लेकिन आप इसे यहाँ भी पा सकते हैं: [https://github.com/simsong/bulk\_extractor](https://github.com/simsong/bulk\_extractor)

यह टूल एक इमेज स्कैन कर सकता है और उसमें से **पीकैप्स (pcaps)**, **नेटवर्क सूचना (URLs, डोमेन, IPs, MACs, मेल)** और अधिक **फ़ाइलें** निकाल सकता है। आपको केवल निम्न कार्रवाई करनी होगी:
```
bulk_extractor memory.img -o out_folder
```
**सभी जानकारी** के माध्यम से नेविगेट करें जो टूल ने इकट्ठा की है (पासवर्ड?), **पैकेट्स** का **विश्लेषण** करें ([**Pcaps विश्लेषण**](../pcap-inspection/) पढ़ें), **अजीब डोमेन** (मैलवेयर से संबंधित या **अस्तित्व नहीं** रखने वाले डोमेन) की खोज करें।

## PhotoRec

आप इसे [https://www.cgsecurity.org/wiki/TestDisk\_Download](https://www.cgsecurity.org/wiki/TestDisk\_Download) में पा सकते हैं।

इसके पास GUI और CLI संस्करण शामिल हैं। आप फ़ाइल प्रकारों का चयन कर सकते हैं जिन्हें आप PhotoRec की खोज करने के लिए चाहते हैं।

![](<../../../.gitbook/assets/image (524).png>)

## binvis

कोड की जांच करें ([यहां](https://code.google.com/archive/p/binvis/)) और वेब पेज टूल की जांच करें ([यहां](https://binvis.io/#/))।

### BinVis की विशेषताएं

* दृश्यात्मक और सक्रिय **संरचना दर्शक**
* विभिन्न फोकस बिंदुओं के लिए एकाधिक प्लॉट
* नमूने के भागों पर ध्यान केंद्रित करना
* PE या ELF एक्जीक्यूटेबल्स में **स्ट्रिंग और संसाधन** देखना, जैसे कि
* फ़ाइलों पर क्रिप्टोएनालिसिस के लिए **पैटर्न** प्राप्त करना
* पैकर या एनकोडर एल्गोरिदम की **पहचान** करना
* पैटर्न के माध्यम से स्टेगानोग्राफी की **पहचान** करना
* **दृश्यात्मक** बाइनरी-डिफिंग

बिनविस एक महान **स्टार्ट-पॉइंट है जिससे आप एक अज्ञात लक्ष्य के साथ परिचित हो सकते हैं** एक ब्लैक-बॉक्सिंग scenario में।

# विशेष डेटा कार्विंग टूल्स

## FindAES

TrueCrypt और BitLocker द्वारा उपयोग की जाने वाली 128, 192 और 256 बिट कुंजी जैसे AES कुंजीयों की खोज करने के लिए उनके कुंजी अनुसूचियों की खोज करके AES कुंजीयों की खोज करता है।

यहां [डाउनलोड करें](https://sourceforge.net/projects/findaes/)।

# पूरक टूल्स

आप चित्रों को टर्मिनल से देखने के लिए [**viu**](https://github.com/atanunq/viu) का उपयोग कर सकते हैं।\
आप लिनक्स कमांड लाइन टूल **pdftotext** का उपयोग करके एक पीडीएफ को पाठ में बदलने और इसे पढ़ने के लिए उपयोग कर सकते हैं।


<figure><img src="/.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

वे संदेश जो अधिक मायने रखते हैं उनकी खोज करें ताकि आप उन्हें जल्दी ठीक कर सकें। Intruder आपकी हमला सतह का ट्रैक करता है, प्रोएक्टिव ध्रुवीकरण स्कैन चलाता है, आपकी पूरी टेक स्टैक, एपीआई से वेब ऐप्स और क्लाउड सिस्टम तक, मुद्दों की खोज करता है। [**इसे आजमाएं मुफ्त में**](https://www.intruder.io/?utm\_source=referral\&utm\_campaign=hacktricks)।

{% embed url="https://www.intruder.io/?utm_campaign=hacktricks&utm_source=referral" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की अनुमति चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके**।

</details>
