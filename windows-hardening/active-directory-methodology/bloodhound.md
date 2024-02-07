# BloodHound और अन्य AD Enum टूल

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं**? या क्या आप **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करना चाहते हैं**? [**सब्सक्रिप्शन प्लान**](https://github.com/sponsors/carlospolop) की जाँच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें।
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) और **मुझे** **Twitter** **🐦**[**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud)** को PRs सबमिट करके।

</details>

## AD Explorer

[AD Explorer](https://docs.microsoft.com/en-us/sysinternals/downloads/adexplorer) Sysinternal Suite से है:

> एक उन्नत Active Directory (AD) दर्शक और संपादक। आप AD Explorer का उपयोग करके एक AD डेटाबेस में आसानी से नेविगेट कर सकते हैं, पसंदीदा स्थानों को परिभाषित कर सकते हैं, ऑब्ज
```bash
# Run it
.\ADRecon.ps1
```
## BloodHound

> BloodHound एक मोनोलिथिक वेब एप्लिकेशन है जिसमें एक एम्बेडेड React फ्रंटएंड और [Sigma.js](https://www.sigmajs.org/) के साथ एक [Go](https://go.dev/) आधारित REST API बैकएंड है। इसे एक [Postgresql](https://www.postgresql.org/) एप्लिकेशन डेटाबेस और एक [Neo4j](https://neo4j.com) ग्राफ डेटाबेस के साथ डिप्लॉय किया गया है, और इसे [SharpHound](https://github.com/BloodHoundAD/SharpHound) और [AzureHound](https://github.com/BloodHoundAD/AzureHound) डेटा कलेक्टर्स द्वारा भोजित किया जाता है।
>
>BloodHound ग्राफ सिद्धांत का उपयोग करता है ताकि एक्टिव डायरेक्टरी या एज़्यूर माहौल में छिपी हुई और अक्सर अनजान रिश्तों को प्रकट कर सके। हमलावर BloodHound का उपयोग करके आसानी से उन उच्च जटिल हमले के मार्गों की पहचान कर सकते हैं जो अन्यथा तेजी से पहचानना असंभव होगा। रक्षक BloodHound का उपयोग करके उनी हमले के मार्गों की पहचान और समाप्ति कर सकते हैं। नीला और लाल दल दोनों BloodHound का उपयोग करके एक्टिव डायरेक्टरी या एज़्यूर माहौल में विशेषाधिकार संबंधों की गहरी समझ प्राप्त करने के लिए आसानी से कर सकते हैं।
>
>BloodHound CE को [BloodHound Enterprise Team](https://bloodhoundenterprise.io) ने बनाया और बनाए रखा है। मूल BloodHound को [@\_wald0](https://www.twitter.com/\_wald0), [@CptJesus](https://twitter.com/CptJesus), और [@harmj0y](https://twitter.com/harmj0y) ने बनाया था।
>
>स्रोत: [https://github.com/SpecterOps/BloodHound](https://github.com/SpecterOps/BloodHound)

इसलिए, [Bloodhound](https://github.com/SpecterOps/BloodHound) एक अद्वितीय उपकरण है जो एक डोमेन को स्वचालित रूप से गणना कर सकता है, सभी जानकारी को सहेज सकता है, संभावित विशेषाधिकार उन्नयन मार्गों को खोज सकता है और ग्राफ का उपयोग करके सभी जानकारी को दिखा सकता है।

Bloodhound 2 मुख्य भागों से मिलकर बना है: **इनजेस्टर्स** और **विजुअलाइजेशन एप्लिकेशन**।

**इनजेस्टर्स** का उपयोग **डोमेन को गणना करने और सभी जानकारी निकालने** के लिए किया जाता है जिसे विजुअलाइजेशन एप्लिकेशन समझेगा।

**विजुअलाइजेशन एप्लिकेशन neo4j का उपयोग करता है** ताकि दिखाए कैसे सभी जानकारी संबंधित है और डोमेन में विशेषाधिकारों को बढ़ाने के विभिन्न तरीके दिखाए।

### स्थापना
BloodHound CE के निर्माण के बाद, पूरे परियोजना को Docker के साथ उपयोग सुविधा के लिए अपडेट किया गया था। शुरू करने का सबसे आसान तरीका इसका पूर्व-कॉन्फ़िगर किया गया Docker Compose विन्यास का उपयोग करना है।

1. Docker Compose स्थापित करें। यह [Docker Desktop](https://www.docker.com/products/docker-desktop/) स्थापना के साथ शामिल होना चाहिए।
2. चलाएँ:
```
curl -L https://ghst.ly/getbhce | docker compose -f - up
```
3. डॉकर कंपोज़ के टर्मिनल आउटपुट में यादृच्छिक रूप से उत्पन्न पासवर्ड का पता लगाएं।
4. ब्राउज़र में, http://localhost:8080/ui/login पर जाएं। एडमिन उपयोगकर्ता नाम और लॉग्स से यादृच्छिक रूप से उत्पन्न पासवर्ड के साथ लॉगिन करें।

इसके बाद आपको यादृच्छिक रूप से उत्पन्न पासवर्ड बदलने की आवश्यकता होगी और आपके पास नया इंटरफेस तैयार हो जाएगा, जिससे आप सीधे इंजेस्टर्स को डाउनलोड कर सकते हैं।

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

## पुराना ब्लडहाउंड
### स्थापना

1. ब्लडहाउंड

दृश्यीकरण एप्लिकेशन स्थापित करने के लिए आपको **नेओ4जे** और **ब्लडहाउंड एप्लिकेशन** स्थापित करने की आवश्यकता है।\
इसे करने का सबसे आसान तरीका यह है:
```
apt-get install bloodhound
```
आप **नियो4ज की समुदाय संस्करण** को [यहाँ से](https://neo4j.com/download-center/#community) डाउनलोड कर सकते हैं।

1. इंजेस्टर्स

आप इंजेस्टर्स को निम्नलिखित स्थान से डाउनलोड कर सकते हैं:

* https://github.com/BloodHoundAD/SharpHound/releases
* https://github.com/BloodHoundAD/BloodHound/releases
* https://github.com/fox-it/BloodHound.py

1. ग्राफ से पथ सीखें

Bloodhound विभिन्न क्वेरी के साथ आता है जो संवेदनशील कम्प्रोमाइस पथ को हाइलाइट करने के लिए है। यह संभावना है कि आप खोज और वस्तुओं के बीच संबंध को बढ़ाने के लिए कस्टम क्वेरी जोड़ सकते हैं!

इस रेपो में क्यूरीज का एक अच्छा संग्रह है: https://github.com/CompassSecurity/BloodHoundQueries

स्थापना प्रक्रिया:
```
$ curl -o "~/.config/bloodhound/customqueries.json" "https://raw.githubusercontent.com/CompassSecurity/BloodHoundQueries/master/BloodHound_Custom_Queries/customqueries.json"
```
### विज़ुअलाइज़ेशन एप्लिकेशन क्रियान्वयन

आवश्यक एप्लिकेशनों को डाउनलोड/स्थापित करने के बाद, उन्हें शुरू करें।\
सबसे पहले, आपको **नियो4ज डेटाबेस शुरू करना होगा**:
```bash
./bin/neo4j start
#or
service neo4j start
```
पहली बार जब आप इस डेटाबेस को शुरू करेंगे तो आपको [http://localhost:7474/browser/](http://localhost:7474/browser/) तक पहुंचने की आवश्यकता होगी। आपसे डिफ़ॉल्ट क्रेडेंशियल्स (neo4j:neo4j) की मांग की जाएगी और आपको **पासवर्ड बदलना अनिवार्य होगा**, इसलिए उसे बदल दें और उसे भूल न जाएं।

अब, **bloodhound एप्लिकेशन** शुरू करें:
```bash
./BloodHound-linux-x64
#or
bloodhound
```
आपसे डेटाबेस क्रेडेंशियल के लिए पूछा जाएगा: **neo4j:\<आपका नया पासवर्ड>**

और bloodhound डेटा इनजेस्ट करने के लिए तैयार हो जाएगा।

![](<../../.gitbook/assets/image (171) (1).png>)


### **Python bloodhound**

अगर आपके पास डोमेन क्रेडेंशियल हैं तो आप किसी भी प्लेटफॉर्म से **पायथन ब्लडहाउंड इंजेस्टर चला सकते हैं** ताकि आपको Windows पर निर्भर नहीं होना पड़े।\
इसे [https://github.com/fox-it/BloodHound.py](https://github.com/fox-it/BloodHound.py) से डाउनलोड करें या `pip3 install bloodhound` करें।
```bash
bloodhound-python -u support -p '#00^BlackKnight' -ns 10.10.10.192 -d blackfield.local -c all
```
यदि आप इसे प्रॉक्सीचेन्स के माध्यम से चला रहे हैं तो DNS संक्षेप काम करने के लिए `--dns-tcp` जोड़ें।
```bash
proxychains bloodhound-python -u support -p '#00^BlackKnight' -ns 10.10.10.192 -d blackfield.local -c all --dns-tcp
```
### Python SilentHound

यह स्क्रिप्ट **LDAP के माध्यम से Active Directory डोमेन को शांति से जांचेगा** और उपयोगकर्ताओं, व्यवस्थापकों, समूह आदि को पार्स करेगा।

इसे देखें [**SilentHound github**](https://github.com/layer8secure/SilentHound).

### RustHound

Rust में BloodHound, इसे [**यहाँ देखें**](https://github.com/OPENCYBER-FR/RustHound).

## Group3r

[**Group3r**](https://github.com/Group3r/Group3r) **** एक उपकरण है जो Active Directory से संबंधित **समूह नीति** में **कमजोरियों** को खोजने के लिए है। \
आपको **किसी भी डोमेन उपयोगकर्ता** का उपयोग करके डोमेन के भीतर से **ग्रुप3r चलाना** होगा।
```bash
group3r.exe -f <filepath-name.log>
# -s sends results to stdin
# -f send results to file
```
## PingCastle

****[**PingCastle**](https://www.pingcastle.com/documentation/) **एडी वातावरण की सुरक्षा स्थिति का मूल्यांकन करता है** और एक अच्छी **रिपोर्ट** प्रदान करता है जिसमें ग्राफ शामिल हैं।

इसे चलाने के लिए, `PingCastle.exe` नामक बाइनरी को चला सकते हैं और यह एक **इंटरैक्टिव सत्र** शुरू करेगा जिसमें विकल्पों का एक मेनू प्रस्तुत किया जाएगा। उपयोग करने के लिए डिफ़ॉल्ट विकल्प **`healthcheck`** है जो **डोमेन** का एक मूल **अवलोकन** स्थापित करेगा, और **misconfigurations** और **वंलरेबिलिटीज़** को खोजेगा।&#x20;
