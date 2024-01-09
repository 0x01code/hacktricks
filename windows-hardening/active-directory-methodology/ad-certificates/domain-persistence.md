# AD CS डोमेन पर्सिस्टेंस

<details>

<summary><strong>AWS हैकिंग सीखें शून्य से लेकर हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा एक्सक्लूसिव [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram group**](https://t.me/peass) में या **Twitter** पर 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) को **फॉलो करें**.
* **HackTricks** के [**github repos**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) में PRs सबमिट करके अपनी हैकिंग ट्रिक्स शेयर करें.

</details>

## चोरी किए गए CA सर्टिफिकेट्स के साथ सर्टिफिकेट्स फोर्जिंग - DPERSIST1

आप कैसे बता सकते हैं कि एक सर्टिफिकेट CA सर्टिफिकेट है?

* CA सर्टिफिकेट **CA सर्वर पर ही मौजूद होता है**, इसकी **प्राइवेट की मशीन DPAPI द्वारा सुरक्षित होती है** (जब तक कि OS TPM/HSM/अन्य हार्डवेयर का उपयोग नहीं करता है).
* सर्ट के लिए **Issuer** और **Subject** दोनों CA के **विशिष्ट नाम पर सेट होते हैं**.
* CA सर्टिफिकेट्स (और केवल CA सर्ट्स) में **“CA Version” एक्सटेंशन होता है**.
* कोई भी EKUs नहीं होते हैं

इस सर्टिफिकेट प्राइवेट की को **निकालने का बिल्ट-इन GUI समर्थित तरीका** CA सर्वर पर `certsrv.msc` के साथ है.\
हालांकि, यह सर्टिफिकेट सिस्टम में संग्रहीत अन्य सर्टिफिकेट्स से **अलग नहीं है**, इसलिए उदाहरण के लिए [**THEFT2 तकनीक**](certificate-theft.md#user-certificate-theft-via-dpapi-theft2) देखें कि कैसे उन्हें **निकाला** जा सकता है.

आप [**certipy**](https://github.com/ly4k/Certipy) का उपयोग करके भी सर्ट और प्राइवेट की प्राप्त कर सकते हैं:
```bash
certipy ca 'corp.local/administrator@ca.corp.local' -hashes :123123.. -backup
```
एक बार जब आपके पास `.pfx` प्रारूप में निजी कुंजी के साथ **CA cert** होता है, तो आप मान्य प्रमाणपत्र बनाने के लिए [**ForgeCert**](https://github.com/GhostPack/ForgeCert) का उपयोग कर सकते हैं:
```bash
# Create new certificate with ForgeCert
ForgeCert.exe --CaCertPath ca.pfx --CaCertPassword Password123! --Subject "CN=User" --SubjectAltName localadmin@theshire.local --NewCertPath localadmin.pfx --NewCertPassword Password123!

# Create new certificate with certipy
certipy forge -ca-pfx CORP-DC-CA.pfx -upn administrator@corp.local -subject 'CN=Administrator,CN=Users,DC=CORP,DC=LOCAL'

# Use new certificate with Rubeus to authenticate
Rubeus.exe asktgt /user:localdomain /certificate:C:\ForgeCert\localadmin.pfx /password:Password123!

# User new certi with certipy to authenticate
certipy auth -pfx administrator_forged.pfx -dc-ip 172.16.126.128
```
{% hint style="warning" %}
**ध्यान दें**: प्रमाणपत्र बनाते समय निर्दिष्ट **उपयोगकर्ता** AD में **सक्रिय/सक्षम** होना चाहिए और **प्रमाणीकरण करने में सक्षम** होना चाहिए क्योंकि इस उपयोगकर्ता के रूप में प्रमाणीकरण विनिमय अभी भी होगा। उदाहरण के लिए, krbtgt खाते के लिए प्रमाणपत्र बनाने की कोशिश काम नहीं करेगी।
{% endhint %}

यह नकली प्रमाणपत्र निर्दिष्ट समाप्ति तिथि तक **मान्य** रहेगा और जब तक मूल CA प्रमाणपत्र मान्य है (आमतौर पर 5 से **10+ वर्षों** तक)। यह **मशीनों** के लिए भी मान्य है, इसलिए **S4U2Self** के साथ संयुक्त होकर, हमलावर किसी भी डोमेन मशीन पर जब तक CA प्रमाणपत्र मान्य है तब तक **निरंतरता बनाए रख सकता है**।\
इसके अलावा, इस विधि से **उत्पन्न प्रमाणपत्र** को **रद्द नहीं किया जा सकता** क्योंकि CA उनके बारे में जानकारी नहीं रखता है।

## रोग नियंत्रण CA प्रमाणपत्रों पर विश्वास - DPERSIST2

ऑब्जेक्ट `NTAuthCertificates` एक या अधिक **CA प्रमाणपत्रों** को अपने `cacertificate` **गुण** में परिभाषित करता है और AD इसका उपयोग करता है: प्रमाणीकरण के दौरान, **डोमेन नियंत्रक** जांचता है कि क्या **`NTAuthCertificates`** ऑब्जेक्ट में प्रमाणीकरण करने वाले **प्रमाणपत्र** के जारीकर्ता क्षेत्र में निर्दिष्ट **CA** के लिए एक प्रविष्टि **शामिल है**। यदि **हां, तो प्रमाणीकरण आगे बढ़ता है**।

एक हमलावर **स्व-हस्ताक्षरित CA प्रमाणपत्र** उत्पन्न कर सकता है और इसे **`NTAuthCertificates`** ऑब्जेक्ट में **जोड़** सकता है। हमलावर ऐसा कर सकते हैं यदि उनके पास **`NTAuthCertificates`** AD ऑब्जेक्ट पर **नियंत्रण** हो (डिफ़ॉल्ट कॉन्फ़िगरेशन में केवल **Enterprise Admin** समूह के सदस्य और **वन के मूल डोमेन** में **Domain Admins** या **Administrators** के सदस्यों के पास ये अनुमतियां होती हैं)। उच्च स्तरीय पहुंच के साथ, कोई भी `certutil.exe -dspublish -f C:\Temp\CERT.crt NTAuthCA126` का उपयोग करके या [**PKI Health Tool**](https://docs.microsoft.com/en-us/troubleshoot/windows-server/windows-security/import-third-party-ca-to-enterprise-ntauth-store#method-1---import-a-certificate-by-using-the-pki-health-tool) का उपयोग करके किसी भी सिस्टम से **`NTAuthCertificates`** ऑब्जेक्ट को **संपादित** कर सकता है।&#x20;

निर्दिष्ट प्रमाणपत्र को ForgeCert के साथ पहले विस्तृत नकली विधि के साथ **काम करना चाहिए** ताकि मांग पर प्रमाणपत्र उत्पन्न किए जा सकें।

## दुर्भावनापूर्ण गलत कॉन्फ़िगरेशन - DPERSIST3

AD CS घटकों के **सुरक्षा विवरण संशोधनों** के माध्यम से **निरंतरता** के लिए अनेक अवसर हैं। "[Domain Escalation](domain-escalation.md)" अनुभाग में वर्णित कोई भी परिदृश्य उच्च स्तरीय पहुंच वाले हमलावर द्वारा दुर्भावनापूर्ण रूप से लागू किया जा सकता है, साथ ही संवेदनशील घटकों के लिए "नियंत्रण अधिकारों" (अर्थात् WriteOwner/WriteDACL/आदि) का जोड़ भी। इसमें शामिल हैं:

* **CA सर्वर का AD कंप्यूटर** ऑब्जेक्ट
* **CA सर्वर का RPC/DCOM सर्वर**
* कंटेनर **`CN=Public Key Services,CN=Services,CN=Configuration,DC=<DOMAIN>,DC=<COM>`** में कोई भी **वंशज AD ऑब्जेक्ट या कंटेनर** (उदाहरण के लिए, Certificate Templates कंटेनर, Certification Authorities कंटेनर, NTAuthCertificates ऑब्जेक्ट, आदि)
* **AD समूह जिन्हें डिफ़ॉल्ट रूप से या वर्तमान संगठन द्वारा AD CS को नियंत्रित करने के अधिकार प्रदान किए गए हैं** (उदाहरण के लिए, निर्मित Cert Publishers समूह और उसके किसी भी सदस्य)

उदाहरण के लिए, डोमेन में **उच्च स्तरीय अनुमतियों** वाला एक हमलावर डिफ़ॉल्ट **`User`** प्रमाणपत्र टेम्पलेट में **`WriteOwner`** अनुमति जोड़ सकता है, जहां हमलावर अधिकार के लिए मुख्य है। इसका दुरुपयोग बाद में करने के लिए, हमलावर पहले **`User`** टेम्पलेट के स्वामित्व को खुद में बदल देगा, और फिर **`mspki-certificate-name-flag`** को **1** पर **सेट** करेगा ताकि **`ENROLLEE_SUPPLIES_SUBJECT`** सक्षम हो सके (अर्थात्, एक उपयोगकर्ता को अनुरोध में एक विषय वैकल्पिक नाम प्रदान करने की अनुमति देना)। फिर हमलावर **टेम्पलेट** में **नामांकन** कर सकता है, एक **डोमेन प्रशासक** नाम को एक वैकल्पिक नाम के रूप में निर्दिष्ट करते हुए, और DA के रूप में प्रमाणीकरण के लिए परिणामी प्रमाणपत्र का उपयोग कर सकता है।

## संदर्भ

* इस पृष्ठ की सभी जानकारी [https://www.specterops.io/assets/resources/Certified_Pre-Owned.pdf](https://www.specterops.io/assets/resources/Certified_Pre-Owned.pdf) से ली गई थी।

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong></a><strong>!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें** तो [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) संग्रह
* 💬 [**Discord समूह**](https://discord.gg/hRep4RUj7f) में **शामिल हों** या [**telegram समूह**](https://t.me/peass) में शामिल हों या मुझे **Twitter** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) पर **अनुसरण करें**।
* **HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs जमा करके अपनी हैकिंग तरकीबें साझा करें।

</details>
