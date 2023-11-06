# BloodHound और अन्य AD Enum टूल

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें, [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके।**

</details>

## AD Explorer

[AD Explorer](https://docs.microsoft.com/en-us/sysinternals/downloads/adexplorer) Sysinternal Suite से है:

> एक उन्नत Active Directory (AD) दर्शक और संपादक। आप AD Explorer का उपयोग करके आसानी से AD डेटाबेस में नेविगेट कर सकते हैं, पसंदीदा स्थानों को परिभाषित कर सकते हैं, डायलॉग बॉक्स खोले बिना ऑब्जेक्ट गुणों और विशेषताओं को देख सकते हैं, अनुमतियों को संपादित कर सकते हैं, ऑब्जेक्ट की स्कीमा देख सकते हैं, और समर्थित खोजों को निष्पादित और सहेज सकते हैं।

### स्नैपशॉट

AD Explorer स्नैपशॉट बना सकता है ताकि आप इसे ऑफ़लाइन जांच सकें।\
इसका उपयोग ऑफ़लाइन वल्न्स खोजने के लिए किया जा सकता है, या एडी डीबी के विभिन्न स्थितियों की तुलना करने के लिए किया जा सकता है।

आपको उपयोगकर्ता नाम, पासवर्ड, और कनेक्शन के लिए दिशा की आवश्यकता होगी (कोई भी AD उपयोगकर्ता आवश्यक है)।

AD का स्नैपशॉट लेने के लिए, `File` --> `Create Snapshot` पर जाएं और स्नैपशॉट के लिए एक नाम दर्ज करें।

## ADRecon

****[**ADRecon**](https://github.com/adrecon/ADRecon) एक टूल है जो AD पर्यावरण से विभिन्न आर्टिफैक्ट निकालता है और उन्हें एकत्रित करता है। जानकारी को एक **विशेष रूप में स्वरूपित** Microsoft Excel **रिपोर्ट** में प्रस्तुत किया जा सकता है जिसमें विश्लेषण को सुविधाजनक बनाने और लक्षित AD पर्यावरण की वर्तमान स्थिति का संपूर्ण चित्र प्रदान करने के लिए सारांश दृश्य शामिल होते हैं।
```bash
# Run it
.\ADRecon.ps1
```
## BloodHound

> BloodHound एक एकल पेज जावास्क्रिप्ट वेब एप्लिकेशन है, जो [Linkurious](http://linkurio.us) पर बनाई गई है, [Electron](http://electron.atom.io) के साथ कंपाइल की गई है, और एक PowerShell ingestor द्वारा भरी हुई [Neo4j](https://neo4j.com) डेटाबेस का उपयोग करती है।
>
> BloodHound ग्राफ सिद्धांत का उपयोग करके एक्टिव डिरेक्टरी माहौल में छिपे हुए और अक्सर अनजाने रिश्तों को प्रकट करने के लिए उपयोग करता है। हमलावार अपने लाभ के लिए BloodHound का उपयोग करके आसानी से उच्च जटिल हमले के मार्गों की पहचान कर सकते हैं जो अन्यथा त्वरित रूप से पहचानना असंभव होगा। संरक्षक BloodHound का उपयोग करके उनी ही हमले के मार्गों की पहचान और उन्हें नष्ट कर सकते हैं। नीला और लाल टीम दोनों BloodHound का उपयोग करके एक्टिव डिरेक्टरी माहौल में विशेषाधिकार संबंधों की गहरी समझ प्राप्त करने के लिए आसानी से कर सकती हैं।
>
> BloodHound का विकास [@\_wald0](https://www.twitter.com/\_wald0), [@CptJesus](https://twitter.com/CptJesus), और [@harmj0y](https://twitter.com/harmj0y) द्वारा किया गया है।
>
> स्रोत: [https://github.com/BloodHoundAD/BloodHound](https://github.com/BloodHoundAD/BloodHound)

इसलिए, [Bloodhound](https://github.com/BloodHoundAD/BloodHound) एक शानदार टूल है जो स्वचालित रूप से एक डोमेन की गणना कर सकता है, सभी जानकारी को सहेज सकता है, संभावित विशेषाधिकार उन्नयन मार्गों को खोज सकता है और ग्राफ का उपयोग करके सभी जानकारी को दिखा सकता है।

Bloodhound 2 मुख्य भागों से मिलकर बना है: **ingestors** और **visualisation application**.

**ingestors** का उपयोग करके डोमेन की गणना की जाती है और सभी जानकारी को निकाला जाता है जिसे visualisation application समझेगा।

**visualisation application neo4j का उपयोग** करके दिखाता है कि सभी जानकारी कैसे संबंधित है और डोमेन में विशेषाधिकार को उन्नयन करने के विभिन्न तरीके दिखाता है।

### स्थापना

1. Bloodhound

विजुअलाइज़ेशन एप्लिकेशन स्थापित करने के लिए आपको **neo4j** और **bloodhound एप्लिकेशन** स्थापित करने की आवश्यकता होगी।\
इसे करने का सबसे आसान तरीका है:
```
apt-get install bloodhound
```
आप **यहां से neo4j की सामुदायिक संस्करण डाउनलोड** कर सकते हैं। [यहां](https://neo4j.com/download-center/#community)।

1. इंजेस्टर्स

आप इंजेस्टर्स को यहां से डाउनलोड कर सकते हैं:

* https://github.com/BloodHoundAD/SharpHound/releases
* https://github.com/BloodHoundAD/BloodHound/releases
* https://github.com/fox-it/BloodHound.py

1. ग्राफ से पथ सीखें

Bloodhound के पास संवेदनशील कमजोरी पथ को हाइलाइट करने के लिए विभिन्न क्वेरी होती हैं। यह संभव है कि आप खोज और वस्तुओं के बीच संबंध को बढ़ाने के लिए कस्टम क्वेरी जोड़ सकते हैं और अधिक!

इस रेपो में अच्छे संग्रह के क्वेरी हैं: https://github.com/CompassSecurity/BloodHoundQueries

स्थापना प्रक्रिया:
```
$ curl -o "~/.config/bloodhound/customqueries.json" "https://raw.githubusercontent.com/CompassSecurity/BloodHoundQueries/master/BloodHound_Custom_Queries/customqueries.json"
```
### विज़ुअलाइज़ेशन ऐप का निष्पादन

आवश्यक एप्लिकेशनों को डाउनलोड/इंस्टॉल करने के बाद, हम उन्हें शुरू करेंगे।
सबसे पहले, आपको **नियो4ज डेटाबेस को शुरू करना होगा**:
```bash
./bin/neo4j start
#or
service neo4j start
```
पहली बार जब आप इस डेटाबेस को शुरू करेंगे, तो आपको [http://localhost:7474/browser/](http://localhost:7474/browser/) तक पहुंच की आवश्यकता होगी। आपसे डिफ़ॉल्ट क्रेडेंशियल्स (neo4j:neo4j) की मांग की जाएगी और आपको **पासवर्ड बदलने की आवश्यकता होगी**, इसलिए उसे बदलें और भूल न जाएं।

अब, **ब्लडहाउंड एप्लिकेशन** को शुरू करें:
```bash
./BloodHound-linux-x64
#or
bloodhound
```
आपको डेटाबेस क्रेडेंशियल्स के लिए प्रोम्प्ट किया जाएगा: **neo4j:\<आपका नया पासवर्ड>**

और ब्लडहाउंड डेटा को इंजेस्ट करने के लिए तैयार हो जाएगा।

![](<../../.gitbook/assets/image (171) (1).png>)

### SharpHound

उनके पास कई विकल्प हैं, लेकिन यदि आप डोमेन से जुड़े हुए एक पीसी से SharpHound चलाना चाहते हैं, अपने मौजूदा उपयोगकर्ता का उपयोग करके सभी जानकारी निकालने के लिए आप यह कर सकते हैं:
```
./SharpHound.exe --CollectionMethods All
Invoke-BloodHound -CollectionMethod All
```
> आप **CollectionMethod** और लूप सत्र के बारे में और अधिक पढ़ सकते हैं [यहां](https://bloodhound.readthedocs.io/en/latest/data-collection/sharphound-all-flags.html)

यदि आप विभिन्न क्रेडेंशियल का उपयोग करके SharpHound को निष्पादित करना चाहते हैं, तो आप एक CMD netonly सत्र बना सकते हैं और वहां से SharpHound को चला सकते हैं:
```
runas /netonly /user:domain\user "powershell.exe -exec bypass"
```
[**ब्लडहाउंड के बारे में और अधिक जानें ired.team पर।**](https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/abusing-active-directory-with-bloodhound-on-kali-linux)

**Windows Silent**

### **पायथन ब्लडहाउंड**

यदि आपके पास डोमेन क्रेडेंशियल हैं, तो आप किसी भी प्लेटफ़ॉर्म से **पायथन ब्लडहाउंड इंजेस्टर चला सकते हैं**, इसलिए आपको Windows पर निर्भरता की आवश्यकता नहीं होती है।\
इसे [https://github.com/fox-it/BloodHound.py](https://github.com/fox-it/BloodHound.py) से डाउनलोड करें या `pip3 install bloodhound` करके इंस्टॉल करें।
```bash
bloodhound-python -u support -p '#00^BlackKnight' -ns 10.10.10.192 -d blackfield.local -c all
```
यदि आप इसे प्रॉक्सीचेन्स के माध्यम से चला रहे हैं तो DNS संक्रियान के लिए प्रॉक्सी के माध्यम से काम करने के लिए `--dns-tcp` जोड़ें।
```bash
proxychains bloodhound-python -u support -p '#00^BlackKnight' -ns 10.10.10.192 -d blackfield.local -c all --dns-tcp
```
### Python SilentHound

यह स्क्रिप्ट **LDAP के माध्यम से Active Directory डोमेन की एनुमरेशन को शांत रूप से करेगा**, जिसमें उपयोगकर्ता, व्यवस्थापक, समूह आदि को पार्स किया जाएगा।

इसे [**SilentHound github**](https://github.com/layer8secure/SilentHound) में देखें।

### RustHound

Rust में BloodHound, [**यहां देखें**](https://github.com/OPENCYBER-FR/RustHound)।

## Group3r

[**Group3r**](https://github.com/Group3r/Group3r) **** एक टूल है जो Active Directory से जुड़े **ग्रुप नीति** में **कमजोरियां** खोजने के लिए है। \
आपको **किसी भी डोमेन उपयोगकर्ता** का उपयोग करके डोमेन के भीतर से **group3r चलाना** होगा।
```bash
group3r.exe -f <filepath-name.log>
# -s sends results to stdin
# -f send results to file
```
## PingCastle

****[**PingCastle**](https://www.pingcastle.com/documentation/) **एडी वातावरण की सुरक्षा स्थिति का मूल्यांकन करता है** और ग्राफों के साथ एक अच्छी **रिपोर्ट** प्रदान करता है।

इसे चलाने के लिए, `PingCastle.exe` नामक बाइनरी को निष्पादित करें और यह एक **सक्रिय सत्र** शुरू करेगा जो विकल्पों के मेनू प्रस्तुत करेगा। उपयोग करने के लिए डिफ़ॉल्ट विकल्प है **`healthcheck`** जो डोमेन का एक मूल **अवलोकन** स्थापित करेगा, और **गलत कॉन्फ़िगरेशन** और **कमजोरियां** खोजेगा।&#x20;

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की अनुमति चाहिए? [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह।
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें और PRs के माध्यम से [hacktricks repo](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud repo](https://github.com/carlospolop/hacktricks-cloud) में सबमिट करें।**

</details>
