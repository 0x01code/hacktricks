# BloodHound और अन्य AD Enum टूल्स

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबरसिक्योरिटी कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे**? या क्या आप PEASS के **नवीनतम संस्करण तक पहुँचना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* **[**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) में शामिल हों या [**telegram समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, [hacktricks repo](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud repo](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके।**

</details>

## AD Explorer

[AD Explorer](https://docs.microsoft.com/en-us/sysinternals/downloads/adexplorer) Sysinternal Suite से है:

> एक उन्नत Active Directory (AD) व्यूअर और एडिटर। आप AD Explorer का उपयोग AD डेटाबेस को आसानी से नेविगेट करने, पसंदीदा स्थानों को परिभाषित करने, डायलॉग बॉक्स खोले बिना ऑब्जेक्ट प्रॉपर्टीज और एट्रिब्यूट्स देखने, परमिशन्स एडिट करने, ऑब्जेक्ट की स्कीमा देखने और सोफिस्टिकेटेड सर्चेस को एक्जीक्यूट करने के लिए कर सकते हैं जिन्हें आप सेव कर सकते हैं और फिर से एक्जीक्यूट कर सकते हैं।

### स्नैपशॉट्स

AD Explorer AD के स्नैपशॉट्स बना सकता है ताकि आप इसे ऑफलाइन चेक कर सकें।\
इसका उपयोग ऑफलाइन वल्न्स की खोज करने या समय के साथ AD DB की विभिन्न स्थितियों की तुलना करने के लिए किया जा सकता है।

आपको AD से कनेक्ट करने के लिए यूजरनेम, पासवर्ड, और दिशा की आवश्यकता होगी।

AD का स्नैपशॉट लेने के लिए, `File` --> `Create Snapshot` पर जाएं और स्नैपशॉट के लिए एक नाम दर्ज करें।

## ADRecon

****[**ADRecon**](https://github.com/adrecon/ADRecon) एक ऐसा टूल है जो एक AD वातावरण से विभिन्न आर्टिफैक्ट्स निकालता है और संयोजित करता है। इस जानकारी को एक **विशेष रूप से फॉर्मेटेड** Microsoft Excel **रिपोर्ट** में प्रस्तुत किया जा सकता है जिसमें सारांश दृश्य और मेट्रिक्स शामिल होते हैं जो विश्लेषण को सुविधाजनक बनाते हैं और लक्षित AD वातावरण की वर्तमान स्थिति की एक समग्र तस्वीर प्रदान करते हैं।
```bash
# Run it
.\ADRecon.ps1
```
## BloodHound

> BloodHound एक एकल वेब एप्लिकेशन है जिसमें एक एम्बेडेड React फ्रंटएंड [Sigma.js](https://www.sigmajs.org/) के साथ और एक [Go](https://go.dev/) आधारित REST API बैकएंड है। इसे [Postgresql](https://www.postgresql.org/) एप्लिकेशन डेटाबेस और [Neo4j](https://neo4j.com) ग्राफ डेटाबेस के साथ तैनात किया जाता है, और इसे [SharpHound](https://github.com/BloodHoundAD/SharpHound) और [AzureHound](https://github.com/BloodHoundAD/AzureHound) डेटा कलेक्टर्स द्वारा फीड किया जाता है।
>
>BloodHound ग्राफ सिद्धांत का उपयोग करके एक Active Directory या Azure वातावरण के भीतर छिपे हुए और अक्सर अनजाने में बने संबंधों को प्रकट करता है। हमलावर BloodHound का उपयोग करके आसानी से उन जटिल हमले के मार्गों की पहचान कर सकते हैं जिन्हें अन्यथा जल्दी से पहचानना असंभव होता। डिफेंडर्स BloodHound का उपयोग करके उन्हीं हमले के मार्गों की पहचान और उन्हें समाप्त कर सकते हैं। नीली और लाल टीमें दोनों BloodHound का उपयोग करके एक Active Directory या Azure वातावरण में विशेषाधिकार संबंधों की गहरी समझ प्राप्त कर सकते हैं।
>
>BloodHound CE को [BloodHound Enterprise Team](https://bloodhoundenterprise.io) द्वारा बनाया गया है और रखरखाव किया जाता है। मूल BloodHound को [@\_wald0](https://www.twitter.com/\_wald0), [@CptJesus](https://twitter.com/CptJesus), और [@harmj0y](https://twitter.com/harmj0y) द्वारा बनाया गया था।
>
>[https://github.com/SpecterOps/BloodHound](https://github.com/SpecterOps/BloodHound) से

तो, [Bloodhound](https://github.com/SpecterOps/BloodHound) एक अद्भुत उपकरण है जो स्वचालित रूप से एक डोमेन को सूचीबद्ध कर सकता है, सभी जानकारी को सहेज सकता है, संभावित विशेषाधिकार वृद्धि मार्गों का पता लगा सकता है और सभी जानकारी को ग्राफ़ का उपयोग करके दिखा सकता है।

Booldhound में मुख्य रूप से दो भाग होते हैं: **ingestors** और **विज़ुअलाइज़ेशन एप्लिकेशन**।

**ingestors** का उपयोग **डोमेन को सूचीबद्ध करने और सभी जानकारी को निकालने** के लिए किया जाता है जिसे विज़ुअलाइज़ेशन एप्लिकेशन समझ सकता है।

**विज़ुअलाइज़ेशन एप्लिकेशन neo4j का उपयोग करता है** यह दिखाने के लिए कि सभी जानकारी कैसे संबंधित है और डोमेन में विशेषाधिकार वृद्धि के विभिन्न तरीके कैसे दिखाए जा सकते हैं।

### स्थापना
BloodHound CE के निर्माण के बाद, पूरी परियोजना को Docker के साथ उपयोग में आसानी के लिए अपडेट किया गया था। शुरू करने का सबसे आसान तरीका इसके पूर्व-निर्धारित Docker Compose कॉन्फ़िगरेशन का उपयोग करना है।

1. Docker Compose स्थापित करें। यह [Docker Desktop](https://www.docker.com/products/docker-desktop/) स्थापना के साथ शामिल होना चाहिए।
2. चलाएँ:
```
curl -L https://ghst.ly/getbhce | docker compose -f - up
```
3. टर्मिनल आउटपुट में Docker Compose के यादृच्छिक रूप से उत्पन्न पासवर्ड को खोजें।
4. एक ब्राउज़र में, http://localhost:8080/ui/login पर नेविगेट करें। admin नाम के साथ लॉगिन करें और लॉग्स से प्राप्त यादृच्छिक रूप से उत्पन्न पासवर्ड का उपयोग करें।

इसके बाद आपको यादृच्छिक रूप से उत्पन्न पासवर्ड बदलने की आवश्यकता होगी और आपके पास नया इंटरफ़ेस तैयार होगा, जिससे आप सीधे इन्जेस्टर्स डाउनलोड कर सकते हैं।

### SharpHound

उनके पास कई विकल्प हैं लेकिन यदि आप डोमेन से जुड़े पीसी से SharpHound चलाना चाहते हैं, अपने वर्तमान उपयोगकर्ता का उपयोग करके और सभी जानकारी निकालना चाहते हैं तो आप कर सकते हैं:
```
./SharpHound.exe --CollectionMethods All
Invoke-BloodHound -CollectionMethod All
```
> **CollectionMethod** और लूप सेशन के बारे में आप यहाँ और पढ़ सकते हैं [यहाँ](https://support.bloodhoundenterprise.io/hc/en-us/articles/17481375424795-All-SharpHound-Community-Edition-Flags-Explained)

यदि आप अलग प्रमाणीकरण (credentials) का उपयोग करके SharpHound चलाना चाहते हैं, तो आप CMD netonly सेशन बना सकते हैं और वहां से SharpHound चला सकते हैं:
```
runas /netonly /user:domain\user "powershell.exe -exec bypass"
```
[**ired.team पर Bloodhound के बारे में और जानें।**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-with-bloodhound-on-kali-linux)

## पुराना Bloodhound
### स्थापना

1. Bloodhound

विज़ुअलाइज़ेशन एप्लिकेशन स्थापित करने के लिए आपको **neo4j** और **bloodhound एप्लिकेशन** स्थापित करने की आवश्यकता होगी।\
इसे करने का सबसे आसान तरीका है:
```
apt-get install bloodhound
```
आप **neo4j का कम्युनिटी वर्जन डाउनलोड कर सकते हैं** [यहाँ से](https://neo4j.com/download-center/#community).

1. इन्जेस्टर्स

आप इन्जेस्टर्स यहाँ से डाउनलोड कर सकते हैं:

* https://github.com/BloodHoundAD/SharpHound/releases
* https://github.com/BloodHoundAD/BloodHound/releases
* https://github.com/fox-it/BloodHound.py

1. ग्राफ से पथ सीखें

Bloodhound में विभिन्न क्वेरीज़ होती हैं जो संवेदनशील समझौता पथ को उजागर करती हैं। इसमें कस्टम क्वेरीज़ जोड़ना संभव है ताकि खोज और ऑब्जेक्ट्स के बीच संबंधों को बेहतर बनाया जा सके!

इस रेपो में क्वेरीज़ का अच्छा संग्रह है: https://github.com/CompassSecurity/BloodHoundQueries

इंस्टालेशन प्रक्रिया:
```
$ curl -o "~/.config/bloodhound/customqueries.json" "https://raw.githubusercontent.com/CompassSecurity/BloodHoundQueries/master/BloodHound_Custom_Queries/customqueries.json"
```
### विज़ुअलाइज़ेशन ऐप निष्पादन

आवश्यक एप्लिकेशन्स को डाउनलोड/इंस्टॉल करने के बाद, आइए उन्हें शुरू करें।\
सबसे पहले आपको **neo4j डेटाबेस को शुरू करना होगा**:
```bash
./bin/neo4j start
#or
service neo4j start
```
पहली बार जब आप इस डेटाबेस को शुरू करेंगे, आपको [http://localhost:7474/browser/](http://localhost:7474/browser/) पर जाना होगा। आपसे डिफ़ॉल्ट क्रेडेंशियल्स (neo4j:neo4j) पूछे जाएंगे और आपसे **पासवर्ड बदलने की आवश्यकता होगी**, इसलिए इसे बदलें और इसे भूलें नहीं।

अब, **bloodhound एप्लिकेशन** शुरू करें:
```bash
./BloodHound-linux-x64
#or
bloodhound
```
आपसे डेटाबेस क्रेडेंशियल्स के लिए पूछा जाएगा: **neo4j:\<आपका नया पासवर्ड>**

और bloodhound डेटा इन्जेस्ट करने के लिए तैयार होगा।

![](<../../.gitbook/assets/image (171) (1).png>)

### **Python bloodhound**

यदि आपके पास डोमेन क्रेडेंशियल्स हैं तो आप किसी भी प्लेटफॉर्म से **python bloodhound ingestor चला सकते हैं** इसलिए आपको Windows पर निर्भर नहीं रहना पड़ेगा।\
इसे [https://github.com/fox-it/BloodHound.py](https://github.com/fox-it/BloodHound.py) से डाउनलोड करें या `pip3 install bloodhound` करके।
```bash
bloodhound-python -u support -p '#00^BlackKnight' -ns 10.10.10.192 -d blackfield.local -c all
```
यदि आप इसे proxychains के माध्यम से चला रहे हैं तो DNS समाधान को प्रॉक्सी के माध्यम से काम करने के लिए `--dns-tcp` जोड़ें।
```bash
proxychains bloodhound-python -u support -p '#00^BlackKnight' -ns 10.10.10.192 -d blackfield.local -c all --dns-tcp
```
### Python SilentHound

यह स्क्रिप्ट **Active Directory Domain को LDAP के माध्यम से शांतिपूर्वक सूचीबद्ध करेगी**, जिसमें उपयोगकर्ताओं, व्यवस्थापकों, समूहों आदि का विश्लेषण होता है।

इसे [**SilentHound github**](https://github.com/layer8secure/SilentHound) पर देखें।

### RustHound

BloodHound in Rust, [**यहाँ जांचें**](https://github.com/OPENCYBER-FR/RustHound)।

## Group3r

[**Group3r**](https://github.com/Group3r/Group3r) **** एक उपकरण है जो Active Directory से जुड़ी **Group Policy** में **भेद्यताओं** का पता लगाता है। \
आपको डोमेन के अंदर के किसी भी होस्ट से **group3r चलाना होगा** **किसी भी डोमेन उपयोगकर्ता** का उपयोग करते हुए।
```bash
group3r.exe -f <filepath-name.log>
# -s sends results to stdin
# -f send results to file
```
## PingCastle

[**PingCastle**](https://www.pingcastle.com/documentation/) **AD पर्यावरण की सुरक्षा स्थिति का मूल्यांकन करता है** और ग्राफ़ के साथ एक अच्छी **रिपोर्ट** प्रदान करता है।

इसे चलाने के लिए, आप `PingCastle.exe` बाइनरी को निष्पादित कर सकते हैं और यह विकल्पों के मेनू के साथ एक **इंटरैक्टिव सत्र** शुरू करेगा। उपयोग करने के लिए डिफ़ॉल्ट विकल्प **`healthcheck`** है जो **डोमेन** का एक बेसलाइन **अवलोकन** स्थापित करेगा, और **गलत कॉन्फ़िगरेशन** और **कमजोरियों** का पता लगाएगा।

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे**? या क्या आप PEASS के **नवीनतम संस्करण तक पहुँच प्राप्त करना चाहते हैं या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में या **Twitter** पर मुझे **फॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपनी हैकिंग ट्रिक्स साझा करें, [hacktricks repo](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud repo](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके**।

</details>
