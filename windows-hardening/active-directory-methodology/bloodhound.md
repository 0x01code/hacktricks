# BloodHound और अन्य AD Enum टूल

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और **मुझे** **Twitter** 🐦[**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud)** को PR जमा करके।

</details>

## AD Explorer

[AD Explorer](https://docs.microsoft.com/en-us/sysinternals/downloads/adexplorer) Sysinternal Suite से है:

> एक उन्नत Active Directory (AD) दर्शक और संपादक। आप AD Explorer का उपयोग करके एक AD डेटाबेस में आसानी से नेविगेट कर सकते हैं, पसंदीदा स्थानों को परिभाषित कर सकते हैं, ऑब्ज
```bash
# Run it
.\ADRecon.ps1
```
## BloodHound

From [https://github.com/BloodHoundAD/BloodHound](https://github.com/BloodHoundAD/BloodHound)

> BloodHound एक एकल पृष्ठ जावास्क्रिप्ट वेब एप्लिकेशन है, जो [Linkurious](http://linkurio.us/) पर निर्मित है, [Electron](http://electron.atom.io/) के ऊपर संकलित है, जिसमें एक [Neo4j](https://neo4j.com/) डेटाबेस C# डेटा कलेक्टर द्वारा पोषित है।

BloodHound ग्राफ सिद्धांत का उपयोग करता है ताकि एक्टिव डायरेक्टरी या एज़्यूर माहौल में छुपी और अक्सर अनजान रिश्तों को प्रकट कर सके। हमलावर BloodHound का उपयोग करके आसानी से उन उच्च जटिल हमले के मार्गों की पहचान कर सकते हैं जो अन्यथा तेजी से पहचानना असंभव होगा। रक्षक BloodHound का उपयोग करके उनी हमले के मार्गों की पहचान और समाप्ति कर सकते हैं। नीला और लाल दल दोनों BloodHound का उपयोग करके एक्टिव डायरेक्टरी या एज़्यूर माहौल में विशेषाधिकार संबंधों की गहरी समझ प्राप्त कर सकते हैं।

इसलिए, [Bloodhound](https://github.com/BloodHoundAD/BloodHound) एक अद्भुत उपकरण है जो एक डोमेन को स्वचालित रूप से गणना कर सकता है, सभी जानकारी को सहेज सकता है, संभावित विशेषाधिकार उन्नति मार्गों को खोज सकता है और ग्राफ का उपयोग करके सभी जानकारी को दिखा सकता है।

Bloodhound 2 मुख्य भागों से मिलकर बना है: **इनजेस्टर्स** और **विजुअलाइजेशन एप्लिकेशन**।

**इनजेस्टर्स** का उपयोग किया जाता है **डोमेन को गणना करने और सभी जानकारी को निकालने** के लिए जिस प्रारूप में विजुअलाइजेशन एप्लिकेशन समझेगा।

**विजुअलाइजेशन एप्लिकेशन neo4j का उपयोग करता है** ताकि दिखाए कैसे सभी जानकारी संबंधित है और डोमेन में विशेषाधिकारों को उन्नत करने के विभिन्न तरीके दिखाए।

### स्थापना
BloodHound CE के निर्माण के बाद, पूरे परियोजना को उपयोग सुविधा के लिए Docker के साथ अपडेट किया गया था। शुरू करने का सबसे आसान तरीका इसका पूर्व-कॉन्फ़िगर किया गया Docker Compose विन्यास का उपयोग करना है।

1. Docker Compose स्थापित करें। यह [Docker Desktop](https://www.docker.com/products/docker-desktop/) स्थापना के साथ शामिल होना चाहिए।
2. चलाएँ:
```
curl -L https://ghst.ly/getbhce | docker compose -f - up
```
3. डॉकर कॉम्पोज के टर्मिनल आउटपुट में यादृच्छिक रूप से उत्पन्न पासवर्ड का पता लगाएं।
4. एक ब्राउज़र में, http://localhost:8080/ui/login पर नेविगेट करें। लॉगिन करें एडमिन उपयोगकर्ता नाम और लॉग से यादृच्छिक रूप से उत्पन्न पासवर्ड के साथ।

इसके बाद आपको यादृच्छिक रूप से उत्पन्न पासवर्ड बदलने की आवश्यकता होगी और आपके पास नया इंटरफेस तैयार हो जाएगा, जिससे आप सीधे इंजेस्टर्स डाउनलोड कर सकते हैं।

### SharpHound

उनके पास कई विकल्प हैं, लेकिन यदि आप डोमेन से जुड़े एक पीसी से SharpHound चलाना चाहते हैं, अपने वर्तमान उपयोगकर्ता का उपयोग करके सभी जानकारी निकालने के लिए आप कर सकते हैं:
```
./SharpHound.exe --CollectionMethods All
Invoke-BloodHound -CollectionMethod All
```
> आप **CollectionMethod** और लूप सत्र के बारे में अधिक पढ़ सकते हैं [यहाँ](https://support.bloodhoundenterprise.io/hc/en-us/articles/17481375424795-All-SharpHound-Community-Edition-Flags-Explained)

यदि आप विभिन्न क्रेडेंशियल का उपयोग करके SharpHound को निष्पादित करना चाहते हैं तो आप एक CMD netonly सत्र बना सकते हैं और वहां से SharpHound चला सकते हैं:
```
runas /netonly /user:domain\user "powershell.exe -exec bypass"
```
[**आईआरईडी टीम में ब्लडहाउंड के बारे में अधिक जानें।**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-with-bloodhound-on-kali-linux)


## Group3r

[**Group3r**](https://github.com/Group3r/Group3r) एक टूल है जो Active Directory से जुड़े **ग्रुप पॉलिसी** में **कमजोरियों** को खोजने के लिए है। \
आपको **किसी भी डोमेन उपयोगकर्ता** का उपयोग करके डोमेन के भीतर स्थित होस्ट से **ग्रुप3r चलाना** होगा।
```bash
group3r.exe -f <filepath-name.log>
# -s sends results to stdin
# -f send results to file
```
## PingCastle

[**PingCastle**](https://www.pingcastle.com/documentation/) **एडी वातावरण की सुरक्षा स्थिति का मूल्यांकन करता है** और एक अच्छी **रिपोर्ट** प्रदान करता है जिसमें ग्राफ शामिल हैं।

इसे चलाने के लिए, `PingCastle.exe` नामक बाइनरी को चला सकते हैं और यह एक **इंटरैक्टिव सत्र** शुरू करेगा जिसमें विकल्पों का एक मेनू प्रस्तुत करेगा। उपयोग करने के लिए डिफ़ॉल्ट विकल्प **`healthcheck`** है जो **डोमेन** का एक मूल **अवलोकन** स्थापित करेगा, और **misconfigurations** और **वंलरेबिलिटीज़** को खोजेगा।&#x20;
