# Mimikatz

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* प्राप्त करें [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com)
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में शामिल हों या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)** का** अनुसरण करें।**
* **अपने हैकिंग ट्रिक्स साझा करें और** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके** अपना योगदान दें।

</details>

इस पृष्ठ की सामग्री [adsecurity.org](https://adsecurity.org/?page\_id=1821) से कॉपी की गई थी

## LM और Clear-Text मेमोरी में

Windows 8.1 और Windows Server 2012 R2 के साथ शुरू होकर, LM हैश और "क्लियर-टेक्स्ट" पासवर्ड अब मेमोरी में नहीं होते हैं।

"क्लियर-टेक्स्ट" पासवर्ड को LSASS में रखने से रोकने के लिए, निम्नलिखित रजिस्ट्री कुंजी को "0" पर सेट करना चाहिए (Digest Disabled):

_HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest "UseLogonCredential" (DWORD)_

## **Mimikatz & LSA Protection:**

Windows Server 2012 R2 और Windows 8.1 में LSA Protection नामक एक नई सुविधा शामिल है जिसमें [LSASS को Windows Server 2012 R2 पर संरक्षित प्रक्रिया के रूप में सक्षम किया जाता है](https://technet.microsoft.com/en-us/library/dn408187.aspx) (Mimikatz एक ड्राइवर के साथ इसे बाइपास कर सकता है, लेकिन इससे इवेंट लॉग में कुछ शोर हो सकता है):

_एलएसए, जिसमें स्थानीय सुरक्षा प्राधिकरण सर्वर सेवा (LSASS) प्रक्रिया शामिल है, स्थानीय और दूरस्थ लॉगिन के लिए उपयोगकर्ताओं को मान्यता प्रदान करता है और स्थानीय सुरक्षा नीतियों को प्रवर्तित करता है। Windows 8.1 ऑपरेटिंग सिस्टम एलएसए के लिए अतिरिक्त सुरक्षा प्रदान करता है ताकि संरक्षित नहीं प्रक्रियाओं द्वारा मेमोरी और कोड इंजेक्शन पढ़ने और लिखने को रोका जा सके। इससे एलएसए द्वारा संग्रहित और प्रबंधित की जाने वाली प्रमाणिकाओं के लिए अतिरिक्त सुरक्षा प्रदान की जाती है।_

LSA संरक्षण सक्षम करना:

1. रजिस्ट्री संपादक (RegEdit.exe) खोलें और रजिस्ट्री कुंजी तक नेविगेट करें जो निम्नलिखित स्थित है: HKEY\_LOCAL\_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa और रजिस्ट्री कुंजी के मान को सेट करें: "RunAsPPL"=dword:00000001।
2. एक नया GPO बनाएं और कंप्यूटर कॉन्फ़िगरेशन, प्राथमिकताएँ, Windows सेटिंग्स पर ब्राउज़ करें। राइट-क्लिक करें रजिस्ट्री, नया के लिए प्वाइंट करें, और फिर रजिस्ट्री आइटम पर क्लिक करें। नई रजिस्ट्री संपत्ति डायलॉग बॉक्स दिखाई देती है। हाइव सूची में, HKEY\_LOCAL\_MACHINE पर क्लिक करें। कुंजी पथ सूची में, SYSTEM\CurrentControlSet\Control\Lsa तक ब्राउज़ करें। मान नाम बॉक्स में, RunAsPPL टाइप करें। मान प्रकार बॉक्स में, REG\_DWORD पर क्लिक करें। मान डेटा बॉक्स में, 00000001 टाइप करें। OK पर क्लिक करें।

LSA Protection गैर-संरक्षित प्रक्रियाओं को LSASS के साथ संवाद करने से रोकता है। Mimikatz अभी भी इसे एक ड्राइवर ("!+") के साथ बाइपास कर सकता है।

[![Mimikatz-Driver-Remove-LSASS-Protection](https://adsecurity.org/wp-content/uploads/2015/09/Mimikatz-Driver-Remove-LSASS-Protection.jpg)](https://adsecurity.org/wp-content/uploads/2015/09/Mimikatz-Driver-Remove-LSASS-Protection.jpg)

### Disabled SeDebugPrivilege को बाइपास करना
डिफ़ॉल्ट रूप से, SeDebugPrivilege को स्थानीय सुरक्षा नीति के माध्यम से व्यवस
```
sc config TrustedInstaller binPath= "C:\Users\Public\procdump64.exe -accepteula -ma lsass.exe C:\Users\Public\lsass.dmp"
sc start TrustedInstaller
```
[![TrustedInstaller-Dump-Lsass](https://1860093151-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-M6yZUYP7DLMbZuztKpV%2Fuploads%2FJtprjloNPADNSpb6S0DS%2Fimage.png?alt=media&token=9b639459-bd4c-4897-90af-8990125fa058)

यह डंप फ़ाइल एक हमलावर्द नियंत्रित कंप्यूटर पर भेजी जा सकती है जहां से क्रेडेंशियल्स निकाले जा सकते हैं।
```
# privilege::debug
# sekurlsa::minidump lsass.dmp
# sekurlsa::logonpasswords
```
## मुख्य

### **घटना**

**EVENT::Clear** - एक घटना लॉग को साफ करें\
[\
![Mimikatz-Event-Clear](https://adsecurity.org/wp-content/uploads/2015/09/Mimikatz-Event-Clear.png)](https://adsecurity.org/wp-content/uploads/2015/09/Mimikatz-Event-Clear.png)

**EVENT:::Drop** - (_**प्रायोगिक**_) नई घटनाओं से बचने के लिए इवेंट सेवा को पैच करें

[![Mimikatz-Event-Drop](https://adsecurity.org/wp-content/uploads/2015/09/Mimikatz-Event-Drop.png)](https://adsecurity.org/wp-content/uploads/2015/09/Mimikatz-Event-Drop.png)

नोट:\
privilege::debug चलाएं और फिर event::drop को चलाएं ताकि इवेंट लॉग को पैच किया जा सके। फिर Event::Clear चलाएं ताकि कोई भी लॉग साफ होने की घटना (1102) लॉग न हो।

### KERBEROS

#### गोल्डन टिकट

गोल्डन टिकट एक TGT है जिसमें KRBTGT NTLM पासवर्ड हैश का उपयोग करके एन्क्रिप्ट और साइन किया जाता है।

गोल्डन टिकट (GT) का निर्माण किसी भी उपयोगकर्ता (वास्तविक या कल्पित) के रूप में किया जा सकता है और डोमेन में किसी भी समूह के सदस्य के रूप में किया जा सकता है (जो लगभग असीमित संख्या में अधिकार प्रदान करता है) और डोमेन में हर एवं हर संसाधन के लिए।

**Mimikatz गोल्डन टिकट कमांड संदर्भ:**

गोल्डन टिकट बनाने के लिए Mimikatz कमांड है "kerberos::golden"

* /domain - पूर्ण नामवाला डोमेन। इस उदाहरण में: "lab.adsecurity.org"।
* /sid - डोमेन का SID। इस उदाहरण में: "S-1-5-21-1473643419-774954089-2222329127"।
* /sids - अधिक अधिकार वाले खातों / समूहों के लिए AD वन के अनुसार अतिरिक्त SID। सामान्यतः, इसमें रूट डोमेन के लिए Enterprise Admins समूह होगा "S-1-5-21-1473643419-774954089-5872329127-519"। [यह पैरामीटर प्रदान किए गए SID को SID History पैरामीटर में जोड़ता है।](https://adsecurity.org/?p=1640)
* /user - अनुकरण करने के लिए उपयोगकर्ता नाम
* /groups (वैकल्पिक) - उपयोगकर्ता के सदस्य होने वाले समूह RID (पहला मुख्य समूह है)।\
एक ही पहुंच प्राप्त करने के लिए उपयोगकर्ता या कंप्यूटर खाता RID जोड़ें।\
डिफ़ॉल्ट समूह: 513,512,520,518,519 विख्यात प्रशासक समूहों के लिए (नीचे सूचीबद्ध)
* /krbtgt - डोमेन KDC सेवा खाता (KRBTGT) के लिए NTLM पासवर्ड हैश। TGT को एन्क्रिप्ट और साइन करने के लिए उपयोग किया जाता है।
* /ticket (वैकल्पिक) - बाद में उपयोग के लिए गोल्डन टिकट फ़ाइल को सहेजने के लिए पथ और नाम प्रदान करें या /ptt का उपयोग करें ताकि तत्काल गोल्डन टिकट को मेमोरी में इंजेक्ट किया जा सके।
* /ptt - /ticket के विकल्प के रूप में - इसका उपयोग करें ताकि जाली टिकट को तत्काल उपयोग के लिए मेमोरी में इंजेक्ट किया जा सके।
* /id (वैकल्पिक) - उपयोगकर्ता RID। Mimikatz की डिफ़ॉल्ट मान्यांकन 500 है (डिफ़ॉल्ट प्रशासक खाता RID)।
* /startoffset (वैकल्पिक) - जब टिकट उपलब्ध होता है तो प्रारंभ ऑफ़सेट (सामान्यतः -10 या 0 सेट किया जाता है यदि यह विकल्प उपयोग किया जाता है)। Mimikatz की डिफ़ॉल्ट मान्यांकन 0 है।
* /endin (वैकल्पिक) - टिकट की आयु। Mimikatz की डिफ़ॉल्ट मान्यांकन 10 वर्ष (\~5,262,480 मिनट) है। Active Directory की डिफ़ॉल्ट केरबेरोस नीति सेटिंग 10 घंटे (600 मिनट) है।
* /renewmax (वैकल्पिक) - नवीनीकरण के साथ अधिकतम टिकट आयु। Mimikatz की डिफ़ॉल्ट मान्यांकन 10 वर्ष (\~5,262,480 मिनट) है। Active Directory की डिफ़ॉल्ट केरबेरोस नीति सेटिंग 7 दिन (10,080 मिनट) है।
* /sids (वैकल्पिक) - AD वन में Enterprise Admins समूह के SID (\[ADRootDomainSID]-519) को सेट करें ताकि AD वन में उच्चाधिकारिक अधिकारों को जाली बनाएं (AD वन में हर डोमेन में AD व्यवस्थापक)।
* /aes128 - AES128 कुंजी
* /aes256 - AES256 कुंजी

गोल्डन टिकट डिफ़ॉल्ट समूह:

* डोमेन उपयोगकर्ता SID: S-1-5-21\<DOMAINID>-513
* डोमेन व्यवस्थापक SID: S-1-5-21\<DOMAINID>-512
* स्कीमा व्यवस्थापक SID: S-1-5-21\<DOMAINID>-518
* एंटरप्राइज व्यवस्थापक SID: S-1-5-21\<DOMAINID>-519 (यह केवल तब प्रभावी होता है जब जाली टिकट वन रूट डोमेन में बनाया जाता है, हालांकि AD वन व्यवस्थापक अधिकारों के लिए /sids पैरामीटर का उपयोग करके जोड़ें)
* समूह नीति निर्माता मालिक SID: S-1-5-21\<DOMAINID>-520
```
.\mimikatz "kerberos::golden /User:Administrator /domain:rd.lab.adsecurity.org /id:512 /sid:S-1-5-21-135380161-102191138-581311202 /krbtgt:13026055d01f235d67634e109da03321 /groups:512 /startoffset:0 /endin:600 /renewmax:10080 /ptt" exit
```
[डोमेन के बीच सोने के टिकट](https://adsecurity.org/?p=1640)

#### सिल्वर टिकट

एक सिल्वर टिकट एक TGS है (फ़ॉर्मेट में TGT के समान) जिसमें लक्षित सेवा खाते का उपयोग करके (SPN मैपिंग द्वारा पहचाना जाता है) NTLM पासवर्ड हैश का उपयोग करके इसे एन्क्रिप्ट और साइन किया जाता है।

**सिल्वर टिकट बनाने के लिए उदाहरण Mimikatz कमांड:**

निम्नलिखित Mimikatz कमांड एक सिल्वर टिकट बनाता है जो सर्वर adsmswin2k8r2.lab.adsecurity.org पर CIFS सेवा के लिए है। इस सिल्वर टिकट को सफलतापूर्वक बनाने के लिए, adsmswin2k8r2.lab.adsecurity.org के लिए AD कंप्यूटर खाता पासवर्ड हैश को खोजा जाना चाहिए, या तो AD डोमेन डंप से या ऊपर दिखाए गए लोकल सिस्टम पर Mimikatz चलाकर (_Mimikatz "privilege::debug" "sekurlsa::logonpasswords" exit_)। NTLM पासवर्ड हैश को /rc4 पैरामीटर के साथ उपयोग किया जाता है। सेवा SPN प्रकार को /service पैरामीटर में पहचाना जाना चाहिए। अंत में, लक्षित कंप्यूटर का पूर्ण-योग्य डोमेन नाम /target पैरामीटर में प्रदान किया जाना चाहिए। /sid पैरामीटर में डोमेन SID को न भूलें।
```
mimikatz “kerberos::golden /admin:LukeSkywalker /id:1106 /domain:lab.adsecurity.org /sid:S-1-5-21-1473643419-774954089-2222329127 /target:adsmswin2k8r2.lab.adsecurity.org /rc4:d7e2b80507ea074ad59f152a1ba20458 /service:cifs /ptt” exit
```
#### [**विश्वास टिकट**](https://adsecurity.org/?p=1588)

एक्टिव डायरेक्टरी ट्रस्ट पासवर्ड हैश निर्धारित होने के बाद, एक ट्रस्ट टिकट उत्पन्न किया जा सकता है। ट्रस्ट टिकट उन दोमेन के बीच साझा पासवर्ड का उपयोग करके बनाए जाते हैं जो एक-दूसरे पर विश्वास करते हैं।\
[ट्रस्ट टिकट पर अधिक पृष्ठभूमि।](https://adsecurity.org/?p=1588)

**ट्रस्ट पासवर्ड (ट्रस्ट कुंजी) डंप करना**
```
Mimikatz “privilege::debug” “lsadump::trust /patch” exit
```
**Mimikatz का उपयोग करके एक जाली विश्वास प्रमाणपत्र (इंटर-वीटीजी) बनाएं**

Mimikatz का उपयोग करके विश्वास प्रमाणपत्र को जाली बनाएं जिसमें कहा जाता है कि प्रमाणपत्र धारक AD वनस्पति में एक एंटरप्राइज व्यवस्थापक है (Mimikatz में विश्वास प्रमाणपत्रों के बीच में सिडीइतिहास, "सिडीएस" का उपयोग करके विश्वास प्रमाणपत्रों के बीच यात्रा करने के लिए, यह मेरा "योगदान" है)। इससे एक बाल डोमेन से मूल डोमेन के पूर्ण प्रशासनिक पहुंच संभव होती है। ध्यान दें कि यह खाता कहीं भी मौजूद नहीं होना चाहिए क्योंकि यह वास्तव में एक सोने का प्रमाणपत्र है जो विश्वास प्रमाणपत्र के बीच यात्रा करता है।
```
Mimikatz “Kerberos::golden /domain:child.lab.adsecurity.org /sid:S-1-5-21-3677078698-724690114-1972670770 /sids:S-1-5-21-1581655573-3923512380-696647894-519 /rc4:49ed1653275f78846ff06de1a02386fd /user:DarthVader /service:krbtgt /target:lab.adsecurity.org /ticket:c:\temp\tickets\EA-ADSECLABCHILD.kirbi” exit
```
विश्वास टिकट विशेष आवश्यक पैरामीटर:

* \*\*/\*\*target - लक्षित डोमेन का FQDN।
* \*\*/\*\*service - लक्षित डोमेन में चल रही केरबेरोस सेवा (krbtgt)।
* \*\*/\*\*rc4 - सेवा केरबेरोस सेवा खाता (krbtgt) के लिए NTLM हैश।
* \*\*/\*\*ticket - बाद में उपयोग के लिए जाली टिकट फ़ाइल को सहेजने के लिए एक पथ और नाम प्रदान करें या /ptt का उपयोग करके स्मृति में स्वर्ण टिकट को तत्काल इंजेक्शन करें।

#### **और KERBEROS**

**KERBEROS::List** - उपयोगकर्ता मेमोरी में सभी उपयोगकर्ता टिकट (TGT और TGS) की सूची बनाएँ। कोई विशेष अधिकार आवश्यक नहीं है क्योंकि यह केवल वर्तमान उपयोगकर्ता के टिकट प्रदर्शित करता है।\
"क्लिस्ट" की कार्यान्वयनता के समान।

**KERBEROS::PTC** - कैश पास करें (NT6)\
Mac OS, Linux, BSD, Unix, आदि जैसे \*निक्स सिस्टम Kerberos क्रेडेंशियल कैश करते हैं। इस कैश डेटा को Mimikatz का उपयोग करके कॉपी किया जा सकता है। Kerberos टिकट्स को ccache फ़ाइल में इंजेक्शन करने के लिए भी उपयोगी है।

Mimikatz के kerberos::ptc का एक अच्छा उदाहरण है [PyKEK के साथ MS14-068 का शोषण](https://adsecurity.org/?p=676)। PyKEK एक ccache फ़ाइल उत्पन्न करता है जिसे kerberos::ptc का उपयोग करके Mimikatz में इंजेक्शन किया जा सकता है।

[![Mimikatz-PTC-PyKEK-ccacheFile](https://adsecurity.org/wp-content/uploads/2015/09/Mimikatz-PTC-PyKEK-ccacheFile.jpg)](https://adsecurity.org/wp-content/uploads/2015/09/Mimikatz-PTC-PyKEK-ccacheFile.jpg)

**KERBEROS::PTT** - टिकट पास करें\
[Kerberos टिकट मिलने के बाद](https://adsecurity.org/?p=1667), इसे दूसरे सिस्टम में कॉपी किया जा सकता है और मौजूदा सत्र में पास किया जा सकता है, जिससे कोई संचार के बिना लॉगऑन की अनुकरण किया जा सकता है। कोई विशेष अधिकार आवश्यक नहीं है।\
SEKURLSA::PTH (पास-द-हैश) के समान।

* /filename - टिकट का फ़ाइलनाम (एकाधिक हो सकते हैं)
* /diretory - एक निर्देशिका पथ, जिसमें सभी .kirbi फ़ाइलें होंगी, इंजेक्शन किया जाएगा।

[![KerberosUnConstrainedDelegation-Mimikatz-PTT-LS-Ticket2](https://adsecurity.org/wp-content/uploads/2015/09/KerberosUnConstrainedDelegation-Mimikatz-PTT-LS-Ticket2.png)](https://adsecurity.org/wp-content/uploads/2015/09/KerberosUnConstrainedDelegation-Mimikatz-PTT-LS-Ticket2.png)

**KERBEROS::Purge** - सभी Kerberos टिकट हटाएं\
"क्लिस्ट पर्ज" की कार्यान्वयनता के समान। टिकट (PTC, PTT, आदि) को पास करने से पहले इस कमांड को चलाएं ताकि सही उपयोगकर्ता संदर्भ का उपयोग किया जा सके।

[![Mimikatz-Kerberos-Purge](https://adsecurity.org/wp-content/uploads/2015/09/Mimikatz-Kerberos-Purge.png)](https://adsecurity.org/wp-content/uploads/2015/09/Mimikatz-Kerberos-Purge.png)

**KERBEROS::TGT** - वर्तमान उपयोगकर्ता के लिए वर्तमान TGT प्राप्त करें।

[![Mimikatz-Kerberos-TGT](https://adsecurity.org/wp-content/uploads/2015/09/Mimikatz-Kerberos-TGT.png)](https://adsecurity.org/wp-content/uploads/2015/09/Mimikatz-Kerberos-TGT.png)

### LSADUMP

**LSADUMP**::**DCShadow** - वर्तमान मशीन को DC के रूप में सेट करें ताकि DC के भीतर नए ऑब्जेक्ट बनाने की क्षमता हो (स्थायी विधि)।\
इसके लिए पूर्ण AD व्यवस्थापक अधिकार या KRBTGT पासवर्ड हैश की आवश्यकता होती है।\
DCShadow सामयिक रूप से कंप्यूटर को प्रतिरूपण के उद्देश्य के लिए "DC" के रूप में सेट करता है:

* AD वनस्पति के वनस्पति में 2 ऑब्जेक्ट बनाएं।
* उपयोग किए जाने वाले कंप्यूटर के SPN में "GC" (ग्लोबल कैटलॉग) और "E3514235-4B06-11D1-AB04-00C04FC2DCD2" (AD रिप्लिकेशन) शामिल करें। [ADSecurity SPN अनुभाग](https://adsecurity.org/?page\_id=183) में Kerberos सेवा प्रमुख नामों पर अधिक जानकारी।
* DrsReplicaAdd और KCC के माध्यम से अद्यतनों को DCs को पुश करें।
* वनस्पति में बनाए गए ऑब्जेक्ट हटाएं।

**LSADUMP::DCSync** - एक DC से एक ऑब्जेक्ट को समकालिक करने के लिए DC से सिंक्रनाइज़ करें (खाता के
```
mimikatz lsadump::lsa /inject exit
```
**LSADUMP::NetSync**

NetSync एक सरल तरीका प्रदान करता है जिसके माध्यम से एक डीसी कंप्यूटर खाता पासवर्ड डेटा का उपयोग करके एक डोमेन कंट्रोलर की भूमिका में छवि बनाने के लिए सिल्वर टिकट का उपयोग करके लक्षित खाता की जानकारी, सहित पासवर्ड डेटा, को डीसीसिंक करता है।

**LSADUMP::SAM** - SAM प्रविष्टियों (रजिस्ट्री या हाइव से) को डिक्रिप्ट करने के लिए SysKey प्राप्त करें। SAM विकल्प स्थानीय सुरक्षा खाता प्रबंधक (SAM) डेटाबेस से कनेक्ट करता है और स्थानीय खातों के लिए क्रेडेंशियल्स को डंप करता है।

**LSADUMP::Secrets** - SysKey प्राप्त करें ताकि SECRETS प्रविष्टियों (रजिस्ट्री या हाइव से) को डिक्रिप्ट किया जा सके।

**LSADUMP::SetNTLM** - एक उपयोगकर्ता के लिए एक नया पासवर्ड/NTLM सेट करने के लिए सर्वर से अनुरोध करें।

[**LSADUMP::Trust**](https://adsecurity.org/?p=1588) - LSA सर्वर से विश्वास प्रमाणन जानकारी (सामान्य या फ्लाई पर पैच) प्राप्त करने के लिए LSA सर्वर से पूछें।

### MISC

[**MISC::Skeleton**](https://adsecurity.org/?p=1275) - डोमेन कंट्रोलर पर LSASS प्रक्रिया में स्केलेटन कुंजी इंजेक्ट करें।
```
"privilege::debug" "misc::skeleton"
```
### विशेषाधिकार

**PRIVILEGE::Backup** - बैकअप विशेषाधिकार प्राप्त करें। डीबग अधिकारों की आवश्यकता होती है।

**PRIVILEGE::Debug** - डीबग अधिकार प्राप्त करें (यह या स्थानिक सिस्टम अधिकारों की आवश्यकता होती है बहुत सारे Mimikatz कमांड के लिए)।

### SEKURLSA

**SEKURLSA::Credman** - क्रेडेंशियल प्रबंधक की सूची देखें

**SEKURLSA::Ekeys** - **केरबेरोस एन्क्रिप्शन कुंजी** की सूची देखें

**SEKURLSA::Kerberos** - सभी प्रमाणित उपयोगकर्ताओं (सेवाओं और कंप्यूटर खाता सहित) के लिए केरबेरोस प्रमाणपत्रों की सूची देखें

**SEKURLSA::Krbtgt** - डोमेन केरबेरोस सेवा खाता (KRBTGT) पासवर्ड डेटा प्राप्त करें

**SEKURLSA::SSP** - SSP प्रमाणपत्रों की सूची देखें

**SEKURLSA::Wdigest** - WDigest प्रमाणपत्रों की सूची देखें

**SEKURLSA::LogonPasswords** - सभी उपलब्ध प्रदाता प्रमाणपत्रों की सूची देखें। यह आमतौर पर हाल ही में लॉग इन किए गए उपयोगकर्ता और कंप्यूटर प्रमाणपत्रों को दिखाता है।

* वर्तमान में लॉग इन किए गए (या हाल ही में लॉग इन किए गए) खातों के लिए LSASS में पासवर्ड डेटा डंप करता है, साथ ही उपयोगकर्ता प्रमाणों के संदर्भ में चल रही सेवाएं भी शामिल होती हैं।
* खाता पासवर्ड को प्रतिबिंबी तरीके से मेमोरी में संग्रहीत किया जाता है। यदि वे मेमोरी में हैं (Windows 8.1/Windows Server 2012 R2 से पहले थे), तो वे प्रदर्शित किए जाते हैं। Windows 8.1/Windows Server 2012 R2 आमतौर पर अधिकांश मामलों में खाता पासवर्ड को इस तरीके से संग्रहीत नहीं करता है। KB2871997 इस सुरक्षा क्षमता को Windows 7, Windows 8, Windows Server 2008R2 और Windows Server 2012 में लागू करता है, हालांकि KB2871997 लागू करने के बाद कंप्यूटर को अतिरिक्त कॉन्फ़िगरेशन की आवश्यकता होती है।
* व्यवस्थापक पहुंच (डीबग अधिकारों के साथ) या स्थानिक सिस्टम अधिकारों की आवश्यकता होती है

**SEKURLSA::Minidump** - LSASS मिनीडंप प्रक्रिया संदर्भ में स्विच करें (lsass डंप पढ़ें)

**SEKURLSA::Pth** - पास-द-हैश और ओवर-पास-द-हैश (अर्थात पासवर्ड की कुंजी पास करें)।

_Mimikatz प्रसिद्ध आपरेशन 'पास-द-हैश' को करने के लिए कार्य कर सकता है, जिसमें उपयोगकर्ता के पासवर्ड के NTLM हैश के साथ दूसरे प्रमाणों के तहत प्रक्रिया चलाई जाती है, इसके बजाय असली पासवर्ड। इसके लिए, यह एक नकली पहचान के साथ एक प्रक्रिया शुरू करता है, फिर नकली जानकारी (नकली पासवर्ड के NTLM हैश) को असली जानकारी (असली पासवर्ड के NTLM हैश) के साथ बदलता है।_

* /user - आप जिस उपयोगकर्ता की प्रतिष्ठा करना चाहते हैं, ध्यान दें कि विशेष रूप से इस प्रसिद्ध खाते के लिए व्यवस्थापक नाम नहीं है।
* /domain - पूर्ण नामित डोमेन नाम - डोमेन के बिना या स्थानिक उपयोगकर्ता/व्यवस्थापक के मामले में, कंप्यूटर या सर्वर नाम, कार्यसमूह या कुछ भी उपयोग करें।
* /rc4 या /ntlm - वैकल्पिक - उपयोगकर्ता के पासवर्ड की RC4 कुंजी / NTLM हैश।
* /run - वैकल्पिक - चलाने के लिए कमांड लाइन - डिफ़ॉल्ट है: शेल के लिए cmd।

[![Mimikatz-Sekurlsa-PTH](https://adsecurity.org/wp-content/uploads/2015/09/Mimikatz-Sekurlsa-PTH.jpg)](https://adsecurity.org/wp-content/uploads/2015/09/Mimikatz-Sekurlsa-PTH.jpg)

**SEKURLSA::Tickets** - हाल ही में प्रमाणित उपयोगकर्ताओं के लिए सभी उपलब्ध केरबेरोस टिकट देखें, जिनमें उपयोगकर्ता खाता और स्थानीय कंप्यूटर के AD कंप्यूटर खाता के संदर्भ में सेवाएं चल रही होती हैं।\
केरबेर
### वॉल्ट

`mimikatz.exe "privilege::debug" "token::elevate" "vault::cred /patch" "exit"` - निर्धारित कार्यों के पासवर्ड प्राप्त करें

\
\
\\

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने की अनुमति** चाहिए? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एकल [**NFT**](https://opensea.io/collection/the-peass-family) संग्रह देखें
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या मुझे **ट्विटर** पर **फ़ॉलो** करें [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**
* **अपने हैकिंग ट्रिक्स साझा करें,** [**hacktricks repo**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud repo**](https://github.com/carlospolop/hacktricks-cloud) **में PR जमा करके।**

</details>
