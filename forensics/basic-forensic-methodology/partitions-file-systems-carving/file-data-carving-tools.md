<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह

- प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके।**

</details>


# कार्विंग उपकरण

## Autopsy

छवियों से फ़ाइलें निकालने के लिए जो सबसे आम उपकरण जो फोरेंसिक्स में उपयोग किया जाता है, वह है [**Autopsy**](https://www.autopsy.com/download/)। इसे डाउनलोड करें, इंस्टॉल करें और इसे फ़ाइल को "छिपी हुई" फ़ाइलें खोजने के लिए इंजेस्ट करें। ध्यान दें कि Autopsy को डिस्क इमेज और अन्य प्रकार की इमेज का समर्थन करने के लिए बनाया गया है, लेकिन साधारित फ़ाइलें नहीं।

## Binwalk <a id="binwalk"></a>

**Binwalk** एक उपकरण है जो एम्बेडेड फ़ाइलें और डेटा के लिए बाइनरी फ़ाइलों की खोज के लिए उपयोग किया जाता है, जैसे छवियां और ऑडियो फ़ाइलें। इसे `apt` के साथ स्थापित किया जा सकता है हालांकि [स्रोत](https://github.com/ReFirmLabs/binwalk) github पर मिल सकता है। **उपयोगी कमांड**:
```bash
sudo apt install binwalk #Insllation
binwalk file #Displays the embedded data in the given file
binwalk -e file #Displays and extracts some files from the given file
binwalk --dd ".*" file #Displays and extracts all files from the given file
```
## Foremost

छिपे हुए फ़ाइलों को खोजने के लिए एक और सामान्य उपकरण है **foremost**। आप foremost के कॉन्फ़िगरेशन फ़ाइल को `/etc/foremost.conf` में ढूंढ सकते हैं। यदि आप केवल कुछ विशिष्ट फ़ाइलों की खोज करना चाहते हैं, तो उन्हें uncomment करें। यदि आप कुछ uncomment नहीं करते हैं, तो foremost डिफ़ॉल्ट रूप से कॉन्फ़िगर किए गए फ़ाइल प्रकारों की खोज करेगा।
```bash
sudo apt-get install foremost
foremost -v -i file.img -o output
#Discovered files will appear inside the folder "output"
```
## **Scalpel**

**Scalpel** एक और टूल है जिसका उपयोग किया जा सकता है **एक फ़ाइल में एम्बेड किए गए फ़ाइलों को खोजने और निकालने** के लिए। इस मामले में, आपको कॉन्फ़िगरेशन फ़ाइल \(_/etc/scalpel/scalpel.conf_\) से उन फ़ाइल प्रकारों को uncomment करने की आवश्यकता होगी जिन्हें आप निकालना चाहते हैं।
```bash
sudo apt-get install scalpel
scalpel file.img -o output
```
## Bulk Extractor

यह टूल काली में शामिल है, लेकिन आप इसे यहाँ भी पा सकते हैं: [https://github.com/simsong/bulk\_extractor](https://github.com/simsong/bulk_extractor)

यह टूल एक इमेज स्कैन कर सकता है और उसमें से **पीकैप्स निकाल सकता है**, **नेटवर्क सूचना (URLs, डोमेन, IPs, MACs, मेल्स)** और अधिक **फ़ाइलें**। आपको केवल यह करना है:
```text
bulk_extractor memory.img -o out_folder
```
**सभी जानकारी** के माध्यम से नेविगेट करें जो टूल ने इकट्ठा की है \(पासवर्ड?\), **पैकेट्स** का **विश्लेषण** करें \(पढ़ें [**Pcaps विश्लेषण**](../pcap-inspection/)\), **अजीब डोमेन** की खोज करें \(मैलवेयर से संबंधित या **अस्तित्वहीन**\).

## PhotoRec

आप इसे [https://www.cgsecurity.org/wiki/TestDisk\_Download](https://www.cgsecurity.org/wiki/TestDisk_Download) में पा सकते हैं।

इसके साथ GUI और CLI संस्करण आता है। आप PhotoRec को खोज करने के लिए चाहिए फ़ाइल-प्रकार का चयन कर सकते हैं।

![](../../../.gitbook/assets/image%20%28524%29.png)

# विशेष डेटा कार्विंग टूल

## FindAES

इसके द्वारा AES कुंजीयों की खोज की जाती है जो उनकी कुंजी अनुसूचियों की खोज करके की जाती है। 128, 192 और 256 बिट कुंजीयों की खोज कर सकता है, जैसे कि TrueCrypt और BitLocker द्वारा उपयोग की जाती है।

यहाँ से डाउनलोड करें [here](https://sourceforge.net/projects/findaes/).

# पूरक टूल

आप [**viu** ](https://github.com/atanunq/viu) का उपयोग कर सकते हैं ताकि आप टर्मिनल में छवियों को देख सकें।
आप लिनक्स कमांड लाइन टूल **pdftotext** का उपयोग कर सकते हैं ताकि आप एक पीडीएफ को पाठ में बदल सकें और इसे पढ़ सकें।



<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी HackTricks में विज्ञापित** की जाए? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने का उपयोग करने की आवश्यकता है? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।

- प्राप्त करें [**official PEASS & HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**telegram समूह**](https://t.me/peass) या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का पालन करें।**

- **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके**।

</details>
