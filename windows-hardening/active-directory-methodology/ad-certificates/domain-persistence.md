# AD CS डोमेन स्थिरता

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

- क्या आप **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS के नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की आवश्यकता है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!

- खोजें [**The PEASS Family**](https://opensea.io/collection/the-peass-family), हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)

- प्राप्त करें [**आधिकारिक PEASS और HackTricks swag**](https://peass.creator-spring.com)

- **शामिल हों** [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **फॉलो** करें मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks_live)**.**

- **अपने हैकिंग ट्रिक्स को [hacktricks रेपो](https://github.com/carlospolop/hacktricks) और [hacktricks-cloud रेपो](https://github.com/carlospolop/hacktricks-cloud) में पीआर जमा करके साझा करें।**

</details>

## चोरी किए गए सीए प्रमाणपत्रों के साथ प्रमाणपत्र बनाना - DPERSIST1

आप कैसे पता लगा सकते हैं कि प्रमाणपत्र एक सीए प्रमाणपत्र है?

* सीए प्रमाणपत्र **सीए सर्वर पर मौजूद होता है**, जिसकी **निजी कुंजी मशीन DPAPI द्वारा सुरक्षित होती है** (यदि ऑपरेटिंग सिस्टम TPM/HSM/अन्य हार्डवेयर का उपयोग करता है तो नहीं।)
* प्रमाणपत्र के **जारक** और **विषय** दोनों को **सीए के विभाजनीय नाम** पर सेट किया जाता है।
* सीए प्रमाणपत्र (और केवल सीए प्रमाणपत्र) **में "सीए संस्करण" विस्तार** होता है।
* **कोई EKU नहीं होती है**

इस प्रमाणपत्र निजी कुंजी को **चोरी करने के लिए** बिल्ट-इन GUI समर्थित तरीका `certsrv.msc` है।\
हालांकि, यह प्रमाणपत्र **सिस्टम में संग्रहीत अन्य प्रमाणपत्रों** से अलग नहीं है, इसलिए उदाहरण के लिए [**THEFT2 तकनीक**](certificate-theft.md#user-certificate-theft-via-dpapi-theft2) की जांच करें और देखें कि इन्हें कैसे **निकालें**।

आप [**certipy**](https://github.com/ly4k/Certipy) का उपयोग करके प्रमाणपत्र और निजी कुंजी प्राप्त कर सकते हैं:
```bash
certipy ca 'corp.local/administrator@ca.corp.local' -hashes :123123.. -backup
```
जब आपके पास `.pfx` प्रारूप में **CA प्रमाणपत्र** के साथ निजी कुंजी होती है, तो आप [**ForgeCert**](https://github.com/GhostPack/ForgeCert) का उपयोग करके मान्य प्रमाणपत्र बना सकते हैं:
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
**नोट**: प्रमाणपत्र बनाने के समय निर्माण किए जाने वाले प्रमाणपत्र के लिए निश्चित किए जाने वाले उपयोगकर्ता को AD में सक्रिय/सक्षम होना चाहिए और प्रमाणीकरण का आदान-प्रदान होना चाहिए क्योंकि इस उपयोगकर्ता के रूप में एक प्रमाणीकरण विनिमय अभी भी होगा। उदाहरण के लिए, krbtgt खाते के लिए एक प्रमाणपत्र निर्मित करने का प्रयास काम नहीं करेगा।
{% endhint %}

यह निर्मित प्रमाणपत्र निर्दिष्ट समाप्ति तिथि तक **मान्य** रहेगा और जब तक मूल CA प्रमाणपत्र मान्य होता है (आमतौर पर 5 से **10+ वर्ष** तक)। यह मशीनों के लिए भी मान्य है, इसलिए **S4U2Self** के साथ मिलाकर, एक हमलावर किसी भी डोमेन मशीन पर **स्थायित्व बनाए रख सकता है** जब तक CA प्रमाणपत्र मान्य होता है।\
इसके अलावा, इस विधि के साथ उत्पन्न **प्रमाणपत्र रद्द नहीं किए जा सकते** क्योंकि CA उनके बारे में जागरूक नहीं है।

## धोखाधड़ी वाले CA प्रमाणपत्रों पर विश्वास - DPERSIST2

वस्त्राधिकारियों को अपने पास **नियंत्रण** होने की स्थिति में होने पर एक **स्व-हस्ताक्षरित CA प्रमाणपत्र** उत्पन्न करने और इसे **`NTAuthCertificates`** ऑब्जेक्ट में जोड़ने की अनुमति होती है। हमलावरों को इसे कर सकते हैं यदि उनके पास **`NTAuthCertificates`** AD ऑब्जेक्ट पर नियंत्रण है (डिफ़ॉल्ट कॉन्फ़िगरेशन में केवल **एंटरप्राइज़ एडमिन** समूह के सदस्य और **डोमेन एडमिन्स** या **व्यवस्थापकों** के सदस्य इन अनुमतियों को होती है)। उच्च पहुंच के साथ, कोई भी सिस्टम से `certutil.exe -dspublish -f C:\Temp\CERT.crt NTAuthCA126` का उपयोग करके या [**PKI स्वास्थ्य टूल**](https://docs.microsoft.com/en-us/troubleshoot/windows-server/windows-security/import-third-party-ca-to-enterprise-ntauth-store#method-1---import-a-certificate-by-using-the-pki-health-tool) का उपयोग करके उपयोगकर्ता **`NTAuthCertificates`** ऑब्जेक्ट को **संपादित** कर सकता है।

निर्दिष्ट प्रमाणपत्र को **ForgeCert** के साथ पहले विस्तृत जालसाजी विधि के साथ काम करना चाहिए ताकि आवश्यकता के अनुसार प्रमाणपत्र उत्पन्न किए जा सकें।

## दुर्भाग्यपूर्ण गलत विन्यास - DPERSIST3

AD CS के सुरक्षा विवरणकारी घटकों के **स्थायित्व** के लिए **सुरक्षा विवरणकारी संशोधन** के माध्यम से असंख्य अवसर हैं। किसी भी परिदृश्य में जो "[डोमेन उन्नयन](domain-escalation.md)" अनुभाग में वर्णित है, उसे उच्च पहुंच वाले हमलावर द्वारा दुर्भाग्यपूर्ण रूप से लागू किया जा सकता है, साथ ही "नियंत्रण अधिकार" (जैसे, WriteOwner/WriteDACL/आदि) को संवेदनशील घटकों में जोड़ने का भी समावेश है। इसमें शामिल हैं:

* **CA सर्वर के AD कंप्यूटर** ऑब्जेक्ट
* CA सर्वर के RPC/DCOM सर्वर
* कंटेनर **`CN=Public Key Services,CN=Services,CN=Configuration,DC=<DOMAIN>,DC=<COM>`** में किसी भी **अवतरण AD ऑब्जेक्ट या कंटेनर** (उदाहरण के लिए, प्रमाणपत्र टेम्पलेट्स कंटेनर, प्रमाणीकरण प्राधिकरण कंटेनर, NTAuthCertificates ऑब्जेक्ट, आदि)
* **डिफ़ॉल्ट रूप से या वर्तमान संगठन द्वारा AD CS को नियंत्रित करने के लिए अधिकार धारित AD समूह** (उदाहरण के लिए, इंटीग्रेटेड Cert प्रकाशक समूह और इसके सदस्यों में से कोई भी)

उदाहरण के लिए, डोमेन में **उच्च पहुंच वाले अनुमतियों** के साथ हमलावर डिफ़ॉल्ट **`User`** प्रमाणपत्र टेम्पलेट में **`WriteOwner`** अनुमति जोड़ सकता है, जहां हमलावर अधिकार के लिए मुख्य है। इसे बाद में दुरुपयोग करने के लिए, हमलावर को पहले **`User`** टेम्पलेट के स्वामित्व को खुद के नाम पर संशोधित करना होगा, और फिर टेम्पलेट पर **`mspki-certificate-name-flag`** को **1** सेट करेगा ताकि **`ENROLLEE_SUPPLIES_SUBJECT`** को सक्षम करें (अर्थात एक उपयोगकर्ता को
